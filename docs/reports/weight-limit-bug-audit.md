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

## Next Steps
- Rework progression tracking to split warmup vs working reps before applying caps.
- Introduce a safe-window detector that only sends capped frames when both cables report near-zero load for a fixed interval.
- Enforce limit vs starting weight comparisons and hide/disable the field when progression is neutral to reduce invalid input combinations.
