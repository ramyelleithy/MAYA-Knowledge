# Input Business Rules Gate

**Specification Template:** `Engine_Contract.md`
**Runtime Position:** Immediately after Normalization, immediately before Contact Resolution.

---

### 1. Purpose

The first, deterministic checkpoint a message passes through — filtering spam, abuse, clearly out-of-scope requests, and messages that map directly to a known, mandatory response pattern, before any reasoning is attempted. It is the mirror of the Output Business Rules Gate, positioned at the opposite end of the pipeline.

### 2. Responsibilities

Evaluate the canonical Message object against deterministic, pre-authorized rules; return a Pass signal for anything not matched, or a Blocked signal with a specific reason code for anything that is.

### 3. Non-Responsibilities

Does not interpret the customer's broader situation, intent, or history — matching is purely pattern-based. Does not compose the fallback response itself — that belongs to the Safe Fallback Composer.

### 4. Inputs

The canonical Message object from Normalization.

### 5. Outputs

A **Pass** signal allowing the turn to proceed to Contact Resolution, or a **Blocked** signal with a reason code, routing to the Safe Fallback Composer.

### 6. Dependencies

Requires Normalization to have already produced the canonical Message object.

### 7. Consumes

The canonical Message object and the Business Rules knowledge defining what patterns must be intercepted.

### 8. Produces

The Pass/Blocked signal that determines whether Contact Resolution or the Safe Fallback Composer handles the remainder of this turn.

### 9. Internal Decisions

None beyond deterministic rule matching — this Engine does not exercise judgment.

### 10. Boundaries

Checks pattern matches only; it never evaluates the quality, fit, or wisdom of anything, and never interprets the customer's situation.

### 11. Failure Modes

The rule-evaluation process itself fails to complete, due to a system fault rather than a genuine rule match.

### 12. Recovery Strategy

Where the failure mode is known to be low-risk, treat as Pass rather than block a legitimate message; otherwise route to the Safe Fallback Composer rather than silently proceeding on an unverified basis.

### 13. Confidence Behavior

Does not apply — this Engine is fully deterministic and carries no confidence value.

### 14. Knowledge Usage

Yes, directly — the rule definitions, a stable form of knowledge per `Knowledge_Quality.md`'s treatment of stable versus temporary knowledge.

### 15. Memory Usage

None — this runs before Contact Resolution and the Memory Engine, deliberately, so it never depends on customer history.

### 16. Business Rules

This Engine is the earliest enforcement point for Business Rules; it does not merely reference them, it is the first structural guarantee that a message the business has already decided how to handle never reaches costlier reasoning stages.

### 17. State Behavior

Stateless — it evaluates the message against fixed, pre-authorized rules without needing conversation history to do so.

### 18. Side Effects

None on stored state; its effect is entirely on which downstream path this turn takes.

### 19. Observability

Every Blocked outcome must be logged with its reason code, at the same high priority as the Output Business Rules Gate's blocked outcomes and the Decision Engine's escalation detections.

### 20. Performance Expectations

Must be fast and cheap — this is the gate that protects the rest of the pipeline's cost and latency from being spent on messages that will never reach a full consultative response.

### 21. Security Considerations

Must not be bypassable by any upstream manipulation of the message content, and must correctly catch known abuse patterns before they reach any stage capable of being manipulated by them.

### 22. Extensibility

Its rule set changes only through revisions to `Business_Rules.md`, never through an independent addition made within this Engine.

### 23. Out of Scope

Quality assessment, content generation, customer history, or any interpretive judgment.

### 24. Invocation Condition

Runs on every turn, unconditionally, immediately after Normalization.

### 25. Determinism Classification

Deterministic.

### 26. Engine Invariants

- Must never allow a message matching a known mandatory-intercept pattern to reach Contact Resolution or any reasoning stage.
- Must never depend on customer memory or conversation history to reach its decision.
- Must fail toward the safer outcome when its own evaluation cannot complete, per Field 12.
