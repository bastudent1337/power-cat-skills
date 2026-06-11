# Power CAT Admin Digest Plugin

This plugin produces a **Power Platform production-impact digest** for tenant admins and partner SAs. It synthesizes Message Center notices, Service Health incidents, Known Issues, and deprecation announcements from the user's M365 mailbox and public Microsoft sources, then filters and **tiers** the findings so the default chat output is safe to forward to a customer.

> **Preview:** This plugin is in [preview](https://www.microsoft.com/en-us/business-applications/legal/supp-powerplatform-preview/). These features are available before official release for customers to provide feedback.

## Prerequisites

- Microsoft 365 mailbox access (the host environment must expose `m365_search_emails` / `m365_get_email` tools, e.g. Clawpilot)
- Optional: web fetch capability for enriching deprecation entries from [Microsoft Learn](https://learn.microsoft.com/power-platform/important-changes-coming)

## Installation

### From the marketplace

```bash
/plugin marketplace add microsoft/power-cat-skills
/plugin install powercat-admin-digest@power-cat-skills
```

### From a local clone

```bash
claude --plugin-dir /path/to/power-cat-skills/plugins/powercat-admin-digest
```

## Skills

### `/powercat-admin-digest` — Power Platform Admin Digest

USE WHEN the user asks "what's affecting Power Platform production right now?", wants an admin digest, or asks for a summary of Message Center / Service Health for Power Platform.

**Usage:** Invoke directly with `/powercat-admin-digest`, or use any of the phrases below to trigger the skill automatically:

- `Give me a power platform admin digest`
- `What's affecting production?`
- `Summarize Message Center for Power Platform`
- `Production-impact summary`

**What it does:**

1. Searches the user's mailbox (default 30-day lookback) for Message Center notices, Service Health threads, deprecation announcements, and Known Issues across Power Apps, Power Automate, Dataverse, Copilot Studio, Power Pages, and Customer Insights – Journeys.
2. Filters to items that genuinely affect production operations (drops feature announcements, marketing recaps, internal logistics).
3. Tags every finding by shareability tier:
   - **Tier 1 — Public** (Microsoft Learn, public KI portal, `aka.ms` links)
   - **Tier 2 — Tenant-restricted** (Message Center, Service Health)
   - **Tier 3 — Microsoft-internal / NDA** (always stripped from the default output)
4. Renders three concise tables: *Active changes*, *Active known issues*, *Recently resolved*.
5. Highlights the top 1–2 items to action today and offers `.docx` / `.pptx` export.

**Variants:**

- `internal` — adds a clearly-labelled Tier 3 section (do-not-forward).
- `public-only` — emits Tier 1 rows only, replacing MC#### with the public Learn equivalent where available.
- `.docx` / `.pptx` — re-renders the tables via the matching skill.

**Privacy guardrails baked in:**

- ICM / LSI / internal triage references never appear in customer-shareable output.
- Customer escalation TrackingIDs, named engagement details, and internal feature-control flags are filtered.
- Outbound sends (email / Teams) require explicit recipient + content preview before transmission.

## Support

If you face issues with:

- **Using this skill:** Report your issue here: [https://github.com/microsoft/power-cat-skills/issues](https://github.com/microsoft/power-cat-skills/issues). (Microsoft Support won't help with issues related to this plugin, but they will help with related, underlying platform and feature issues.)
- **The core features in Microsoft Power Platform:** Use your standard channel to contact Microsoft Support.

## License

See the [LICENSE](../../LICENSE) file for license information.
