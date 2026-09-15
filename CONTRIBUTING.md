# Contributing to Smart Climate

Thank you for helping improve the Smart Climate HVAC control system. This project is built on Home Assistant, so the workflow is YAML-first: automations, helpers, and blueprints rather than application code.

## Getting Started

1. **Fork and branch** off `main` with a short-lived feature branch:
   ```bash
   git checkout -b feature/<short-description>
   ```
   Keep branches under 1–3 days. Prefer feature flags over long-lived branches.

2. **Run the test environment**:
   ```bash
   docker-compose up -d
   ```

3. **Validate before committing**:
   ```bash
   docker exec homeassistant ha config check
   ```

## Commit Conventions

Use the format `<type>: <short description>` and explain **why**, not just what.

- `feat` — New automation, helper, or dashboard
- `fix` — Corrects incorrect automation logic or entity reference
- `refactor` — Restructures YAML without changing behavior
- `test` — Adds or updates test scenarios
- `docs` — Specification or README changes
- `chore` — Docker config, dependencies, tooling

Examples:
```bash
feat: add occupancy-aware setpoint adjustment automation

Presence detection now triggers eco mode after a 30-minute
absence timeout, reducing HVAC runtime while maintaining
comfort when occupants return.
```

Each commit should do one logical thing. Keep changes under ~100 lines per commit/PR; split anything larger.

## Home Assistant–Specific Guidelines

- Use **native triggers and conditions** instead of Jinja2 logic where possible.
- Jinja2 is for data values only, not control flow.
- Reference entities by explicit `entity_id` — never `this.entity_id`.
- All thresholds live in `input_number` helpers, not hardcoded in YAML.
- Do not commit runtime state under `ha-config/.storage/`, `*.db`, `*.log`, or `*.lock`. These are ignored by `.gitignore`.
- Only commit source files: `blueprints/`, `specs/`, `tasks/`, `scripts/`, `tests/`, `resources/`.

## Testing

Before opening a PR, run the manual test scenarios covering your change:

- Summer day scenario — comfort maintained, no short-cycling
- Winter night scenario — eco mode active, energy savings
- Absence → presence transition — comfort mode resumes correctly
- Mode switching — summer ↔ winter ↔ intermediate works without conflicts

Verify with `docker exec homeassistant ha config check` and by inspecting entity history for rapid toggling.

## Pull Requests

1. Ensure `main` is up to date and rebase if needed.
2. Fill in the PR description with the problem, the fix, and how to verify it.
3. Reference the relevant task in `tasks/task-list.md`.
4. Include any new manual test results.

## Security

Never commit secrets, device credentials, or authentication tokens. The `.gitignore` excludes `.env`, `*.pem`, `*.key`, and `credentials/` — if you discover a secret has been committed, rotate it immediately and use `git filter-repo` to remove it from history.
