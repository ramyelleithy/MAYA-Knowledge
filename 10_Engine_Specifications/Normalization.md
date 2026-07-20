# Normalization

**Specification Template:** `Engine_Contract.md`
**Runtime Position:** Immediately after Communication Gateway, immediately before Input Business Rules Gate.

---

### 1. Purpose
Converts a channel-native message into a single canonical Message object regardless of its original form — text, voice, image, or document — so every later Engine can reason about "a message" without caring how it arrived.

### 2. Responsibilities
Parse the raw envelope; transcribe voice content to text where applicable; classify message type; carry forward referral metadata unaltered.

### 3. Non-Responsibilities
Does not interpret intent or meaning beyond format conversion. Does not decide project or customer identity.

### 4. Inputs
The raw channel-native envelope and metadata from the Communication Gateway.

### 5. Outputs
A canonical Message object: sender identity, text content, message type, timestamp, and referral data.

### 6. Dependencies
Requires the Communication Gateway to have already produced the envelope.

### 7. Consumes
The raw envelope.

### 8. Produces
The canonical Message object every downstream Engine builds on.

### 9. Internal Decisions
How to classify message type; whether a transcription is usable enough to proceed on.

### 10. Boundaries
Format conversion only — never interprets what a message means.

### 11. Failure Modes
Unsupported format; transcription failure; corrupted content.

### 12. Recovery Strategy
Where content cannot be normalized, exit early with a safe, generic acknowledgment rather than proceeding with malformed input.

### 13. Confidence Behavior
Transcription quality may carry a confidence value under the Confidence Contract, since a poor transcription affects everything downstream; not applicable for plain text messages.

### 14. Knowledge Usage
None.

### 15. Memory Usage
None.

### 16. Business Rules
None apply directly.

### 17. State Behavior
Stateless.

### 18. Side Effects
None.

### 19. Observability
Log message type and normalization outcome, including transcription confidence where relevant.

### 20. Performance Expectations
Fast; only partial LLM use, limited to voice transcription.

### 21. Security Considerations
Must handle any customer-uploaded content (voice, image, document) consistent with privacy principles even at this early stage.

### 22. Extensibility
New message formats can be supported without altering any downstream Engine, since they all consume the same canonical shape.

### 23. Out of Scope
Interpretation, project detection, customer identity.

### 24. Invocation Condition
Every turn, unconditionally.

### 25. Determinism Classification
Hybrid — format parsing is deterministic; voice transcription is interpretive.

### 26. Engine Invariants
- Must never pass a message downstream that has not been reduced to the canonical shape.
- Must never silently drop referral metadata.
- Must never treat a failed or low-quality transcription as though it succeeded cleanly.
