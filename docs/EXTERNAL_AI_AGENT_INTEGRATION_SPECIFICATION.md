# External AI Agent Integration Specification

> Status: proposed architecture baseline for WG-031. Planning only. This document does not authorise implementation, service provisioning, external exposure or production credentials.

## 1. Purpose and design decision

Workflow Gateway is the external access boundary for AI assistants, agents and applications. It exposes stable REST/OpenAPI and inbound MCP interfaces, then forwards authenticated, scoped requests to the versioned Konam platform API. Konam is the system of record and authority for capability discovery, platform identity/permissions, routing, execution, approvals, task status, results, usage and audit.

**Decision proposed:** provide two Gateway-side interface adapters over a single integration layer to Konam's governed platform services:

1. A versioned REST API described by OpenAPI.
2. A remote MCP server adapter that presents approved Workflow Gateway capabilities as MCP tools.

These are interface layers, not separate orchestration engines, registries, job systems or permission stores. Gateway's internal chat, REST clients and MCP clients must converge on the same Gateway-to-Konam integration contract. Konam performs authoritative policy enforcement and execution.

## 2. Terminology and integration direction

There are two different MCP directions and they must not be conflated:

- **Outbound MCP client integration:** Workflow Gateway connects to third-party MCP servers to consume their tools. This is already in the plugin/tool integration direction and is covered by WG-021.
- **Inbound MCP server exposure:** an external assistant connects to Workflow Gateway's MCP endpoint and consumes our registered workflows as tools. This is the additional capability specified here.

The same platform may support both directions through adapters, but each connection has its own trust boundary, identity, permissions and audit events.

## 3. Logical architecture

External clients connect to a REST or MCP adapter at the Workflow Gateway boundary. The adapter authenticates the external client/user at the edge, validates request schemas and forwards an appropriately scoped request to the Konam platform API. Konam then performs:

- Authoritative identity, workspace, membership, role and entitlement validation.
- Capability and workflow discovery, filtered by current permissions.
- Routing and policy evaluation.
- Workflow/tool/model execution admission and durable task supervision.
- Human approval decisions and enforcement.
- Canonical task status, events and result services.
- Permission-checked file-reference resolution.
- Platform usage metering, limits and audit.

No external interface may call workflow engines, model providers or workers directly while bypassing Konam. Gateway must not implement a parallel workflow registry, authoritative permission store, routing engine or execution supervisor.

## 4. REST API and OpenAPI

### 4.1 API conventions

- Use a versioned base path, initially proposed as `/v1`.
- Publish an OpenAPI 3.x document as the contract for supported endpoints, authentication schemes, schemas, error responses and deprecation policy.
- Use JSON request and response bodies; use standard HTTP status semantics.
- Validate all inputs and outputs against registered JSON Schemas.
- Use correlation IDs and stable execution IDs; do not expose internal database identifiers as credentials.
- Support pagination, bounded response sizes, request timeouts and rate-limit headers.
- Define compatibility rules: additive optional fields may be introduced compatibly; breaking changes require a new API version or an explicitly versioned contract.
- Generate API reference documentation and contract tests from the same maintained OpenAPI source where practical.

### 4.2 Initial logical endpoint set

An initial machine-readable contract is maintained in [api/openapi/v1.yaml](../api/openapi/v1.yaml). It is a proposed planning baseline, not a deployed API or a final provider-specific authentication configuration; it must be reviewed and refined during WG-031 and WG-011/WG-015 implementation planning. It defines the initial endpoint set and schemas, not an instruction to expose every workflow:

| Operation | Purpose |
|---|---|
| `GET /v1/capabilities` | Discover capabilities visible and usable by the authenticated principal |
| `GET /v1/workflows/{workflow_id}` | Retrieve permitted workflow description, version and input/output schema |
| `POST /v1/executions` | Submit an authorised workflow execution request |
| `GET /v1/executions/{execution_id}` | Retrieve status and permitted progress information |
| `POST /v1/executions/{execution_id}/cancel` | Request cancellation where supported and authorised |
| `GET /v1/executions/{execution_id}/result` | Retrieve a completed, authorised result |
| `GET /v1/files/{file_id}` | Resolve or download a file through a permission-checked mechanism |

