# Development and Testing Standards

> Status: **proposed** (WG-005); the CI approach in section 6 was approved by the owner on 27 November 2026. Planning document: it governs future implementation work and authorises none. Tooling named below is a **candidate that requires validation** (see the technology ADRs under WG-002); nothing here provisions a service or adds a workflow. The owner must approve the CI approach before any workflow file is added.

## 1. Principles

1. **Decisions before code.** Implementation issues start only after their dependencies (ADRs, threat model, database specification) are reviewed and the owner approves the implementation handover.
2. **Free-first.** No paid service is subscribed to or upgraded without owner approval.
3. **Security by default.** Default deny, least privilege, tenant isolation, no secrets in the repository.
4. **Small, reviewable changes.** One issue, one pull request, one concern.
5. **Evidence over assertion.** An issue is closed only with the evidence its acceptance criteria require.

## 2. Branching and pull requests

- `main` is the only long-lived branch. Direct pushes to `main` are not part of the workflow; **the owner approves every merge to `main`**. Branch protection on `main` was enabled on 27 November 2026 with owner approval.
- Branch names: `<type>/<issue-id>-<short-slug>`, for example `feat/wg-008-conversations`, `fix/wg-014-routing-fallback`, `docs/wg-005-standards`. Types: `feat`, `fix`, `docs`, `test`, `chore`, `security`.
- One pull request per issue where practical. The title starts with the issue ID, for example `WG-008: implement conversation management`. Link the issue; do not use closing keywords until the owner has accepted the work.
- Commit messages: imperative, present tense, issue ID in the body or title. No secrets, tokens or customer data in commits, diffs, screenshots or logs.
- Review: every pull request needs the checklist in `.github/pull_request_template.md` completed. ChatGPT performs independent architectural review of significant changes; the owner decides on merge.
- Changes that alter an approved decision follow [Contribution and Change Control](../project-management/CONTRIBUTION_AND_CHANGE_CONTROL.md) and require a new ADR.

## 3. Definition of done

A change is done only when all of the following are true:

- [ ] Every acceptance criterion in the issue is met and the evidence is linked in the pull request.
- [ ] Tests required by section 5 exist and pass; no test was weakened or skipped to pass.
- [ ] Tenant isolation: every new tenant-owned table, query, cache key and file path carries and enforces the workspace identifier, and a cross-tenant test exists.
- [ ] Authorisation is enforced server-side at the point of use, not only in the interface.
- [ ] Sensitive actions emit audit events; logs contain no prompts, outputs, credentials or personal data by default.
- [ ] Inputs and outputs are validated against a schema; errors are mapped to documented error codes.
- [ ] No secrets, keys or customer data are present in code, fixtures, history or documentation.
- [ ] Documentation is updated (affected specification, ADR status, issue register) and links resolve.
- [ ] Dependencies added are justified, licence-compatible and recorded.
- [ ] No unfinished workflow is exposed as a commercial capability.
- [ ] The owner has approved the merge.

## 4. Coding conventions (proposed)

Conventions are stack-neutral until the technology ADRs are accepted.

- Typed code throughout: static typing for the backend and TypeScript strict mode for the interface (if those candidates are accepted).
- Formatters and linters are mandatory and configured in the repository so local and CI results match.
- Layering: interface code never calls providers or the database directly; all access goes through backend services that enforce authentication, authorisation, entitlements, approvals and audit.
- Configuration comes from environment variables or a secret store; defaults are safe and local-only.
- Errors are explicit and never leak internals, provider payloads or credentials to the client.
- Public contracts (REST/OpenAPI, workflow metadata, adapter interface) are versioned; breaking changes need an ADR.

## 5. Test strategy

| Level | Purpose | Minimum expectation |
|---|---|---|
| Unit | Pure logic: routing policy, lifecycle rules, permission checks, validators | Required for all new logic; fast and deterministic |
| Integration | Services with real storage or a faithful local equivalent | Required for persistence, authentication flows, adapters |
| Contract | Schemas and APIs stay compatible (OpenAPI, workflow metadata, adapter interface) | Required for every public contract; fails on breaking change |
| End-to-end | Key user journeys through the interface | A small set per milestone, not exhaustive |
| Security | Cross-tenant access, authorisation bypass, injection, secret leakage, approval bypass | Required for WG-007, WG-020, WG-022, WG-023, WG-030; traced to the threat model (WG-003) |
| Performance | Latency, concurrency, cost under expected load | Baselines before pilot (WG-030) |

