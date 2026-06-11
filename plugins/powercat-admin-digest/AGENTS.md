# AGENTS.md — Admin Digest Plugin

This file provides guidance to AI Agents when working with the **powercat-admin-digest** plugin.

## What This Plugin Is

A plugin for Power Platform admins and partner SAs that produces a **production-impact digest** by synthesizing Message Center notices, Service Health incidents, Known Issues, and deprecations from the user's M365 mailbox and public Microsoft sources. Default output is **customer-shareable** (Tier 1 + Tier 2 only); Tier 3 (NDA / Microsoft-internal) content is stripped unless the user explicitly asks for the `internal` variant.

Skills delegate `.docx` / `.pptx` export to the matching first-party skills (`docx` / `pptx`).

## Local Development

Test this plugin locally:

```bash
claude --plugin-dir /path/to/plugins/powercat-admin-digest
```

## Architecture

```
.claude-plugin/plugin.json     ← Plugin metadata (name, version, keywords)
AGENTS.md                      ← Plugin guidance for AI agents (this file)
CLAUDE.md                      ← Symlink → AGENTS.md
README.md
skills/
  powercat-admin-digest/
    powercat-admin-digest.md   ← Production-impact digest with shareability tiering
```

## Skills

| Skill | Description |
|-------|-------------|
| `/powercat-admin-digest` | Power Platform production-impact digest (Message Center, Service Health, KIs, deprecations) with Public / Tenant-restricted / Microsoft-internal tiering |

### `/powercat-admin-digest` — Power Platform Admin Digest

Pulls findings from the user's mailbox and public Microsoft sources, filters to production-impact items, tags each by shareability tier, and renders three concise tables.

**Triggers:** `admin digest`, `power platform digest`, `what's affecting production`, `production-impact summary`, `message center summary`, `service health summary`

**What it does:**

1. Searches the mailbox (default 30-day lookback, configurable) using `m365_search_emails` for Message Center notices, Service Health threads, deprecations, and Known Issues across Power Apps, Power Automate, Dataverse, Copilot Studio, Power Pages, and Customer Insights – Journeys.
2. Reads top hits with `m365_get_email` (text body); batches up to 4–6 in parallel and backs off serially on Graph 429 throttling.
3. Filters to production-impact items (drops feature announcements, marketing recaps, internal logistics).
4. Tags each item Tier 1 / Tier 2 / Tier 3.
5. Renders tables-only chat output: *Active changes — review and act*, *Active known issues*, *Recently resolved (Service Health)*.
6. Names the top 1–2 items to action today and offers `.docx` / `.pptx` export.

**Variants:**

- `internal` — adds Tier 3 section, clearly labelled "Do not forward externally"
- `public-only` — Tier 1 rows only
- `.docx` / `.pptx` — re-renders tables via the `docx` / `pptx` skill

**Safety guardrails:**

- Never includes ICM / LSI / internal-portal links, customer escalation TrackingIDs, or internal feature-control flags in default (customer-shareable) output.
- Outbound sends (email / Teams) require explicit recipient + content preview before transmission.

**Requires:**

- Host capability to call `m365_search_emails` and `m365_get_email` (e.g. Clawpilot's M365 toolset).
- Optional `web_fetch` for enriching deprecation entries from Microsoft Learn.

## Prerequisites

- Microsoft 365 mailbox access via the host's M365 tooling.

## Why this plugin exists

Tenant admins routinely need to brief customers on "what changed and what might break" without leaking NDA content from internal escalation threads. This skill does that filtering automatically, so the default chat output is forwardable as-is.
