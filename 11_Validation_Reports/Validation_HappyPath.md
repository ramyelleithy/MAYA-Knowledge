# System Validation — Journey 1: Happy Path

**Objective:** Not to validate a customer experience, but to validate architectural contracts, using a representative journey as the vehicle. Every Engine is checked against the four categories agreed: Contract Compliance, Responsibility Boundary, Artifact Integrity, Pipeline Integrity.

---

## Scenario

A new contact messages MAYA on WhatsApp, having clicked a Meta ad referencing a specific project. The message asks about pricing for a two-bedroom unit and states a readiness to move soon. Understanding is sufficient. No escalation trigger applies. No Business Rule is violated. The Decision Engine classifies this turn as Recommendation. A single Primary Option and one Alternative are produced, each with honest Trade-offs and a Precondition on availability. No exceptional branch is activated anywhere in the turn.

This journey is chosen first not for its simplicity, but because it is the one journey that exercises nearly every Engine in the Main Consultation Pipeline while introducing the fewest exceptional branches — if it cannot satisfy every contract, no edge case is worth validating yet.

---

## Engine-by-Engine Trace

### Communication Gateway
Receives the raw WhatsApp webhook payload, including referral data. Produces the raw envelope untouched.
- **Contract Compliance:** Holds — no interpretation attempted, referral data preserved.
- **Responsibility Boundary:** Holds — no encroachment into Normalization's territory.
- **Artifact Integrity:** N/A at this stage — no structured artifact yet exists.
- **Pipeline Integrity:** Holds — hands off to Normalization, the only permitted next step.
- **Finding:** See "Open Finding 1" below — this is the point at which a Turn-level identifier should exist and currently does not.

### Normalization
Produces the canonical Message object: text content, message type (text), sender identifier, referral data intact.
- **Contract Compliance:** Holds.
- **Responsibility Boundary:** Holds — no intent or project interpretation attempted.
- **Artifact Integrity:** Holds — canonical Message object complete.
- **Pipeline Integrity:** Holds — proceeds to Input Business Rules Gate.

### Input Business Rules Gate
Message does not match any spam, abuse, or restricted-topic pattern. Returns Pass.
- **Contract Compliance:** Holds.
- **Responsibility Boundary:** Holds — no interpretation of intent attempted, purely pattern-based.
- **Artifact Integrity:** N/A — binary signal only.
- **Pipeline Integrity:** Holds — proceeds to Contact Resolution rather than the Safe Fallback Composer, correctly, since no rule matched.

### Contact Resolution
No existing record matches this sender identifier. Returns Contact Status = New.
- **Contract Compliance:** Holds.
- **Responsibility Boundary:** Holds — no role classification attempted, exactly as its contract requires.
- **Artifact Integrity:** N/A — a status flag, not a structured object.
- **Pipeline Integrity:** Holds — Memory Engine receives the New status correctly.

### Memory Engine
Given Contact Status = New, retrieval is skipped entirely; an empty Customer Memory object is produced.
- **Contract Compliance:** Holds — this is the explicitly documented behavior for a New Contact.
- **Responsibility Boundary:** Holds.
- **Artifact Integrity:** Holds — an empty object is a valid, complete object for this case, not a missing one.
- **Pipeline Integrity:** Holds.

### Conversation Context Engine
No prior conversation history exists for this thread. Produces an empty Conversation Context containing only this turn's message.
- **Contract Compliance:** Holds.
- **Responsibility Boundary:** Holds.
- **Artifact Integrity:** Holds.
- **Pipeline Integrity:** Holds.

### Business Context Engine
Loads the current, applicable Business Rules set successfully.
- **Contract Compliance:** Holds.
- **Responsibility Boundary:** Holds — no enforcement attempted, only retrieval.
- **Artifact Integrity:** Holds — Business Context complete.
- **Pipeline Integrity:** Holds — its presence is what later allows Context Builder to proceed without blocking.

