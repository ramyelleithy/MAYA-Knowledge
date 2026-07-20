# Reasoning Conclusion

## Status

This document defines the Reasoning Conclusion as an architectural object — the thing the Reasoning Engine produces and the Decision Engine consumes as its primary input. It does not describe how a Reasoning Conclusion is produced, how any model reasons, or any implementation detail. `Reasoning_Engine.md` is deliberately not written until this object is defined and approved, because an Engine cannot be responsibly specified before the shape of its output is settled.

---

## What Is a Reasoning Conclusion?

A Reasoning Conclusion is the single, structured statement of understanding that results from connecting everything relevant known about a turn — context, knowledge, and the customer's Mental Model — into a coherent picture of the situation. It is the boundary object between everything the earlier Engines have gathered and interpreted, and the judgment the Decision Engine still has to make about what to do next.

It is not a narrative, and it is not a transcript of how that understanding was reached. `Reasoning_Model.md` already establishes that reasoning builds understanding rather than merely retrieving it; the Reasoning Conclusion is the finished result of that process, held in a form precise enough for another Engine to act on without needing to re-derive or re-interpret it.

## Why It Exists

Every Engine downstream of Reasoning depends on a stable, well-defined understanding of the situation rather than raw context. Without a Reasoning Conclusion as a distinct, contracted object, the Decision Engine would either have to re-interpret the Unified Runtime Context itself — duplicating the Reasoning Engine's work inconsistently — or act on an understanding that was never made explicit enough to inspect, question, or hold accountable. Defining this object precisely is what allows the Reasoning Engine and the Decision Engine to remain two genuinely separate responsibilities rather than one blurred one.

## What It Must Contain

- **Situation Assessment** — the synthesized understanding of the customer's current situation, in light of their goal, the project under discussion, and everything else understood about them. This is the "so what," not a restatement of raw facts — what the gathered evidence actually means, taken together.
- **Supporting Evidence** — the specific confirmed facts and context elements the Situation Assessment is grounded in, referenced rather than reproduced in full, so the assessment can be traced back to what actually supports it.
- **Active Hypotheses** — the hypotheses from the customer's Mental Model that genuinely bear on this conclusion, carried forward with whatever confidence they currently hold, per `Mental_Model.md`.
- **Conflicts** — any contradiction in the evidence, or between hypotheses, that the reasoning process encountered and could not fully resolve. A Reasoning Conclusion that quietly smooths over a real conflict is not a faithful one.
- **Unknowns** — what remains genuinely unknown and relevant to the situation, stated plainly rather than omitted for the sake of appearing complete.
- **Assumptions** — anything treated as true for the purpose of reaching this conclusion that is not itself confirmed, explicitly flagged as an assumption rather than blended into confirmed fact.
- **Confidence** — the degree of certainty attached to the Situation Assessment as a whole, expressed per the degrees of certainty established in `Truth_Model.md` — never a single flattened tone applied regardless of how well-grounded the assessment actually is. Where the Situation Assessment synthesizes more than one upstream confidence value, this Confidence must never exceed the lowest confidence among the upstream inputs that are genuinely load-bearing for the specific conclusion reached — an upstream input the conclusion does not actually depend on does not lower it merely by being uncertain itself. This is a dependency-aware ceiling, not a blanket minimum across every available signal.
- **Decision-Relevant Signals** — the specific aspects of the situation most pertinent to whatever the Decision Engine will need to weigh next, surfaced explicitly rather than left for the Decision Engine to dig out of the Situation Assessment on its own.

## What It Must Not Contain

- **A chosen Decision Type or committed course of action.** This is the most important exclusion. The Reasoning Conclusion describes the situation; it must never decide what to do about it — that boundary is what keeps the Reasoning Engine and the Decision Engine genuinely separate. This is why the object contains Decision-Relevant Signals rather than anything resembling a "Recommended Direction": surfacing what matters for the decision is reasoning's job; choosing the action based on it belongs entirely to the Decision Engine.
- **Invented facts.** Every element of Supporting Evidence must trace back to something actually confirmed in Knowledge, Memory, or Context — never filled in because it would make the assessment more complete.
- **Composed, customer-facing language.** This object is an internal architectural artifact, not a draft of anything a customer will read.
- **Silently omitted Conflicts or Unknowns.** Leaving out a genuine conflict or unknown to present a cleaner-looking conclusion is a failure of this object's purpose, not an improvement to it.
- **A judgment about business-rule permissibility.** Whether a given situation is permitted, restricted, or requires escalation is evaluated by the Decision Engine against Business Context — the Reasoning Conclusion describes the situation as it is, not what is allowed to be done about it.

## Components, Restated as a Structure

1. Situation Assessment
2. Supporting Evidence
3. Active Hypotheses
4. Conflicts
5. Unknowns
6. Assumptions
7. Confidence
8. Decision-Relevant Signals

This differs from the illustrative structure originally proposed in one deliberate respect: "Recommended Direction" has been replaced with "Decision-Relevant Signals," for the reason stated above. A Reasoning Conclusion that recommended a direction would quietly do the Decision Engine's job for it, undermining the separation of responsibilities this entire Level 2 effort has been built to preserve — including the explicit invariant already written into `Decision_Engine.md` that it must never classify an action without a completed Reasoning Conclusion to classify from. That invariant only means something if the Reasoning Conclusion itself stops short of the classification.

## Semantic Contract

Every element inside a Reasoning Conclusion must be descriptive, never prescriptive. This object describes reality as the system currently understands it; it does not describe what should be done about that reality. No component of a Reasoning Conclusion — Situation Assessment, Supporting Evidence, Active Hypotheses, Conflicts, Unknowns, Assumptions, Confidence, or Decision-Relevant Signals — may contain a Recommendation, a Decision, an Action, a Plan, a Command, or a Priority ranking. Each of these belongs entirely to another Engine downstream, and their appearance inside a Reasoning Conclusion would mean the Reasoning Engine has quietly stepped into a responsibility that is not its own.

This is what gives the exclusion of "Recommended Direction" its full force: the Semantic Contract is not a rule about one field, it is a rule about the nature of every field. A Decision-Relevant Signal may note, descriptively, that a particular conflict or piece of evidence is likely to matter a great deal to whatever comes next — but it may never say what should be done in light of that, and it may never rank one course of action above another. The moment any part of this object tells the runtime what to do rather than what is true, it has left the boundary this object exists to hold.

---

## Status

**Final.** This definition is now a formal part of MAYA's architecture. `Reasoning_Engine.md` is written against it.
