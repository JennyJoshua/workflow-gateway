# ADR-010: Personal and Organisation Workspace Tenancy

- **Status:** Proposed — not owner-approved
- **Date:** 2026-11-26
- **Deciders:** Owner (final approval); ChatGPT (architecture review)
- **Related issues:** WG-002, WG-004, WG-007, WG-023, WG-027

## Context

The product serves both individual customers and organisations. A user may work alone, belong to several organisations, or administer an organisation. Identity must not be conflated with ownership of tenant data.

## Options considered

| Option | Advantages | Disadvantages | Cost/free-tier implications | Lock-in and exit considerations |
|---|---|---|---|---|
| Shared database with workspace/tenant key and RLS | Efficient early architecture; supports personal and organisation workspaces; standard PostgreSQL | Requires rigorous policy design and cross-tenant testing | No separate database cost per tenant; database capacity and hosting still matter | Low-to-moderate; standard relational schema is portable |
| Separate database per organisation | Stronger physical separation and independent data lifecycle | Operationally complex; migrations, backups and connections multiply | Higher infrastructure and maintenance cost | More infrastructure coupling; portable but costly to operate |
| User-owned records only, add organisations later | Simplest initial schema | Retrofitting organisation memberships and shared ownership can be disruptive | Lowest early complexity | High future schema migration cost |

## Proposed decision

Use a **workspace-based tenancy model**:
- Each user has a personal workspace.
- An organisation workspace has members and role-based permissions.
- A user may belong to multiple workspaces.
- Tenant-owned data is scoped to a workspace/tenant ID.
- The initial logical design may use a shared PostgreSQL database with RLS as defence in depth, subject to threat-model and RLS validation.
- Keep the possibility of dedicated databases for future enterprise/data-residency needs without implementing that complexity in v1.

Do not use a user ID as the only tenant boundary. Do not assume authentication implies permission to access every workspace.

## Security, privacy and data residency

- Check membership, role and workspace status server-side on every access.
- RLS policies must enforce workspace boundaries for exposed data.
- User-editable profile metadata must not grant roles or privileges.
- Define organisation ownership transfer, member removal, workspace closure, data export and deletion.
- Test cross-workspace reads and writes with negative tests.
- Define retention and deletion behaviour for personal and organisation workspaces separately.

## Consequences

### Positive
- Supports the approved individual and organisational customer segments.
- A single identity can safely participate in several organisations.
- Keeps a path open for future enterprise isolation.

### Negative / trade-offs
- Requires careful workspace context handling throughout API and database layers.
- RLS and membership logic need substantial negative testing.
- Organisation lifecycle and data deletion become explicit product requirements.

### Operational implications
- WG-004 must define workspace, membership, role and tenant keys.
- WG-023 must verify cross-tenant isolation.
- WG-027 builds on the same membership model.

## Validation plan

1. Draw personal and organisation workspace data flows.
2. Test a user belonging to two organisations plus a personal workspace using synthetic data.
3. Define access behaviour after member removal and organisation closure.
4. Prototype RLS policies only after database implementation is approved.

## References

- https://supabase.com/docs/guides/database/postgres/row-level-security
- https://www.postgresql.org/docs/current/ddl-rowsecurity.html

## Approval record

- **ChatGPT review:** Proposed — initial review 2026-11-26
- **Owner decision:** Pending