### Conversation State Recognition
With no prior history and a New Contact, defaults to the earliest journey position (Entry/Understanding), with high confidence given the unambiguous absence of history.
- **Contract Compliance:** Holds.
- **Responsibility Boundary:** Holds — no intent interpretation attempted.
- **Artifact Integrity:** Holds — confidence value present, per the Confidence Contract.
- **Pipeline Integrity:** Holds.

### Intent Understanding
Classifies the message as Intent (a forming purpose — pricing plus a stated readiness to move soon — distinct from a bare Question), with high confidence given the explicit statement of urgency.
- **Contract Compliance:** Holds.
- **Responsibility Boundary:** Holds — does not attempt to update the Mental Model itself.
- **Artifact Integrity:** Holds — confidence value present.
- **Pipeline Integrity:** Holds.

### Project Engine
First contact with a valid referral code; deterministic mapping identifies the project. No shift-detection judgment is needed since no prior project was established. Project Brain loads successfully.
- **Contract Compliance:** Holds.
- **Responsibility Boundary:** Holds — no suitability judgment attempted.
- **Artifact Integrity:** Holds — Project Context complete, no confidence value attached, correctly, since this was the deterministic first-contact path.
- **Pipeline Integrity:** Holds — the Brain Loader ran because a project was confirmed; detection ran because none was previously established, exactly matching its conditional invocation rule.

### Customer Engine
Forms initial hypotheses: purpose likely investment-or-living undetermined yet, budget unstated, urgency high (from Intent), unit-type interest confirmed (two-bedroom). Role hypothesis: likely genuine customer, moderate confidence — explicitly not finalized.
- **Contract Compliance:** Holds.
- **Responsibility Boundary:** Holds — role held as hypothesis, not a label; no recommendation attempted.
- **Artifact Integrity:** Holds — each hypothesis carries its own confidence.
- **Pipeline Integrity:** Holds.

### Context Builder
Assembles all six context objects, including the empty Memory and Conversation contexts, which are valid, complete objects rather than missing ones.
- **Contract Compliance:** Holds.
- **Responsibility Boundary:** Holds — no interpretation introduced.
- **Artifact Integrity:** Holds — Business Context's presence confirmed before proceeding, per its blocking policy.
- **Pipeline Integrity:** Holds.

### Reasoning Engine
Synthesizes a Situation Assessment: a new, apparently genuine customer with a specific, clearly stated need (two-bedroom, price, urgency), against a Project Brain confirming a matching unit exists. Supporting Evidence cites the Project Brain's pricing and availability data plus the customer's own stated urgency. One Unknown is surfaced (budget not yet stated). Confidence: moderately high, reflecting good project-side evidence but incomplete customer-side information.
- **Contract Compliance:** Holds — all eight components of the Reasoning Conclusion are present, including a Reasoning Trace.
- **Responsibility Boundary:** Holds — Decision-Relevant Signals note that urgency and a clear match exist, without recommending anything, per the Semantic Contract.
- **Artifact Integrity:** Holds structurally. **Open Finding 2** below concerns how this Engine's overall Confidence value should be derived from the several distinct upstream confidence values (Conversation State, Intent, Customer Engine's hypotheses) it synthesized — this is not currently specified anywhere with a stated rule.
- **Pipeline Integrity:** Holds.

### Decision Engine
No escalation trigger applies. Classifies this turn as Recommendation, with a Decision Rationale citing the clear match and stated urgency from the Reasoning Conclusion. Execution Constraints state a Recommendation should not exceed one Alternative. A Decision Trace ID is generated here.
- **Contract Compliance:** Holds.
- **Responsibility Boundary:** Holds — the Decision Rationale cites the Reasoning Conclusion rather than re-narrating it.
- **Artifact Integrity:** Holds — confidence attached, Escalation Trigger correctly absent.
- **Pipeline Integrity:** Holds — proceeds to Planning rather than the Handoff Composer, correctly.
- **Note on Finding 1:** This is the first point in the turn at which a trace identifier is created. Everything produced before this Engine — the Reasoning Conclusion and its Trace — has no identifier of its own to be retroactively linked by. See Open Finding 1.

