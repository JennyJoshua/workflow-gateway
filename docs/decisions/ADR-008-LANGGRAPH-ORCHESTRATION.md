# ADR-008: Custom Orchestration First; LangGraph Optional

- **Status:** Proposed — not owner-approved
- **Date:** 2026-11-26
- **Deciders:** Owner (final approval); ChatGPT (architecture review)
- **Related issues:** WG-002, WG-010, WG-014, WG-015, WG-016, WG-022

## Context

Workflow Gateway needs deterministic routing, approval pauses, retries, asynchronous execution and quality checks. The architecture identifies LangGraph as a possible stateful orchestration framework, but the initial product should avoid unnecessary agent complexity.

## Options considered

| Option | Advantages | Disadvantages | Cost/free-tier implications | Lock-in and exit considerations |
|---|---|---|---|---|
| Custom Python state machine first | Simple, explicit policy enforcement, minimal dependencies and full control | Project must implement persistence, retries, resumability and observability carefully | No orchestration framework licence cost; engineering cost remains | Low framework lock-in if state and contracts are application-owned |
| LangGraph OSS library | Stateful graph execution, streaming, persistence/checkpointing and human-in-the-loop primitives | Learning curve and graph-specific state model; durable execution still needs correct persistence configuration and admission records | OSS library available; hosted deployment/observability services are separate product choices | Moderate library coupling; keep domain state and workflow contracts independent |
| Hosted agent/orchestration platform | Faster managed operations and observability | Provider dependence, data handling and recurring costs | Likely paid at production scale; verify current terms | Higher platform lock-in |

## Proposed decision

Start with a **small explicit Python orchestration/state-machine module**. Keep the workflow state model, execution records and approval rules in platform-owned contracts. Reassess LangGraph only when the system requires complex branching, resumable multi-step graphs, or human-in-the-loop state transitions that justify the dependency.

Do not equate an orchestration framework's checkpointing with the platform's durable job admission, queueing, idempotency or audit requirements. Those remain platform responsibilities.

## Security, privacy and data residency

- Graph state may contain prompts, user data, tool outputs and credentials; exclude secrets and apply retention controls.
- Persist only the minimum required state.
- Ensure approval interrupts cannot be bypassed by retries or graph resumption.
- Validate all tool outputs and never trust graph/model state as authorisation.
- If a checkpointer is used, review serializer security and encryption at rest.

## Consequences

### Positive
- Avoids adopting a framework before complexity justifies it.
- Keeps policy and execution state under project control.
- Allows a later LangGraph adapter if required.

### Negative / trade-offs
- Custom orchestration must be carefully tested for recovery, retries and state transitions.
- Some advanced graph features may take longer to implement manually.

### Operational implications
- Define state transition invariants and idempotency in WG-016.
- Use durable external storage; in-memory checkpoints are for tests only.
- Add evaluation tests for branching, retries, interruptions and resumption.

## Validation plan

1. Model the initial routing and approval flow as a small deterministic state machine.
2. Compare it with a LangGraph proof of concept using synthetic inputs.
3. Test crash/restart recovery and approval interruption with a persistent checkpointer.
4. Measure operational complexity before deciding whether to adopt LangGraph.

## References

- https://docs.langchain.com/oss/python/langgraph/overview
- https://docs.langchain.com/oss/python/langgraph/persistence
- https://docs.langchain.com/oss/python/langgraph/checkpointing

## Approval record

- **ChatGPT review:** Proposed — initial review 2026-11-26
- **Owner decision:** Pending
