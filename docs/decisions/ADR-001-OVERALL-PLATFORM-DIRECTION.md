# ADR-001: Overall Platform Direction

- **Status:** Accepted; workflow-first routing decision superseded by ADR-002
- **Date:** 26 November 2026
- **Deciders:** Owner, with ChatGPT as Chief AI Architect and Independent Reviewer
- **Specification baseline:** Version 1.0 (historical decision record)

## Context

Workflow Gateway must give customers one conversational interface to proprietary workflows, multiple AI models, plugins and tools. Several strategic choices had to be made before architecture work could proceed.

## Original decision (historical)

| Decision | Approved direction at the time |
|---|---|
| Platform relationship | Both: standalone application and primary interface to the larger commercial workflow platform |
| Model supply | Both: platform-managed models and customer-provided API keys (BYOK) |
| Workflow routing | Configurable, with proprietary workflow-first default |
| Development approach | Build the chat foundation now and connect workflows progressively |

## Current interpretation

ADR-002 supersedes only the earlier **proprietary workflow-first routing default**. The other decisions in this ADR remain in force unless changed by a later ADR. Current routing is AI-first, workflow-second: select the simplest safe and suitable capability, including direct AI, external tools, existing workflows or combinations.

## Consequences of the original decision

The historical decision established the need for a standalone useful chat experience, model choice, policy-based routing, workflow lifecycle governance and progressive integrations. These remain relevant, but workflows are no longer the default preferred route merely because a matching workflow exists.

## Alternatives considered at the time

Standalone only; interface-only to the commercial platform; platform-managed models only; BYOK only; workflow-only routing; building all workflows before the chat.

## Not decided by this ADR

Technology choices (Supabase, Render, FastAPI, Next.js, LiteLLM, LangGraph, MCP) are **proposed and require validation** through further ADRs (WG-002). Licensing, pricing, billing provider and the first workflow to connect are open.

## Review

This is a historical decision record. Current strategic direction is defined by [ADR-002](ADR-002-AI-FIRST-WORKFLOW-SECOND.md). See [Contribution and Change Control](../../project-management/CONTRIBUTION_AND_CHANGE_CONTROL.md).
