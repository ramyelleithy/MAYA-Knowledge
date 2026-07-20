# Reasoning Engine

**Specification Template:** `Engine_Contract.md` (final, 26 fields)
**Output Object:** `Reasoning_Conclusion.md` (final, 8 components, bound by its Semantic Contract)
**Runtime Position:** Level 1 Main Consultation Pipeline — immediately after Context Builder, immediately before Decision Engine.

---

### 1. Purpose

The Reasoning Engine exists to connect everything gathered about this turn — the Unified Runtime Context, the customer's Mental Model, and the Project Brain — into a single, coherent Reasoning Conclusion the Decision Engine can act on. Without it, understanding would remain scattered across separate context objects, and every downstream Engine would have to reconstruct its own interpretation of the situation independently, inconsistently, and without a shared, inspectable result to be held accountable to.

**Reasoning is not merely synthesis; it is evidence-based judgment performed over synthesized context.** Combining context into a single picture is a necessary part of this Engine's work, but it is not the whole of it. Synthesis alone would produce a summary; this Engine must also weigh that synthesized picture against the evidence behind it, test it against contradicting possibilities, and hold it to the confidence it actually earns. An implementation that reduces this Engine to intelligent summarization has not built a Reasoning Engine — it has built something with the same name and a materially smaller responsibility.

### 2. Responsibilities

- Synthesize the Unified Runtime Context, the Mental Model, and the Project Brain into a Situation Assessment, per `Reasoning_Conclusion.md`.
- Ground every element of that assessment in Supporting Evidence traceable to confirmed Knowledge, Memory, or Context — never inventing a fact, per `Business_Rules.md` and `Truth_Model.md`.
- Surface the Active Hypotheses relevant to this turn, any Conflicts encountered, the Unknowns that remain, and any Assumptions made along the way, exactly as `Reasoning_Conclusion.md` defines them.
- Where the Unified Runtime Context includes a prior Recommendation retrieved by the Memory Engine, evaluate whether its stated Preconditions still hold against the freshly loaded Project Brain — surfacing any Precondition that no longer holds as a Conflict, exactly as any other contradiction in evidence would be surfaced.
- Attach a Confidence value proportional to the actual strength of the evidence behind the Situation Assessment.
- Identify Decision-Relevant Signals descriptively, never prescriptively, per the Semantic Contract in `Reasoning_Conclusion.md`.
- Actively seek evidence that would contradict its own emerging assessment, not only evidence that confirms it, per `Reasoning_Model.md`'s principle that reasoning does not seek to confirm itself.

### 3. Non-Responsibilities

- Does not decide what action to take. The Reasoning Conclusion this Engine produces must never contain a Recommendation, Decision, Action, Plan, Command, or Priority — that boundary belongs entirely to the Decision Engine, and is enforced by the Semantic Contract in `Reasoning_Conclusion.md`.
- Does not check for mandatory escalation triggers — that belongs to the Decision Engine, even where this Engine's assessment may descriptively note that a situation appears sensitive.
- Does not compose any customer-facing language.
- Does not update the Mental Model directly. It consumes the Mental Model already produced by the Customer Engine; any implications this turn's reasoning has for future understanding are surfaced within the Reasoning Conclusion, not written back to memory by this Engine.
- Does not retrieve knowledge from raw sources itself — it consumes what the Project Engine, Business Context Engine, and the wider Knowledge system have already resolved and loaded.
- Does not retrieve a referenced prior Recommendation itself — that belongs to the Memory Engine; this Engine only evaluates one already retrieved and present in the Unified Runtime Context.
- Does not expose a chain of thought or any internal model reasoning process as part of its output. The Reasoning Trace it produces is a structured record of evidence, surviving hypotheses, and remaining conflicts — never a transcript of how any underlying model actually arrived at them.

### 4. Inputs

The Unified Runtime Context, in full, as assembled by the Context Builder — comprising the Message, Conversation, Memory, Business, Project, and Customer contexts, including, where relevant, a prior Recommendation retrieved by the Memory Engine.

### 5. Outputs

- A single Reasoning Conclusion object, containing exactly the eight components defined in `Reasoning_Conclusion.md`: Situation Assessment, Supporting Evidence, Active Hypotheses, Conflicts, Unknowns, Assumptions, Confidence, and Decision-Relevant Signals. No additional or substitute components are produced.
- A **Reasoning Trace** — an internal architectural record of which evidence mattered, which hypotheses survived, and which conflicts remained on the way to the Situation Assessment. This is not a chain of thought and not an exposure of internal model reasoning; it is a structured account of inputs and outcomes, useful for Reflection and for later explainability, without revealing how any underlying model arrived at its output.

