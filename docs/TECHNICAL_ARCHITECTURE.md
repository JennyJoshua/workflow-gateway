# Technical Architecture

> Status: planning. Everything below is a **logical architecture**. Technology names are **candidates (proposed)** and **require validation**; none is a final decision and nothing has been provisioned.

## Architectural principle

The ecosystem is **AI-first, workflow-second**. The Workflow Gateway application is the customer-facing conversational product and external access boundary. It submits user goals to Konam; the **Konam Capability Router**, an internal logical capability in the Konam Automation Platform, performs authoritative problem assessment and routing. Konam can use model reasoning directly, combine models with authorised tools, invoke reusable workflows, or recommend assessment of a recurring unmet need. It must not create a workflow for every task.

See [ADR-002: AI-first, Workflow-second Architecture](decisions/ADR-002-AI-FIRST-WORKFLOW-SECOND.md).

## Cross-repository responsibility boundary

The repositories are independently versioned and deployable. They communicate through a versioned, authenticated Konam platform API; they do not share a database or duplicate authoritative services.

- **Workflow Gateway owns:** customer-facing web/chat experience, canonical customer conversation and message history, UI/session/presentation state, external REST/OpenAPI and inbound MCP adapters, and translation between external contracts and the Konam API.
- **Konam owns:** platform identity and workspace authority, membership, roles and entitlements, policy, the Konam Capability Router, model/provider and tool/workflow registries, execution supervision, workflow execution, approvals, canonical task/execution state, usage and platform audit.
- **Gateway is not a second platform control plane:** it must not independently make authoritative routing decisions, maintain duplicate capability/workflow registries, execute workflows directly, or override Konam permissions and approvals. It may perform edge authentication and validation as defence in depth.

