# ZITADEL Verification Spike (plan)

> **Status: approved by the owner on 28 November 2026; not yet run.** Purpose: settle the unverified points in [Identity Provider Options](../IDENTITY_PROVIDER_OPTIONS.md) before Konam decides (integration specification, open decision 1). Free account only. No paid service, production data, real customer, secret in chat, secret in the repository, or code in this repository is part of this spike.

## 1. What blocked me from running it

I cannot create the ZITADEL account myself because sign-up needs your email and verification. A self-hosted instance in my sandbox is not feasible (no Docker or Postgres, and under 500 MB of free memory) and would not show how the cloud free plan behaves. So the spike needs one short task from you, after which I can run most checks.

## 2. What you do (about 15 minutes, on a phone is fine)

1. Sign up for a **ZITADEL Cloud free** instance. Pick a region (Australia is the closest of the four offered). Do not enter payment details; stay on Free.
2. In the instance, create a project named `konam-spike`.
3. Create one application in that project that uses the **device authorization** flow (a public client, so there is **no secret**).
4. Create a test human user with a throwaway email and strong password that you do not use anywhere else.
5. Send me only: the **instance address (issuer URL)** and the **application client ID**. Both are public values. **Never send passwords, client secrets or keys.**
6. When I start a check, I will give you a short code. Open the link on your phone, sign in as the test user and approve it.

When the spike ends, delete the test instance.

## 3. Checks I will run and what counts as a pass

| # | Check | Pass criterion |
|---|---|---|
| T1 | Discovery document | Lists authorisation, token, JWKS, revocation, introspection and device-authorisation endpoints |
| T2 | Sign in through the device flow | Tokens are issued; access-token lifetime is short by default or configurable to short |
| T3 | Audience | The access token can carry the project as its audience, so Konam can reject tokens meant for something else |
| T4 | Client identification | A token claim identifies the calling client, so Gateway-originated calls are distinguishable in audit |
| T5 | Refresh behaviour | Refresh tokens rotate or have a bounded life; recorded because it decides whether OD-3 holds |
| T6 | Revocation | A revoked token is reported as inactive by introspection. This needs an API application credential, so it is **optional** and only with a throwaway secret you are comfortable sharing and then deleting |
| T7 | Token exchange (beta) | After enabling the feature, a token can be narrowed in scope and audience, and a request to add a new audience is refused. Findings are informational because v1 must not depend on it |
| T8 | Usage counting | The instance usage view shows how daily active users are counted and confirms the free limit and region |

## 4. Decision rule

- **ZITADEL stays the recommendation** if T1 to T5 pass and T8 confirms the 100 daily active user limit comfortably covers the pilot.
- **Re-open the comparison** (Keycloak, or a paid option approved by the owner) if T3 or T4 fail, if T5 forces a server-held session design the owner rejects, or if the free limits prove too small.
- T6 and T7 inform risk but do not decide the outcome alone.

## 5. Results

Not yet recorded. When the checks are run, results go in a pull request that updates this file, with no tokens, codes, secrets or personal data included.

## 6. Non-authorisation

This plan does not create an account, register an application in any production system, select the provider, or change the Gateway or Konam decisions. The final identity-provider choice belongs to Konam and needs the owner's approval.
