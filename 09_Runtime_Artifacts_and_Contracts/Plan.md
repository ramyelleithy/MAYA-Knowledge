# Plan

## Status

This document defines the Plan as an architectural object — the structured artifact the Planning Engine produces, consumed by the conditional Recommendation Engine (for its Scope) and by the Response Composer (for its Communication Plan). It does not describe how a Plan is generated or any implementation detail. `Planning_Engine.md` is deliberately not written until this object is defined and approved, for the same reason `Reasoning_Conclusion.md` and `Recommendation.md` preceded their respective Engines.

---

## What Is a Plan?

A Plan is not the response itself, and it is not the recommendation itself. It is the structured statement of **how** this turn's committed Decision Type will be carried out — its scope, its shape, and its delivery strategy — produced before the substantive content that will eventually fill that shape necessarily exists.

This is a deliberate resolution of a timing question: within a single turn, the Planning Engine runs once, before the conditional Recommendation Engine. It therefore cannot plan the delivery of specific recommendation content that has not yet been produced. What it can do, and what this object exists to hold, is decide the *shape* delivery should take — independent of content — so that the Response Composer, once actual content exists (a Recommendation, an Answer, or otherwise), has a strategy to apply rather than one to invent on the spot.

## Why It Exists

Without a distinct, contracted Plan, the boundary between "deciding how to proceed" and "deciding what the actual content is" would blur — either Planning would have to wait for content that isn't ready yet, or downstream Engines would each have to improvise their own delivery strategy inconsistently. Defining this object precisely keeps Planning's responsibility to shape and sequencing, strictly separate from the Recommendation Engine's responsibility to content, and the Response Composer's responsibility to language.

## What It Must Contain

- **Goal** — the immediate objective this turn's plan serves, stated in terms of the committed Decision Type (for example, helping the customer weigh a specific option, resolving a specific ambiguity, or moving them toward a next step).
- **Scope** — where the Decision Type is Recommendation, the specific bounds handed to the Recommendation Engine: which project(s) and option types are eligible for consideration, refining the general Execution Constraints already set by the Decision Engine into this turn's specific terms. Where the Decision Type is not Recommendation, this component states plainly that it does not apply.
- **Communication Plan** — the general shape and pacing strategy for delivering this turn's eventual content: what should be led with, what should be held back for a later message, and roughly how the delivery should be paced across the conversation, consistent with `Conversation.md`'s preference for short, focused messages and `Consultation_Strategy.md`'s chosen approach for this customer. This is a strategy for form, decided independent of the specific content that will eventually fill it.
- **Fallback Behavior** — what this turn should do if the planned path cannot be completed as intended — for instance, if the Recommendation Engine declines to produce a Recommendation. A Plan that only describes the intended path, with no stated fallback, is incomplete.
- **Decision Trace Reference** — the identifier of the Decision Engine's classification this Plan was built to carry out, preserving the same end-to-end traceability already established for decisions and recommendations.

## What It Must Not Contain

- **The actual recommendation content.** Primary Option, Trade-offs, Preconditions, and every other component of a Recommendation belong entirely to the Recommendation Engine, produced after this Plan, not decided within it.
- **Composed, customer-facing language.** A Plan describes a strategy for delivery; it is never itself a draft of what the customer will read.
- **A re-classification of the Decision Type.** The Plan is built to carry out the Decision Engine's classification, not to revisit it.
- **Content-dependent pacing decisions that cannot actually be made without knowing the content.** Where a pacing decision genuinely cannot be made until the content exists, the Communication Plan should state the general principle to apply once that content is available, rather than pretending to decide something it cannot yet know.

## Semantic Contract

A Plan describes **how** this turn's committed action will be carried out — its scope, its shape, and its delivery strategy — never **what** is being recommended, answered, or said, and never the specific words that will eventually express it. A Plan that begins to contain actual content has quietly done the Recommendation Engine's or the Response Composer's job for it; a Plan that begins to contain composed language has quietly done the Response Composer's job twice over.

## Components, Restated as a Structure

1. Goal
2. Scope
3. Communication Plan
4. Fallback Behavior
5. Decision Trace Reference

---

**Awaiting approval of this object definition before `Planning_Engine.md` is written.**
