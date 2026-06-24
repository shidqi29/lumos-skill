# Lumos Marketplace

A Claude Code plugin marketplace for the **Lumos Framework** skill — responsive layouts in Webflow _or_ from-scratch vanilla HTML/CSS/JS.

## What's inside

```
lumos-marketplace/
├── .claude-plugin/
│   └── marketplace.json          # marketplace catalog (lists the plugins)
├── plugins/
│   └── lumos/
│       ├── .claude-plugin/
│       │   └── plugin.json        # plugin manifest
│       └── skills/
│           └── lumos-skill/       # the skill itself
│               ├── SKILL.md
│               ├── assets/lumos-foundation.css
│               └── references/vanilla-mode.md
└── README.md
```

## Before you publish

Edit the placeholders in **both** `.claude-plugin/marketplace.json` and `plugins/lumos/.claude-plugin/plugin.json`:

- `owner` / `author` → your name, email, GitHub URL
- `repository` / `homepage` → your repo URL

The `name` field in `marketplace.json` (`lumos-marketplace`) is what users type after the `@` when installing.

## Test locally (no GitHub needed)

From inside this folder:

```bash
claude                                  # start Claude Code here
/plugin marketplace add .               # register this directory as a marketplace
/plugin install lumos@lumos-marketplace # install the plugin
/plugins                                # confirm it loaded
```

## Publish on GitHub

```bash
git init
git add .
git commit -m "feat: lumos plugin marketplace"
git branch -M main
git remote add origin https://github.com/your-username/lumos-marketplace.git
git push -u origin main
```

Then anyone (including you, on any machine/project) installs with:

```bash
/plugin marketplace add your-username/lumos-marketplace
/plugin install lumos@lumos-marketplace
```

## Updating

Push changes to the repo, bump the `version` fields, and users refresh with:

```bash
/plugin marketplace update lumos-marketplace
```

## Using the skill

Once installed, just describe the task (e.g. "build a Lumos landing page from scratch in HTML/CSS/JS") and Claude triggers it automatically, or invoke explicitly with `/lumos:lumos-skill`.
