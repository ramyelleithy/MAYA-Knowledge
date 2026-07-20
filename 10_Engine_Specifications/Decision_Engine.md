# Decision Engine

**Specification Template:** `Engine_Contract.md` (final, 26 fields)
**Runtime Position:** Level 1 Main Consultation Pipeline — immediately after Reasoning Engine, immediately before Planning Engine.

---

### 1. Purpose

The Decision Engine exists to make the one judgment that separates understanding from action: given everything MAYA now knows and believes about this turn, what kind of action is actually the right one to take. Every Engine before it in the pipeline exists to supply the material this judgment requires; every Engine after it exists to carry out whatever this judgment commits to. Without a single Engine responsible for this, the choice among valid actions described in `Decision_Framework.md` would either be made inconsistently by whichever downstream Engine happened to need it first, or never made explicitly at all — leaving the runtime to drift into action without a traceable decision behind it.

### 2. Responsibilities

- Classify the appropriate Decision Type for this turn — Answer, Question, Recommendation, Clarification, Escalation, or Refusal — per `Decision_Framework.md`.
- Detect whether a mandatory escalation trigger, as defined in `Business_Rules.md`, applies to this turn, and commit to Escalation immediately and unconditionally when one is found.
- Weigh the Reasoning Conclusion and the Unified Runtime Context against the fixed priority order established in `Decision_Framework.md` — Truth, Safety, Business Rules, Customer Context, Trust, Understanding, Recommendation, and Progress, in that order.
- Attach an explicit confidence value to its classification, proportional to the strength of the evidence behind it.
- Deliver its classification, and the Decision Rationale behind it, in a form the Planning Engine can act on without needing to re-derive the judgment itself.

### 3. Non-Responsibilities

- Does not perform reasoning about the customer's situation — that belongs to the Reasoning Engine; this Engine classifies based on a conclusion already reached, it does not reach one itself.
- Does not plan the specific steps or shape of the response — that belongs to the Planning Engine.
- Does not produce a recommendation itself — that belongs to the Recommendation Engine, which this Engine's classification may invoke but never performs.
- Does not compose the customer-facing message — that belongs to the Response Composer.
- Does not update the Mental Model, memory, or any persisted customer understanding — that belongs to the Customer Engine and the Memory & State Writer.

### 4. Inputs

- The Reasoning Conclusion produced by the Reasoning Engine, including its confidence.
- The Unified Runtime Context assembled by the Context Builder — carrying, within it, the Business Context, Conversation State, Project Context, and Customer Context (Mental Model).
- The confidence values already attached by upstream interpretive Engines, per the Confidence Contract.

### 5. Outputs

- A single committed Decision Type.
- A **Decision Rationale** — a statement of why this Decision Type was selected, given the Reasoning Conclusion and the priority order. This is distinct from the Reasoning Engine's own reasoning process: the Reasoning Engine explains what is understood about the situation; the Decision Rationale explains only why that understanding led to this particular choice of action. This Engine never reproduces or re-narrates the Reasoning Engine's work — it cites it.
- A confidence value for the classification itself.
- **Execution Constraints** — the specific limits the chosen Decision Type imposes on how it may be carried out, handed to the Planning Engine so it does not have to derive them independently. Every Decision Type carries its own constraints: a Clarification constrains Planning to a single, high-value question rather than a list of them; a Recommendation constrains Planning to a bounded number of options rather than an exhaustive comparison; an Escalation constrains everything downstream to stop autonomous consultation immediately rather than continuing to reason or plan further. These constraints are a direct consequence of the Decision Type itself, not a separate judgment — which is why they belong here rather than being left for the Planning Engine to reconstruct on its own.
- A **Decision Trace ID** — a unique reference for this specific decision, carried forward by every Engine that acts on it (Planning, the conditional Recommendation Engine, Reflection, and the Response Composer). This is an architectural concept, not an implementation detail: every downstream artifact produced in service of this decision should be traceable back to the single decision that caused it, so that any decision in the system can be followed end to end during debugging or audit.
- Where applicable, an explicit Escalation Trigger identifier naming which mandatory condition was met.

