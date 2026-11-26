# Security and Governance

> Status: planning. Controls below are requirements for future implementation. The formal threat model is WG-003.

## Formal threat model

The detailed design-time analysis, trust boundaries, STRIDE and AI-specific threats, mitigation mapping, approval policy and tabletop scenarios are documented in [Security Threat Model](SECURITY_THREAT_MODEL.md). It is a draft for review; controls are requirements, not implemented guarantees. WG-003 remains open until the owner/reviewer accepts the document and its acceptance criteria are verified.

## Security principles

Default deny; least privilege; defense in depth; tenant isolation everywhere; secrets never in code, logs or clients; every sensitive action audited; humans approve high-risk actions.

## Customer authentication and permissions

- Authentication provider is a proposed candidate (for example Supabase Auth) requiring validation and owner approval.
- Role-based permissions (owner, admin, member) enforced server-side; entitlements gate workflows, models and tools.
- Authorisation is checked at discovery, routing and execution, not only in the interface.

## Tenant isolation

Organisation identifier on all tenant-owned data, enforced in the application and (proposed) at the database layer. Cross-tenant access must be provably impossible in tests (WG-023, WG-030).

## BYOK and secrets

Customer keys are encrypted at rest, write-only, used server-side only for the owning tenant, rotatable and revocable, and never logged (WG-020). Platform secrets are held in a secret store, never in the repository.

## Privacy

Data minimisation; defined retention and deletion; no prompts or outputs in logs by default; provider data-handling terms reviewed before a provider is enabled.

## AI-specific threats

Prompt injection, tool poisoning, over-privileged agents, data exfiltration via tools, malicious plugins or MCP servers, and unsafe output rendering. Mitigations: treat user and tool content as untrusted, constrain tool permissions, allow-list outbound hosts, validate schemas, sanitise output, and require approval for writes with real-world effect.

## Human approval gates

Execution pauses for human approval when policy flags an action as high risk, for example: write or destructive operations in external systems, repository or deployment changes, actions involving sensitive data, spending beyond limits, and any action a workflow marks as requiring approval. Approver, time, decision and context are recorded immutably (WG-022). Approval cannot be bypassed by routing or retries.

## Monitoring and audit logs

Audit log is append-only and records authentication events, permission changes, registry lifecycle transitions, routing decisions, approvals, credential access and configuration changes. Monitoring covers errors, latency, cost and abuse signals.

## Usage metering and future commercialisation

Usage events (model tokens, workflow executions, tool calls) are captured per tenant, separating platform-managed from BYOK use (WG-024). Plans, entitlements and quotas (WG-026) build on metering. Selecting a billing provider is a future decision requiring owner approval.

## Governance and approvals

The owner must explicitly approve: main-branch merges; production deployments; destructive actions; paid services and upgrades; material scope changes; hosted database migrations.

- **ChatGPT** is Chief AI Architect and Independent Reviewer: architecture oversight, review of decisions, plans and completed work, identification of weaknesses.
- **Claude** acted as GitHub setup operator for this handover and provides implementation handovers when requested, without standing authority to change the architecture.

## Repository security

Public repository: no secrets, credentials or customer data may be committed. `.gitignore` excludes environment files and key material. Secret scanning is to be established under WG-005.

## Not authorised in the current phase

Application code; Supabase provisioning; Render deployment; paid service configuration; production secrets; customer account connection; database migrations; external workflow integrations.

See also: [Technical Architecture](TECHNICAL_ARCHITECTURE.md), [Contribution and Change Control](../project-management/CONTRIBUTION_AND_CHANGE_CONTROL.md).
