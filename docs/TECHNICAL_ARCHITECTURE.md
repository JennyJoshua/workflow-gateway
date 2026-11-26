# Technical Architecture

> Status: planning. Everything below is a **logical architecture**. Technology names are **candidates (proposed)** and **require validation**; none is a final decision and nothing has been provisioned.

## Logical layers

### A. Customer interface
Responsive web chat, conversation history, model selector, workflow discovery, plugin and integration management, execution timeline, user settings and usage information. Recommended candidate: Next.js.

### B. Application backend
Authentication, user and organisation management, conversation management, permissions and API validation. Recommended candidate: Python FastAPI.

### C. Central orchestration engine
Five logical responsibilities (not necessarily separate AI agents in the first implementation; avoid unnecessary complexity):

1. **Intent Analyst** — converts a request into a structured intent.
2. **Workflow Discovery Agent** — finds matching workflows in the registry.
3. **Model and Tool Router** — chooses the execution strategy, models and tools.
4. **Execution Supervisor** — runs and tracks execution, handles failures and approvals.
5. **Quality Reviewer** — checks results before they reach the customer.

See [Routing and Orchestration Specification](ROUTING_AND_ORCHESTRATION_SPECIFICATION.md).

### D. Registries
Three separate registries: **proprietary workflows**, **external models**, **plugins and tools**. See [Workflow Registry Specification](WORKFLOW_REGISTRY_SPECIFICATION.md).

### E. Execution adapters
Proprietary workflow APIs; GitHub Apps and GitHub Actions; external AI model providers; MCP servers; direct third-party APIs; internal agent capabilities. Each adapter implements a common interface (validate input, execute, report events, map errors).

### F. Infrastructure candidates (evaluate, do not provision)

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

1. User sends a message in the chat.
2. Backend authenticates, authorises and loads conversation context.
3. Intent Analyst produces a structured intent (clarifies if uncertain).
4. Workflow Discovery Agent searches the workflow registry within the tenant's entitlements.
5. Router applies the routing policy (workflow-first by default) and records the decision.
6. If approval is required, execution pauses until a human decides.
7. Execution Supervisor invokes the chosen adapters; events stream to the timeline.
8. Quality Reviewer checks the result; retry or fallback if needed.
9. Response is returned; usage and audit events are recorded.

## Database entities (initial proposal; WG-004 finalises)

organisations, users, memberships, conversations, messages, models, workflows, workflow_versions, plugins, routing_policies, routing_decisions, executions, execution_events, approvals, credentials (encrypted payload or secret reference only), usage_events, audit_logs, entitlements.

Every tenant-owned entity carries an organisation identifier. Row-level security is a proposed enforcement layer requiring validation.

## Observability

Structured logs without sensitive content, execution events, metrics and traces; audit logs are append-only. See [Security and Governance](SECURITY_AND_GOVERNANCE.md).

## Architecture decisions and review proposals

The ADR register is maintained in [docs/decisions/README.md](decisions/README.md). ADR-001 records the four owner-approved product directions. ADR-002 through ADR-008 are **proposed technology assessments only**; none is an accepted technology decision. The independent review refinements are recorded separately in [Architecture Review Recommendations](ARCHITECTURE_REVIEW_RECOMMENDATIONS.md) and require owner review before they become binding requirements.

## Constraints

- Workflow lifecycle and availability rules are enforced server-side.
- No unfinished workflow is exposed as a commercial service.
- Provider, workflow and tool credentials are never exposed to the client.

Related: [Product Requirements](PRODUCT_REQUIREMENTS.md), [ADR-001](decisions/ADR-001-OVERALL-PLATFORM-DIRECTION.md), [Implementation Roadmap](IMPLEMENTATION_ROADMAP.md).
