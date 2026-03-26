# AGENTS.md

Guidance for coding agents and contributors working in this repository.
Keep changes aligned with the repo's real tooling and current code patterns.

## Project Snapshot
- Project: `Anker-Solix-Api`
- Version in `pyproject.toml`: `3.4.0`
- Python requirement: `>=3.12`
- README documents Python `3.12` and `3.13`
- Workflow: uv, async Python, executable validation scripts
- Dependencies live in `project.dependencies`
- No build backend is configured; uv is used for environment and command execution
- This is an experimental reverse-engineered integration, not an official Anker API

## Repository Layout
- `api/`: core API, session, polling, MQTT, types, and device logic
- `common.py`: shared helpers and console logging for scripts
- `test_api.py`: executable API validation script
- `test_c1000x_mqtt_controls.py`: executable MQTT integration/control script
- `monitor.py`, `mqtt_monitor.py`, `export_system.py`, `energy_csv.py`: focused utilities
- `examples/`: anonymized sample exports for offline development/debugging
- `docs/`: device and integration documentation

## Environment Setup
Preferred setup:

```sh
uv sync
```

Refresh the lockfile when dependencies change:

```sh
uv lock
```

Run scripts through uv:

```sh
uv run python <script>.py
```

Examples:

```sh
uv run python ./test_api.py
uv run python ./monitor.py --site-id <SITE_ID>
```

## Build, Lint, and Validation Commands
There is no dedicated build step configured in this repo.
There is also no checked-in `Makefile`, `tox`, `nox`, or formal `pytest` workflow.

Use these commands as the current validation surface:

```sh
uv sync
pre-commit run --all-files
uv run isort .
uv run black .
uv run flake8 --format pylint .
uv run pylint --output-format parseable .
uv run prettier --check .
```

- `pre-commit run --all-files` is the best single repo-wide check.
- `prettier` is configured in pre-commit even though the repo is Python-heavy.
- Do not claim these commands are CI-enforced unless a workflow is added.

## Test and Verification Workflows
This repo relies more on executable scripts and live integration checks than on a formal unit-test suite.

Primary validation scripts:

```sh
uv run python ./test_api.py
uv run python ./test_c1000x_mqtt_controls.py
```

Focused verification commands:

```sh
uv run python ./monitor.py --site-id <SITE_ID>
uv run python ./monitor.py --device-id <DEVICE_ID>
uv run python ./mqtt_monitor.py --device-sn <DEVICE_SN>
uv run python ./export_system.py
uv run python ./energy_csv.py
```

Single-test guidance:
- There is no standard selector like `pytest path::test_name`.
- `test_api.py` narrows behavior through booleans such as `TESTAUTHENTICATE`, `TESTAPIMETHODS`, `TESTAPIENDPOINTS`, and `TESTAPIFROMJSON`.
- The closest thing to a single test is running one script, one focused utility command, or locally adjusting those toggles.
- Some checks require real credentials, network access, cloud access, or real devices.
- Prefer example-based validation when touching parsing, cache, or transformation logic that can run against `examples/` data.

## Formatting and Linting Rules
- Use Black formatting.
- Use isort with `profile = "black"`.
- Flake8 ignores: `E501`, `E226`, `W503`, `E231`.
- Pylint disables include: `import-error`, `line-too-long`, `invalid-name`, `protected-access`.
- Pre-commit also enforces AST checks, mixed-line-ending checks, and trailing-whitespace cleanup.
- Let formatter/linter behavior drive formatting decisions.
- Do not hand-format imports in ways that fight `isort` or `black`.

## Imports
Follow the import grouping already used in core modules:
1. Standard library imports
2. Third-party imports
3. Local package imports

Use a blank line between groups.

Additional repo patterns:
- Prefer `from __future__ import annotations` in new core modules when consistent with nearby files.
- Use `TYPE_CHECKING` for type-only imports when it avoids runtime cycles or unnecessary imports.
- Preserve existing relative-import style inside `api/`.

## Types
- Add explicit type hints for public functions, async methods, and important attributes.
- Use Python 3.12 style annotations and `|` unions.
- Match the surrounding file's typing style instead of introducing a new local style.
- Use precise types where they are cheap and stable.
- Use `Any` or broad `dict` shapes only when payloads are genuinely dynamic or reverse-engineered.
- Add new enums, dataclasses, and shared typed constants to central type modules instead of scattering literals.

## Naming
Default naming for new internal code:
- functions and variables: `snake_case`
- classes: `PascalCase`
- module loggers: `_LOGGER`
- user-facing script logger: `CONSOLE`

Compatibility matters here:
- Do not rename existing public API fields or parameters just to normalize style.
- Preserve established camelCase names such as `countryId`, `siteId`, `deviceSn`, `fromFile`, `testDir`, `logLevel`, and `endpointLimit` when touching compatibility-facing APIs.
- Avoid broad naming cleanups in request/session/device models unless the change is deliberate and verified.

## Error Handling and Resilience
- Reuse existing exception classes and `raise_error()` from `api/errors.py` for API response handling.
- Do not create parallel exception hierarchies for the same server codes.
- Follow existing retry, throttling, and session patterns in `api/session.py` for transient failures.
- Preserve retry-aware handling for auth expiry, rate limits, connection errors, and temporary server failures.
- In payload normalization and cache-update paths, logging and continuing can be preferable to aborting a whole refresh when one field is malformed.
- If you catch broadly at a boundary, log enough context to debug the issue.

## Logging and Async Patterns
- Prefer logger calls over bare prints in library code.
- Use `%s` formatting in reusable library logging paths.
- Keep credential-bearing values masked or out of logs.
- Preserve the existing `aiohttp.ClientSession` + async workflow.
- Do not introduce sync wrappers around core network behavior without a strong reason.
- Reuse existing session, poller, and helper utilities before adding new request logic.

## Documentation and Comments
- Preserve module, class, and function docstrings; they are common across the codebase.
- Add docstrings to new public modules and functions.
- Add comments when behavior is non-obvious, reverse-engineered, compatibility-driven, or protocol-specific.
- Do not add comments that merely restate obvious code.
- If behavior changes, update `README.md` or relevant files under `docs/`.

## Working With Integration Scripts and Example Data
- Treat `test_*.py` files here as executable validation/integration scripts unless a formal test runner is added.
- Prefer validating parsing and cache behavior against `examples/` exports when possible.
- Be careful with scripts that talk to live services, trigger MQTT activity, or modify device settings.
- Keep device-specific docs such as `docs/C1000X_Integration_Guide.md` aligned with implementation changes.

## Local Data and Commit Safety
Do not commit ignored files or local secrets.

Important ignored paths and patterns include:
- `.env`
- `client.py`
- `**/authcache`
- `**/exports`
- `**/mqttdumps`
- `daily_energy_*.csv`

The README also expects local credential usage and personal scratch tooling to remain local.

## Cursor and Copilot Rules
No repo-local agent rule files were found when this file was written:
- no `.cursor/rules/`
- no `.cursorrules`
- no `.github/copilot-instructions.md`

If any of those files are added later, update this document.

## Agent Summary
- Use uv commands and the existing async script workflow.
- Verify with `pre-commit run --all-files` and the narrowest realistic script-based check.
- Do not describe the repo as having a formal pytest suite when it does not.
- Preserve compatibility-driven naming and request/session patterns.
- Reuse central types, errors, and helpers instead of scattering new conventions.