### 6. Dependencies

- Requires the Reasoning Engine to have already produced a Reasoning Conclusion for this turn.
- Requires the Unified Runtime Context to be fully assembled, including Business Context specifically — escalation detection cannot responsibly run without business rules already loaded.

### 7. Consumes

- The Reasoning Conclusion, as its primary basis for judgment.
- The Business Context, specifically the mandatory escalation conditions defined in `Business_Rules.md`.
- The Conversation State, which constrains which Decision Types are even reasonable at this point in the customer's journey.
- The Customer Context (Mental Model), whose confidence levels affect how much weight a Recommendation classification can responsibly bear.

### 8. Produces

- The Decision Type that every subsequent Engine in this turn — Planning, the conditional Recommendation Engine, Reflection, and Response Composer — organizes its own work around.
- The Execution Constraints that bound how the Planning Engine may carry out this turn's chosen action, sparing it from having to re-derive limits that follow directly from the Decision Type itself.
- The Decision Trace ID that ties every downstream artifact of this turn back to the single decision that produced it.
- The Escalation Trigger, when present, which the Handoff Composer depends on entirely to determine why and how a handoff is being prepared.

### 9. Internal Decisions

- It alone is authorized to select which Decision Type applies to this turn.
- It alone determines whether a mandatory escalation trigger has been met.
- It may hold a decision at a lower-commitment Decision Type than the Reasoning Conclusion might superficially support, where the confidence behind that conclusion does not justify a stronger commitment.

### 10. Boundaries

Its judgment concerns **which** action is appropriate, never **how** that action should be carried out — it decides that a recommendation is the right response to this turn, never what that recommendation should actually contain. Its authority is scoped to the current turn only; it does not revisit or revise a decision already committed to in a prior turn.

### 11. Failure Modes

- Cannot reach an adequately confident classification given the inputs available.
- The inputs within the Unified Runtime Context conflict in a way the priority order does not cleanly resolve.
- The escalation-detection check itself fails to evaluate, due to a system-level fault rather than a genuine absence of a trigger.
- The Reasoning Conclusion arrives incomplete, missing, or below a usable confidence threshold.

### 12. Recovery Strategy

- Where classification confidence is inadequate, default to the most conservative Decision Type available — ordinarily Clarification.
- Where the escalation-detection check itself cannot be evaluated, fail toward Escalation rather than away from it — the cost of an unnecessary handoff is far lower than the cost of a missed mandatory one.
- Where the Reasoning Conclusion is missing or unusable, do not attempt to classify from incomplete grounds; route to the safe fallback path rather than guessing a Decision Type.

### 13. Confidence Behavior

This Engine's own classification carries an explicit confidence value under the Confidence Contract, proportional to the strength of the Reasoning Conclusion and the upstream confidence values it consumed. It is also the primary consumer of confidence in the entire pipeline: it weighs the confidence attached by Conversation State Recognition, Intent Understanding, Project Engine's shift-detection, Customer Engine, and Reasoning Engine together, rather than treating any single upstream value as decisive on its own.

### 14. Knowledge Usage

Consumes Business Rules knowledge directly — specifically the mandatory escalation conditions defined in `Business_Rules.md` — as delivered through the Business Context. It does not consume Project Knowledge or Knowledge Sources directly; those have already been incorporated into the Reasoning Conclusion it receives, and it does not re-derive them.

### 15. Memory Usage

Does not read or write Memory directly. It consumes the Mental Model only as it already arrives, incorporated into the Unified Runtime Context by the Customer Engine and Context Builder — it holds no independent memory access of its own.

### 16. Business Rules

