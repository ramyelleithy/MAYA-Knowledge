# Implementation Audit — Production Workflow vs. Phoenix Runtime v1.0

**Source workflow:** `MAYA - WhatsApp Sales Agent` (`X1QVNNYUFbJftuc2`), active, last updated 2026-07-17.
**Method:** Every node in the production workflow, mapped against the frozen Level 1 Runtime Architecture. No redesign performed. This is a mapping exercise only.

**Scope note:** This audit maps the 23-Engine model only. It does not cover **Sales State Engine v1**, a separate additive capability built on the development workflow (`jI4meYNr11hP6nbJ`) after this audit — see `Sales_State_Engine_v1.md`. That work replaced how `sales_state` is produced (see row 21 and row 25 below, both updated to reflect it) but is not itself part of the frozen Level 1 Engine Mapping Matrix.

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
| 15 | Get Project Code Mapping (Sheets) / Search Project Folder / Check Project Folder Exists | Project Engine (Detection → Brain Loader) | Boundary Repair Complete | None — see Analysis Update below (Missing Empty-Result Handling) | Boundary Repair — Complete |
| 16 | Get Project Bible / Download / Extract / Get Sales Playbook / Download / Extract / Merge Bible + Playbook | Project Engine (Brain Loader) | Compliant | None — the Project Brain, faithfully implemented | Reuse |
| 17 | Default Reply – Project Not Found | Safe Fallback Composer | Boundary Repair Complete | None — see Analysis Update below (Finding #17) | Boundary Repair + Bug Fix — Complete |
| 18 | Log Project Not Found | Observability (cross-cutting) | Compliant | None | Reuse |
| 19 | Prepare Context (Set Fields) | Context Builder | Partial | Assembles context correctly but has no Customer Engine input feeding it — no Mental Model exists upstream | Refactor |
| 20 | Is Direct Callback Selection? / Build Direct Callback Reply | Decision Engine (hardcoded special case) | Partial | A legitimate pattern (deterministic classification for a specific, unambiguous input) but undocumented as such | Refactor (formalize as an explicit deterministic Decision Engine path) |
| 21 | Ask MAYA (OpenAI GPT-4o) | Reasoning Engine + Decision Engine + Planning Engine + Recommendation Engine + Response Composer (all five, collapsed) | Non-compliant (structural) | Five distinct Engines with five distinct contracts are currently one LLM call with one combined output schema. On the development workflow, the schema's own `sales_state` field is now vestigial — it is still emitted by the LLM but no longer read by anything downstream; `sales_state` is produced separately by Sales State Engine v1 (see `Sales_State_Engine_v1.md`) | Refactor (major) |
| 22 | Parse MAYA Response | Partial stand-in for Reflection Engine | Partial | Only validates JSON structure and supplies a fallback on parse failure; does not evaluate reasoning quality, confidence, or Preconditions | Refactor |
| 23 | Is Parse Error? / Log Parse Error | Observability (cross-cutting) | Compliant | None | Reuse |
| 24 | Is Callback Requested? / Notify – Callback Request / Log Callback Request | Business-specific side action (outside Runtime Engine scope) | Compliant | None — legitimately outside the 23-Engine model, a Propify-specific escalation notification | Reuse |
| 25 | Should Persist Summary? / Persist Conversation State | Memory & State Writer | Partial | On the development workflow, persists `conversation_summary` only — `sales_state` was deliberately removed from this node's write in favor of the new unconditional `Persist Sales State` node (Sales State Engine v1), to avoid a last-write-wins race between two nodes writing the same column; no Mental Model, no Recommendation History, no Turn ID | Refactor |
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

#### Analysis Update — Finding #15: Missing Empty-Result Handling after Project Mapping Lookup (2026-07-21)

**Original Assessment:** Compliant (no action required)

**Current Assessment:** Boundary Repair Required — Complete

**Status:** Closed — Fixed and verified end-to-end

**Root Cause:** `Get Project Code Mapping (Google Sheets)` has no `Always Output Data` setting. When a project code has no matching row (a customer references a project code the business doesn't recognize), the node returns zero items. n8n's default execution semantics mean a node receiving zero input items does not execute at all — so the entire downstream chain (`Search Project Folder` → `Check Project Folder Exists` → `Default Reply - Project Not Found`) silently never ran. The execution completed with status `success` (not an error), making this failure invisible to any error-based monitoring. This is a genuine defect, not a relocation or documentation issue: architecturally, the Sheets mapping is the authoritative directory for Project Code → Folder Name — an unmapped code should short-circuit immediately to Safe Fallback Composer, with no reason to query Google Drive at all.

**Engine Contract Affected:** The implicit item-propagation contract between the Project Engine's mapping-lookup step and its Brain-Loader continuation — a lookup step must always emit at least one item downstream (a found row, or an explicit not-found signal), for either branch to execute.

**Runtime Contract Affected:** The Project Not Found notification path — the customer-facing guarantee that an unrecognized project code produces an explicit reply. This could not be fulfilled because execution died upstream of the reply logic.

**Fix Applied (development workflow `jI4meYNr11hP6nbJ`; production `X1QVNNYUFbJftuc2` untouched):**
- `Get Project Code Mapping (Google Sheets)`: `Always Output Data` enabled (Node Setting only — no parameter, expression, or credential changed).
- New node `Project Mapping Found?` (IF) inserted between `Get Project Code Mapping (Google Sheets)` and `Search Project Folder`, checking whether `Folder Name` is non-empty. True → `Search Project Folder` (identical to the previous direct connection, for every case where a row is actually found — this is a structural no-op for the existing "found" path). False → `Default Reply - Project Not Found` + `Log Project Not Found` (both pre-existing targets, reused as-is).
- `Search Project Folder` and `Check Project Folder Exists` were deliberately left untouched — an earlier candidate fix (only enabling `Always Output Data` with a Drive-query fallback string) was rejected after analysis showed it would risk a false-positive Drive match (`contains ''` matches every file in the parent folder) rather than the architecturally correct immediate short-circuit.

**Verification method:** Structural Verification (65 nodes; the new node plus one `alwaysOutputData: true` setting were the only changes; all connections diffed and confirmed unchanged elsewhere) and full Runtime Verification (Execution `1200` on `jI4meYNr11hP6nbJ`, `TEST_NONEXISTENT_PROJECT_ZZZ` scenario) — see Finding #17 below for the shared end-to-end result, since both fixes were validated in the same execution.

#### Analysis Update — Finding #17: Default Reply – Project Not Found (2026-07-21)

**Original Assessment:** Compliant (no action required)

**Current Assessment:** Boundary Repair + Bug Fix — Complete, verified end-to-end

**Status:** Closed

This node was blocked from ever executing until Finding #15 (above) was fixed. Once reachable, two further defects were found and fixed in sequence, plus one false alarm was resolved.

**Root Cause 1 — Lost Credential Bindings (Boundary Repair):** The development workflow (`jI4meYNr11hP6nbJ`), a duplicate of production, lost three credential bindings during the duplication process: `Load Conversation Messages` (Chatwoot API Auth), `Get Project Code Mapping (Google Sheets)` (Google Service Account account), and `Send WhatsApp Reply` (WhatsApp Cloud API Auth). Each surfaced only when the pipeline reached that specific node. Fixed by re-attaching the same credentials already in use on the equivalent production nodes — no parameter, expression, or connection touched on any of the three nodes.

**Root Cause 2 — Reply Field Contract Violation (Bug Fix):** `Default Reply - Project Not Found` wrote its message text to a field named `reply_text`, then attempted `reply: {{ $json.reply_text }}` in the same Set node. n8n Set-node assignments do not chain — `$json` inside every assignment refers to the node's original input, not to sibling assignments computed in the same operation — so `reply` always evaluated against the node's actual (empty) input and resolved to `null`. Every other reply-producing node in the workflow (`Parse MAYA Response`, `Canned Response - Financing`, `Canned Response - Spam`, `Build Direct Callback Reply`, and the `Ask MAYA` LLM's own structured-output schema) already used `reply` as the sole canonical field — confirmed by a full workflow-wide audit of every writer and reader of `reply`/`reply_text`. `reply_text` is not a parallel contract; it exists only as a derived logging alias created downstream in `Prepare WhatsApp Payload` for `Chatwoot Sync - Outgoing`. Fixed by rewriting `Default Reply - Project Not Found` to assign `reply` directly (matching the pattern of every other reply-producing node) and removing the broken derived assignment. No other node was touched — `Prepare WhatsApp Payload` and `Chatwoot Sync - Outgoing` already correctly consumed `reply` / derived their own `reply_text`, and self-corrected once given a valid `reply` value.

**False Alarm — Test Data, Not a System Defect:** During external verification, `Send WhatsApp Reply` failed against the live Meta Graph API with a generic `(#100) Invalid parameter` error, reproducible identically outside n8n (Graph API Explorer, direct HTTP calls) with a verified-valid System User token, correct WABA/Phone-Number ownership, and an open 24-hour customer window. The cause was eventually isolated: the test scenario's simulated customer number (`201505158793`) was, digit-for-digit, the business's own registered WhatsApp number (`+20 15 05158793`) — Meta's API correctly rejects a Business Phone Number attempting to message itself. This was a defect in the test fixture used for manual verification, not in MAYA, n8n, or the Meta integration. No system change resulted from this finding; noted here so future testing avoids reusing the business's own number as a simulated customer.

**Final Verification — Execution `1200`** (development workflow, `jI4meYNr11hP6nbJ`, real recipient number, `TEST_NONEXISTENT_PROJECT_ZZZ` scenario): full end-to-end success —
`Get Project Code Mapping (Google Sheets)` → `Project Mapping Found?` (false) → `Default Reply - Project Not Found` (`reply` populated correctly) → `Should Persist Summary?` → `Prepare WhatsApp Payload` (`text.body` populated correctly) → `Send WhatsApp Reply` (Meta returned a real `wamid`, message delivered and confirmed received) → `Chatwoot Sync - Outgoing` (logged) → `Send Final Response`. `Log Project Not Found` also fired exactly once, unchanged. Production (`X1QVNNYUFbJftuc2`) was not modified at any point in this investigation.

**Regression check:** The only connection change with any reach into other conversation paths was the `Get Project Code Mapping (Google Sheets) → Project Mapping Found?` reroute (Finding #15) — for a matched project code, `Folder Name` is non-empty, so the node routes to `Search Project Folder` exactly as the previous direct connection did; behavior for the "project found" path is unchanged by construction. Financing, Spam, Direct Callback, and File Request paths share no modified node or connection with this session's fixes (confirmed by structural diff) and were not exercised.

### What is missing entirely (Build new)

The Output Business Rules Gate, the Reflection Engine, the Turn Identity Contract, and Recommendation History do not exist in any form today. These are the direct, load-bearing outputs of the five-journey validation pass — production simply predates them.

---

## Recommended Implementation Order

Following the vertical-slice strategy already agreed, and respecting "never rewrite working production behavior without architectural justification":

1. **Relocate** the Business Rules Check to run immediately after Normalization (low-risk, high-value fix — this alone closes the first structural finding above). PR-001 (Architectural Leak removed) is complete; PR-002 (Accidental Coupling) is pending; the relocation itself has not been performed yet.
2. **Introduce** the Turn Identity Contract at Receive WhatsApp Messages (additive, does not disturb existing behavior).
3. **Decompose** "Ask MAYA" into its constituent Engines — the significant piece of new work, done incrementally: first extract Decision Engine's classification (including the already-correct hardcoded callback pattern) as its own explicit step, then Reasoning, then Planning and Recommendation.
4. **Add** the Output Business Rules Gate and Reflection Engine as new steps before delivery.
5. **Extend** persistence to cover the Mental Model and Recommendation History, not just conversation_summary and (as of Sales State Engine v1) sales_state.

Nothing above requires discarding the production workflow. Every step is additive or relocative, consistent with the audit's own findings.

---

## Status as of Sales State Engine v1

Findings #8 (Input Business Rules Gate boundary repair, PR-001), #9, #13 (Confidence half), #15, and #17 are closed, as detailed in their Analysis Update sections above — this closes what this project's protocol calls **Phoenix Runtime Stabilization**. PR-002 (Finding #8's remaining prerequisite) and the Position Taxonomy half of Finding #13 remain open, as does everything under "What is missing entirely."

Separately, and orthogonally to this audit, **Sales State Engine v1** was implemented on the same development workflow — see `Sales_State_Engine_v1.md` for its architecture, transition rules, and a Beta Validation Log of bugs found and fixed against it after implementation. Two bugs were found and fixed there (13 nodes missing credentials on the matched-project path; `New` never promoting to `Exploring` when a project code arrives on a customer's first message), and Batch 1 of adversarial testing (9 scenarios: terminal-state stickiness, multi-project tracking, and regression across Project Not Found / Project Found / Callback / Brochure) passed with zero further defects. See `Release_Notes_MAYA_v2_Beta.md` for the full account.
