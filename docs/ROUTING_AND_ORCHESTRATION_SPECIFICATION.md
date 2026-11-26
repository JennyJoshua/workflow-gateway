# Routing and Orchestration Specification

> Status: proposed baseline for WG-010 and WG-014. Foundational direction: AI-first, workflow-second. Detailed routing policy remains subject to review and validation.

## Purpose

Workflow Gateway receives a user's desired outcome and selects the simplest safe, suitable and authorised way to achieve it. It is not a workflow launcher that must find a workflow for every request. Direct AI reasoning is a first-class route; tools and workflows are selected when they add necessary capability, data access, repeatability or operational control.

## Decision pipeline

Intent and context analysis → capability assessment → route/plan selection → policy and approval evaluation → execution supervision → quality review. These are logical responsibilities and may be modules of one service in the first implementation.

## Intent and context analysis

Output (structured): goal, task type, entities, constraints, available context, data sensitivity, risk level, confidence, expected repeatability, required freshness, and whether approval is likely needed. If confidence is low or essential information is missing, ask a clarifying question instead of guessing. User input is untrusted; intent never grants permissions.

## Capability assessment

Evaluate available capabilities in context:

1. **Direct AI:** Can an authorised model solve the task through reasoning, conversation, analysis, drafting or coding without external actions?
2. **AI plus tools:** Does the task require current external information, account data, computation, or an action in another system?
3. **Existing workflow:** Is there a registered workflow whose purpose, contract, reliability, repeatability and governance fit this request?
4. **Combined plan:** Would a sequence of model, tool and workflow capabilities provide a better result, with each step independently authorised?
5. **Unmet need:** If none is suitable, can the request be answered safely with a limitation, clarification or alternative? If the need is recurring and valuable, recommend a separate workflow-development assessment.

Do not create or publish a workflow automatically because discovery returned no match. One-off tasks and general reasoning should normally remain AI tasks unless structured execution is materially beneficial.

## Routing policy

The default is **AI-first, workflow-second**, not workflow-first. Select the simplest safe and suitable route based on task fit. A workflow is preferred only when its repeatability, integration, business rules, consistency or governance provide a real advantage for the current request.

Policy modes may be configurable per organisation and user within platform guardrails: balanced/automatic (default), model-preferred, workflow-preferred, or manual selection. A preference influences selection but cannot bypass availability, entitlements, permissions, safety controls or approval gates. Exact policy modes require validation before implementation.

Decision factors: capability fit, confidence, data sensitivity, risk, task repeatability, required integrations, model/tool/workflow availability, cost, latency, freshness, entitlement, customer policy and approval requirements.

## Strategies

- Direct model response.
- Model plus one or more external tools.
- Existing reusable workflow.
- Model/tool/workflow multi-step plan.
- Clarify (ask the user).
- Explain limitation or decline (unsafe, unavailable or not permitted).
- Recommend workflow-development assessment for a recurring unmet need (separate governance and owner decision; not an execution route).

Every routing decision records the candidate routes considered, selected route, concise reasons, relevant policy and approval outcomes, and result. Sensitive prompt or payload content should not be copied into audit records unnecessarily.

## Model routing

Models are described through a provider-neutral abstraction and capability metadata (supported task classes, context limits, modalities, tool-use support, availability and cost class). Selection considers task needs, user preference, policy, cost, privacy and availability, with provider fallback and circuit breaking. New and existing models should be integrable without redesigning the Gateway. BYOK keys are used only for the owning tenant. No model is assumed to be universally best.

## Tool and MCP integrations

Plugins and external MCP servers are registered with tool schemas and permission scopes, enabled per tenant, and invoked through a common outbound adapter with timeouts and output limits. Tool output is untrusted. Write actions require approval according to policy.

**Integration direction distinction:** outbound MCP means Workflow Gateway connects to third-party MCP servers and consumes their tools. Inbound MCP means external assistants connect to a Workflow Gateway MCP server and consume our approved capabilities. Inbound exposure is specified separately in [External AI Agent Integration Specification](EXTERNAL_AI_AGENT_INTEGRATION_SPECIFICATION.md) and tracked by WG-031.

REST/OpenAPI and inbound MCP adapters must call the same discovery, authorisation, execution, approval, status and result services used by the internal chat. They must not create parallel routing or execution systems. External discovery is filtered by the caller's current grants, capability availability and entitlement. Every invocation is authorised again at execution time.

## Workflow discovery and suitability

Workflow discovery is one part of capability assessment, not the mandatory first step. Filter by lifecycle, availability, entitlement, external visibility and permission before ranking. Rank by capability fit, contract compatibility, reliability and suitability for the task. An empty result is valid; it does not itself imply that a new workflow should be created.

Workflow candidates should generally represent recurring processes with measurable value or a need for deterministic steps, triggers, integrations, business rules, approvals, monitoring or audit. The workflow registry must not become a catalogue of one-off prompts or thin wrappers around general model reasoning.

## New workflow recommendation

When a request reveals a recurring unmet need, the Gateway may capture evidence and recommend a workflow-development assessment. Assessment should consider recurrence, business value, repeatability, process stability, integration requirements, risk, maintenance burden and whether an existing model/tool capability is sufficient. Any workflow development, scope expansion, publication or commercialisation requires the applicable owner approvals and separate backlog planning. No automatic generation, deployment or publication is authorised by this specification.

## Execution supervision

Tracks state (queued, running, awaiting approval, succeeded, failed, cancelled, timed out), handles retries with backoff, cancellation, timeouts and duplicate callbacks, and emits events for the execution timeline. Each step in a multi-capability plan is independently authorised and its output treated as untrusted input to later steps. A state-machine library such as LangGraph is a candidate, not a requirement.

## Human approval

Approval is an independent governance decision, not a routing strategy. Where policy requires, execution enters awaiting approval and cannot continue until a recorded human decision. Approval applies to the specific action, scope and context presented; changes to those invalidate the approval. Routing, model output, tool output, retries or external-client confirmation cannot bypass approval. See [Security and Governance](SECURITY_AND_GOVERNANCE.md).

## Quality review

Rule-based checks first (schema, emptiness, error signatures, policy compliance); optional model-based review is proposed. Failed review triggers retry or alternative routing only where policy permits. Outcomes and user feedback are recorded.

## Evaluation

Maintain evaluation sets for intent classification, direct-AI-versus-tool decisions, workflow suitability and discovery relevance, routing accuracy, cost/latency trade-offs, approval decisions and adversarial prompt-injection cases (WG-005, WG-010, WG-030).

Related: [Technical Architecture](TECHNICAL_ARCHITECTURE.md), [Workflow Registry Specification](WORKFLOW_REGISTRY_SPECIFICATION.md), [ADR-002](decisions/ADR-002-AI-FIRST-WORKFLOW-SECOND.md).