### 6. Dependencies

Requires the Context Builder to have already produced the Unified Runtime Context — which in turn requires every context-producing Engine (Contact Resolution, Memory Engine, Conversation Context Engine, Business Context Engine, Conversation State Recognition, Intent Understanding, Project Engine, Customer Engine) to have already completed its work for this turn.

### 7. Consumes

The Unified Runtime Context in its entirety — every one of its six constituent contexts — as the sole material this Engine reasons over.

### 8. Produces

The Reasoning Conclusion: the sole and authoritative basis the Decision Engine builds its classification upon. Nothing downstream of the Decision Engine reasons about the situation independently; all of it inherits its understanding of the situation from this object. Alongside it, the Reasoning Trace is made available to the Reflection Engine and to any later audit of this turn, without exposing internal model reasoning.

### 9. Internal Decisions

- It alone decides how to weigh and synthesize the six context slices into a single Situation Assessment.
- It alone decides which of the customer's hypotheses are genuinely active and relevant to this specific turn.
- It alone determines what counts as a genuine Conflict or a meaningful Unknown worth surfacing, as distinct from noise not worth including.

### 10. Boundaries

Its judgment concerns understanding the situation, never what should be done about it — the Semantic Contract in `Reasoning_Conclusion.md` is the absolute boundary here, not a stylistic preference. It reasons about the present turn's situation; it does not retroactively revise a Reasoning Conclusion already produced for a prior turn, though it may note, descriptively, where this turn's understanding differs from an earlier one.

### 11. Failure Modes

- The Unified Runtime Context arrives incomplete — for instance, missing Business Context, which should already have been blocked by the Business Context Engine's own failure policy, and therefore indicates an upstream fault rather than a normal condition to reason around.
- Genuinely conflicting evidence that cannot be resolved into any single coherent assessment.
- Insufficient evidence to responsibly reach an assessment above minimal confidence.
- Drift into prescriptive language — a Recommendation, Decision, Action, Plan, Command, or Priority appearing inside what should be a purely descriptive conclusion, in violation of the Semantic Contract.
- Producing a structurally incomplete Reasoning Conclusion — one or more of the eight required components missing entirely, as distinct from a component being present but low in confidence.

### 12. Recovery Strategy

- Where evidence is insufficient, produce a Reasoning Conclusion with correspondingly low Confidence and prominent Unknowns, rather than manufacturing a confident-sounding assessment the evidence does not support.
- Where a Conflict cannot be resolved, state it explicitly as a Conflict rather than arbitrarily favoring one side of it.
- Where Business Context is missing, halt rather than reason without governing rules in view — per `Business_Rules.md`'s Rule Hierarchy, Business Rules sit above Reasoning, and reasoning in their absence is not a safe degradation, it is a violation of that hierarchy.
- Where prescriptive language is detected in a draft conclusion, it must be revised to a descriptive form before being emitted — a Reasoning Conclusion is never allowed to leave this Engine in a form that breaches the Semantic Contract.
- Where any of the eight required components cannot be produced at all, the Engine must fail the turn rather than emit a structurally incomplete Reasoning Conclusion. A low-confidence Situation Assessment with prominent Unknowns is a complete, valid object; a conclusion silently missing its Unknowns or Conflicts component is not, and must never reach the Decision Engine.

### 13. Confidence Behavior

Central to this Engine's output. The Confidence attached to the Situation Assessment must reflect the actual strength of the Supporting Evidence behind it, never the fluency or coherence of the synthesis itself. Where the Situation Assessment depends on more than one upstream confidence value — from Conversation State Recognition, Intent Understanding, or the Customer Engine's hypotheses — the resulting Confidence must never exceed the lowest confidence among the upstream inputs that are genuinely load-bearing for that specific conclusion. An upstream input the conclusion does not materially depend on does not lower it merely by being uncertain itself; this is a dependency-aware ceiling, not an indiscriminate minimum across every available signal. This is the confidence value the Decision Engine leans on most heavily of any single input it receives, per `Decision_Framework.md`'s requirement that expressed certainty never exceed available evidence.

### 14. Knowledge Usage

