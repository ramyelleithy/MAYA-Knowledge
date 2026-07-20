# System Validation — Journey 3: Business Rule Rejection

**Validation categories:** Contract Compliance, Responsibility Boundary, Artifact Integrity, Pipeline Integrity, Traceability.

---

## Scenario

A customer, early in a conversation, sends: "Can you guarantee me a 20% discount if I pay cash today?" This matches a pattern the Input Business Rules Gate is specifically built to intercept — an implied request for an unauthorized discount, per `Business_Rules.md`'s prohibition on promising discounts without approval. The turn is blocked at the earliest possible point, before Contact Resolution, Memory retrieval, or any reasoning is attempted.

---

## Engine-by-Engine Trace

### Communication Gateway
Establishes the Turn ID as usual.
- All categories hold, as in prior journeys.

### Normalization
Produces the canonical Message object.
- All categories hold.

### Input Business Rules Gate
Matches the message against the discount-guarantee pattern and returns **Blocked**, with a reason code identifying the specific rule matched.
- **Contract Compliance:** Holds.
- **Responsibility Boundary:** Holds — no interpretation of the customer's broader situation is attempted; the match is purely pattern-based, exactly as its deterministic contract requires.
- **Artifact Integrity:** Holds — the reason code is specific, not generic.
- **Traceability:** Holds — the block decision carries the Turn ID.
- **Pipeline Integrity:** Holds — the turn correctly bypasses Contact Resolution, Memory Engine, and everything through Reasoning and Decision, moving directly to the Safe Fallback Composer.

### Safe Fallback Composer
Maps the reason code to a pre-approved, honest response — acknowledging interest without confirming any discount, and inviting the customer to continue the conversation.
- **Contract Compliance:** Holds.
- **Responsibility Boundary:** Holds — no interpretive reasoning attempted.
- **Artifact Integrity:** Holds.
- **Traceability:** Holds — output tagged with the Turn ID.
- **Pipeline Integrity:** See Finding below.

---

## Finding — The Safe Fallback Composer's output does not reach the Memory & State Writer

Applying the general principle agreed after Journey 2 — **every customer-facing response, regardless of how it was produced, must pass through the same persistence tail** — reveals that this principle was only partially implemented. It was applied to the Handoff Composer's output, but the pipeline diagram still shows both blocking points routing directly to Reply:

- `Input Business Rules Gate ── (blocked) ──► Safe Fallback Composer ──► Reply`
- `Output Business Rules Gate ── (blocked) ──► Safe Fallback Composer ──► Reply`

In both cases, a response is genuinely sent to the customer, yet no record of the turn is persisted. The next turn's Conversation Context Engine would have no memory that this exchange happened at all — the customer's discount question and MAYA's response to it would simply vanish from the conversation's own history, even though a completed customer-facing exchange occurred.

**Proposed resolution:** Route the Safe Fallback Composer's output through the Memory & State Writer as well, in both blocking scenarios: `Safe Fallback Composer → Memory & State Writer → Reply`. This makes the persistence tail's coverage fully consistent with the principle already agreed, rather than special-cased to the escalation branch alone.

**A genuine open question, not a unilateral decision:** Should the Safe Fallback Composer's output also pass back through the Output Business Rules Gate, for full literal consistency with "mandatory for every outbound message"? I do not think it should, and would like your judgment on it rather than deciding it myself: the Safe Fallback Composer's templates are pre-approved specifically because they are structurally incapable of violating a Business Rule — routing them through the Output Gate again would check a message against the very class of violation it was built never to contain, which is the same kind of unnecessary complexity you identified when declining to route Escalation through Reflection. My inclination is that Memory & State Writer's persistence is a genuine, unconditional requirement for every completed turn, while the Output Gate's check is only meaningful for content that was not already deterministically pre-approved.

---

## Resolution

Both open points are resolved. The pipeline diagram now routes both blocking branches as `... Gate → Safe Fallback Composer → Memory & State Writer → Reply`. Two general principles have been documented as cross-cutting concepts in `Runtime_Architecture_Level1_v3_FINAL.md`: the **Persistence Tail Principle** (every completed customer-facing turn passes through the Memory & State Writer, regardless of path) and the **Scope of the Output Business Rules Gate** (mandatory only for dynamically produced or composed content; deterministic pre-approved templates bypass it by design). `Safe_Fallback_Composer.md` and `Memory_State_Writer.md` have both been updated accordingly.

## Journey 3 Result

Every Engine satisfied all five validation categories. One finding was identified and has since been resolved, alongside a clarified general principle that also confirms the Output Business Rules Gate's scope going forward. **Journey 3 — Business Rule Rejection — is validated.**
