# WhatsApp Reply

**Specification Template:** `Engine_Contract.md`
**Runtime Position:** Final stage of the Main Consultation Pipeline, mirroring the Communication Gateway at entry.

---

### 1. Purpose

Delivers the final composed message to the customer through the channel it arrived from — the single point of egress, mirroring the Communication Gateway's role as the single point of entry.

### 2. Responsibilities

Deliver the composed message to the customer's channel identifier, and report delivery success or failure.

### 3. Non-Responsibilities

Does not compose, decide, plan, or persist anything. It transmits what it is given.

### 4. Inputs

The final message text and the customer's channel identifier.

### 5. Outputs

A delivered message, and a delivery confirmation or failure signal.

### 6. Dependencies

Requires the Response Composer to have already produced the final message text.

### 7. Consumes

The final message text and channel identifier.

### 8. Produces

The delivery outcome — nothing else in the runtime depends on this Engine's output beyond confirming the turn has concluded.

### 9. Internal Decisions

None — this is a transport operation, not a judgment.

### 10. Boundaries

Delivery only; it does not alter the message or make any decision about its content.

### 11. Failure Modes

Delivery fails due to a channel or network-level fault.

### 12. Recovery Strategy

Retry according to standard delivery-layer policy; this is a transport concern, not a consultative one, and never requires re-running any earlier Engine.

### 13. Confidence Behavior

Does not apply.

### 14. Knowledge Usage

None.

### 15. Memory Usage

None.

### 16. Business Rules

None apply directly.

### 17. State Behavior

Stateless.

### 18. Side Effects

Yes — a message is actually sent to the customer, an effect visible outside the runtime itself.

### 19. Observability

Delivery success and failure should be logged, tied to the same Decision Trace Reference as the rest of the turn.

### 20. Performance Expectations

Should be fast; this is a transport operation.

### 21. Security Considerations

Must ensure the message is delivered only to the intended, verified channel identifier for this customer.

### 22. Extensibility

Should a new communication channel be added in the future, only this Engine (and its mirror, the Communication Gateway) needs to change — nothing upstream in the pipeline should need to know which channel a message is delivered through.

### 23. Out of Scope

Anything beyond transport and delivery confirmation.

### 24. Invocation Condition

Runs on every turn, unconditionally, once the Response Composer has produced a message.

### 25. Determinism Classification

Deterministic.

### 26. Engine Invariants

- Must never alter the message it was given to deliver.
- Must never deliver to a channel identifier other than the one verified for this customer and this turn.
- Must never silently drop a delivery failure without logging it.
