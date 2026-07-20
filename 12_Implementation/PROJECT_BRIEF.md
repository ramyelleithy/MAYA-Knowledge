# MAYA / Phoenix Runtime — Project Brief for Claude Code

## What This Project Is

MAYA is an AI Senior Property Consultant built for **Propify Brokerage Agency**, an Egyptian real estate brokerage. She operates on WhatsApp, covering the full Egyptian real estate market across many developers — not scoped to any single developer's inventory. She is explicitly **not** a chatbot or customer-support bot: she is designed as a consultative, honest, memory-carrying property consultant that cannot be misled into over-promising and cannot quit like a human salesperson can.

The project has two completed phases and is now entering a third:

1. **Philosophy / Knowledge Base** — complete, frozen. 32 documents defining MAYA's identity, principles, psychology model, business rules, and cognitive philosophy.
2. **Runtime Architecture (Level 1)** — complete, frozen at **v1.0**. A full specification of 23 "Engines" (discrete runtime responsibilities) forming the Main Consultation Pipeline, validated against five end-to-end customer journeys.
3. **Implementation (Level 2)** — just started. An Implementation Audit has mapped the existing production n8n workflow against the frozen architecture. This is the phase you are being asked to help with.

## Where the Files Are

- **Knowledge Base repository:** `MAYA-Knowledge` on GitHub (`ramyelleithy/MAYA-Knowledge`) — 32 files organized into 7 folders (Foundation, Behavior & Values, Cognitive Architecture, Mind Models, Knowledge System, Project System, Customer & Recommendation).
- **Runtime Architecture release package:** `Phoenix_Runtime_Level1_Architecture_v1.0_Frozen` — 68 files: the full Engine Contract template, 23 Engine specifications, 3 runtime artifact definitions (Reasoning Conclusion, Recommendation, Plan), and 5 validation journey reports.
- **Implementation Audit:** `Implementation_Audit.md` — a full Engine Mapping Matrix comparing the production n8n workflow (`MAYA - WhatsApp Sales Agent`, workflow ID `X1QVNNYUFbJftuc2`) against the frozen architecture.

All three should be placed together in this project folder before work begins.

## Core Architectural Concept

The Runtime is organized as a pipeline of **Engines**, each documented against a fixed 26-field contract (`Engine_Contract.md`): Purpose, Responsibilities, Non-Responsibilities, Inputs, Outputs, Dependencies, Consumes, Produces, Internal Decisions, Boundaries, Failure Modes, Recovery Strategy, Confidence Behavior, Knowledge Usage, Memory Usage, Business Rules, State Behavior, Side Effects, Observability, Performance Expectations, Security Considerations, Extensibility, Out of Scope, Invocation Condition, Determinism Classification, and Engine Invariants.

The Main Consultation Pipeline, in order:

```
Communication Gateway -> Normalization -> Input Business Rules Gate -> Contact Resolution ->
Memory Engine -> Conversation Context Engine -> Business Context Engine ->
Conversation State Recognition -> Intent Understanding -> Project Engine -> Customer Engine ->
Context Builder -> Reasoning Engine -> Decision Engine -> Planning Engine ->
Recommendation Engine (conditional) -> Reflection Engine -> Output Business Rules Gate ->
Response Composer -> Memory & State Writer -> WhatsApp Reply
```

Two conditional branches exist: **Handoff Composer** (triggered by mandatory escalation) and **Safe Fallback Composer** (triggered by either Business Rules Gate blocking a turn). Both branches route through the Output Business Rules Gate and Memory & State Writer before delivery, exactly like the normal path — this was a specific correction made during validation.

Three cross-cutting contracts apply to every Engine: the **Turn Identity Contract** (every artifact carries a Turn ID, producing Engine name, timestamp, and artifact type), the **Confidence Contract** (dependency-aware — a synthesized confidence never exceeds the lowest confidence among the inputs it materially depends on), and the **Persistence Tail Principle** (every completed turn, on any path, passes through the Memory & State Writer).

## Current State of Production

A working n8n workflow already exists and is live (`MAYA - WhatsApp Sales Agent`). The Implementation Audit found:

- **Strong, reusable as-is:** Communication Gateway, Normalization (including Arabic voice transcription), Project Engine (both detection and Project Brain loading, including a working cache-based re-detection avoidance pattern), Conversation Context Engine, WhatsApp Reply, and the file-delivery branch.
- **Two structural problems:**
  1. The Business Rules Check (financing/spam detection) runs too late in the pipeline — after Contact Resolution, Memory retrieval, and full Project Brain loading, instead of immediately after Normalization. The detection logic itself is fine, but a later dependency trace found the relocation is not a pure position fix: two of its supporting nodes had false dependencies on data that doesn't exist that early (see "Analysis Update — Finding #8" in `Implementation_Audit.md` for the full breakdown and PR-001/PR-002 status).
  2. The entire cognitive core — Reasoning, Decision, Planning, and Recommendation, plus part of Response Composer — is currently one single OpenAI GPT-4o call with one combined structured-output schema. This is the central piece of Level 2 work: decomposing this into properly separated Engine calls.
- **Missing entirely:** the Output Business Rules Gate, the Reflection Engine, the Turn Identity Contract, and Recommendation History (needed to re-verify a prior recommendation's Preconditions when a customer returns).

## Agreed Implementation Strategy

Both a full Implementation Audit and the following principle were explicitly agreed before any code should be touched:

> Never rewrite working production behavior without architectural justification. Reuse what already satisfies its Engine contract. Refactor only where architectural boundaries require it. Build only what genuinely does not exist.

The agreed implementation order (a vertical-slice / walking-skeleton strategy, not full-depth-first):

1. Relocate the Business Rules Check to run immediately after Normalization. (In progress, split into sub-PRs after dependency analysis: PR-001 — remove the `customer_message` false dependency, done; PR-002 — remove the `project_code` false dependency, pending; then the relocation itself. See `Implementation_Audit.md`, Finding #8.)
2. Introduce the Turn Identity Contract at the entry point.
3. Split the current combined "Chatwoot Sync - Incoming" node into a genuine Contact Resolution step and a genuine Memory Engine retrieval step.
4. Decompose the single "Ask MAYA" LLM call into its constituent Engines, incrementally: Decision Engine's classification first, then Reasoning, then Planning and Recommendation.
5. Add the Output Business Rules Gate and Reflection Engine as new steps before delivery.
6. Extend persistence to cover the Mental Model and Recommendation History, not just the current conversation_summary and sales_state fields.

## What to Do With This Brief

Read `Implementation_Audit.md` first — it has the full node-by-node mapping. Then read the specific Engine specifications relevant to whichever step above is being worked on. Do not propose new architectural concepts without flagging them explicitly — the architecture is frozen at v1.0, and any genuinely new architectural need should be raised as a question, not implemented silently, since changes now require evidence from implementation, not speculative improvement.
