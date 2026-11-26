# ADR-003: Render as an Application Hosting Candidate

- **Status:** Proposed — not owner-approved
- **Date:** 2026-11-26
- **Deciders:** Owner (final approval); ChatGPT (architecture review)
- **Related issues:** WG-002, WG-005, WG-016, WG-030

## Context

The platform needs hosting for the API, web frontend and eventually durable background workers. The project is free-first, but long-running executions and customer-facing reliability cannot depend on a sleeping or ephemeral free service.

## Options considered

| Option | Advantages | Disadvantages | Cost/free-tier implications | Lock-in and exit considerations |
|---|---|---|---|---|
| Render | Git-based deployment, supports Node/Python and containers; managed web services and data stores | Free web services spin down, have ephemeral filesystems and limited resources; free instances are explicitly not for production | Free web services are available for experimentation; paid compute starts at listed monthly prices; background workers are not among the free service types described in the free deployment guide | Low-to-moderate lock-in when using standard containers, environment variables and PostgreSQL |
| Self-hosted container/VPS | Maximum control and portability | User must manage OS, TLS, restarts, monitoring, patching, backups and scaling | Software can be free; server and operational time are not | Low platform lock-in, high operational burden |
| Another managed container platform | Could offer different free/low-cost runtime characteristics | Additional platform evaluation and migration effort | Current limits and pricing require a separate comparison | Depends on proprietary deployment features |

## Proposed decision

Keep Render as the **preferred prototype hosting candidate**, not as an approved production platform. Design deployables as standard containers or conventional Node/Python services so they can move to another host.

Do not place a production API or durable job worker on a free Render instance. Do not rely on local filesystem persistence. Do not start a paid service without explicit owner approval.

## Security, privacy and data residency

- Store secrets only in protected service configuration, never in repository files or build output.
- Confirm region availability and data-processing terms before storing customer data.
- Keep the database separate from ephemeral application filesystems.
- Apply network restrictions and outbound allow-lists where available.
- Define service health checks, restart behaviour, log redaction and deployment rollback.

## Consequences

### Positive
- Simple repository-connected deployment path for prototypes.
- Supports Node.js, Python and container-based services.
- Standard containers reduce migration cost.

### Negative / trade-offs
- Free web services sleep after 15 minutes without inbound traffic and may take about a minute to wake.
- Free services have ephemeral filesystems and can restart at any time.
- Free Render Postgres databases expire after 30 days.
- A reliable worker architecture will likely require paid compute or a different free-compatible execution model.

### Operational implications
- Keep execution state in durable external storage.
- Separate web/API and worker processes.
- Reassess hosting when pilot reliability and expected usage are defined.

## Validation plan

1. Test a non-production service using synthetic data.
2. Test cold-start, restart and deploy behaviour.
3. Test container portability outside Render.
4. Compare the cost of always-on API and worker services before production.
5. Confirm region and data-processing terms.

## References

- https://render.com/docs/free
- https://render.com/pricing

## Approval record

- **ChatGPT review:** Proposed — initial review 2026-11-26
- **Owner decision:** Pending
