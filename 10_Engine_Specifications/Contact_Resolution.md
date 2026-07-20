# Contact Resolution

**Specification Template:** `Engine_Contract.md`
**Runtime Position:** Immediately after Input Business Rules Gate, immediately before Memory Engine.

---

### 1. Purpose
Determines whether the incoming contact identifier matches an existing customer record, or represents a new contact — a narrow, factual question that everything touching memory depends on.

### 2. Responsibilities
Perform a lookup by contact identifier; return a Contact Status (Existing or New) and, where matched, the record reference.

### 3. Non-Responsibilities
Does not classify role — whether this contact is likely a customer, broker, or competitor remains an evolving hypothesis inside the Customer Engine's Mental Model, never decided here. Does not load memory content itself.

### 4. Inputs
The canonical Message object's sender identifier.

### 5. Outputs
A Contact Status (Existing or New) and, where matched, a record reference.

### 6. Dependencies
Requires Normalization to have produced the canonical Message.

### 7. Consumes
The sender identifier.

### 8. Produces
The Contact Status, consumed by the Memory Engine to decide whether retrieval is attempted at all.

### 9. Internal Decisions
What counts as a match, left open at the architecture level and decided at implementation.

### 10. Boundaries
Factual identity matching only — never role inference.

### 11. Failure Modes
Lookup service unavailable; an ambiguous match against multiple candidates.

### 12. Recovery Strategy
Default to "New Contact" on any failure or ambiguity — proceeding with a blanker understanding is safer than assuming an incorrect match.

### 13. Confidence Behavior
Does not apply — this is a deterministic match, not an interpretive judgment.

### 14. Knowledge Usage
None.

### 15. Memory Usage
Yes — this lookup is what determines whether the Memory Engine has anything to retrieve at all.

### 16. Business Rules
None apply directly.

### 17. State Behavior
Stateful — performs a lookup against persisted records.

### 18. Side Effects
None beyond the lookup itself.

### 19. Observability
Log every resolution outcome, especially ambiguous matches that fell back to the safe default.

### 20. Performance Expectations
Fast — a simple lookup.

### 21. Security Considerations
Must never leak one customer's record to another through a mismatched lookup.

### 22. Extensibility
Matching logic can evolve from exact to more sophisticated matching without changing this Engine's contract.

### 23. Out of Scope
Role classification; memory content retrieval.

### 24. Invocation Condition
Every turn, unconditionally, early in the pipeline.

### 25. Determinism Classification
Deterministic.

### 26. Engine Invariants
- Must never assume a match without genuine confirmation.
- Must default to New Contact on any doubt.
- Must never classify role — only contact identity.
