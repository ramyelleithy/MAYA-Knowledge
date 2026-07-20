# Engine Contract

## Status

**Final.** This document is the official, approved template for documenting any Engine within the MAYA Runtime. It does not describe any specific Engine — not Communication, not Memory, not Reasoning, not any other. It defines the contract every Engine must be documented against at Level 2. From this point forward, this document is revised only if a new architectural principle is discovered that applies to every Engine — never to accommodate a single Engine's special case.

## Purpose of This Document

An Engine, within MAYA's runtime, is any discrete unit of responsibility identified in the Level 1 Runtime Architecture — Communication Gateway, Memory Engine, Reasoning Engine, Decision Engine, and every other stage named there. Each of these will eventually need its own dedicated specification. Rather than let each specification take its own shape, this contract fixes one shape all of them must follow, so that any Engine — including ones not yet imagined — can be fully documented by filling in the same fields, in the same order, with the same rigor.

A contract of this kind exists because a runtime built from many independently designed parts only stays coherent if every part answers the same questions about itself. Without a shared contract, some Engines would be documented thoroughly and others carelessly, some would define their boundaries and others would not, and the runtime as a whole would slowly become harder to reason about as it grew — exactly the failure this repository's Foundation has been built from the start to prevent.

---

## The Contract

Every Engine specification in this runtime must address the following, in this order. An Engine that cannot be fully described using only these fields has not yet been understood well enough to design.

### 1. Purpose

Why this Engine exists at all — the specific gap in the runtime it fills, and what would be missing from MAYA's behavior if it did not exist. Purpose is a statement of necessity, not of function; it answers "why must something like this exist," which is a different question from "what does it do."

### 2. Responsibilities

The specific things this Engine is accountable for doing. Responsibilities should be stated as outcomes the Engine is answerable for, not as a description of internal steps — what must be true after this Engine has run, not how it gets there.

### 3. Non-Responsibilities

The things this Engine must never take on, even where it would be technically capable of doing so. This section exists because capability is not permission — an Engine drifting into a neighboring Engine's responsibility, even helpfully, degrades the clarity the entire contract exists to protect. Every non-responsibility listed here should name the Engine that actually owns it instead.

### 4. Inputs

What this Engine receives in order to do its work — named precisely enough that another Engine's specification could reference these same inputs without ambiguity about what they contain.

### 5. Outputs

What this Engine produces. Every output should be traceable to a responsibility listed in Section 2 — an Engine should not produce something that no responsibility of its accounts for.

### 6. Dependencies

What this Engine requires to exist or to have already run before it can do its own work — other Engines, prior state, or conditions that must already be satisfied. Dependencies describe order and prerequisite, not data flow on their own.

### 7. Consumes

What this Engine draws on from the outputs of other Engines or from the wider runtime, stated in terms of what it actually uses, not merely what happens to be available to it. An Engine should consume only what its responsibilities genuinely require.

### 8. Produces

What this Engine contributes forward to the rest of the runtime beyond its immediate Outputs — the lasting contribution its work makes available to Engines that have not yet run, including anything written to persisted state. Where Outputs describes what leaves this Engine directly, Produces describes what becomes available to the runtime as a whole as a result.

### 9. Internal Decisions

The judgments this Engine is authorized to make on its own, without needing to defer to another Engine or to a human. This section draws the line between autonomy and escalation — what this Engine may decide, versus what it must hand elsewhere.

### 10. Boundaries

The edges of this Engine's authority and scope, stated plainly enough that a genuine edge case can be resolved by reading this section alone. Boundaries differ from Non-Responsibilities: non-responsibilities name things this Engine must not do; boundaries describe the outer limit of what it is even asked to consider in the first place.

### 11. Failure Modes

The specific ways this Engine can fail — not a general acknowledgment that failure is possible, but the actual, anticipated ways its particular responsibilities can go wrong.

### 12. Recovery Strategy

What the runtime does when this Engine fails in one of the ways named above — proceed with degraded input, fall back to a safe default, halt the turn, or escalate. Every failure mode named in Section 11 should have a corresponding recovery behavior here; an unrecovered failure mode is an unfinished contract.

### 13. Confidence Behavior

Whether this Engine's output requires a confidence value under the Confidence Contract established in the Runtime Architecture, and if so, what that confidence is meant to represent for this specific Engine's judgment. Engines whose outputs are deterministic should state plainly that confidence does not apply, rather than omitting the section.

### 14. Knowledge Usage

Whether and how this Engine draws on Knowledge as defined in this repository's Knowledge system — which categories of knowledge it depends on, and whether that dependency is direct or inherited from an input it consumes.

### 15. Memory Usage

Whether and how this Engine reads or writes Memory as defined in `Memory_Model.md` — distinguishing clearly between an Engine that merely receives memory already loaded by another Engine, one that reads memory directly, and one that writes to it.

### 16. Business Rules

How this Engine is bound by `Business_Rules.md` — which rules apply to it directly, whether it is expected to enforce any rule itself, and how it behaves when a business rule constrains what would otherwise be its natural output.

### 17. State Behavior

