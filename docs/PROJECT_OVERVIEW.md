# Project Overview

> Status: planning and foundation. Specification baseline: Version 1.1 (26 November 2026). Nothing described here is implemented yet.

## Vision

Workflow Gateway is an AI-powered problem-solving and workflow execution platform. Customers describe the outcome they need; the platform selects and coordinates the most suitable combination of AI intelligence, external tools and reusable workflows, under clear security and approval controls.

## Foundational principle: AI-first, workflow-second

Do not create a predefined workflow for every problem. Connected AI models can already solve many tasks dynamically through reasoning, conversation, document analysis, coding and tool integrations. A workflow should be created or selected when structured, repeatable execution adds value—not merely because a task can be represented as steps.

The platform has three complementary capabilities:

1. **AI Intelligence:** model-agnostic AI models dynamically solve suitable problems without requiring a predefined workflow.
2. **Workflow Automation:** reusable workflows handle recurring tasks that benefit from consistent execution, triggers, integrations, business rules, approvals and monitoring.
3. **Workflow Gateway:** the intelligent decision and coordination layer determines which capability or combination is appropriate.

For each request, the Gateway should progressively assess:
- Can an AI model solve this directly?
- Does the task require external tools, data or integrations?
- Is an existing workflow suitable and authorised?
- If no suitable capability exists, is the need recurring and valuable enough to recommend developing a new workflow?
- Does any part of the plan require human approval?

The decision should favour the simplest safe and suitable route. It must not automatically create workflows for one-off or readily solvable tasks. Recommending a new workflow is not authority to create, publish or execute one; development remains subject to separate scope and owner approval.

## Problem

Customers face a fragmented landscape of AI models, plugins, automation tools and business processes. Choosing and combining them requires expertise and creates governance gaps. The platform provides a single problem-oriented interface and makes the capability choice explainable and governable.

## Target customers and market

- **Small and medium businesses** that want problem-solving and business automation without integrating many tools.
- **Teams and organisations** that need governed, auditable and permissioned AI use.
- **Technical and power users** who want to bring their own API keys (BYOK).
- **Workflow developers and partners** (later phase) who publish reusable workflows to the ecosystem.

Market sizing, pricing and go-to-market are **not yet defined** and require owner input and validation.

## Approved product direction

| Decision | Approved direction |
|---|---|
| Platform relationship | Both: standalone application and primary interface to the larger commercial workflow platform |
| Model supply | Both: platform-managed models and customer-provided API keys (BYOK) |
| Problem-solving strategy | AI-first, workflow-second; choose the simplest safe and suitable capability |
| Workflow role | Reusable governed automation for recurring tasks where consistency and operational controls add value |
| Development approach | Build the chat foundation and connect models, tools and workflows progressively |

The foundational AI-first direction is recorded in [ADR-002](decisions/ADR-002-AI-FIRST-WORKFLOW-SECOND.md). It supersedes ADR-001's earlier workflow-first routing default; the other ADR-001 decisions remain in force.

## Principles

1. AI-first, workflow-second — do not duplicate general model capabilities with unnecessary predefined workflows.
2. Model independence — no single provider or model is required; new and existing models can be integrated through stable abstractions.
3. Workflows for repeatability — use workflows when triggers, integrations, business rules, approvals, monitoring or consistent execution justify them.
4. No automatic workflow proliferation — an unmatched request may be handled by AI/tools, clarified, declined, or lead to a separately approved workflow-development proposal.
5. Security and tenant isolation — permissions and entitlements apply to every route and step.
6. Human accountability — high-risk or policy-controlled actions require recorded human approval.
7. Explainability — users and administrators can understand why a route was selected.
8. Simplicity first — avoid unnecessary agent and workflow complexity.
9. Free-first development; no paid services without owner approval.
10. Honest availability — unfinished workflows are never presented as working commercial services.

## Planned workflow product candidates

These are **planned product candidates, not completed integrations**. The registry must be able to host them in future; none exists today.

- RepoGuard
- IssuePilot
- ReleasePilot
- DocuFlow
- AgentManager

## Current phase and boundaries

The current assignment is strategic architecture and planning documentation only. This update does not authorise application code, infrastructure provisioning, deployment, paid configuration, production secrets, customer account connection, database migration, external workflow integration, or expansion of the approved MVP scope. See [Security and Governance](SECURITY_AND_GOVERNANCE.md).

## Glossary

- **AI Intelligence** — model capabilities used for dynamic reasoning and problem-solving.
- **Workflow** — a reusable, versioned automation invoked through a defined contract.
- **Capability** — an available model, tool, integration, workflow or internal service that can contribute to solving a request.
- **Registry** — a governed catalogue of models, workflows, plugins or tools.
- **Adapter** — code that connects to one kind of backend behind a common interface.
- **BYOK** — bring your own key: customer-supplied model provider credentials.
- **Tenant** — a customer organisation whose data is isolated from others.
- **Approval gate** — a point where execution pauses for human approval.

## Document map

- [Product Requirements](PRODUCT_REQUIREMENTS.md)
- [Technical Architecture](TECHNICAL_ARCHITECTURE.md)
- [Security and Governance](SECURITY_AND_GOVERNANCE.md)
- [Workflow Registry Specification](WORKFLOW_REGISTRY_SPECIFICATION.md)
- [Routing and Orchestration Specification](ROUTING_AND_ORCHESTRATION_SPECIFICATION.md)
- [Implementation Roadmap](IMPLEMENTATION_ROADMAP.md)
- [Acceptance Criteria](ACCEPTANCE_CRITERIA.md)
- [ADR-001](decisions/ADR-001-OVERALL-PLATFORM-DIRECTION.md)
- [ADR-002](decisions/ADR-002-AI-FIRST-WORKFLOW-SECOND.md)
