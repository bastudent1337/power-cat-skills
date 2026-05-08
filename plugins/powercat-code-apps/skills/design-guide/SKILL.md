---
name: code-apps-design-guide
version: 1.0.0
description: "Generate a brand-aligned design guide for a new code app instead of using the default template styling. Use when: the user wants to start a new app with their own brand identity, mentions 'design guide', 'brand guidelines', 'style guide', or wants to customize the look and feel before building. This skill runs BEFORE the code-apps plugin to establish brand tokens, then delegates app creation to /create-code-app."
user-invocable: true
allowed-tools: Read, Write, Edit, Bash, Glob, Grep, WebFetch, Agent, TaskCreate, TaskUpdate, TaskList, Skill
---

# Design Guide Generator

$ARGUMENTS

## Purpose

Create a comprehensive design guide tailored to the customer's brand, then hand off to the **code-apps plugin** (`/create-code-app`) to scaffold and build the app. This skill does NOT replicate scaffolding, deployment, or data source wiring — it focuses exclusively on brand extraction and design token generation, then delegates to the plugin for everything else.

## Prerequisites

The **code-apps plugin** must be installed. If not already available, instruct the user:

```
/plugin marketplace add microsoft/power-platform-skills
/plugin install code-apps@power-platform-skills
```

## Workflow

### Step 0: Verify Plugin Installation

This step is only needed when the user wants to create a new app. If the user only wants the design guide and tokens for an existing project, skip to Step 1.

Run each substep in order:

**0a.** Run `/plugin marketplace list`. If `microsoft/power-platform-skills` is missing, ask: *"The Power Platform Skills marketplace isn't registered. Want me to add it?"* If yes, run `/plugin marketplace add microsoft/power-platform-skills`. If no, stop and offer to generate just the design guide without app creation.

**0b.** Run `/plugin list`. If `code-apps` is missing, ask: *"The code-apps plugin isn't installed. Want me to install it?"* If yes, run `/plugin install code-apps@power-platform-skills`. If no, stop and offer to generate just the design guide without app creation.

**0c.** Confirm `/create-code-app` resolves. If it does not, ask: *"The plugin seems broken. Want me to reinstall it?"* If yes, run `/plugin uninstall code-apps` then `/plugin install code-apps@power-platform-skills`.

After all substeps pass, proceed to Step 1.

### Step 1: Gather Brand Inputs

Use the ask-questions tool to collect brand information from the customer:

1. **Brand name** — What is the brand or product name?
2. **Brand assets** — Ask the customer to provide one or more of the following:
   - A link to an existing CSS stylesheet (e.g. a CDN-hosted brand stylesheet)
   - A PDF or Word document containing brand guidelines
   - A link to an existing website whose styling should be referenced
   - A logo file or link to a logo
   - Any specific colours, fonts, or design preferences described in text
3. **Platform targets** — Is this for web, mobile, or both? Are there specific responsive breakpoints or information architecture patterns they need?

### Step 2: Analyse Brand Assets

Based on what the customer provides:

#### If a CSS stylesheet link is provided:
- Fetch the stylesheet using the fetch_webpage tool
- Extract colour variables (custom properties, hex values, rgb/hsl values)
- Extract typography (font families, sizes, weights, line heights)
- Extract spacing scales, border radii, shadows
- Note any component-level patterns (buttons, cards, navigation)

#### If a PDF or document is provided:
- Read the document to extract:
  - **Colour palette** — Primary, secondary, accent, neutral colours with exact values
  - **Typography** — Font families, type scale, heading hierarchy, body text specs
  - **Logo usage** — Clear space rules, minimum sizes, colour variants, placement guidelines
  - **Imagery & iconography** — Style, tone, preferred icon sets
  - **Layout & grid** — Column systems, margins, gutters, breakpoints
  - **Information architecture** — Navigation patterns, page hierarchy, mobile vs desktop behaviour
  - **Tone & voice** — If described, note for UI copy guidance

#### If a website link is provided:
- Fetch the page and inspect the rendered styles
- Extract the dominant colour palette
- Identify fonts in use
- Note layout patterns and component styles

