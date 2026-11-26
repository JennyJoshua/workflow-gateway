# Implementation Roadmap

> Status: planning. No implementation is authorised yet. Phases map to the `phase:*` labels and to GitHub milestones.

## Phases

| Phase | Milestones | Goal |
|---|---|---|
| Foundation | M0 | Governance, ADRs, threat model, database specification, standards |
| MVP | M1 to M5 and the MVP items of M7 | A working chat platform that routes to workflows and models safely, ready for a controlled pilot |
| Expansion | M6, M8, M9 | Tools ecosystem, commercial platform, marketplace |

## Milestone mapping

| Milestone | Issues |
|---|---|
| M0 Architecture and Project Foundation | WG-001 to WG-005 |
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
2. Chat foundation now, workflows connected progressively (approved direction).
3. Security-critical items (tenant isolation, BYOK, approvals) are designed early and verified before any pilot.
4. Commercial and ecosystem features follow a proven MVP.

## Recommended first implementation batch (for owner approval)

Planning batch (no application code): WG-002, WG-003, WG-004, WG-005, then close out WG-001.

First code batch, only after the planning batch is reviewed and the owner approves: WG-007 (authentication), WG-008 (conversations) and WG-009 (model abstraction), followed by WG-006 (chat interface). WG-011 (workflow metadata contract) can proceed in parallel as a design task.

## Resolve before development begins

- Licensing approach and repository visibility confirmation.
- ADR validation spikes for Supabase, Render, FastAPI, Next.js, LiteLLM, LangGraph and MCP, including free-tier limits.
- Threat model reviewed (WG-003) and approval-gate triggers agreed.
- Database specification reviewed (WG-004), including tenancy and RLS approach.
- Owner approval of the first implementation handover and of any service accounts to be created.
- Pilot customer segment and first candidate workflow for internal availability.

See [Acceptance Criteria](ACCEPTANCE_CRITERIA.md).
