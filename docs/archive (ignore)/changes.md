# Vitruvian Trainer Rescue - Change Plan
**Last Updated:** October 26, 2025  
**Purpose:** Track proposed changes for improving the Vitruvian Trainer web control application  
**Approach:** Each change ships as its own PR to maintain focused scope and enable incremental delivery  

---

## Change 1: Implement Safe Auto-Stop for Just Lift Mode
**PR Scope:** Enable Just Lift mode with safety-first auto-stop mechanism

### What / Why / Impact
- **What:** Implement automatic stop detection for Just Lift mode based on cable position stagnation
- **Why:** Currently disabled for safety; users need free-lift capability for warmups and testing
- **Impact:** Enables core functionality that users expect from their hardware

### Impact Analysis
- **1st Order (Immediate):** Just Lift mode becomes available; users gain free-lifting capability
- **2nd Order (Adjacent Systems):** Rep detection logic needs adjustment; telemetry monitoring must track position deltas; UI must clearly indicate auto-stop status
- **3rd Order (Long-term):** Sets safety precedent for all modes; establishes pattern for position-based safety checks; may require tuning based on hardware variations

### Implementation Notes
- Monitor cable positions over rolling window (e.g., 2-3 seconds)
- Trigger stop when position variance falls below threshold
- Add visual indicator for auto-stop armed/triggered state
- Preserve manual STOP button as primary safety control

---

## Change 2: Add Robust Reconnection Flow
**PR Scope:** Implement graceful BLE reconnection with state preservation

### What / Why / Impact
- **What:** Add automatic reconnection attempts with exponential backoff after GATT disconnects
- **Why:** BLE connections drop frequently due to signal issues; current flow requires full restart
- **Impact:** Reduces user frustration and preserves workout continuity

### Impact Analysis
- **1st Order (Immediate):** Connection drops recover automatically; UI shows reconnection status
- **2nd Order (Adjacent Systems):** Device state must be cached; telemetry gaps need handling; workout history needs continuity markers
- **3rd Order (Long-term):** Enables background operation patterns; improves perceived reliability; may mask underlying hardware issues

### Implementation Notes
- Cache last known device ID for quick reconnect
- Implement 3-attempt retry with 2s, 4s, 8s backoff
- Preserve workout state during brief disconnects (<30s)
- Clear visual feedback during reconnection attempts

---

## Change 3: Add Workout History Persistence
**PR Scope:** Store and display workout history using localStorage

### What / Why / Impact
- **What:** Persist completed workouts to browser localStorage with export capability
- **Why:** Users need to track progress over time; current state is lost on page refresh
- **Impact:** Enables progress tracking without external dependencies

### Impact Analysis
- **1st Order (Immediate):** Workouts saved locally; history viewable across sessions
- **2nd Order (Adjacent Systems):** Need data migration strategy; export format standardization; storage quota management
- **3rd Order (Long-term):** Foundation for analytics features; enables workout sharing; creates data portability expectations

### Implementation Notes
- Store as timestamped JSON with mode, weight, reps, duration
- Implement 90-day rolling window to manage storage
- Add CSV export for external analysis
- Include data version field for future migrations

---

## Change 4: Enhance Error Recovery and Diagnostics
**PR Scope:** Improve error handling with actionable user feedback

### What / Why / Impact
- **What:** Add structured error recovery flows with clear user guidance
- **Why:** Current errors show technical details; users need actionable next steps
- **Impact:** Reduces support burden and improves self-service troubleshooting

### Impact Analysis
- **1st Order (Immediate):** Errors display user-friendly messages with recovery steps
- **2nd Order (Adjacent Systems):** Logging system needs structure; help documentation needs linking; state machine needs recovery paths
- **3rd Order (Long-term):** Enables community-driven troubleshooting; reduces abandonment rate; builds error pattern database

### Implementation Notes
- Map technical errors to user messages
- Add "Try This" suggestions for common failures
- Include diagnostic data collection (with consent)
- Link to relevant documentation sections

---

## Change 5: Implement Custom Workout Programs
**PR Scope:** Add ability to create and save custom workout parameters

