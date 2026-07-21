# Implementation Audit — Production Workflow vs. Phoenix Runtime v1.0

**Source workflow:** `MAYA - WhatsApp Sales Agent` (`X1QVNNYUFbJftuc2`), active, last updated 2026-07-17.
**Method:** Every node in the production workflow, mapped against the frozen Level 1 Runtime Architecture. No redesign performed. This is a mapping exercise only.

---

## Engine Mapping Matrix

| # | Current Node | Target Runtime Engine | Compliance | Required Action | Reuse / Refactor / Replace |
|---|---|---|---|---|---|
| 1 | Receive WhatsApp Messages | Communication Gateway | Partial | Add Turn ID generation | Refactor |
| 2 | Meta Verification Webhook + Respond to Meta Verification | Communication Gateway (out-of-band) | Compliant | None — correctly outside the Turn flow | Reuse |
| 3 | Is Customer Message? | Communication Gateway | Compliant | None | Reuse |
| 4 | Ignore Non-Message Webhook | Communication Gateway | Compliant | None | Reuse |
| 5 | Inspect Referral | Normalization | Compliant | None | Reuse |
| 6 | Is Voice Message? / Get Media URL / Download Media / Transcribe Audio | Normalization | Compliant | None (transcription confidence not yet propagated) | Reuse |
| 7 | Resolve Customer Message | Normalization | Compliant | None | Reuse |
| 8 | Business Rules Check / Blocked by Business Rules? / Is Spam Category? / Canned Response – Financing / Canned Response – Spam | Input Business Rules Gate + Safe Fallback Composer | Non-compliant (sequencing) | PR-001 done (leak removed); remove remaining false dependency (PR-002), then move to run immediately after Normalization, before Contact Resolution / Memory — see Analysis Update below | Boundary Repair In Progress (1 of 2 prerequisite fixes done) |
| 9 | Chatwoot Sync – Incoming | Contact Resolution | Compliant | None — see Analysis Update below (ADR-001) | Closed — False Positive (Resolved by ADR-001) |
| 10 | Load Conversation Messages | Conversation Context Engine | Compliant | None | Reuse |
| 11 | Format Conversation History | Conversation Context Engine | Compliant | None | Reuse |
| 12 | Load Conversation State | Memory Engine | Partial | Extend schema: no Turn ID, no Recommendation History | Refactor |
| 13 | Determine Greeting & State | Conversation State Recognition | Partial | Confidence value added (interim, high/low) — see Analysis Update below. Position taxonomy still missing: no explicit taxonomy exists in `Conversation_State_Model.md`, and none has been defined — Requires ADR | Refactor (split: Confidence done; Position Taxonomy Requires ADR) |
| 14 | Extract Project Code (Regex) / Has Project Code? / Save Project Code / Get Cached Project Code / Apply Cached Project Code | Project Engine (Detection) | Compliant | None — a clean implementation of the conditional detection rule | Reuse |
| 15 | Get Project Code Mapping (Sheets) / Search Project Folder / Check Project Folder Exists | Project Engine (Detection → Brain Loader) | Compliant | None | Reuse |
| 16 | Get Project Bible / Download / Extract / Get Sales Playbook / Download / Extract / Merge Bible + Playbook | Project Engine (Brain Loader) | Compliant | None — the Project Brain, faithfully implemented | Reuse |
| 17 | Default Reply – Project Not Found | Safe Fallback Composer | Compliant | None | Reuse |
| 18 | Log Project Not Found | Observability (cross-cutting) | Compliant | None | Reuse |
| 19 | Prepare Context (Set Fields) | Context Builder | Partial | Assembles context correctly but has no Customer Engine input feeding it — no Mental Model exists upstream | Refactor |
| 20 | Is Direct Callback Selection? / Build Direct Callback Reply | Decision Engine (hardcoded special case) | Partial | A legitimate pattern (deterministic classification for a specific, unambiguous input) but undocumented as such | Refactor (formalize as an explicit deterministic Decision Engine path) |
| 21 | Ask MAYA (OpenAI GPT-4o) | Reasoning Engine + Decision Engine + Planning Engine + Recommendation Engine + Response Composer (all five, collapsed) | Non-compliant (structural) | Five distinct Engines with five distinct contracts are currently one LLM call with one combined output schema | Refactor (major) |
| 22 | Parse MAYA Response | Partial stand-in for Reflection Engine | Partial | Only validates JSON structure and supplies a fallback on parse failure; does not evaluate reasoning quality, confidence, or Preconditions | Refactor |
| 23 | Is Parse Error? / Log Parse Error | Observability (cross-cutting) | Compliant | None | Reuse |
| 24 | Is Callback Requested? / Notify – Callback Request / Log Callback Request | Business-specific side action (outside Runtime Engine scope) | Compliant | None — legitimately outside the 23-Engine model, a Propify-specific escalation notification | Reuse |
| 25 | Should Persist Summary? / Persist Conversation State | Memory & State Writer | Partial | Persists conversation_summary and sales_state only; no Mental Model, no Recommendation History, no Turn ID | Refactor |
| 26 | Prepare WhatsApp Payload | Response Composer | Partial | Combined with payload/channel-formatting concerns that belong to WhatsApp Reply | Refactor |
| 27 | Has File Request? / Get Client Files Folder / Find Matching Client File / Client File Found? / Download Client File / Upload Media to WhatsApp / Send Client File via WhatsApp | File Request branch (anticipated, undesigned, in `Runtime_Architecture.md`'s open items) | Compliant with intent | None required now | Reuse |
| 28 | Log File Not Found | Observability (cross-cutting) | Compliant | None | Reuse |
| 29 | Send WhatsApp Reply | WhatsApp Reply | Compliant | None | Reuse |
| 30 | Chatwoot Sync – Outgoing | Memory & State Writer (partial) | Partial | Persists the outgoing message to Chatwoot only, not to the Runtime's own memory/state store | Reuse (as one part of a broader Memory & State Writer) |
| 31 | Send Final Response (Respond to Webhook) | WhatsApp Reply / Communication Gateway (transport close) | Compliant | None | Reuse |
| — | Output Business Rules Gate | — | Missing entirely | No deterministic check exists between the LLM's output and delivery | Build new |
| — | Reflection Engine | — | Missing entirely | No review of reasoning/decision/recommendation quality before sending | Build new |
| — | Turn Identity Contract | — | Missing entirely | No Turn ID exists anywhere in the schema | Build new |
| — | Recommendation History | — | Missing entirely | No Recommendation object is persisted; Preconditions cannot be re-verified | Build new |
| — | Decision Trace / Execution Constraints / Plan object | — | Missing entirely | No structured Decision, Plan, or Recommendation objects exist — only a flat JSON reply | Build new |

---

## Summary Findings

### What already satisfies the architecture (Reuse as-is)

The context-and-infrastructure layer is strong: Communication Gateway, Normalization (including voice transcription), Project Engine (both Detection and Brain Loading, including the conditional re-detection logic), Conversation Context Engine, WhatsApp Reply, and the file-delivery branch all already behave close to, or exactly like, their frozen Engine contracts. This is a significant amount of production-proven work that should not be touched without cause.

### What is partially compliant (Refactor)

Several nodes do real work that maps to a real Engine, but combine responsibilities that the architecture keeps separate, or are missing a field the architecture requires (confidence, Turn ID, structured objects). These need reshaping, not rebuilding:
- Determine Greeting & State is a simplified, non-probabilistic stand-in for Conversation State Recognition.
- Memory persistence (Load/Persist Conversation State, Chatwoot Sync – Outgoing) stores far less than `Memory_State_Writer.md` requires.
- Prepare WhatsApp Payload mixes Response Composer's job with WhatsApp Reply's payload-shaping concern.

**Note:** Finding #9 (Chatwoot Sync – Incoming) previously appeared in this list ("does both Contact Resolution and Memory retrieval in one call"). It has been closed as a False Positive — see "Analysis Update — Finding #9" below.

#### Analysis Update — Finding #9: Chatwoot Sync – Incoming (2026-07-21)

**Original Assessment:** Refactor (split Contact Resolution and Memory Engine, described as "one call today")

**Current Assessment:** Closed — False Positive (Resolved by ADR-001)

**Status:** Closed

A code-level trace of the sub-workflow `Phoenix - Chatwoot Sync` (n8n ID `jky0e1OeyDQuuD1p`), which `Chatwoot Sync – Incoming` calls, found no Memory Engine responsibility inside it at all — it performs Contact Resolution (find/create Chatwoot contact, find/create conversation) followed by a `Create Message` call that posts the message into that Chatwoot conversation. No Data Table is read or written anywhere in this sub-workflow. The actual Memory Engine responsibility (durable `conversation_summary`/`sales_state` retrieval, keyed by `conversation_id`) already exists as a fully separate node, `Load Conversation State` (Finding #12), reading from its own Data Table (`64FX2qQEstLuJSwV`). There was never a combined Contact-Resolution-plus-Memory-Engine call to split — that separation already existed in production.

This left one genuine, narrower architectural question: whether the `Create Message` (Chatwoot message-logging) side effect belongs inside Contact Resolution's boundary or should be its own responsibility. This question was raised as **ADR-001** and decided: **keep the current design** (message logging stays bundled with Contact Resolution inside `Chatwoot Sync – Incoming`/`Chatwoot Sync – Outgoing`), on the grounds that no operational evidence justifies the added complexity and failure surface of splitting a tightly-coupled, single-purpose call, consistent with how `Chatwoot Sync – Outgoing` (Finding #30) was already accepted without a split. Full reasoning, options, and trade-offs are recorded in ADR-001 (change-management record, not committed to this repository — available in the implementation session history).

**Implementation Impact:** None. No node, connection, or Engine Contract was or will be modified for this finding. `Contact_Resolution.md` and `Memory_Engine.md` remain unchanged and unaffected.

### What is structurally non-compliant (the two must-fix findings)

1. **The Input Business Rules Gate runs too late.** In production, the financing/spam check happens after Contact Resolution, Memory retrieval, and full Project Brain loading — all the cost the architecture's early gate exists specifically to avoid. The logic itself (the regex patterns) is sound and reusable; only its position in the pipeline is wrong. **See "Analysis Update — Finding #8" below: a subsequent dependency trace found this is not a pure relocation.**

#### Analysis Update — Finding #8: Input Business Rules Gate (2026-07-21)

**Original Assessment:** Refactor (Relocate only)

**Current Assessment:** Boundary Repair Required before Relocation

**Status:** Partially Implemented — PR-001 Complete (Architectural Leak resolved); PR-002 Pending (Accidental Coupling, re-scoped — see below); Relocation Itself Still Pending

A deeper dependency trace — tracing every field read by `Business Rules Check`, `Blocked by Business Rules?`, and `Log Business Rule Block` back to the node that actually produces it — found that the original assessment understated the work required. Relocating the gate as originally proposed (a pure connection change, no node logic touched) would have silently broken it. Three dependencies were found:

- **Architectural Leak — `customer_message`:** `Business Rules Check` reads `$json.customer_message`, which is set by `Prepare Context (Set Fields)` as a verbatim alias of `resolved_customer_message` (already produced by `Resolve Customer Message`, the Normalization step). No data from Contact Resolution, Memory, or Project Engine is involved — this is a naming artifact of pipeline position, not a real dependency.
- **Accidental Coupling — `Extract Project Code (Regex)`:** `Log Business Rule Block` depends on this node's `projectCode` output. The node's own code reads only `Inspect Referral`'s output (Normalization-stage data) — it has no genuine dependency on Contact Resolution, Memory, or Conversation State. Its late position in the graph (after `Determine Greeting & State`) is an arbitrary wiring choice, not a data requirement.
- **Required Dependency — `conversation_id`:** `Log Business Rule Block` also depends on the Chatwoot `conversation_id`, which is genuinely created/resolved inside Contact Resolution (`Phoenix - Chatwoot Sync` sub-workflow) and cannot exist before it runs. This is the one dependency that is architecturally real and cannot be eliminated — only accommodated (the Feedback Logger's `conversation_id` field is optional by schema, so it can be sent empty when the gate blocks a message before Contact Resolution has run).

**Implementation Impact:** This task is no longer a pure Refactor. It requires removing the two false dependencies (Architectural Leak + Accidental Coupling) before the relocation itself can be safely performed. The relocation plan (three connection changes: `Resolve Customer Message → Business Rules Check`, `Blocked by Business Rules? (Not Blocked) → Chatwoot Sync - Incoming`, `Is Direct Callback Selection? (Not Direct Callback) → Ask MAYA`) remains valid once those two prerequisite fixes are in place.

##### PR-001 — Business Rules Independence (`customer_message`) — Complete

Applied to the development workflow (`MAYA - WhatsApp Sales Agent copy`, n8n workflow ID `jI4meYNr11hP6nbJ`; production `X1QVNNYUFbJftuc2` untouched). `Resolve Customer Message`'s code now also returns `customer_message` as a verbatim duplicate of `resolved_customer_message`, computed at Normalization time instead of only later at `Prepare Context (Set Fields)`. Nothing else changed: same node count (64), identical connections graph, identical credentials on every node, no other node's parameters touched — confirmed by a full structural diff against the pre-change workflow export.

Verification method: **Structural Verification only**, not a live/end-to-end test. Justification: the change is purely additive (a duplicate field, not consumed by any node while the gate has not yet been relocated — `Business Rules Check` still reads `customer_message` from `Prepare Context`, unchanged) with no altered execution path, no altered business logic, no altered consumer, and no altered runtime behavior — the value is provably identical to the existing `resolved_customer_message` field by construction. Per this project's test-selection rule, additive changes with no consumer/runtime-behavior change require structural verification only, not integration or end-to-end testing.

##### PR-002 — Accidental Coupling (`project_code`) — Not started, re-scoped

The originally proposed fix (physically relocating `Extract Project Code (Regex)` to run immediately after `Inspect Referral`) was found, on closer inspection of its outgoing connection, to also drag the entire downstream Project Engine detection/loading chain (`Has Project Code?` through `Merge Bible + Playbook`, ~15 nodes) into an earlier pipeline position — a much larger change than a single-node relocation, and one that touches Project Engine, which is out of scope for a boundary-repair PR. Deferred to its own independent PR. No node has been modified for this finding yet.

2. **The entire Cognitive Core is one LLM call.** "Ask MAYA (OpenAI GPT-4o)" currently performs Reasoning, Decision, Planning, and Recommendation together, and even reaches partway into Response Composer's job, all in a single structured-output schema (reply, sales_state, file_request, callback_requested, metadata.confidence). This is the one finding that genuinely requires new architecture-driven work, not relocation — it is precisely the gap Level 2 exists to close.

#### Analysis Update — Finding #13: Determine Greeting & State (2026-07-21)

**Original Assessment:** Refactor (add confidence value; add explicit position taxonomy from `Conversation_State_Model.md`)

**Current Assessment:** Split into two independent parts with different status.

**Status:** Confidence — Complete (interim representation). Position Taxonomy — Requires ADR, not started.

Verification of `Conversation_State_Model.md` found it does not define any concrete, named taxonomy of conversation-state positions — only general principles (multiple weighted position hypotheses, revised continuously; no fixed labels). The only concrete stage names anywhere in the repository are `Conversation.md`'s "Conversation Lifecycle Stages" (Entry, Understanding, Qualification, Recommendation, Clarification, Decision Support, Human Handoff), a different document describing conversation flow, with no explicit statement that it is meant to be state-recognition's taxonomy. Implementing "an explicit position taxonomy from `Conversation_State_Model.md`" as originally worded would mean inventing a taxonomy not actually present in that document. This part is therefore **Requires ADR** — not implemented, not scheduled, pending a future architectural decision on which document (if any) supplies the taxonomy.

The confidence-value part was independent and well-documented (`Runtime_Architecture.md`'s Confidence Contract explicitly requires Conversation State Recognition to attach a confidence value), but the contract leaves the **value format** itself as an open Level 2 design choice ("How the Confidence Contract is technically represented and thresholded"). Three interim representations were considered — a categorical evidence-source label (`confirmed`/`inferred`), a coarse numeric flag (`0`/`1`), and a structured object with a `basis` field — and rejected: the first conflated confidence with evidence source rather than confidence itself, the second risked being misread as a calibrated probability, and the third was more structural commitment than warranted for a first implementation.

##### Confidence Value — Complete

Applied to the development workflow (`jI4meYNr11hP6nbJ`; production `X1QVNNYUFbJftuc2` untouched). `Determine Greeting & State`'s code now also returns `confidence: hasExistingSummary ? 'high' : 'low'` — a plain two-value categorical label representing confidence level itself (not evidence source, not a numeric calibration). This is explicitly an **interim implementation**, not a finalized calibration standard for the Confidence Contract; it may be replaced by a different representation once that format is formally decided, without requiring any other engine to change (no current consumer reads this field). No other field, node, connection, or credential was touched — confirmed by a full structural diff against the pre-change workflow export (64 nodes before and after, identical connections graph, zero unexpected parameter or credential changes).

Verification method: **Structural Verification only**, consistent with this project's test-selection rule for additive changes with no altered consumer or runtime behavior.

##### Position Taxonomy — Requires ADR, not started

No taxonomy has been assumed, referenced, or implemented. `sales_state`'s existing string values (`'Greeting'`, `'Discovery'`, and whatever was previously persisted) are unchanged. This remains open until a dedicated architectural decision determines the taxonomy's source.

### What is missing entirely (Build new)

The Output Business Rules Gate, the Reflection Engine, the Turn Identity Contract, and Recommendation History do not exist in any form today. These are the direct, load-bearing outputs of the five-journey validation pass — production simply predates them.

---

## Recommended Implementation Order

Following the vertical-slice strategy already agreed, and respecting "never rewrite working production behavior without architectural justification":

1. **Relocate** the Business Rules Check to run immediately after Normalization (low-risk, high-value fix — this alone closes the first structural finding above). PR-001 (Architectural Leak removed) is complete; PR-002 (Accidental Coupling) is pending; the relocation itself has not been performed yet.
2. **Introduce** the Turn Identity Contract at Receive WhatsApp Messages (additive, does not disturb existing behavior).
3. **Decompose** "Ask MAYA" into its constituent Engines — the significant piece of new work, done incrementally: first extract Decision Engine's classification (including the already-correct hardcoded callback pattern) as its own explicit step, then Reasoning, then Planning and Recommendation.
4. **Add** the Output Business Rules Gate and Reflection Engine as new steps before delivery.
5. **Extend** persistence to cover the Mental Model and Recommendation History, not just conversation_summary and sales_state.

Nothing above requires discarding the production workflow. Every step is additive or relocative, consistent with the audit's own findings.
