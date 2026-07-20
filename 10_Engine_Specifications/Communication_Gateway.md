# Communication Gateway

**Specification Template:** `Engine_Contract.md`
**Runtime Position:** Entry point of the Main Consultation Pipeline.

---

### 1. Purpose
The single point of contact with the external messaging channel — receives raw inbound events and hands them off untouched, so that no downstream Engine ever needs to know which channel a message arrived through. It is also where the Turn ID is established, giving every artifact produced for this turn a shared identifier from the very start.

### 2. Responsibilities
Receive the raw webhook payload from the messaging provider; extract channel metadata (sender identifier, timestamp, referral data where present); establish the Turn ID for this turn, per the Turn Identity Contract; pass the envelope, tagged with that Turn ID, to Normalization unaltered.

### 3. Non-Responsibilities
Does not interpret, transcribe, or normalize content. Does not decide anything about what a message means.

### 4. Inputs
The raw webhook payload from the messaging provider.

### 5. Outputs
A raw, channel-native message envelope plus channel metadata, and the newly established Turn ID.

### 6. Dependencies
None upstream within the runtime — this is the entry point.

### 7. Consumes
The external provider's payload format.

### 8. Produces
The envelope Normalization consumes, and the Turn ID that every subsequent artifact in this turn will carry.

### 9. Internal Decisions
None.

### 10. Boundaries
Channel reception only — no interpretation of any kind.

### 11. Failure Modes
Malformed payload; provider outage; duplicate delivery of the same event.

### 12. Recovery Strategy
Standard delivery-layer retry and dead-letter handling. A duplicate delivery must be detectable and must never be allowed to silently create two separate turns from one customer message.

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
None beyond acknowledging receipt to the provider.

### 19. Observability
Log every inbound event with its provider-level metadata, traceable back to the Decision Trace Reference once one exists for this turn.

### 20. Performance Expectations
Must be very fast — any delay here delays the entire pipeline.

### 21. Security Considerations
Must verify the payload genuinely originates from the legitimate provider (for example, signature verification) before passing it downstream.

### 22. Extensibility
Adding a new communication channel means adding a new instance of this Engine's contract for that channel, without requiring any change downstream.

### 23. Out of Scope
Interpretation, normalization, or any judgment about content.

### 24. Invocation Condition
Triggered by every inbound provider event, unconditionally.

### 25. Determinism Classification
Deterministic.

### 26. Engine Invariants
- Must never alter the payload it receives.
- Must never pass an unverified payload through as though it were verified.
- Must never allow a single inbound event to create more than one turn.
- Must never fail to establish a unique Turn ID for a genuinely new inbound event.
