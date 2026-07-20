# Conversation Context Engine

**Specification Template:** `Engine_Contract.md`
**Runtime Position:** Immediately after Memory Engine, immediately before Business Context Engine.

---

### 1. Purpose
Assembles the relevant recent conversation history for this thread into a single, structured Conversation Context.

### 2. Responsibilities
Retrieve and structure the recent messages in this thread relevant to the current turn.

### 3. Non-Responsibilities
Does not interpret conversation state or intent — those belong to Conversation State Recognition and Intent Understanding.

### 4. Inputs
The canonical Message object and the thread's stored conversation history.

### 5. Outputs
A Conversation Context object.

### 6. Dependencies
Requires Normalization to have completed.

### 7. Consumes
The stored conversation history for this thread.

### 8. Produces
The Conversation Context, consumed by Conversation State Recognition and later by the Context Builder.

### 9. Internal Decisions
How much history is genuinely relevant to include.

### 10. Boundaries
Assembly only — no interpretation.

### 11. Failure Modes
History storage unavailable or incomplete.

### 12. Recovery Strategy
Proceed with whatever history is available; missing history degrades quality without halting the turn.

### 13. Confidence Behavior
Does not apply.

### 14. Knowledge Usage
None.

### 15. Memory Usage
None directly — reads conversation history, not customer memory.

### 16. Business Rules
None apply directly.

### 17. State Behavior
Stateful.

### 18. Side Effects
None — read-only.

### 19. Observability
Log whether full or partial history was retrieved for this turn.

### 20. Performance Expectations
Should be fast.

### 21. Security Considerations
Must retrieve only this thread's history, never another customer's.

### 22. Extensibility
The underlying storage mechanism can change without affecting downstream consumers.

### 23. Out of Scope
Interpretation of conversation state or intent.

### 24. Invocation Condition
Every turn, unconditionally.

### 25. Determinism Classification
Deterministic.

### 26. Engine Invariants
- Must never retrieve another customer's conversation history.
- Must never fabricate history that does not exist.
