# MAYA Runtime Architecture — Level 1 (v3, Final)
## The Main Consultation Pipeline

**Status:** Final — consolidates v1 and v2, resolves the two remaining review points, and is the version intended for formal approval. No implementation (n8n, nodes, JSON, expressions, or code) should begin until this document is explicitly signed off.

---

## 1. Resolution: Recommendation Engine Remains Independent

A late review raised a real question: should Recommendation be removed as its own Engine and folded into Planning, since it is "just one Decision Type among several"?

The premise is correct — Recommendation is one of six possible outcomes of the Decision Engine's classification (Answer, Question, Recommendation, Clarification, Escalation, Refusal), and `Recommendation_Model.md` itself defines a recommendation as a decision, not a capability. But the conclusion does not follow. Two distinct responsibilities were being conflated:

- **Classifying** that Recommendation is the appropriate action for this turn — a cheap, categorical judgment. This belongs entirely to the **Decision Engine**, and only there.
- **Producing** the actual recommendation once that classification is made — a potentially complex process involving eligibility screening, suitability scoring, comparison across alternatives, business priority weighting, and confidence-weighted ranking, per the future direction already anticipated for MAYA. This deserves its own isolated Engine for the same reason Reasoning, Project, and Customer each have their own — a distinct, non-trivial responsibility should not be absorbed into a stage whose job is something else.

Folding recommendation production into the Planning Engine would give Planning two unrelated jobs at once — sequencing steps, and running a scoring process — which violates the single-responsibility principle this entire architecture has been built around. The Recommendation Engine remains independent, and remains strictly conditional: it is invoked only after the Decision Engine has already, separately, classified this turn's action as Recommendation. Its existence does not imply recommendation is the default or expected endpoint of a consultation — the majority of turns will never reach it, exactly as `Consultation_Methodology.md` requires.

---

## 2. The Complete Pipeline, End to End

```
Customer Message
      │
      ▼
Communication Gateway
      │
      ▼
Normalization
      │
      ▼
Input Business Rules Gate ──── (blocked) ──► Safe Fallback Composer ──► Memory & State Writer ──► Reply
      │ (passed)
      ▼
Contact Resolution
      │
      ▼
Memory Engine
      │
      ▼
Conversation Context Engine
      │
      ▼
Business Context Engine
      │
      ▼
Conversation State Recognition
      │
      ▼
Intent Understanding
      │
      ▼
Project Engine (Detection → Brain Loader, conditional)
      │
      ▼
Customer Engine (Mental Model Update)
      │
      ▼
Context Builder (Unified Runtime Context Assembly)
      │
      ▼
Reasoning Engine
      │
      ▼
Decision Engine ──── (escalation) ──► Handoff Composer ──► Output Business Rules Gate ──► Memory & State Writer ──► Reply
      │ (Answer / Question / Clarification / Refusal / Recommendation)
      ▼
Planning Engine
      │
      ▼
Recommendation Engine (conditional — only if Decision = Recommendation)
      │
      ▼
Reflection Engine
      │
      ▼
Output Business Rules Gate ──── (blocked) ──► Safe Fallback Composer ──► Memory & State Writer ──► Reply
      │ (passed)
      ▼
Response Composer
      │
      ▼
Memory & State Writer
      │
      ▼
WhatsApp Reply
```

---

## 3. Full Stage Roster (Locked)

This is the complete, final set of runtime responsibilities for the Main Consultation Pipeline. Each carries forward its full specification from v1/v2 (name, responsibility, inputs, outputs, reason for existing, failure behavior, statefulness, and its dependency on Knowledge, Memory, and the LLM), unchanged except where noted:

