# AGENTS.md — PowerCAT-Overflow Plugin

This file provides guidance to AI Agents when working with the **powercat-overflow** plugin.

## What This Plugin Is

A plugin for Power Platform makers, ALM owners, and CoE reviewers that audits **every Power Automate cloud flow inside a Power Platform solution `.zip`** against Microsoft's published coding guidelines. The skill unpacks the solution, walks each flow, classifies findings by impact (low / medium / high), writes a single `[SolutionName].findings.json` next to the uploaded ZIP, and opens the **hosted PowerCAT-Overflow viewer** (<https://microsoft.github.io/power-cat-skills/PowerCAT-Overflow.html>) with both the solution and findings files pre-loaded so the user can explore the results visually.

All citations in findings must come from the authoritative source list at `Common/PowerCAT OverFlow/sources.md` in this repo. Findings that cannot anchor to that list are not emitted.

## Local Development

Test this plugin locally:

```bash
claude --plugin-dir /path/to/plugins/powercat-overflow
```

## Architecture

```
.claude-plugin/plugin.json                     ← Plugin metadata (name, version, keywords)
AGENTS.md                                      ← Plugin guidance for AI agents (this file)
CLAUDE.md                                      ← Pointer → AGENTS.md
README.md                                      ← User-facing plugin docs
skills/
  powercat-overflow/
    SKILL.md                                   ← Solution-level cloud-flow review skill
```

External assets consumed at run time (not bundled — fetched live from this repo):

- `Common/PowerCAT OverFlow/sources.md` — authoritative citation list (hard-required)
- `Common/PowerCAT OverFlow/solution.findings.schema.json` — output schema (validation)
- `Common/PowerCAT OverFlow/StressTest100Flows_1_0_0_0.zip` — sample solution used in the README "Explore it now" walkthrough
- `https://microsoft.github.io/power-cat-skills/PowerCAT-Overflow.html` — hosted viewer (GitHub Pages)

## Skills

| Skill | Description |
|-------|-------------|
| `/powercat-overflow` | Reviews every Power Automate cloud flow inside a Power Platform solution `.zip`, writes a single solution-level findings JSON, then opens the hosted viewer with both files loaded. |

### `/powercat-overflow` — Solution-level Cloud-Flow Review

**Triggers:** `powercat overflow`, `overflow my solution`, `review my solution`, `evaluate flows in this solution`, `audit Power Automate solution`, `check this solution against guidelines`

**What it does:**

1. Fetches the canonical source list (`Common/PowerCAT OverFlow/sources.md`) and caches it for the run. Hard-fails if unreachable — no citations means no findings.
2. Locates the user's solution `.zip` (preferring tagged files; otherwise prompts via `m_ask_user`).
3. Unpacks the ZIP, reads `solution.xml` (display name, unique name, version) and enumerates every `Workflows/*.json` file.
4. For each flow, walks every action recursively (`actions`, `else.actions`, `cases.*.actions`, `default.actions`) and emits findings classified as **low**, **medium**, or **high** across: Complexity, Maintainability, Security, Performance, Reliability, Observability, Testability, and Naming.
5. Validates the aggregated output against `solution.findings.schema.json` (also fetched live), then writes `[SolutionName].findings.json` next to the input ZIP.
6. Opens the hosted viewer via Playwright and uploads both the `.zip` and the `.findings.json` so the user can browse findings overlaid on the visual flow diagram.

**Safety guardrails:**

- The skill **never** fabricates citation URLs — every `source` field must match an entry from the Step 0 source list.
- The skill writes exactly one output file per run (`[SolutionName].findings.json`) and never modifies the input solution.
- Hosted-viewer interaction is read-only — Playwright uploads files but does not submit, share, or persist anything server-side.

**Requires:**

- Host capability to run Playwright (the skill drives the hosted viewer in a real browser).
- `web_fetch` for retrieving the source list and schema from this repo.
- File-system access to unpack the ZIP into a temp working directory.

## Prerequisites

- A Power Platform solution `.zip` containing a `Workflows/` folder with one or more cloud flow JSON files.
- Network access to `microsoft.github.io` and `raw.githubusercontent.com` for the source list, schema, and hosted viewer.

## Why this plugin exists

ALM-owned Power Automate solutions accumulate technical debt fast: hardcoded URLs, magic numbers, missing `secureData` flags, deep nested Conditions, sibling Scopes that should be child flows, duplicate `Copyof-…` flows shipped alongside originals. Reviewers had no fast way to scan an entire solution end-to-end without opening each flow by hand in the maker portal. `powercat-overflow` automates that scan, anchors every finding to Microsoft's published guidance, and renders the results on top of the actual flow diagram so the reviewer can act in minutes instead of hours.
