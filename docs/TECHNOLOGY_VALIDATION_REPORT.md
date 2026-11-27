# Technology Validation Report

> Status: **evidence for owner review** (supports WG-002). Gathered 27 November 2026 from vendor documentation and, where noted, third-party reporting. It does **not** accept any ADR, provision any service or authorise implementation. Vendor limits change; re-verify against the linked source before committing money or data.

## Summary

| ADR | Candidate | Validation result | Effect on the proposal |
|---|---|---|---|
| [ADR-003](decisions/ADR-003-SUPABASE-DATABASE-AUTH.md) | Supabase | Free plan fits **development and demos only** | Pilot with real customer data needs a paid plan (owner approval) |
| [ADR-004](decisions/ADR-004-RENDER-HOSTING.md) | Render | Free tier cannot run workers or cron; free Postgres expires | Do not use Render Postgres; free web service is demo-only |
| [ADR-005](decisions/ADR-005-FASTAPI-BACKEND.md) | FastAPI | **Not validated in this pass** | Spike still required |
| [ADR-006](decisions/ADR-006-NEXTJS-FRONTEND.md) | Next.js | Current, MIT, formal security-release process | Pin versions and adopt an update cadence |
| [ADR-007](decisions/ADR-007-LITELLM-MODEL-GATEWAY.md) | LiteLLM | MIT core; governance features are paid | Keep a thin in-house abstraction; LiteLLM stays optional |
| [ADR-008](decisions/ADR-008-LANGGRAPH-ORCHESTRATION.md) | LangGraph | **Not validated in this pass** | No change; custom orchestration first stands |
| [ADR-009](decisions/ADR-009-MCP-TOOL-INTEGRATION.md) | MCP | Authorization is OAuth 2.1 based and still evolving | Pin a spec revision; align the external-agent design |
| ADR-010 to ADR-012 | Internal decisions | No external validation needed | ADR-011 (job queue) affected by the Render finding |

## Supabase (ADR-003)

