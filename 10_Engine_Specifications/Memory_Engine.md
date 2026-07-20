# Memory Engine

**Specification Template:** `Engine_Contract.md`
**Runtime Position:** Immediately after Contact Resolution, immediately before Conversation Context Engine.

---

### 1. Purpose
Retrieves everything already known about this specific customer from prior interactions, conditioned on Contact Resolution, so the conversation does not begin from nothing — including, where relevant, a prior Recommendation the customer may be referencing.

### 2. Responsibilities
Load durable preferences, priorities, constraints, decision style, and relationship state for an Existing Contact; skip retrieval entirely for a confirmed New Contact. Where the current message plausibly references a prior recommendation, retrieve the relevant record from Recommendation History alongside the Customer Memory.

### 3. Non-Responsibilities
Does not interpret or update the Mental Model — that belongs to the Customer Engine. This Engine only retrieves what is already stored. It does not evaluate whether a retrieved Recommendation's Preconditions still hold — that belongs to the Reasoning Engine, which alone has access to freshly loaded Project Brain data to check them against.

### 4. Inputs
The Contact Status and record reference from Contact Resolution.

### 5. Outputs
A Customer Memory object, per the standards of what is worth retaining established in `Memory_Model.md` and `Customer_Knowledge.md`, together with the relevant prior Recommendation from Recommendation History, where the current message plausibly references one.

### 6. Dependencies
Requires Contact Resolution to have completed.

### 7. Consumes
The record reference.

### 8. Produces
The Customer Memory object, and where relevant, the referenced prior Recommendation, both consumed downstream by the Customer Engine and, ultimately, the Reasoning Engine.

### 9. Internal Decisions
None beyond retrieval itself.

### 10. Boundaries
Retrieval only — never interprets or revises what it retrieves.

### 11. Failure Modes
Storage unavailable; partial record corruption.

### 12. Recovery Strategy
Proceed with an empty or partial memory object rather than halting; log this as a degraded, not silent, condition.

### 13. Confidence Behavior
Does not apply — retrieval, not judgment.

### 14. Knowledge Usage
None.

### 15. Memory Usage
Yes — this is its entire purpose.

### 16. Business Rules
Bound by the privacy and retention principles in `Customer_Knowledge.md`.

### 17. State Behavior
Stateful.

### 18. Side Effects
None — read-only.

### 19. Observability
Log retrieval success or failure, and whether retrieval was skipped for a New Contact.

### 20. Performance Expectations
Should be fast — a lookup, not a judgment.

### 21. Security Considerations
Must retrieve only the record matched to this specific customer, never another's.

### 22. Extensibility
The underlying storage mechanism can change without affecting this Engine's contract.

### 23. Out of Scope
Interpreting or updating memory content.

### 24. Invocation Condition
Every turn, conditioned on Contact Resolution's output — skipped entirely for a confirmed New Contact.

### 25. Determinism Classification
Deterministic.

### 26. Engine Invariants
- Must never retrieve another customer's record.
- Must never fabricate memory content where none exists.
- Must degrade gracefully rather than halt the turn on a partial failure.
