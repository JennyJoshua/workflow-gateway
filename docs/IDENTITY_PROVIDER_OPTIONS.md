# Identity Provider Options (input to Konam open decision 1)

> **Status: evidence for the owner and for Konam, prepared 28 November 2026.** Konam owns platform identity, and the identity provider and token flow are an open Konam decision ([integration specification](https://github.com/Konam-Automation-Platform/automation-business-control-centre/blob/main/docs/CROSS_REPOSITORY_INTEGRATION_SPECIFICATION.md), open decision 1). This document does not choose a provider, create an account, change ADR-003 or authorise implementation. Vendor terms change; re-check each source before committing.

## 1. What the provider must do

Derived from the integration specification and the [Gateway-to-Konam contract proposal](KONAM_API_CONSUMER_CONTRACT_PROPOSAL.md):

| # | Requirement | Why |
|---|---|---|
| R1 | Standard OpenID Connect sign-in (authorisation code with PKCE) | Gateway edge validates Konam-issued identity |
| R2 | **Short-lived, audience-restricted delegated tokens** for Gateway calling the Konam API (token exchange or an equivalent) | The integration specification requires delegated, narrowly scoped user context |
| R3 | Revocation and introspection | Konam re-validates on every operation, so revocation must take effect quickly |
| R4 | A separate service identity for the Gateway | Gateway and user actions stay distinguishable in audit |
| R5 | MFA or passkeys | Needed before any customer pilot |
| R6 | Free-first, low operating burden | Free-first rule; the owner works mostly from a phone and GitHub |
| R7 | Standards-based and portable | Avoid lock-in; allow a later move |
| R8 | A region choice for stored identity data | Customer data-location expectations are unconfirmed |

Workspaces, membership and roles stay in Konam. The provider supplies identity, not workspace authority, so it does not need to model workspaces as organisations.

## 2. Evidence

### Keycloak (open-source, self-hosted)

Sources: [Keycloak 26.2 token exchange announcement](https://www.keycloak.org/2025/05/standard-token-exchange-kc-26-2), [26.8 release notes](https://www.keycloak.org/docs/26.8.2/release_notes/), [specifications implemented](https://www.keycloak.org/securing-apps/specifications).

- Standard token exchange (RFC 8693) is officially supported from 26.2; the vendor's compliance table describes the current implementation as covering the internal-to-internal case only.
- Release 26.8 adds token-exchange delegation for AI agents and automation as a **preview** feature, with audit events and client policy controls.
- Token revocation and introspection, PKCE, DPoP and SCIM are listed; the specification page marks SCIM as tech preview in one release and supported in another, so check the version in use.
- It is self-hosted software: there is no vendor-hosted free tier. It needs its own always-on host and database, which the free Render tier cannot provide (see the [validation report](TECHNOLOGY_VALIDATION_REPORT.md)). The software licence and its server resource needs were not verified in this pass.

### ZITADEL (cloud with a free plan; also self-hostable)

Sources: [pricing](https://zitadel.com/pricing), [pricing detail](https://zitadel.com/pricing/detail), [billing definitions](https://zitadel.com/docs/legal/service-description/billing), [April 2024 newsletter](https://zitadel.com/blog/zitadel-product-newsletter-april-2024).

- Free plan, US$0 per month: unlimited users and organisations, **100 daily active users**, 3 external identity providers, service users, all security features, and a data-residency choice of EU, US, Switzerland or Australia. Pro starts at US$100 per month with 25,000 daily active users.
- A daily active user is anyone who authenticates or refreshes a token that day; service users that authenticate also count.
- OAuth 2.0 Token Exchange (RFC 8693) was announced in April 2024 as a feature to enable, aimed at impersonation and delegation. Its current maturity and exact behaviour were **not verified** here.
- A self-hosted edition exists; its licence was not verified in this pass. Third-party pages quote conflicting free-tier user numbers; only the vendor's own figures are used above.

### Auth0 (software as a service)

Sources: [Auth0 pricing](https://auth0.com/pricing), [plan comparison](https://auth0.com/auth0-vs), [published pricing listing](https://auth0.com/llms-full.txt), [third-party summary dated 11 November 2026](https://ssojet.com/blog/auth0-free-tier-limits).

- Free plan: up to 25,000 monthly active users, 5 organisations, 1 enterprise connection, passkeys and passwordless sign-in, and one custom domain after credit-card verification.
- **Custom Token Exchange is not in Free or Essentials**; it appears from the Professional plan. Auth0's listing gives Professional at about US$240 per month (consumer) or US$800 per month (business) at 500 monthly active users.
- Free tenants have lower rate limits than paid tenants.

### Not evaluated

Supabase Auth, Clerk, Amazon Cognito, Microsoft Entra External ID, Authentik, Ory and Firebase were not researched. Supabase Auth in particular is **not** a candidate for platform identity under the ADR-003 clarification, because Konam, not the Gateway database provider, is authoritative for identity.

## 3. Comparison against the requirements

| Requirement | Keycloak | ZITADEL | Auth0 |
|---|---|---|---|
| R1 OIDC with PKCE | Yes | Standards-based (not separately checked) | Yes |
| R2 Delegated, audience-restricted tokens | Supported (standard exchange; delegation in preview) | Token exchange announced 2024; **needs verification** | **Paid plan only** |
| R3 Revocation and introspection | Documented | Not checked | Not checked |
| R4 Service identity | Standard client credentials | Service users, which count toward daily active users | Machine-to-machine tokens are a paid add-on at higher tiers |
| R5 MFA or passkeys | Yes | Included in free security features | Passkeys free; some MFA features paid |
| R6 Free-first, low ops | **Weak**: needs paid or self-managed hosting | **Strongest** for a small pilot | Free is generous, but delegation is paywalled |
| R7 Portability | Strong (open source) | Strong (standards, self-hostable) | Weaker (proprietary SaaS) |
| R8 Region choice | Wherever it is hosted | EU, US, Switzerland, Australia | Not checked |

## 4. Recommendation (for the owner's and Konam's decision)

1. **Shortlist ZITADEL Cloud Free as the development and small-pilot candidate.** It best fits free-first with minimal operations, offers the nearest-to-Papua-New-Guinea region of the four it lists (Australia), and is standards-based so Konam can leave later. Its limit is 100 daily active users, with service users counted.
2. **Keep Keycloak as the fallback** if token-exchange behaviour is insufficient in ZITADEL or if self-hosting becomes affordable. It has the clearest delegation support but no free hosting path.
3. **Do not choose Auth0 for v1** unless the owner approves a paid plan, because delegated tokens (R2) are not in Free.
4. **Run a verification spike before deciding** (no paid service): in a free ZITADEL test instance, prove that a user token can be exchanged for a short-lived, audience-restricted token for the Konam API, that revocation takes effect, that service identities are separable in audit events, and how daily active users are counted. Only the owner can approve creating the account.

## 5. Open points

- Who holds the provider account and billing (Konam owner decision).
- Whether 100 daily active users covers the pilot, including service users.
- Data-location expectations for customers (unconfirmed).
- SMS or phone sign-in cost and coverage for the target market (not researched); email, passkeys or authenticator apps avoid per-message cost.
- Konam must reflect the final choice in the contract's security scheme and in the Gateway database decision on sessions (OD-3, which currently assumes short-lived validated tokens).

## 6. Non-authorisation

This document creates no account, tenant, secret, integration or code and does not select a provider.
