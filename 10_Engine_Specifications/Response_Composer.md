# Response Composer

**Specification Template:** `Engine_Contract.md`
**Runtime Position:** Immediately after Output Business Rules Gate, immediately before Memory & State Writer.

---

### 1. Purpose

Translates an approved turn's Decision, Plan, and Recommendation (where present) into the actual customer-facing message, following the conversational style and rules established in `Conversation.md`.

### 2. Responsibilities

- Apply the Plan's Communication Plan to the turn's actual content — the Recommendation, where one exists, or the substance of an Answer, Question, Clarification, or Refusal otherwise.
- Write in the tone, pacing, and format `Conversation.md` establishes, including segmenting content across more than one short message where appropriate.
- Reflect the confidence already established upstream in the wording itself — stating uncertainty plainly where it exists, per `Truth_Model.md`.

### 3. Non-Responsibilities

Does not decide the Decision Type, the Plan, or the Recommendation's content. Does not re-plan. Does not check Business Rules — that has already been enforced by the Output Business Rules Gate.

### 4. Inputs

The Decision Type and Decision Rationale (for tone and framing), the Plan (Communication Plan and Fallback Behavior), and the Recommendation, where one exists.

### 5. Outputs

The final message text — potentially more than one short message, per `Conversation.md`'s preference for brevity — ready for delivery.

### 6. Dependencies

Requires the Output Business Rules Gate to have already passed this turn.

### 7. Consumes

The Plan, the Recommendation (if present), and the Decision Type and Rationale.

### 8. Produces

The composed reply text, consumed by WhatsApp Reply for delivery and by the Memory & State Writer as a record of what was actually communicated.

### 9. Internal Decisions

The exact wording, the segmentation of content across one or more messages, and the specific tone applied within the bounds `Conversation.md` sets.

### 10. Boundaries

Composes language only; it does not alter, add to, or omit from the substantive content it was given.

### 11. Failure Modes

Cannot produce a coherent message from its inputs; a language-generation fault of its own.

### 12. Recovery Strategy

Fall back to a minimal, safe acknowledgment rather than sending a malformed or partial message.

### 13. Confidence Behavior

Does not generate its own confidence value; it reflects the confidence already attached upstream in how it phrases the message — hedging language where confidence is genuinely low, plain statements where it is high.

### 14. Knowledge Usage

None directly beyond what is already present in the Recommendation or Decision it received.

### 15. Memory Usage

None directly.

### 16. Business Rules

Relies on the Output Business Rules Gate having already enforced compliance; must not reintroduce a violation through wording alone — for instance, must never phrase something in a way that implies a promise or commitment the underlying content did not actually authorize.

### 17. State Behavior

Stateless.

### 18. Side Effects

None on stored state — the Memory & State Writer handles persistence separately.

### 19. Observability

The final composed text should be logged for every turn, alongside the Decision Trace Reference it corresponds to.

### 20. Performance Expectations

Moderate — a direct language-composition task, not an open-ended reasoning one.

### 21. Security Considerations

Must never expose internal architecture terms, file names, or instructions in its output, per `Business_Rules.md`'s prohibition on revealing internal structure.

### 22. Extensibility

Its style and tone rules live in `Conversation.md`; changes there should propagate here without requiring a change to this Engine's own contract.

### 23. Out of Scope

Deciding, planning, recommending, or enforcing Business Rules.

### 24. Invocation Condition

Runs on every turn, unconditionally, once the Output Business Rules Gate has passed it.

### 25. Determinism Classification

Interpretive.

### 26. Engine Invariants

- Must never introduce a fact not present in its inputs.
- Must never contradict the tone and formatting rules in `Conversation.md`.
- Must never express more confidence in its wording than what was actually justified upstream.