Consumes Project Knowledge (via the Project Context) and any Knowledge Sources already resolved into the Unified Runtime Context. It does not perform its own retrieval from Knowledge Sources directly — that resolution has already happened upstream, in the Project Engine and the Knowledge system feeding it.

### 15. Memory Usage

Consumes the Mental Model as part of the Customer Context. Does not write to Memory. Any implication this turn's reasoning has for how the customer should be understood going forward is expressed within the Reasoning Conclusion itself, to be picked up by the ordinary Customer Engine and Memory & State Writer path on this or a future turn — never written directly by this Engine.

### 16. Business Rules

Bound by the Rule Hierarchy in `Business_Rules.md`. Reasoning may never contradict an applicable business rule; where the Business Context indicates a constraint, the Situation Assessment reflects that constraint as part of the situation, rather than reasoning around it to reach a more convenient-seeming conclusion.

### 17. State Behavior

Stateless with respect to persistence. It reads the Unified Runtime Context assembled for this turn and produces a Reasoning Conclusion; it does not itself read or write anything that persists beyond the current turn.

### 18. Side Effects

None. This Engine is a pure function of its inputs to a single output object.

### 19. Observability

The complete Reasoning Conclusion — all eight components — and its accompanying Reasoning Trace should be logged for every turn, since together they are the foundation the Decision Engine's classification rests on entirely. Turns where Confidence is low, where a Conflict was left unresolved, or where Unknowns are significant deserve particular visibility, since these are the turns most likely to warrant review. The Reasoning Trace is what makes such review possible without needing to inspect internal model behavior directly.

### 20. Performance Expectations

Expected to be the most computationally demanding Engine in the Main Consultation Pipeline, since it performs the deepest synthesis of any stage. It is expected to take longer and carry greater cost than the deterministic gates or simpler classification stages elsewhere in the pipeline, and this cost is accepted as appropriate to the depth of judgment this Engine is responsible for.

### 21. Security Considerations

Supporting Evidence may reference sensitive customer or business information; this Engine must handle that information consistently with the privacy principles in `Customer_Knowledge.md`, even though its output is an internal object rather than something customer-facing. The Reasoning Conclusion should not become a vector for exposing confidential information if it is ever surfaced beyond its intended, internal audience.

### 22. Extensibility

The eight-component structure of the Reasoning Conclusion is fixed by `Reasoning_Conclusion.md`. This Engine may improve how it synthesizes each component over time, but it may not add, remove, or reshape components independently — any change to the object's structure requires a revision to `Reasoning_Conclusion.md` itself, never a unilateral extension made within this Engine.

### 23. Out of Scope

Classifying a Decision Type; checking for mandatory escalation triggers; producing a recommendation; composing any customer-facing message; writing to memory.

### 24. Invocation Condition

Unconditional. This Engine runs on every turn, once the Unified Runtime Context has been assembled — there is no path through the normal pipeline in which reasoning is skipped.

### 25. Determinism Classification

Interpretive. This Engine's entire function is model-based synthesis and judgment; unlike the Decision Engine, no part of its core responsibility is deterministic.

### 26. Engine Invariants

- Must never include a Recommendation, Decision, Action, Plan, Command, or Priority in its output, per the Semantic Contract in `Reasoning_Conclusion.md`.
- Must never invent a fact to complete an otherwise incomplete Situation Assessment.
- Must never suppress a genuine Conflict or Unknown in order to present a more confident-seeming conclusion.
- Must never assign a Confidence value higher than what its Supporting Evidence actually justifies.
- Must never produce a Situation Assessment that contradicts an applicable Business Rule.
- Must never seek only evidence that confirms its initial hypothesis while disregarding available evidence that contradicts it.
- Must never emit a partial Reasoning Conclusion. All eight components defined in `Reasoning_Conclusion.md` must be present, even where one of them states plainly that little or nothing applies — the Engine fails the turn rather than hand the Decision Engine an object with a component silently missing.
- Must never treat a prior Recommendation's Precondition as still valid without checking it against the freshly loaded Project Brain when that Recommendation is present in context.

---

**Status: Approved**, incorporating the Reasoning-versus-Synthesis principle, the Reasoning Trace output, and the completeness invariant.

Next: before `Recommendation_Engine.md` is written, the Recommendation object itself must be defined — mirroring the sequence already used for Reasoning — so that Planning Engine's eventual specification can reference a settled shape for what a recommendation actually is, rather than an assumed one.
