# ADR-005: Next.js as the Frontend Framework Candidate

- **Status:** Proposed — not owner-approved
- **Date:** 2026-11-26
- **Deciders:** Owner (final approval); ChatGPT (architecture review)
- **Related issues:** WG-002, WG-005, WG-006, WG-025

## Context

Workflow Gateway needs a responsive, accessible chat interface that works on mobile and desktop, supports authenticated sessions, streams responses, and later exposes workflow discovery, approvals, settings and usage.

## Options considered

| Option | Advantages | Disadvantages | Cost/free-tier implications | Lock-in and exit considerations |
|---|---|---|---|---|
| Next.js | React framework with server and static deployment choices; broad ecosystem | More complexity than a simple static SPA; server features require suitable hosting | Framework is open source; hosting and third-party services may cost | Low-to-moderate if standard Node.js/Docker deployment is used; avoid unnecessary provider-specific features |
| React + Vite SPA | Simpler client-side app and static hosting; smaller deployment surface | Backend owns more routing/session/rendering concerns; SSR not built in | Framework/build tools are free; static hosting can be free within limits | Low lock-in; standard browser assets |
| Other full-stack framework | May provide a simpler integrated experience | Requires separate ecosystem and deployment comparison | Depends on framework and host | Depends on framework-specific conventions |

## Proposed decision

Retain Next.js as the **preferred frontend candidate**, with mobile-first responsive design. Use standard Node.js or Docker deployment as the portability baseline. A static export may be suitable for a public informational shell but should not be assumed to support all authenticated and streaming application needs.

## Security, privacy and data residency

- Never place provider secrets or privileged database keys in browser bundles.
- Treat all model, workflow and tool output as untrusted.
- Use secure session handling and server-side authorisation.
- Protect against XSS, CSRF where applicable, unsafe redirects and malicious Markdown/HTML.
- Keep sensitive actions behind backend permission checks and approval gates.

## Consequences

### Positive
- Supports a rich chat UI and responsive application structure.
- Can deploy to a Node.js server or Docker host, not only one vendor.
- Can evolve from a static shell to server-rendered features if needed.

### Negative / trade-offs
- Requires care around server/client boundaries and browser exposure of environment variables.
- Adds framework complexity compared with a small SPA.

### Operational implications
- Use a separate backend API contract; do not place business authorisation solely in frontend routes.
- Include accessibility, mobile viewport and streaming/error-state tests.

## Validation plan

1. Compare a small chat screen prototype with React + Vite before selecting the final UI architecture.
2. Verify streaming and authentication behaviour on the intended host.
3. Test a standard Node.js or Docker deployment path.
4. Confirm that required Next.js features do not depend on one hosting vendor.

## References

- https://nextjs.org/docs/app/getting-started/deploying
- https://nextjs.org/docs/pages/guides/self-hosting

## Approval record

- **ChatGPT review:** Proposed — initial review 2026-11-26
- **Owner decision:** Pending
