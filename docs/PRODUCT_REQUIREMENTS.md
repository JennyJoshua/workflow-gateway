# Product Requirements

> Status: planning. Requirements are the baseline for the backlog; items marked *proposed* require validation before they become commitments.

## Personas

- **Business user** — states goals in plain language, wants results and clear progress.
- **Organisation administrator** — manages members, integrations, keys, routing policy and usage.
- **Platform operator (owner)** — manages the workflow registry, lifecycle, availability and approvals.
- **Workflow developer** (later) — publishes workflows to the platform.

## Functional requirements

| ID | Requirement | Issues |
|---|---|---|
| FR-01 | Responsive web chat with streaming responses | WG-006 |
| FR-02 | Account registration, sign-in and session management | WG-007 |
| FR-03 | Persisted conversations and messages with history | WG-008 |
| FR-04 | Model-independent provider abstraction and model selector | WG-009, WG-019 |
| FR-05 | Intent analysis producing structured intents and clarifying questions | WG-010 |
| FR-06 | Configurable routing, proprietary workflow-first by default | WG-014, WG-025 |
| FR-07 | Workflow registry with lifecycle and availability tiers | WG-011, WG-012 |
| FR-08 | Workflow discovery by capability | WG-013 |
| FR-09 | Workflow execution, sync and async, with tracking | WG-015, WG-016 |
| FR-10 | Execution timeline visible in chat | WG-017 |
| FR-11 | Result quality review | WG-018 |
| FR-12 | Customer-supplied API keys (BYOK) | WG-020 |
| FR-13 | Plugin and MCP integration registry | WG-021 |
| FR-14 | Human approval gates for high-risk actions | WG-022 |
| FR-15 | Tenant isolation and permissions | WG-023 |
| FR-16 | Usage metering and customer usage information | WG-024 |
| FR-17 | Customer settings and routing controls | WG-025 |
| FR-18 | Subscriptions and entitlements | WG-026 |
| FR-19 | Organisation and team accounts | WG-027 |
| FR-20 | Workflow developer publishing and marketplace | WG-028, WG-029 |
| FR-21 | Audit log of security-relevant and routing events | WG-003, WG-004, WG-023 |

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

## User experience

- One conversational interface; no need to know which workflow, model or tool will be used.
- Model selector for users who want to choose; workflow discovery for browsing what is available to them.
- Plugin and integration management in settings.
- Execution timeline shows intent, routing choice, steps, approvals and results in plain language.
- Clear messaging when something is not available, needs approval, or has failed.
- Settings and usage page showing routing preferences, keys (masked) and usage.

## Out of scope for this phase

Any application code or deployment; real integrations with proprietary workflows; billing provider selection; public launch.

## Open questions (require owner input or validation)

- Initial customer segment and pilot customers.
- Pricing model and plan structure.
- Which first workflows are suitable for internal availability.
- Licensing approach (see [Contribution and Change Control](../project-management/CONTRIBUTION_AND_CHANGE_CONTROL.md)).

See also: [Technical Architecture](TECHNICAL_ARCHITECTURE.md), [Acceptance Criteria](ACCEPTANCE_CRITERIA.md).
