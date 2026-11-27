# Contribution and Change Control

## Roles

| Role | Responsibility |
|---|---|
| Owner (JennyJoshua) | Final authority. Approves merges to main, production deployments, destructive actions, paid services and upgrades, material scope changes and hosted database migrations. |
| ChatGPT — Chief AI Architect and Independent Reviewer | Strategic architecture oversight; reviews technical decisions, implementation plans and completed work; identifies architectural and security weaknesses; recommends changes. |
| Claude — GitHub setup operator (this handover) | Established the repository, documentation, milestones and issues. Provides implementation handovers when requested. Has no standing authority to redesign or change the architecture. |

## Approval boundaries

The owner must explicitly approve:

1. Main-branch merges.
2. Production deployments.
3. Destructive actions (deleting data, repositories, branches or history).
4. Paid services and upgrades of existing services.
5. Material scope changes.
6. Hosted database migrations.

Routine documentation and repository setup within an approved handover are authorised. **Software implementation is not authorised by the setup handover**; each implementation batch needs an explicit owner approval and handover.

## Change control process

1. Raise or update a GitHub issue describing the change and its reason.
2. If an approved decision is affected, write or amend an ADR (docs/decisions/). Approved decisions are never changed silently.
3. ChatGPT reviews the proposal; the owner approves material changes.
4. Make the change on a branch and open a pull request; the owner approves the merge to main.
5. Update affected documents together so they stay consistent, and update the issue register and milestones.

## Issue rules

- Issue titles keep their exact WG-NNN identifier.
- An issue is closed only when its acceptance criteria are met and reviewed, never because its description has been written.
- Issues are not assigned to people without authorisation.
- Labels: `priority:*`, `type:*`, `phase:*` are applied to every issue.

## Main branch protection

The proposed protection policy is documented in [Branch Protection Proposal](BRANCH_PROTECTION_PROPOSAL.md). It is not enabled. The owner must explicitly approve the exact settings before any GitHub branch-protection or ruleset changes are made. The proposal accounts for the current single-maintainer arrangement and the absence of validated CI status checks.

## Security rules

- No secrets, keys or customer data in the repository (see .gitignore and docs/SECURITY_AND_GOVERNANCE.md).
- Free-first development: no paid service is subscribed to or upgraded without owner approval.

## Licensing (open decision)

No licence has been chosen. The LICENSE file is a placeholder stating all rights are reserved. The owner must decide on a licensing approach appropriate to a proprietary commercial product. Implications while unresolved: the public repository can be viewed and forked on GitHub, but no other reuse rights are granted, and outside contributors cannot clearly contribute. Alternatively, the repository could be made private.

## Related documents

- [Security and Governance](../docs/SECURITY_AND_GOVERNANCE.md)
- [ADR-001](../docs/decisions/ADR-001-OVERALL-PLATFORM-DIRECTION.md)
- [Milestones](MILESTONES.md) and [Issue Register](ISSUE_REGISTER.md)