Bound absolutely. Mandatory escalation triggers from `Business_Rules.md` override every other consideration in this Engine's classification, deterministically, regardless of how compelling a competing Decision Type might otherwise appear from the Reasoning Conclusion alone. This is the one point within an otherwise interpretive Engine where the outcome must remain fully deterministic, matching the Hybrid classification noted in Field 25.

### 17. State Behavior

Stateless with respect to persistence. It reads what earlier Engines have already assembled for this turn and produces a classification; it does not itself read or write anything that persists beyond the current turn.

### 18. Side Effects

None on stored state directly. Its effect is entirely on control flow — determining whether this turn proceeds to the Planning Engine along the normal path, or diverts immediately to the Handoff Composer.

### 19. Observability

Every classification made, the Decision Rationale and confidence attached to it, and — with particular importance — every escalation trigger detected, whether or not it was ultimately acted on, must be logged and reviewable. The Decision Trace ID attached to each decision is what makes this practical: it lets every artifact produced downstream in service of a decision — a plan, a recommendation, a reflection outcome, a composed message — be followed back to the single decision responsible for it, without needing to reconstruct that link after the fact. This Engine carries the primary structural responsibility for guaranteeing that mandatory business rules were genuinely honored on any given turn, and its decisions therefore warrant the highest visibility of any Engine in the runtime.

### 20. Performance Expectations

Expected to be fast relative to the Reasoning Engine, since it performs a bounded, categorical classification over an already-produced conclusion rather than open-ended reasoning of its own. The deterministic escalation check embedded within it should resolve essentially instantly.

### 21. Security Considerations

Because an escalation decision can carry real legal and financial weight, per the Human Authority boundaries in `Business_Rules.md`, the integrity of this Engine's escalation detection is a matter of genuine consequence. It must not be capable of being bypassed by upstream manipulation of context, and its classification — particularly any escalation determination — should be tamper-evident once logged.

### 22. Extensibility

New Decision Types may only be introduced if `Decision_Framework.md` is first revised to authorize them; this Engine does not define or extend its own classification set independently. Likewise, its escalation-trigger list must only grow through changes to `Business_Rules.md`, never through an independent addition made within this Engine alone.

### 23. Out of Scope

The construction of a plan, a recommendation, or a composed message; any assessment of a project's suitability for a customer; any direct interaction with the customer whatsoever.

### 24. Invocation Condition

Unconditional. This Engine runs on every turn, once the Reasoning Engine has produced a conclusion — there is no path through the normal pipeline in which a decision does not need to be made.

### 25. Determinism Classification

Hybrid. The overall classification of Decision Type is interpretive, weighing priorities and evidence through model-based judgment. The mandatory escalation-trigger check embedded within that judgment must remain fully deterministic, exactly as established in the Level 1 Runtime Architecture and reaffirmed in Field 16 above.

### 26. Engine Invariants

- Must never classify a Decision Type with confidence higher than what the Reasoning Conclusion and the upstream confidence values actually support.
- Must never bypass, soften, or delay a mandatory escalation trigger, regardless of how strong a competing Decision Type appears.
- Must never classify Recommendation as the Decision Type unless the Reasoning Engine has already completed its conclusion for this turn.
- Must never allow a Business Rule to be overridden by a persuasive-sounding judgment produced elsewhere in the pipeline — Business Rules outrank generated reasoning without exception.
- Must never revise or contradict a decision already committed to in a prior turn without new evidence arising in the current one.
- Must never restate or reproduce the Reasoning Engine's own reasoning as its Decision Rationale — the Rationale explains the choice of action, not the understanding the Reasoning Engine already explained.
- Must never emit Execution Constraints inconsistent with the Decision Type they accompany — each Decision Type's constraints follow directly from what that type means, not from a separate judgment made case by case.

---

**Status: Approved**, incorporating the Decision Rationale/Reasoning distinction, Execution Constraints, and Decision Trace ID.

Next: `Reasoning_Engine.md` — since the Decision Engine is now fully specified, the next document defines precisely how the Reasoning Conclusion it depends on is actually reached.
