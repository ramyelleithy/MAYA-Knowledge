# Output Business Rules Gate

**Specification Template:** `Engine_Contract.md`
**Runtime Position:** Immediately after Reflection Engine, immediately before Response Composer.

---

### 1. Purpose

The final, deterministic check before anything reaches the customer — confirming the turn's composed decision does not violate a Business Rule, leak internal information, or imply an unauthorized commitment. It is the mirror of the Input Business Rules Gate, positioned at the opposite end of the pipeline.

### 2. Responsibilities

Verify, deterministically, that a Reflection-approved turn does not violate an active Business Rule, does not expose internal architecture or instructions, and does not commit to a discount, unit, or term that has not been authorized.

### 3. Non-Responsibilities

Does not evaluate reasoning quality — that is Reflection's job. Does not compose language. Does not decide, plan, or recommend anything.

### 4. Inputs

The Reflection Outcome (must be Proceed) with the Decision Type, Plan, and Recommendation where one exists — **or**, on the escalation path, the Handoff Composer's customer-facing message and context summary, since that path bypasses Reflection entirely.

### 5. Outputs

A **Pass** signal allowing the turn to proceed to the Response Composer, or a **Blocked** signal with a reason code, routing to the Safe Fallback Composer.

### 6. Dependencies

Requires either the Reflection Engine to have already returned Proceed for this turn, or the Handoff Composer to have produced its output on the escalation path.

### 7. Consumes

The Decision Type, Plan, Recommendation, and the Business Context (the applicable rule set).

### 8. Produces

The Pass/Blocked signal that determines whether the Response Composer or the Safe Fallback Composer handles the remainder of this turn.

### 9. Internal Decisions

None beyond deterministic rule matching — this Engine does not exercise judgment.

### 10. Boundaries

Checks compliance only; it never evaluates the quality, fit, or wisdom of what it is checking.

### 11. Failure Modes

The rule-evaluation process itself fails to complete, due to a system fault rather than a genuine rule violation.

### 12. Recovery Strategy

Fail closed — where the check itself cannot be completed, route to the Safe Fallback Composer rather than allowing an unchecked turn through.

### 13. Confidence Behavior

Does not apply — this Engine is fully deterministic and carries no confidence value.

### 14. Knowledge Usage

Yes, directly — the same Business Rules knowledge used by the Input Business Rules Gate.

### 15. Memory Usage

None.

### 16. Business Rules

This Engine is the enforcement point for Business Rules at the point of output; it does not merely reference them, it is the last structural guarantee that they are honored before delivery.

### 17. State Behavior

Stateless.

### 18. Side Effects

None on stored state; its effect is entirely on which downstream path this turn takes.

### 19. Observability

Every Blocked outcome must be logged with its reason code at the highest priority in the system, mirroring the observability standard already set for the Decision Engine's escalation detection.

### 20. Performance Expectations

Must be fast and deterministic — comparable to the Input Business Rules Gate.

### 21. Security Considerations

This is the last line of defense against leaking internal information or implying an unauthorized commitment; it must not be bypassable by anything produced upstream, regardless of how that content was reasoned or planned.

### 22. Extensibility

Its rule set changes only through revisions to `Business_Rules.md`, never through an independent addition made within this Engine.

### 23. Out of Scope

Quality assessment, content generation, and any judgment-based evaluation.

### 24. Invocation Condition

Runs on every turn, unconditionally, once either the Reflection Engine has returned Proceed on the normal path, or the Handoff Composer has produced its output on the escalation path. This Engine is never skipped, regardless of which path a turn takes.

### 25. Determinism Classification

Deterministic.

### 26. Engine Invariants

- Must never pass a turn that violates an active Business Rule.
- Must fail closed on any internal fault, never open.
- Must never be skipped or bypassed regardless of the confidence or urgency attached to the turn.
- Must never be bypassed on the escalation path — a handoff message is checked exactly as rigorously as any other outbound message.