Sources: [pricing](https://supabase.com/docs/pricing), [pausing FAQ](https://supabase.com/kb/faqs/platform/free-project-pausing), [database-only cost FAQ](https://supabase.com/kb/faqs/platform/database-only-cost).

- Free plan: 500 MB database per project (shared CPU, 500 MB RAM), 2 active projects, 50,000 monthly active users, 1 GB file storage, 5 GB egress, community support.
- Free projects **pause after one week of low activity**; paused projects are not counted against the two-project limit. Pro projects are never paused.
- The pricing table lists automatic backups and point-in-time recovery as **not included** on Free.
- Pro is $25 per month per organisation before extra usage, with daily backups kept for 7 days.

**Implications.** Acceptable for development and demonstrations. Not acceptable for a pilot holding customer data: no backups and automatic pausing conflict with the retention, recovery and availability requirements in the [database specification](DATABASE_SPECIFICATION.md) and [threat model](SECURITY_THREAT_MODEL.md). Moving to Pro is a **paid-service decision for the owner**. Row-level-security fit and exit cost still need a hands-on spike.

## Render (ADR-004)

Source: [Render free-tier documentation](https://render.com/docs/free).

- Free web services spin down after 15 minutes without inbound traffic; spin-up takes about one minute.
- 750 free instance hours per workspace per month; when exhausted, free web services are suspended until the next month.
- Filesystem is ephemeral; free services cannot attach a persistent disk, run one-off jobs, scale beyond one instance, or receive private-network traffic.
- Only web services, Postgres, Key Value and static sites have free instances; other service types (for example background workers and cron jobs) do not.
- Free Postgres: 1 GB, **expires 30 days after creation** (14-day grace, then deleted), no backups, no managed connection pooling, only one per workspace.

**Implications.**
1. Do **not** use Render Postgres for anything durable. Supabase (or another approved database) holds data.
2. A free web service is a demo host only. It cannot run the always-on worker implied by [ADR-011](decisions/ADR-011-DURABLE-JOB-QUEUE.md); for demos the queue must be database-backed with work claimed by the web process, and production workers need a paid instance (owner approval).
3. Cold starts of about one minute affect the chat experience; acceptable for demos, not for a pilot.

## LiteLLM (ADR-007)

Sources: [LiteLLM enterprise documentation](https://docs.litellm.ai/docs/enterprise.md); third-party comparisons (for example [Vercel](https://vercel.com/i/litellm-vs-openrouter), [TrueFoundry](https://www.truefoundry.com/blog/litellm-enterprise)). Third-party claims below are marked and should be confirmed with the vendor.

- Open-source core under the MIT licence: one OpenAI-compatible API over many providers, virtual keys, budgets and rate limits, spend tracking.
- Vendor documentation lists SSO and SCIM, audit logs and delegated admin roles under the paid Enterprise licence. Third parties report SSO is free for up to 5 users, and that JWT auth and key rotation also sit behind the licence (third-party, unverified).
- Running the proxy means operating extra infrastructure (database and caching) ourselves.

**Implications.** Our own tenancy, BYOK, audit and metering requirements (WG-020, WG-023, WG-024) must live in our application regardless, so we should not depend on Enterprise-only features. Keep a thin in-house provider abstraction (WG-009) with LiteLLM as an optional library or adapter, as ADR-007 already proposes. Avoid running the LiteLLM proxy in v1.

## MCP (ADR-009)

Sources: [MCP authorization specification](https://modelcontextprotocol.io/specification/latest/basic/authorization) and the [2026-07-28 revision](https://modelcontextprotocol.io/specification/2026-07-28/basic/authorization).

- Authorization is optional. For HTTP transports, implementations SHOULD follow the specification; STDIO transports SHOULD NOT and take credentials from the environment.
- A protected MCP server acts as an OAuth 2.1 resource server and MUST publish OAuth 2.0 Protected Resource Metadata (RFC 9728). Clients MUST send resource indicators (RFC 8707).
- Client registration: Client ID Metadata Documents are preferred; pre-registration is allowed; Dynamic Client Registration is **deprecated** and kept for compatibility.
- Scope handling follows least privilege, with step-up authorisation when more scopes are needed. The specification is still changing between revisions.

**Implications.** The OAuth 2.1 with PKCE direction in the [External AI Agent Integration Specification](EXTERNAL_AI_AGENT_INTEGRATION_SPECIFICATION.md) is consistent with MCP. ADR-009 should pin a specific specification revision, plan for Client ID Metadata Documents rather than relying on Dynamic Client Registration, and treat per-client compatibility as something to verify.

## Next.js (ADR-006)

Sources: [nextjs.org](https://nextjs.org) and its [blog index](https://nextjs.org/blog/llms.txt).

- Latest stable release listed: 16.4.0; MIT licensed.
- Since July 2026 the project follows a formal, pre-announced security-release process, with scheduled monthly releases; the August 2026 release addressed a critical-severity vulnerability.

**Implications.** Pin exact versions, subscribe to the advisories, and add a security-update cadence to the [development standards](DEVELOPMENT_AND_TESTING_STANDARDS.md). Hosting for the interface was not evaluated here.

## Not validated in this pass

- **FastAPI (ADR-005)** and **LangGraph (ADR-008):** no evidence gathered; a spike is still required.
- **Supabase row-level-security fit, exit cost and data-residency/region choice** for the target market.
- **Model-provider terms** (data handling, retention) for each provider before it is enabled.
- **Vercel or alternative hosting** for the Next.js interface.

## Free-first environment recommendation

For the first code batch, no hosted service is needed. Tests run on GitHub-hosted runners against a throwaway Postgres container created for each run, and migrations are applied only to that disposable database. Hosted Supabase or Render accounts are created only when a demo is needed and then only with owner approval. Adding the workflow that does this requires owner approval of the CI approach (WG-005).

## Cost triggers to flag to the owner

1. Pilot with real customer data: Supabase Pro (about $25 per month, per vendor page) for backups and no pausing.
2. Always-on backend or workers: a paid Render instance, or an alternative host.
3. Governance features in LiteLLM Enterprise: not planned; avoid dependency on them.