### Planning Engine
Goal: help the customer evaluate a specific matching unit. Scope: the Entry Project only, per the Recommendation Rule, since nothing has shown it unsuitable. Communication Plan: lead with the Primary Option and its price, hold the Alternative and Trade-offs for a follow-up message if the customer engages further. Fallback Behavior: if the Recommendation Engine declines, fall back to a clarifying question about budget.
- **Contract Compliance:** Holds — all five Plan components present.
- **Responsibility Boundary:** Holds — no recommendation content invented.
- **Artifact Integrity:** Holds — Decision Trace Reference correctly carried forward from the Decision Engine.
- **Pipeline Integrity:** Holds — proceeds to the Recommendation Engine, correctly, since the Decision Type is Recommendation.

### Recommendation Engine
Produces a Recommendation: Primary Option (the matching two-bedroom unit), Grounding (Project Brain pricing/availability, customer's stated need), Supporting Rationale (fits stated unit type and urgency), Trade-offs (a modest premium versus a smaller unit type), Preconditions (unit remains available, price remains as quoted), one Alternative (a comparable unit at a different price point), Confidence bounded by the Reasoning Conclusion's moderate-high confidence.
- **Contract Compliance:** Holds — all nine Recommendation components present.
- **Responsibility Boundary:** Holds — no delivery sequencing decided here; that remains within the Plan already produced.
- **Artifact Integrity:** Holds — Decision Trace Reference present; Confidence does not exceed its upstream ceiling; exactly one Alternative, consistent with the Execution Constraint Planning passed down.
- **Pipeline Integrity:** Holds.

### Reflection Engine
Reviews the full chain: Reasoning Conclusion, Decision, Plan, and Recommendation are internally consistent; the one stated Unknown (budget) does not undermine the Recommendation's Confidence, since the Recommendation's own Preconditions and Trade-offs already account for the residual uncertainty. Returns Proceed.
- **Contract Compliance:** Holds.
- **Responsibility Boundary:** Holds — flags nothing to revise, alters nothing itself.
- **Artifact Integrity:** Holds — Reflection Outcome correctly formed as Proceed with no target Engine needed.
- **Pipeline Integrity:** Holds — proceeds to the Output Business Rules Gate.

### Output Business Rules Gate
The Recommendation contains no unauthorized discount, no committed inventory beyond what the Project Brain confirms, and no internal information. Returns Pass.
- **Contract Compliance:** Holds.
- **Responsibility Boundary:** Holds — no quality judgment attempted, purely rule matching.
- **Artifact Integrity:** N/A — binary signal only.
- **Pipeline Integrity:** Holds — proceeds to the Response Composer.

### Response Composer
Composes a short first message presenting the Primary Option and its price, per the Communication Plan's instruction to hold the Alternative and Trade-offs for later, phrased with the confidence level the Recommendation actually carries — not overstated.
- **Contract Compliance:** Holds.
- **Responsibility Boundary:** Holds — introduces no new facts.
- **Artifact Integrity:** N/A — final text, not a structured object.
- **Pipeline Integrity:** Holds.

### Memory & State Writer
Persists the updated Mental Model (including the still-tentative role hypothesis and the newly surfaced budget Unknown), the Conversation State (now advanced past Entry), and a record of the turn referencing its Decision Trace ID.
- **Contract Compliance:** Holds.
- **Responsibility Boundary:** Holds — persists what it was given, exercises no independent judgment about what to keep.
- **Artifact Integrity:** Holds.
- **Pipeline Integrity:** Holds — the last Engine before delivery.

### WhatsApp Reply
Delivers the composed message to the verified channel identifier successfully.
- **Contract Compliance:** Holds.
- **Responsibility Boundary:** Holds.
- **Artifact Integrity:** N/A.
- **Pipeline Integrity:** Holds — turn concludes cleanly, no unexpected Engine skipped, no branch incorrectly activated.

---

## Open Findings

Two gaps surfaced during this trace. Neither invalidates the architecture; both are the kind of thing this validation exercise exists to catch before Level 2 execution work begins.

### Open Finding 1 — No Turn-level identifier exists prior to the Decision Engine

The Decision Trace ID is first created at the Decision Engine. Everything produced earlier in the turn — the Reasoning Conclusion and its Reasoning Trace, and every context object — has no identifier of its own until the Decision Engine's ID retroactively applies to what comes after it. For full observability, as `Reasoning_Engine.md` and every context-producing Engine's Observability field already demand, these earlier artifacts should be traceable back to this specific turn from the moment they are created, not only from the moment a decision is reached.

**Proposed resolution:** Introduce a **Turn ID**, established once at the Communication Gateway or Normalization, carried by every artifact produced for this turn from that point forward. The Decision Trace ID remains the specific reference to the decision made within a turn; the Turn ID is the broader reference that ties the entire turn — including everything produced before a decision exists — together. This is a small, additive change to the cross-cutting concepts already established, not a revision to any Engine's core contract.

### Open Finding 2 — No stated rule for how the Reasoning Engine's Confidence aggregates multiple upstream confidence values

`Reasoning_Engine.md` requires that its Confidence never exceed what its Supporting Evidence justifies, and the Decision Engine, Planning Engine, and Recommendation Engine all correctly treat a single upstream confidence value as a ceiling. But the Reasoning Engine itself synthesizes several distinct upstream confidence values at once — from Conversation State Recognition, Intent Understanding, and the Customer Engine's individual hypotheses — into one overall Situation Assessment confidence, and no document currently states the rule for how that aggregation should behave.

**Proposed resolution:** State explicitly, likely as an addition to `Reasoning_Conclusion.md`'s Confidence component or to `Reasoning_Engine.md`'s Confidence Behavior field, that the Situation Assessment's Confidence can never exceed the lowest confidence among the upstream signals it materially relies on — a synthesis is only as strong as its weakest genuinely load-bearing input, not an average that lets a strong signal paper over a weak one.

---

## Resolved Findings

Both findings identified during this trace have since been resolved at the architecture level, before continuing to Journey 2.

### Resolved — Turn ID

A **Turn Identity Contract** has been added to `Runtime_Architecture_Level1_v3_FINAL.md` as a cross-cutting concept, alongside the existing Confidence Contract. The Turn ID is established once, at the Communication Gateway, and every artifact produced within a turn — every context object, the Reasoning Conclusion and its Reasoning Trace, the Decision, the Plan, the Recommendation, and the composed reply — carries it, along with the producing Engine's name, a timestamp or sequence marker, and its artifact type. `Communication_Gateway.md` has been updated to reflect this as an explicit responsibility, and `Engine_Contract.md`'s Observability field now states this baseline applies to every Engine without needing to be restated in each individual specification. The Decision Trace ID remains what it already was: a reference to the specific decision made within a turn, nested inside the broader scope the Turn ID provides.

### Resolved — Confidence Aggregation

`Reasoning_Conclusion.md`'s Confidence component and `Reasoning_Engine.md`'s Confidence Behavior field have both been updated with a dependency-aware rule: the Situation Assessment's Confidence must never exceed the lowest confidence among the upstream inputs that are genuinely load-bearing for the specific conclusion reached. An upstream input the conclusion does not materially depend on does not lower it merely by being uncertain itself. This keeps the Confidence Contract conservative without making every assessment hostage to unrelated uncertainty.

---

## Journey 1 Result

Every Engine satisfied all four validation categories. Two cross-cutting gaps were identified and have since been resolved at the architecture level, both additively, without altering any Engine's core responsibilities. **Journey 1 — Happy Path — is validated.**
