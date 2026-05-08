# Power CAT Code Apps Plugin

This plugin provides open‑source skills based on years of Power CAT experience working with large and complex enterprise customers on Microsoft Power Platform. The skills capture practical brand and design patterns to help teams build visually consistent code apps on Power Platform.

> **Preview:** This plugin is currently in [preview](https://www.microsoft.com/en-us/business-applications/legal/supp-powerplatform-preview/). These features are available before official release for customers to provide feedback.

## Prerequisites

- [code-apps@power-platform-skills](https://github.com/microsoft/power-platform-skills)

## Installation

### From the marketplace

```bash
/plugin marketplace add microsoft/power-cat-skills
/plugin install powercat-code-apps@power-cat-skills
```

### From a local clone

```bash
claude --plugin-dir /path/to/power-cat-skills/plugins/powercat-code-apps
```

## Skills

### `/code-apps-design-guide`

Generate a brand-aligned design guide for a new code app instead of using the default template styling. USE WHEN the user wants to start a new app with their own brand identity, mentions 'design guide', 'brand guidelines', 'style guide', or wants to customize the look and feel before building. This skill runs BEFORE the code-apps plugin to establish brand tokens, then delegates app creation to `/create-code-app`.

**Usage:** Invoke directly with `/code-apps-design-guide`, or use any of the keywords below to trigger the skill automatically:

- `Create a design guide`
- `Generate brand tokens`
- `Match our brand styles`
- `Use our brand guidelines`

**What it does:**
1. Verifies the code-apps plugin is installed (and offers to install it if not)
2. Gathers brand inputs — name, assets (CSS, PDF, website, logo), and platform targets
3. Analyses provided assets to extract colours, typography, spacing, and layout patterns
4. Produces a `design-guide.md` with full design token tables and component specifications
5. Hands off to `/create-code-app` with the design guide as context
