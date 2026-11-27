# Workflow Gateway — Initial Database Specification

> **Status: Owner accepted the direction on 27 November 2026, but acceptance of this specification is conditional on the re-baseline required by the ownership update below. Until that re-baseline is done and reviewed, no schema may be implemented and WG-004 is not complete.**
>
> Planning artifact only. This document defines a portable relational data model, not executable migrations. PostgreSQL is used as the logical reference because it supports the required relational constraints and row-level security, but Supabase and every hosting/database product remain proposed candidates. No database has been provisioned and no schema has been deployed.
>
> **Cross-repository ownership update (owner-confirmed):** This draft must not be implemented as one Gateway database containing every entity listed below. Workflow Gateway owns canonical customer conversations/messages and UI/session data. Konam owns authoritative platform identity, workspace memberships/roles/entitlements, model/tool/workflow registries, routing policies/decisions, plans, executions/events, approvals, credential governance, usage and platform audit. Gateway may retain only the minimum opaque Konam references and non-authoritative identity linkage needed for its application. This database specification must be re-baselined to separate Gateway-owned storage from Konam-owned platform data before WG-004 is accepted or any schema is implemented. No direct database sharing is permitted.
>
> **Security note:** No plaintext API keys, OAuth refresh tokens, passwords, session tokens or other reusable secrets may be stored in ordinary application tables.

## 1. Purpose and design principles

This historical planning draft describes a broader logical data model. Following the confirmed cross-repository boundary, its physical Gateway database scope is limited to Gateway-owned conversations/messages, UI/session state and necessary non-authoritative references. Platform-wide entities remain described here only for migration/reconciliation analysis and must be stored and governed by Konam, not duplicated in Gateway.

Principles:

1. **Workspace is the tenant boundary.** Personal and organisation workspaces receive the same isolation guarantees.
2. **Default deny.** Every read and write is authorised by the backend; database row-level security (RLS), where supported, provides defence in depth.
3. **Explicit ownership.** User identity, workspace membership, resource ownership and platform administration are distinct concepts.
4. **Immutable version identity.** Published workflow versions and approved action definitions cannot be silently changed in place.
5. **Durable execution.** Jobs and state transitions survive process restarts; duplicate delivery is expected and handled idempotently.
6. **Approval binds to an exact action.** A previous approval cannot authorise changed parameters or a different target.
7. **Credential separation.** Store secret references and safe metadata separately from encrypted secret material or an external secret manager.
8. **Data minimisation.** Keep only the payloads needed for execution, support and audit, with explicit retention and deletion rules.
9. **Portable design.** Avoid depending on Supabase-specific types or features in the logical model.
10. **Auditability.** Security-relevant events are append-only to application roles and separately protected from ordinary tenant data.

## 2. Scope and non-goals

### In scope

- User identity references and personal/organisation workspaces.
- Memberships, roles and entitlements.
- Conversations, messages and file metadata.
- Workflow registry, immutable workflow versions, availability and tenant allow-lists.
- Model and plugin/tool catalogues and tenant connections.
- Encrypted credential references and credential lifecycle metadata.
- Routing policies and routing decisions.
- Executions, execution events, idempotency, job dispatch/outbox and callbacks.
- Approval requests and decisions.
- Usage events and audit logs.
- Retention, deletion, export, indexes, constraints and RLS design.

### Out of scope

- Executable SQL migrations, live database provisioning or Supabase configuration.
- Final vendor selection, pricing or hosting commitments.
- Payment processing, invoicing and subscription-provider integration.
- Storing raw provider API keys in application-readable columns.
- Full file/blob storage implementation.
- Production backup configuration or a claim that any proposed control is already operating.

## 3. Tenant and identity model

### 3.1 Workspace is the universal tenant

**Ownership clarification:** Konam is the authoritative source for workspace identity, membership, roles and entitlements. Gateway must not maintain a competing membership/role database. Any Gateway-side workspace identifier is a reference resolved and authorised through Konam.

A **workspace** is the tenant boundary and can be either:

- `personal`: controlled by one user; or
- `organisation`: controlled by an organisation and its authorised members.

A user may own a personal workspace and belong to multiple organisation workspaces. Every tenant-owned business record carries a non-null `workspace_id`. Organisation-specific records may additionally carry `organisation_id` where that relationship is meaningful, but `organisation_id` is not the universal tenant key.

This intentionally refines the wording in WG-004 that says every tenant-owned table carries an organisation identifier. That wording cannot represent personal workspaces safely or accurately. The issue should be updated to say **every tenant-owned table carries workspace_id; organisation_id is additionally required for organisation-scoped records where applicable**.

The workspace context must be derived from the authenticated session and verified membership, not trusted from a model response, user-supplied payload or workflow callback.

### 3.2 Identity and membership

Authentication provider owns password verification and primary authentication credentials. The application stores a stable external identity subject, not password hashes, unless a separately reviewed identity architecture explicitly requires otherwise.

Membership roles are workspace-scoped. Initial proposed roles:

| Role | Intended authority |
|---|---|
| owner | Workspace ownership, member/role administration, policy and workspace lifecycle |
| admin | Manage members and settings within delegated limits; cannot transfer ownership unless explicitly authorised |
| member | Use workspace features allowed by entitlements and policy |
| viewer | Read-only access to permitted workspace resources |
| billing_manager | View/manage billing and usage settings only, if billing is later implemented |

Platform operator privileges are separate from customer workspace roles and must not be granted merely by being a workspace owner. Platform administration requires a separately authenticated and audited control plane.

### 3.3 Ownership and lifecycle

- Each personal workspace has one owning user.
- Each organisation workspace has one current owner and may have multiple members.
- Ownership transfer is an explicit, audited operation with acceptance by the new owner.
- Removing a member revokes future workspace access and active integration authorisations attributable to that member, subject to a defined continuity policy.
- Suspending a workspace blocks new executions and external actions; read/export access for owners is governed by the suspension reason and policy.
- Deletion is a controlled lifecycle, not a cascade that immediately destroys audit evidence.

## 4. Logical entity catalogue

The following catalogue is the proposed baseline. Names are logical table names, not final physical naming conventions.

### 4.1 Identity and tenancy

| Entity | Key fields | Purpose and rules |
|---|---|---|
| users | id, auth_subject, email_normalised, display_name, status, created_at, updated_at | Application profile linked to external auth subject; unique auth_subject; email uniqueness policy must account for provider verification and case normalisation |
| workspaces | id, workspace_type, name, slug, owner_user_id, status, created_at, deleted_at | Tenant root; type is personal or organisation; personal workspace owner required; organisation owner required |
| organisations | id, workspace_id, legal_name, display_name, registration_country, status | Organisation profile associated one-to-one with an organisation-type workspace; no organisation row for a personal workspace |
| memberships | id, workspace_id, user_id, role, status, invited_by_user_id, joined_at, revoked_at | Workspace access; unique active membership per workspace/user; membership changes audited |
| workspace_invitations | id, workspace_id, email_normalised, role, token_digest, expires_at, accepted_by_user_id, status | Invitation workflow; store token digest only; single-use, expiry and replay protection |
| workspace_settings | workspace_id, settings_json, version, updated_by_user_id, updated_at | Validated workspace preferences; secrets and credentials prohibited |

### 4.2 Conversations and files

| Entity | Key fields | Purpose and rules |
|---|---|---|
| conversations | id, workspace_id, created_by_user_id, title, status, created_at, updated_at, archived_at | Conversation container; tenant-scoped; creator must be a workspace member |
| messages | id, workspace_id, conversation_id, sequence_no, sender_type, sender_user_id, content, content_format, model_ref, created_at, redaction_state | Conversation messages; unique conversation/sequence; sender type constrained; model/tool content treated as untrusted |
| message_attachments | id, workspace_id, message_id, file_object_ref, filename_display, media_type, size_bytes, checksum, classification, created_at | Metadata only; bytes stored in separately controlled object storage; validate content type and access |
| data_export_jobs | id, workspace_id, requested_by_user_id, scope_json, status, object_ref, expires_at, completed_at | Controlled export workflow; owner/admin policy; export links short-lived and audited |

