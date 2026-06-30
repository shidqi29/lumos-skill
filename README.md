# Lumos Skill for Claude Code

A Claude Code skill that teaches Claude to build responsive layouts using the
**[Lumos Framework](https://lumos.timothyricks.com/)** by Timothy Ricks — either
inside an existing **Webflow** project _or_ in a **plain HTML/CSS/JS** project
built from scratch.

## What is this?

Lumos is a breakpointless, utility-driven approach to building responsive websites.
Instead of hardcoded pixel values it leans on fluid sizing, container queries,
utility classes (`u-*`), and a strict component naming convention. (This skill keeps
the breakpointless `u-grid-*` grid but routes all other responsiveness through
Webflow's native breakpoints — see the [changelog](CHANGELOG.md) for the rationale.)

This skill packages all of those conventions so Claude follows them automatically.
When it's active, any layout Claude generates will:

- Use Lumos utility + component class naming (`hero_title u-text-style-h2`)
- Stay **breakpointless for the grid** (`u-grid-*` via container queries) and use Webflow's native breakpoints (`@media` at 991 / 767 / 479px) for every other element, so it imports cleanly into the Designer
- Use design tokens for color, spacing, and typography (no hex codes, no raw `px`)
- Follow the trigger/state system for hover and interactive states (no `:hover` in CSS)
- Stay accessible (ARIA roles, keyboard nav, reduced-motion support)

## Who is it for?

- **Webflow developers** using the Lumos Framework who want Claude to generate
  paste-ready markup that matches their existing project conventions.
- **Anyone building a vanilla site** (landing page, static site, component) who
  wants Lumos conventions without Webflow — the skill ships a foundation
  stylesheet it drops into your project to bootstrap everything.

## Two modes

The skill auto-detects which mode applies:

| Mode | When | What happens |
|------|------|--------------|
| **Webflow / existing Lumos** | You're in a Webflow site or paste Lumos markup to extend | Generates components only — never redefines utilities, variables, or resets (they already exist in your project) |
| **Vanilla / from scratch** | You ask for plain HTML/CSS/JS, a landing page, or a layout "without Webflow" | Generates the [foundation stylesheet](plugins/lumos/skills/lumos-skill/assets/lumos-foundation.css) first (reset, tokens, utilities, responsive system), then builds components on top |

## Installation

Add the marketplace, then install the plugin:

```bash
/plugin marketplace add shidqi29/lumos-skill
/plugin install lumos@lumos-skill-by-shidqi
```

Or test it locally from a clone of this repo:

```bash
claude                                     # start Claude Code in this folder
/plugin marketplace add .                  # register this directory as a marketplace
/plugin install lumos@lumos-skill-by-shidqi
/plugins                                    # confirm it loaded
```

## How to use it

Once installed, just describe what you want — the skill triggers automatically when
your request involves Lumos, Webflow layouts, responsive sections, utility classes,
or building a page from scratch. For example:

- _"Build a Lumos hero section with a heading, paragraph, and two buttons"_
- _"Make a responsive pricing grid from scratch in HTML/CSS/JS using Lumos conventions"_
- _"Add a tabs component to this Webflow project"_

You can also invoke it explicitly:

```
/lumos:lumos-skill
```

When building a full vanilla page, the skill also references
[vanilla-mode.md](plugins/lumos/skills/lumos-skill/references/vanilla-mode.md) for
the project skeleton, HTML boilerplate, and how to customize the design tokens
(brand colors, spacing, type scale, fonts) in section 2 of the foundation file.

## What's inside

```
lumos-skill/
├── .claude-plugin/
│   └── marketplace.json              # marketplace catalog (required — do not git-ignore)
├── plugins/
│   └── lumos/
│       ├── .claude-plugin/
│       │   └── plugin.json           # plugin manifest (required — do not git-ignore)
│       └── skills/
│           └── lumos-skill/          # the skill itself
│               ├── SKILL.md          # all the Lumos rules & conventions
│               ├── assets/lumos-foundation.css   # generatable foundation for vanilla mode
│               └── references/
│                   ├── vanilla-mode.md            # full-page guide for from-scratch builds
│                   └── webflow-variable-naming.md # naming so vanilla CSS imports as Webflow Variables
├── .gitignore
├── CHANGELOG.md                         # dated history of skill changes (newest first)
└── README.md
```

> **Note on `.claude-plugin/`** — despite the leading dot, these folders hold the
> required manifests and **must be committed**. Git-ignoring them breaks installation
> from GitHub. The `.gitignore` only excludes OS/editor junk.

## Updating

Bump the `version` fields in [marketplace.json](.claude-plugin/marketplace.json) and
[plugin.json](plugins/lumos/.claude-plugin/plugin.json), push to GitHub, and users
refresh with:

```bash
/plugin marketplace update lumos-skill-by-shidqi
```

## License

MIT
