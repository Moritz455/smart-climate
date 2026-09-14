# Phase 3: Task Breakdown — Smart HVAC Rule-Based Control

Each task follows the template:
- [ ] Task: [Description]
  - Acceptance: [What must be true when done]
  - Verify: [How to confirm — test command, build, manual check]
  - Files: [Which files will be touched]

---

## Task T1: Climate Mode Helper Setup

- [ ] Task: Setup input_select for seasonal modes and input_number for deadband thresholds
  - Acceptance:
    - `input_select.moritz_seasonal_mode` exists with values: "summer", "winter", "intermediate"
    - `input_number.moritz_deadband` exists with value 0.5 (°C)
    - `input_number.moritz_indoor_setpoint` exists with default 21 (°C)
    - `input_number.moritz_outdoor_threshold` exists with values 19, 23 (°C)
    - All entities visible in HA entity registry
  - Verify:
    - HA config check passes: `ha config check`
    - All entities listed in states list: `ha get_state` each entity
  - Files created:
    - `specs/climate_helpers.md` (entity reference table)
    - `blueprints/climate_helpers.yaml` (optional backup config)

---

## Task T2: Basic Hysteresis Control Automation

- [ ] Task: Implement core hysteresis-based heating/cooling automation
  - Acceptance:
    - Automation `automation.hysteresis_control` created with triggers on sensor.moritz_magisches_auge_temperatur
    - Logic: if temp > setpoint + 0.5°C → cooling activate (shutters close); if temp < setpoint - 0.5°C → heating activate (shutters open)
    - Hysteresis deadband prevents short-cycling (min 10-min gap between cycles)
    - Automation respects input_select.moritz_seasonal_mode
  - Verify:
    - Automation triggers correctly when temp crosses thresholds
    - No rapid on/off toggling (check entity history)
    - Manual override from UI works
  - Files modified:
    - `blueprints/climate_automation.yaml` (new automation)
    - `.storage/` (entity state updates, auto-generated)

---

## Task T3: Seasonal Mode Automation

- [ ] Task: Implement outdoor-temperature-triggered seasonal mode switching
  - Acceptance:
    - Automation `automation.seasonal_mode_switch` created
    - Logic: outdoor temp > 23°C → summer mode; outdoor temp < 19°C → winter mode; between → intermediate
    - Mode changes persist across HA restarts
    - Integration with Adaptive Cover Pro climate_status entities
  - Verify:
    - Mode switches correctly at threshold crossings
    - Summer mode: shutters closed day, open night
    - Winter mode: shutters open day (solar), closed night
    - HA config reload preserves mode settings
  - Files modified:
    - `blueprints/seasonal_switch.yaml` (new automation)
    - `automations/climate/` directory updates

---

## Task T4: Occupancy-Aware Setpoint Adjustment

- [ ] Task: Implement presence-detecting eco/comfort mode switching
  - Acceptance:
    - Automation logic: presence detected → comfort band active; absence → eco band
    - Comfort band: 20-22°C winter / 22-24°C summer
    - Eco band: 18-20°C winter / 24-26°C summer
    - Sleep schedule support: narrower band at night (22-23°C winter / 23-24°C summer)
  - Verify:
    - Presence triggers comfort mode switch
    - Absence triggers eco mode after timeout (default 30 min)
    - Sleep schedule active during configured hours
    - Manual override respected
  - Files modified:
    - `blueprints/occupancy_adjust.yaml` (new automation/script)
    - May add `input_boolean.moritz_presence_eco_mode` helper

---

## Task T5: Solar Integration Coordination

- [ ] Task: Coordinate with Adaptive Cover Pro climate_status entities
  - Acceptance:
    - Algorithm consumes sensor.fenster_rolladen_climate_status and sensor.dachfenster_rolladen_climate_status
    - Solar gain override logic passed to Adaptive Cover Pro (no local solar logic needed)
    - Climate mode respects integration's seasonal state
    - No conflicts between local automation and Adaptive Cover Pro
  - Verify:
    - Climate status entities update when automation runs
    - No duplicate mode commands sent to shutters
    - Summer/winter modes align with Adaptive Cover Pro state
    - HA config check passes
  - Files modified:
    - `blueprints/solar_coordination.yaml` (coordination automation)
    - Existing Adaptive Cover Pro configs preserved

---

## Task T6: Dashboard & Monitoring

- [ ] Task: Create visual climate control dashboard
  - Acceptance:
    - Dashboard accessible at Lovelace path: `climate-control`
    - Displays: current indoor temp, outdoor temp, seasonal mode, setpoint, HVAC status
    - Shows: short-cycle counter (if active)
    - Manual override buttons for all modes
  - Verify:
    - Dashboard loads without errors
    - All entity states display correctly
    - Manual overrides update UI and automation state
    - Dashboard responsive on mobile
  - Files created:
    - `dashboard/climate_dashboard.yaml`
    - `specs/dashboard-spec.md`

---

## Task T7: Testing & Validation Suite

- [ ] Task: Create manual test scenarios and verify all success criteria
  - Acceptance:
    - Scenario 1: Summer day — comfort maintained, no short-cycling
    - Scenario 2: Winter night — eco mode active, energy savings
    - Scenario 3: Absence → presence transition — comfort mode resumes
    - Scenario 4: Mode switching — summer ↔ winter ↔ intermediate
    - All scenarios pass acceptance criteria
  - Verify:
    - Each scenario test documented and run
    - Energy savings measurement (baseline vs after)
    - Success criteria met (see spec § Success Criteria)
    - Edge cases handled gracefully
  - Files created:
    - `tests/hvac_test_scenarios.md`
    - `tests/hvac_test_results.md`

---

## Task T8: Final Validation & Documentation

- [ ] Task: Final integration test and spec sign-off
  - Acceptance:
    - HA config check passes: `ha config check`
    - All automations functional
    - Performance targets met (20% energy savings, zero short-cycling)
    - Success criteria from spec verified
    - Documentation complete and reviewed
  - Verify:
    - Run all tasks T1-T7 acceptance criteria
    - Generate test report from T7
    - Review spec success criteria checklist complete
    - Commit spec and plan to version control
  - Files:
    - `specs/SPEC-hvac-control.md` final review
    - `specs/technical-plan.md` final review
    - All task completions marked in task list