# Sales State Engine — v1 Implementation Record

**Scope:** MAYA v2, new capability — not part of the frozen Level 1 Runtime Architecture audit (`Implementation_Audit.md`). This is additive work on top of the stabilized production/development split, not a compliance fix.

**Workflow:** Implemented on the development workflow (`MAYA - WhatsApp Sales Agent copy`, n8n ID `jI4meYNr11hP6nbJ`) only. Production (`MAYA - WhatsApp Sales Agent`, `X1QVNNYUFbJftuc2`) was not touched.

**Status:** Implemented, structurally verified, and runtime-verified for the core loop (New → Qualifying → Exploring, with reply-payload integrity and Chatwoot sync confirmed end-to-end across two live executions). Several transition rules follow the same verified code path but were not individually exercised — see Manual Test Scenarios below.

---

## What this replaces

Before this change, `sales_state` was emitted by the "Ask MAYA" LLM call itself, as part of its structured JSON output — the model decided which funnel stage a customer was in, alongside deciding what to say. Sales State Engine removes that responsibility from the model entirely: state is now computed by a deterministic rule engine, and the LLM only reads it as context.

---

## Architecture

### New nodes (development workflow only)

| Node | Type | Position in pipeline |
|---|---|---|
| `Sales State Reader` | Code | Between `Extract Project Code (Regex)` and `Has Project Code?` — runs before the project-found/not-found fork, so it applies to every path including the short-circuited "Project Not Found" reply. |
| `Sales State Writer` | Code | Between `Should Persist Summary?` and `Prepare WhatsApp Payload` (both outputs) — the point where all five reply-generating branches (Financing, Spam, Direct Callback, Ask MAYA, Project Not Found) already converge. |
| `Persist Sales State` | Data Table (upsert) | Immediately after the Writer — unconditional write, independent of the pre-existing conditional `Persist Conversation State`. |
| `Sync Sales State to Chatwoot` | HTTP Request | Immediately after persistence — `POST /conversations/{id}/custom_attributes`, `onError: continueRegularOutput` so a Chatwoot failure never blocks message delivery. |

### Existing nodes modified (minimal, targeted)

- **`Prepare Context (Set Fields)`** — the `sales_state` assignment now reads `$('Sales State Reader')` instead of `$('Determine Greeting & State')`. One expression changed; the other eight fields (project_name, customer_message, project_bible, sales_playbook, conversation_history, conversation_summary, is_first_message, conversation_id) are untouched.
- **`Persist Conversation State`** — the `sales_state` column mapping was removed from its upsert. It now persists only `conversation_id`, `conversation_summary`, `phone_number`, `updated_at`, exactly as before minus the field this new engine now owns exclusively. Removing this avoided a last-write-wins race between two nodes writing the same column under different conditions.
- **`Prepare WhatsApp Payload`** — found and fixed during Runtime Verification (see below): its `reply` / `file_request` / `callback_requested` / `preferred_callback_time` references were redirected from `$json` to `$('Sales State Writer')`, because the two new nodes sitting between them (`Persist Sales State`, `Sync Sales State to Chatwoot`) both replace `$json` with their own operation response, same as any Data Table or HTTP Request node — an item-shape reset, not a bug in either new node individually.

### Data model — Data Table `conversation_state` (`64FX2qQEstLuJSwV`)