Gateway-to-Konam calls use OIDC-backed user identity and short-lived, audience-restricted delegated tokens or an equivalent validated token-exchange pattern. Keep the represented user and Gateway service identity distinct. The concrete provider and flow remain open pending technical validation. See the [Konam cross-repository integration specification](https://github.com/Konam-Automation-Platform/automation-business-control-centre/blob/main/docs/CROSS_REPOSITORY_INTEGRATION_SPECIFICATION.md).

## Logical layers

### A. Customer interface
Responsive conversational experience, conversation history, optional model selection, capability discovery, integration management, execution timeline, approval requests, user settings and usage information. Recommended candidate: Next.js.

### B. Gateway application backend
Customer session handling, canonical conversation/message history, UI-specific state, edge request validation and the secure Konam API client. Konam remains authoritative for platform identity, workspace membership, permissions, entitlements and execution policy. Recommended candidate: Python FastAPI.

### C. Konam-owned capability routing and execution (integration dependency)
These are Konam Automation Platform responsibilities, not a second orchestration engine implemented in this repository. Gateway submits the user's goal and permitted context to Konam and presents the resulting status and output. The internal Konam Capability Router may comprise logical modules rather than separate services:

1. **Intent and Context Analyst** — understands the desired outcome, constraints, context, data sensitivity, risk and uncertainty.
2. **Capability Assessor** — evaluates whether direct model reasoning is sufficient, whether external tools are required, and whether an existing workflow is suitable.
3. **Model and Tool Router** — selects compatible models and tools using capability, policy, cost, latency, availability and user preference.
4. **Workflow Discovery** — finds reusable workflows only where structured execution is appropriate.
5. **Plan and Execution Supervisor** — coordinates selected capabilities, validates each step, handles failures and pauses for approvals.
6. **Quality Reviewer** — checks outputs and execution outcomes before returning them.

These are logical responsibilities; avoid creating multiple agents or services unless evidence justifies the complexity.

### D. Capability registries
Konam owns the authoritative registries for **AI models/providers**, **plugins and tools**, and **reusable workflows**. Gateway may display permission-filtered capability summaries returned by Konam, but must not create a competing authoritative registry. Registry presence does not mean a capability is suitable, available, entitled or authorised for a particular request.

### E. Execution adapters
Konam owns and governs model-provider adapters, authorised tool/API integrations, outbound MCP clients, workflow-engine adapters and internal capability adapters. Gateway does not invoke workflow engines or provider credentials directly; it calls the Konam API. Model abstraction remains provider- and model-agnostic.

### F. External capability interfaces

External AI assistants, agents and applications may consume approved Workflow Gateway capabilities through two adapters at the application boundary:

- **Versioned REST API + OpenAPI:** documented discovery, execution, status, result and file-reference contracts.
- **Inbound MCP server:** exposes selected Workflow Gateway capabilities as tools to compatible external assistants.

Both adapters authenticate and validate external requests at the Gateway edge, then call the versioned Konam API. Konam performs authoritative capability discovery, platform authorisation, routing, execution supervision, approval, entitlement enforcement and platform audit. Gateway must not host parallel versions of those services. This inbound MCP server direction is distinct from outbound MCP integrations used by Konam to consume third-party tools.

External callers require authenticated client identity and narrowly scoped, revocable authorisation. Gateway owns the external interface contract; Konam remains authoritative for workspace context, capability availability, entitlement, data scope, approvals and platform rate limits. See [External AI Agent Integration Specification](EXTERNAL_AI_AGENT_INTEGRATION_SPECIFICATION.md) and WG-031.

### G. Infrastructure candidates (evaluate, do not provision)

| Candidate | Possible role | Status |
|---|---|---|
| Supabase | Auth, Postgres, row-level security, storage | Proposed; requires validation (free-tier limits, RLS fit, exit cost) |
| Render | Backend and worker hosting | Proposed; requires validation (free-tier behaviour such as sleeping services) |
| FastAPI | Application backend | Proposed |
| Next.js | Customer interface | Proposed |
| LiteLLM | Model gateway across providers | Proposed; compare with a thin in-house abstraction |
| LangGraph | Orchestration state machines | Proposed; may be unnecessary in the first implementation |
| MCP | Standard for tool and plugin integration | Proposed |

Each will be assessed in an ADR (WG-002). Free-first development is preferred; no paid service is subscribed to or upgraded without owner approval.

## Request flow (logical)

1. User submits a goal through the Gateway conversational interface.
2. Gateway authenticates the user at its edge, loads the permitted conversation context and submits a scoped request to the Konam API.
3. Konam validates the principal, workspace membership, entitlements, scopes and policy. Konam's Intent and Context Analyst structures the goal, constraints, sensitivity, risk and uncertainty; it asks for clarification when essential information is missing.
4. The Konam Capability Router considers, in order of suitability rather than a hard-coded workflow preference:
   - direct model reasoning without tools;
   - model reasoning with one or more authorised external tools;
   - an existing reusable workflow, if its repeatability and controls fit the task;
   - a multi-capability plan combining models, tools and/or workflows.
5. If no suitable route exists, Konam may explain the limitation or recommend a separate workflow-development assessment when the need appears recurring and valuable. It must not silently create a workflow or expand scope.
6. Konam policy and authorisation checks apply to the complete plan and every consequential step. Human approval is requested where required; execution pauses until a recorded decision.
7. Konam's Execution Supervisor invokes permitted adapters and tracks authoritative progress, failures, cancellation and approvals.
8. Konam's quality controls check the result; retry or alternative routing occurs only within policy and permissions.
9. Konam records routing rationale, usage and platform audit events. Gateway presents the result and task status.

## Decision dimensions

Konam's routing policy considers task fit, model capability, tool requirements, workflow suitability, confidence, data sensitivity, risk, cost, latency, availability, entitlement, tenant policy and approval requirements. No route may bypass permissions or human approval.

## Database entities (proposed baseline; WG-004 specification)

Gateway-owned customer experience data may include conversations and messages. Konam-owned platform data includes organisations/workspaces, users and memberships, models, model_capabilities, workflows, workflow_versions, plugins, tools, routing_policies, routing_decisions, plans, plan_steps, executions, execution_events, approvals, credential references, usage_events, audit_logs and entitlements. Do not duplicate Konam's canonical platform records in Gateway.

The detailed logical schema, relationships, constraints, indexes, workspace tenancy, RLS coverage, retention, deletion, export and recovery requirements are defined in [Database Specification](DATABASE_SPECIFICATION.md). The universal tenant key is `workspace_id`, covering both personal and organisation workspaces; `organisation_id` is additionally used where an organisation relationship applies. This resolves the limitation of using an organisation identifier as the tenant key for personal workspaces.

The database specification is a planning artifact only. PostgreSQL is the logical reference; Supabase remains a proposed candidate. No database has been provisioned and no migrations have been created or run.

## Observability

Gateway records edge request handling, conversation operations and UI failures with correlation IDs. Konam records authoritative routing, execution, policy/approval, usage and platform audit events. Logs must minimise sensitive content. See [Security and Governance](SECURITY_AND_GOVERNANCE.md).

## Security threat model

The design-time threat analysis and trust-boundary data-flow diagram are maintained in [Security Threat Model](SECURITY_THREAT_MODEL.md). It identifies security controls that must be implemented and verified; it does not claim that the logical architecture is already secure in operation.

## Architecture decisions and review proposals

The ADR register is maintained in [docs/decisions/README.md](decisions/README.md). ADR-001 records the four owner-approved product directions. ADR-003 through ADR-012 are **proposed technology and cross-cutting architecture assessments only**; none is an accepted technology decision. The independent review refinements are recorded separately in [Architecture Review Recommendations](ARCHITECTURE_REVIEW_RECOMMENDATIONS.md) and require owner review before they become binding requirements.

## Constraints

- The model layer must remain provider- and model-agnostic.
- A model's ability to propose an action does not grant permission to perform it.
- Workflow lifecycle, platform identity, entitlement, routing and availability rules are authoritative in Konam and enforced server-side.
- No unfinished workflow is exposed as a commercial service.
- Provider, workflow and tool credentials are never exposed to the client.
- Gateway external interfaces call the Konam API and cannot bypass Konam approvals, authorisation or tenant isolation.
- Capability discovery is permission-filtered; successful authentication alone does not grant access.

Related: [Product Requirements](PRODUCT_REQUIREMENTS.md), [ADR-001](decisions/ADR-001-OVERALL-PLATFORM-DIRECTION.md), [ADR-002](decisions/ADR-002-AI-FIRST-WORKFLOW-SECOND.md), [Implementation Roadmap](IMPLEMENTATION_ROADMAP.md).
