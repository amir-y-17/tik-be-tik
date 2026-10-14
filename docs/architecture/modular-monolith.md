# Modular Monolith Boundaries

## Purpose

A Modular Monolith is one deployable application composed of modules that have clear responsibilities. It is not a collection of folders with unrestricted dependencies.

## Initial domain modules

### `identity`

Owns:
- The canonical internal user model
- User-related account rules
- User registration and authentication flows when implemented
- Future external-identity linking, such as linking a Telegram account to an existing Tik be Tik account

Does not own:
- Task lifecycle rules
- Telegram message rendering or UI-specific logic

### `tasks`

Owns:
- Task model and task lifecycle rules
- Task creation, updates, completion, and deletion rules
- Queries for tasks visible to the authenticated user
- Task API serializers, views, URLs, and tests

Does not own:
- Password management
- Telegram IDs or bot command parsing

### `common`

Contains only genuinely cross-cutting primitives that have more than one legitimate consumer, such as a carefully justified abstract base model or shared exception types. It must not become a miscellaneous dumping ground.

### Future modules

Create a module only when the product has a real responsibility for it. Potential future modules include `notifications`, `integrations`, and `audit`. Do not create empty modules in Phase 1 just to make the directory tree look scalable.

## Suggested structure for a domain module

```text
apps/tasks/
├── __init__.py
├── apps.py
├── models.py
├── services.py
├── selectors.py
├── api/
│   ├── __init__.py
│   ├── serializers.py
│   ├── views.py
│   └── urls.py
└── tests/
    ├── __init__.py
    ├── test_models.py
    ├── test_services.py
    └── test_api.py
```

Start with only the files that are needed. `services.py` is for business operations that benefit from a clear service boundary; `selectors.py` is for reusable or non-trivial read queries. Do not create layers containing only pass-through wrappers without a clear benefit.

## Dependency rules

1. API views translate HTTP requests into application operations; they should not accumulate complex business rules.
2. Business rules must not import Nginx, Docker, Telegram SDKs, or UI code.
3. A module should not modify another module's data by bypassing its rules.
4. Django relations to the user model use `settings.AUTH_USER_MODEL`; runtime code uses `get_user_model()` when it needs the active model class.
5. Avoid circular imports. If two modules need each other, reconsider ownership or define a narrow interface.
6. Cross-module access to another module's models may be necessary in Django, but it must not become a license to bypass that module's services and invariants.

## When to consider extraction

Do not split a module into a separate service merely because it has many files. Consider extraction only when there is evidence such as independent scaling needs, distinct operational requirements, a separate release cadence, or a strong team-ownership boundary. A future service extraction must define network contracts, failure behavior, data ownership, and migration strategy first.
