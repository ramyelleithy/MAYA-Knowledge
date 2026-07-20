# Planning Engine

**Specification Template:** `Engine_Contract.md` (final, 26 fields)
**Output Object:** `Plan.md` (final, 5 components, bound by its Semantic Contract)
**Runtime Position:** Level 1 Main Consultation Pipeline — immediately after Decision Engine, immediately before the conditional Recommendation Engine.

---

### 1. Purpose

The Planning Engine exists to translate the Decision Engine's committed action type into a concrete Plan — the scope and delivery shape needed to carry that action out — so that the conditional Recommendation Engine and the Response Composer each have a structured strategy to work from, rather than having to independently invent one at the moment they act.

### 2. Responsibilities

- Establish the Goal for this turn, given the committed Decision Type and its Execution Constraints.
- Where the Decision Type is Recommendation, establish the Scope handed to the Recommendation Engine — which project(s) and option types are eligible for consideration — consistent with the Execution Constraints already set by the Decision Engine and the Recommendation Rule in `Business_Rules.md`.
- Produce a content-independent Communication Plan: what should be led with, what should be held back, and how delivery should be paced across the conversation, consistent with `Conversation.md` and the currently active `Consultation_Strategy.md` approach.
- Define Fallback Behavior for what this turn should do if the planned path cannot be completed as intended.
- Attach the Decision Trace Reference linking this Plan back to the classification it was built to carry out.

### 3. Non-Responsibilities

- Does not classify the Decision Type — that belongs to the Decision Engine; this Engine only runs once that classification already exists.
- Does not determine the actual content of a recommendation — that belongs to the Recommendation Engine.
- Does not compose any customer-facing language — that belongs to the Response Composer.
- Does not detect escalation triggers — Escalation is short-circuited directly from the Decision Engine to the Handoff Composer, bypassing this Engine entirely.
- Does not re-evaluate business-rule permissibility from scratch — it operates within the Execution Constraints the Decision Engine has already established.

### 4. Inputs

- The Decision Type, Decision Rationale, Execution Constraints, and Decision Trace ID from the Decision Engine.
- The Conversation State.
- The Consultation Strategy currently in effect.
- The Customer Context (Mental Model), for fitting pacing and tone to this customer.
- The Project Context, where the Decision Type is Recommendation.

### 5. Outputs

A single Plan object, containing exactly the five components defined in `Plan.md`: Goal, Scope, Communication Plan, Fallback Behavior, and Decision Trace Reference.

### 6. Dependencies

Requires the Decision Engine to have already committed a non-Escalation Decision Type and produced its Execution Constraints; requires the Conversation State and the current Consultation Strategy to already be available within the Unified Runtime Context.

### 7. Consumes

The Decision Type, Decision Rationale, Execution Constraints, and Decision Trace ID; the Conversation State; the Consultation Strategy; the Customer Context; and, where relevant, the Project Context.

### 8. Produces

The Plan: consumed by the conditional Recommendation Engine for its Scope, and by the Response Composer for its Goal, Communication Plan, and Fallback Behavior.

### 9. Internal Decisions

- It alone decides the specific Scope boundaries for a Recommendation, within the limits the Decision Engine's Execution Constraints already impose.
- It alone decides the Communication Plan's pacing and sequencing strategy for this turn.
- It alone decides the Fallback Behavior for this turn's planned path.

### 10. Boundaries

Plans how this turn's committed action will be carried out, never what that action's substantive content is, and never the literal words used to express it — the Semantic Contract in `Plan.md` is the absolute boundary here. Its authority is scoped to the current turn only.

### 11. Failure Modes

- The Decision Type or Execution Constraints arrive missing or ambiguous.
- The Conversation State or Consultation Strategy is unavailable.
- A genuine conflict exists between the Execution Constraints and what the Conversation State or Business Rules would otherwise suggest is appropriate.
- No coherent Scope can be determined for a Recommendation decision — for instance, if Project Context was never established, though this should ordinarily have been caught upstream.