Large message payloads may later require a separate content/blob strategy. The database must not silently duplicate large binary files or retain unnecessary prompt/tool payloads indefinitely.

### 4.3 Proprietary workflow registry

| Entity | Key fields | Purpose and rules |
|---|---|---|
| workflows | id, platform_owner_id, stable_key, name, description, owner_type, owner_workspace_id, status, created_at | Stable workflow identity; platform-owned workflows may have no customer workspace; unique stable_key |
| workflow_versions | id, workflow_id, version, lifecycle_state, availability_scope, input_schema, output_schema, execution_mode, adapter_type, adapter_config_ref, permissions_json, approval_policy_json, data_classification, released_at, created_at | Immutable versioned contract; unique workflow/version; adapter config contains references, never secrets |
| workflow_capabilities | workflow_version_id, capability_key, weight | Search/discovery capabilities; unique version/capability |
| workflow_customer_allowlist | id, workflow_version_id, workspace_id, granted_by, granted_at, expires_at, status | Explicit selected-customer access; required only for selected-customer availability |
| workflow_dependencies | workflow_version_id, dependency_type, dependency_ref, version_constraint | Versioned dependency declarations; validate cycles and compatibility before release |
| workflow_lifecycle_events | id, workflow_id, workflow_version_id, from_state, to_state, actor_type, actor_id, reason, approval_ref, created_at | Append-only transition history; authorised state machine only |

A workflow version's executable contract, schemas, required permissions, approval policy and material adapter configuration are immutable after release. Corrections require a new version. Runtime discovery must filter lifecycle, availability, workspace allow-list and entitlement before ranking. A suspended/deprecated version must not accept new executions.

The platform workflow catalogue is distinct from customer-owned workflow definitions. If customer-authored workflows are introduced later, ownership and publication permissions require a separate review.

### 4.4 Models, plugins and integrations

| Entity | Key fields | Purpose and rules |
|---|---|---|
| model_providers | id, provider_key, display_name, status, policy_metadata, created_at | Platform catalogue of provider identities; no credentials |
| models | id, provider_id, model_key, display_name, capabilities_json, input_limits_json, status | Model catalogue and supported capability metadata |
| plugins | id, plugin_key, display_name, publisher, status, trust_level, manifest_version | Platform catalogue of integration/tool packages |
| plugin_versions | id, plugin_id, version, manifest_json, permission_scopes_json, endpoint_ref, status, created_at | Immutable plugin manifest/version; no embedded secrets |
| workspace_integrations | id, workspace_id, plugin_version_id, connected_by_user_id, status, granted_scopes_json, external_account_ref, credential_id, connected_at, revoked_at | Workspace connection and consent; bind to workspace and credential; revocation stops future use |
| integration_tool_permissions | id, workspace_id, integration_id, tool_key, allowed, scope_json, updated_by_user_id, updated_at | Explicit tool-level enablement and least-privilege scopes |

Tool manifests and descriptions are untrusted input. A plugin being registered or speaking MCP does not automatically make it trusted or authorised. Tool permissions are enforced by the backend at invocation time.

### 4.5 Credentials and secret references

| Entity | Key fields | Purpose and rules |
|---|---|---|
| credentials | id, workspace_id, credential_type, provider_or_integration_ref, owner_user_id, secret_store_ref, encrypted_payload_ref, encryption_key_version, masked_hint, status, created_at, expires_at, rotated_at, revoked_at | Credential metadata and pointer to protected secret material; never store plaintext |
| credential_access_events | id, workspace_id, credential_id, actor_type, actor_id, purpose, execution_id, outcome, created_at | Append-only evidence of credential use; never include secret values |

Only one of `secret_store_ref` or `encrypted_payload_ref` is expected for a credential instance. The selected storage pattern must be decided in the credential-security ADR before implementation. If envelope encryption is selected, ciphertext, nonce, authentication tag and key-version metadata are stored separately from the key-encryption key. The key-encryption key must not be stored in the same database.

