# ADR-002: Create a Custom User Model Before the First Migration

- **Status:** Accepted
- **Date:** 2026-10-14
- **Decision owners:** Tik be Tik project maintainers

## Context

The same Tik be Tik account must be usable across future clients. The application needs a stable internal identity independent of the login email and external identities such as Telegram accounts. Django's default user model makes changing the user model later more difficult, particularly after other migrations reference it.

## Decision

Create `identity.User` by extending Django's `AbstractUser`, remove the built-in `username` field, and use a unique email address as `USERNAME_FIELD`. Implement a custom user manager that creates regular and superuser accounts from email. Configure `AUTH_USER_MODEL = "identity.User"` before the first migration.

The internal database primary key is the canonical application identity. Email is a login credential/identifier, not a foreign-key substitute. External providers are linked to the internal user via a separate model when those integrations are introduced.

## Consequences

### Positive

- One account model shared by all clients
- Email-based login without maintaining a second username login identifier
- Room to link Telegram and other external identities without changing the canonical user key
- A deliberate user model avoids a painful late migration

### Negative / trade-offs

- A custom manager and compatible admin/forms configuration are required.
- The choice must be made and tested before initial migrations are committed.
- Email normalization and uniqueness require consistent validation at API and database layers.

## Implementation notes

- Use `settings.AUTH_USER_MODEL` for model relationships.
- Use `get_user_model()` in runtime code where a model class is needed.
- Never use an external provider ID as the primary key for `User`.
- Do not add provider-specific columns to `User`; use a future external-identity table with a uniqueness constraint on `(provider, provider_user_id)`.
- Add account linking only with proof of control of both accounts and a short-lived, single-use linking process.

## Review triggers

Revisit authentication policy before public release if product requirements change to support passwordless signup, multiple emails per account, an external identity provider, or organization-managed single sign-on.
