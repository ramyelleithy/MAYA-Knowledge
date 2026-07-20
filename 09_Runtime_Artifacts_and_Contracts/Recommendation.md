# Recommendation

## Status

**Final.** This definition, including the independent Preconditions component added on review, is now a formal part of MAYA's architecture. `Recommendation_Engine.md` is written against it.

---

## What Is a Recommendation?

A Recommendation is not text. It is not "the best project." It is not a list of projects. It is a structured object representing the specific consultative decision MAYA has reached about what would genuinely help this customer move forward — in whatever form that decision actually takes, per `Recommendation_Model.md`: a project, an area, a unit type, a suggestion to delay a purchase, a request to gather more information, or a transfer to a call or a site visit.

It exists only once the Decision Engine has already classified this turn's Decision Type as Recommendation. It is produced for the Planning Engine to consume — Planning builds the execution plan around it; it does not build the Recommendation itself.

## Why It Exists

Without a distinct, contracted object, "the recommendation" would be whatever informal shape happened to emerge from the Recommendation Engine's internal process — inconsistent from one turn to the next, and impossible for the Planning Engine to reliably build a plan around. Defining this object precisely is what allows the Recommendation Engine and the Planning Engine to remain genuinely separate responsibilities: one decides what is being recommended and why; the other decides how and when it reaches the customer.

## What It Must Contain

- **Recommendation Type** — which recognized form this recommendation takes: a specific project or unit, an area, a deferral of purchase, a request for further information, or a transfer to a call or site visit. Every Recommendation must declare its type explicitly, since Planning's execution approach differs meaningfully by type.
- **Primary Option** — the specific entity or action being recommended, stated precisely enough for Planning to act on without needing to interpret or infer it.
- **Grounding** — the specific confirmed facts, hypotheses, and Decision-Relevant Signals this recommendation is built from, referenced rather than reproduced in full, so the recommendation can be traced back to what actually supports it — mirroring how Supporting Evidence grounds a Reasoning Conclusion.
- **Supporting Rationale** — the reasoning connecting the customer's understood priorities and situation to why the Primary Option genuinely fits them, stated clearly enough that its logic could be explained to the customer if asked.
- **Trade-offs** — the honest costs, limitations, or open questions attached to the Primary Option. A Recommendation that omits its own trade-offs to appear artificially ideal has failed `Consultation_Methodology.md`'s requirement that every option's genuine drawbacks be held with the same weight as its strengths.
- **Preconditions** — what must remain true for this recommendation to still be valid: a budget threshold not exceeded, a unit's continued availability, financing eligibility, geographic acceptance, or any similarly hard condition. This is deliberately distinct from Trade-offs: a trade-off is a cost or concession accepted within a recommendation that remains valid; a precondition is a fact whose failure means the recommendation is no longer valid at all. Grounding and Supporting Rationale explain why the recommendation was made; Preconditions state what could make it stop being true — a forward-looking check, not a backward-looking justification. Given that `Project_Lifecycle.md` and `Knowledge_Quality.md` already establish that facts like price and availability are inherently temporary, and `Conversation_State_Model.md` allows a conversation to resume after days or weeks, a recommendation needs an explicit, checkable statement of what must still hold for it to remain safe to act on.
- **Alternatives** — where genuinely warranted, a small, bounded set of additional options (ordinarily no more than one alternative and, rarely, a single fallback), each carrying its own brief rationale. This is never an exhaustive comparison; `Recommendation_Model.md` is explicit that MAYA does not overwhelm a customer with a large number of options.
- **Confidence** — the degree of certainty behind this recommendation's fit, expressed per the degrees of certainty in `Truth_Model.md`, and tied directly to the quality of the understanding and evidence it is grounded in — never to how well-formed the recommendation happens to read.
- **Decision Trace Reference** — the identifier of the Decision Engine's classification that authorized this Recommendation to be produced at all, preserving the same end-to-end traceability already established for decisions.

## What It Must Not Contain

- **Composed, customer-facing language.** A Recommendation is content for Planning and, eventually, the Response Composer to work with — it is never itself a draft of what the customer will read.
- **A plan for how or when to deliver it.** Sequencing, pacing, and the shape of the conversation around this recommendation belong entirely to the Planning Engine. A Recommendation that already dictates its own delivery has quietly done Planning's job for it.
- **A re-classification of the Decision Type.** By the time this object is produced, the Decision Engine has already committed to Recommendation as this turn's action; the Recommendation object elaborates that decision, it does not revisit it.
- **Invented facts about any project, unit, or alternative.** Every element of Grounding must trace back to something actually confirmed in the Project Brain, Knowledge, or Memory — never filled in to make the recommendation feel more complete.
- **More options than a customer can reasonably weigh.** A Recommendation listing an unbounded or exhaustive set of alternatives has failed the same principle that governs conversation itself — clarity before complexity.
- **A confidence value greater than what its Grounding and Supporting Rationale actually justify.**

## Semantic Contract

Every element inside a Recommendation must describe a substantive consultative option and its honest basis — never how or when that option should be communicated. A Recommendation answers only one question: **what is being recommended, and why does it fit this customer.** It never answers **how this should be said, in what order, or across how many messages** — those questions belong entirely to the Planning Engine and the Response Composer downstream. The moment a Recommendation begins to shape its own delivery, it has crossed out of its own boundary and into a responsibility that was never its own.

## Components, Restated as a Structure

1. Recommendation Type
2. Primary Option
3. Grounding
4. Supporting Rationale
5. Trade-offs
6. Preconditions
7. Alternatives
8. Confidence
9. Decision Trace Reference

---

**Awaiting approval of this object definition before `Recommendation_Engine.md` is written.**