Rules:
- Decryption occurs server-side only, immediately before a provider/tool call.
- Plaintext is not returned to the browser, model, logs, traces, audit events or error messages.
- Credentials are workspace-bound and access is checked against the active workspace and integration.
- Rotation/revocation is supported; member departure triggers review/revocation of member-owned connections.
- Secret-store access is least-privilege and auditable.
- Development uses synthetic credentials only until the security controls are implemented and tested.

### 4.6 Routing and policy

| Entity | Key fields | Purpose and rules |
|---|---|---|
| routing_policies | id, workspace_id, version, policy_json, status, created_by_user_id, created_at, activated_at | Versioned workspace routing preferences; validated policy schema |
| routing_decisions | id, workspace_id, conversation_id, message_id, policy_version, selected_route_type, selected_route_ref, rationale_code, decision_metadata_redacted, created_at | Records route decision without unnecessary prompt or secret content |

A routing decision is evidence of a choice, not permission to execute. Every chosen workflow/tool/model route is separately checked against current membership, entitlements, lifecycle, scopes and approval policy.

### 4.7 Executions, jobs and events

| Entity | Key fields | Purpose and rules |
|---|---|---|
| executions | id, workspace_id, requested_by_user_id, conversation_id, workflow_version_id, route_type, route_ref, status, idempotency_key, input_ref, output_ref, error_code, created_at, started_at, completed_at | Durable execution record; tenant-scoped; avoid storing sensitive payloads inline where references suffice |
| execution_events | id, workspace_id, execution_id, event_sequence, event_type, safe_summary, metadata_redacted, actor_type, actor_id, created_at | Append-only execution timeline; unique execution/event_sequence |
| execution_attempts | id, workspace_id, execution_id, attempt_no, worker_ref, status, started_at, finished_at, error_code, retryable | Individual worker attempts; bounded retries and no secrets in errors |
| job_outbox | id, workspace_id, aggregate_type, aggregate_id, event_type, payload_minimal, status, available_at, published_at, attempts, created_at | Transactional outbox for reliable dispatch; minimal payload; unique event identity |
| callback_receipts | id, workspace_id, execution_id, source_ref, nonce_digest, received_at, signature_valid, accepted, rejection_code | Replay protection and callback evidence; never trust callback claims without verification |
| idempotency_records | id, workspace_id, operation_scope, idempotency_key, request_digest, resource_type, resource_id, status, expires_at, created_at | Prevent duplicate effective actions; unique workspace/scope/key; same key with changed digest is rejected |

Execution state is a controlled state machine, for example: queued, running, awaiting_approval, succeeded, failed, cancelled, timed_out. Invalid or stale transitions are rejected. Worker delivery is at-least-once in the logical design; the system must not claim exactly-once delivery. Effective side effects require idempotency support or a documented reconciliation path.

Do not store arbitrary executable instructions in queue payloads. Queue messages should carry identifiers and minimal validated metadata; workers reload and re-authorise the execution from authoritative storage.

### 4.8 Approvals

| Entity | Key fields | Purpose and rules |
|---|---|---|
| approval_requests | id, workspace_id, execution_id, requested_by_user_id, approver_user_id, action_type, target_ref, action_digest, workflow_version_id, policy_version, data_scope_digest, status, reason, expires_at, created_at | Immutable approval request bound to exact proposed action |
| approval_decisions | id, workspace_id, approval_request_id, decided_by_user_id, decision, reason, decided_at, consumed_at | One-time decision record; authorised approver only |

The action digest must be computed server-side from a canonical representation of the exact action, target, parameters, data scope, workflow/tool version and relevant policy context. A change to any material component requires a new approval. Approval is not inferred from model output, retrieved text, a prior unrelated approval or a client-supplied boolean.

Execution must re-check the approver's authority, current membership, current entitlement, action digest, expiry, status and one-time consumption before performing the side effect. Explicit policy denies cannot be overridden by approval.

### 4.9 Entitlements, usage and audit

