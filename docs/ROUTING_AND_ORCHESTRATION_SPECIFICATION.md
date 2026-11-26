# Routing and Orchestration Specification

> Status: proposed baseline for WG-010 and WG-014. Approved direction: routing is configurable with a proprietary workflow-first default.

## Pipeline

Intent analysis, workflow discovery, routing, execution supervision, quality review. In the first implementation these may be modules of one service rather than separate AI agents.

## Intent analysis

Output (structured): goal, task type, entities, constraints, data sensitivity, risk level, confidence and whether approval is likely needed. If confidence is low the platform asks a clarifying question instead of guessing. User input is untrusted; the intent never grants permissions.

## Workflow discovery

Filter by lifecycle, availability and entitlement; then rank by capability match and confidence (keyword and tag matching first; embeddings are proposed and need validation). Each match includes an explanation. An empty result is valid and triggers fallback routing.

## Routing policy

Default policy: **proprietary workflow first**. If a qualifying workflow exists with sufficient confidence, use it. Otherwise consider models, plugins and tools, internal agents or a combination. The platform does not force a workflow when another solution is genuinely more suitable.

Policy modes (configurable per organisation and user within platform guardrails): workflow-first (default), model-first, manual selection. Policies can never bypass entitlements or approval gates.

Decision factors: confidence, data sensitivity, cost, latency, entitlement, customer policy, approval requirements, provider and tool availability.

## Strategies

- Single workflow.
- Single model response.
- Model plus tools.
- Multi-step plan combining workflows, models and tools.
- Clarify (ask the user).
- Decline (unsafe, unavailable or not permitted).

Every decision is stored with its reasons (routing_decisions) and can be explained to the user and administrators.

## Model routing

Models are described in the model registry (capabilities, context size, cost class, availability, platform-managed or BYOK). Selection considers task needs, user selection, policy, cost and availability, with provider fallback and circuit breaking. BYOK keys are used only for the owning tenant.

## Plugin and MCP integrations

Plugins and external MCP servers are registered with tool schemas and permission scopes, enabled per tenant, and invoked through a common outbound adapter with timeouts and output limits. Tool output is untrusted. Write actions require approval according to policy.

**Integration direction distinction:** outbound MCP means Workflow Gateway connects to third-party MCP servers and consumes their tools. Inbound MCP means external assistants connect to a Workflow Gateway MCP server and consume our approved workflows. Inbound exposure is specified separately in [External AI Agent Integration Specification](EXTERNAL_AI_AGENT_INTEGRATION_SPECIFICATION.md) and tracked by WG-031.

REST/OpenAPI and inbound MCP adapters must call the same discovery, authorisation, execution, approval, status and result services used by the internal chat. They must not create parallel routing or execution systems. External discovery is filtered by the caller's current grants, workflow lifecycle, availability and entitlement. Every invocation is authorised again at execution time.

## Execution supervision

Tracks state (queued, running, awaiting approval, succeeded, failed, cancelled, timed out), handles retries with backoff, cancellation, timeouts and duplicate callbacks, and emits events for the execution timeline. A state-machine library such as LangGraph is a candidate, not a requirement.

## Quality review

Rule-based checks first (schema, emptiness, error signatures); optional model-based review is proposed. Failed review triggers retry or alternative routing. Outcomes and user feedback are recorded.

## Human approval

Where policy requires, execution enters awaiting approval and cannot continue until a recorded decision. See [Security and Governance](SECURITY_AND_GOVERNANCE.md).

## Evaluation

Maintain evaluation sets for intent classification, workflow discovery relevance and routing decisions, plus adversarial prompt-injection cases (WG-005, WG-010, WG-030).

Related: [Technical Architecture](TECHNICAL_ARCHITECTURE.md), [Workflow Registry Specification](WORKFLOW_REGISTRY_SPECIFICATION.md).