Discovery must return only workflows that pass lifecycle, availability, entitlement and caller-permission checks. A workflow being technically registered does not make it externally discoverable.

### 4.3 Execution request and response

An execution request should identify the workflow and version, provide schema-valid inputs, and carry an idempotency key. Konam derives and validates the authoritative principal, tenant/workspace context, permissions and approval policy from trusted server-side identity context; client-supplied identity or workspace values are selectors only and must be independently checked.

The acknowledgement should contain an execution ID, accepted/queued status, status URL and whether human approval is pending. Synchronous completion may return a result directly only where the workflow contract and timeout policy allow it. Long-running work returns an acknowledgement and is polled or receives a supported event/callback notification.

A stable result envelope should contain:
- execution ID and workflow identity/version;
- terminal or current status;
- validated structured output;
- created-file metadata and protected references;
- timestamps and safe usage summary where permitted;
- warnings and a standard error object when relevant.

Errors should have stable machine-readable codes, safe human-readable messages, retryability and a correlation ID. Never return stack traces, secrets, provider credentials or another tenant's data.

## 5. MCP server adapter

The platform should expose a remote MCP server endpoint using a transport and protocol version supported by target clients at implementation time. The endpoint is an adapter over the same backend services used by REST.

Prefer a small, stable generic tool surface initially, for example:
- `list_available_workflows` — permission-filtered capability discovery;
- `get_workflow_schema` — retrieve permitted workflow details and schemas;
- `execute_workflow` — submit a validated execution request;
- `get_execution_status` — inspect status and safe progress;
- `get_execution_result` — retrieve permitted structured results and file references;
- `cancel_execution` — request cancellation if supported.

A generic tool surface avoids registering every workflow as a permanently distinct MCP tool and reduces tool-definition churn. Per-workflow tools may be considered later if client discovery quality or usability demonstrates a clear benefit. Tool descriptions and schemas must be concise, stable, versioned and free of secret/internal implementation details.

MCP tool availability is not authorisation. Every invocation must re-check the caller's current grants, workflow lifecycle, entitlement, data scope, rate limits and approval requirements. Tool output is untrusted when consumed by another model.

## 6. External identity and authorisation

### 6.1 Principals

Distinguish at least:
- a human user acting through an external assistant;
- an authorised external application or agent acting under a delegated grant;
- a platform-controlled service identity used for approved machine-to-machine operations.

Record both the authenticated client/agent identity and the represented human/workspace context where delegation applies. Do not treat an AI model name or natural-language claim as an identity.

### 6.2 Preferred authorisation patterns

- For interactive user-connected clients, evaluate OAuth 2.1 authorisation-code flow with PKCE and narrowly scoped access tokens.
- For trusted non-interactive integrations, evaluate a machine-to-machine client credential or workload identity pattern, with separately provisioned scopes, short-lived tokens where possible, rotation and revocation.
- Use API keys only as a constrained compatibility option where a target integration explicitly requires them; keys must be high-entropy, stored hashed where feasible, shown once, scoped, revocable, rate-limited and never treated as a substitute for delegated user consent.
- Do not assume every external assistant supports every OAuth grant or credential mechanism. ChatGPT MCP compatibility must be validated against current product documentation and the actual target account/workspace configuration before integration.
- Do not assume BYOK model-provider credentials are external-agent credentials. They serve different purposes and must remain separate.

### 6.3 Scope model

Define scopes at minimum across:
- workspace or tenant boundary;
- workflow identity and permitted versions;
- operation (discover, read schema, execute, read status, read result, download file, cancel);
- data classification and any permitted data categories;
- environment or deployment stage, if multiple environments exist.

