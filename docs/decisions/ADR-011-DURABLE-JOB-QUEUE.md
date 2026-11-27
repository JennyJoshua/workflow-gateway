# ADR-011: Durable Job Admission and Queue Architecture

- **Status:** Proposed — not owner-approved
- **Date:** 2026-11-26
- **Deciders:** Owner (final approval); ChatGPT (architecture review)
- **Related issues:** WG-002, WG-004, WG-015, WG-016, WG-017, WG-022

## Context

Some workflows and external integrations take longer than a normal HTTP request. The platform must accept work durably, return a tracking identifier, execute in a worker, survive restarts and prevent duplicate side effects. An in-memory task or a web request held open is not sufficient.

## Options considered

| Option | Advantages | Disadvantages | Cost/free-tier implications | Lock-in and exit considerations |
|---|---|---|---|---|
| PostgreSQL-backed job table plus transactional outbox and worker | Reuses relational storage; durable state; transactional admission; low additional infrastructure | Polling and lock contention need design; database should not become an unbounded high-volume queue | Can avoid a separate queue service at low volume; database capacity and worker hosting still have costs | Low lock-in; portable SQL and worker contract |
| Redis-compatible queue | Mature queue patterns, high throughput and scheduling ecosystem | Adds another stateful service and operational dependency; persistence/eviction configuration matters | Managed Redis/Key Value may have free limits but production reliability may require paid service | Moderate; queue library and Redis semantics create migration effort |
| Managed message queue | Strong managed delivery and scaling characteristics | Provider-specific APIs, IAM and operational model | Free allowances vary; sustained usage may incur cost | Moderate-to-high service coupling unless hidden behind an adapter |

## Proposed decision

Use a **database-backed durable job admission model as the initial candidate**, with a separate worker process and a transactional outbox. Keep the queue transport behind an adapter so a dedicated broker can be introduced if measured throughput or scheduling needs justify it.

Minimum invariants:
1. Persist the execution request and idempotency key before acknowledging acceptance.
2. Dispatch only after the database transaction commits.
3. Claim jobs safely when multiple workers run.
4. Persist state transitions and execution events.
5. Use bounded retries, timeouts and explicit terminal states.
6. Ensure external side effects use idempotency or reconciliation where supported.
7. Authenticate callbacks and reject duplicates or stale state transitions.
8. Provide cancellation and recovery semantics.

PostgreSQL row locking with `SKIP LOCKED` is one possible multi-consumer claim mechanism, but it must be load-tested and used only for queue-like access patterns.

## Security, privacy and data residency

- Queue payloads contain references and minimum required metadata, not plaintext credentials.
- Enforce tenant/workspace scope when workers load execution context.
- Authenticate worker identities and callbacks.
- Treat job payloads and tool results as untrusted.
- Audit retries, approvals, cancellations and terminal outcomes.
- Apply retention and deletion rules to job payloads and event history.

## Consequences

### Positive
- Durable acceptance and tracking can be built on the primary relational store.
- Avoids adding Redis or a managed broker before demand justifies it.
- Uses standard database primitives and a portable worker contract.

### Negative / trade-offs
- Database polling adds load and needs careful indexing and claim logic.
- High-volume workloads may outgrow the initial queue model.
- Exactly-once execution cannot be guaranteed across arbitrary external systems; idempotency and reconciliation are required.

### Operational implications
- Separate API process from worker process.
- Define worker concurrency, backoff, dead-letter handling, queue age monitoring and recovery.
- Do not use ephemeral web-service filesystems for job state.

## Validation plan

1. Model duplicate submission, worker crash, timeout, retry and callback replay.
2. Test concurrent job claiming with synthetic jobs.
3. Test restart recovery and tenant isolation.
4. Load-test queue polling before adopting it for production.
5. Compare a dedicated broker only if measured requirements justify it.

## References

- https://www.postgresql.org/docs/current/sql-select.html
- https://render.com/docs/free

## Approval record

- **ChatGPT review:** Proposed — initial review 2026-11-26
- **Owner decision:** Pending
