# Project Engine

**Specification Template:** `Engine_Contract.md`
**Runtime Position:** Immediately after Intent Understanding, immediately before Customer Engine. (Combines Project Detection and Project Brain Loader.)

---

### 1. Purpose
Identifies which project this conversation concerns and loads its complete knowledge, so the Customer Engine and Reasoning Engine can interpret the situation in relation to it rather than in the abstract.

### 2. Responsibilities
On first contact, map the referral code to a project deterministically; on later turns, recognize whether a genuine shift is warranted, per the Recommendation Rule in `Business_Rules.md`; load the identified Project Brain in full.

### 3. Non-Responsibilities
Does not evaluate whether the project actually suits the customer — that belongs to the Recommendation Engine. Does not switch projects outside the conditions the Recommendation Rule sets.

### 4. Inputs
The canonical Message object (including referral data where present) and the current Conversation State.

### 5. Outputs
A Project Identifier (or a "no change" signal) and the loaded Project Brain.

### 6. Dependencies
Requires Normalization and Conversation State Recognition to have completed.

### 7. Consumes
Referral data, Conversation State, and Project Knowledge sources.

### 8. Produces
The Project Context, consumed by the Customer Engine, the Reasoning Engine, and, where relevant, the Recommendation Engine.

### 9. Internal Decisions
Whether a shift away from the established project is genuinely warranted.

### 10. Boundaries
Identification and loading only — never a suitability judgment.

### 11. Failure Modes
No project can be identified; the Project Brain is incomplete or unavailable.

### 12. Recovery Strategy
Where no project can be confidently identified, proceed toward a state that requests clarification rather than guessing. Where the Project Brain is incomplete, proceed transparently about the limitation, per `Truth_Model.md`.

### 13. Confidence Behavior
The shift-detection judgment carries a confidence value under the Confidence Contract; the deterministic first-contact mapping does not.

### 14. Knowledge Usage
Yes, directly and centrally.

### 15. Memory Usage
Yes — checks whether a project is already established before attempting to re-detect one.

### 16. Business Rules
Directly applies the Recommendation Rule: default to the Entry Project, and only widen or shift where a genuine mismatch has been shown.

### 17. State Behavior
Stateful — its output persists across the conversation once established.

### 18. Side Effects
None beyond its own output.

### 19. Observability
Log every detection and shift decision, with confidence where the judgment is interpretive.

### 20. Performance Expectations
The deterministic mapping is fast; the Brain load and shift-judgment carry moderate cost.

### 21. Security Considerations
Must not let a project's confidential commercial knowledge leak into a context where it could reach the wrong customer or appear inappropriately as an Alternative.

### 22. Extensibility
New projects are added via Project Knowledge sources without requiring a change to this Engine's contract.

### 23. Out of Scope
Suitability judgment; recommendation.

### 24. Invocation Condition
Conditional in its detection component — it only re-runs detection when no project is yet established or a shift is genuinely warranted; otherwise it reuses the already-established project. The Brain Loader runs whenever a project is confirmed.

### 25. Determinism Classification
Hybrid.

### 26. Engine Invariants
- Must never guess a project without genuine confidence.
- Must never switch projects without a shift genuinely warranted per `Business_Rules.md`.
- Must never load a Project Brain before confirming the project's identity.
