# Workflow Gateway: Gateway-owned Database Specification (WG-004 re-baseline)

> **Status: proposed logical specification, prepared 27 November 2026, awaiting owner review.** It replaces the physical scope of the [original database specification](DATABASE_SPECIFICATION.md), which is kept as a historical, broader draft. This is a planning artefact: no schema, migration, database or hosted service exists or is authorised. Supabase and every other product remain candidates subject to [ADR-003](decisions/ADR-003-SUPABASE-DATABASE-AUTH.md) as clarified.

## 1. Purpose and boundary

Workflow Gateway owns the customer experience, the canonical conversation and message history, UI and session state, the external edge adapters and the client for the Konam API. Konam owns platform identity, workspaces, membership, roles, entitlements, registries, routing, execution, approvals, usage and platform audit ([Technical Architecture](TECHNICAL_ARCHITECTURE.md); [Konam integration specification](https://github.com/Konam-Automation-Platform/automation-business-control-centre/blob/main/docs/CROSS_REPOSITORY_INTEGRATION_SPECIFICATION.md)).

Rules that follow:

1. **No shared database.** Gateway and Konam communicate only through the versioned Konam API.
2. **No second control plane.** Gateway stores no users, memberships, roles, workspaces, registries, routing decisions, executions, approvals, credentials, entitlements or platform audit.
3. **References, not authority.** A workspace or user identifier stored by Gateway is an opaque reference issued by Konam. It is never proof of authority.
4. **Konam state is mirrored, not owned.** Where Gateway keeps a task state, it is a display cache of what Konam last returned. Gateway never marks a task successful, failed or approved.
5. **No secrets in ordinary tables.** No API keys, refresh tokens, passwords or reusable session tokens are stored in plain form.
6. **Data minimisation.** Keep only what the experience needs, with explicit retention and deletion.

## 2. Disposition of the original specification

| Original entity group | Disposition |
|---|---|
| conversations, messages, message_attachments | **Kept (Gateway-owned)**, re-keyed on Konam workspace and user references |
| data_export_jobs, deletion_requests, retention policy for conversation data | **Kept, narrowed** to conversation data (sections 4.7, 4.8) |
| idempotency_records | **Kept as a Gateway edge table** for Gateway operations only; Konam keeps its own for platform operations |
| users, organisations, memberships, invitations, workspace_settings | **Konam-owned.** Gateway keeps only opaque references |
| workflows, workflow_versions, workflow_capabilities, workflow_customer_allowlist | **Konam-owned** |
| models, model_capabilities, plugins, plugin_versions, tools, workspace_integrations, integration_tool_permissions | **Konam-owned** |
| credentials, credential_access_events | **Konam-owned.** Gateway holds no provider or integration credentials |
| routing_policies, routing_decisions | **Konam-owned** |
| executions, execution_events, execution_attempts, job_outbox, callback_receipts | **Konam-owned** |
| approval_requests, approval_decisions | **Konam-owned.** Gateway displays and forwards the user's decision |
| workspace_entitlements, usage_events, audit_logs | **Konam-owned** |

## 3. Identity and workspace references

- `user_ref`: the stable subject of the Konam-validated identity. It is a reference, not a user row. Gateway holds no password hashes and no profile data beyond display preferences.
- `workspace_ref`: the workspace resolved for the request from validated identity and verified by Konam. It is never taken from a request body, model output, tool output or callback.
- Gateway has no membership table. Every request is authenticated at the edge and the workspace context and permission are obtained from Konam as the integration specification requires. Edge checks are defence in depth only.
- Type and format of both references (UUID or string) follow the Konam API contract and are an open decision (OD-4).

## 4. Logical entities

All tables below carry `workspace_ref` (not null) unless stated. Types are logical; physical types are chosen when migrations are approved.

### 4.1 conversations

| Field | Notes |
|---|---|
| id | opaque UUID, primary key |
| workspace_ref | Konam-issued reference |
| created_by_user_ref | reference of the creating user |
| title | display text; bounded length |
| status | `active`, `archived`, `deletion_pending` |
| created_at, updated_at, archived_at | UTC |
| delete_after | nullable; set by the retention policy once decided |

Visibility inside a workspace is an open decision (OD-1). Recommended default: **private to the creator**, with no administrator read access except through an audited, Konam-defined process.

### 4.2 messages

| Field | Notes |
|---|---|
| id | opaque UUID |
| workspace_ref, conversation_id | composite foreign key to the conversation within the same workspace |
| sequence_no | unique per conversation |
| sender_type | `user`, `assistant`, `system_notice` |
| sender_user_ref | required when `sender_type` is `user`, null otherwise (check constraint) |
| content, content_format | `text` or `markdown`; bounded size (OD-8) |
| model_label | nullable display label returned by Konam; non-authoritative |
| redaction_state | `none`, `redacted`, `deleted` |
| created_at | UTC |

Assistant and system content is untrusted and is rendered safely. Gateway is the canonical store of conversation content. Konam keeps only the minimum task context it needs.

### 4.3 message_attachments (depends on OD-6)

Metadata only: id, workspace_ref, message_id, `file_ref` (opaque reference validated by Konam), display filename, media type, size, checksum, created_at. File bytes are never stored in this table. The storage vendor, scanning, size and type limits are open (OD-6).

### 4.4 konam_request_links

Links a user's message to the Konam request it created.

| Field | Notes |
|---|---|
| id | opaque UUID |
| workspace_ref, conversation_id, user_message_id | composite foreign keys |
| konam_request_id | issued by Konam; unique per workspace |
| correlation_id | propagated across Gateway, Konam and adapters |
| idempotency_key | the key used for submission |
| submitted_by_user_ref, submitted_at | |
| last_known_state | `accepted`, `queued`, `running`, `awaiting_approval`, `succeeded`, `failed`, `cancel_requested`, `cancelled`, `timed_out` |
| last_state_at, last_event_cursor | for polling |
| result_message_id | nullable; set when Konam returns a terminal success and its result is stored as an assistant message |

The state vocabulary matches the integration specification. Updates are monotonic: a terminal state is never replaced by an earlier one. This table is a display cache and is not evidence of execution.

### 4.5 idempotency_records (Gateway edge)

workspace_ref, operation_scope, idempotency_key, request_digest, response_ref, created_at, expires_at. Unique on (workspace_ref, operation_scope, idempotency_key). The same key with a different digest is a conflict.

### 4.6 ui_preferences, and gateway_sessions (conditional)

- `ui_preferences`: user_ref, optional workspace_ref, an allow-listed set of display settings (locale, theme, default response mode, preferred model label), updated_at. Preferences never override Konam policy, entitlement or approvals.
- `gateway_sessions`: only if OD-3 chooses server-side sessions. Fields: id, user_ref, selected workspace_ref, `session_token_hash` (never the token), created_at, expires_at, revoked_at, last_seen_at. If short-lived validated tokens are chosen instead, this table does not exist.

### 4.7 conversation_export_jobs

id, workspace_ref, requested_by_user_ref, scope, status, `object_ref`, expires_at, completed_at. Exports cover only the requester's authorised scope, exclude secrets and other users' private data, are delivered through a short-lived private reference, and creation and download are logged with the correlation ID.

### 4.8 conversation_deletion_requests

id, workspace_ref, scope (conversation or workspace), reason (`user_request`, `retention_expiry`, `konam_workspace_closure`), requested_by_user_ref, status, requested_at, completed_at, per-store evidence. Completion is recorded only when every applicable store has an evidenced result.

## 5. Relationships

- A conversation has many messages; a message has many attachments.
- A user message may have one konam_request_link; a successful link may reference one assistant result message.
- All relations are workspace-scoped through composite foreign keys, for example (workspace_ref, conversation_id) referencing conversations (workspace_ref, id). There are no polymorphic references in this scope, which removes the typed-reference concern raised in review item R3 for Gateway data.

## 6. Constraints and indexes

- Opaque UUID primary keys; sequential identifiers are never used as access tokens.
- Unique (conversation_id, sequence_no); unique (workspace_ref, konam_request_id); unique (workspace_ref, operation_scope, idempotency_key).
- Check constraints: `sender_type` against `sender_user_ref`; status enumerations; non-negative sizes; `expires_at` after creation.
- UTC timestamps; explicit enumerations rather than free-text statuses.
- Indexes: (workspace_ref, created_by_user_ref, updated_at) on conversations; (conversation_id, sequence_no) on messages; (workspace_ref, last_known_state) on konam_request_links for polling; expiry indexes for retention sweeps.

## 7. Tenant isolation

Application rules, for every request:

1. Authenticate at the edge and validate issuer, audience, expiry and signature.
2. Resolve the workspace reference from validated identity and Konam, never from caller-supplied fields.
3. Confirm with Konam that the principal may act in that workspace for the operation.
4. Check that every row read or written carries the same `workspace_ref` (and, if OD-1 keeps conversations private, the same `user_ref`).
5. Apply the same checks to workers, polling jobs, exports and deletion jobs.

Database defence in depth:

- Row-level security enabled on every table above, with policies keyed to a per-transaction setting for `workspace_ref` (and `user_ref` if conversations are private) that only the backend sets after validation.
- The application connects as a role that is not the table owner and cannot bypass row-level security. Migrations use a separate role, used only against disposable databases unless the owner approves otherwise.
- **No direct browser access to the database.** On Supabase, the Data API must not expose Gateway tables, and the anonymous and authenticated roles receive no grants.
- Service-level credentials never reach browser code.

### 7.1 Required negative tests

- A request in workspace A cannot read, update, delete or infer workspace B data, by any route including guessed identifiers.
- A message cannot be attached to a conversation in another workspace.
- A forged or altered workspace reference in a body, header or token claim is rejected.
- A private conversation is not readable by another member of the same workspace (if OD-1 keeps the default).
- A stale or lower-order Konam state never overwrites a terminal state.
- A replayed submission with the same idempotency key does not create a second Konam request; the same key with different content is rejected.
- A direct database connection with the application role cannot bypass row-level security, and browser-facing roles have no access.
- Logs contain no message content or credentials.

## 8. Reliability and integrity

- Submission to Konam is idempotent; a network retry cannot create a second consequential request.
- If Konam is unavailable, Gateway shows an explicit unavailable state and does not fall back to an ungoverned path.
- Duplicate and out-of-order event delivery is tolerated when mirroring Konam state.
- Conversation operations are audited at the edge through structured logs with correlation IDs. Gateway keeps no platform audit store; Konam owns authoritative audit.

## 9. Data classification and retention (durations undecided)

Durations are **not decided** and must be set by the owner before any pilot (OD-2). Indefinite retention is not assumed acceptable.

| Data | Examples | Proposed handling |
|---|---|---|
| Conversation content | messages and results | Gateway governs retention, with user archive and deletion; no unnecessary copies in logs |
| Attachment metadata | filenames, references | Deleted with the message; file objects follow OD-6 |
| Request links | Konam request IDs, cached state | Retained while the conversation exists; contain no content |
| Idempotency records | digests, references | Short, expiry-based |
| Sessions (if used) | token hashes | Expire; revoke on logout and on Konam revocation signals |
| Preferences | display settings | Retained while the account is active |
| Export artefacts | export files | Short-lived, private, expire automatically |
| Backups | snapshots | Defined expiry and documented deletion propagation |

Deletion must identify every copy: relational rows, object storage, caches, derived summaries, logs and backups, plus the coordinated deletion of Konam-held task context (OD-7). Provider-side deletion is not promised without evidence.

## 10. Migration, backup and recovery (future requirements)

No migration is created by this document. When implementation is separately approved:

- Use ordered, reviewed, immutable migration files; test them only against disposable databases in CI before any shared environment.
- Hosted migrations need explicit owner approval; separate schema change from backfill and destructive cleanup.
- Use synthetic seed data only.
- Backups are encrypted, access-controlled and restore-tested; recovery objectives are set before a pilot.
- After every migration, verify grants, row-level security, constraints and the negative tests in section 7.1.

## 11. Threat-model mapping

The controls here address cross-workspace access, forged workspace context, replay and duplicate submission, leakage of content into logs, direct database exposure, stale-state tampering and untrusted model content. The independent review is to map each to the control identifiers in the [Security Threat Model](SECURITY_THREAT_MODEL.md).

## 12. Open decisions for the owner

1. **OD-1** Conversation visibility within a workspace (recommended: creator-private).
2. **OD-2** Retention durations for conversations, attachment metadata, idempotency records, sessions and exports.
3. **OD-3** Session model: short-lived validated tokens or a server-side session table. Depends on the identity provider and token flow (Konam open decision 1).
4. **OD-4** Format of `workspace_ref` and `user_ref`, to match the Konam API contract.
5. **OD-5** Whether Konam also retains full result content, and the single source of truth for results.
6. **OD-6** Attachment storage vendor, scanning, size and type limits, and who stores the bytes.
7. **OD-7** Deletion and export coordination between Gateway and Konam, including workspace closure.
8. **OD-8** Maximum message size and the large-content strategy.

## 13. Dependencies

- The Gateway-to-Konam API contract (Konam repository) fixes reference formats, error codes and the state vocabulary.
- The identity provider and token flow decision.
- ADR-003 and ADR-010 as clarified; ADR-005 conditions apply to the backend that uses this schema.

## 14. Acceptance checklist for WG-004

- [x] Gateway-owned entities are defined and the Konam-owned entities are explicitly excluded.
- [x] Identity and workspace are treated as opaque Konam references.
- [x] Isolation, row-level security and negative tests are specified.
- [x] Retention, export and deletion are specified as proposals.
- [x] Migration and recovery requirements are stated without performing them.
- [ ] Owner decisions OD-1 to OD-8 (at least OD-1 to OD-4 before any schema work).
- [ ] Owner review and acceptance of this re-baseline.
- [ ] Independent cross-document consistency review and threat-model mapping.
- [ ] Alignment with the approved Gateway-to-Konam API contract.
