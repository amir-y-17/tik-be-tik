# ADR-001: Use a Modular Monolith for the Backend

- **Status:** Accepted
- **Date:** 2026-10-14
- **Decision owners:** Tik be Tik project maintainers

## Context

Tik be Tik will expose one backend to multiple client types, including a Telegram bot, mobile app, and desktop app. The project is in its initial stage and has a small implementation scope. Introducing several independently deployed services would create avoidable operational work around service discovery, networking, distributed failures, deployment, and data consistency.

## Decision

Build the initial backend as a Django Modular Monolith. Deploy it as one application, initially using PostgreSQL, Gunicorn, Nginx, and Docker Compose. Separate domain responsibilities into Django apps with explicit ownership and dependency rules.

## Consequences

### Positive

- One API and one canonical source of task/user data
- Simpler local development, testing, and deployment
- Easier transactional consistency within PostgreSQL
- Clear domain boundaries create a path for later extraction if evidence requires it

### Negative / trade-offs

- Modules share a process and deployment lifecycle.
- Django apps can become tightly coupled if boundaries are not enforced.
- Scaling one domain independently is not possible without later architectural work.

## Rejected alternatives

- **Microservices from day one:** rejected because there is no demonstrated need for independent deployment or scaling and operational cost would be disproportionate.
- **A single unstructured Django app:** rejected because domain responsibilities would become harder to test and evolve as clients/features are added.

## Review triggers

Revisit this decision if independent deployment, materially different scaling profiles, isolated reliability boundaries, or separate team ownership become real requirements. Do not split services based on file count alone.
