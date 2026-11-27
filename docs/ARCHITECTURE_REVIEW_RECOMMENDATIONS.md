# Architecture Review Recommendations

> **Status: proposed for owner review. These are architecture recommendations from the independent review, not accepted decisions and not implementation authorisation.**
>
> Related work: WG-002 (ADRs), WG-003 (threat model), WG-004 (database specification), WG-011 (workflow contract), WG-016 (asynchronous execution), WG-020 (BYOK), WG-023 (tenant isolation).

## Purpose

The existing product and technical documents establish a strong direction. The following refinements should be made explicit before the database specification and technology decisions are finalised.

## 1. Separate personal accounts from organisation workspaces

**Recommendation:** Support a personal workspace for an individual customer and organisation workspaces with memberships and roles. A user may belong to multiple organisations without duplicating their identity.

- Keep user identity separate from tenant-owned business data.
- Model organisation membership, role and status explicitly.
- Treat personal workspace data and organisation workspace data as distinct security scopes.
- Make every tenant-owned record carry a workspace/tenant scope, not rely only on a user ID.
- Define ownership transfer, member removal and organisation deletion behaviour.

**Why:** The product serves both individuals and organisations. A single organisation-only tenant model would create unnecessary friction for individual customers and could make later organisation support harder.

**Related:** WG-004, WG-007, WG-023, WG-027.

## 2. Treat customer API keys as high-value secrets

**Recommendation:** Use envelope encryption or an equivalent managed key-encryption design. Store only encrypted credential material or a reference to an external secret store; never expose plaintext keys to the browser, ordinary application logs or workflow outputs.

- Encrypt each tenant's credentials with tenant-scoped data-encryption keys.
- Protect key-encryption keys separately using a suitable key-management mechanism.
- Decrypt only server-side, immediately before an authorised provider call.
- Support replace/rotate, revoke and delete operations.
- Record access metadata without recording the secret itself.
- Ensure backups, exports, support tooling and error traces cannot leak keys.

**Why:** BYOK is a core product requirement and a compromise could expose customer accounts and create financial harm.

**Related:** WG-003, WG-020, WG-023.

## 3. Make proprietary workflows independent of the chat application

**Recommendation:** Define a stable, versioned workflow API contract. The chat interface should be one client of the workflow execution platform, not the only way workflows can be invoked.

- Define workflow identity, version, capability metadata and JSON input/output schemas.
- Standardise authentication, tenant context, entitlements and permission checks.
- Standardise synchronous and asynchronous execution responses.
- Standardise idempotency, callbacks/polling, error shapes and audit events.
- Keep workflow adapters behind a stable internal interface.

**Why:** The user's proprietary workflows are the commercial engine. They should be sellable as standalone products, callable by other applications, and reusable in other channels without rebuilding them inside the chat UI.

**Related:** WG-011, WG-015, WG-016, WG-028.

## 4. Design asynchronous execution as a first-class capability

**Recommendation:** Use durable job records and a queue/worker execution model for long-running workflows. Do not rely on a single web request remaining open until work completes.

- Persist the execution request before dispatch.
- Use explicit states: queued, running, awaiting approval, succeeded, failed, cancelled and timed out.
- Use idempotency keys to prevent duplicate execution.
- Make retries bounded and policy-controlled.
- Persist execution events and expose progress to the chat timeline.
- Support cancellation and recovery after worker or host restarts.
- Treat callbacks as untrusted and authenticate them.

**Why:** GitHub workflows, multi-step automations and external tools can take longer than normal web requests. Durable execution improves reliability and user visibility.

**Related:** WG-015, WG-016, WG-017, WG-022.

## 5. Separate AI routing from authorisation

**Recommendation:** The AI router may recommend a workflow, model or tool, but a deterministic backend policy layer must independently authorise every action immediately before execution.

- Check identity, workspace membership, role, entitlement and workflow lifecycle.
- Check tool scopes, data classification, customer policy and spending limits.
- Enforce approval gates server-side.
- Re-check permissions after an approval if the underlying context has changed.
- Do not allow prompt content, model output or a routing decision to grant permissions.
- Record the decision, policy version, approver and execution outcome.

**Why:** An LLM is probabilistic and its output can be influenced by prompt injection. It must not become the authority for access control.

**Related:** WG-003, WG-014, WG-022, WG-023.

## Recommended disposition

1. Owner reviews these five recommendations.
2. ChatGPT converts accepted recommendations into ADRs or explicit requirements.
3. WG-003 maps the associated threats and mitigations.
4. WG-004 uses the accepted tenancy, credential and execution requirements to define the data model.
5. No technology, hosted service or schema is finalised until its ADR has evidence and owner approval.

## Not authorised by this document

This proposal does not authorise implementation, service provisioning, credential creation, database migrations, paid subscriptions, changes to approved product direction or a change to the current backlog scope.