| Entity | Key fields | Purpose and rules |
|---|---|---|
| entitlement_plans | id, plan_key, display_name, feature_limits_json, status | Platform-defined plan catalogue; pricing/billing integration deferred |
| workspace_entitlements | id, workspace_id, plan_id, feature_key, limit_value, valid_from, valid_until, status, source_ref | Effective workspace feature/limit grant; enforce server-side |
| usage_events | id, workspace_id, actor_user_id, execution_id, provider_id, model_id, workflow_version_id, event_type, quantity, unit, cost_estimate, occurred_at, metadata_redacted | Append-only metering facts; avoid treating estimates as invoices |
| audit_logs | id, workspace_id, actor_type, actor_id, action, resource_type, resource_id, outcome, correlation_id, metadata_redacted, occurred_at | Append-only security/business evidence; no secret or unnecessary content |
| retention_policies | id, workspace_id, data_category, retention_days, legal_hold, policy_version, updated_by_user_id, updated_at | Explicit retention settings within platform minimum/maximum rules |
| deletion_requests | id, workspace_id, requested_by_user_id, scope_json, status, requested_at, completed_at, verification_ref | Track deletion across relational, object, derived and provider stores |

Entitlement records must not be treated as a substitute for current role and permission checks. Usage events record measurable activity; billing and pricing policy are future decisions.

## 5. Relationships and cardinality

Key relationships:

- `users 1—many memberships`; `workspaces 1—many memberships`.
- `workspaces 1—many conversations`; `conversations 1—many messages`; messages may have many attachment metadata rows.
- `workflows 1—many workflow_versions`; versions may have many capabilities, dependencies and lifecycle events.
- `workflow_versions 1—many executions`; executions have many events and attempts.
- `workspaces 1—many credentials, integrations, policies, executions, approvals, usage events and retention policies`.
- `model_providers 1—many models`; `plugins 1—many plugin_versions`.
- `workspace_integrations` may reference a credential; a credential may be referenced by one or more authorised integrations only where explicitly intended.
- `executions 1—many approval_requests` and `execution_events`.
- `approval_requests 1—zero/one approval_decision` in the initial one-decision model; if re-review is needed, create a new request rather than overwriting the prior decision.

Foreign keys should prevent cross-workspace references wherever both parent and child are tenant-owned. Prefer composite constraints such as `(workspace_id, conversation_id)` referencing a unique `(workspace_id, id)` on conversations. This prevents a row in Workspace A from referencing a conversation in Workspace B even if an application bug supplies the other workspace's ID.

Platform-owned catalogue tables (models, provider catalogue, workflow identity and plugin catalogue) are not tenant-owned. Tenant connections, policies, allow-lists, executions and usage are tenant-owned and must carry workspace_id.

## 6. Keys, constraints and indexes

### 6.1 Key strategy

- Use opaque UUID primary keys for application-generated entities unless a documented reason requires otherwise.
- Never expose sequential database IDs as authorisation tokens.
- Use stable text keys only for human-readable catalogue identifiers, not as sole foreign keys.
- Use UTC timestamps for persisted event times.
- Use explicit status enums/check constraints or controlled lookup values; avoid arbitrary status strings.
- Store provider payloads as validated JSON only where schema evolution requires it; relationally model fields used for security, joins, filtering or uniqueness.

### 6.2 Required uniqueness and integrity constraints

At minimum:

- Unique `users.auth_subject`.
- Unique `organisations.workspace_id`.
- Unique active `memberships(workspace_id, user_id)`.
- Unique `workflow_versions(workflow_id, version)`.
- Unique `workflow_capabilities(workflow_version_id, capability_key)`.
- Unique `models(provider_id, model_key)`.
- Unique `plugin_versions(plugin_id, version)`.
- Unique `messages(conversation_id, sequence_no)` (with workspace consistency enforced).
- Unique `execution_events(execution_id, event_sequence)`.
- Unique `execution_attempts(execution_id, attempt_no)`.
- Unique `idempotency_records(workspace_id, operation_scope, idempotency_key)`.
- Unique `callback_receipts(source_ref, nonce_digest)` where callback source semantics permit.
- Check constraints for workspace type/owner, lifecycle transitions, non-negative quantities and valid time ranges.
- Foreign keys for all references; composite workspace-scoped foreign keys for tenant-owned relations.
- Prevent deleting referenced immutable workflow versions while execution/audit history depends on them.

