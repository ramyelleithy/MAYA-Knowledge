# Recommendation Engine

**Specification Template:** `Engine_Contract.md` (final, 26 fields)
**Output Object:** `Recommendation.md` (final, 9 components, bound by its Semantic Contract)
**Runtime Position:** Level 1 Main Consultation Pipeline — conditional, invoked after Planning Engine, only when Decision Type = Recommendation.

---

### 1. Purpose

The Recommendation Engine exists to produce the actual Recommendation once the Decision Engine has classified this turn's action as Recommendation — performing the specific, often genuinely complex work of determining what should be recommended and why, so that Planning has a concrete, trustworthy artifact to build the rest of the turn's execution around. It exists as its own Engine, distinct from Planning, because deciding what fits a customer and deciding how to communicate it are different kinds of work, per the reasoning already settled in the Level 1 Runtime Architecture.

### 2. Responsibilities

- Determine the Primary Option — and, where genuinely warranted, a small number of Alternatives — that best fits the customer's understood situation, grounded in the Reasoning Conclusion and the relevant Project Brain(s).
- Populate every component of the Recommendation object defined in `Recommendation.md`: Recommendation Type, Primary Option, Grounding, Supporting Rationale, Trade-offs, Preconditions, Alternatives, Confidence, and Decision Trace Reference.
- Weigh the multiple factors `Recommendation_Model.md` identifies — need, financial capacity, goal, risk tolerance, priorities, preferences, market state, and business objectives — without letting any single factor decide alone.
- State honest Trade-offs for every option included, never omitted to make an option appear more ideal than it is.
- State explicit Preconditions — the facts that must remain true for this recommendation to still hold.
- Assign Confidence proportional to the strength of the understanding and evidence actually available, never to how well the recommendation happens to read.
- Where understanding is insufficient to responsibly recommend anything, decline to produce a Recommendation rather than forcing one.

### 3. Non-Responsibilities

- Does not decide that Recommendation is the appropriate Decision Type for this turn — that belongs to the Decision Engine; this Engine only runs once that classification has already been made.
- Does not decide how, when, or in what sequence the recommendation is delivered — that belongs to the Planning Engine.
- Does not compose any customer-facing language — that belongs to the Response Composer.
- Does not authorize discounts, commit inventory, or grant any permission beyond what the Project Brain and Business Context already establish as true.
- Does not perform reasoning about the customer's situation itself — it consumes the Reasoning Conclusion already produced, it does not reinterpret the situation independently.

### 4. Inputs

- The Reasoning Conclusion, specifically its Situation Assessment, Active Hypotheses, Decision-Relevant Signals, and Confidence.
- The Decision Engine's classification (Decision Type = Recommendation) and its Decision Trace ID.
- The Planning Engine's scope for this turn — which project(s) and options are within bounds to consider, per the Execution Constraints the Decision Engine and Planning Engine have already established.
- The Customer Context (Mental Model).
- The Project Brain(s) for the project(s) under consideration.

### 5. Outputs

A single Recommendation object, containing exactly the nine components defined in `Recommendation.md`: Recommendation Type, Primary Option, Grounding, Supporting Rationale, Trade-offs, Preconditions, Alternatives, Confidence, and Decision Trace Reference. Where a responsible Recommendation cannot be produced, an explicit decline signal is emitted instead, per Field 12 below.

### 6. Dependencies

Requires the Decision Engine to have already classified this turn as Recommendation; requires the Reasoning Engine's Reasoning Conclusion to exist for this turn; requires the Planning Engine to have already established the scope this Recommendation may draw on; requires the Project Engine to have already loaded the Project Brain(s) relevant to that scope.

### 7. Consumes

The Reasoning Conclusion, the Decision Engine's classification and Decision Trace ID, the Planning Engine's scope for this turn, the Customer Context, and the relevant Project Brain(s).

### 8. Produces

The Recommendation object: the sole basis the Response Composer works from, via the plan the Planning Engine builds around it, and the artifact the Reflection Engine reviews before anything reaches the customer.

### 9. Internal Decisions

- It alone decides which specific option, among what the relevant Project Brain(s) actually offer, constitutes the best fit for this customer.
- It alone decides how many Alternatives, if any, are genuinely warranted — bounded, per `Recommendation_Model.md`, to a small number rather than an exhaustive comparison.
- It alone decides the Confidence level attached to the Recommendation, within the ceiling set by the confidence it inherited from upstream.

### 10. Boundaries

Its judgment concerns what to recommend and why, never how or when to communicate it — the Semantic Contract in `Recommendation.md` is the absolute boundary here. It produces a Recommendation for this turn's classification only; it does not retroactively revise a Recommendation already delivered in an earlier turn, though a later turn's Recommendation may supersede it as circumstances change, per `Recommendation_Model.md`'s principle that a recommendation is never final.

### 11. Failure Modes

- Understanding is insufficient to responsibly ground any specific Primary Option.
- The relevant Project Brain is missing or materially incomplete for the option(s) under consideration.
- A genuine, unresolved tie exists between two or more options that the available understanding cannot responsibly break.
- Honest Preconditions or Trade-offs cannot actually be stated — often a sign the option is not yet understood well enough to recommend at all.
- Producing a structurally incomplete Recommendation object, with one or more required components missing.