1. **Communication Gateway** — receives the raw inbound event and establishes the Turn ID for everything produced in this turn. Stateless. No Knowledge/Memory/LLM.
2. **Normalization** — converts to a canonical Message object. Stateless. Partial LLM use (voice transcription only).
3. **Input Business Rules Gate** — deterministic pre-filter. Stateless. Depends on Knowledge (rules). No LLM.
4. **Contact Resolution** — determines Existing vs. New contact. Stateful. Depends on Memory (lookup). No LLM.
5. **Memory Engine** — loads customer memory, conditioned on Contact Resolution. Stateful. Depends on Memory. No LLM.
6. **Conversation Context Engine** — assembles conversation history. Stateful. No Knowledge/Memory dependency of its own. No LLM.
7. **Business Context Engine** — loads applicable business rules. Stateless in effect. Depends on Knowledge. No LLM. Must fail loudly, never silently.
8. **Conversation State Recognition** — locates the customer's position in their journey. Stateful. Depends on Memory indirectly. Uses LLM. Bound by the Confidence Contract.
9. **Intent Understanding** — classifies the customer's intent for this message. Stateless per turn. Uses LLM. Bound by the Confidence Contract.
10. **Project Engine** (Detection → Brain Loader) — identifies the project and loads its full knowledge; runs before the Customer Engine. Stateful (Detection); stateless per-turn (Loader). Depends on Knowledge directly. Detection's shift-judgment uses LLM and is bound by the Confidence Contract; first-contact mapping is deterministic.
11. **Customer Engine** — updates the Mental Model, including the role hypothesis, now informed by Project Context. Stateful. Depends on Memory directly. Uses LLM. Bound by the Confidence Contract.
12. **Context Builder** — pure assembly of the six context objects into one Unified Runtime Context. Stateless. No Knowledge/Memory/LLM of its own.
13. **Reasoning Engine** — connects context, understanding, and knowledge into a conclusion. Stateless with respect to persistence. Depends on Knowledge and Memory indirectly. Uses LLM centrally. Bound by the Confidence Contract.
14. **Decision Engine** — classifies the appropriate action type and detects mandatory escalation. Stateless with respect to persistence. Depends on Knowledge directly (escalation rules, deterministic). Uses LLM for the overall judgment. Consumes the Confidence Contract.
15. **Planning Engine** — plans the specific steps for the committed action, including the scope of a recommendation where applicable. Stateless with respect to persistence. Depends on Knowledge/Memory indirectly. Uses LLM.
16. **Recommendation Engine (conditional)** — produces the actual recommendation when the Decision Engine has classified this turn as Recommendation; independent per Section 1 above. Stateless with respect to persistence. Depends on Knowledge directly (Project Brain, and alternatives where relevant). Depends on Memory indirectly. Uses LLM. Bound by the Confidence Contract.
17. **Reflection Engine** — reviews the quality of reasoning, the decision, and any recommendation before finalizing. Stateless. No direct Knowledge/Memory dependency. Uses LLM.
18. **Output Business Rules Gate** — final deterministic safety check before composition. Stateless. Depends on Knowledge. No LLM. Fails closed.
19. **Response Composer** — turns the approved decision into the actual customer-facing message. Stateless. Uses LLM.
20. **Memory & State Writer** — persists the updated Mental Model, Customer Knowledge, and Conversation State. Stateful — this is where state is written. No Knowledge/LLM dependency.
21. **WhatsApp Reply** — delivers the message. Stateless. No Knowledge/Memory/LLM.

**Conditional branches:**
- **Handoff Composer** — triggered from the Decision Engine's escalation detection; prepares the customer and the human consultant's context. Its output is routed through the Output Business Rules Gate and the Memory & State Writer before delivery, exactly like the normal path's tail, per the resolution in `Validation_Escalation.md`. Stateful. Depends on Memory directly. Uses LLM.
- **Safe Fallback Composer** — triggered from either Business Rules Gate; produces a pre-approved safe response, deterministic by construction and therefore bypassing the Output Business Rules Gate by design, per the Scope principle above; its output still routes through the Memory & State Writer before delivery, per the Persistence Tail Principle. Stateless. Depends on Knowledge minimally. No LLM, by design.

**The Persistence Tail Principle** (cross-cutting, not a stage): Every completed customer-facing turn must pass through the Memory & State Writer before delivery, regardless of which path produced the response — the normal consultation path, the escalation path, or either Business Rules Gate's blocked path. A response the customer actually receives is a completed turn, and the fact that it occurred, and whatever it revealed, must survive into the next turn's context exactly as any other turn's would.

