# Workflow Registry Specification

> Status: proposed baseline for WG-011 and WG-012. Details require review before becoming final.

## Purpose

The workflow registry stores proprietary workflows and governs who may see and run them. It is one of three separate registries (workflows, models, plugins and tools).

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
| description, capabilities, example_intents | Used for discovery |
| owner | Responsible party |
| lifecycle_state, availability | As above, plus allow-list for selected customers |
| input_schema, output_schema | JSON schemas |
| execution_mode | sync or async; timeout; idempotency support |
| adapter | Execution adapter type and endpoint reference (no secrets) |
| permissions | Required permissions and data classification |
| approval | Whether and when human approval is required |
| metering | Usage and pricing hints (future) |
| dependencies | Other workflows, models or tools required |

## Execution contract (proposed)

**Request:** execution_id, workflow_id and version, workspace_id (the tenant context for both personal and organisation workspaces), validated inputs, idempotency key, callback reference for async runs.

**Response (sync):** status, outputs validated against output_schema, usage data.

**Async:** immediate acknowledgement with a tracking reference; status reported by signed callback or polling; states queued, running, awaiting approval, succeeded, failed, cancelled, timed out.

**Errors:** a common error shape (code, message, retryable flag). Timeouts and schema violations are recorded as failed executions.

## Validation rules

- Reject registration without schemas, data classification and approval flags.
- Reject lifecycle transitions that are not permitted.
- Never return a workflow to a tenant that lacks entitlement for it.

## Planned workflow product candidates

RepoGuard, IssuePilot, ReleasePilot, DocuFlow and AgentManager are **planned candidates, not completed integrations**. None is registered and none may be presented as available.

See [Routing and Orchestration Specification](ROUTING_AND_ORCHESTRATION_SPECIFICATION.md) and [Security and Governance](SECURITY_AND_GOVERNANCE.md).
