# Release Notes — MAYA v2 Beta

**Scope:** Sales State Engine v1, implemented and beta-validated on the development workflow (`MAYA - WhatsApp Sales Agent copy`, n8n ID `jI4meYNr11hP6nbJ`). Production (`MAYA - WhatsApp Sales Agent`, `X1QVNNYUFbJftuc2`) was not touched by any of this work. This release sits on top of, and does not modify, the closed Phoenix Runtime Stabilization work (see `Implementation_Audit.md`) or the frozen Level 1 Runtime Architecture (see `08_Runtime_Architecture/`).

---

## What Was Implemented

`sales_state` is no longer decided by the "Ask MAYA" LLM call as part of its structured output. It is now computed by a deterministic rule engine — `Sales State Reader` (reads prior state, detects project codes, computes Interest Score signals) and `Sales State Writer` (applies file-request/callback-request bonuses, resolves the final state) — with the LLM only reading the result as context. Four new nodes were added: `Sales State Reader`, `Sales State Writer`, `Persist Sales State`, `Sync Sales State to Chatwoot`. Three existing nodes were modified in minimally scoped ways: `Prepare Context (Set Fields)` (one expression redirected), `Persist Conversation State` (`sales_state` column removed from its write, to avoid a race with the new unconditional writer), and `Prepare WhatsApp Payload` (four expression references redirected — found and fixed during runtime verification).

Twelve sales funnel states are modeled: `New`, `Qualifying`, `Exploring`, `Interested`, `Objection Handling`, `Awaiting Callback`, `Negotiation`, `Reservation`, `Follow-up`, `Dormant`, `Closed Won`, `Closed Lost`. Project tracking (`active_project`, `project_history` with per-project turn count and Interest Score) is independent of `sales_state`, so a customer can move between projects without losing funnel progress or resetting their stage.

Full design detail, the complete transition-rule set, and the data model are in `Sales_State_Engine_v1.md`.

## Key Changes From the Pre-Beta Design

- **`Prepare WhatsApp Payload` required a fix not anticipated in the original design.** The two new nodes sitting between the Writer and the payload builder (`Persist Sales State`, `Sync Sales State to Chatwoot`) each replace `$json` with their own operation response — the same item-shape reset any Data Table or HTTP Request node causes. Four expression references were redirected to read from `Sales State Writer` directly instead of `$json`.
- **The `Qualifying → Exploring` transition needed to also fire from `New`.** The original rule only checked `salesState === 'Qualifying'`, which assumed a project code always arrives on the customer's second message or later. It does not for Click-to-WhatsApp ad leads, where the referral (and therefore the project code) arrives together with the very first message — the majority real-world entry point. Fixed as a one-line condition change; see Bugs Fixed below.
- **Follow-up/Dormant auto-entry and the Chatwoot override read-back were both deliberately scoped out**, not overlooked — see Known Limitations.

## Bugs Fixed (Beta Validation)

Two real defects were found and fixed after implementation, through direct code reading followed by live reproduction — not through the Manual Test Scenarios list, which had not yet been run when these were found:

1. **13 nodes had no credential assigned**, blocking every conversation that ever reached a real, matched project code (the `Project Not Found` short-circuit path never touches these nodes, which is why this had gone unnoticed). Affected: 8 Google Drive nodes, `Ask MAYA (OpenAI GPT-4o)`, and 4 WhatsApp-media HTTP nodes. Fixed by wiring each to the correct existing credential. Verified live end-to-end (brochure PDF uploaded and delivered); Project Not Found path regression-checked as unaffected.
2. **A customer's first message never left `New` if it already carried a project code** — the standard Click-to-WhatsApp-ad entry path. Fixed with the one-line condition change described above. Verified live (first message with referral → `Exploring` on turn 1); regression-checked that a first message with no referral still correctly stays `New`.

Full root-cause detail, verification method, and regression checks for both are in `Sales_State_Engine_v1.md`'s Beta Validation Log.

## What Has Been Tested