Five columns added (all `string`; JSON payloads and ISO timestamps stored as text, consistent with `conversation_history`'s existing storage pattern):

| Column | Holds |
|---|---|
| `previous_active_state` | What Follow-up/Dormant interrupted, for resume. |
| `active_project` | The project code currently being discussed. |
| `last_project_change` | ISO timestamp of the last `active_project` change. |
| `project_history` | JSON array — one entry per distinct project code raised this conversation, each with `first_seen_at`, `last_seen_at`, `turn_count`, `interest_score`. |
| `sales_state_updated_at` | ISO timestamp of the last Sales State write — distinct from the pre-existing `updated_at` (which tracks summary updates only). |

`sales_state` and `conversation_id` already existed on this table and are reused as-is.

---

## Taxonomy and transition rules

Twelve states: `New`, `Qualifying`, `Exploring`, `Interested`, `Objection Handling`, `Awaiting Callback`, `Negotiation`, `Reservation`, `Follow-up`, `Dormant`, `Closed Won`, `Closed Lost`. Full per-state goal/entry/exit/allowed-transitions definitions, the transition diagram, and the Interest Score signal table are in the approved design (not duplicated here to avoid drift between two copies — see the design artifact referenced in the implementation session).

Implemented in v1 (deterministic, in `Sales State Reader` / `Sales State Writer`):
- `New → Qualifying` on the second customer message.
- `Qualifying → Exploring` on first detected project code (also fires directly from `New` when the project code arrives on the customer's first message, e.g. Click-to-WhatsApp ad referral — see Beta Validation Log, Bug 2).
- `Exploring → Interested` on Interest Score ≥ 4 for `active_project` (signals: `callback_requested` +5, `file_request` +3, payment/budget/availability keyword matches +2 each, location keyword +1, 3rd turn on the same project +1).
- `Exploring / Interested / Objection Handling → Awaiting Callback` on `callback_requested = true`.
- Resume from `Follow-up` / `Dormant` to `previous_active_state` on any new customer message.
- `→ Closed Lost` on a narrow, high-precision explicit-decline keyword match, from any non-terminal state.
- Project tracking (`active_project`, `project_history`) independent of `sales_state`, so switching between projects mid-conversation does not reset funnel progress.

## Decisions different from the approved design

1. **Follow-up / Dormant entry is not implemented.** The design specified silence-based entry (3-day / 30-day thresholds), but the pipeline is purely reactive — it only runs when a message arrives, at which point the customer is by definition no longer silent. Entering these states correctly requires a scheduled sweep (a new, separate n8n workflow querying `conversation_state` for stale conversations) that does not exist yet. Implementing it was out of scope for this pass per "least possible change" — flagged here rather than built silently. The *resume* half (exiting Follow-up/Dormant back to `previous_active_state`) is implemented and works today; only the *entry* half is missing, and is inert until that scheduled workflow is built.
2. **Objection Handling, Negotiation, Reservation, Closed Won remain manual-only**, exactly as the approved design specified — reachable only via a human-set Chatwoot custom attribute. The read-back override (engine reading an agent-set attribute as authoritative) was **not implemented**: it would require a `sales_state` Custom Attribute to already be defined at the Chatwoot account level, which was not verified to exist, and guessing at its configuration risked a silent misconfiguration rather than a clean failure. `Sync Sales State to Chatwoot` (write direction) was implemented and confirmed working live — the read-back is the one open half of that mechanism.
3. **`Prepare WhatsApp Payload` was modified** — not anticipated in the design, discovered during Runtime Verification (see below). Scoped to redirecting four expression references to a different existing node; no parameter, business rule, or connection beyond that was touched.

## Risks / known gaps

- Interest Score weights and the threshold (≥4) are reasoned defaults, not calibrated against real conversation data — flagged as provisional in the design and unchanged since.
- Follow-up/Dormant auto-entry inert until a scheduled sweep workflow is built (see Decision 1).
- Chatwoot override read-back not implemented (see Decision 2) — Negotiation/Reservation/Closed Won/Closed Won have no automated or semi-automated entry path yet, only fully manual via direct Data Table edit or a future override mechanism.
- `Sync Sales State to Chatwoot` assumes Chatwoot silently accepts arbitrary custom attribute keys not previously defined at the account level — true in the executions observed during Runtime Verification, but not confirmed against Chatwoot's account-level configuration directly.
- Only the `New → Qualifying → Exploring` path and reply-payload integrity were exercised live end-to-end. Interest Score accumulation, `Awaiting Callback`, and `Closed Lost` share the same verified code path but were not individually triggered in this session.

---

## Manual Test Scenarios (for follow-up bug-hunting)

1. Same conversation, 3+ messages referencing the same project code → confirm `turn_count` increments and the +1 repeated-engagement point lands on exactly the 3rd turn, not every turn after.
2. A message containing a budget figure (e.g. "الميزانية حوالي 5 مليون") on an `Exploring` conversation with an `active_project` → confirm Interest Score reaches ≥4 and `sales_state` becomes `Interested`.
3. A message that triggers `callback_requested` (the existing "رقم للتواصل" flow) from `Exploring` → confirm direct jump to `Awaiting Callback`, bypassing `Interested`.
4. Customer switches project mid-conversation (references a second, different project code) while in `Exploring` or `Interested` → confirm `sales_state` does **not** reset, `active_project` updates, and a second `project_history` entry is created without disturbing the first.
5. A message matching the decline-keyword pattern → confirm `Closed Lost`, and confirm it is **not** falsely triggered by unrelated negative-sounding phrases (precision check).
6. Financing and Spam canned-response paths → confirm they pass through `Sales State Writer` / `Persist Sales State` / `Sync Sales State to Chatwoot` without error (same convergence point as the tested paths, not individually exercised this session).
7. Direct Callback selection path (`Build Direct Callback Reply`) → same convergence check as #6, plus confirm `callback_requested = true` correctly drives `→ Awaiting Callback`.
8. Inspect the Chatwoot conversation UI directly after a test message to confirm the `sales_state` / `active_project` custom attributes are visible to agents, not just present in the API response.
9. A returning customer whose conversation was last active >30 days ago sends a new message → confirm today's known gap (no `Follow-up`/`Dormant` state was ever entered, since no scheduled sweep exists) rather than assuming it was silently handled.

---

## Beta Validation Log

Bug-hunting pass against the live development workflow, started after v1 was marked implemented. Each entry is a real defect reproduced on `jI4meYNr11hP6nbJ`, root-caused, and fixed with the smallest possible change. No new features or refactors were introduced during this pass.

### Bug 1 — Matched-project path unreachable: 13 nodes had no credential assigned

**Symptom:** every execution that ever reached a *real, matched* project code (as opposed to the `Project Not Found` short-circuit) failed with `Node does not have any credentials set` / `Credentials not found`. Prior "verified live" executions referenced in this document's Status line all took the not-found or no-referral path, which never touches these nodes — so this had never been exercised end-to-end since the workflow copy was created.

**Root cause:** `Search Project Folder (Google Drive)`, `Get Project Bible (Drive)`, `Download Project Bible (File)`, `Get Sales Playbook (Drive)`, `Download Sales Playbook (File)`, `Get Client Files Folder`, `Find Matching Client File`, `Download Client File`, `Ask MAYA (OpenAI GPT-4o)`, `Upload Media to WhatsApp`, `Get Media URL`, `Send Client File via WhatsApp`, and `Notify - Callback Request` had no credential wired to them (or referenced a stale/deleted credential ID) on the development copy. Each was fixed to the same credential already used successfully elsewhere in the same workflow: `Google Service Account account` (`kJL1uYdg58njQ54X`) for the Drive nodes, `OpenAI account` (`O26guWZr5ghyoN0e`) for Ask MAYA, `WhatsApp Cloud API Auth` (`RvoZFXZ80vm4GJrg`) for the WhatsApp HTTP nodes.

**Fix:** `setNodeCredential` on each of the 13 nodes, pointing at the correct existing credential. No parameters, logic, or connections touched.

**Verification:** live execution `1231` — a full turn (project-matched question → brochure file request) completed end-to-end: Ask MAYA replied, `Sales State Writer` ran, `Persist Sales State` and `Sync Sales State to Chatwoot` succeeded, the PDF brochure was uploaded to WhatsApp and delivered (`Send Client File via WhatsApp` succeeded), and the reply text was sent.

**Regression check:** re-ran the previously-working `Project Not Found` path (execution `1236`) — still succeeds unchanged, since that path never touches the fixed nodes.

### Bug 2 — First message with a project code never left `New`

**Symptom:** a customer's very first message — the normal Click-to-WhatsApp-ad entry point, where the referral headline (and therefore the project code) arrives together with message #1 — left `sales_state` at `New` indefinitely instead of advancing to `Exploring`, even though `active_project` was correctly detected and recorded.

**Root cause:** in `Sales State Reader`, the project-code block only promoted state to `Exploring` when `salesState === 'Qualifying'`. But `New → Qualifying` only fires when `isFirstMessage === false`, so on message #1 the state was still `New` when the project-code check ran, and the `Exploring` promotion never matched. The transition rule "`Qualifying → Exploring` on first detected project code" (as documented) implicitly assumed the project code always arrives on message 2+; it does not for CTWA leads, which is the majority real-world entry path.

**Fix:** one-line condition change in `Sales State Reader` — `if (salesState === 'Qualifying') salesState = 'Exploring';` → `if (salesState === 'Qualifying' || salesState === 'New') salesState = 'Exploring';`. Nothing else in the node changed.

**Verification:** live execution `1245` on a fresh phone number, first message with `[MRG-SHERATON]` referral → `Sales State Reader` output `sales_state: "Exploring"`, `active_project: "MRG-SHERATON"` on message #1; confirmed synced to Chatwoot and reflected in the reply.

**Regression check:** live execution `1249` — fresh phone number, first message with no referral → `sales_state` correctly stays `"New"`, `active_project` stays `null`. The pre-existing `New → Qualifying` (message 2, no project) and `Qualifying → Exploring` (message 2+, with project) paths are unchanged by this fix.
