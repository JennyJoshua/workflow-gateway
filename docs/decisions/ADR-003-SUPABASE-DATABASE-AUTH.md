# ADR-003: Supabase for Database and Authentication

- **Status:** Proposed — not owner-approved
- **Date:** 2026-11-26
- **Deciders:** Owner (final approval); ChatGPT (architecture review)
- **Related issues:** WG-002, WG-004, WG-007, WG-020, WG-023

## Context

The platform needs relational, tenant-scoped persistence, authentication, authorisation support, and potentially file storage. Supabase is the current candidate because it provides managed PostgreSQL and adjacent services. The database must remain portable and must not make application security depend solely on a vendor-specific feature.

## Decision drivers

- Free-first development without an early paid commitment.
- PostgreSQL compatibility and exportability.
- Strong tenant isolation and auditable access control.
- Authentication integration with a Python API and Next.js frontend.
- Backup, recovery, inactivity and quota limits.
- Avoiding dependence on proprietary service APIs where standard PostgreSQL is sufficient.

## Options considered

| Option | Advantages | Disadvantages | Cost/free-tier implications | Lock-in and exit considerations |
|---|---|---|---|---|
| Supabase | Managed PostgreSQL, Auth, Storage, RLS and developer tooling | Free projects pause after inactivity; quotas and some operational features limited | Free tier currently lists 2 active projects, 500 MB database, 5 GB egress, 1 GB file storage; inactivity pause after one week; automatic backups not included | PostgreSQL data is portable; Auth, Storage, Edge Functions and RLS policy conventions create some service-specific migration work |
| Self-managed PostgreSQL + separate auth | Maximum control and portability | More operational work, backup, patching and security responsibility | Software can be free; hosting and operations are not necessarily free | Lower provider lock-in, higher operator burden |
| Other managed PostgreSQL/auth provider | Alternative operational model | Requires comparative testing and account setup | Free tiers vary and must be rechecked | Depends on provider-specific auth and storage choices |

## Proposed decision

Use Supabase as the **preferred development candidate**, not yet as a final production commitment. Keep the core schema in standard PostgreSQL, keep business logic in the application backend, and isolate Supabase Auth/Storage usage behind well-defined interfaces where practical.

Do not treat the Free plan as a production availability or backup solution. A production decision requires an owner-approved cost and reliability plan.

## Security, privacy and data residency

- Enable RLS on every exposed tenant-owned table and test cross-tenant denial.
- Do not use user-editable user metadata for authorisation decisions.
- Never expose service-role or secret keys in browser code.
- Ensure privileged functions and views do not bypass intended access policies.
- Confirm data-region choice and provider data-processing terms before real customer data is stored.
- BYOK secrets require a separately reviewed encryption/key-management design; do not assume database encryption alone solves credential lifecycle security.

## Consequences

### Positive
- Fast path to relational data, auth and RLS testing.
- PostgreSQL reduces data-model lock-in.
- Free tier is suitable for early experiments.

### Negative / trade-offs
- Free projects may pause after one week of inactivity and do not include automatic backups.
- Quotas are too constrained to assume suitability for a commercial production workload.
- Supabase-specific auth and storage features create migration work.

### Operational implications
- Export and restore procedure must be tested.
- Quota, pause and backup behaviour must be included in pilot readiness.
- No project provisioning or schema migration is authorised by this ADR.

## Validation plan

1. Compare Supabase Auth with a backend-owned identity abstraction.
2. Prototype RLS policies against personal and organisation workspace scenarios using synthetic data only.
3. Test export/restore from PostgreSQL.
4. Confirm the intended region, current plan limits and data-processing terms.
5. Define a backup and recovery strategy before any production use.

## References

- https://supabase.com/pricing
- https://supabase.com/docs/guides/platform/billing-on-supabase
- https://supabase.com/docs/guides/storage/serving/bandwidth
- https://supabase.com/docs/guides/database/postgres/row-level-security

## Approval record

- **ChatGPT review:** Proposed — initial review 2026-11-26
- **Owner decision:** Pending