### 12. Recovery Strategy

- Where understanding is insufficient, decline to produce a Recommendation entirely, signaling this back so the turn can fall back to a more conservative form of response rather than receiving a forced, low-quality recommendation.
- A genuine tie between options is not itself a failure — present the strongest option as the Primary Option and the other as an Alternative, with honest Trade-offs on each, and let the customer's own priorities decide, per `Consultation_Methodology.md`'s approach to handling alternatives.
- Where any required component — especially Preconditions or Trade-offs — cannot be honestly populated, the Engine must fail the turn rather than emit an incomplete object, mirroring the completeness invariant already established for the Reasoning Engine.

### 13. Confidence Behavior

Confidence must be proportional to the strength of the Grounding available and the confidence already carried by the Reasoning Conclusion and the Mental Model — this Engine cannot manufacture a higher confidence than what it inherited from upstream. Where upstream confidence is low, the Recommendation's own Confidence must reflect that ceiling rather than exceed it.

### 14. Knowledge Usage

Depends directly and centrally on Project Knowledge, via the Project Brain(s) in scope. Recommendation quality is more sensitive to the completeness of `Project_Knowledge.md`'s categories — commercial, operational, comparative, and consultative — than any other Engine in the pipeline, since a recommendation drawing on incomplete project knowledge is exactly the situation Field 11's failure modes describe.

### 15. Memory Usage

Consumes the Mental Model to weigh the customer's preferences and priorities. Does not write to Memory directly — any implication this Recommendation should have for future understanding is captured within its own Grounding and Supporting Rationale, to be picked up by the ordinary Customer Engine and Memory & State Writer path afterward.

### 16. Business Rules

Bound directly by the Recommendation-specific rules in `Business_Rules.md`: never recommend without understanding, never recommend outside confirmed knowledge, never recommend something known to conflict with the customer's needs, and never recommend something because it is easier to sell. Also bound by Company Protection — no promised discount or committed inventory may appear as though authorized unless the Project Brain actually confirms it, and any such limit must surface honestly as a Precondition or Trade-off rather than being smoothed over.

### 17. State Behavior

Stateless with respect to persistence.

### 18. Side Effects

None on stored state. Its effect is producing the Recommendation object consumed later in the same turn.

### 19. Observability

Every Recommendation object — all nine components — should be logged, with particular attention to Confidence, Preconditions, and any decline-to-recommend outcome. Preconditions specifically deserve ongoing visibility: a Precondition that no longer holds by the time a customer acts on a recommendation, or by the time a Project Update event arrives per the Runtime Entry Events in the Level 1 Architecture, should be capable of being caught rather than silently ignored.

### 20. Performance Expectations

Invoked conditionally, so its overall cost across the runtime is lower than an Engine that runs on every turn. When it does run, it may be nearly as demanding as the Reasoning Engine, since it performs its own weighing of multiple factors across potentially more than one Project Brain — a cost accepted given that it is only incurred when genuinely needed.

### 21. Security Considerations

Handles project commercial and comparative knowledge, which may be sensitive per `Knowledge_Governance.md`, alongside customer financial and preference information, which is bound by the privacy principles in `Customer_Knowledge.md` — both at once. It must never let one project's confidential comparative standing leak into how an Alternative is described in a way that violates the Competitive Neutrality principle in `Business_Rules.md`, including never fabricating or implying criticism of a competing project or developer.

### 22. Extensibility

The nine-component structure of the Recommendation is fixed by `Recommendation.md`. This Engine may refine how it weighs factors or evaluates fit over time, but any change to the object's structure requires revising `Recommendation.md` itself first, never a unilateral extension made within this Engine.

### 23. Out of Scope

Classifying the Decision Type; detecting mandatory escalation; planning delivery sequencing or pacing; composing customer-facing language; authorizing discounts or inventory commitments; writing to memory.

### 24. Invocation Condition

Strictly conditional. Invoked only when the Decision Engine has classified this turn's Decision Type as Recommendation. It is never invoked for any other Decision Type.

### 25. Determinism Classification

Interpretive. Weighing multiple factors and evaluating fit across options is model-based judgment throughout; no part of this Engine's core function is deterministic.

### 26. Engine Invariants

- Must never produce a Recommendation without a completed Reasoning Conclusion to ground it in.
- Must never recommend an option known to conflict with the customer's confirmed needs or constraints.
- Must never omit Trade-offs or Preconditions in order to make an option appear more ideal than it is.
- Must never present more than a small, bounded number of Alternatives — ordinarily no more than one, and rarely a second as a fallback.
- Must never assign a Confidence value higher than what the upstream Reasoning Conclusion and Mental Model confidence actually support.
- Must never emit a structurally incomplete Recommendation object — all nine components must be present, or the Engine must fail the turn rather than hand the Planning Engine an incomplete artifact.
- Must never fabricate or imply criticism of a competing project or developer when populating Trade-offs or Alternatives.

---

**This specification is offered for review under `Engine_Contract.md` and `Recommendation.md`. Awaiting approval before proceeding to the next Engine.**
