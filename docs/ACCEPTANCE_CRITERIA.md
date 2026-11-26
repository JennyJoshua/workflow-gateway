# Acceptance Criteria

> Status: planning. Criteria for each milestone mirror the GitHub milestone descriptions. An issue is complete only when its own acceptance criteria are met, reviewed and approved; writing the description does not complete it.

## Project setup (this handover)

- [x] Repository exists, public, default branch main, Issues enabled.
- [x] All documents in the agreed structure exist and cross-link.
- [x] 10 milestones and 30 issues exist with correct milestone assignments and labels.
- [x] No application code, secrets or deployment configuration in the repository.
- [x] README states this is a planning and foundation project.
- [ ] Licensing decision confirmed by the owner (open).
- [ ] Independent architectural review by ChatGPT completed (pending).

## Milestone criteria

| Milestone | Acceptance criteria |
|---|---|
| M0 | ADRs and database specification reviewed by ChatGPT and approved by the owner; threat model reviewed; standards documented; no application code produced |
| M1 | A signed-in user can hold persisted conversations with at least one model through the abstraction layer |
| M2 | A request yields a structured intent and an explainable routing decision; customer-configurable policy with safe fallback |
| M3 | Workflows register with validated metadata; lifecycle and availability rules enforced; unfinished workflows never shown as commercially available |
| M4 | An internally available workflow runs from chat with visible progress, a recorded outcome and a quality review result |
| M5 | At least two providers selectable; customer keys encrypted, never displayed after entry, never logged, usable only within the owning tenant |
| M6 | A plugin or MCP server can be registered, enabled per tenant with explicit permissions and invoked with audit logging |
| M7 | High-risk actions require recorded approval; cross-tenant access impossible in tests; threat-model mitigations verified; owner signs off pilot readiness |
| M8 | Usage metered accurately per tenant; entitlements gate features; teams administered; any paid service approved by the owner |
| M9 | Third-party workflows pass review, security scanning and lifecycle gates; commercial terms approved by the owner before launch |

## Cross-cutting definition of done

1. Acceptance criteria of the issue are met and demonstrated.
2. Tests required by the issue pass; security-relevant work reviewed by ChatGPT.
3. Documentation and ADRs updated and consistent.
4. Audit-log and tenant-isolation implications addressed.
5. No secrets committed; no unapproved paid service used.
6. Owner approves the merge to main.

See [Implementation Roadmap](IMPLEMENTATION_ROADMAP.md) and [Security and Governance](SECURITY_AND_GOVERNANCE.md).