### 6.3 Initial indexes

- Membership lookup: `memberships(user_id, status, workspace_id)` and `memberships(workspace_id, status, role)`.
- Workspace recency: `conversations(workspace_id, updated_at DESC)`.
- Message history: `messages(workspace_id, conversation_id, sequence_no)`.
- Workflow discovery: indexes on lifecycle_state, availability_scope and stable_key; capability index by capability_key and workflow_version_id.
- Selected-customer allow-list: `workflow_customer_allowlist(workspace_id, workflow_version_id, status)`.
- Execution dashboard: `executions(workspace_id, created_at DESC)` and `executions(workspace_id, status, created_at)`.
- Worker polling: `job_outbox(status, available_at)`; index only fields required for queue admission.
- Event timeline: `execution_events(workspace_id, execution_id, event_sequence)`.
- Approval inbox: `approval_requests(workspace_id, approver_user_id, status, expires_at)`.
- Usage reporting: `usage_events(workspace_id, occurred_at, event_type)`.
- Audit investigation: `audit_logs(workspace_id, occurred_at DESC)` and `audit_logs(actor_id, occurred_at DESC)`.
- Credential lookup: `credentials(workspace_id, status, credential_type)`; never index secret material.

Index choices must be confirmed against query plans and actual data volume; do not add broad indexes to every column.

## 7. Tenant isolation and RLS proposal

### 7.1 Mandatory application rules

For every tenant-scoped request:

1. Authenticate the user.
2. Resolve the requested workspace from trusted session/request context.
3. Verify active membership and role for that workspace.
4. Verify resource ownership and workspace_id on every read/write.
5. Check entitlement, integration scopes, workflow lifecycle and approval requirements.
6. Apply the same checks in API endpoints, workers, callbacks, exports and background jobs.
7. Record security-relevant outcomes without exposing sensitive payloads.

Never accept workspace_id from a model, tool, callback or untrusted document as authoritative. If a client sends a workspace identifier, treat it only as a selector and verify membership server-side.

### 7.2 RLS coverage proposal

RLS should be enabled and tested on all tenant-owned tables, including:

- organisations, memberships, invitations, workspace_settings;
- conversations, messages, message_attachments, data_export_jobs;
- workflow_customer_allowlist where workspace-scoped;
- workspace_integrations, integration_tool_permissions, credentials, credential_access_events;
- routing_policies, routing_decisions;
- executions, execution_events, execution_attempts, job_outbox, callback_receipts, idempotency_records;
- approval_requests, approval_decisions;
- workspace_entitlements, usage_events, audit_logs, retention_policies, deletion_requests.

Platform catalogue tables require a separate read/write policy: public/tenant users receive only approved catalogue metadata; platform operators have tightly scoped management privileges. Private workflow metadata and unreleased versions must not leak through discovery.

RLS policies must use a trusted database identity/context that cannot be forged by the end user. If a privileged service role bypasses RLS, it must be confined to server-side code, never shipped to the browser, and every query must still enforce workspace scope. Background workers must establish and validate workspace context for each job.

### 7.3 Required negative tests

- User in Workspace A cannot read, update, delete or infer Workspace B records.
- Member cannot elevate own role or invite an owner.
- Revoked member cannot access prior workspace resources or use revoked integration credentials.
- Workspace A cannot attach a message/execution/approval to a Workspace B parent row.
- Callback cannot change another workspace's execution.
- Worker cannot process a job whose workspace/tenant context does not match the authoritative execution record.
- RLS and API-level tests agree; one layer must not compensate for a broken other layer.
- Platform catalogue discovery does not reveal internal-only workflows or private tenant metadata.

