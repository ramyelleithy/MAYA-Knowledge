# System Validation — Journey 2: Escalation

**Validation categories:** Contract Compliance, Responsibility Boundary, Artifact Integrity, Pipeline Integrity, and — newly added following Journey 1 — **Traceability**: is the Turn ID present, is the producing Engine identifiable, is the Artifact Type correct, is the Timestamp/Sequence preserved, is the Decision Trace Reference created or propagated correctly, and can every artifact be traced back to this specific turn.

---

## Scenario

An existing customer, mid-conversation about a specific unit, sends: "I want to reserve this unit and pay the deposit today." This is an unambiguous reservation intent — a mandatory escalation trigger per `Business_Rules.md`. This journey deliberately differs from Journey 1 in two ways at once: the contact is Existing rather than New, and the outcome is Escalation rather than Recommendation — exercising both a different early-pipeline path and the primary exceptional branch.

---

## Engine-by-Engine Trace

Steps identical in behavior to Journey 1 (Communication Gateway, Normalization, Input Business Rules Gate, Conversation Context Engine, Business Context Engine, Context Builder) are confirmed compliant across all five categories without repeating the full detail already established. Differences and the escalation branch itself are detailed below.

### Communication Gateway
Establishes a new Turn ID for this turn, distinct from any prior turn with this same customer.
- **Traceability:** Holds — the Turn ID is fresh per turn, not reused across a returning customer's separate conversations.
- All other categories: Holds, as in Journey 1.

### Contact Resolution
This time, the sender identifier matches an existing record. Returns Contact Status = Existing, with the record reference.
- **Contract Compliance:** Holds.
- **Responsibility Boundary:** Holds — still no role classification attempted, even though this is a known contact.
- **Artifact Integrity:** Holds.
- **Traceability:** Holds — the Turn ID is new for this turn even though the customer record itself is not.
- **Pipeline Integrity:** Holds — Memory Engine correctly attempts retrieval given the Existing status, the opposite path from Journey 1.

### Memory Engine
Retrieves the existing Customer Memory: prior stated preferences, the project already under discussion, and prior conversation markers.
- All categories hold, including Traceability — the retrieved memory belongs to this Turn ID's processing even though the memory itself predates this turn.

### Conversation State Recognition
Given prior history showing advanced engagement (unit-level discussion already occurred), recognizes the position as late-stage — Decision Support or Ready — with high confidence.
- All categories hold. This is the first journey to actually exercise recognition of an advanced position rather than defaulting to Entry.

### Intent Understanding
Classifies the message as Commitment — the strongest of the five intent states in `Psychology.md` — with very high confidence given the explicit, unambiguous language ("reserve," "pay the deposit today").
- All categories hold.

### Project Engine
The project is already established from prior turns; no referral remapping occurs. Detection is skipped per its conditional invocation rule; the Brain Loader still runs to ensure current knowledge is available.
- **Pipeline Integrity:** Holds — this is the first journey to confirm the "no re-detection needed" path actually behaves as specified, distinct from Journey 1's first-contact mapping path.
- All other categories hold.

### Customer Engine
Updates the Mental Model: the reservation intent is now a confirmed hypothesis at high confidence, superseding the earlier, more tentative purpose hypotheses.
- All categories hold, including Traceability.

### Reasoning Engine
Produces a Situation Assessment reflecting a customer at the point of commitment, with Decision-Relevant Signals noting the explicit reservation language — descriptively, not prescriptively. Confidence is high, and correctly bounded by the load-bearing upstream inputs (Intent's very high confidence, Conversation State's high confidence) rather than diluted by any unrelated signal.
- **Artifact Integrity:** Holds — the dependency-aware Confidence rule resolved after Journey 1 is exercised correctly here for the first time under a genuinely high-confidence scenario.
- All other categories hold.