**Scope of the Output Business Rules Gate:** The Output Business Rules Gate is mandatory only for responses whose content is dynamically produced or composed during runtime — the normal path's Response Composer output and the escalation path's Handoff Composer output. Deterministic, pre-approved templates, such as those the Safe Fallback Composer selects from, are considered intrinsically compliant by construction and bypass the gate by design: checking a template that cannot violate a Business Rule against the very check built to catch such violations adds no safety and only unnecessary complexity.

**The Confidence Contract** (cross-cutting, not a stage): Conversation State Recognition, Intent Understanding, Project Engine's shift-detection, Customer Engine, Reasoning Engine, and Recommendation Engine must each attach an explicit confidence level to their output. The Decision Engine is the primary consumer of these values, per `Decision_Framework.md`'s requirement that expressed certainty never exceed the evidence behind it. Where an Engine synthesizes more than one upstream confidence value into a single new assessment — as the Reasoning Engine does — that assessment's confidence must never exceed the lowest confidence among the upstream inputs that are genuinely load-bearing for the specific conclusion being reached. An upstream input that the conclusion does not actually depend on does not lower it merely by being uncertain itself; the rule is dependency-aware, not a blanket minimum across every available signal.

**The Turn Identity Contract** (cross-cutting, not a stage): Every artifact produced within a single turn — every context object, the Reasoning Conclusion and its Reasoning Trace, the Decision, the Plan, the Recommendation, and the composed reply — must carry a **Turn ID**, the **Engine Name** that produced it, a **Timestamp or Sequence** marker, and its **Artifact Type**. The Turn ID is established once, at the Communication Gateway, and is carried forward by every Engine from that point on — it is the umbrella identifier for the entire lifecycle of a turn, independent of whether or when a decision within that turn is ever reached. The Decision Trace ID remains exactly what it already is: a reference to the specific decision made within a turn, nested inside the broader scope the Turn ID provides. This distinction matters because meaningful observability requires tracing a turn's artifacts from the moment they are created, not only from the moment a decision exists to retroactively link them.

---

## 4. Runtime Entry Events

The Main Consultation Pipeline is **one runtime entry point, not the entire runtime.** It handles the case where a customer message arrives and a reply is composed. But MAYA's runtime, over its full lifetime, will need to respond to more than that single kind of event.

This section documents that the future architecture is **event-driven**, not a single linear pipeline — without designing any of these other entries now. None of the following are being built at Level 1; they are named here so that Level 2 design decisions do not accidentally assume the Consultation Pipeline is the only way the system is ever entered.

Anticipated Runtime Entry Events include:

- **Customer Message** — the entry this document fully specifies.
- **Human Handoff Result** — a human consultant's outcome flowing back into the system after a handoff (for example, closing the loop on a conversation MAYA previously escalated).
- **Project Update** — a change in a project's knowledge (price, availability, offer, stage) that may need to propagate into ongoing conversations already referencing that project.
- **Customer File Uploaded** — a customer sending a document or image outside the flow of a text message.
- **Internal CRM or Business Event** — an internal system state change relevant to an ongoing customer relationship.
- **Scheduled Follow-up** — a deliberate, planned re-engagement with a customer, distinct from a customer-initiated message.
- **Timeout Trigger** — the passage of time itself becoming the reason a runtime action is needed, rather than any external message.
- **Conversation Reopening** — a customer returning after an extended gap, where continuity (per `Conversation_State_Model.md` and `Memory_Model.md`) must be re-established rather than assuming a fresh start.

Each of these is a distinct runtime entry, and each will eventually need its own specification at the same level of rigor as the Consultation Pipeline. None of that design work is in scope for Level 1. What Level 1 commits to is only this: the Consultation Pipeline must be designed so that nothing within it assumes it is the sole way state is ever touched, since it demonstrably will not be.

---

## 5. What Remains Open for Level 2

- The exact contents and internal shape of each Engine's inputs, outputs, and contracts.
- How the Confidence Contract is technically represented and thresholded.
- How Contact Resolution technically identifies a match.
- How the File Request capability attaches to this pipeline.
- The full specification of each Runtime Entry Event named in Section 4, each in its own turn.
- Retry, timeout, and idempotency policy for each stage.

---

**This is the version submitted for final approval of Level 1.**