**Structural + live, pre-beta:** `New → Qualifying → Exploring` core loop, reply-payload integrity, Chatwoot sync (write direction).

**Beta Validation Batch 1 (9 scenarios, live, zero failures):**
- Terminal-state stickiness: `Closed Lost` does not reopen on a new project + strong interest signals (`active_project` still updates; `sales_state` does not).
- Multi-project tracking: switching across 2 and 3 distinct projects, and returning to the first — `project_history` entries stay fully independent (no cross-contamination), the returning-project entry updates in place (not duplicated), and `sales_state` never resets on a project switch.
- Regression: Project Not Found, Project Found (full Drive + LLM + WhatsApp path), Callback, and Brochure delivery all still pass after the credential fix and the `New`-promotion fix.

**Not yet tested** (deferred to later batches): `Closed Won` terminal-state stickiness (blocked — no update/upsert tool available for the Data Table, only insert; requires either a manual edit through the n8n UI or a dedicated data-edit batch), Interest Score cumulative accumulation to the ≥4 threshold in isolation, direct `Exploring → Awaiting Callback` in isolation, Financing/Spam/Direct-Callback convergence onto the new Writer/Persist/Sync point, Google Drive/Sheets empty-result handling beyond what Phoenix Runtime Stabilization already covered, and non-text message types (voice notes, button replies). See `MAYA_Adversarial_Test_Plan.md` (61 scenarios total, sent to the user) for the complete backlog and its priority ordering.

## What Has Not Been Implemented Yet

- **Follow-up/Dormant automatic entry.** The pipeline is purely reactive — it only runs when a message arrives, at which point the customer is by definition not silent. Entering these states on a silence threshold (3-day/30-day) requires a separate scheduled sweep workflow that does not exist yet. The *resume* half (exiting back to `previous_active_state` on a new message) is implemented and works.
- **Chatwoot override read-back.** A human agent can already have their `sales_state` write land in Chatwoot (write direction, confirmed working). The engine does not yet read an agent-set override back as authoritative — this was deliberately not implemented because it would require a `sales_state` Custom Attribute already defined at the Chatwoot account level, which was not verified to exist.
- **PR-002** (Input Business Rules Gate's remaining prerequisite fix) and the **full Ask MAYA decomposition** into separate Reasoning/Decision/Planning/Recommendation Engine calls — both pre-date Sales State Engine v1 and remain open from the original Implementation Audit.

## Known Limitations

- Interest Score weights and the ≥4 threshold are reasoned defaults, not calibrated against real conversation data.
- `Sync Sales State to Chatwoot` assumes Chatwoot silently accepts arbitrary custom-attribute keys not previously defined at the account level — true in every execution observed so far, not confirmed against Chatwoot's account-level configuration directly.
- A Closed-Lost customer who re-engages with a callback request still receives the full callback flow (time-selection buttons, `callback_requested: true`) while `sales_state` remains `Closed Lost` — flagged during Batch 1 as a product-decision question, not fixed, since it wasn't established which behavior is correct.
- `turn_count` in `project_history` only increments on turns where the project's referral code reappears, not on every turn the project stays the active topic — relevant to interpreting the repeated-engagement Interest Score bonus correctly.
- No Data Table update/upsert tool is available in this environment (only insert) — any test or fix requiring a direct edit to an existing `conversation_state` row needs the n8n UI directly.

## Next Steps

Per the adversarial test plan's priority ordering: Google Drive empty-folder-result handling (same bug class as the credential findings, unexercised), the two forbidden-transition checks still open (`Closed Won` stickiness, pending a Data Table edit path), the two open product-decision questions (Closed-Lost-with-callback behavior; whether `sales_state` should be affected by a mid-conversation project switch while in `Awaiting Callback`/`Interested`), then the remaining Batches (Customer Journey, Conversation Memory, Attachments, non-text message types, and the Runtime Failures / data-edit batch). None of this is scheduled to start automatically — each batch proceeds only on explicit instruction, per this project's protocol.
