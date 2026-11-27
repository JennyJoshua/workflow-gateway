# Product Requirements

> Status: planning. Requirements are the baseline for the backlog; items marked *proposed* require validation before they become commitments.

## Personas

- **Business user** — states goals in plain language, wants results and clear progress.
- **Organisation administrator** — manages members, integrations, keys, routing preferences and usage through Gateway; Konam enforces and stores authoritative platform permissions and entitlements.
- **Platform operator (owner)** — manages capability availability, workflow lifecycle, platform policy and approvals in Konam; Gateway provides the customer-facing interface where appropriate.
- **Workflow developer** (later) — publishes reusable workflows to the platform.

## Foundational product principle

The ecosystem is **AI-first, workflow-second**. Do not create a predefined workflow for every problem. AI models should solve suitable tasks dynamically; reusable workflows are used when recurring execution benefits from repeatability and governance. Konam's Capability Router makes the authoritative route decision and Konam enforces platform policy, permissions, execution and approvals. Workflow Gateway provides the conversational experience and external access boundary. See ADR-002 and the [cross-repository integration specification](https://github.com/Konam-Automation-Platform/automation-business-control-centre/blob/main/docs/CROSS_REPOSITORY_INTEGRATION_SPECIFICATION.md).

## Cross-repository requirement ownership

These are product-level requirements for the combined Gateway/Konam experience, not a declaration that every capability is implemented in the Gateway repository. Gateway owns conversations, presentation and external REST/OpenAPI/MCP interfaces. Konam owns authoritative identity, tenancy, capability routing, registries, execution, approvals, usage and platform audit. The Gateway backlog must be reconciled against this split before implementation; no duplicate platform services should be built.

## Functional requirements

| ID | Requirement | Issues |
|---|---|---|
| FR-01 | Responsive web chat with streaming responses | WG-006 |
| FR-02 | Account registration, sign-in and session management | WG-007 |
| FR-03 | Persisted conversations and messages with history | WG-008 |
| FR-04 | Model-independent provider abstraction and model selector | WG-009, WG-019 |
| FR-05 | Intent analysis producing structured intents and clarifying questions | WG-010 |
| FR-06 | Explainable, configurable routing across direct AI, tools and reusable workflows; no mandatory workflow preference (authoritative routing supplied by Konam) | WG-014, WG-025 |
| FR-07 | Customer-facing workflow catalogue backed by Konam's authoritative registry and lifecycle | WG-011, WG-012 |
| FR-08 | Permission-filtered capability discovery through Konam's authoritative platform services | WG-013 |
| FR-09 | Submit authorised workflow/task requests to Konam and display its authoritative sync/async status | WG-015, WG-016 |
| FR-10 | Execution timeline visible in chat | WG-017 |
| FR-11 | Result quality review | WG-018 |
| FR-12 | Customer-supplied API keys (BYOK) | WG-020 |
| FR-13 | Plugin and MCP integration registry | WG-021 |
| FR-14 | Present and submit approval requests enforced and recorded by Konam | WG-022 |
| FR-15 | Tenant-safe Gateway sessions and external access, integrated with Konam's authoritative tenancy and permissions | WG-023 |
| FR-16 | Usage metering and customer usage information | WG-024 |
| FR-17 | Customer settings and routing controls | WG-025 |
| FR-18 | Subscriptions and entitlements | WG-026 |
| FR-19 | Organisation and team accounts | WG-027 |
| FR-20 | Workflow developer publishing and marketplace | WG-028, WG-029 |
| FR-21 | Audit log of security-relevant and routing events | WG-003, WG-004, WG-023 |
| FR-22 | Authorised external AI assistants and applications can discover, invoke, monitor and retrieve results from approved capabilities through versioned REST/OpenAPI and MCP interfaces | WG-031; implementation mapped to WG-007, WG-011, WG-013, WG-015, WG-016, WG-021, WG-022, WG-023 |
| FR-23 | Gateway submits each goal to Konam; the Konam Capability Router assesses direct AI, authorised tools, existing workflows or a multi-capability plan and records the authoritative explainable routing decision | WG-010, WG-014 |
| FR-24 | When no suitable capability exists, Konam may recommend a separate workflow-development assessment for recurring needs, which Gateway may present to the user; it must not automatically create or publish a workflow | Planning requirement; backlog mapping to be reviewed during normal issue refinement |

## Non-functional requirements

| ID | Requirement |
|---|---|
| NFR-01 | Security: default-deny authorisation, encrypted secrets, no secrets in code or logs |
| NFR-02 | Privacy: data minimisation, tenant-scoped storage, defined retention and deletion |
| NFR-03 | Reliability: durable execution state that survives restarts; idempotent callbacks |
| NFR-04 | Performance (*proposed targets, validate in WG-030*): first streamed token within a few seconds for model replies; UI interactions feel immediate; long workflows run asynchronously with visible progress |
| NFR-05 | Availability (*proposed*): a pilot-appropriate target set by the owner before pilot |
| NFR-06 | Observability: structured logs, metrics, traces and execution events without sensitive content |
| NFR-07 | Accessibility: keyboard and screen-reader support, mobile-first layout |
| NFR-08 | Extensibility: new models, workflows and tools added through registries and adapters |
| NFR-09 | Cost control: free-first development; usage metering to track costs |
| NFR-10 | Maintainability: documented standards, tests and ADRs |
| NFR-11 | Interoperability: versioned, documented external interfaces with stable schemas and backward-compatibility rules |
| NFR-12 | Model neutrality: routing and capability contracts must not depend on one model provider, model family or orchestration vendor |

## User experience

- One conversational interface; users state the problem without needing to choose a workflow.
- The platform can answer directly with AI, use tools, invoke workflows or combine capabilities.
- Optional model selection for users who want control; capability discovery for browsing what is available to them.
- Plugin and integration management in settings.
- Execution timeline explains the selected route, steps, approvals and results in plain language.
- Clear messaging when something is unavailable, requires clarification or approval, or has failed.
- Settings and usage page showing routing preferences, keys (masked) and usage.

## Out of scope for this phase

Any application code or deployment; real integrations with proprietary workflows; billing provider selection; public launch; automatic generation or publication of new workflows.

## Open questions (require owner input or validation)

- Initial customer segment and pilot customers.
- Pricing model and plan structure.
- Which first workflows are suitable for internal availability, based on recurring needs and measurable value.
- Licensing approach (see [Contribution and Change Control](../project-management/CONTRIBUTION_AND_CHANGE_CONTROL.md)).

See also: [Technical Architecture](TECHNICAL_ARCHITECTURE.md), [Routing and Orchestration Specification](ROUTING_AND_ORCHESTRATION_SPECIFICATION.md), [ADR-002](decisions/ADR-002-AI-FIRST-WORKFLOW-SECOND.md).
