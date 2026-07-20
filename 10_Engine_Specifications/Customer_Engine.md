# Customer Engine

**Specification Template:** `Engine_Contract.md`
**Runtime Position:** Immediately after Project Engine, immediately before Context Builder.

---

### 1. Purpose
Updates the customer's Mental Model, now informed by the Project Context, so that understanding a customer's need is always interpreted in relation to the specific project under discussion.

### 2. Responsibilities
Confirm, contradict, create, or adjust confidence on hypotheses about the customer's needs, goals, constraints, priorities, and likely role, per `Mental_Model.md` and `Customer_Model.md`.

### 3. Non-Responsibilities
Does not decide the Decision Type. Does not produce a recommendation. Does not treat role as a fixed label — only ever as a revisable hypothesis.

### 4. Inputs
The canonical Message object, the Intent classification, the Project Context, and the prior Mental Model carried from Memory.

### 5. Outputs
An updated Mental Model, with an explicit confidence value per hypothesis.

### 6. Dependencies
Requires Intent Understanding and the Project Engine to have completed.

### 7. Consumes
The Message, the Intent classification, the Project Context, and the prior Mental Model.

### 8. Produces
The Customer Context, consumed by the Context Builder, the Reasoning Engine, the Planning Engine, and the Recommendation Engine.

### 9. Internal Decisions
Which hypotheses to confirm, contradict, create, or adjust, and by how much.

### 10. Boundaries
Understanding only — never recommendation or decision.

### 11. Failure Modes
An update cannot be responsibly made given the available signal.

### 12. Recovery Strategy
Carry the prior Mental Model forward unchanged rather than discarding accumulated understanding due to a single failed update.

### 13. Confidence Behavior
Bound by the Confidence Contract, tracked per individual hypothesis.

### 14. Knowledge Usage
None directly.

### 15. Memory Usage
Yes, directly — both reads the prior Mental Model and produces its update.

### 16. Business Rules
None apply directly.

### 17. State Behavior
Stateful.

### 18. Side Effects
None beyond its own output — actual persistence happens later, at the Memory & State Writer.

### 19. Observability
Log every hypothesis change and its confidence.

### 20. Performance Expectations
Moderate.

### 21. Security Considerations
Bound by the privacy principles in `Customer_Knowledge.md`.

### 22. Extensibility
The hypothesis categories are defined in `Mental_Model.md` and `Customer_Model.md`; changes there should propagate here.

### 23. Out of Scope
Recommendation; decision-making.

### 24. Invocation Condition
Every turn, unconditionally.

### 25. Determinism Classification
Interpretive.

### 26. Engine Invariants
- Must never treat a hypothesis as confirmed fact.
- Must never discard the prior Mental Model without genuinely contradicting evidence.
- Must never finalize a role classification as permanent.
