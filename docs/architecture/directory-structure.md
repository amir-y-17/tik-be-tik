# Target Repository Structure

This is the intended direction for the repository. Create directories incrementally when they have a real purpose; the presence of a path in this document does not mean that its feature is implemented.

```text
tik-be-tik/
├── backend/
│   ├── config/
│   │   ├── settings/
│   │   │   ├── __init__.py
│   │   │   ├── base.py
│   │   │   ├── development.py
│   │   │   ├── production.py
│   │   │   └── test.py
│   │   ├── __init__.py
│   │   ├── urls.py
│   │   ├── asgi.py
│   │   └── wsgi.py
│   ├── apps/
│   │   ├── __init__.py
│   │   ├── identity/
│   │   ├── tasks/
│   │   └── common/
│   ├── tests/
│   │   └── integration/
│   ├── requirements/
│   │   ├── base.txt
│   │   ├── development.txt
│   │   └── production.txt
│   ├── .env.example
│   ├── manage.py
│   └── pyproject.toml
├── telegram-bot/                 # Future client; not implemented in Phase 1
├── mobile/                       # Future client; not implemented in Phase 1
├── desktop/                      # Future client; not implemented in Phase 1
├── nginx/
│   ├── nginx.conf
│   └── conf.d/
│       └── default.conf
├── deploy/
│   └── docker/                   # Deployment-specific assets, as needed
├── docs/
│   ├── architecture/
│   ├── api/
│   └── development/
├── scripts/                      # Helpful repeatable developer/ops commands
├── .github/
│   ├── workflows/                # Add workflows when CI is introduced
│   └── PULL_REQUEST_TEMPLATE.md
├── .dockerignore
├── .gitignore
├── compose.yaml
├── CHANGELOG.md
└── README.md
```

## Important placement decisions

- Keep the Django project package (`config/`) and `manage.py` inside `backend/` so the backend can be built and tested independently of client projects.
- Keep `compose.yaml` at the repository root so the whole stack can be started with one command.
- Keep Nginx configuration outside the Django source tree because it configures deployment infrastructure, not domain behavior.
- Keep `.env.example` in `backend/` if it documents backend-only variables. If Compose-level variables are shared by multiple services, keep a separate root `.env.example` and document both clearly. Do not maintain ambiguous duplicate files.
- Do not commit `.env`, real secrets, local database files, virtual environments, or generated static files.
- `requirements/base.txt` is the source of truth for runtime dependencies. `requirements/development.txt` and `requirements/production.txt` include it and add environment-specific tools. Use `pyproject.toml` for tool configuration (for example Ruff and pytest), not a second copy of runtime dependency declarations.
