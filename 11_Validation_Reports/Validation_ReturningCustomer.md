# System Validation — Journey 4: Regression / Returning Customer

**Validation categories:** Contract Compliance, Responsibility Boundary, Artifact Integrity, Pipeline Integrity, Traceability.

---

## Scenario

An existing customer returns after a two-week gap. In a prior turn, MAYA produced a Recommendation for a specific unit, with Preconditions stating the unit's availability and price were current as of that turn. The customer now writes: "Is that unit you mentioned still available? I want to move forward." This journey is chosen to exercise continuity across a genuine time gap, referencing a past Recommendation rather than a past preference — something none of the first three journeys required.

---

## Engine-by-Engine Trace

### Contact Resolution
Matches the existing record. Contact Status = Existing.
- All categories hold, as in Journey 2.

### Memory Engine
Retrieves the existing Customer Memory: prior stated preferences, budget, and the project already under discussion.
- **Contract Compliance:** Holds against `Memory_Model.md`'s and `Customer_Knowledge.md`'s stated categories of what is worth retaining.
- **Pipeline Integrity:** Holds. **See Finding below** — the categories retained do not currently include MAYA's own prior Recommendation, which this turn's customer message directly depends on.

### Conversation Context Engine
Retrieves the prior conversation history, including the turn in which the Recommendation was originally given.
- All categories hold — the raw conversation history is available; the question is whether the structured Recommendation object itself is separately retrievable, addressed below.

### Conversation State Recognition
Recognizes a position consistent with returning to a near-decision point — closer to Decision Support than Entry, since the customer is referencing a specific prior recommendation and expressing readiness — with moderate-high confidence.
- All categories hold. This is the first journey to validate recognition of a position resuming mid-journey after a gap, rather than starting fresh or already being mid-conversation within a single session.

### Intent Understanding
Classifies the message as Intent — a forming purpose to proceed, contingent on confirmation the unit is still available — distinct from outright Commitment, since it is conditioned on the answer to the customer's own question.
- All categories hold.

### Project Engine
The project is already established; detection is correctly skipped. The Brain Loader reloads the Project Brain fresh for this turn, per its contract — correctly addressing the two-week gap by re-reading current knowledge rather than assuming nothing has changed.
- **Pipeline Integrity:** Holds, and positively confirms `Project_Lifecycle.md`'s concern about temporal knowledge staleness is already handled at this layer: the Project Brain itself is never treated as still valid purely because it was valid two weeks ago.

### Customer Engine
Updates the Mental Model: readiness-to-proceed hypothesis strengthens; the customer's reference to "that unit you mentioned" is noted as pointing to a specific prior recommendation, though the Customer Engine holds only the customer-facing preferences and hypotheses this repository already defines as memory — not MAYA's own prior output.
- All categories hold structurally. This is where the underlying gap becomes visible, addressed below.

### Reasoning Engine
Synthesizes a Situation Assessment: a returning customer referencing a specific unit, wanting confirmation before proceeding. Supporting Evidence can cite the current Project Brain's availability data directly. An Unknown is surfaced: which specific unit and price were actually quoted two weeks ago, since the original Recommendation's specifics are not present in what Memory retained.
- **Artifact Integrity:** Holds structurally, but the surfaced Unknown is itself evidence of the underlying gap — the Reasoning Engine has to treat something as unknown that MAYA itself determined and stated two weeks ago.

---

## Finding — Prior Recommendations are not retained as retrievable artifacts, so their Preconditions can never be re-verified

`Recommendation.md` defines Preconditions specifically so that a recommendation's continued validity can be checked — explicitly citing that a returning conversation, per `Conversation_State_Model.md`, is exactly the situation this exists for. But `Memory_Model.md` and `Customer_Knowledge.md`'s categories of what is worth retaining describe only customer-facing information — preferences, priorities, constraints, decision style, objections, goals. Neither includes MAYA's own prior Recommendation objects. `Memory_State_Writer.md` persists "a record of the turn" referencing its Decision Trace ID, but does not specify that the Recommendation object itself, with its Preconditions, is retained in a form later retrievable.

The result: when a customer references a specific prior recommendation, as in this scenario, there is currently no architectural mechanism for retrieving what was actually recommended, and therefore no way to re-verify its Preconditions rather than treating the reference as an Unknown to rediscover from scratch. This defeats much of the purpose Preconditions was introduced to serve.

**Proposed resolution, offered for your judgment rather than applied unilaterally:**
1. Extend `Memory_State_Writer.md`'s persisted turn record to include the full Recommendation object, where one was produced, not only a reference to the Decision Trace ID.
2. Add an explicit responsibility to the Reasoning Engine (or, arguably more precisely, to the Project Engine, since it already reloads current Project Brain data every turn): when a customer's message references a prior recommendation, retrieve the persisted Recommendation object and re-verify its Preconditions against the freshly loaded Project Brain, rather than treating the referenced recommendation as unknown.
3. Where a Precondition no longer holds — the unit is no longer available, or the price has changed — this becomes exactly the kind of Conflict the Reasoning Conclusion is already built to surface, and the Recommendation Engine would then need to run again to produce an updated Recommendation rather than reuse the stale one.

I would lean toward placing this responsibility on the Reasoning Engine rather than the Project Engine, since re-verifying a specific prior Recommendation against current knowledge is an act of judgment connecting old and new evidence — closer to reasoning than to project retrieval — but I would like your view before this is written into any Engine's contract.

---

## Resolution

Resolved with a cleaner separation than either option I proposed. Rather than assigning historical-recommendation retrieval to the Reasoning Engine or the Project Engine, a new persisted artifact — **Recommendation History** — has been introduced, and the existing retrieval/reasoning boundary is preserved exactly:

- `Memory_State_Writer.md` now persists the full Recommendation object, where one was produced, tagged with a Recommendation ID, Turn ID, Decision Trace Reference, and timestamp.
- `Memory_Engine.md` now retrieves the relevant prior Recommendation from that history alongside Customer Memory, when the current message plausibly references one — retrieval only, no evaluation.
- `Reasoning_Engine.md` now evaluates whether a retrieved prior Recommendation's Preconditions still hold against the freshly loaded Project Brain, surfacing any that no longer hold as an ordinary Conflict — reasoning only, no retrieval.

No Engine took on a responsibility outside its existing boundary: the Memory Engine retrieves, the Project Engine loads current project knowledge, the Reasoning Engine reasons, and the Recommendation Engine recommends — exactly as before.

## Journey 4 Result

Every Engine satisfied all five validation categories once this resolution was applied. **Journey 4 — Regression / Returning Customer — is validated.**
