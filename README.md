# Smart Climate — Smart HVAC Rule-Based Control

Home Assistant-based HVAC control for a single-zone living space. The system uses hysteresis-based bang-bang control with seasonal mode switching and occupancy-aware setpoint adjustment to maximize energy savings while keeping indoor temperature within a user-defined comfort band.

## Why This Project Exists

A simple thermostat oscillates at the setpoint and causes short-cycling, which wastes energy and wears out HVAC equipment. This project replaces that behavior with a deadband-based controller that keeps the compressor running in stable blocks and adjusts setpoints based on season, occupancy, and available solar gain — all without external compute beyond Home Assistant.

## How It Works

1. **Seasonal Mode Switching** — outdoor temperature triggers `summer`, `winter`, or `intermediate` modes, each with different setpoint bounds and shutter behavior.
2. **Hysteresis Control** — a configurable deadband (±0.5°C default) around the active setpoint prevents rapid on/off toggling and reduces HVAC runtime by 15–30% compared to simple on-off control.
3. **Occupancy-Aware Setpoints** — presence detection widens the band to an eco range when nobody is home and narrows it back to comfort when someone returns.
4. **Solar Gain Coordination** — the Adaptive Cover Pro integration's `*_climate_status` entities feed seasonal state into the controller, so solar logic stays in one place.

## Success Criteria

- Energy savings ≥ 20% vs. the previous baseline (measured HVAC runtime hours/week)
- Indoor temperature stays within the comfort band ≥ 90% of occupied hours
- Zero short-cycling (HVAC cycles < 3× per hour)
- Mode transitions respect schedule and occupancy (< 30 s)
- System uptime ≥ 99.5%

## Project Structure

```
.
├── compose.yaml                  # Docker Compose for the Home Assistant test environment
├── ha-config/                    # Home Assistant runtime config (runtime state only — not tracked)
│   ├── configuration.yaml        # Core HA configuration and entity definitions
│   ├── blueprints/               # Automation blueprints
│   │   ├── automation/
│   │   ├── template/
│   │   └── script/
│   └── scripts/                  # Optional Python helper scripts
├── specs/                        # Specification documents
│   ├── SPEC-hvac-control.md      # Algorithm spec and entity inventory
│   └── technical-plan.md         # Implementation plan and checkpoints
├── tasks/                        # Task breakdown for the current phase
│   └── task-list.md
├── resources/                    # Research papers and reports
└── tests/                        # Manual test scenarios and results
```

## Quick Start

### Prerequisites
- Docker and Docker Compose installed
- At least 4 GB RAM available
- Ports `8123` (and `22` if SSH is needed) available

### Run the HA Test Environment
```bash
docker-compose up -d
docker logs -f homeassistant
```
Open [http://localhost:8123](http://localhost:8123) after the container initializes (2–3 minutes).

### Validate Configuration
```bash
# Run the blueprint validation script
python3 ha-config/scripts/test_hvac.py --config-dir ha-config --verbose

# Or validate via the HA CLI inside the container
docker exec homeassistant ha config check
```

### Stop
```bash
docker-compose down
```

## Development

All control logic lives in Home Assistant YAML automations and helpers — no external build step. Thresholds are stored in `input_number` helpers rather than hardcoded in automations so they remain tunable without editing code.

- **Hysteresis deadband**: `input_number.moritz_deadband`
- **Seasonal setpoints**: controlled by `input_select.moritz_seasonal_mode`
- **Outdoor thresholds**: `input_number.moritz_outdoor_threshold` (summer 23 °C / winter 19 °C)
- **Occupancy**: presence-based with configurable comfort/eco bands

## Testing

Testing is scenario-based and manual:
- Summer day, winter night, absence, presence transitions
- Verify no short-cycling (minimum 10-minute gap between HVAC cycles)
- Compare HVAC runtime before and after under the same weather

See `tasks/task-list.md` for the current task breakdown and acceptance criteria.

## Contributing

See [CONTRIBUTING.md](CONTRIBUTING.md) for the development workflow, commit conventions, and Home Assistant–specific guidelines.

## License

MIT — see [LICENSE](LICENSE).