Whether this Engine is Stateless or Stateful, stated precisely: an Engine is Stateful only if it reads or writes state that persists beyond the current turn; an Engine that merely receives already-assembled context for the current turn and does not itself touch persisted state is Stateless, regardless of how much information passes through it.

### 18. Side Effects

Whether this Engine changes anything outside its own return value — writing to memory, triggering an escalation, sending a message, or altering any state a later turn would observe. An Engine with no side effects should state this explicitly rather than leaving the question unaddressed.

### 19. Observability

What about this Engine's operation should be visible and measurable from the outside — which of its decisions, confidence values, or failures should be loggable and reviewable, so that its behavior over time can actually be understood rather than trusted blindly. Every Engine, without needing to restate it in its own specification, carries the baseline established by the Turn Identity Contract: every artifact it produces is tagged with the Turn ID, the Engine's own name, a timestamp or sequence marker, and its artifact type. An Engine's own Observability field describes what is distinctive about its logging beyond that shared baseline, not a replacement for it.

### 20. Performance Expectations

The general expectations this Engine is held to regarding speed and resource use, stated at the level appropriate to an architecture document — for instance, whether it is expected to be fast and cheap (as a deterministic gate should be) or is permitted to be slower and more resource-intensive because of the depth of judgment it performs. This is not a specific latency target; it is a statement of the kind of cost this Engine is allowed to carry.

### 21. Security Considerations

What this Engine must protect — customer privacy, confidential business information, or the integrity of the runtime itself — and any way its specific responsibilities create exposure that other Engines do not share.

### 22. Extensibility

How this Engine is expected to grow or change over time without breaking the rest of the runtime — what kinds of future changes should be safe to make to it, and what about its current contract must remain stable for other Engines to keep depending on it safely.

### 23. Out of Scope

What is deliberately excluded from this Engine's concern entirely — not merely delegated to another Engine (that belongs in Non-Responsibilities), but genuinely outside what this Engine, or arguably any Engine, is meant to handle at all.

---

## Additional Fields Found Necessary on Review

Reviewing the fields above against the Level 1 Runtime Architecture, and against subsequent review, surfaced three properties that recur across Engines but were not captured by any of the original twenty-three fields. All three are added to the contract.

### 24. Invocation Condition

Whether this Engine runs on every turn unconditionally, or only under specific conditions — and if conditional, exactly what determines whether it runs. The Level 1 Architecture already distinguishes Engines that always execute (such as Reasoning) from ones that are strictly conditional (such as Recommendation, invoked only when the Decision Engine classifies a turn as a recommendation). Without a dedicated field for this, an Engine's conditionality would either be buried inside its Inputs or left ambiguous — and ambiguity here has already proven consequential once, in the review that led to clarifying Recommendation Engine's conditional status.

### 25. Determinism Classification

Whether this Engine's behavior is Deterministic (the same input always produces the same output, with no model-based judgment involved), Interpretive (its output depends on model-based judgment and can reasonably vary), or Hybrid (it combines a deterministic component with an interpretive one, as the Decision Engine's escalation check does within an otherwise interpretive judgment). This classification matters because deterministic and interpretive Engines are held to different expectations throughout this repository — deterministic Engines are expected to fail loudly and predictably, while interpretive Engines are expected to carry confidence and be subject to reflection. Leaving this implicit risks an Engine being held to the wrong standard.

### 26. Engine Invariants

The specific things this Engine may never do, under any scenario, regardless of input, context, confidence, or pressure from elsewhere in the runtime. Invariants are distinct from the four fields they most resemble, and the distinction matters:

- **Responsibilities** describe what this Engine is meant to accomplish; Invariants describe what it must never do while accomplishing it.
- **Business Rules** describe constraints imposed on this Engine from outside, by the business; Invariants can include rules the Engine must hold to even where no external business rule addresses the situation at all.
- **Boundaries** describe the outer edge of this Engine's scope — what it is and is not asked to consider; Invariants apply within that scope, governing how it behaves once it is already operating inside its own boundaries.
- **Failure Modes** describe ways this Engine can fail when something goes wrong; Invariants describe lines that must hold even when everything else is going right — they are not failure conditions to recover from, they are conditions that must never occur in the first place.

An Invariant is a piece of this Engine's internal physics — a law its behavior obeys regardless of circumstance, in the same way `Reasoning_Model.md` treats never inventing a fact, or `Business_Rules.md` treats truthfulness, as absolute rather than situational. Every Engine specification should state its invariants explicitly rather than leaving them to be inferred from its other fields, because an unstated invariant is one that can be quietly violated by a future change to the Engine without anyone recognizing the violation as one.

---

## How This Contract Is Used

Every Engine named in the Level 1 Runtime Architecture will, at Level 2, be documented as a complete answer to all twenty-six fields above, in this same order, using this same document as the template. An Engine specification that skips a field is not considered complete. An Engine specification that introduces a new field not found here should instead prompt a revision to this contract itself, so that the same new field becomes available to every other Engine — this document grows only when a genuinely new, universal question is discovered, never when a single Engine merely wants special treatment.

---

**Approved. Level 2 Engine design begins from this template.**