Default deny. A token or grant must not gain broader access because a model requests it, a workflow name is guessed, or a tool returns a resource identifier. Revocation and membership/entitlement changes must take effect promptly, including for long-running jobs where policy requires re-checking before consequential steps.

## 7. Approval and execution governance

External invocation must use Konam's approval manager and action-bound approval model. Konam, not Gateway or the assistant, decides whether approval is required. Client-side confirmation, model reasoning, MCP tool annotations or a previous unrelated approval are not substitutes for a valid platform approval.

For high-risk or irreversible actions:
- record the exact action, target, parameters, data scope and workflow version in the approval context;
- pause execution until an eligible approver decides;
- re-check current permission, entitlement, expiry and one-time consumption before side effects;
- require a new approval if material parameters change;
- audit the requesting principal, external client, represented user, approval and outcome.

## 8. Asynchronous execution, events and chaining

REST and MCP clients must use the same durable execution state machine and idempotency rules as the internal chat application. Expose safe status and progress; do not promise exactly-once side effects where downstream systems cannot guarantee them.

Initial support should use execution acknowledgement plus status polling. Signed callbacks or streaming notifications may be added where justified, with source authentication, replay protection, bounded payloads and safe egress rules.

Workflow chaining and execution supervision are owned by Konam's Capability Router and execution services. External agents may submit a multi-step goal only through a defined, policy-checked request contract; they must not obtain unrestricted internal workflow-to-workflow privileges. Each step is independently authorised, metered and audited by Konam, with approval gates applied at the relevant action boundary.

## 9. Results and generated files

Return structured output validated against the workflow's declared output schema. Files must remain private by default. External clients receive opaque, short-lived, permission-checked references; the server re-checks execution ownership, workspace scope, caller grant and expiry at retrieval time.

Do not place long-lived public storage URLs, credentials or sensitive file contents in MCP descriptions, logs or audit metadata. File download limits, content type, malware scanning and retention/deletion policy must align with the platform's file-handling controls.

## 10. Abuse controls, observability and audit

Apply per-client, per-principal and per-workspace rate and concurrency limits. Enforce payload, execution duration, result-size and file-download bounds. Use idempotency keys and duplicate-request detection. Provide client-level disable/revoke controls and anomaly signals.

Audit at least:
- client registration, consent, scope grants and revocation;
- authentication failures and token validation outcomes;
- capability discovery and denied discovery where security relevant;
- execution submissions, status/result access and cancellation;
- approval requests and decisions;
- file-reference creation and retrieval;
- rate-limit and policy-denial events.

Logs and traces must redact tokens, API keys, secrets and unnecessary prompt/output content. Retention and deletion requirements apply equally to external-agent activity.

## 11. ChatGPT and external-client compatibility

The platform must remain assistant-independent. ChatGPT is an important target, not the protocol authority for the whole ecosystem.

Before enabling a specific client, validate:
- whether that client supports remote MCP servers or OpenAPI-based actions/connectors;
- supported transport and protocol versions;
- supported OAuth flows, PKCE, scopes and token audiences;
- whether read and write tools are available for the user's plan and workspace;
- administrator approval, tool publication and connection policies;
- confirmation behaviour, timeouts, payload limits and file handling.

A ChatGPT MCP connection may require OAuth 2.1 authorisation-code with PKCE and may not support machine-to-machine client credentials or arbitrary API-key authentication as the MCP connection credential. These are client-specific constraints to verify at integration time, not reasons to weaken platform security. OpenAI API applications using remote MCP are a separate integration path and must manage their own authorisation.

If a client cannot meet the platform's authentication or permission requirements, do not create a weaker public endpoint for it. Use another supported integration path or defer that client.

## 12. Mapping to the existing backlog

