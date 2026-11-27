# Architecture Decision Records

This directory contains the platform's decision history. **Proposed** ADRs are not owner-approved decisions and must not be treated as implementation instructions.

| ADR | Subject | Status | Related issue |
|---|---|---|---|
| [ADR-001](ADR-001-OVERALL-PLATFORM-DIRECTION.md) | Overall platform direction | Accepted — four owner-approved product directions | WG-001 |
| [ADR-002](ADR-002-AI-FIRST-WORKFLOW-SECOND.md) | AI-first, workflow-second architecture | Accepted — owner-approved strategic principle; supersedes ADR-001's workflow-first routing default | WG-001, WG-014 |
| [ADR-003](ADR-003-SUPABASE-DATABASE-AUTH.md) | Supabase database and authentication candidate | Accepted — owner-approved 27 November 2026 | WG-002 |
| [ADR-004](ADR-004-RENDER-HOSTING.md) | Render hosting candidate | Proposed | WG-002 |
| [ADR-005](ADR-005-FASTAPI-BACKEND.md) | FastAPI backend candidate | Proposed | WG-002 |
| [ADR-006](ADR-006-NEXTJS-FRONTEND.md) | Next.js frontend candidate | Accepted — owner-approved 27 November 2026 | WG-002 |
| [ADR-007](ADR-007-LITELLM-MODEL-GATEWAY.md) | LiteLLM model gateway candidate | Proposed; optional, not mandatory for v1 | WG-002 |
| [ADR-008](ADR-008-LANGGRAPH-ORCHESTRATION.md) | Custom orchestration first; LangGraph optional | Proposed | WG-002 |
| [ADR-009](ADR-009-MCP-TOOL-INTEGRATION.md) | MCP as one supported integration protocol | Proposed | WG-002 |
| [ADR-010](ADR-010-PERSONAL-AND-ORGANISATION-TENANCY.md) | Personal and organisation workspace tenancy | Accepted — owner-approved 27 November 2026 | WG-002, WG-004, WG-023, WG-027 |
| [ADR-011](ADR-011-DURABLE-JOB-QUEUE.md) | Durable job admission and queue architecture | Proposed | WG-002, WG-016 |
| [ADR-012](ADR-012-BYOK-SECRET-ENCRYPTION.md) | BYOK credential encryption and key management | Proposed | WG-002, WG-003, WG-020 |
| [ADR template](ADR-TEMPLATE.md) | Standard format for future decisions | Template | WG-002 |

Validation evidence for the proposed technology ADRs: [Technology Validation Report](../TECHNOLOGY_VALIDATION_REPORT.md).

## Decision rules

- An ADR is **Accepted** only after explicit owner approval is recorded.
- ChatGPT provides independent architecture review and recommendations.
- Technology candidates remain replaceable until evidence and owner approval support acceptance.
- Rejected or superseded decisions remain in the history with their rationale.
- Every material change to an accepted decision requires a new or revised ADR and owner approval.