## 8. Execution, approval and audit integrity

- Create execution, initial event and outbox record in one database transaction.
- Outbox publisher marks dispatch only after successful queue publication; recovery scans pending records.
- Worker claims are bounded and lease-based; expired leases can be recovered.
- Every attempt records a distinct attempt number; retries do not create a new logical action identity.
- State transitions are conditional and monotonic; stale callbacks cannot regress state.
- Idempotency keys are scoped by workspace and operation; same key/different request digest is a conflict.
- External side effects use the downstream provider's idempotency facility when available. Otherwise define a reconciliation/manual-review strategy per workflow.
- Callback authentication includes source identity, signature, timestamp and replay nonce; callback content is schema-validated and cannot choose arbitrary destinations.
- Approval is consumed atomically with the transition to authorised execution; race/replay tests are required.
- Audit events are append-only to application roles. Corrections use compensating events, not edits to prior evidence.
- Database administrators may technically alter records; stronger tamper evidence or external immutable storage may be considered before production, based on risk and cost.

## 9. Data classification and retention proposal

Retention durations below are **not yet decided**. WG-004 acceptance requires the owner to review the categories and choose durations before pilot. Until then, do not assume indefinite retention is acceptable.

| Data category | Examples | Proposed handling |
|---|---|---|
| Identity and membership | User profile, role, invitation | Retain while account/workspace active; delete or anonymise subject to legal and audit needs |
| Conversation content | Messages, prompts, outputs | Workspace-configurable retention; user deletion/archive; avoid unnecessary copies in logs |
| Attachments | Uploaded files and metadata | Private storage, malware/content validation, expiry and deletion of object plus metadata |
| Workflow catalogue | Workflow metadata and immutable versions | Retain version history while referenced by executions/audit; deprecate rather than mutate |
| Credentials | Secret reference, ciphertext, masked hint | Revoke promptly; delete protected material after revocation/retention obligations; metadata retained only as needed |
| Execution payloads | Inputs/outputs and results | Prefer encrypted object references; minimise content; short operational retention unless user elects to save |
| Execution events | Progress and state changes | Retain sufficient for support and security; redact content |
| Approvals and audit | Decision, actor, action digest, outcome | Retain under a defined security/legal schedule; append-only |
| Usage events | Model/workflow consumption | Retain for cost reporting and dispute period; no unnecessary prompt content |
| Logs and traces | Operational diagnostics | Short retention; payload suppression and secret redaction; access restricted |
| Backups | Database snapshots | Defined expiry, access controls and deletion propagation/expiry documentation |
| Provider-side data | Model/tool processing | Governed by provider terms, data policy and customer disclosure; platform deletion cannot promise provider deletion without evidence |

Deletion workflow must identify all copies: relational rows, object storage, caches, search indexes, derived summaries, logs, backups and third-party providers. Legal hold may suspend deletion for a specific scope and must be recorded. Deletion completion must not be claimed until each applicable store has an evidenced result. Backup expiry may be delayed but must have a documented maximum period.

### Export

Workspace export should include the requesting user's authorised scope, preserve relationships and timestamps, exclude secrets and other users' private data unless the requester is authorised, and be delivered through a short-lived private object reference. Export creation and download are audited.

## 10. Migration, backup and recovery approach (future implementation requirements)

No migration is created by this specification. When implementation is separately approved:

- Use ordered, immutable migration files with checksums and review.
- Test migrations against an isolated disposable database before any shared environment.
- Separate schema migration from data backfill and destructive cleanup.
- Require explicit owner approval for hosted database migrations, especially production or irreversible changes.
- Provide forward migration and documented rollback/compensation strategy; do not assume every migration is safely reversible.
- Use synthetic seed data only; never seed production credentials or personal data.
- Backups must be encrypted, access-controlled and periodically restore-tested.
- Document recovery point and recovery time objectives before customer pilot.
- Verify RLS policies, grants, indexes, constraints and tenant isolation after every migration.
- Record migration actor, version, environment, approval and result.

## 11. Threat-model mapping

