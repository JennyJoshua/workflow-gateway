# ADR-004: FastAPI as the Backend Framework Candidate

- **Status:** Proposed — not owner-approved
- **Date:** 2026-11-26
- **Deciders:** Owner (final approval); ChatGPT (architecture review)
- **Related issues:** WG-002, WG-005, WG-009, WG-010, WG-015

## Context

The platform needs a backend API for authentication integration, tenant authorisation, orchestration, workflow adapters, asynchronous job admission, audit events and model provider integration. Python is a practical fit for the AI ecosystem, while the framework itself should not dictate hosting or database choices.

## Options considered

| Option | Advantages | Disadvantages | Cost/free-tier implications | Lock-in and exit considerations |
|---|---|---|---|---|
| FastAPI | Python ASGI framework, typed request/response models, OpenAPI support and async support | Requires disciplined architecture for background jobs, dependency management and security | Framework has no hosting fee; compute and managed services remain separate costs | Low lock-in; standard Python and HTTP interfaces |
| Django + Django REST Framework | Mature batteries-included framework, ORM and admin | More framework surface than needed for a focused API/orchestration service | Framework is free; hosting costs remain | Low-to-moderate framework coupling; portable relational data |
| Node.js API framework | One language across Next.js and API; broad ecosystem | AI/data integrations may be less convenient for Python-first orchestration | Frameworks are generally free; hosting costs remain | Low-to-moderate; depends on chosen framework and libraries |

## Proposed decision

Retain FastAPI as the **preferred backend framework candidate**. Keep business rules in explicit service modules, use typed request/response contracts, and do not place long-running jobs inside ordinary HTTP request lifetimes.

## Security, privacy and data residency

- Authentication and authorisation must be verified server-side.
- Validate every input and enforce workspace/tenant scope at service boundaries.
- Keep provider keys server-side and out of logs.
- Apply request size limits, rate limits, timeouts and outbound network restrictions.
- Use a separate worker process for durable background execution.

## Consequences

### Positive
- Strong fit for Python AI and integration libraries.
- Standard HTTP/OpenAPI contracts support external workflow clients.
- Hosting can be changed independently of the framework.

### Negative / trade-offs
- Requires deliberate choices for ORM, migrations, job queues, sessions and background workers.
- Async syntax alone does not provide durable job execution or recovery.

### Operational implications
- Pin dependencies and commit lockfiles.
- Select a test strategy and application structure in WG-005.
- Keep framework-specific objects out of domain contracts where practical.

## Validation plan

1. Build a small synthetic API proof of concept only after implementation is separately approved.
2. Test authentication middleware, request validation, OpenAPI generation and async provider calls.
3. Test worker separation and graceful shutdown.
4. Compare FastAPI with Django REST Framework against the actual initial feature set.

## References

- https://fastapi.tiangolo.com/deployment/
- https://fastapi.tiangolo.com/deployment/concepts/
- https://fastapi.tiangolo.com/deployment/manually/

## Approval record

- **ChatGPT review:** Proposed — initial review 2026-11-26
- **Owner decision:** Pending
