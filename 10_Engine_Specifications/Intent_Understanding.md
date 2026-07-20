# Intent Understanding

**Specification Template:** `Engine_Contract.md`
**Runtime Position:** Immediately after Conversation State Recognition, immediately before Project Engine.

---

### 1. Purpose
Interprets what the customer is actually seeking with this specific message, distinguishing Question, Interest, Intent, Decision, or Commitment per `Psychology.md`.

### 2. Responsibilities
Classify this message's intent into one of the five states `Psychology.md` defines, with an attached confidence value.

### 3. Non-Responsibilities
Does not update the Mental Model itself — that belongs to the Customer Engine. Does not decide the Decision Type — that belongs to the Decision Engine.

### 4. Inputs
The canonical Message object and the current Conversation State.

### 5. Outputs
An Intent classification with confidence.

### 6. Dependencies
Requires Normalization and Conversation State Recognition to have completed.

### 7. Consumes
The Message content and the Conversation State.

### 8. Produces
The Intent classification, consumed by the Customer Engine and the Reasoning Engine.

### 9. Internal Decisions
Which of the five intent states best fits this specific message.

### 10. Boundaries
Classification only — never decides what to do about it.

### 11. Failure Modes
The message is too ambiguous to classify with adequate confidence.

### 12. Recovery Strategy
Fall back to the most literal, surface-level reading of the message rather than guessing at a deeper intent that cannot be supported.

### 13. Confidence Behavior
Bound by the Confidence Contract.

### 14. Knowledge Usage
None.

### 15. Memory Usage
Indirectly, through the Conversation State.

### 16. Business Rules
None apply directly.

### 17. State Behavior
Stateless per turn, though it consumes stateful context.

### 18. Side Effects
None.

### 19. Observability
Log the classification and its confidence for every turn.

### 20. Performance Expectations
Fast relative to the Reasoning Engine.

### 21. Security Considerations
None distinct.

### 22. Extensibility
The five-state taxonomy is defined in `Psychology.md`; changes there should propagate here.

### 23. Out of Scope
Updating the Mental Model; deciding the Decision Type.

### 24. Invocation Condition
Every turn, unconditionally.

### 25. Determinism Classification
Interpretive.

### 26. Engine Invariants
- Must never classify a surface-level question as commitment, or vice versa, without genuine supporting evidence.
- Must always attach a confidence value to its classification.