| Threat-model concern | Data-model/control response | Verification |
|---|---|---|
| Cross-workspace access | workspace_id on tenant records; composite foreign keys; RLS; API and worker checks | Cross-tenant negative tests across every access path |
| Role/entitlement escalation | memberships, workspace roles, workspace_entitlements; separate platform roles | Role matrix and deny-by-default tests |
| Prompt injection/tool poisoning | route decisions do not grant authority; tool permissions and scopes checked at invocation | Adversarial tool tests and scope-denial tests |
| BYOK theft | credentials table contains references/ciphertext metadata only; separate key protection; access events | Synthetic canary secret scans of logs, traces, errors and outputs |
| Approval bypass/replay | immutable action_digest, expiry, one-time decision and atomic consumption | Parameter mutation, expiry, replay and race tests |
| Duplicate jobs/side effects | outbox, idempotency records, attempts and reconciliation | Worker crash/restart and duplicate-delivery tests |
| Callback forgery | callback receipts, signature, timestamp, nonce and monotonic state transitions | Forged, stale, duplicate and cross-tenant callback tests |
| Audit tampering | append-only application semantics, restricted roles and optional external integrity sink | Privilege tests and audit integrity checks |
| Retention/deletion failure | retention policies, deletion requests and store inventory | End-to-end deletion and backup-expiry evidence |
| Cost exhaustion | entitlement limits and usage events; rate/concurrency budgets enforced by service layer | Load and abuse tests with bounded spend |

## 12. Open decisions requiring owner review

1. Confirm workspace_id as the universal tenant key and amend WG-004 wording accordingly.
2. Confirm whether personal workspace creation is automatic at sign-up or user-initiated.
3. Confirm initial workspace roles and whether billing_manager is needed in v1.
4. Decide organisation ownership transfer and member-removal behaviour.
5. Select the credential storage pattern only after validating secret-manager availability, free-tier limits and key-management security.
6. Define data classification and allowed provider/tool data categories.
7. Set retention durations for conversations, attachments, execution payloads, events, usage and audit.
8. Decide whether platform-owned workflow registry data shares the same database or a separately controlled schema/service.
9. Define whether workflow publishers beyond the platform owner exist in the first release.
10. Define exact entitlement limits and whether they are manual/internal during the initial pilot.
11. Decide whether full-text/vector search is required initially; it is not a prerequisite for the relational baseline.
12. Confirm applicable legal, privacy, jurisdiction and data-residency requirements before real customer data is used.

All items above remain proposed until owner approval. The technology ADRs remain proposals and this specification does not accept Supabase, Render, FastAPI, Next.js or any model/tool provider.

## 13. Acceptance checklist for WG-004

- [x] Logical entities cover organisations, users, memberships, conversations, messages, models, workflows, workflow_versions, plugins, executions, execution_events, approvals, credentials (reference/ciphertext only), usage_events, audit_logs and entitlements.
- [x] Additional entities needed for tenancy, integrations, routing, idempotency, durable jobs, retention and deletion are defined.
- [x] Personal and organisation workspace tenancy is modelled.
- [x] Tenant keys, key relationships, uniqueness and indexing notes are specified.
- [x] RLS coverage and application-layer authorisation rules are proposed.
- [x] Credential and approval security requirements are mapped to the threat model.
- [x] Retention, deletion and export behaviour are specified as proposals.
- [x] Migration, backup and recovery requirements are documented without performing them.
- [x] Unresolved choices are explicitly marked as proposed.
- [ ] Owner review and acceptance.
- [ ] Independent cross-document consistency review after owner feedback.

## 14. Review record

- **Issue:** WG-004 — Define initial database specification.
- **Document status:** Proposed; owner acceptance pending.
- **Architecture reviewer:** ChatGPT, Chief AI Architect and Independent Reviewer.
- **Source inputs:** Product Requirements, Technical Architecture, Workflow Registry Specification, Security Threat Model, ADR proposals and WG-004 issue.
- **Implementation status:** None. No database provisioned; no migrations generated or executed.
- **Last prepared:** 26 November 2026.
