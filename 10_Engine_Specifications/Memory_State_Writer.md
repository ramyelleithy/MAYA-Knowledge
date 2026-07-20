# Memory & State Writer

**Specification Template:** `Engine_Contract.md`
**Runtime Position:** Immediately after Response Composer, immediately before WhatsApp Reply.

---

### 1. Purpose

Persists everything from this turn that should carry forward — the updated Mental Model, any changes to Customer Knowledge, and the current Conversation State — so the continuous understanding described throughout this repository actually survives past the end of a single turn.

### 2. Responsibilities

- Persist the updated Mental Model produced by the Customer Engine.
- Persist any updates to Customer Knowledge warranted by this turn, per `Customer_Knowledge.md`'s standard of what is worth retaining.
- Persist the Conversation State reached by the end of this turn.
- Where a Recommendation was produced this turn, persist it in full as a **Recommendation History** record — the Recommendation object itself, tagged with a Recommendation ID, the Turn ID, the Decision Trace Reference, and a timestamp — so it can be retrieved and re-evaluated on a future turn.
- Record the turn — its Decision Trace Reference, Decision Type, and outcome — for future review.

### 3. Non-Responsibilities

Does not decide what should be remembered — that judgment belongs to the Customer Engine and the standards in `Memory_Model.md`; this Engine persists what it is given. Does not compose or deliver anything customer-facing.

### 4. Inputs

The updated Mental Model, the Conversation State, the Decision Type and outcome, the final composed message — whether that message came from the Response Composer on the normal path, the Handoff Composer on the escalation path (after passing the Output Business Rules Gate), or the Safe Fallback Composer on either blocked path — and, where one was produced, the Recommendation object.

### 5. Outputs

Confirmation that persistence succeeded. No customer-facing output.

### 6. Dependencies

Requires the Customer Engine's Mental Model update and the final Conversation State for this turn to already exist.

### 7. Consumes

The updated Mental Model, Conversation State, Decision Trace Reference, Decision Type, and composed message.

### 8. Produces

The persisted state that the Memory Engine, Conversation Context Engine, and Conversation State Recognition will read on the customer's next turn, including any Recommendation History available for the Memory Engine to retrieve.

### 9. Internal Decisions

None of substance — it persists what it receives rather than exercising independent judgment about what deserves to persist.

### 10. Boundaries

Writes state; it does not interpret, filter, or second-guess what it is given to write.

### 11. Failure Modes

Persistence fails partially or entirely due to a system fault.

### 12. Recovery Strategy

The reply must still be sent to the customer even if persistence fails, but the failure must be logged loudly — a silently lost update to memory quietly degrades every future turn with this customer, and must never pass unnoticed.

### 13. Confidence Behavior

Does not apply — this Engine does not make an assertion about the world; it records what has already been concluded elsewhere.

### 14. Knowledge Usage

None.

### 15. Memory Usage

This is where Memory is written, not merely read — the defining responsibility of this Engine.

### 16. Business Rules

None apply directly beyond the general privacy and retention principles already established in `Customer_Knowledge.md`.

### 17. State Behavior

Stateful — by definition, this is where state is written.

### 18. Side Effects

Yes, deliberately — persisting updated memory, customer knowledge, and conversation state is its entire purpose.

### 19. Observability

Every persistence attempt, and especially every failure, must be logged; a failed write here is one of the highest-priority failures in the entire runtime, since its effects compound silently across future turns if unnoticed.

### 20. Performance Expectations

Should be fast; this is a write operation, not a judgment-based one.

### 21. Security Considerations

Must uphold the privacy principles in `Customer_Knowledge.md` when persisting anything about a customer, and must not persist information beyond what those principles consider worth retaining.

### 22. Extensibility

What is considered worth persisting is governed by `Memory_Model.md` and `Customer_Knowledge.md`; this Engine's own contract should not need to change as those standards evolve, since it persists whatever it is handed rather than deciding independently what qualifies.

### 23. Out of Scope

Deciding what is worth remembering; composing or delivering any message; reasoning, deciding, planning, or recommending.

### 24. Invocation Condition

Runs on every completed turn, unconditionally, once a customer-facing message has been produced — by the Response Composer on the normal path, the Handoff Composer on the escalation path (after passing the Output Business Rules Gate), or the Safe Fallback Composer on either blocked path. This Engine is never skipped, regardless of which path a turn takes, per the Persistence Tail Principle.

### 25. Determinism Classification

Deterministic.

### 26. Engine Invariants

- Must never silently drop a persistence failure without logging it loudly.
- Must never persist information beyond what `Customer_Knowledge.md` and `Memory_Model.md` consider worth retaining.
- Must never delay or block the customer's reply while attempting persistence — the reply proceeds regardless of this Engine's outcome.
- Must never be skipped on any path — the normal path, the escalation path, or either blocked path — per the Persistence Tail Principle.
