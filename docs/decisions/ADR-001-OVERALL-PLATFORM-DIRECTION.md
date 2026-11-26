# ADR-001: Overall Platform Direction

- **Status:** Accepted (four product directions approved by the owner; recorded from the ChatGPT handover dated 26 November 2026)
- **Date:** 26 November 2026
- **Deciders:** Owner, with ChatGPT as Chief AI Architect and Independent Reviewer
- **Specification baseline:** Version 1.0

## Context

Workflow Gateway must give customers one conversational interface to proprietary workflows, multiple AI models, plugins and tools. Several strategic choices had to be made before architecture work could proceed.

## Decision

| Decision | Approved direction |
|---|---|
| Platform relationship | Both: standalone application and primary interface to the larger commercial workflow platform |
| Model supply | Both: platform-managed models and customer-provided API keys (BYOK) |
| Workflow routing | Configurable, with proprietary workflow-first default |
| Development approach | Build the chat foundation now and connect workflows progressively |

## Consequences

- The platform must be independently useful (chat and models) before workflows are connected.
- Tenancy, BYOK security and metering are required early.
- Routing needs a policy engine rather than hard-coded behaviour (WG-014, WG-025).
- Unfinished workflows must never appear as commercial services; the registry enforces lifecycle and availability.
- Workflow integrations arrive progressively and each needs owner approval before starting.

## Alternatives considered

Standalone only; interface-only to the commercial platform; platform-managed models only; BYOK only; workflow-only routing; building all workflows before the chat. These were not chosen because the approved combination balances independence, commercial differentiation and delivery risk.

## Not decided by this ADR

Technology choices (Supabase, Render, FastAPI, Next.js, LiteLLM, LangGraph, MCP) are **proposed and require validation** through further ADRs (WG-002). Licensing, pricing, billing provider and the first workflow to connect are open.

## Review

Changes to this ADR require a new ADR, ChatGPT review and owner approval. See [Contribution and Change Control](../../project-management/CONTRIBUTION_AND_CHANGE_CONTROL.md) and [Technical Architecture](../TECHNICAL_ARCHITECTURE.md).
