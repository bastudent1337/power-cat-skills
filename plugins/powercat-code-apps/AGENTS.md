# AGENTS.md — Code Apps Plugin

This file provides guidance to AI Agents when working with the **powercat-code-apps** plugin.

## What This Plugin Is

A plugin for generating brand-aligned design guides for Power Apps code apps. It extracts brand tokens (colours, typography, spacing, layout patterns) from customer-provided assets and produces a `design-guide.md` that is consumed by the **code-apps plugin** (`/create-code-app`) to scaffold and build the app.

Skills orchestrate specialist agents via the `Task` tool. Agents are not invoked directly by users.

## Local Development

Test this plugin locally:

```bash
claude --plugin-dir /path/to/plugins/powercat-code-apps
```

## Architecture

```
.claude-plugin/plugin.json     ← Plugin metadata (name, version, keywords)
AGENTS.md                      ← Plugin guidance for AI agents (this file)
CLAUDE.md                      ← Symlink → AGENTS.md
skills/
  design-guide/
    SKILL.md                   ← Generate a brand-aligned design guide before scaffolding a code app
```

## Skills

| Skill | Description |
|-------|-------------|
| `/code-apps-design-guide` | Generate a brand-aligned design guide from customer assets, then hand off to `/create-code-app` |

## Prerequisites

The **code-apps plugin** from the official marketplace must be installed for end-to-end app creation:

```bash
/plugin marketplace add microsoft/power-platform-skills
/plugin install code-apps@power-platform-skills
```
