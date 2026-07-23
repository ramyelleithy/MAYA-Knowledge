> **Provenance:** Recovered from `ramyelleithy/PBI-Sales-Agent` on 2026-07-23. Real history (2026-07-10 to 07-14) of the original server provisioning, Chatwoot install, and early bug fixes — predates this repo and `Implementation_Audit.md`. Kept as historical record, not maintained going forward (use `Implementation_Audit.md` and `Sales_State_Engine_v1.md` for current status).

# CHANGELOG (PBI-Sales-Agent, 2026-07-10 to 2026-07-14)

---

## [1.2.0] — 2026-07-14

### WhatsApp Delivery — Root Cause & Fix

Root cause was two `n8n` bugs — **not** a Meta account/permissions issue (App is Published, Business Verification Approved, webhook/`messages` subscription healthy):

1. **"Send WhatsApp Reply" node** — JSON Body field was in `Fixed` mode instead of `Expression` mode.
2. **"Prepare WhatsApp Payload" node** — the `whatsapp_payload` field was missing the `{{ }}` wrapper around `JSON.stringify()`, so it sent the raw literal code as text instead of an evaluated value.

Both fixed. End-to-end delivery confirmed with multiple real messages received.

**Standing lesson:** when MAYA sends a WhatsApp message and receives a valid `wamid` but the message never arrives, check `n8n` node modes (`Fixed` vs `Expression`) and expression wrappers first — assume Meta infrastructure is fine.

### Chatwoot Integration — Phoenix Chat Layer (built & tested)

Installed **Chatwoot v4.14.0-ce** (web + worker + Redis) on Phoenix Server, behind Nginx with SSL, as a **logging and monitoring layer only** — not an inbound processor. `n8n` remains the sole inbound gateway from Meta, preserving `referral.headline` / project code data.

Built and tested end-to-end the **"Phoenix - Chatwoot Sync"** sub-workflow:

```
Search/Create Contact
    ↓
Get Contact Conversations
    ↓
Filter Open Conversation (Code node: status=open, sort by updated_at DESC)
    ↓
If "Has Open Conversation?"
    ├── true  → use existing conversation_id
    └── false → Create New Conversation → extract id
    ↓
Merge Conversation ID
    ↓
Create Message
```

**Key lesson:** never rely on a POST failing as the "search or create" business logic unless the API actually enforces uniqueness. Chatwoot enforces unique phone numbers on Contacts, but does **not** prevent duplicate Conversations for the same contact + inbox — must explicitly search/filter first, decide, then create.

**Phoenix Chat Layer principle:** "MAYA shouldn't know Chatwoot — Phoenix should know Chatwoot." The `MAYA - WhatsApp Sales Agent` workflow calls the sub-workflow twice — `Chatwoot Sync - Incoming` (after `Inspect Referral`) and `Chatwoot Sync - Outgoing` (after `Send WhatsApp Reply`) — passing `company=propify_real_estate`, `agent=MAYA`, `channel=whatsapp` (currently hardcoded; TODO: move to the `phoenix_config` Postgres schema). This keeps Chatwoot-specific details (like `inbox_identifier`) out of MAYA's own workflow logic.

Recurring `n8n` pitfall hit again here: HTTP Request node fields silently revert to `Fixed` mode instead of `Expression`, sending `{{ }}` expressions as literal text. Always verify the Fixed/Expression toggle when a node returns an unexpected literal-string error or a 404 referencing the raw expression text.

### Netdata — Resource Monitoring (installed)

Installed via the official kickstart script. Port `19999` is correctly blocked by the `ufw` firewall (not publicly accessible) — access is via SSH tunnel:

```
ssh -L 19999:localhost:19999 propify@169.58.3.233
```
then `http://localhost:19999`.

Baseline readings: ~24% RAM, ~3% CPU — well below the 85%-sustained-for-two-weeks threshold that triggers VPS upgrade consideration.

### Pending (next versions)
- Submit Meta App Review for `whatsapp_business_messaging` Advanced Access (currently "Ready for testing" / Standard Access)
- Claude ↔ `n8n` MCP connection (previously deferred; now prioritized over step-by-step Claude-in-Chrome operation)
- Minor cleanup: remove the isolated, now-unused "Search or Create Conversation" / "Search Existing Conversation" nodes from the old `Phoenix - Chatwoot Sync` design
- Resume the Three-Tier Knowledge Model implementation (paused during the Postgres migration)
- Review `Project_Mapping` — several projects still unlinked; must be fixed before any new ad campaign targets a project not yet in the sheet
- Continue Aqarmap scraper data expansion and the WhatsApp groups collector classification/approval workflow

---

