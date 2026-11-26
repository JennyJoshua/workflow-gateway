# WG-004 — Independent Database Specification Review

> **Review outcome: Conditionally satisfactory as a logical planning specification.**
>
> This is an architecture/document review, not an executable-schema review, security audit, legal review or owner acceptance. The database specification remains **Proposed**. WG-004 remains open.

## 1. Review scope

Reviewed on 26 November 2026:

- `docs/DATABASE_SPECIFICATION.md`
- Product requirements
- Technical architecture
- Workflow registry specification
- Routing and orchestration specification
- Security threat model
- Security and governance
- ADR-009 workspace tenancy, ADR-010 durable jobs and ADR-011 BYOK credential security

The review checked coverage, terminology, tenant boundaries, cross-document consistency, security implications and whether proposed decisions were incorrectly represented as accepted.

## 2. Outcome

The specification is suitable to serve as the current logical baseline for further owner review. It covers the principal data domains and connects them to the threat model. It correctly keeps implementation, vendor selection, migrations and production operation out of scope.

No critical conceptual blocker was identified that requires discarding or restructuring the whole specification. However, several details must be resolved before it can be translated into physical SQL or accepted as an implementation-ready schema.

## 3. Cross-document corrections made

| Finding | Correction |
|---|---|
| Security and governance still described an organisation identifier as the key on all tenant data | Replaced with the universal `workspace_id` model and linked the database specification |
| Workflow registry API contract used `tenant_id` while the data model uses `workspace_id` | Updated contract to use `workspace_id` for personal and organisation workspaces |
| Routing policy description referred only to organisation and user | Updated it to cover personal and organisation workspaces |

The technical architecture already describes `workspace_id` as the universal tenant key. ADR-009 already proposes personal workspaces and workspace-scoped tenant data.

## 4. Required clarifications before physical schema design

These are not reasons to reject the logical specification, but they must be addressed before executable migrations are prepared.

### R1 — Platform-level audit events

The proposed `audit_logs` entity is shown with `workspace_id`, while platform operator events may not belong to a customer workspace. Decide whether to:
- permit a null workspace only for explicitly typed platform-scope audit events with separate platform authorisation; or
- maintain a separate platform audit store/table.

Do not weaken tenant RLS to solve this.

### R2 — Workflow ownership and catalogue boundaries

The `workflows` entity combines platform-owned and potentially customer-owned workflows. Before customer-authored workflows are enabled, define ownership constraints, publication permissions, visibility and who may transition lifecycle states. For v1, the platform-owned catalogue can remain the only publisher if that matches the approved scope.

### R3 — Polymorphic references

Fields such as `selected_route_ref`, `route_ref`, `provider_or_integration_ref`, `resource_ref`, `aggregate_id` and `target_ref` cannot all be protected by ordinary foreign keys when they point to different entity types. For physical design, use typed reference columns/tables, a constrained reference registry, or explicitly document where application-level referential checks are unavoidable. Do not leave security-critical references unchecked.

### R4 — Membership uniqueness

“Unique active membership” generally needs a partial unique index or an equivalent membership lifecycle design. Confirm the chosen database supports the selected constraint form; if portability is required, document the alternative. Prevent duplicate active memberships under concurrent invitation acceptance.

### R5 — Approval assignment

Clarify whether `approver_user_id` is mandatory at request creation or whether the request can be assigned to an eligible approver group and claimed later. The database and API must ensure only an eligible approver can decide it, and only one valid decision can be consumed.

### R6 — Platform audit and append-only guarantees

Application-level append-only behaviour does not prevent a sufficiently privileged database administrator from altering rows. Before production, decide whether external immutable audit export, signed hash chaining or another tamper-evidence control is proportionate.

### R7 — Retention and legal requirements

The document correctly leaves durations undecided. Before customer pilot, define retention by data class, backup expiry, deletion propagation, legal hold and relevant jurisdictions. Do not market deletion as complete until all in-scope stores and providers have been accounted for.

## 5. Security review observations

Positive design elements:

- Universal workspace tenancy rather than organisation-only tenancy.
- Composite workspace-scoped foreign-key requirement.
- RLS plus independent API/worker/callback authorisation.
- Credential references/ciphertext metadata rather than plaintext secrets.
- Immutable workflow versions and lifecycle history.
- Action-bound approvals with expiry and one-time consumption.
- Durable outbox, attempts, idempotency and reconciliation.
- Explicit retention, deletion and export design.
- Threat-to-schema mapping and negative-test requirements.

Implementation must prove these controls. Their presence in documentation is not evidence that they are operating.

## 6. Required owner decisions

Owner review is still required for:

1. Confirming `workspace_id` as the universal tenant key.
2. Personal workspace creation and organisation ownership lifecycle.
3. Initial roles and platform-vs-workspace administration boundary.
4. Credential storage/key-management approach.
5. Data classification and retention durations.
6. Workflow publisher/ownership scope for the first release.
7. Platform-level audit storage approach.
8. Applicable privacy, jurisdiction and data-residency obligations.

Technology ADRs 002–011 remain proposed unless separately accepted by the owner.

## 7. Review disposition

- [x] Logical entity coverage reviewed.
- [x] Tenant model reviewed.
- [x] Threat-model alignment reviewed.
- [x] Cross-document terminology corrections made.
- [x] Open implementation-level questions recorded.
- [ ] Owner acceptance of database specification.
- [ ] Owner decisions on unresolved data/tenancy policies.
- [ ] Physical schema and migration design (future, separately authorised).
- [ ] Independent implementation/security tests (future).

**Recommendation:** Keep WG-004 open. Treat the database specification as the proposed logical baseline after owner review. Do not provision a database or create migrations until the relevant decisions and approvals are complete.
