# Release Notes — Phoenix Runtime, Level 1 Architecture v1.0

This release captures every architectural change that emerged during the five-journey validation pass conducted after the Level 1 Runtime and its Engine specifications were first completed. Changes are grouped by the journey that surfaced them.

---

## Journey 1 — Happy Path

**Finding 1: No Turn-level identifier existed prior to the Decision Engine.**
Resolved by introducing the **Turn Identity Contract** as a cross-cutting runtime concept, documented in `Runtime_Architecture.md`. Every artifact produced within a turn now carries a Turn ID, the producing Engine's name, a timestamp or sequence marker, and its artifact type. The Turn ID is established at the Communication Gateway and carried forward by every Engine from that point on. The Decision Trace ID remains a narrower reference to the specific decision made within a turn, nested inside the Turn ID's broader scope. `Engine_Contract.md`'s Observability field (19) was updated so every Engine inherits this baseline without needing to restate it individually.

**Finding 2: No stated rule for how the Reasoning Engine's Confidence should aggregate multiple upstream confidence values.**
Resolved with a **dependency-aware rule**: the Situation Assessment's Confidence must never exceed the lowest confidence among the upstream inputs that are genuinely load-bearing for the specific conclusion reached — not a blanket minimum across every available signal. Documented in both `Reasoning_Conclusion.md`'s Confidence component and `Reasoning_Engine.md`'s Confidence Behavior field.

---

## Journey 2 — Escalation

**Finding: The escalation branch bypassed both the Output Business Rules Gate and the Memory & State Writer.**
As originally specified, a turn escalated by the Decision Engine went `Handoff Composer → Reply` directly — with no deterministic safety check on the handoff message, and no persistence of the understanding reached at the point of handoff.

Resolved by establishing the principle that **every customer-facing response, regardless of how it was produced, must pass through the same safety and persistence tail**. The corrected branch: `Decision Engine → (escalation) → Handoff Composer → Output Business Rules Gate → Memory & State Writer → WhatsApp Reply`. Reflection remains correctly excluded from this branch, since a mandatory escalation is a deterministic rule match, not an interpretive judgment requiring review.

---

## Journey 3 — Business Rule Rejection

**Finding: Both Business Rules Gates' blocked paths bypassed the Memory & State Writer.**
Applying the principle from Journey 2 more broadly revealed it had only been applied to the escalation branch. Both `Input Business Rules Gate → Safe Fallback Composer → Reply` and `Output Business Rules Gate → Safe Fallback Composer → Reply` were missing the persistence step.

Resolved by extending the same persistence tail to both blocked paths: `... Gate → Safe Fallback Composer → Memory & State Writer → Reply`. This produced two documented general principles in `Runtime_Architecture.md`:
- **The Persistence Tail Principle** — every completed customer-facing turn passes through the Memory & State Writer, regardless of path.
- **Scope of the Output Business Rules Gate** — mandatory only for dynamically produced or composed content; deterministic, pre-approved templates (such as the Safe Fallback Composer's) are compliant by construction and bypass the gate by design.

---

## Journey 4 — Regression / Returning Customer

**Finding: Prior Recommendations were not retained as retrievable artifacts, so their Preconditions could never be re-verified on a later turn.**
`Recommendation.md`'s Preconditions component exists specifically to support this kind of check, but no mechanism existed to retrieve what had actually been recommended in a prior turn.

Resolved by introducing **Recommendation History** as a new persisted artifact, with responsibilities distributed cleanly across existing Engine boundaries — no Engine gained a responsibility outside its own domain:
- `Memory_State_Writer.md` now persists the full Recommendation object where one was produced, tagged with a Recommendation ID, Turn ID, Decision Trace Reference, and timestamp.
- `Memory_Engine.md` now retrieves the relevant prior Recommendation alongside Customer Memory, when the current message plausibly references one — retrieval only.
- `Reasoning_Engine.md` now evaluates whether a retrieved prior Recommendation's Preconditions still hold against the freshly loaded Project Brain, surfacing any that no longer hold as an ordinary Conflict — reasoning only.

---

## Journey 5 — Recommendation Tie & Preconditions

**No architectural flaw found.** The Recommendation Engine's tie-handling Recovery Strategy, the Planning Engine's scope-widening on an explicit customer request for alternatives, and Reflection's review of a genuinely tied outcome all behaved exactly as specified.

**One observation was recorded and deliberately not acted on:** `Recommendation.md`'s Preconditions component does not distinguish a stable precondition from a more fragile, time-sensitive one. This is left as a candidate for future, evidence-justified refinement rather than acted on now, per the architecture-freeze condition agreed for this journey.

---

## Summary of Net Changes

- 1 new cross-cutting contract added: **Turn Identity Contract**.
- 1 aggregation rule clarified: **dependency-aware Confidence**.
- 2 general principles documented: **Persistence Tail Principle**, **Scope of the Output Business Rules Gate**.
- 1 new persisted artifact introduced: **Recommendation History**.
- 7 Engine specifications updated to reflect the above: `Communication_Gateway.md`, `Reasoning_Engine.md`, `Output_Business_Rules_Gate.md`, `Memory_State_Writer.md`, `Handoff_Composer.md`, `Safe_Fallback_Composer.md`, `Memory_Engine.md`.
- 0 changes made on the basis of hypothetical improvement alone once the freeze condition was in effect (Journey 5).
