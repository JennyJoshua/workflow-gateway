# Implementation Roadmap

> Status: planning. No implementation is authorised yet. Phases map to the `phase:*` labels and to GitHub milestones.

## Strategic direction

The combined product is **AI-first, workflow-second**. Konam provides model-powered problem-solving, authoritative routing and governed workflow execution. Workflow Gateway provides the conversational application and external access interfaces. The Gateway roadmap must focus on its own customer experience and integration work rather than duplicating Konam platform services. This is an architectural boundary, not permission to reorder or expand approved implementation batches. See ADR-002 and the [cross-repository integration specification](https://github.com/Konam-Automation-Platform/automation-business-control-centre/blob/main/docs/CROSS_REPOSITORY_INTEGRATION_SPECIFICATION.md).

## Cross-repository delivery boundary

The milestone and issue register below is retained for traceability only; this documentation change does not silently rewrite issue scopes. Before any Gateway implementation batch is authorised, the affected issues must be reviewed and classified as (a) Gateway-owned UI/session/conversation/external-adapter work or (b) a dependency on a Konam-owned platform service. Do not implement a second authoritative router, model/tool/workflow registry, execution supervisor, permission/entitlement system, approval manager or platform audit store in Gateway. Any required Konam work must be planned and approved in the Konam repository.

## Phases

| Phase | Milestones | Goal |
|---|---|---|
| Foundation | M0 | Governance, ADRs, threat model, database specification, standards |
| MVP | M1 to M5 and the MVP items of M7 | A working AI problem-solving chat that can select direct models, tools and suitable workflows safely, ready for a controlled pilot |
| Expansion | M6, M8, M9 | Broader tools ecosystem, commercial platform, reusable workflow ecosystem and marketplace |

## Milestone mapping

| Milestone | Issues |
|---|---|
| M0 Architecture and Project Foundation | WG-001 to WG-005, WG-031 |
| M1 Chat Application Foundation | WG-006 to WG-009 |
| M2 Intent Analysis and Routing | WG-010, WG-014 |
| M3 Workflow Registry and Discovery | WG-011 to WG-013 |
| M4 Workflow Execution | WG-015 to WG-018 |
| M5 External Models and BYOK | WG-019, WG-020 |
| M6 Plugins and Tool Integrations | WG-021 |
| M7 Security, Approvals and Observability | WG-022, WG-023, WG-030 |
| M8 Commercial Platform | WG-024 to WG-027 |
| M9 Marketplace and Ecosystem | WG-028, WG-029 |

Full list with links: [Issue Register](../project-management/ISSUE_REGISTER.md). Milestones: [Milestones](../project-management/MILESTONES.md).

## Sequencing principles

1. Decisions before code: ADRs, threat model and database specification first.
2. Establish the conversational model foundation and provider-neutral abstraction; AI must be useful without requiring a workflow.
3. Build intent analysis and explainable capability routing across direct AI, external tools and reusable workflows.
4. Develop the workflow registry and execution capability for recurring tasks where governed repeatability adds value; do not create a workflow for every catalogue problem.
5. Security-critical items (tenant isolation, BYOK, approvals and external-agent authorisation) are designed early and verified before any pilot.
6. External REST/OpenAPI and MCP interfaces are designed around shared backend services; they do not introduce a parallel execution platform. WG-031 is an M0 planning issue, while implementation details remain in the existing M3, M4, M6 and M7 work.
7. Commercial and ecosystem features follow a proven MVP.

## Recommended first implementation batch (for owner approval)

Planning batch (no application code): WG-002, WG-003, WG-004, WG-005, then close out WG-001.

First code batch, only after the planning batch is reviewed and the owner approves: WG-007 (authentication), WG-008 (conversations) and WG-009 (model abstraction), followed by WG-006 (chat interface). WG-011 (workflow metadata contract) can proceed in parallel as a design task. This batch remains unchanged by ADR-002.

## Resolve before development begins

- Licence decided: proprietary, all rights reserved (27 November 2026). Repository visibility: public for now (owner-confirmed 27 November 2026; revisit before commercial release).
- ADR validation: Supabase, Render, FastAPI, LiteLLM, MCP and Next.js reviewed in the [Technology Validation Report](TECHNOLOGY_VALIDATION_REPORT.md); LangGraph spike outstanding. ADR-005 accepted with conditions on 27 November 2026.
- Threat model reviewed (WG-003) and approval-gate triggers agreed.
- Database specification reviewed (WG-004), including tenancy and RLS approach.
- Owner approval of the first implementation handover and of any service accounts to be created.
- Pilot customer segment and first candidate workflow for internal availability, selected based on recurring need and measurable value.
- WG-031 architecture and security requirements reviewed before implementation work in affected milestones.

See [Acceptance Criteria](ACCEPTANCE_CRITERIA.md).
