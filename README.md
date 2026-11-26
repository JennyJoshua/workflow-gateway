# Workflow Gateway

> **Status: planning and foundation project — not a production application.** This repository currently contains specifications, governance documents and a GitHub Issues backlog only. No application code, deployment or hosted service exists yet.

Workflow Gateway is an intelligent AI workflow orchestration platform that connects proprietary automation workflows, multiple AI models, plugins and external tools through one conversational interface.

## What it is

A model-independent conversational AI platform. Customers describe what they want to accomplish. A central orchestration engine analyses the intent, searches the proprietary workflow catalogue, evaluates available models and tools, chooses an execution strategy and returns the result through a single chat interface.

## Why it exists

Businesses otherwise juggle many AI models, plugins and automation tools, each with its own interface, pricing, permissions and risk profile. Workflow Gateway gives customers one place to state a goal while the platform decides how best to achieve it, with governance, approvals and auditability built in.

## Key differentiator

The proprietary business automation workflows are the principal commercial differentiator. By default the platform is **workflow-first**: a qualifying proprietary workflow is preferred. It does **not** force a workflow when another solution is genuinely more suitable.

## Intended customers

- Small and medium businesses that want automation without assembling tools themselves.
- Teams and organisations that need governed, auditable AI use.
- Technical customers who want to bring their own model API keys (BYOK).
- In a later phase, third-party workflow developers (ecosystem).

## How workflows, models and plugins interact

| Capability | Role | Registry |
|---|---|---|
| Proprietary workflows | Preferred way to complete matching business tasks | Workflow registry |
| External AI models | General reasoning, drafting and fallback; platform-managed or customer BYOK | Model registry |
| Plugins and tools (including MCP servers) | Access to third-party systems and data | Plugin and tool registry |
| Internal agents | Specialist helpers inside the platform | Internal capabilities |

The orchestration engine may use one capability or combine several in a multi-step plan, subject to customer policy, entitlements and human approval gates.

## Architecture at a glance

1. **Customer interface** — responsive web chat (candidate: Next.js).
2. **Application backend** — authentication, tenancy, conversations, permissions (candidate: Python FastAPI).
3. **Central orchestration engine** — Intent Analyst, Workflow Discovery Agent, Model and Tool Router, Execution Supervisor, Quality Reviewer (logical responsibilities, not necessarily separate agents).
4. **Three registries** — proprietary workflows, external models, plugins and tools.
5. **Execution adapters** — workflow APIs, GitHub Apps and Actions, model providers, MCP servers, third-party APIs, internal agents.

Technology choices are **proposed and require validation**. See [Technical Architecture](docs/TECHNICAL_ARCHITECTURE.md).

## Current development status

Phase: **M0 — Architecture and Project Foundation (planning).** The repository, documentation and backlog (10 milestones, 30 issues) have been established. Software implementation has **not** started and is not authorised until the owner approves the first implementation batch.

## Roadmap

| Milestone | Theme |
|---|---|
| [M0](https://github.com/JennyJoshua/workflow-gateway/milestone/1) | Architecture and Project Foundation |
| [M1](https://github.com/JennyJoshua/workflow-gateway/milestone/2) | Chat Application Foundation |
| [M2](https://github.com/JennyJoshua/workflow-gateway/milestone/3) | Intent Analysis and Routing |
| [M3](https://github.com/JennyJoshua/workflow-gateway/milestone/4) | Workflow Registry and Discovery |
| [M4](https://github.com/JennyJoshua/workflow-gateway/milestone/5) | Workflow Execution |
| [M5](https://github.com/JennyJoshua/workflow-gateway/milestone/6) | External Models and BYOK |
| [M6](https://github.com/JennyJoshua/workflow-gateway/milestone/7) | Plugins and Tool Integrations |
| [M7](https://github.com/JennyJoshua/workflow-gateway/milestone/8) | Security, Approvals and Observability |
| [M8](https://github.com/JennyJoshua/workflow-gateway/milestone/9) | Commercial Platform |
| [M9](https://github.com/JennyJoshua/workflow-gateway/milestone/10) | Marketplace and Ecosystem |

Details: [Implementation Roadmap](docs/IMPLEMENTATION_ROADMAP.md).

## Documentation

- [Project Overview](docs/PROJECT_OVERVIEW.md)
- [Product Requirements](docs/PRODUCT_REQUIREMENTS.md)
- [Technical Architecture](docs/TECHNICAL_ARCHITECTURE.md)
- [Security and Governance](docs/SECURITY_AND_GOVERNANCE.md)
- [Workflow Registry Specification](docs/WORKFLOW_REGISTRY_SPECIFICATION.md)
- [Routing and Orchestration Specification](docs/ROUTING_AND_ORCHESTRATION_SPECIFICATION.md)
- [Implementation Roadmap](docs/IMPLEMENTATION_ROADMAP.md)
- [Acceptance Criteria](docs/ACCEPTANCE_CRITERIA.md)
- [Architecture Review Recommendations](docs/ARCHITECTURE_REVIEW_RECOMMENDATIONS.md) (proposed)\n- [Architecture Decision Register](docs/decisions/README.md)\n- [ADR-001 Overall Platform Direction](docs/decisions/ADR-001-OVERALL-PLATFORM-DIRECTION.md)
- [Milestones](project-management/MILESTONES.md)
- [Issue Register](project-management/ISSUE_REGISTER.md)
- [Contribution and Change Control](project-management/CONTRIBUTION_AND_CHANGE_CONTROL.md)

## Governance

The owner approves main-branch merges, production deployments, destructive actions, paid services and upgrades, material scope changes and hosted database migrations. ChatGPT acts as Chief AI Architect and Independent Reviewer. See [Contribution and Change Control](project-management/CONTRIBUTION_AND_CHANGE_CONTROL.md).

## Licence

Licensing is **unresolved pending owner confirmation**. See [LICENSE](LICENSE). Until it is resolved, all rights are reserved.
