# ADR-009: MCP as a Supported Tool Integration Protocol

- **Status:** Proposed — not owner-approved
- **Date:** 2026-11-26
- **Deciders:** Owner (final approval); ChatGPT (architecture review)
- **Related issues:** WG-002, WG-003, WG-021, WG-022, WG-023

## Context

The platform will connect external tools and systems. Model Context Protocol (MCP) provides a standard way for hosts, clients and servers to exchange context and expose tools. The platform must decide whether MCP is a universal integration mechanism or one supported adapter among several.

## Options considered

| Option | Advantages | Disadvantages | Cost/free-tier implications | Lock-in and exit considerations |
|---|---|---|---|---|
| MCP adapter plus direct API adapters | Standard interface where supported; direct APIs remain available for specialised integrations | Requires secure MCP client/server lifecycle and protocol-version management | Protocol itself has no platform subscription; individual servers/providers may have costs | Low protocol lock-in; tool schemas and server-specific behaviour still need adapters |
| Direct API integrations only | Full control over each integration and narrower trust boundary | Repeated connector work and inconsistent conventions | No protocol fee; engineering effort scales with each integration | Low protocol lock-in, higher maintenance |
| MCP-only integrations | Consistent tool interface for compatible servers | Cannot cover every system; protocol does not solve platform entitlements or business workflow semantics | No protocol fee; server operation may cost | Risk of over-reliance on a protocol ecosystem |

## Proposed decision

Support MCP as **one integration protocol**, not the only integration method and not the proprietary workflow contract. Keep direct API adapters for cases where they are simpler, more reliable or provide better control.

All MCP tools must pass through the same platform policy, tenant isolation, entitlement, approval, output validation, audit and rate-limit layers as direct tools.

## Security, privacy and data residency

- Treat tool descriptions, annotations and returned content as untrusted unless independently verified.
- Require explicit customer consent for data access and tool invocation.
- Restrict tools by tenant and least-privilege scopes.
- Authenticate remote servers and validate transport and origin.
- Apply timeouts, output limits, schema validation and outbound host allow-lists.
- Require approval for consequential writes or external side effects.
- Never allow MCP server instructions to override platform policy or system controls.

## Consequences

### Positive
- Supports interoperability with an expanding tool ecosystem.
- Avoids building every connector from scratch.
- Direct APIs remain a fallback and can provide tighter control.

### Negative / trade-offs
- Adds protocol lifecycle, compatibility and server trust management.
- MCP protocol support alone does not provide safe authorisation or business-level workflow contracts.

### Operational implications
- Pin supported protocol versions and test compatibility.
- Maintain an allow-listed server catalogue and permission manifest.
- Review each server independently before enabling it for customers.

## Validation plan

1. Select one read-only test server and one controlled write-capable test server.
2. Test authentication, consent, permission scopes, timeouts, output limits and audit events.
3. Run prompt-injection and malicious-tool-description scenarios.
4. Compare a direct API adapter with MCP for the same integration.

## References

- https://modelcontextprotocol.io/specification/2026-07-28
- https://modelcontextprotocol.io/specification/2026-07-28/basic

## Approval record

- **ChatGPT review:** Proposed — initial review 2026-11-26
- **Owner decision:** Pending
