# First Implementation Handover (Proposal, revised for the Konam boundary)

> Status: **classification and ADR clarifications confirmed by the owner on 27 November 2026; implementation still not authorised.** No application code may be written until the owner approves this revised handover and the gates in section 5 are cleared. This revision supersedes the earlier version, which assumed Workflow Gateway would build accounts and a model-provider layer. That conflicts with the owner-confirmed ownership boundary in the [Technical Architecture](../docs/TECHNICAL_ARCHITECTURE.md) and the [Konam cross-repository integration specification](https://github.com/Konam-Automation-Platform/automation-business-control-centre/blob/main/docs/CROSS_REPOSITORY_INTEGRATION_SPECIFICATION.md).

## 1. Why this was revised

The roadmap requires every affected issue to be reviewed and classified as **(a) Gateway-owned** or **(b) a dependency on a Konam-owned platform service** before any Gateway implementation batch is authorised. Gateway owns the customer chat experience, canonical conversation and message history, UI/session state, the external REST/OpenAPI and MCP edge adapters, and the client for the Konam API. Konam owns platform identity, workspaces, membership, roles and entitlements, the model/provider, tool and workflow registries, routing, execution, approvals, usage and platform audit. Gateway must not build a second control plane and must not share a database with Konam.

## 2. Proposed classification of the M1 issues (owner to confirm)

| Issue | As written | Proposed classification | What Gateway builds |
|---|---|---|---|
| WG-006 Responsive chat interface | Chat UI | **Gateway-owned** | The interface, against the Gateway API |
| WG-007 Authentication and user accounts | Sign-up, sign-in, personal workspace | **Mostly a Konam dependency** | Edge authentication only: validate Konam-issued OIDC identity and tokens, session handling. **No user store, no membership, no workspace creation.** |
| WG-008 Conversation and message management | Conversations and messages by `workspace_id` | **Gateway-owned** | Canonical conversation and message storage and API. Workspace identifiers are references supplied by validated identity, never proof of authority. |
| WG-009 Model provider abstraction | Provider-neutral model interface | **Konam dependency** | A Konam API client interface with a stub for tests. Gateway holds no provider credentials and makes no provider calls. |

The issue texts are **not** rewritten by this proposal. Rewriting them is a change-control decision for the owner.

## 3. Proposed sequence

**Batch 0: design, no application code (needs owner approval of each item)**
1. Re-baseline the [database specification](../docs/DATABASE_SPECIFICATION.md) to Gateway-owned storage only (conversations, messages, UI/session state, opaque Konam references). Its own ownership update makes this a precondition for completing WG-004.
2. Draft the Gateway-to-Konam API contract (submit request, retrieve request, retrieve events, cancel, submit approval decision, discover capabilities). Konam owns the schema, so this is planned and approved in the Konam repository.
3. Decide the identity provider and token flow (open decision 1 in the integration specification).
4. Define conversation retention, export and deletion (open decision 2).
5. Owner confirms or amends the classification in section 2.

**Batch 1: first Gateway code, only after Batch 0**
1. Scaffold `apps/api` and CI, with pinned versions (ADR-005 conditions).
2. WG-008: Gateway-owned conversations and messages on the re-baselined schema.
3. Konam API client interface and a stub; correlation IDs and the stable error envelope from the integration specification.
4. WG-007 (reduced): edge token validation against a test identity provider.
5. WG-006: the chat interface.

## 4. Reconciliations needed for already-accepted ADRs

These ADRs were accepted before the boundary was applied to them. They are **not** edited here; the owner decides.

- **ADR-003 (Supabase for database and authentication):** the architecture limits Supabase to a candidate for **Gateway-owned conversation data only**. Supabase Auth must not become the platform identity, because Konam is authoritative and the identity provider is not yet chosen. Proposed clarification: accept ADR-003 for Gateway data storage only.
- **ADR-010 (personal and organisation workspace tenancy):** workspace authority and membership belong to Konam. Proposed clarification: Gateway treats workspaces as Konam-issued references and applies isolation by those references, rather than defining or creating workspaces.

## 5. Entry gates

- [x] Licence decided: proprietary, all rights reserved; copyright holder Mavos Solutions (27 November 2026).
- [x] Repository stays public for now (owner-confirmed 27 November 2026).
- [x] Branch protection enabled on `main` (27 November 2026).
- [x] CI approach in the standards approved; each workflow still arrives in its own reviewed pull request.
- [x] ADR-003, ADR-005, ADR-006 and ADR-010 accepted with conditions (27 November 2026), subject to the reconciliations in section 4.
- [x] Threat model accepted as the security baseline (27 November 2026).
- [x] Repository layout approved: `apps/api`, `apps/web`, `packages/`, `db/`.
- [x] No paid service is authorised for the first batch (owner-confirmed 27 November 2026).
- [ ] Database specification re-baselined to Gateway-owned scope and reviewed (precondition for WG-004 and any schema).
- [ ] Gateway-to-Konam API contract drafted and approved in the Konam repository.
- [ ] Identity provider and token flow decided.
- [ ] Conversation retention, export and deletion defined.
- [x] Classification in section 2 confirmed by the owner, and the ADR reconciliations in section 4 decided to be applied (27 November 2026).

## 6. Environment

- Tests run in CI against an ephemeral Postgres container ([validation report](../docs/TECHNOLOGY_VALIDATION_REPORT.md)); no Supabase project, Render service, hosted database or production secret is created, and hosted migrations remain forbidden without owner approval.
- Identity in tests uses signed test tokens from a local test identity provider, never real accounts.
- The Konam API is replaced by a stub that implements the agreed contract.

## 7. Definition of done

As in the [standards](../docs/DEVELOPMENT_AND_TESTING_STANDARDS.md), plus:

- Cross-workspace read, write and delete attempts fail for every endpoint, using Konam-validated references.
- No code path stores credentials, executes workflows, selects models or makes routing, approval or entitlement decisions.
- Konam unavailability produces an explicit unavailable state and never falls back to an ungoverned path.
- Contract tests cover each Gateway endpoint and the Konam client stub.

## 8. Out of scope

Workflow execution, routing, model and tool registries, approval decisions, BYOK and credential storage, platform audit, billing, marketplace, production deployment, hosted migrations and customer data.
