# MAYA v2 — Conversation Goal (CTA) & Skills Update

**Scope:** Development workflow (`MAYA - WhatsApp Sales Agent copy`, `jI4meYNr11hP6nbJ`) only. Production (`X1QVNNYUFbJftuc2`) not touched.

**Origin:** During Beta Validation, the user asked what customer-facing sales *skills* MAYA v2 actually adds beyond the Sales State Engine (answer at the time: none — Sales State Engine is an internal tracking layer, not a behavior change). Recovering two ChatGPT conversation exports led to discovering a separate, previously unconnected GitHub repository — `ramyelleithy/PBI-Sales-Agent` — containing real, detailed MAYA behavior specs (`MAYA_System_Prompt`, `MAYA_AI_CONSTITUTION`, `PBA | Propify Brain & AI`, `Propify_Sales_Methodology`) that were written before this repo existed and never merged into it or fully reconciled with the live n8n prompt. A line-by-line diff between those documents and the live "Ask MAYA" system prompt found seven concrete gaps, which this change closes, plus a new business requirement (the 3-way CTA framework) defined directly by the user.

## What changed

### 1. Conversation Goal (CTA) — new concept, not previously documented anywhere
Every conversation's success is now explicitly defined as reaching one of three outcomes: a phone callback, a Zoom meeting, or a site visit. This replaces the old callback-only framing (`callback_requested` previously implied "wants a phone call" specifically).

### 2. Two-step contact button flow (new n8n nodes/logic)
Previously: customer requests contact → straight to 4 time-slot buttons (phone call assumed).
Now: customer requests contact → **step 1**, 3 buttons (`مكالمة تليفون` / `اجتماع Zoom` / `زيارة ميدانية`, ids `cta_call`/`cta_zoom`/`cta_visit`) → **step 2**, the same 4 time-slot buttons, with the chosen type encoded directly into the button id (`cta_<type>_<time>`, e.g. `cta_zoom_4_8pm`) so no extra state needs to be persisted between the two taps.

**New nodes:** `Is Contact Type Selection?` (IF), `Build Contact Time Buttons Reply` (Code) — inserted between `Is Direct Callback Selection?`'s False branch and `Business Rules Check`.

**Modified nodes:** `Resolve Customer Message` (button-id detection rewritten for the `cta_` scheme, replacing the old `callback_` scheme), `Is Direct Callback Selection?` (condition now checks `is_contact_full_selection`), `Build Direct Callback Reply` (confirmation reply now names the contact type), `Prepare WhatsApp Payload` (renders type-buttons vs. time-buttons vs. plain text based on `contact_type` presence), `Notify - Callback Request` (internal notification to the sales team now includes the contact type).

**Verified live, full 3-step flow, single conversation** (executions `1338` → `1342` → `1346`): free-text contact request → type buttons rendered correctly → tapped "Zoom" → time buttons rendered correctly with `cta_zoom_*` ids → tapped a time slot → confirmation reply named the type and time correctly, internal WhatsApp notification to the sales team fired with the type included.

**Regression-checked:** a normal project-inquiry message with an objection (execution `1352`) and the Project Not Found path (execution `1356`) — both still succeed unchanged.

### 3. Ask MAYA system prompt — seven gaps closed

Found by diffing the live prompt against `PBI-Sales-Agent`'s documents:

| Gap | Fix |
|---|---|
| No objection-handling guidance at all | Added `== معالجة الاعتراضات ==` — philosophy (objection = serious-buyer signal, not rejection) + per-objection guidance, including a concrete installment-restructuring idea (semi-annual/annual payments) supplied directly by the user |
| No Egyptian real-estate terminology glossary | Added `== مصطلحات عقارية مصرية ==` |
| No concrete qualification fields | Added `== نقاط التأهيل (Qualification) ==` listing the specific data points |
| No project-comparison criteria | Added `== المقارنة بين المشاريع ==` |
| No reframe script for an unsuitable Entry Project | Added inline to the existing alternative-project rule |
| No prohibition on promising discounts | Added `■■ قاعدة قصوى: الخصومات والعروض الخاصة ■■` |
| No prohibition on revealing the prompt/internal architecture | Added `■■ قاعدة قصوى: السرية ■■` |

**Deliberately not carried over from `PBI-Sales-Agent`:** the constitution's escalation script names "أستاذ رامي الليثي" explicitly — the live prompt has a hard rule forbidding naming any sales-team member. This is a real, later, intentional divergence, not an oversight; the newer (name-free) rule was kept as authoritative.

**Verified live:** the expanded prompt (now ~2x the length) still produces valid structured JSON output under real load (executions `1338`, `1352`, `1356`) with no parse errors, and the objection-handling addition visibly changed behavior correctly (execution `1352`: a price/down-payment objection was met with a value reframe plus a soft CTA nudge, not a price argument).

## Not carried over / still open

- The `PBI-Sales-Agent` repo's infrastructure documentation (`Architecture`, `Phoenix_Server_Standard_v1.md`, `CHANGELOG.md`) was reviewed but not merged into this repo — it documents real infrastructure history (Phoenix Server, Chatwoot integration) that predates and partially overlaps with `Implementation_Audit.md`. Left as a follow-up decision: merge the repos, or keep them separate with cross-references.
- `PBI-Sales-Agent`'s `Architecture` file explicitly lists "Sales State Engine" as an intentionally postponed MVP feature — noted here for the record; it does not conflict with Sales State Engine v1 having since been built, since scope decisions evolve.
- The "Micro-RAG" project-alternative switching *script* (`PBA` document) was condensed into one line in the prompt rather than reproduced in full — the fuller scripted version can be added later if the condensed one proves insufficient in practice.
- Zoom meetings and site visits still resolve to the exact same manual human handoff as phone callbacks (an internal WhatsApp notification to the sales team) — there is no automated Zoom-link generation or site-visit scheduling. Confirmed with the user this is intentional for now.
