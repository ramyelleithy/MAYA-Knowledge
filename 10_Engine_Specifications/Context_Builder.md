# Context Builder

**Specification Template:** `Engine_Contract.md`
**Runtime Position:** Immediately after Customer Engine, immediately before Reasoning Engine.

---

### 1. Purpose
Performs pure assembly of the six context objects — Message, Conversation, Memory, Business, Project, and Customer — into a single Unified Runtime Context, with no extraction or interpretation of its own.

### 2. Responsibilities
Combine the six already-finished context objects into one coherent structure.

### 3. Non-Responsibilities
Does not extract, interpret, or analyze anything itself — every input it receives is already finished work from another Engine.

### 4. Inputs
The Message, Conversation, Memory, Business, Project, and Customer context objects.

### 5. Outputs
The Unified Runtime Context.

### 6. Dependencies
Requires all six context-producing Engines to have completed for this turn.

### 7. Consumes
The six context objects.

### 8. Produces
The Unified Runtime Context, consumed by the Reasoning Engine and, indirectly, every Engine after it.

### 9. Internal Decisions
None — pure combination.

### 10. Boundaries
Assembly only.

### 11. Failure Modes
One or more constituent context objects is missing.

### 12. Recovery Strategy
Proceed with whichever context objects are available; a missing slice degrades the Unified Context rather than blocking it — except Business Context, whose absence must block, per that Engine's own failure policy.

### 13. Confidence Behavior
Does not apply directly — it carries forward the confidence values already attached to its constituent parts without adding any of its own.

### 14. Knowledge Usage
None directly.

### 15. Memory Usage
None directly.

### 16. Business Rules
None apply directly, beyond requiring Business Context's presence before proceeding.

### 17. State Behavior
Stateless.

### 18. Side Effects
None.

### 19. Observability
Log which context slices were present or missing for this turn.

### 20. Performance Expectations
Fast — mechanical combination, not judgment.

### 21. Security Considerations
None distinct.

### 22. Extensibility
Adding a new context type requires updating this Engine's assembly logic, not the individual context-producing Engines.

### 23. Out of Scope
Interpretation of any kind.

### 24. Invocation Condition
Every turn, unconditionally.

### 25. Determinism Classification
Deterministic.

### 26. Engine Invariants
- Must never interpret or alter any constituent context object.
- Must never proceed without Business Context present.
- Must never silently substitute a missing context slice with fabricated content.
