> **Provenance & status:** Recovered from `ramyelleithy/PBI-Sales-Agent` on 2026-07-23. Real infrastructure history (Phoenix Server, Chatwoot, n8n deployment) that predates and partially overlaps with `Implementation_Audit.md`. Note: the "MVP Scope" section below lists "Sales State Engine" as an intentionally postponed feature — Sales State Engine v1 has since been built (see `Sales_State_Engine_v1.md`); that's a later scope decision, not a contradiction to resolve.

# MAYA Architecture

## Overview

MAYA is the AI Sales Executive of Propify Brokerage Agency.

Its mission is to communicate with real estate leads through WhatsApp, qualify them, answer project questions, send official project files, handle objections, and move qualified leads toward booking or purchasing.

This is Version 1 (MVP).

The architecture is intentionally simple to allow fast deployment, fast testing, and continuous improvement.

---

# Core Components

## GitHub

GitHub is the permanent knowledge repository of MAYA.

It stores everything that defines how MAYA thinks and works.

Examples:

- MAYA System Prompt
- Propify Sales Methodology
- Architecture Documentation
- AI Rules
- Business Rules
- Future Improvements
- Changelog

GitHub never stores project operational content such as brochures, price lists, images, or videos.

---

## Google Drive

Google Drive is MAYA's operational knowledge base.

Each project has its own folder.

Example:

MAYA/
└── Projects/
    └── Mountain View iCity/
        ├── Project_Bible.md
        ├── Sales_Playbook.md
        ├── Client_Files/
        ├── Price List.xlsx
        └── Unit Plans/

Google Drive stores:

- Project Bible
- Sales Playbook
- Price Lists
- Payment Plans
- Brochures
- Master Plans
- Images
- Videos
- Unit Plans
- Client Files

Google Drive is the single source of truth for all project content.

---

## n8n

n8n is the workflow and automation engine.

Responsibilities:

- Receive WhatsApp messages.
- Read Meta Click-to-WhatsApp referral.
- Detect the entry project.
- Load the required project files from Google Drive.
- Build the context.
- Call OpenAI.
- Retrieve requested files.
- Send files back to the customer.
- Save conversation history.

n8n is self-hosted (not the cloud/managed offering). See the **Infrastructure — Phoenix Server** section below for how and where it runs.

---

## OpenAI

OpenAI is the conversation engine.

Responsibilities:

- Understand customer messages.
- Follow the MAYA System Prompt.
- Use the Project Bible and Sales Playbook provided by n8n.
- Generate natural sales conversations.
- Qualify customers.
- Handle objections.
- Guide customers toward the next sales step.

OpenAI never becomes the source of truth.

It only reasons over the context provided to it.

---

# Infrastructure — Phoenix Server

n8n runs self-hosted on **Phoenix Server**, a dedicated VPS that is the shared foundation for MAYA and future Phoenix OS agents — not a single-purpose n8n box.

**Server:** Contabo Cloud VPS, Ubuntu 24.04 LTS, 4 vCPU / 8GB RAM / 75GB NVMe, auto-backup enabled.

### Stack

- **Docker + Docker Compose** — every service is fully independent (no monolithic compose files).
- **PostgreSQL** — n8n's database (not the default SQLite), for production stability.
- **Nginx** (reverse proxy) + **Let's Encrypt/Certbot** — n8n is reachable only via `https://n8n.propify-egy.com`, never by raw IP or unencrypted port.
- **Portainer** — visual container monitoring and management (`https://<server-ip>:9443`).
- **Redis** — deliberately deferred. Will be added only when there is a real, measured need (Queue Mode, multiple workers, or execution load that requires it) — not preemptively.

### Server folder structure

```
/opt/phoenix
├── compose/          # one folder per service (n8n, chatwoot, uptime-kuma, portainer, ...)
│   └── n8n/
│       ├── docker-compose.yml
│       ├── .env
│       └── README.md
├── data/              # bind-mounted service data (n8n, postgres, chatwoot, redis)
├── backups/
├── logs/
└── scripts/           # backup.sh, restore.sh, update.sh — added as real needs arise
```

### Architectural rules

1. **One folder per service** — nothing shared between services' compose files.
2. **No monolithic docker-compose** — each service can be updated or restarted independently without touching the others (e.g. `cd compose/chatwoot && docker compose pull && docker compose up -d`).
3. **Every service ships with `.env`, `docker-compose.yml`, and `README.md` from day one**, even if the README is two lines.

### n8n deployment specifics

- Container names: `phoenix-n8n`, `phoenix-postgres`.
- Dedicated Docker network: `phoenix-net`.
- n8n's internal port (5678) is bound to `127.0.0.1` only — never exposed directly to the internet. Nginx is the only public-facing entry point.
- Health checks configured on both the n8n and Postgres containers.
- Restart policy: `unless-stopped`.

### Scaling policy

Upgrading the VPS tier is based on **measured data from Netdata** (e.g. sustained 85%+ RAM for two continuous weeks) — never a guess.

### Implemented