| Existing issue | Add or clarify |
|---|---|
| WG-007 Authentication | Human sessions and delegated external-client authorisation are distinct; do not conflate them |
| WG-011 Workflow metadata | Stable capability descriptions, external visibility policy, input/output schemas, required scopes and compatibility metadata |
| WG-013 Capability search | Caller-filtered discovery; no leakage of hidden workflow existence or metadata |
| WG-015 Execution adapter | Reusable application service contract, idempotent invocation, external principal context and standard result/error envelope |
| WG-016 Async tracking | External status/result access and optional authenticated notifications using existing durable jobs |
| WG-021 Plugin/MCP registry | Preserve outbound MCP client support and add inbound Workflow Gateway MCP server adapter as a separate direction |
| WG-022 Approval manager | External requests use the same platform-enforced approval gates |
| WG-023 Tenant isolation and permissions | Client identity, delegated grants, scopes, revocation, per-client limits and cross-tenant negative tests |
| WG-030 Security/pilot testing | REST/OpenAPI and MCP contract, auth, abuse, data-leakage and compatibility tests |
| WG-031 External-agent architecture | Owns this design and cross-issue traceability |

Do not create duplicate authoritative authentication/authorisation, capability discovery, queue/task supervision, approval, registry, execution or audit subsystems in Workflow Gateway. Gateway retains its customer-facing session and conversation history responsibilities; Konam owns platform identity and canonical execution/governance records.

## 13. Sequencing and release boundaries

- **M0:** complete and review this architecture and threat/control mapping (WG-031).
- **M3:** define external-safe workflow metadata and permission-filtered discovery within WG-011/WG-013.
- **M4:** expose the existing execution and async tracking services through a stable API contract within WG-015/WG-016.
- **M6:** implement outbound MCP consumption and, when prioritised, the inbound MCP server adapter within WG-021.
- **M7:** verify delegated identity, scope enforcement, revocation, approvals, audit and adversarial cases within WG-022/WG-023/WG-030.
- **M9:** consider third-party workflow publishing and broader developer ecosystem only after controlled external access is proven.

This does not change the MVP milestone order or authorise implementation ahead of the existing owner-approved batches.

## 14. Open decisions before implementation

1. Select and validate the OAuth/identity provider and supported grants without assuming all clients support the same flows.
2. Decide whether external agent registrations are platform-wide, workspace-owned or both, and define who can approve them.
3. Finalise scope vocabulary and consent/re-consent behaviour.
4. Decide the initial external visibility tiers and whether internal-only workflows can ever be exposed to allow-listed agents.
5. Choose polling first or justify callbacks/streaming for initial release.
6. Define file type, size, malware scanning and retention controls.
7. Validate the MCP SDK, transport and protocol version against target clients.
8. Define API rate, concurrency and execution budgets by client and workspace.
9. Complete threat-model review and legal/privacy review before real customer data or public exposure.

All decisions remain proposed until separately reviewed and accepted. No endpoint, OAuth client, MCP server, secret, database migration or external connection is created by this specification.

## 15. Cross-repository ownership boundary

- Workflow Gateway owns customer conversations and messages, external client registration/presentation at its edge, REST/OpenAPI and inbound MCP adapters, request translation, and user-facing status/result presentation.
- Konam owns authoritative platform identity and workspace permissions, entitlements, capability/model/tool/workflow registries, request routing, execution, approvals, canonical task records, usage and platform audit.
- Gateway-to-Konam calls use the versioned authenticated Konam API. The user identity and Gateway service identity remain distinguishable; Konam validates the delegated context and re-authorises consequential steps.
- Gateway must not call model providers or workflow engines directly for platform tasks, maintain a competing authoritative registry, or bypass Konam policy.
- Gateway owns canonical conversation/message history. Konam retains only the minimum task context or references needed for execution and governance. Define coordinated retention, export and deletion before implementation.

See the [Konam cross-repository integration specification](https://github.com/Konam-Automation-Platform/automation-business-control-centre/blob/main/docs/CROSS_REPOSITORY_INTEGRATION_SPECIFICATION.md).

## 16. Review and status

- Tracking issue: WG-031.
- Document status: proposed planning baseline.
- Scope: architecture and backlog alignment only.
- Implementation status: none.
- Owner approval: architectural direction and documentation work approved; individual production integrations and deployments remain outside this scope.
