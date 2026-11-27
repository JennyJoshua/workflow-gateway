# First Implementation Handover (Proposal)

> Status: **proposed, not authorised.** No application code may be written until the owner approves this handover and the gates below are cleared. Nothing in this document provisions a service, adds a workflow or changes an approved decision.

## 1. Purpose

Define the smallest first code batch that is consistent with the roadmap, the approved direction (ADR-001, ADR-002) and the proposed standards, so the owner can approve, amend or reject it.

## 2. Proposed batch

| Order | Issue | Outcome |
|---|---|---|
| 1 | WG-007 Authentication and user accounts | Sign-up and sign-in; a personal workspace created per user; roles owner, admin and member enforced server-side ([ADR-010](decisions/ADR-010-PERSONAL-AND-ORGANISATION-TENANCY.md)) |
| 2 | WG-008 Conversation and message management | Conversations and messages scoped by `workspace_id`; soft delete; retention rules from the [database specification](DATABASE_SPECIFICATION.md) |
| 3 | WG-009 Model provider abstraction | Provider-neutral interface with a **stub provider** for tests; no live provider keys |
| 4 | WG-006 Responsive chat interface | Chat UI against the above, after the backend contracts are reviewed |

WG-011 (workflow metadata contract) may proceed in parallel as a design task. BYOK (WG-020), routing (WG-014), workflows and external interfaces are **out of scope** for this batch.

## 3. Entry gates (all require the owner)

- [x] Licence decision recorded: proprietary, all rights reserved (27 November 2026).
- [ ] Acceptance of the ADRs this batch depends on: ADR-003 (database and authentication), ADR-006 (interface) and ADR-010 (tenancy) accepted 27 November 2026; **ADR-005 (backend) awaits the FastAPI spike**; ADR-007 optional.
- [ ] Threat model and database specification reviewed and accepted.
- [x] Branch protection approved and enabled on `main` (27 November 2026).
- [x] CI approach in the standards approved by the owner (27 November 2026); each workflow still arrives in its own reviewed pull request.
- [ ] Confirmation that no paid service is authorised for this batch.
- [ ] Approval of the repository layout (section 4).

## 4. Proposed layout (to be confirmed by the ADRs)

A single repository with `apps/api` (backend), `apps/web` (interface), `packages/` (shared contracts), `db/` (reviewed SQL migrations, applied only to disposable databases in CI) and the existing `docs/`. The OpenAPI contract in `api/openapi/v1.yaml` stays a planning artefact and is not implemented in this batch.

## 5. Environment

- Tests run in CI against an ephemeral Postgres container ([validation report](TECHNOLOGY_VALIDATION_REPORT.md)).
- No Supabase project, Render service, hosted database or production secret is created. Hosted migrations remain forbidden without owner approval.
- Authentication in tests uses a local test identity provider or signed test tokens, never real accounts.

## 6. Definition of done

As in the [standards](DEVELOPMENT_AND_TESTING_STANDARDS.md), plus for this batch:

- Cross-workspace read, write and delete attempts are tested for every endpoint and fail.
- Role checks are tested for owner, admin and member.
- Audit events exist for sign-in, membership change and conversation deletion; logs contain no message content.
- A contract test covers each new endpoint.
- Stub provider tests prove no real provider call is made.

## 7. Pull request plan

One pull request per issue, each linked to its issue, each reviewed independently by ChatGPT and merged only on owner approval. The scaffolding and CI pull request comes first and needs the CI approval above.

## 8. Risks and open questions

- Free-tier limits make demos fragile (see the validation report); the pilot needs paid hosting.
- Region and data-residency choice for the target market is undecided.
- FastAPI and LangGraph have not been validated; a short spike precedes any dependency on them.
- ADR-011 (durable queue) must be revisited before WG-016 because free Render cannot run workers.

## 9. Out of scope

Workflow execution, routing policy, BYOK, plugins and MCP, billing, marketplace, production deployment, hosted migrations, and any customer data.