Rules:

- Bug fixes include a regression test that fails before the fix.
- Tests must not call live provider or customer systems by default. External calls are stubbed or replayed from fixtures; any live test is opt-in, uses dedicated test credentials and is never run on pull requests from forks.
- Test data is synthetic. Real customer data is never used.
- Flaky tests are fixed or removed promptly, not retried indefinitely.

### 5.1 Testing LLM-dependent behaviour

- Separate deterministic logic (routing policy, validation, approval gating) from model calls so most behaviour is testable without a model.
- Use recorded fixtures for model responses in unit and integration tests.
- Maintain a versioned **evaluation set** for intent analysis and routing (WG-010, WG-014): input, expected structured intent, acceptable routes, and cases that must never route to a write action without approval.
- Include adversarial cases: prompt injection in user text and tool output, tool poisoning, requests to reveal credentials or other tenants' data.
- Track evaluation results over time; a drop beyond an agreed threshold blocks the change. Thresholds are set by the owner on advice of the reviewer.
- Assertions on model output check structure and constraints, not exact wording.

## 6. Continuous integration (owner-approved approach)

No workflow is added by this document. The owner approved the following baseline on 27 November 2026; each workflow is still added in its own reviewed pull request:

1. Format, lint and type checks.
2. Unit, integration and contract tests.
3. Dependency vulnerability scanning and licence check.
4. Secret scanning on every push (the repository already has GitHub secret scanning and push protection enabled; keep both on).
5. Documentation link check.

Constraints: free-tier runners only; workflows use the minimum `permissions:` (read-only by default); third-party actions are pinned to a full commit SHA; no workflow can deploy, run hosted database migrations or use production secrets without a separate owner-approved, manually triggered, environment-protected workflow; CI tokens are least-privilege and short-lived.

## 7. Secret handling

- Never commit secrets, keys, tokens, `.env` files or customer data. `.gitignore` excludes environment files and key material; `.env.example` with placeholder values is allowed.
- Platform secrets live in a secret store approved by the owner; customer BYOK keys are encrypted, write-only and never logged (WG-020).
- If a secret is committed: treat it as compromised, revoke and rotate it immediately, then remove it from history only with owner approval. Deleting the commit alone is not sufficient.
- Any suspected exposure is reported to the owner at once and recorded in the audit trail.

## 8. Dependencies and supply chain

- Prefer well-maintained, permissively licensed packages; record each new dependency and why it is needed.
- Lock files are committed; automated update proposals (for example Dependabot pull requests) are reviewed like any other change.
- Alerts for known vulnerabilities are monitored; critical findings block release.

## 9. Environments and data

- Local and test environments use synthetic data and non-production credentials.
- Production deployment, hosted database migrations and paid services require explicit owner approval and are out of scope until separately authorised.
- Migrations are reviewed as code, are reversible where possible, and are rehearsed on a disposable copy before any hosted run.

## 10. Worked example: applying these standards to WG-008 (conversation and message management)

1. Branch `feat/wg-008-conversations`; pull request titled `WG-008: implement conversation management`.
2. Unit tests: message ordering, title generation rules, permission checks (owner, admin, member).
3. Integration tests: create, list, rename, delete (soft delete) conversations; pagination; retention rules from the database specification.
4. Security tests: a user in workspace A cannot list, read, write or delete workspace B's conversations by any route, including by guessing identifiers; logs contain no message content.
5. Contract test: conversation endpoints match the documented schema; breaking change fails.
6. Definition-of-done check: audit events for deletion and export, workspace identifier on every table and query, documentation and issue register updated, owner merge approval recorded.

The example is actionable with the checklist above; gaps found when applying it to a real issue are corrected in this document through a normal pull request.

## 11. Open decisions

- Final CI tooling choices per component (the overall approach is owner-approved; workflows arrive with the scaffold pull request).
- Evaluation-set ownership, thresholds and review cadence.
- Coverage targets (proposed: set per component after the first implementation batch rather than a global number now).
- Required reviews and required status checks on `main`: protection is enabled without them (see the branch protection proposal); revisit when CI checks are stable or a second maintainer joins.
