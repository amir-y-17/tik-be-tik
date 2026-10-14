# Identity and Authentication Strategy

**Status:** Accepted for the initial implementation; details may evolve through a new ADR.  
**Last reviewed:** 2026-10-14

## 1. Decision summary

Tik be Tik will have one canonical internal user identity shared by every client.

- Create a custom Django user model in the `identity` app before the first database migration.
- Use email address and password as the initial account login credentials.
- Normalize email addresses consistently and enforce uniqueness in the database.
- Use the internal user primary key for database relationships; do not use email addresses, usernames, or external platform IDs as foreign keys.
- Use Django session authentication for Django Admin and a token-based API authentication scheme for first-party API clients. The concrete token library and settings must be pinned and tested before the authentication endpoints are released.
- Treat Telegram as an external identity to be linked to a Tik be Tik account, not as the canonical internal user ID.

## 2. Custom user model

The initial model should extend `AbstractUser`, remove the built-in `username` field, and use a unique email as `USERNAME_FIELD`. A custom user manager is required because creation and superuser creation must accept email instead of username.

The model must be configured with:

```python
AUTH_USER_MODEL = "identity.User"
```

The Django app's Python path may be `apps.identity`, while its app label remains `identity`. The exact `AppConfig` and imports must be verified when scaffolding the backend.

**Critical:** configure the custom model before running the first `migrate` command or generating committed migrations that depend on Django's default user model. Changing the user model after data and migrations exist is possible but substantially harder.

Keep the initial user model small. Add profile fields only when the product needs them. Do not store Telegram-specific fields directly on `User`, because a user may eventually link more than one external identity.

## 3. Account lifecycle

Initial policy:

1. A user registers using email and password.
2. The password is handled by Django's password hashing mechanisms; it is never stored or logged in plain text.
3. Email is normalized consistently and must be unique.
4. Email verification and password recovery are explicit follow-up features; do not pretend an unverified email is verified.
5. Authorization is enforced by the backend for every operation on user-owned data.

Email login and account recovery should be covered by rate limits and security tests before public deployment.

## 4. Telegram linking in a future phase

When the Telegram bot is introduced, add a separate external-identity model rather than adding `telegram_id` as the primary identity field on `User`.

Conceptual fields:

- `user`: foreign key to the canonical internal user
- `provider`: e.g. `telegram`
- `provider_user_id`: external account ID stored as a string
- `provider_username`: optional, mutable display metadata
- `created_at` and `updated_at`

Enforce a database uniqueness constraint on `(provider, provider_user_id)`. A provider username is not a stable identifier and must not be used as the linking key.

Account linking must require proof of control of both sides. A recommended flow is to generate a short-lived, single-use linking code inside an authenticated Tik be Tik session, then redeem it through a verified Telegram update. Do not link accounts merely because a client submits a Telegram ID. Verify Telegram webhook/update authenticity and keep bot tokens only in server-side secrets.

The bot must not trust a user ID supplied by an untrusted client as authorization. The backend must resolve the authenticated or verified external identity to the canonical `User` and apply normal permissions.

## 5. API authentication

- Django Admin uses its normal session-based login.
- Public API clients use bearer access tokens with short lifetimes and a documented refresh/revocation policy.
- Keep access-token lifetime short; protect refresh credentials in platform-appropriate secure storage.
- Never store long-lived secrets in a mobile/desktop app binary.
- The Telegram bot backend should use verified Telegram identities and a narrowly scoped server-side integration path; it must not store a user's long-lived refresh token in the bot.
- Require HTTPS outside local development.

A token proves identity; it does not automatically grant access to every object. Object-level ownership and permissions remain mandatory.

## 6. Django coding rules

Use `settings.AUTH_USER_MODEL` for model relationships, for example:

```python
from django.conf import settings
from django.db import models

class Example(models.Model):
    owner = models.ForeignKey(
        settings.AUTH_USER_MODEL,
        on_delete=models.CASCADE,
        related_name="examples",
    )
```

Use `get_user_model()` in runtime code when the active user model class is required. Avoid importing `django.contrib.auth.models.User` directly.

## 7. Acceptance criteria

- [ ] `AUTH_USER_MODEL` points to the custom model before the first migration.
- [ ] Superuser creation works with email-based identification.
- [ ] Duplicate normalized email addresses are rejected.
- [ ] Passwords are never stored in plain text.
- [ ] A user cannot read or modify another user's tasks by guessing an object ID.
- [ ] No Telegram-specific field is used as the canonical user key.
- [ ] Account linking requires a short-lived, single-use proof and verified Telegram updates when implemented.
