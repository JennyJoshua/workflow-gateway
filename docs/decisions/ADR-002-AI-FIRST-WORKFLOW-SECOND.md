# ADR-002: AI-first, Workflow-second Architecture

- **Status:** Accepted (owner-approved strategic principle; documentation change pending merge)
- **Date:** 26 November 2026
- **Deciders:** Owner; ChatGPT as Chief AI Architect and Independent Reviewer
- **Scope:** Foundational product and architecture principle; planning/specification only

## Context

A workflow catalogue can easily become a collection of predefined automations for every conceivable problem. That would duplicate capabilities already available in connected AI models, increase development and maintenance costs, and force users and operators to manage unnecessary workflows.

AI models can already solve many problems dynamically through reasoning, conversation, document analysis, coding and tool integrations. Reusable workflows remain valuable for recurring tasks where deterministic execution, triggers, integrations, business rules, approvals, monitoring and auditability provide material value.

The platform therefore needs to be designed as an AI-powered problem-solving and workflow execution platform, rather than merely a workflow library.

## Decision

Adopt **AI-first, workflow-second** as a foundational architectural principle.

The platform has three complementary capabilities:

1. **AI Intelligence:** connected AI models solve suitable problems dynamically without requiring a predefined workflow.
2. **Workflow Automation:** reusable workflows handle recurring tasks that benefit from consistent execution, triggers, integrations, business rules, approvals and monitoring.
3. **Konam Capability Router:** Konam's internal, model-agnostic decision and coordination capability assesses the request and selects direct AI, authorised external tools, an existing workflow, or a governed combination. The separate Workflow Gateway application provides the customer-facing conversational experience and external REST/OpenAPI/MCP access boundary.

For each request, the Konam Capability Router should progressively assess:

- Can AI solve this directly?
- Does the task require external tools, data or integrations?
- Is an existing workflow suitable?
- If no suitable capability exists, is the unmet need recurring and valuable enough to recommend a separate workflow-development assessment?
- Does execution require human approval?

The default is the simplest safe and suitable capability—not a workflow-first preference. A workflow is appropriate when its repeatability, operational controls, integration requirements or consistency provide a meaningful advantage.

## Architectural requirements

- **Model agnosticism:** provider- and model-neutral abstractions; new and existing models can be integrated without redesigning routing.
- **First-class direct AI:** direct model reasoning is a supported route, not merely a fallback after workflow discovery.
- **Capability-based routing:** decisions consider task fit, confidence, context, sensitivity, risk, freshness, cost, latency, availability, entitlement and user/organisation policy.
- **Tool use:** use external tools when the request requires current information, external data, computation or actions beyond model-only reasoning.
- **Workflow suitability:** reserve workflows for recurring or operationally governed tasks; do not populate the registry with one-off tasks or thin wrappers around general reasoning.
- **No automatic workflow proliferation:** an unmatched request does not automatically trigger workflow creation. Konam may recommend a separate workflow-development assessment when a recurring need is evidenced; Gateway may present that recommendation.
- **Human approval:** Konam owns approval policy and enforcement for high-risk or policy-controlled actions. AI reasoning, Gateway UI state or client confirmation cannot grant permission.
- **Explainability and audit:** Konam records the authoritative selected route, concise rationale, capability steps and approval outcomes without unnecessarily retaining sensitive content; Gateway presents safe explanations and propagates correlation IDs.
- **Shared governance:** Gateway provides the customer and external-client access boundary; Konam provides authoritative platform identity, authorisation, tenant, entitlement, execution, audit and approval controls.

## Consequences

### Positive

- Users can receive value from AI even when no proprietary workflow exists.
- The workflow catalogue remains focused on reusable, valuable automations.
- New models and tools can expand problem-solving capability without requiring a new workflow for each capability.
- Workflow investment can be prioritised by recurrence, measurable value and governance needs.
- Routing decisions become an explicit product capability and a source of explainability.

### Trade-offs and risks

- Capability assessment and routing are more sophisticated than a simple workflow-first lookup.
- Model quality, latency, cost and provider behaviour vary; evaluation and fallback policies are needed.
- Direct AI responses may be less deterministic than governed workflows; user expectations and approval controls must reflect this.
- Multi-capability plans introduce orchestration, prompt-injection and data-boundary risks.
- The platform must avoid over-engineering a multi-agent system before evidence requires it.

## Impact on previous decisions

This ADR **supersedes the workflow-first routing default in ADR-001**. ADR-001's other decisions remain in force: standalone and platform-interface roles, platform-managed models plus BYOK, and progressive development.

The existing milestone sequence and recommended first implementation batch remain unchanged. This ADR does not authorise application implementation, infrastructure provisioning, paid services, integrations, deployment, database changes, automatic workflow generation, or expansion of the currently approved scope.

## Alternatives considered

- **Workflow-first:** rejected because it risks duplicating general AI capabilities and incentivising unnecessary workflow creation.
- **AI-only:** rejected because recurring business processes still need reliable triggers, integrations, deterministic rules, approvals and monitoring.
- **User manually selects every capability:** rejected as the default because the Gateway's value is to make the decision while keeping it explainable and controllable.

## Review and governance

This ADR records the strategic direction explicitly approved by the owner. Its documentation changes remain subject to the repository's normal review and merge controls. Any material change requires a new ADR and owner approval under project change control.
