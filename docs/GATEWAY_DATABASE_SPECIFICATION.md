# Workflow Gateway: Gateway-owned Database Specification (WG-004 re-baseline)

> **Status: proposed logical specification, prepared 27 November 2026, awaiting owner review. OD-1, OD-3 and OD-4 are decided (section 12); the OD-2 retention defaults await owner confirmation.** It replaces the physical scope of the [original database specification](DATABASE_SPECIFICATION.md), which is kept as a historical, broader draft. This is a planning artefact: no schema, migration, database or hosted service exists or is authorised. Supabase and every other product remain candidates subject to [ADR-003](decisions/ADR-003-SUPABASE-DATABASE-AUTH.md) as clarified.

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
- Both references are stored as **opaque text, never parsed** (decision OD-4, section 12): length at most 128, a restricted character set, compared by exact match, with no meaning attached. Konam should issue non-guessable identifiers, but Gateway does not depend on their format.

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

**Visibility (decision OD-1, owner, 27 November 2026): conversations are private to their creator.** No workspace member, including an administrator, can read another member's conversation. Any compliance or legal-hold access must be a Konam-defined, audited process specified with the API contract.

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

### 4.6 ui_preferences (and the session decision)

- `ui_preferences`: user_ref, optional workspace_ref, an allow-listed set of display settings (locale, theme, default response mode, preferred model label), updated_at. Preferences never override Konam policy, entitlement or approvals.
- `gateway_sessions`: **not part of v1** (decision OD-3, section 12). Gateway relies on short-lived tokens validated at the edge and re-validated by Konam on every operation, so there is no session table. Adding one later needs a specification amendment and owner approval.

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
4. Check that every row read or written carries the same `workspace_ref` and, for conversations and everything under them, the same `user_ref` (conversations are private).
5. Apply the same checks to workers, polling jobs, exports and deletion jobs.

Database defence in depth:

- Row-level security enabled on every table above, with policies keyed to per-transaction settings for `workspace_ref` and `user_ref` that only the backend sets after validation.
- The application connects as a role that is not the table owner and cannot bypass row-level security. Migrations use a separate role, used only against disposable databases unless the owner approves otherwise.
- **No direct browser access to the database.** On Supabase, the Data API must not expose Gateway tables, and the anonymous and authenticated roles receive no grants.
- Service-level credentials never reach browser code.

### 7.1 Required negative tests

- A request in workspace A cannot read, update, delete or infer workspace B data, by any route including guessed identifiers.
- A message cannot be attached to a conversation in another workspace.
- A forged or altered workspace reference in a body, header or token claim is rejected.
- A private conversation is not readable, listable or exportable by another member of the same workspace, including an administrator.
- A stale or lower-order Konam state never overwrites a terminal state.
- A replayed submission with the same idempotency key does not create a second Konam request; the same key with different content is rejected.
- A direct database connection with the application role cannot bypass row-level security, and browser-facing roles have no access.
- Logs contain no message content or credentials.

## 8. Reliability and integrity

- Submission to Konam is idempotent; a network retry cannot create a second consequential request.
- If Konam is unavailable, Gateway shows an explicit unavailable state and does not fall back to an ungoverned path.
- Duplicate and out-of-order event delivery is tolerated when mirroring Konam state.
- Conversation operations are audited at the edge through structured logs with correlation IDs. Gateway keeps no platform audit store; Konam owns authoritative audit.

## 9. Data classification and retention (proposed defaults, awaiting owner approval)

The durations below are **proposals for the owner to approve, change or reject (OD-2)**. They are not decided, are not legal advice, and no jurisdiction or customer obligation has been confirmed. Organisation workspaces may need different periods, to be set through Konam policy (OD-7). Indefinite retention is not assumed acceptable.

