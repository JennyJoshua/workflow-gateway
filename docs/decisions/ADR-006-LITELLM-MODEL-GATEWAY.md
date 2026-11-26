# ADR-006: LiteLLM as an Optional Model Gateway Candidate

- **Status:** Proposed — not owner-approved
- **Date:** 2026-11-26
- **Deciders:** Owner (final approval); ChatGPT (architecture review)
- **Related issues:** WG-002, WG-009, WG-019, WG-020, WG-024

## Context

The product must support platform-managed models and customer-provided API keys (BYOK) across multiple providers. A model gateway could centralise provider compatibility, retries, budgets and usage. However, a gateway must not become the only place where platform authorisation or customer policy is enforced.

## Options considered

| Option | Advantages | Disadvantages | Cost/free-tier implications | Lock-in and exit considerations |
|---|---|---|---|---|
| LiteLLM self-hosted gateway | OpenAI-compatible interface across many providers; routing, retries, budgets and usage features | Adds a service, database/configuration and operational surface; features vary by edition | Open-source core is self-hostable; enterprise features such as SSO/RBAC/audit may require paid licensing | Moderate configuration coupling; isolate behind the platform's own provider interface |
| Thin in-house provider adapter | Minimal dependencies and direct control over request flow, logs and BYOK handling | More provider integrations and maintenance owned by the project | No gateway licence cost; engineering effort is the cost | Low vendor lock-in; provider APIs still vary |
| Direct provider SDKs throughout app | Fastest small prototype | Provider-specific logic spreads across the codebase; difficult to govern and meter consistently | No gateway cost; model API usage still costs | Higher application-level coupling |

## Proposed decision

Do **not** make LiteLLM a mandatory platform dependency in the first implementation. Define and own a small internal model-provider contract first. Evaluate LiteLLM as an optional adapter/gateway when more than one application or team needs central budgets, key management, routing or usage reporting.

If adopted, keep the platform's tenant authorisation, entitlements, approvals and audit policy outside the gateway and enforce them in the backend.

## Security, privacy and data residency

- Confirm where prompts, responses, provider keys and request metadata are stored or logged.
- Disable or minimise prompt/response logging unless explicitly required and disclosed.
- Separate platform-managed credentials from customer BYOK credentials.
- Ensure virtual keys are not treated as a substitute for tenant identity and entitlements.
- Review the gateway's supply chain, deployment image, database access and admin API exposure.

## Consequences

### Positive
- Avoids premature infrastructure and operational complexity.
- Preserves a path to centralised provider management later.
- Keeps the product independent of a single gateway.

### Negative / trade-offs
- The initial provider adapter requires more project-owned code.
- Centralised budgets and provider administration may need to be built later if demand warrants them.

### Operational implications
- Record model usage in the platform's own usage events regardless of gateway choice.
- Keep model provider adapters contract-tested.

## Validation plan

1. Compare LiteLLM's current open-source and enterprise feature boundaries.
2. Test one platform-managed provider and one BYOK provider through the internal contract.
3. Test key redaction, usage capture, retries, timeouts and provider fallback.
4. Evaluate whether a separate gateway materially reduces maintenance before adopting it.

## References

- https://docs.litellm.ai/docs/
- https://docs.litellm.ai/docs/simple_proxy/
- https://docs.litellm.ai/docs/proxy/configuration/overview
- https://www.litellm.ai/

## Approval record

- **ChatGPT review:** Proposed — initial review 2026-11-26
- **Owner decision:** Pending
