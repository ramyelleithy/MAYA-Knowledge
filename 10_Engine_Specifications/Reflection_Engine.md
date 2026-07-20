# Reflection Engine

**Specification Template:** `Engine_Contract.md`
**Runtime Position:** Immediately after Planning Engine and the conditional Recommendation Engine, immediately before Output Business Rules Gate.

---

### 1. Purpose

Reviews the quality of this turn's Reasoning Conclusion, Decision, Plan, and Recommendation (where produced) before anything reaches the customer, per `Reflection_Model.md` — a check on the turn's own thinking, not a re-performance of it.

### 2. Responsibilities

- Assess whether the Reasoning Conclusion, Decision, Plan, and Recommendation (if present) are internally consistent and adequate given their own stated confidence.
- Detect cases where confidence appears overstated, a Conflict or Unknown appears understated, or the Decision Type no longer fits given what downstream Engines actually produced.
- Either confirm the turn is ready to proceed, or send it back to one specific earlier Engine with a stated concern.
- Never let a genuinely identified concern pass through silently.

### 3. Non-Responsibilities

- Does not reason about the customer's situation itself.
- Does not re-classify the Decision Type — it may flag a concern about it, but only the Decision Engine may re-classify.
- Does not alter a Recommendation's content — it may flag a concern, but only the Recommendation Engine may revise it.
- Does not compose customer-facing language.
- Does not enforce Business Rules directly — that remains the Output Business Rules Gate's job.

### 4. Inputs

The Reasoning Conclusion and its Reasoning Trace; the Decision Type, Decision Rationale, and Execution Constraints; the Plan; the Recommendation, where one was produced.

### 5. Outputs

A Reflection Outcome: either **Proceed**, or **Revise**, naming the single earlier Engine the turn should return to and the specific concern that triggered it.

### 6. Dependencies

Requires the Reasoning Conclusion, Decision, Plan, and (where applicable) Recommendation to already exist for this turn.

### 7. Consumes

The same objects listed under Inputs.

### 8. Produces

The Reflection Outcome — the last checkpoint before the Output Business Rules Gate and the Response Composer.

### 9. Internal Decisions

It alone decides whether this turn's chain of conclusions holds together as a whole; it alone decides which single earlier Engine a "Revise" outcome should be directed to.

### 10. Boundaries

Reviews; never re-performs reasoning, deciding, planning, or recommending itself. Scoped to the current turn only.

### 11. Failure Modes

- Cannot complete its review with the time or information available.
- Identifies a genuine concern but cannot determine which Engine should address it.
- Risk of repeated "Revise" cycles without convergence.

### 12. Recovery Strategy

- If review cannot complete, proceed with the pre-reflection conclusions rather than stalling the turn indefinitely.
- Cap revision cycles at one per turn. If a concern remains unresolved after a single revision attempt, escalate to the Handoff Composer as a safe default rather than looping.

### 13. Confidence Behavior

Does not assert a new first-order confidence about the world. It performs a meta-check: whether the confidence values already attached elsewhere are actually justified by what was gathered, and may flag them as unjustified where they are not.

### 14. Knowledge Usage

None directly — it reviews conclusions that have already incorporated knowledge, rather than consulting knowledge itself.

### 15. Memory Usage

None directly — it reviews the Mental Model only as already reflected within the Reasoning Conclusion.

### 16. Business Rules

Checks that nothing in the turn's conclusions contradicts the Rule Hierarchy already established; does not itself enforce rules — that remains the deterministic responsibility of the Output Business Rules Gate.

### 17. State Behavior

Stateless with respect to persistence.

### 18. Side Effects

None on stored state. Its effect is entirely on control flow — proceed, or return to a specific earlier Engine.

### 19. Observability

Every Reflection Outcome, especially every "Revise," should be logged along with the specific concern raised, building a record of where the pipeline most often needs self-correction.

### 20. Performance Expectations

Expected to be fast relative to the Reasoning Engine, since it reviews rather than re-derives; it should not become a bottleneck given that it runs on every turn.

### 21. Security Considerations

No distinct exposure beyond the objects it reviews; any logged concern must not itself leak confidential detail outside its intended audience.

### 22. Extensibility

The Reflection Outcome format (Proceed / Revise plus target Engine and concern) may gain new outcome types over time, but such additions belong here, not improvised ad hoc elsewhere.

### 23. Out of Scope

Reasoning, deciding, planning, or recommending on its own; enforcing Business Rules directly; composing any customer-facing message.

### 24. Invocation Condition

Runs on every turn, unconditionally, once Planning (and Recommendation, where applicable) have completed.

### 25. Determinism Classification

Interpretive — assessing whether a chain of conclusions holds together is a judgment call, not a fixed rule check.

### 26. Engine Invariants

- Must never let a turn proceed past a genuinely identified concern without at least one revision attempt.
- Must never revise a Reasoning Conclusion, Decision, Plan, or Recommendation directly itself — it may only flag and redirect to the Engine responsible.
- Must never allow more than one revision cycle per turn without escalating rather than looping indefinitely.
- Must never approve a turn whose expressed confidence is not actually supported by its own Grounding or Evidence.