| Data | Examples | Proposed default | Notes |
|---|---|---|---|
| Conversation content | messages and results | **12 months after last activity**, then `deletion_pending` with 30 days' notice and an export offer. Users can delete or export at any time; a deletion request completes within 30 days | Balances usefulness against data minimisation; easy to lengthen later, hard to shorten after the fact |
| Attachment metadata and files | filenames, `file_ref`, stored objects | Removed with the message; stored objects deleted within 30 days of message deletion | Object handling depends on OD-6 |
| Request links | Konam request IDs, cached state | Deleted with the conversation | Contain no message content |
| Idempotency records | digests, references | **48 hours** after creation | Must outlast any client retry window |
| Sessions | none in v1 | Not applicable | Token lifetime is set by Konam's flow; recommend it be short |
| UI preferences | display settings | Kept while the Konam account is active; deleted within 30 days of Konam's closure notice | |
| Export artefacts | export files | Download link valid **15 minutes**; object deleted after **24 hours** | Private, short-lived references only |
| Logs and traces | structured edge logs | **30 days**, no message content or credentials | Correlation IDs only |
| Backups | database snapshots | No longer than the provider's backup window (7 days on the Pro plan figure in the validation report) | Deleted data may persist in backups until they expire; do not claim immediate deletion there |

Deletion must identify every copy: relational rows, object storage, caches, derived summaries, logs and backups, plus the coordinated deletion of Konam-held task context (OD-7). Completion is claimed only when each applicable store has an evidenced result. Provider-side deletion is not promised without evidence.

## 10. Migration, backup and recovery (future requirements)

No migration is created by this document. When implementation is separately approved:

- Use ordered, reviewed, immutable migration files; test them only against disposable databases in CI before any shared environment.
- Hosted migrations need explicit owner approval; separate schema change from backfill and destructive cleanup.
- Use synthetic seed data only.
- Backups are encrypted, access-controlled and restore-tested; recovery objectives are set before a pilot.
- After every migration, verify grants, row-level security, constraints and the negative tests in section 7.1.

## 11. Threat-model mapping

The controls here address cross-workspace access, forged workspace context, replay and duplicate submission, leakage of content into logs, direct database exposure, stale-state tampering and untrusted model content. The independent review is to map each to the control identifiers in the [Security Threat Model](SECURITY_THREAT_MODEL.md).

## 12. Decisions and open decisions

### Decided (27 November 2026)

- **OD-1 Conversation visibility: private to the creator.** Owner decision. Consequence: no administrator read access to other members' conversations; any compliance or legal-hold access is a Konam-defined, audited process to be specified with the API contract.
- **OD-3 Session model: short-lived validated tokens, no `gateway_sessions` table in v1.** Decided by Claude under the owner's delegation ("choose the best for me"). Reasons: Konam is the authoritative identity layer, so a Gateway session store would duplicate session authority, add a secret-bearing table and complicate revocation; Konam re-validates the token on every operation, so revocation takes effect without a Gateway-side list. Revisit only if the identity flow chosen by Konam cannot support this; that needs an amendment and owner approval.
- **OD-4 Reference format: opaque text, never parsed.** Decided by Claude under the same delegation. `workspace_ref` and `user_ref` are text, at most 128 characters, from a restricted character set, compared by exact match. Reasons: no coupling to Konam's internal ID scheme, any ID type is accepted, and nothing can be inferred from an identifier. Konam is asked to issue non-guessable identifiers.

Items decided by delegation remain open to the owner to overrule, and OD-3 and OD-4 must still be reconciled with the Konam API contract and identity-provider decision.

### Proposed, awaiting owner confirmation

- **OD-2 Retention durations:** the defaults in section 9.

### Still open

- **OD-5** Whether Konam also retains full result content, and the single source of truth for results.
- **OD-6** Attachment storage vendor, scanning, size and type limits, and who stores the bytes.
- **OD-7** Deletion and export coordination between Gateway and Konam, including workspace closure.
- **OD-8** Maximum message size and the large-content strategy.

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
- [x] OD-1, OD-3 and OD-4 decided (OD-3 and OD-4 by delegation).
- [ ] OD-2 retention defaults confirmed by the owner.
- [ ] OD-5 to OD-8 decided (OD-5, OD-7 and OD-8 can follow the Konam API contract).
- [ ] Owner review and acceptance of this re-baseline.
- [ ] Independent cross-document consistency review and threat-model mapping.
- [ ] Alignment with the approved Gateway-to-Konam API contract.
