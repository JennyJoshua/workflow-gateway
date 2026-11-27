# Workflow Gateway

> **Status: planning and foundation project — not a production application.** This repository currently contains specifications, governance documents and a GitHub Issues backlog only. No application code, deployment or hosted service exists yet.

Workflow Gateway is an AI-powered problem-solving and workflow execution platform. It connects model-agnostic AI intelligence, external tools and integrations, reusable automation workflows, and governed execution through one conversational interface.

## What it is

Customers describe the problem or outcome they need through Workflow Gateway. The Gateway submits the request to the Konam Automation Platform, whose internal Konam Capability Router determines the most suitable handling path: direct AI reasoning, AI with authorised external tools, an existing reusable workflow, or a recommendation to assess a recurring unmet need. Konam enforces platform permissions, execution policy and human approvals.

## Foundational principle: AI-first, workflow-second

Do not create a predefined workflow for every problem. AI models can already solve many tasks dynamically through reasoning, conversation, document analysis, coding and tool use. Workflows are valuable where repeatability, reliable execution, triggers, integrations, business rules, approvals and monitoring justify a structured process.

The platform has three complementary capabilities:

1. **AI Intelligence** — connected, model-agnostic AI capabilities solve suitable problems dynamically without requiring a predefined workflow.
2. **Workflow Automation** — reusable workflows provide consistent, governed execution for recurring or operational tasks.
3. **Konam Capability Router** — Konam's internal, model-agnostic decision and coordination capability selects between direct AI, authorised tools, existing workflows or a recommendation for workflow-development assessment, subject to Konam policy and approvals. The separate Workflow Gateway application is the customer-facing conversational and external-access layer.

The default decision principle is to use the simplest safe and suitable capability, not to prefer workflows merely because they exist. This supersedes the earlier workflow-first routing default recorded in ADR-001; see [ADR-002](docs/decisions/ADR-002-AI-FIRST-WORKFLOW-SECOND.md).

## Why it exists

Businesses otherwise juggle many AI models, plugins and automation tools, each with its own interface, pricing, permissions and risk profile. Workflow Gateway gives customers one place to state a goal while the platform decides how best to achieve it, with governance, approvals and auditability built in.

## How capabilities interact

| Capability | Role | Registry / interface |
|---|---|---|
| AI models and reasoning | Dynamic problem-solving, analysis, drafting and planning | Model and capability registry |
| External tools and integrations | Access to third-party systems, data and actions | Plugin and tool registry |
| Reusable workflows | Repeatable, governed execution for recurring tasks | Workflow registry |
| Internal agents | Optional specialist capabilities where justified | Internal capability interfaces |

Konam's orchestration layer may use one capability or combine several in a multi-step plan, subject to tenant policy, entitlements and human approval gates. A new workflow is a development recommendation, not an automatic response to every unmatched request.

## Architecture at a glance

1. **Customer interface** — responsive conversational experience.
2. **Gateway application backend** — customer sessions, canonical conversation history, presentation state and edge request validation.
3. **Konam platform API and control plane** — authoritative identity, workspace membership, permissions, entitlements, policy, task records and audit.
4. **Konam Capability Router and registries** — internal request assessment, model/tool/workflow selection, capability registries and execution supervision.
5. **Konam execution and governance services** — authorised adapters, workflow execution, approvals, usage and platform audit. Gateway's external REST/OpenAPI/MCP adapters call these services through the versioned Konam API.

Technology choices are **proposed and require validation**. See [Technical Architecture](docs/TECHNICAL_ARCHITECTURE.md).

## Current development status

Phase: **M0 — Architecture and Project Foundation (planning).** The repository, documentation and backlog are established. Software implementation has **not** started and is not authorised until the owner approves the first implementation batch.

## Roadmap

| Milestone | Theme |
|---|---|
| [M0](https://github.com/Konam-Automation-Platform/workflow-gateway/milestone/1) | Architecture and Project Foundation |
| [M1](https://github.com/Konam-Automation-Platform/workflow-gateway/milestone/2) | Chat Application Foundation |
| [M2](https://github.com/Konam-Automation-Platform/workflow-gateway/milestone/3) | Intent Analysis and Routing |
| [M3](https://github.com/Konam-Automation-Platform/workflow-gateway/milestone/4) | Workflow Registry and Discovery |
| [M4](https://github.com/Konam-Automation-Platform/workflow-gateway/milestone/5) | Workflow Execution |
| [M5](https://github.com/Konam-Automation-Platform/workflow-gateway/milestone/6) | External Models and BYOK |
| [M6](https://github.com/Konam-Automation-Platform/workflow-gateway/milestone/7) | Plugins and Tool Integrations |
| [M7](https://github.com/Konam-Automation-Platform/workflow-gateway/milestone/8) | Security, Approvals and Observability |
| [M8](https://github.com/Konam-Automation-Platform/workflow-gateway/milestone/9) | Commercial Platform |
| [M9](https://github.com/Konam-Automation-Platform/workflow-gateway/milestone/10) | Marketplace and Ecosystem |

Details: [Implementation Roadmap](docs/IMPLEMENTATION_ROADMAP.md).

## Documentation

- [Project Overview](docs/PROJECT_OVERVIEW.md)
- [Product Requirements](docs/PRODUCT_REQUIREMENTS.md)
- [Technical Architecture](docs/TECHNICAL_ARCHITECTURE.md)
- [External AI Agent Integration Specification](docs/EXTERNAL_AI_AGENT_INTEGRATION_SPECIFICATION.md)
- [Proposed External REST API — OpenAPI v1](api/openapi/v1.yaml) (planning contract; not deployed)
- [Database Specification](docs/DATABASE_SPECIFICATION.md) (proposed; no migrations; see [independent review](docs/DATABASE_SPECIFICATION_REVIEW.md))
- [Security and Governance](docs/SECURITY_AND_GOVERNANCE.md)
- [Workflow Registry Specification](docs/WORKFLOW_REGISTRY_SPECIFICATION.md)
- [Routing and Orchestration Specification](docs/ROUTING_AND_ORCHESTRATION_SPECIFICATION.md)
- [Implementation Roadmap](docs/IMPLEMENTATION_ROADMAP.md)
- [Acceptance Criteria](docs/ACCEPTANCE_CRITERIA.md)
- [Security Threat Model](docs/SECURITY_THREAT_MODEL.md)\n- [Architecture Review Recommendations](docs/ARCHITECTURE_REVIEW_RECOMMENDATIONS.md) (proposed)
- [Architecture Decision Register](docs/decisions/README.md)
- [ADR-001 Overall Platform Direction](docs/decisions/ADR-001-OVERALL-PLATFORM-DIRECTION.md)
- [ADR-002 AI-first, Workflow-second Architecture](docs/decisions/ADR-002-AI-FIRST-WORKFLOW-SECOND.md)
- [Milestones](project-management/MILESTONES.md)
- [Issue Register](project-management/ISSUE_REGISTER.md)
- [Contribution and Change Control](project-management/CONTRIBUTION_AND_CHANGE_CONTROL.md)

## Governance

The owner approves main-branch merges, production deployments, destructive actions, paid services and upgrades, material scope changes and hosted database migrations. This strategic architecture update does not authorise implementation, deployment, service provisioning or expansion of the approved MVP scope. ChatGPT acts as Chief AI Architect and Independent Reviewer. See [Contribution and Change Control](project-management/CONTRIBUTION_AND_CHANGE_CONTROL.md).

## Licence

Licensing is **unresolved pending owner confirmation**. See [LICENSE](LICENSE). Until it is resolved, all rights are reserved.
