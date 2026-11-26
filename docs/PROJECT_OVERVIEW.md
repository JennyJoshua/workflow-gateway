# Project Overview

> Status: planning and foundation. Specification baseline: Version 1.0 (26 November 2026). Nothing described here is implemented yet.

## Vision

Workflow Gateway is a model-independent conversational AI platform through which customers can access:

- Proprietary business automation workflows.
- External AI models from multiple providers.
- Third-party plugins and tools.
- Specialist internal agents.
- Multi-step combinations of these capabilities.

Customers describe what they want to accomplish. A central orchestration agent analyses their intent, searches the proprietary workflow catalogue, evaluates available models and tools, chooses an execution strategy and returns the result through one chat interface.

The proprietary workflows are the principal commercial differentiator. The platform must not force proprietary workflows when another solution is genuinely more suitable.

## Problem

Customers face a fragmented landscape of AI models, plugins and automation tools. Each differs in interface, quality, cost, permissions and risk. Choosing and combining them requires expertise and creates governance gaps.

## Target customers and market

- **Small and medium businesses** that want business automation without integrating many tools.
- **Teams and organisations** that need governed, auditable and permissioned AI use.
- **Technical and power users** who want to bring their own API keys (BYOK).
- **Workflow developers and partners** (later phase) who publish workflows to the ecosystem.

Market sizing, pricing and go-to-market are **not yet defined** and require owner input and validation.

## Approved product direction

| Decision | Approved direction |
|---|---|
| Platform relationship | Both: standalone application and primary interface to the larger commercial workflow platform |
| Model supply | Both: platform-managed models and customer-provided API keys (BYOK) |
| Workflow routing | Configurable, with proprietary workflow-first default |
| Development approach | Build the chat foundation now and connect workflows progressively |

Recorded in [ADR-001](decisions/ADR-001-OVERALL-PLATFORM-DIRECTION.md). These decisions are not changed without a new ADR and owner approval.

## Principles

1. Model independence — no single provider is required.
2. Workflow-first by default, never workflow-only.
3. Honest availability — unfinished workflows are never presented as working commercial services.
4. Security, tenant isolation and human approval for high-risk actions.
5. Simplicity first — avoid unnecessary agent complexity in the first implementation.
6. Free-first development; no paid services without owner approval.
7. Explainability — customers and administrators can see why a route was chosen.

## Planned workflow product candidates

These are **planned product candidates, not completed integrations**. The registry must be able to host them in future; none exists today.

- RepoGuard
- IssuePilot
- ReleasePilot
- DocuFlow
- AgentManager

## Current phase and boundaries

The current assignment is project establishment and planning documentation only. No application code, Supabase provisioning, Render deployment, paid configuration, production secrets, customer account connection, database migration or external workflow integration is authorised. See [Security and Governance](SECURITY_AND_GOVERNANCE.md).

## Glossary

- **Workflow** — a proprietary, versioned business automation invoked through a defined contract.
- **Registry** — a governed catalogue (workflows, models, plugins and tools).
- **Adapter** — code that executes against one kind of backend behind a common interface.
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
- [Architecture Review Recommendations](ARCHITECTURE_REVIEW_RECOMMENDATIONS.md) (proposed; owner review pending)\n- [Architecture Decision Register](decisions/README.md)\n- [ADR-001](decisions/ADR-001-OVERALL-PLATFORM-DIRECTION.md)