### What / Why / Impact
- **What:** Allow users to define custom rep ranges, rest periods, and resistance curves
- **Why:** Fixed modes don't match all training styles; users want personalization
- **Impact:** Extends hardware utility beyond original app capabilities

### Impact Analysis
- **1st Order (Immediate):** Custom programs creatable and executable; UI for parameter input
- **2nd Order (Adjacent Systems):** Protocol validation for edge cases; preset management system; import/export format definition
- **3rd Order (Long-term):** Community program sharing; safety validation responsibilities; potential hardware warranty concerns

### Implementation Notes
- Validate parameters against safe operating ranges
- Store presets in localStorage with unique IDs
- Add import/export via JSON format
- Include disclaimer about custom program risks

---

## Change 6: Add Real-Time Form Feedback
**PR Scope:** Implement velocity and symmetry analysis with visual feedback

### What / Why / Impact
- **What:** Analyze cable velocity and left/right balance to provide form coaching
- **Why:** Proper form prevents injury; asymmetry indicates compensation patterns
- **Impact:** Adds value beyond original app through real-time biomechanical feedback

### Impact Analysis
- **1st Order (Immediate):** Velocity and balance metrics displayed; form alerts triggered
- **2nd Order (Adjacent Systems):** Telemetry processing needs enhancement; UI needs feedback widgets; threshold tuning required
- **3rd Order (Long-term):** Creates coaching framework; enables injury prevention features; may influence training methodology

### Implementation Notes
- Calculate velocity from position deltas
- Track left/right load differential percentage
- Visual indicators for tempo and balance
- Configurable thresholds for different exercises

---

## Change 7: Optimize Canvas Rendering Performance
**PR Scope:** Improve graph rendering efficiency for smoother visualization

### What / Why / Impact
- **What:** Implement requestAnimationFrame-based rendering with dirty rectangle optimization
- **Why:** Current implementation redraws entire canvas; causes stuttering on lower-end devices
- **Impact:** Smoother user experience and reduced battery consumption

### Impact Analysis
- **1st Order (Immediate):** Smoother graph updates; reduced CPU usage
- **2nd Order (Adjacent Systems):** Animation timing coordination; other UI updates need synchronization; memory management for render buffers
- **3rd Order (Long-term):** Enables more complex visualizations; improves mobile device support; sets performance baseline

### Implementation Notes
- Use requestAnimationFrame for render loop
- Track dirty regions to minimize redraws
- Implement double buffering if needed
- Add FPS counter for performance monitoring

---

## Change 8: Add Accessibility Improvements
**PR Scope:** Enhance keyboard navigation and screen reader support

### What / Why / Impact
- **What:** Add ARIA labels, keyboard shortcuts, and focus management
- **Why:** Current UI requires precise clicking; excludes users with disabilities
- **Impact:** Makes application usable by wider audience per WCAG guidelines

### Impact Analysis
- **1st Order (Immediate):** Keyboard navigation works; screen readers announce states
- **2nd Order (Adjacent Systems):** Focus trap management; shortcut conflict resolution; help documentation for shortcuts
- **3rd Order (Long-term):** Sets accessibility standard; influences future UI decisions; may require alternative interaction patterns

### Implementation Notes
- Add ARIA live regions for status updates
- Implement tab order for controls
- Keyboard shortcuts for start/stop/connect
- High contrast mode support

---

## Implementation Priority
Based on user safety and value delivery:
1. **High Priority:** Changes 1, 2, 4 (safety and reliability)
2. **Medium Priority:** Changes 3, 5, 6 (enhanced functionality)
3. **Lower Priority:** Changes 7, 8 (optimization and accessibility)

## Testing Requirements
Each PR must include:
- Hardware validation for protocol changes
- Cross-browser testing (Chrome, Edge, Opera)
- Edge case scenarios per change scope
- Regression testing of existing features
- Documentation updates where applicable

## Risk Mitigation
- Each change ships independently to limit blast radius
- Feature flags for experimental changes where appropriate
- Rollback plan documented in PR description
- Community testing before marking stable

---

*Note: This plan follows AGENTS.md principles with focus on KISS, YAGNI, and Fail Fast. Each change maintains orthogonality and ships as its own PR.*
