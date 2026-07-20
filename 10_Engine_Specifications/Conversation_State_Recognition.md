# Conversation State Recognition

**Specification Template:** `Engine_Contract.md`
**Runtime Position:** Immediately after Business Context Engine, immediately before Intent Understanding.

---

### 1. Purpose
Determines the customer's current position within their decision journey, per `Conversation_State_Model.md`, so that everything downstream interprets this turn in light of where the customer actually stands.

### 2. Responsibilities
Recognize progress, regression, or genuine ambiguity in the customer's position within their journey.

### 3. Non-Responsibilities
Does not interpret intent for this specific message — that belongs to Intent Understanding. Does not decide what to do about the recognized position.

### 4. Inputs
The assembled Conversation Context, and indirectly, the Customer Memory.

### 5. Outputs
A Conversation State estimate, with an explicit confidence value.

### 6. Dependencies
Requires the Conversation Context Engine and Memory Engine to have completed.

### 7. Consumes
The Conversation Context and Customer Memory.

### 8. Produces
The Conversation State, consumed by Intent Understanding, the Planning Engine, and the Context Builder.

### 9. Internal Decisions
Which position best fits the available signals; how much ambiguity to carry forward rather than resolve prematurely.

### 10. Boundaries
Recognition only — never prescribes next steps.

### 11. Failure Modes
Insufficient signal to determine position with adequate confidence.

### 12. Recovery Strategy
Default to the least presumptive state — earlier in the journey — rather than assuming an advanced position that might not exist.

### 13. Confidence Behavior
Bound by the Confidence Contract; must attach an explicit confidence value to its estimate.

### 14. Knowledge Usage
None.

### 15. Memory Usage
Yes, indirectly, through the Conversation Context.

### 16. Business Rules
None apply directly.

### 17. State Behavior
Stateful — depends on and updates an ongoing estimate held across the conversation.

### 18. Side Effects
None beyond its own output carrying forward.

### 19. Observability
Log the estimated state and its confidence for every turn.

### 20. Performance Expectations
Moderate — an interpretive judgment, but a bounded one.

### 21. Security Considerations
None distinct.

### 22. Extensibility
The set of recognized positions is defined in `Conversation_State_Model.md`; changes there should propagate here.

### 23. Out of Scope
Interpreting specific message intent; deciding next steps.

### 24. Invocation Condition
Every turn, unconditionally.

### 25. Determinism Classification
Interpretive.

### 26. Engine Invariants
- Must never assume an advanced position without genuine evidence.
- Must always attach a confidence value to its estimate.
- Must never resolve genuine ambiguity by silently picking one position.