### Decision Engine
Detects the mandatory escalation trigger — reservation intent — and classifies this turn as **Escalation**, deterministically and unconditionally, regardless of how strong the case for a Recommendation might otherwise have appeared from the Reasoning Conclusion alone. Generates the Decision Trace ID. Execution Constraints for this turn state: stop all autonomous consultation immediately.
- **Contract Compliance:** Holds.
- **Responsibility Boundary:** Holds — the escalation check remained fully deterministic within an otherwise interpretive Engine, exactly as its Hybrid classification requires.
- **Artifact Integrity:** Holds — Escalation Trigger is populated and named specifically ("reservation intent"), not left generic.
- **Traceability:** Holds — Decision Trace ID created and tagged with the same Turn ID established at the Communication Gateway.
- **Pipeline Integrity:** Holds — the turn exits the normal path entirely and moves to the Handoff Composer, correctly bypassing Planning, the Recommendation Engine, and Reflection.

### Handoff Composer
Composes a customer-facing message that reads as a natural continuation rather than an abrupt handoff, and a context summary for the human consultant capturing the project under discussion, the unit in question, and the customer's stated readiness — so nothing already shared needs to be repeated.
- **Contract Compliance:** Holds.
- **Responsibility Boundary:** Holds — does not re-evaluate the escalation decision itself.
- **Artifact Integrity:** Holds structurally.
- **Traceability:** Holds — both outputs carry the Turn ID and Decision Trace Reference.
- **Pipeline Integrity:** See Finding 1 and Finding 2 below — this is where this journey surfaces genuine gaps that Journey 1 could not have revealed, since Journey 1 never exercised this branch.

---

## Findings

### Finding 1 — The escalation path bypasses the Output Business Rules Gate entirely

`Output_Business_Rules_Gate.md`'s Invocation Condition is stated as running "once the Reflection Engine has returned Proceed." The escalation branch never passes through Reflection, and therefore never passes through the Output Business Rules Gate either — meaning the Handoff Composer's customer-facing message reaches the customer with no deterministic final check for leaked internal information or an implied unauthorized commitment. This is precisely the kind of check `Business_Rules.md` treats as non-negotiable, and it is currently structurally absent from the one branch where a customer is being told something consequential.

**Proposed resolution:** Route the Handoff Composer's output through the Output Business Rules Gate before delivery. Update `Output_Business_Rules_Gate.md`'s Invocation Condition to trigger on either the Reflection Engine's Proceed outcome or the Handoff Composer's output, and update the pipeline diagram accordingly: Decision Engine → (escalation) → Handoff Composer → Output Business Rules Gate → next step. Reflection itself remains correctly skipped for Escalation, since the escalation trigger is a deterministic, mandatory rule match rather than a judgment call — but the deterministic safety gate is a different concern entirely and should not have been skipped along with it.

### Finding 2 — The escalation path bypasses the Memory & State Writer entirely

`Memory_State_Writer.md`'s Invocation Condition is stated as running "once the Response Composer has produced a message." In the escalation branch, the Response Composer never runs — the Handoff Composer produces the customer-facing message directly. As currently specified, this means the updated Mental Model and Conversation State reached during this turn — including the newly confirmed reservation-intent hypothesis — are never persisted following an escalation. The next turn, whether handled by MAYA after the human consultant's involvement or by the consultant reviewing prior context, would be working from stale understanding.

**Proposed resolution:** Route the Handoff Composer's output through the Memory & State Writer as well, immediately after the Output Business Rules Gate confirms it. Update `Memory_State_Writer.md`'s Invocation Condition to trigger on either the Response Composer's or the Handoff Composer's output, and update the pipeline diagram accordingly: Handoff Composer → Output Business Rules Gate → Memory & State Writer → Reply.

---

## Resolved Findings

Both findings have been resolved at the architecture level. The pipeline diagram in `Runtime_Architecture_Level1_v3_FINAL.md` now routes the escalation branch as: Decision Engine → (escalation) → Handoff Composer → Output Business Rules Gate → Memory & State Writer → Reply. `Output_Business_Rules_Gate.md` and `Memory_State_Writer.md` have both had their Inputs, Dependencies, and Invocation Condition fields updated to trigger on either the normal path or the escalation path, and each has gained an explicit Invariant guaranteeing it is never bypassed regardless of which path a turn takes. `Handoff_Composer.md`'s Produces field has been updated to reflect the corrected routing.

## Journey 2 Result

Every Engine satisfied all five validation categories. Two structural gaps were found at the escalation branch's tail — both invisible to Journey 1, since Journey 1 never exercised this branch — and both have since been resolved. **Journey 2 — Escalation — is validated.**
