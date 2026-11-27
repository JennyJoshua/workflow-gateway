# Workflow Registry Specification

 > Status: proposed Gateway integration/consumption contract for WG-011 and WG-012. Details require review before becoming final.
>
> **Ownership boundary (owner-confirmed):** Konam owns the authoritative workflow registry, lifecycle, availability, entitlements, discovery and execution admission. This document specifies the capability metadata and discovery/consumption behaviour required by Workflow Gateway; it does not authorise a duplicate Gateway workflow registry or execution database. Gateway displays and forwards requests using permission-filtered results from Konam.

## Purpose

Konam's authoritative workflow registry stores reusable, versioned automations and governs who may discover and run them. Workflow Gateway consumes the permission-filtered registry API as part of the customer experience. The model registry and plugin/tool registry are also Konam-owned platform capabilities.

**AI-first, workflow-second:** the registry is not intended to contain a predefined workflow for every possible problem. Direct model reasoning and model-plus-tool solutions are first-class alternatives. Register workflows where recurring execution benefits materially from repeatable steps, triggers, integrations, business rules, approvals, monitoring or auditability.

## Workflow suitability and admission

A proposed workflow should have a clear reusable purpose and evidence of value. Assessment should consider:

- Recurrence and expected reuse across users or organisations.
- Stable process steps and well-defined inputs/outputs.
- Business value compared with direct AI or existing tools.
- Required integrations, triggers and external actions.
- Need for deterministic rules, approvals, monitoring or audit.
- Security, data classification, maintenance burden and operating cost.

Do not register one-off prompts, generic reasoning tasks or thin wrappers around model capabilities merely to increase catalogue size. An unmatched request may be handled directly by AI/tools or lead to a recommendation for separate workflow-development assessment. Workflow creation, publication and commercialisation remain subject to normal owner approval and lifecycle governance.

## Lifecycle

Every workflow version is in exactly one state:

1. In development
2. Under testing
3. Available for internal use
4. Available for selected customers
5. Commercially available
6. Suspended or deprecated

Permitted forward path is 1 to 5. Any state from 3 to 5 may move to 6. Moving back from 5 to an earlier state is allowed only by suspension (6) and a new version. Every transition records actor, time and reason and requires an authorised administrator; promotion to state 4 or 5 requires recorded owner approval.

## Availability tiers

The registry distinguishes three availability scopes:

| Lifecycle state | Internal availability | Selected-customer availability | Commercial availability |
|---|---|---|---|
| In development | No | No | No |
| Under testing | Test tenants only | No | No |
| Available for internal use | Yes | No | No |
| Available for selected customers | Yes | Allow-listed customers | No |
| Commercially available | Yes | Yes | Yes, subject to entitlements |
| Suspended or deprecated | No new runs | No new runs | No new runs |

**Unfinished workflows (states 1 and 2) are never exposed as functioning commercial services.** Discovery filters by lifecycle, availability and entitlement before ranking.

## Metadata contract (proposed)

| Field | Description |
|---|---|
| id, name, version | Stable identifier, display name, semantic version |
| description, capabilities, example_intents | Used for discovery and suitability assessment |
| owner | Responsible party |
| lifecycle_state, availability | As above, plus allow-list for selected customers |
| input_schema, output_schema | JSON schemas |
| execution_mode | sync or async; timeout; idempotency support |
| adapter | Execution adapter type and endpoint reference (no secrets) |
| permissions | Required permissions and data classification |
| approval | Whether and when human approval is required |
| metering | Usage and pricing hints (future) |
| dependencies | Other workflows, models or tools required |
| external_visibility | Whether the workflow may be discovered through approved external interfaces; default deny |
| required_scopes | Minimum operation/data scopes needed for discovery, execution and result access |
| api_compatibility | Stable public capability identifier and contract/version compatibility metadata |
| suitability_evidence | Optional documented rationale for recurrence, repeatability and value (proposed; validate in WG-011) |

## External discovery policy (proposed)

External discovery is a filtered view of the existing workflow registry, not a second catalogue. A workflow is returned only after server-side checks of lifecycle state, availability tier, caller identity, workspace access, entitlement, external visibility policy and required scopes. Internal-only, suspended, deprecated, unreleased or otherwise unauthorised workflows must not leak through names, counts, search suggestions or error differences.

The registry metadata describes capability and required permissions; it does not grant permission. The external caller's current grant is checked again at execution and result retrieval. External exposure defaults to deny and requires an explicit publication/allow-list decision.

## Execution contract (proposed)

**Request:** execution_id, workflow_id and version, workspace_id (the tenant context for both personal and organisation workspaces), validated inputs, idempotency key, callback reference for async runs.

**Response (sync):** status, outputs validated against output_schema, usage data.

**Async:** immediate acknowledgement with a tracking reference; status reported by signed callback or polling; states queued, running, awaiting approval, succeeded, failed, cancelled, timed out.

**Errors:** a common error shape (code, message, retryable flag). Timeouts and schema violations are recorded as failed executions.

## Validation rules

- Reject registration without schemas, data classification and approval flags.
- Reject lifecycle transitions that are not permitted.
- Never return a workflow to a tenant that lacks entitlement for it.
- Do not treat a workflow match as proof that a workflow is the best route; the Gateway evaluates suitability against direct AI and tool-based alternatives.
- No automatic registration or publication from an unmatched user request.

## Planned workflow product candidates

RepoGuard, IssuePilot, ReleasePilot, DocuFlow and AgentManager are **planned candidates, not completed integrations**. None is registered and none may be presented as available. Each candidate should be justified by recurring use and value rather than by the assumption that every problem needs its own workflow.

See [Routing and Orchestration Specification](ROUTING_AND_ORCHESTRATION_SPECIFICATION.md), [ADR-002](decisions/ADR-002-AI-FIRST-WORKFLOW-SECOND.md) and [Security and Governance](SECURITY_AND_GOVERNANCE.md).