## [1.1.0] — 2026-07-12

### Infrastructure Migration: Phoenix Server

**Server provisioning:**
- Provisioned Phoenix Server (Contabo Cloud VPS, Ubuntu 24.04 LTS, 4 vCPU / 8GB RAM / 75GB NVMe, auto-backup enabled)
- Hardened base OS: non-root sudo user (`propify`), SSH key-only auth (password login disabled), UFW firewall (22/80/443), Fail2Ban

**Container platform:**
- Installed Docker Engine + Docker Compose
- Installed Portainer CE for visual container monitoring
- Established `/opt/phoenix` folder structure: `compose/` (one subfolder per service), `data/` (bind-mounted service data), `backups/`, `logs/`, `scripts/`
- Adopted architectural rules: one folder per service, no monolithic docker-compose files, every service ships with `.env` + `docker-compose.yml` + `README.md` from day one

**n8n deployment:**
- Deployed `n8n` + `PostgreSQL` via Docker Compose (replacing the default SQLite) under container names `phoenix-n8n` / `phoenix-postgres` on a dedicated `phoenix-net` Docker network
- Configured `.env`-based secrets, bind-mounted volumes, `unless-stopped` restart policy, and health checks on both containers
- n8n internal port bound to `127.0.0.1` only — never exposed directly to the internet
- Registered `subdomain` `n8n.propify-egy.com`, configured Nginx as reverse proxy, and issued a Let's Encrypt/Certbot SSL certificate (auto-renewing)
- Redis deliberately deferred — no current need for Queue Mode or multiple workers

**Workflow migration (OpenHosst → Phoenix Server):**
- Exported and re-imported the "MAYA - WhatsApp Sales Agent" `n8n` workflow unchanged
- Re-created and re-linked all credentials on the new instance: Google Drive (new Service Account key), Google Sheets, OpenAI (new API key)
- Updated the Meta WhatsApp Business API webhook Callback URL to `https://n8n.propify-egy.com/webhook/whatsapp-webhook` and re-verified successfully
- Published the workflow and ran a full end-to-end test (pinned data + Execute workflow) — all nodes completed successfully with a correct, honest MAYA response

**Documentation:**
- Updated `Architecture` with a new "Infrastructure — Phoenix Server" section and the previously-undocumented "Naming Convention — Project Code" section

### Pending (next versions)
- Install Chatwoot (conversation monitoring + human takeover)
- Install Netdata (resource monitoring; VPS upgrade decision to be based on 85%+ sustained RAM over two weeks, not a guess)
- Resume the Three-Tier Knowledge Model work, paused during the server migration
- Continue the knowledge base and data-tooling items carried over from 1.0.0 (see below)

---

## [1.0.0] — 2026-07-10

### Initial Release (MVP)

**Architecture:**
- Defined core component stack: n8n + OpenAI + Google Drive + GitHub
- Established Three-Tier Knowledge Model (Entry Project → Pivot → Deep Dive)
- Defined Project Code naming convention for Meta Ads
- Established escalation policy (reserve / site visit / unit selection / negotiation only)

**Knowledge Base (Google Drive):**
- Created unified root folder: `Propify Property Intelligence`
- Established standard project folder structure: `Project_Bible.md + Sales_Playbook.md + Client_Files/`
- Documented 9 projects (8 active + 1 empty template flagged for deletion):
  - Margins: Sheraton Residences ✅ (official brochure)
  - Margins: Lusail Residence ✅ (official brochure)
  - Margins: Oaks ✅ (official brochure)
  - Margins: Zia ✅ (official brochure)
  - Upwyde: White Residence ⚠️ (internet sources, price conflicts documented)
  - Madinet Masr: Taj City ⚠️ (internet sources, price conflicts by phase)
  - Madinet Masr: Sarai ⚠️ (internet sources, price conflicts by phase)
  - El Batal: Rock Vera ⚠️ (internet sources, major price conflict 3.75M vs 10.7M)
  - Mountain View iCity 🚫 (empty template — not a real project, flagged for manual deletion)

**Data Tools:**
- WhatsApp Collector (Node.js + Baileys): monitoring 163 developer groups, writing to Raw_Data/WhatsApp/
- Aqarmap Scraper (Node.js + Playwright): successfully scraped التجمع الخامس → 275 unique projects after dedup
- Projects_Master (Google Sheet): pilot batch of 25 projects + 275 from scraper pending import

**Pending (next versions):**
- Import 275 Aqarmap projects into Projects_Master
- Build WhatsApp classification + approval workflow in n8n
- Verify existing n8n workflow after Drive restructure
- Expand knowledge base beyond current 8 projects
- Obtain official brochures for non-official project files
