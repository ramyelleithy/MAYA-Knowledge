# Business Context Engine

**Specification Template:** `Engine_Contract.md`
**Runtime Position:** Immediately after Conversation Context Engine, immediately before Conversation State Recognition.

---

### 1. Purpose
Loads the Business Rules and policies applicable to this turn, making them an explicit, isolated part of context rather than something quietly assumed elsewhere.

### 2. Responsibilities
Retrieve the current, applicable set of Business Rules for this turn.

### 3. Non-Responsibilities
Does not enforce or interpret rules itself — enforcement belongs to the Input and Output Business Rules Gates and the Decision Engine.

### 4. Inputs
None beyond the fact that a turn is in progress — this is largely a static load.

### 5. Outputs
A Business Context object.

### 6. Dependencies
None beyond the pipeline having started.

### 7. Consumes
The stored Business Rules configuration.

### 8. Produces
The Business Context, consumed by nearly every downstream Engine.

### 9. Internal Decisions
None — a static load.

### 10. Boundaries
Retrieval only.

### 11. Failure Modes
Rules configuration unavailable or stale.

### 12. Recovery Strategy
Must fail loudly and block progress rather than proceeding without Business Rules in view, given their governing authority over everything downstream.

### 13. Confidence Behavior
Does not apply.

### 14. Knowledge Usage
Yes — this is its entire purpose.

### 15. Memory Usage
None.

### 16. Business Rules
This Engine is how Business Rules become available to the rest of the runtime; it does not itself enforce them.

### 17. State Behavior
Stateless in effect, since rules change rarely, though technically sourced from persisted configuration.

### 18. Side Effects
None.

### 19. Observability
Log which rule-set version was loaded for this turn.

### 20. Performance Expectations
Should be fast, ideally cached given how rarely rules change.

### 21. Security Considerations
Must ensure the loaded rules are the currently authorized version, never stale or tampered with.

### 22. Extensibility
The rule set changes only through revisions to `Business_Rules.md`.

### 23. Out of Scope
Enforcement, interpretation.

### 24. Invocation Condition
Every turn, unconditionally.

### 25. Determinism Classification
Deterministic.

### 26. Engine Invariants
- Must never allow a turn to proceed silently without Business Rules loaded.
- Must never load an outdated or unauthorized rule set.
