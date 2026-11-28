# Gateway-to-Konam API: Consumer Contract Proposal

> **Status: proposal for Konam to adopt or amend, prepared 27 November 2026.** Konam owns the internal platform API and its schema. This document and the [OpenAPI draft](../api/konam/konam-api-v1.consumer-proposal.yaml) state what Workflow Gateway needs, so that the contract can be agreed in the Konam repository. Nothing here authorises implementation, exposes an endpoint, selects a vendor or creates a Konam deliverable. Paths, schemas and limits are provisional.

## 1. Why this exists

The [Konam integration specification](https://github.com/Konam-Automation-Platform/automation-business-control-centre/blob/main/docs/CROSS_REPOSITORY_INTEGRATION_SPECIFICATION.md) sequences the work as: confirm the boundary, then define and review the machine-readable internal API schema, then identity, then a minimal submit and status path. The Konam backlog has no issue for the schema step, and it blocks both the Gateway client and the Gateway database re-baseline (reference formats, state vocabulary, error codes). This proposal is a Gateway-side starting point for that step. It is consumer-driven: it asks only for what Gateway must have to deliver the chat experience without owning platform logic.

## 2. Rules taken from the integration specification

1. Konam derives the principal, workspace and scopes from validated identity and membership. A workspace identifier in a header or body is a **selector only**.
2. Long-running work is **asynchronous**. An acknowledgement means accepted, not succeeded.
3. Konam **owns state transitions**. Gateway displays them and never marks a task successful, failed or approved.
4. Files are passed as **opaque references**, never URLs or local paths.
5. Submission is **idempotent**; errors use a **stable envelope** with a retryability flag and correlation ID.
6. If Konam is unavailable, Gateway shows an explicit unavailable state and never falls back to an ungoverned path.
7. Gateway holds no provider or workflow-engine credentials and has no database access to Konam.
8. Events start with **authenticated polling**; SSE only if justified; webhooks deferred.

## 3. Operations

| Operation | Proposed route | Notes |
|---|---|---|
| Submit request | `POST /v1/requests` | Returns 202 with request ID, correlation ID, initial state, status reference and creation time |
| Retrieve request | `GET /v1/requests/{requestId}` | Authoritative state; result only when `succeeded`; error only when `failed` or `timed_out`; pending approval when `awaiting_approval` |
| Retrieve events | `GET /v1/requests/{requestId}/events?after=&limit=` | Cursor-based polling; ordered by sequence within a request |
| Cancel request | `POST /v1/requests/{requestId}/cancel` | A request, not a guarantee; a terminal request returns its current state |
| Submit approval decision | `POST /v1/requests/{requestId}/approvals/{approvalId}/decision` | Gateway forwards the user's `approve` or `reject`; Konam checks authority and that the approval is still pending |
| List capabilities | `GET /v1/capabilities` | Permission-filtered summaries for display only; listing confers no authority |
| Compatibility declaration | `GET /v1/compatibility` | Minimum and maximum supported Gateway client versions, as the integration specification requires |

## 4. Conventions

- **Identifiers:** every reference (`request_id`, `correlation_id`, `conversation_ref`, `file_ref`, `approval_id`, `capability_ref`) is opaque text of at most 128 characters using letters, digits and `. _ : -`, compared by exact match and never parsed. This matches the Gateway database decision on opaque references.
- **Headers:** `Authorization: Bearer` (short-lived, audience-restricted delegated user token); `X-Workspace-Ref` (selector); `Idempotency-Key` (required on every write); `X-Correlation-Id` (optional inbound; Konam generates one if absent and always returns it); `Gateway-Client-Version`; response `Konam-API-Version`. The Gateway service identity is carried separately and stays distinguishable in audit records. The mechanism depends on the identity-provider decision.
- **Versioning:** a `/v1` path, additive changes within a major version, a new major version or a documented window for breaking changes, and generated contract tests in CI once implementation is authorised.
- **Request body:** goal, optional constraints, conversation reference, only the context needed, attachment references, and preferences (response mode, locale, a model preference as a label). Preferences never override policy, entitlement, availability, safety or approval gates. The body has no principal, workspace, role or entitlement fields.
- **States:** `accepted`, `queued`, `running`, `awaiting_approval`, `succeeded`, `failed`, `cancel_requested`, `cancelled`, `timed_out`.
- **Error envelope:** `error.code`, a safe `message`, `retryable` and `correlation_id`. Proposed codes: `validation_error`, `authentication_error`, `authorization_error`, `not_found`, `quota_exceeded`, `conflict`, `idempotency_conflict`, `timeout`, `capability_unavailable`, `approval_required`, `service_unavailable`, `internal_error`.

Proposed HTTP mapping: 400 validation, 401 authentication, 403 authorisation, 404 not found (also used for another workspace's resource, so existence is not disclosed), 409 conflict, idempotency conflict and approval required, 422 capability unavailable, 429 quota and rate limits, 500 internal, 503 unavailable (with `Retry-After`), 504 timeout.

## 5. Behaviour Gateway will implement against this contract

- Send a fresh idempotency key per user action and reuse it on network retries; treat the same key with different content as a conflict.
- Poll while a request is active, starting around one second and backing off to a ceiling of a few seconds, slower while `awaiting_approval`, stopping at a terminal state and honouring `Retry-After`. All intervals are placeholders pending Konam's event-semantics decision.
- Tolerate duplicate and out-of-order events. Apply a state only if it does not regress a terminal state.
- Store a successful result as an assistant message only when Konam returns `succeeded` with a result.
- Show an explicit unavailable state on `service_unavailable` or timeout. Never retry through any other path.
- Treat all result content as untrusted and render it safely.
- Log the correlation ID with every edge operation, without message content or credentials.

## 6. Mapping to the Gateway-owned data model

The request-link record in the [Gateway-owned database specification](GATEWAY_DATABASE_SPECIFICATION.md) holds `konam_request_id`, `correlation_id`, the submission idempotency key, the last known state and its time, the last event cursor, and an optional result message. That table is a display cache of what Konam last returned and is never evidence of execution.

## 7. Planned use inside Gateway (after separate authorisation)

A Konam client interface with a **stub** that implements this contract, consumer-driven contract tests against the OpenAPI, and failure tests (unavailable, timeout, duplicate, out of order, cross-workspace).

## 8. Questions for Konam

1. Adopt, amend or replace this draft, and where it will live (Konam repository, as the integration specification requires).
2. Identity: the provider, claims, scopes, revocation, and how the Gateway service identity is conveyed.
3. Workspace resolution when a user belongs to several workspaces: is a selector header acceptable?
4. Reference formats and whether `correlation_id` is supplied or generated (this draft allows both).
5. Event semantics: polling cadence and limits, ordering guarantees, retention of events, and any future SSE criteria.
6. Maximum sizes for goal, context and result (the Gateway-owned database specification calls this OD-8).
7. File references: who stores the bytes, and scanning and lifecycle (OD-6).
8. Deletion and export coordination, including workspace closure (OD-7), and whether Konam retains full results (OD-5).
9. Timeout budgets and retry ceilings for the pilot objectives.

## 9. Acceptance and non-authorisation

- [x] The six operations in the integration specification are covered, plus the compatibility declaration it requires.
- [x] The draft validates as OpenAPI 3.1.
- [x] Nothing in the draft gives Gateway authority over identity, routing, approval or execution.
- [ ] Konam owner review and adoption in the Konam repository.
- [ ] Identity-provider decision reflected in the security scheme.
- [ ] Event semantics and limits agreed.

This proposal does not authorise application code, API exposure, a Konam deliverable, vendor selection, paid services, live customer data or production deployment.