- **Chatwoot** (v4.14.0-ce) — conversation logging + human takeover. See **Phoenix Chat Layer** below.
- **Netdata** — resource monitoring, installed via kickstart script. Port `19999` blocked by `ufw`; accessed only via SSH tunnel (`ssh -L 19999:localhost:19999 propify@169.58.3.233`, then `http://localhost:19999`). Feeds the scaling policy above.

### Deferred / not yet built

- Open WebUI / Ollama (needs GPU, not a current priority)
- Additional MCP servers

---

# Phoenix Chat Layer — Chatwoot Integration

**Principle:** MAYA shouldn't know Chatwoot — Phoenix should know Chatwoot.

Chatwoot is a **logging and monitoring layer only**, never an inbound processor. `n8n` remains the sole inbound gateway from Meta, which preserves `referral.headline` / Project Code data that would otherwise be lost. Each company (Propify, Canton Fashion, Afflou, Sheetify) gets its own separate Chatwoot Inbox.

Integration logic lives in a reusable sub-workflow, **"Phoenix - Chatwoot Sync"**, kept separate from MAYA's own workflow:

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

**Key lesson:** never rely on a POST failing as "search or create" logic unless the API actually enforces uniqueness. Chatwoot enforces unique phone numbers on Contacts, but does **not** prevent duplicate Conversations for the same contact + inbox — must explicitly search/filter first, then decide, then create.

The `MAYA - WhatsApp Sales Agent` workflow calls this sub-workflow twice:

- **Chatwoot Sync - Incoming** — after `Inspect Referral`
- **Chatwoot Sync - Outgoing** — after `Send WhatsApp Reply`

Both calls pass `company=propify_real_estate`, `agent=MAYA`, `channel=whatsapp` (currently hardcoded — TODO: move to the `phoenix_config` Postgres schema, which already holds normalized `companies` / `channels` / `agents` / `inboxes` tables). MAYA's workflow never references Chatwoot's `inbox_identifier` directly; that resolution happens inside the sub-workflow.

---

# Runtime Flow

Customer

↓

Meta Click-to-WhatsApp

↓

n8n Webhook

↓

Extract Referral

↓

Chatwoot Sync - Incoming (Phoenix Chat Layer, logging only)

↓

Identify Entry Project

↓

Load Project Bible

↓

Load Sales Playbook

↓

Build Context

↓

OpenAI

↓

Generate Response

↓

If customer requests a file

↓

Retrieve file from Google Drive

↓

Send WhatsApp Reply

↓

Chatwoot Sync - Outgoing (Phoenix Chat Layer, logging only)

↓

Save Conversation

---

# Source of Truth

AI Rules

→ GitHub

Project Knowledge

→ Google Drive

Automation

→ n8n

Conversation

→ OpenAI

Infrastructure

→ Phoenix Server (this document)

---

# MVP Scope

Version 1 includes:

- WhatsApp conversations
- Meta Click-to-WhatsApp
- One project per conversation entry
- Project Bible
- Sales Playbook
- Google Drive knowledge
- Official file delivery
- Customer qualification
- Objection handling
- Human handoff when required

The following features are intentionally postponed:

- Recommendation Engine
- Planner
- Scoring System
- Sales State Engine
- Multi-project intelligence
- Advanced Memory
- Analytics Engine

---

# Design Principles

- Keep the architecture simple.
- GitHub defines HOW MAYA works.
- Google Drive contains WHAT MAYA knows.
- n8n orchestrates the workflow.
- OpenAI generates the conversation.
- Add complexity only after the MVP proves itself.

---

## Naming Convention — Project Code

كل إعلان (`Ad`) على `Meta` لازم يبدأ الـ (`headline`) بتاعه بكود مشروع ثابت بين قوسين مربعين، مثال:

```
[MV_ICITY] موانتن فيو أيسيتي - وحدات بإطلالة على البحيرة
```

**السبب:** الاعتماد على اسم الحملة (`Campaign Name`) أو الـ (`Ad ID` / `source_id`) غير موثوق، لأن مشروع واحد ممكن يبقى له عدة حملات وعدة (`Ad IDs`) بمرور الوقت. الـ (`Project Code`) ثابت ومتحكم فيه بالكامل من رامي وقت إنشاء أي إعلان جديد، مش (`Meta`) اللي بتولّده.

**كيف يعمل داخل `n8n`:**

```
Inspect Referral (headline)
    ↓
Code node (regex: /^\[(.+?)\]/)
    ↓
Project_Mapping (Google Sheet: Project Code → Folder Name)
    ↓
Google Drive (فولدر المشروع)
    ↓
Project_Bible.md + Sales_Playbook.md
    ↓
OpenAI node context
```

**قاعدة صارمة:** أي إعلان جديد بدون `Project Code` في بداية الـ (`headline`) لا يمكن لـ `MAYA` تحديد المشروع الخاص به بشكل موثوق، ولازم يتصحح قبل التشغيل الفعلي.

---

Ship first.

Improve continuously.

Never over-engineer before the business validates the need.
