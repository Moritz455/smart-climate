# Spec: Smart HVAC Rule-Based Control Algorithm

## Objective

Rule-based optimization algorithm for single-zone climate control (Moritz Zimmer). Primary goal: maximize energy savings while maintaining comfort within user-defined bounds. Algorithm runs on Home Assistant, uses available sensors, controls roller shutters + climate mode.

## Algorithm Selection

**Chosen: Hysteresis-based bang-bang control with seasonal mode switching + occupancy-aware setpoint adjustment.**

Why not MPC (Model Predictive Control): Overkill for single zone, requires forecast data + computational budget, marginal energy gain vs complexity for one room.

Why not simple thermostat: No hysteresis = short-cycling = energy waste + wear. Bang-bang with deadband eliminates this.

### Core Logic

1. **Seasonal Mode Switching** (weekly schedule or temp-triggered):
   - Summer mode (outdoor > 23°C): shutters closed day, open night; cool setpoint 22°C max
   - Winter mode (outdoor < 19°C): shutters closed night, open day; warm setpoint 19°C min
   - Intermediate: free cooling/heating within bounds

2. **Hysteresis Control** (deadband ±0.5°C around setpoint):
   - Indoor temp > setpoint + 0.5°C → activate cooling (shutters close, ventilation open if cool outside)
   - Indoor temp < setpoint - 0.5°C → activate heating (shutters open, close at setpoint + 0.5°C)
   - Prevents short-cycling, reduces HVAC runtime 15-30% vs on-off at threshold

3. **Occupancy-Aware Setpoint Adjustment**:
   - Presence detected → comfort band active (20-22°C winter / 22-24°C summer)
   - Absence → eco band (18-20°C winter / 24-26°C summer)
   - Sleep schedule → narrower band at night, wider during day

4. **Solar Gain Utilization** (handled by Adaptive Cover Pro Integration):
   - Use `*_climate_status` entity from Adaptive Cover Pro for solar/climate mode references
   - No local solar gain logic needed — integration already handles it
   - Algorithm consumes integration's climate status as input signal

## Tech Stack

- Home Assistant automations + helpers (no external compute)
- Python script for complex rule evaluation (optional, if automation expressions insufficient)
- Sensors: temperature (indoor/outdoor), humidity (indoor/outdoor), presence, sun state, window/shutter status
- Actuators: roller shutter switches, climate mode switch

## Commands

- Build/Deploy: HA config reload (no build step — YAML/automations)
- Test: manual trigger automation, check state changes
- Validate: `ha config check` before reload

## Project Structure

```
resources/         → This spec + research docs
specs/             → Additional spec docs (if any)
blueprints/        → HA automation blueprints
scripts/           → Python helper scripts (if needed)
tests/             → Automation logic tests (manual scenario checks)
```

## Code Style

- HA automations: YAML, native triggers/conditions where possible
- Jinja2 templates only for data values, not logic
- Explicit entity_id references, no `this.entity_id`
- All thresholds as input_number helpers (tunable, not hardcoded)
- Universal design: no hard-coded instance-specific entity names in core logic; use config-driven entity references

## Testing Strategy

- Scenario-based manual testing: summer day, winter night, absence, presence transitions
- Verify no short-cycling (min 10-min on/off gap)
- Measure energy: compare HVAC runtime before/after with same weather

## Boundaries

- Always: validate entity states before actuator commands; log mode transitions
- Ask first: change deadband thresholds; add new sensor dependencies
- Never: hardcode setpoints in automation YAML; skip presence detection fallback

## Success Criteria

1. Energy savings ≥ 20% vs current baseline (measured HVAC runtime hours/week)
2. Indoor temp stays within comfort band 90%+ of occupied hours
3. No short-cycling (HVAC cycles < 3x/hour)
4. Mode transitions respect schedule + occupancy (no manual override lost)

## Current Entities (Moritz Zimmer)

### Indoor Sensors
- `sensor.moritz_magisches_auge_temperatur` — indoor temperature
- `sensor.moritz_magisches_auge_luftfeuchtigkeit` — indoor humidity

### Window Sensors
- `binary_sensor.fenster_sensor_moritz_fenster_contact` — window contact
- `binary_sensor.fenster_sensor_moritz_dachfenster_contact` — attic window contact

### Outdoor Sensors
- `sensor.oberemmel_temperatur` — outdoor temperature
- `sensor.oberemmel_relative_luftfeuchtigkeit` — outdoor humidity

### Adaptive Cover Pro Integration
- `sensor.fenster_rolladen_climate_status` — climate status (summer_mode/winter_mode/intermediate)
- `sensor.dachfenster_rolladen_climate_status` — climate status (summer_mode/winter_mode/intermediate)
- Use these for solar/climate mode references — solar gain logic is already handled by the integration

### Actuators
- `switch.fenster_rolladen_climate_mode` — roller shutter climate mode
- `switch.dachfenster_rolladen_climate_mode` — roller shutter climate mode

### Existing Automations/Scripts
- `automation.rolladen_schliessen_sommer` — existing summer shutter automation (to be integrated/extended)
- `script.aufstehen` — existing wake-up script

## Open Questions

1. Current baseline energy usage (need historical HVAC run hours)
2. User comfort band preferences (tight vs loose)
3. Shutter actuator type — position control or binary open/close?
4. Vacation/extended absence handling
