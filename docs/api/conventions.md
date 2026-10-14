# API Conventions

## Base path and versioning

Public API endpoints use the `/api/v1/` prefix. The version represents the public contract, not the application package version.

Initial endpoint examples:

```text
GET     /api/v1/tasks/
POST    /api/v1/tasks/
GET     /api/v1/tasks/{id}/
PATCH   /api/v1/tasks/{id}/
DELETE  /api/v1/tasks/{id}/
```

Operational health checks are not user-facing task resources. Their exact paths are defined with the deployment health-check configuration and may live outside `/api/v1/`.

## HTTP and data format

- Use HTTPS outside local development.
- Use JSON for request and response bodies unless an endpoint explicitly accepts a file upload.
- Use HTTP methods according to their normal semantics: GET reads, POST creates, PATCH partially updates, DELETE removes.
- Use trailing slashes consistently, matching Django/DRF conventions.
- Use UTC for timestamps in storage and return timezone-aware ISO 8601 timestamps.

## Authentication and authorization

- Authenticated API requests use `Authorization: Bearer <access-token>` once the token flow is implemented.
- Every user-owned query must be scoped to the authenticated user.
- Never trust a `user_id` supplied in request JSON to decide ownership.
- Use 401 for missing/invalid authentication and 403 for an authenticated caller who lacks permission. Object visibility policies may intentionally return 404 to avoid disclosing the existence of another user's object; apply that policy consistently.

## Validation and errors

Use DRF validation and conventional status codes. Return field-level validation errors for invalid input. Do not leak stack traces, secrets, database details, or internal exception messages in production responses.

Error response structure should be consistent. Start with DRF's standard error format unless the project has a concrete reason to introduce a custom error envelope.

## Compatibility policy

- Backwards-compatible additions may be released within `/api/v1/`.
- Removing or renaming fields, changing their meaning incompatibly, or changing required request properties requires a migration plan and a new API version when compatibility cannot be maintained.
- Document intentional contract changes and add API tests.
- Generate and review an OpenAPI schema once the initial endpoint set stabilizes.

## Minimum API tests

- Anonymous requests are rejected where authentication is required.
- A user can access their own tasks.
- A user cannot list, retrieve, modify, or delete another user's tasks.
- Invalid payloads return stable validation errors.
- Task creation and updates persist correctly.
- API behavior remains consistent regardless of whether the caller is a mobile app, desktop app, or Telegram integration.
