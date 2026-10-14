# Development Setup Principles

This document records setup decisions before the backend is scaffolded. Replace placeholders with tested commands as implementation progresses.

## Runtime baseline

As of 2026-10-14, the suggested baseline for this project is:

- Python 3.13
- Django 6.1 LTS series, using the latest available security patch
- Django REST Framework 3.18 series
- PostgreSQL for the containerized application environment

Django 6.1 is selected for its LTS support window rather than choosing the newest feature series solely because it is newer. Re-check supported versions and security patches when starting a new deployment or performing an upgrade.

Suggested dependency constraints, once the existing files have been reviewed:

```text
Django>=6.1,<6.2
djangorestframework>=3.18,<3.19
```

These are version constraints, not a fully locked dependency set. `requirements/base.txt` is the source of truth for runtime dependencies; `development.txt` and `production.txt` include it and add environment-specific packages. Use `pyproject.toml` for tool configuration, not a second copy of runtime dependencies. We will review your existing files before editing them.

## Environment and secrets

- `.env.example` contains names and safe placeholder values only.
- `.env` contains local values and must remain untracked.
- Production secrets must be injected by the deployment environment; do not rely on committed defaults.
- A development-only `SECRET_KEY` fallback must never be used when production settings are active.

## Local workflow

1. Create and activate a local virtual environment.
2. Install the declared dependencies.
3. Run Django system checks.
4. Create the custom User model and configure `AUTH_USER_MODEL`.
5. Only then generate and run initial migrations.
6. Add tests as endpoints are implemented.
7. Run tests and checks before committing.

## Git conventions

Use Conventional Commit prefixes consistently:

- `docs:` documentation-only changes
- `feat:` user-facing behavior
- `fix:` bug fixes
- `test:` tests
- `build:` dependencies, build scripts, Docker, and packaging
- `refactor:` behavior-preserving restructuring

Examples:

```text
docs: document initial architecture and identity strategy
build: scaffold Django backend
feat: add custom email-based user model
test: cover task ownership permissions
```

Do not create a release tag until the tagged commit has been tested and the working tree is clean.
