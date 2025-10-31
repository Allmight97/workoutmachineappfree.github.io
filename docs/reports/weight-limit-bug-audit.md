# Weight Limit Feature Audit (2025-11-19)

## Summary
The per-cable load limit implementation in `app.js` applies progression updates during warmup, forces the cap without a safe telemetry window, and lacks the validation rules documented in `docs/planning/Features/feat_weight_limit.md`. These gaps explain the field reports in `docs/releases/CHANGELOG.md` for Bugs A–C.

## Findings
- **Warmup progression causes early cap** (app.js:603-645): `updateProgramTrackingAfterRep()` runs every completed rep, including warmups. With non-zero progression, `currentProgramParams.perCableKg` advances during warmup and `maybeApplyLoadLimit()` clamps before working reps begin. Users experience the jump from unloaded reps straight to the limit (Bug B) and a Just Lift-like plateau for the remainder of the program (Bug C).
- **Cap updates ignore safe window** (app.js:603-645): `maybeApplyLoadLimit()` always invokes `device.updateProgramParams()` immediately after detecting the cap. There is no check that both cables are unloaded, violating the spec's safety model and aligning with Bug A (session aborts or drops load when limit is set).
- **Missing limit validation rules** (app.js:1355-1384): Inputs allow limits lower than the starting weight for positive progression (and vice versa). The invalid combination silently passes, triggering the cap on the first limit check and reproducing the "session collapses" symptoms.
- **UI always exposes limit field** (index.html:648-678): The field is visible even when progression is 0, encouraging users to set unsupported combinations and obscuring the true trigger for Bug A. While secondary, it contradicts the planning doc’s conditional UX.

## Telemetry Capture Recommendations
To validate the hypotheses and design safe-window logic:
1. **Console Log Snapshot**: Enable test mode (`?testMode=true`) and reproduce the bug with real hardware. Capture logs around:
   - The first `BOTTOM detected!` message (warmup completion)
   - `Per-cable load limit applied at ...`
   - Any `Sending updated program frame ...` lines
2. **Monitor Samples**: Record the timestamped `monitor` characteristic dumps (JSON or raw hex) covering the transition from warmup to working reps. Verify loads and force values at the moment the update fires.
3. **Reproduction Matrix**: For each run, note: starting weight, progression, limit, mode, Just Lift toggle, observed rep count before load change, and whether the workout auto-completed. This helps isolate validation gaps.
4. **Command Timing**: Export BLE operation timestamps (queue logs) to confirm whether `updateProgramParams` executes while loads are non-zero; optionally video the console if direct export isn’t available.

## Greenfield Implementation Plan
1. **Program Plan Model**
   - Create a small module (e.g., `programLimitPlan.js`) that encapsulates session math: starting weight, progression, limit, rep index. Provide pure helpers (`computeTarget(repIndex)`, `shouldSaturate(repIndex)`, `isValid()` with start/limit checks).
   - Value: isolates logic, enables unit tests, keeps controller lean (Design 4/5, DRY 4/5).

2. **Safe-Window Gate**
   - Add a reusable telemetry helper (e.g., `safeWindow.js`) subscribing to monitor samples and emitting `onUnloaded` events when both cables report load below a threshold for a minimum window.
   - Value: enforces the spec’s “only adjust when unloaded” rule; shared with existing Just Lift auto-stop logic (SoC 4/5, Fail Fast 4/5).

3. **Controller Integration**
   - In `startProgram()`, build a `ProgramPlan` from inputs, validate, and store it alongside session state. Hide/disable the limit field when progression is neutral.
   - On each rep completion (after warmup), request the next target from the plan. If saturation occurs, enqueue `device.updateProgramParams()` inside the safe-window callback.
   - Value: maintains existing workflow while making progression explicit and testable (KISS 4/5).

4. **Validation & UX**
   - Enforce limit ≥ start for positive progression and limit ≤ start for negative progression; reject invalid combos with alerts and disable Start until resolved.
   - Toggle the limit field visibility/labels based on progression sign to match `feat_weight_limit.md`.

5. **Testing**
   - Unit-test the plan helpers for positive/negative progression, early saturation, and “no limit” cases.
   - Simulate monitor samples to exercise safe-window gating in test mode.
   - Hardware smoke: confirm capped sessions ramp correctly and Just Lift holds at the limit without auto-stop glitches.

## Migration Bridge from Current Codebase
1. **Add Safe-Window Instrumentation First**
   - Introduce a non-invasive telemetry helper that logs when loads fall below the rest threshold. This can coexist with the current limit logic and informs threshold tuning without behavior changes.

2. **Refactor Progression Tracking**
   - Split warmup vs working progression inside `updateProgramTrackingAfterRep()`; ensure warmup reps no longer increment `perCableKg`. This change fixes the premature cap while leaving the rest of the logic intact.

3. **Strengthen Validation**
   - Update `startProgram()` validation to enforce limit/start relationships and hide/disable the input when progression is zero. This is low risk and fails fast at the UI boundary.

4. **Introduce Program Plan Module**
   - Extract progression math into a helper module and wire it in, preserving current behavior. Once plan outputs match existing expectations (minus the warmup fix), remove redundant controller math.

5. **Replace Immediate Cap with Safe-Window Update**
   - Swap the direct `device.updateProgramParams()` call with the safe-window callback that queues the update only when unloaded. Ensure telemetry logs confirm the window before sending.

6. **Cleanup & DRY Pass**
   - After behavior is correct and telemetry validated, consider revisiting PR C to streamline parsing/validation helpers and keep the controller maintainable.

Each step delivers value while keeping diffs focused: stopping early cap regressions, protecting against mid-load updates, and aligning the UI/validation with the documented spec.
