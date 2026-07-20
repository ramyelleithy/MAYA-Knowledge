# Handoff Composer

**Specification Template:** `Engine_Contract.md`
**Runtime Position:** Conditional branch, triggered directly from the Decision Engine's escalation detection.

---

### 1. Purpose
Prepares the customer for a clean transfer to a human consultant and assembles the context that consultant needs, making the mandatory escalation guarantee in `Business_Rules.md` structural rather than incidental.

### 2. Responsibilities
Compose a customer-facing handoff message that feels like a natural continuation, per `Conversation.md`; assemble a context summary for the human consultant so nothing the customer has already shared needs repeating.

### 3. Non-Responsibilities
Does not make the escalation decision itself — that belongs entirely to the Decision Engine. Does not continue consultative reasoning once triggered.

### 4. Inputs
The Reasoning Conclusion, the Mental Model, the Conversation State, and the Escalation Trigger identified by the Decision Engine.

### 5. Outputs
A customer-facing handoff message and a consultant-facing context summary.

### 6. Dependencies
Requires the Decision Engine to have already triggered Escalation.

### 7. Consumes
The Reasoning Conclusion, Mental Model, Conversation State, and Escalation Trigger.

### 8. Produces
The handoff message and the context summary — the handoff message is routed through the Output Business Rules Gate and the Memory & State Writer before delivery via WhatsApp Reply, exactly like the normal path's tail, rather than bypassing them; the context summary is delivered to the human consultant through whatever channel that requires.

### 9. Internal Decisions
How to phrase the handoff naturally, per `Conversation.md`'s Human Handoff principles.

### 10. Boundaries
Composition and context assembly only — never re-evaluates the escalation decision itself.

### 11. Failure Modes
Cannot assemble a complete context summary for the human consultant.

### 12. Recovery Strategy
Fall back to a generic, honest message informing the customer a human will follow up, rather than blocking the handoff on a failed summary.

### 13. Confidence Behavior
Does not apply.

### 14. Knowledge Usage
Minimal — which specific human consultant or channel a given escalation type routes to.

### 15. Memory Usage
Yes, directly — reads the Mental Model and Conversation State to build the consultant-facing summary.

### 16. Business Rules
Bound entirely by the mandatory escalation rule that triggered it.

### 17. State Behavior
Stateful.

### 18. Side Effects
Yes — initiates the actual handoff to a human consultant.

### 19. Observability
Log every handoff, its trigger, and the context summary provided.

### 20. Performance Expectations
Moderate.

### 21. Security Considerations
The context summary must respect the privacy principles in `Customer_Knowledge.md` even when shared internally with a human consultant.

### 22. Extensibility
Routing logic (which consultant or channel handles a given escalation type) can evolve independently of this Engine's contract.

### 23. Out of Scope
Making the escalation decision; continuing consultative reasoning.

### 24. Invocation Condition
Conditional — only when the Decision Engine triggers Escalation.

### 25. Determinism Classification
Interpretive for the composition itself; the routing logic may be largely deterministic.

### 26. Engine Invariants
- Must never re-evaluate or reverse the escalation decision.
- Must never fail to notify the customer that a human will follow up.
- Must never omit context the human consultant genuinely needs, forcing the customer to repeat themselves.
