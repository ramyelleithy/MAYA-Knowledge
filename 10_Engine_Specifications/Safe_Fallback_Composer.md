# Safe Fallback Composer

**Specification Template:** `Engine_Contract.md`
**Runtime Position:** Conditional branch, triggered from either the Input or Output Business Rules Gate.

---

### 1. Purpose
Produces a pre-approved, safe response whenever a Business Rules Gate blocks a turn from proceeding normally, so a blocked message never simply vanishes.

### 2. Responsibilities
Map a gate's reason code to an appropriate, safe, honest response.

### 3. Non-Responsibilities
Does not reason, decide, plan, or recommend.

### 4. Inputs
The blocking gate's reason code.

### 5. Outputs
A safe, generic message appropriate to that reason code.

### 6. Dependencies
Requires either Business Rules Gate to have returned Blocked.

### 7. Consumes
The reason code.

### 8. Produces
The fallback message, routed through the Memory & State Writer before delivery via WhatsApp Reply, per the Persistence Tail Principle. It does not route through the Output Business Rules Gate, per that gate's documented scope: mandatory only for dynamically produced or composed content, which this Engine's pre-approved templates are not.

### 9. Internal Decisions
Which pre-approved response template fits the reason code.

### 10. Boundaries
Fallback composition only.

### 11. Failure Modes
No matching template exists for a given reason code.

### 12. Recovery Strategy
A hard-coded, minimal acknowledgment as the absolute last resort.

### 13. Confidence Behavior
Does not apply.

### 14. Knowledge Usage
Minimal — mapping reason codes to pre-approved responses.

### 15. Memory Usage
None.

### 16. Business Rules
Exists specifically to honor a rule that has already blocked the turn.

### 17. State Behavior
Stateless.

### 18. Side Effects
None beyond producing the fallback message.

### 19. Observability
Log every fallback invocation and its reason code.

### 20. Performance Expectations
Must be fast and maximally reliable — this is the last resort in the pipeline.

### 21. Security Considerations
Must never leak the specific internal rule or reasoning that caused the block to the customer in a way that exposes architecture.

### 22. Extensibility
New reason codes and templates can be added without changing this Engine's contract.

### 23. Out of Scope
Any judgment-based response.

### 24. Invocation Condition
Conditional — only when a Business Rules Gate returns Blocked.

### 25. Determinism Classification
Deterministic, by design, so this path can never fail for the same reason the gate itself blocked the turn.

### 26. Engine Invariants
- Must never fail to produce some response, even in the worst case.
- Must never expose the specific internal rule or architecture that caused the block.
- Must never attempt interpretive reasoning of its own.