### Step 3: Generate the Design Guide

Produce a `design-guide.md` file in the working directory. Build it in four focused substeps:

**3a. Colour Palette** — Write the colour sections first:

```markdown
# [Brand Name] Design Guide

## Colour Palette

### Primary Colours
| Name | Hex | RGB | Usage |
|------|-----|-----|-------|

### Secondary / Accent Colours
| Name | Hex | RGB | Usage |
|------|-----|-----|-------|

### Neutral / Greyscale
| Name | Hex | RGB | Usage |
|------|-----|-----|-------|

### Semantic Colours
| Name | Hex | Usage |
|------|-----|-------|
| Success | ... | Positive actions, confirmations |
| Warning | ... | Caution states |
| Error | ... | Destructive actions, errors |
| Info | ... | Informational notices |
```

**3b. Typography & Logo** — Append the type and logo sections:

```markdown
## Typography

### Font Families
- **Headings:** [Font name] — [source/link]
- **Body:** [Font name] — [source/link]
- **Monospace:** [Font name] — [source/link]

### Type Scale
| Level | Size | Weight | Line Height | Usage |
|-------|------|--------|-------------|-------|
| H1 | ... | ... | ... | Page titles |
| H2 | ... | ... | ... | Section headings |
| H3 | ... | ... | ... | Subsection headings |
| Body | ... | ... | ... | Default text |
| Small | ... | ... | ... | Captions, labels |

## Logo Usage
- Minimum clear space: [value]
- Minimum size: [value]
- Colour variants: Full colour / Monochrome / Reversed
- Placement: [guidelines]
```

**3c. Spacing, Layout & Breakpoints** — Append the layout sections:

```markdown
## Spacing & Layout

### Spacing Scale
| Token | Value | Usage |
|-------|-------|-------|

### Grid System
- Columns: [count]
- Gutter: [value]
- Max content width: [value]

### Breakpoints
| Name | Min Width | Behaviour |
|------|-----------|-----------|
| Mobile | 0 | Single column, stacked navigation |
| Tablet | ... | ... |
| Desktop | ... | ... |
| Wide | ... | ... |
```

**3d. Components & Information Architecture** — Append the component and IA sections:

```markdown
## Components

### Buttons
- Border radius, padding, states (default, hover, active, disabled, focus)

### Cards
- Border radius, shadow, padding

### Navigation
- Pattern: [top bar / sidebar / bottom tabs / hamburger]
- Mobile behaviour: [description]

## Information Architecture
- Navigation depth: [flat / 2 levels / deep hierarchy]
- Mobile navigation: [bottom tabs / hamburger / drawer]
- Key page templates: [list]
```

### Step 4: Generate Design Tokens File

Create a `design-tokens.css` file with CSS custom properties extracted from the brand analysis:

```css
:root {
  /* Colours */
  --color-primary: ...;
  --color-primary-light: ...;
  --color-primary-dark: ...;
  --color-secondary: ...;
  --color-accent: ...;
  --color-background: ...;
  --color-surface: ...;
  --color-text: ...;
  --color-text-muted: ...;
  --color-success: ...;
  --color-warning: ...;
  --color-error: ...;
  --color-info: ...;

  /* Typography */
  --font-heading: ...;
  --font-body: ...;
  --font-mono: ...;
  --text-xs: ...;
  --text-sm: ...;
  --text-base: ...;
  --text-lg: ...;
  --text-xl: ...;
  --text-2xl: ...;
  --text-3xl: ...;

  /* Spacing */
  --space-xs: ...;
  --space-sm: ...;
  --space-md: ...;
  --space-lg: ...;
  --space-xl: ...;
  --space-2xl: ...;

  /* Radii */
  --radius-sm: ...;
  --radius-md: ...;
  --radius-lg: ...;
  --radius-full: 9999px;

  /* Shadows */
  --shadow-sm: ...;
  --shadow-md: ...;
  --shadow-lg: ...;
}
```

### Step 5: Confirm with the Customer