### 12. Recovery Strategy

- If the Decision Type or Execution Constraints are missing, this Engine cannot responsibly proceed — it signals failure back rather than guessing at a Plan.
- If the Conversation State or Consultation Strategy is unavailable, default to the most conservative Communication Plan available: short, single-topic, and making no assumption of advanced customer readiness.
- If a coherent Scope cannot be determined for a Recommendation, narrow it to the Entry Project alone rather than leaving Scope undefined, mirroring `Business_Rules.md`'s default of remaining with the Entry Project unless it has been shown unsuitable.

### 13. Confidence Behavior

Does not apply in the way it does for an interpretive judgment about the world — a Plan is a structural artifact, not an assertion about reality, and therefore does not itself carry a Confidence value under the Confidence Contract. However, this Engine consumes the confidence levels attached to its upstream inputs, and lower upstream confidence should lead it toward a more conservative Communication Plan and a narrower Scope, rather than being disregarded.

### 14. Knowledge Usage

Indirect and limited. It draws on Business Rules only as already resolved into the Execution Constraints it receives, and on Project Context only enough to establish Scope boundaries for a Recommendation — it does not consult Project Knowledge in depth itself; that belongs to the Recommendation Engine.

### 15. Memory Usage

Consumes the Customer Context (Mental Model) to fit its Communication Plan to this customer's situation and pace. Does not write to Memory.

### 16. Business Rules

Bound by the Execution Constraints already established by the Decision Engine, and directly applies the Recommendation Rule from `Business_Rules.md` when setting Scope — defaulting to the Entry Project and only widening Scope where a shift is genuinely warranted.

### 17. State Behavior

Stateless with respect to persistence.

### 18. Side Effects

None on stored state. Its effect is entirely on shaping how the remainder of this turn proceeds.

### 19. Observability

The full Plan — all five components — should be logged for every turn it runs on, with particular attention to Scope decisions (which projects or options were made eligible, and why) and any Fallback Behavior that was actually triggered, so that recommendation-scope choices remain auditable over time.

### 20. Performance Expectations

Expected to be lightweight and fast relative to the Reasoning Engine or the Recommendation Engine — it performs structural decisions over already-resolved inputs, not open-ended synthesis or evaluation of options.

### 21. Security Considerations

Scope decisions must respect the Competitive Neutrality principle in `Business_Rules.md` — this Engine must never make a competitor's project eligible for consideration in a way that principle would not otherwise permit, and must never widen Scope beyond what the Execution Constraints authorize.

### 22. Extensibility

The five-component structure of the Plan is fixed by `Plan.md`. This Engine may refine how it sets Scope or shapes a Communication Plan over time, but any change to the object's structure requires revising `Plan.md` itself first.

### 23. Out of Scope

Classifying the Decision Type; detecting escalation triggers; determining the actual content of a recommendation; composing any customer-facing language.

### 24. Invocation Condition

Runs on every turn except where the Decision Engine has classified Escalation, which bypasses this Engine entirely via the direct short-circuit to the Handoff Composer.

### 25. Determinism Classification

Hybrid. Scope-setting for a Recommendation leans heavily on deterministic rules (defaulting to the Entry Project, per `Business_Rules.md`), while shaping the Communication Plan to this customer's conversation involves a thinner layer of interpretive judgment about pacing and fit.

### 26. Engine Invariants

- Must never determine the actual content of a recommendation, answer, or response.
- Must never compose customer-facing language.
- Must never override or contradict the Decision Engine's classification or Execution Constraints.
- Must never expand Scope beyond what the Execution Constraints and Business Rules permit.
- Must never produce a Plan without a valid Decision Trace Reference linking it to the Decision Engine's classification.
- Must never leave Fallback Behavior undefined for a turn whose planned path is not guaranteed to succeed.

---

**This specification is offered for review under `Engine_Contract.md` and `Plan.md`. Awaiting approval before proceeding to the next Engine.**
