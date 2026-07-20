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
| 8 | Business Rules Check / Blocked by Business Rules? / Is Spam Category? / Canned Response – Financing / Canned Response – Spam | Input Business Rules Gate + Safe Fallback Composer | Non-compliant (sequencing) | Move to run immediately after Normalization, before Contact Resolution / Memory | Refactor (relocate; logic itself reusable) |
| 9 | Chatwoot Sync – Incoming | Contact Resolution + Memory Engine (combined) | Partial | Split responsibilities: contact/conversation resolution vs. memory retrieval are one call today | Refactor |
| 10 | Load Conversation Messages | Conversation Context Engine | Compliant | None | Reuse |
| 11 | Format Conversation History | Conversation Context Engine | Compliant | None | Reuse |
| 12 | Load Conversation State | Memory Engine | Partial | Extend schema: no Turn ID, no Recommendation History | Refactor |
| 13 | Determine Greeting & State | Conversation State Recognition | Partial | Currently rule-based on message count / stored summary presence only; no confidence value, no explicit position taxonomy from `Conversation_State_Model.md` | Refactor |
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
- Chatwoot Sync – Incoming currently does both Contact Resolution and Memory retrieval in one call.
- Determine Greeting & State is a simplified, non-probabilistic stand-in for Conversation State Recognition.
- Memory persistence (Load/Persist Conversation State, Chatwoot Sync – Outgoing) stores far less than `Memory_State_Writer.md` requires.
- Prepare WhatsApp Payload mixes Response Composer's job with WhatsApp Reply's payload-shaping concern.

### What is structurally non-compliant (the two must-fix findings)

1. **The Input Business Rules Gate runs too late.** In production, the financing/spam check happens after Contact Resolution, Memory retrieval, and full Project Brain loading — all the cost the architecture's early gate exists specifically to avoid. The logic itself (the regex patterns) is sound and reusable; only its position in the pipeline is wrong.

2. **The entire Cognitive Core is one LLM call.** "Ask MAYA (OpenAI GPT-4o)" currently performs Reasoning, Decision, Planning, and Recommendation together, and even reaches partway into Response Composer's job, all in a single structured-output schema (reply, sales_state, file_request, callback_requested, metadata.confidence). This is the one finding that genuinely requires new architecture-driven work, not relocation — it is precisely the gap Level 2 exists to close.

### What is missing entirely (Build new)

The Output Business Rules Gate, the Reflection Engine, the Turn Identity Contract, and Recommendation History do not exist in any form today. These are the direct, load-bearing outputs of the five-journey validation pass — production simply predates them.

---

## Recommended Implementation Order

Following the vertical-slice strategy already agreed, and respecting "never rewrite working production behavior without architectural justification":

1. **Relocate** the Business Rules Check to run immediately after Normalization (low-risk, high-value fix — this alone closes the first structural finding above).
2. **Introduce** the Turn Identity Contract at Receive WhatsApp Messages (additive, does not disturb existing behavior).
3. **Split** Chatwoot Sync – Incoming into a genuine Contact Resolution step and a genuine Memory Engine retrieval step.
4. **Decompose** "Ask MAYA" into its constituent Engines — the significant piece of new work, done incrementally: first extract Decision Engine's classification (including the already-correct hardcoded callback pattern) as its own explicit step, then Reasoning, then Planning and Recommendation.
5. **Add** the Output Business Rules Gate and Reflection Engine as new steps before delivery.
6. **Extend** persistence to cover the Mental Model and Recommendation History, not just conversation_summary and sales_state.

Nothing above requires discarding the production workflow. Every step is additive or relocative, consistent with the audit's own findings.