Present the design guide summary and ask:
- Does the colour palette look correct?
- Are the fonts right?
- Any adjustments to spacing, sizing, or component styles?
- Should any sections be expanded or removed?

Iterate until the customer is satisfied with the design tokens.

### Step 6: Delegate to the Code Apps Plugin

Once the design guide is finalised, invoke the code-apps plugin to scaffold and build the app:

1. **Invoke `/create-code-app`** — Let the plugin handle:
   - Prerequisites check (Node.js, Git)
   - Requirements gathering (what the app does, data sources)
   - Scaffolding via `npx degit`
   - `npx power-apps init`
   - Baseline build & deploy
   - Data source wiring (`/add-dataverse`, `/add-sharepoint`, etc.)

2. **Pass brand context** — When `/create-code-app` reaches its "Implement App" step (Step 8), provide:
   - The `design-tokens.css` file to import into the app's `src/` directory
   - The design guide as reference for component styling decisions
   - Theme preference derived from the brand (light/dark/auto)
   - Any information architecture requirements (nav patterns, breakpoints, mobile behaviour)

### Step 7: Apply Brand Styling Post-Scaffold

After the plugin scaffolds the app, apply the brand layer:

1. **Copy `design-guide.md`** into the project root so it lives alongside the code
2. **Copy `design-tokens.css`** into `src/styles/design-tokens.css`
3. **Import tokens** in the app's main CSS/entry point:
   ```css
   @import './styles/design-tokens.css';
   ```
4. **Update `index.html`** — Add any external font links (Google Fonts, Adobe Fonts, etc.)
5. **Override template defaults** — Replace the generic theme in `src/App.css` or equivalent with brand colours, typography, and spacing from the tokens
6. **Apply component patterns** — If the design guide specifies button styles, card styles, or navigation patterns, update the template's base components accordingly
7. **Create `.github/copilot-instructions.md`** (or append to it if it exists) with the following content so that all future Copilot interactions reference the design guide:

   ```markdown
   ## Design Guide

   This project has a brand design guide at `design-guide.md` in the project root.

   When creating or modifying UI components, always:
   - Refer to `design-guide.md` for colour palette, typography, spacing, component patterns, and information architecture rules
   - Use the CSS custom properties defined in `src/styles/design-tokens.css` — never hardcode colour values, font stacks, or spacing
   - Follow the breakpoints and responsive behaviour described in the design guide
   - Respect logo usage rules (clear space, minimum size, colour variants)
   - Match the navigation pattern and information architecture specified in the guide
   - Maintain WCAG accessibility contrast ratios documented in the colour palette
   ```

### Step 8: Verify Build

Run `npm run build` to confirm the brand styling doesn't break the app. Fix any issues.

Then let the code-apps plugin continue with its normal deployment flow (`npx power-apps push`).

## Integration Notes

- This skill is a **pre-step** to `/create-code-app` — it runs first, then delegates
- The `design-guide.md` file MUST live in the project root and be referenced by `.github/copilot-instructions.md` so that all future Copilot sessions (new features, refactors, component additions) automatically consult the brand guide
- The design tokens file (`src/styles/design-tokens.css`) is the runtime contract — components use these variables, never raw values
- Do NOT duplicate what the code-apps plugin already does (scaffolding, init, deploy, data sources)
- If the user already has a scaffolded app, skip Step 6 and go straight to Step 7 (apply tokens to existing project)
- The code-apps plugin handles all Power Platform concerns (connectors, generated services, `npx power-apps` CLI)
- If the user updates their brand guidelines later, re-run Steps 2–5 to regenerate tokens, then update the files in the project — the copilot-instructions reference ensures future work stays aligned

## Notes

- Always prefer exact colour values from brand documents over approximations
- If the brand document specifies accessibility requirements (WCAG levels), include contrast ratios in the colour table
- If no brand assets are provided, ask the customer to describe their brand in terms of adjectives (modern, playful, corporate, minimal, etc.) and generate a suggested palette
- Keep the design guide as a living document — it can be updated as the project evolves
- The design tokens should use CSS custom properties so they work regardless of the CSS framework the app uses
