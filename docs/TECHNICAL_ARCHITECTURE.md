# Technical Architecture

> Status: planning. Everything below is a **logical architecture**. Technology names are **candidates (proposed)** and **require validation**; none is a final decision and nothing has been provisioned.

## Architectural principle

The platform is **AI-first, workflow-second**. Workflow Gateway is a problem-solving decision and coordination layer, not merely a workflow catalogue or workflow launcher. It can use model reasoning directly, combine models with external tools, invoke reusable workflows, or recommend that a recurring unmet need be considered for future workflow development. It should not create a workflow for every task.

See [ADR-002: AI-first, Workflow-second Architecture](decisions/ADR-002-AI-FIRST-WORKFLOW-SECOND.md).

## Logical layers

### A. Customer interface
Responsive conversational experience, conversation history, optional model selection, capability discovery, integration management, execution timeline, approval requests, user settings and usage information. Recommended candidate: Next.js.

### B. Application backend
Authentication, user and organisation management, conversation management, permissions, policy evaluation and API validation. Recommended candidate: Python FastAPI.

### C. Central orchestration and decision engine
Logical responsibilities (not necessarily separate AI agents or services in the first implementation):

1. **Intent and Context Analyst** — understands the desired outcome, constraints, context, data sensitivity, risk and uncertainty.
2. **Capability Assessor** — evaluates whether direct model reasoning is sufficient, whether external tools are required, and whether an existing workflow is suitable.
3. **Model and Tool Router** — selects compatible models and tools using capability, policy, cost, latency, availability and user preference.
4. **Workflow Discovery** — finds reusable workflows only where structured execution is appropriate.
5. **Plan and Execution Supervisor** — coordinates selected capabilities, validates each step, handles failures and pauses for approvals.
6. **Quality Reviewer** — checks outputs and execution outcomes before returning them.

These are logical responsibilities; avoid creating multiple agents or services unless evidence justifies the complexity.

### D. Capability registries
Three distinct registries: **external AI models**, **plugins and tools**, and **reusable workflows**. Registry presence does not mean a capability is suitable, available, entitled or authorised for a particular request. See [Workflow Registry Specification](WORKFLOW_REGISTRY_SPECIFICATION.md).

### E. Execution adapters
Model-provider adapters; direct third-party APIs; outbound MCP clients; proprietary workflow APIs; GitHub Apps and Actions; internal capability adapters. Each adapter implements a common contract for capability description, validation, execution, event reporting and error mapping. Model abstraction must remain provider- and model-agnostic so new and existing models can be integrated without changing the decision model.

### F. External capability interfaces

External AI assistants, agents and applications may consume approved Workflow Gateway capabilities through two adapters at the application boundary:

- **Versioned REST API + OpenAPI:** documented discovery, execution, status, result and file-reference contracts.
- **Inbound MCP server:** exposes selected Workflow Gateway capabilities as tools to compatible external assistants.

Both adapters call the same application services, capability discovery, authorisation, execution supervisor, approval manager, entitlement checks and audit services. They are not separate orchestration engines or registries. This inbound MCP server direction is distinct from the platform's outbound MCP client integration, where Workflow Gateway consumes tools hosted by third parties.

External callers require authenticated client identity and narrowly scoped, revocable authorisation. The backend remains authoritative for workspace context, capability availability, entitlement, data scope, approvals and rate limits. See [External AI Agent Integration Specification](EXTERNAL_AI_AGENT_INTEGRATION_SPECIFICATION.md) and WG-031.

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

1. User submits a goal through the conversational interface.
2. Backend authenticates the principal, authorises access and loads permitted conversation context.
3. Intent and Context Analyst structures the goal, constraints, sensitivity, risk and uncertainty; it asks a clarifying question when needed.
4. Capability Assessor considers, in order of suitability rather than a hard-coded workflow preference:
   - direct model reasoning without tools;
   - model reasoning with one or more authorised external tools;
   - an existing reusable workflow, if its repeatability and controls fit the task;
   - a multi-capability plan combining models, tools and/or workflows.
5. If no suitable route exists, the platform may explain the limitation or recommend a separate workflow-development assessment when the need appears recurring and valuable. It must not silently create a workflow or expand scope.
6. Policy and authorisation checks apply to the complete plan and every consequential step. Human approval is requested where required; execution pauses until a recorded decision.
7. Execution Supervisor invokes permitted adapters and tracks progress, failures, cancellation and approvals.
8. Quality Reviewer checks the result; retry or alternative routing only within policy and permissions.
9. Response, routing rationale, usage and audit events are recorded.

## Decision dimensions

Routing is based on task fit, model capability, tool requirements, workflow suitability, confidence, data sensitivity, risk, cost, latency, availability, entitlement, user/organisation policy and approval requirements. No route may bypass permissions or human approval.

## Database entities (initial proposal; WG-004 finalises)

organisations, users, memberships, conversations, messages, models, model_capabilities, workflows, workflow_versions, plugins, tools, routing_policies, routing_decisions, plans, plan_steps, executions, execution_events, approvals, credentials (encrypted payload or secret reference only), usage_events, audit_logs, entitlements.

Every tenant-owned entity carries an organisation identifier. Row-level security is a proposed enforcement layer requiring validation. These are conceptual entities, not an approved schema or migration plan.

## Observability

Structured logs without sensitive content, execution events, metrics and traces; audit logs are append-only. Record the chosen route and concise rationale, the capabilities invoked, policy/approval decisions, and outcome. See [Security and Governance](SECURITY_AND_GOVERNANCE.md).

## Constraints

- The model layer must remain provider- and model-agnostic.
- A model's ability to propose an action does not grant permission to perform it.
- Workflow lifecycle and availability rules are enforced server-side.
- No unfinished workflow is exposed as a commercial service.
- Provider, workflow and tool credentials are never exposed to the client.
- External interfaces reuse the same backend security and execution services; they do not bypass approvals or tenant isolation.
- Capability discovery is permission-filtered; successful authentication alone does not grant access.

Related: [Product Requirements](PRODUCT_REQUIREMENTS.md), [ADR-001](decisions/ADR-001-OVERALL-PLATFORM-DIRECTION.md), [ADR-002](decisions/ADR-002-AI-FIRST-WORKFLOW-SECOND.md), [Implementation Roadmap](IMPLEMENTATION_ROADMAP.md).
