---
name: "powercat-admin-digest"
description: "Produce a Power Platform admin digest of items that can affect production operations — Message Center notices, Service Health incidents, Known Issues, and deprecations — synthesized from the user's mailbox and public Microsoft sources. Categorizes findings into shareability tiers (Public / Tenant-restricted / Microsoft-internal) and outputs a concise customer-shareable summary by default. Triggers: 'admin digest', 'power platform digest', 'what's affecting production', 'production-impact summary"
---

# powercat-admin-digest

Produce a **Power Platform production-impact digest** for an admin or partner audience. Pulls from the user's M365 mailbox (Message Center notices, partner threads, support escalations, MVP/NDA discussions) and from public Microsoft sources, then filters and categorizes by shareability tier.

## When to invoke

User asks things like:
- "What's affecting Power Platform production right now?"
- "Give me an admin digest"
- "Summarize Message Center / Service Health for Power Platform"
- "What outages or upcoming changes should I know about?"

## Inputs (ask only if missing)

- **Lookback window** — default: last 30 days. Accept "last 7 days", "last quarter", explicit dates.
- **Audience** — default: customer-shareable (Tier 1 + Tier 2). Other options: "internal" (all tiers, including Tier 3) or "public-only" (Tier 1 only).
- **Scope** — default: all Power Platform (Power Apps, Power Automate, Dataverse, Copilot Studio, Power Pages, CI-J). Accept narrower scope if requested.

Do not ask about format up front. Default to a concise chat response (tables only). Offer .docx / .pptx export at the end.

## Gathering steps (in parallel where possible)

1. **Mailbox** — call `m365_search_emails` with `startDate` = lookback start, multiple queries in parallel:
   - `"Message Center Power Platform"`
   - `"service health Power Apps Power Automate Dataverse incident"`
   - `"deprecation Power Platform breaking change"`
   - `"MC1" OR "MC2"` (catches MC#### references)
   - `"known issue" Power Platform`
2. **Read top hits** with `m365_get_email` using `bodyContentType: "text"`. Batch up to 4–6 in parallel; if Graph throttles (429), back off and retry sequentially.
3. **Extract** for each item: title, MC# / KI# / ICM#, effective date, scope, customer impact, action required, workaround if any, and the *source* (which mailbox thread or public URL).
4. (Optional, only if user asks for "latest public" view) fetch the Microsoft Learn deprecations page via `web_fetch`: `https://learn.microsoft.com/power-platform/important-changes-coming`.

## Filtering — production-impact only

KEEP an item if it affects production operations in any of these ways:
- Changes a default that ran for years (retention, storage tiering, IPs, auth)
- Retires or deprecates a feature with a hard date
- Active regression / known issue currently hitting customers
- Resolved incident that may still require follow-up (e.g., RCA, data integrity check)

DROP items that are:
- Pure feature announcements with no breaking change
- Marketing / event recaps
- Internal team logistics
- Already past their cutover with no remediation needed

## Tier classification (CRITICAL — drives shareability)

Apply this tagging to every kept item:

| Tier | Definition | Examples |
|------|-----------|----------|
| **Tier 1 — Public** | On the open web. Anyone can read. | `learn.microsoft.com` pages, public KI portal entries (`admin.powerplatform.microsoft.com/support/knownissues/<id>`), Release Planner, `aka.ms` links, public roadmap |
| **Tier 2 — Tenant-restricted** | Visible to tenant admins via Admin Center, not on the open web. | Message Center notices (MC####), Service Health incidents, PPAC notifications |
| **Tier 3 — Microsoft-internal / NDA** | Only Microsoft employees or NDA partners. **Never put in customer-facing output.** | ICM #, LSI #, Iridias links, internal v-team emails, MVP NDA threads, internal Kusto queries, FCB flag mitigations, internal customer escalation TrackingIDs, named customer engagement details |

**Default output is Tier 1 + Tier 2 only.** Tier 3 items are summarized internally for the user's own awareness but MUST NOT appear in shareable output. If user asks for "internal" audience, include Tier 3 in a separate clearly-labeled section.

## Output format

Default response = **tables only**, three sections, concise:

### Active changes — review and act
| # | Change | What it means | Reference |

### Active known issues
| # | Issue | Status | Workaround |

### Recently resolved (Service Health)
| # | Incident | When |

Rules for the tables:
- Reference column: MC#### · public `aka.ms/...` or `learn.microsoft.com/...` link (no Iridias, no ICM).
- Workaround column: only customer-applicable steps. Don't paste FCB flags or internal triage steps.
- Keep rows short — one line each where possible. Bold the change name.
- If a section is empty, omit it entirely.

After the tables, in one sentence, name the **top 1–2 items to action today** and offer `.docx` / `.pptx` export.

## Privacy guardrails

- Never include named customer accounts, TrackingIDs, or escalation specifics in shareable output.
- Never include ICM / LSI / Iridias / PR numbers in shareable output.
- Never include FCB flag commands in shareable output.
- If the session contains MIP-classified content, warn the user before producing any exported file.
- If the user asks you to *send* the digest to a third party (email, Teams), preview the exact content and recipient list first and require explicit confirmation, per standard outbound rules.

## Variants the user may ask for

- **"public-only"** → emit Tier 1 rows only; replace MC#### references with their public Learn equivalents where available.
- **"internal"** → add a fourth section "Microsoft-internal context" with Tier 3 items, clearly marked "Do not forward externally."
- **".docx" / ".pptx" export** → invoke the matching skill (`docx` / `pptx`) and re-render the tables there. Strip any Tier 3 content even if the user has it; ask before including Tier 3.

## Worked example (shape only — do not copy verbatim)

```
## Active changes — review and act
| # | Change | What it means | Reference |
| 1 | **Backup retention 28 → 7 days** (May 11, 2026) | Default restore window shrunk... | MC1298714 · aka.ms/14441 |
...
```

