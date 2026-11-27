# Main Branch Protection Proposal

> **Status: approved by the owner on 27 November 2026 and enabled on `main` with the rule set in section 3. Controls marked defer, later or optional were deliberately not enabled. Sections 5 and 6 below describe the original proposal and are retained for history.**
>
> Repository: `Konam-Automation-Platform/workflow-gateway` (transferred from `JennyJoshua/workflow-gateway`).

## 1. Purpose

Protect `main` as the stable, reviewed project baseline while allowing the owner to work with ChatGPT as Chief AI Architect and Independent Reviewer. This proposal is a governance recommendation, not authorisation to enable branch protection.

## 2. Current verified state

- Default branch: `main`.
- `main` is currently reported as **not protected** by the GitHub branch listing.
- The project has one known owner/operator and no confirmed second human maintainer.
- The project change-control policy requires an owner-approved merge to `main`.
- CI workflows and required status checks have not yet been established.

## 3. Proposed rules for `main`

| Control | Recommendation | Reason |
|---|---|---|
| Require a pull request before merging | Enable | Keeps changes reviewable and creates a durable change record. |
| Required approving reviews | Do not require a second GitHub approval at this stage | A second maintainer is not established. A mandatory independent GitHub reviewer could block the sole owner from merging. ChatGPT's independent review should be recorded in the PR discussion/checklist, followed by the owner's explicit merge approval. Revisit if a trusted human maintainer joins. |
| Require conversation resolution | Enable where available | Keeps review feedback addressed before merge. |
| Require status checks | Add only after CI checks exist and have been validated | Avoids selecting nonexistent or unreliable checks. |
| Require branches to be up to date | Consider later, together with stable required CI checks | Adds value once parallel PR activity and CI exist. |
| Restrict force pushes | Enable | Protects commit history. |
| Restrict branch deletion | Enable | Prevents accidental loss of the default branch. |
| Include administrators / enforce for administrators | Prefer enable, subject to confirming the owner can still merge an approved PR | Makes the policy meaningful for the repository owner as well. |
| Require signed commits | Defer | Adds setup friction; reconsider when contributor and release-signing practices are established. |
| Require linear history | Optional; defer | Not essential to the current planning phase. |

## 4. Required operating procedure

1. Make changes on a named topic branch.
2. Open a pull request targeting `main`.
3. ChatGPT reviews the change for scope, architecture, security, consistency and acceptance criteria.
4. Record review findings and resolve material issues in the PR.
5. The owner explicitly approves the merge.
6. Merge only after required checks (once established) pass.
7. Update the relevant GitHub issue and project documentation.

A ChatGPT review is an independent review step, not a GitHub account approval and not a substitute for the owner's merge decision.

## 5. Rollout sequence

- **Now:** retain this proposal in the repository; do not alter branch settings.
- **After repository transfer:** confirm the destination repository and the owner's effective permissions.
- **Before enabling:** the owner reviews and explicitly approves the exact rule set.
- **After approval:** enable the rules on the canonical repository, test a normal pull-request path, and verify that the owner can complete an approved merge.
- **Later:** add required CI checks only after their names and behaviour are stable; revisit review requirements if another trusted maintainer is added.

## 6. Explicit exclusions

This proposal does not authorise:
- Enabling or changing branch protection or repository rulesets.
- Creating CI workflows or required status checks.
- Adding collaborators or changing repository roles.
- Changing repository visibility, licence or ownership.
- Merging any branch into `main`.

## 7. Review trigger

Revisit this proposal after the organisation transfer, when CI is introduced, or when the project gains another trusted maintainer.
