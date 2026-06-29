# Webflow Variable Naming (vanilla → Webflow import)

When a vanilla Lumos project is imported into Webflow with the HTML-to-Webflow tool, every CSS custom property whose **name** follows this convention is injected as a **native Webflow Variable** — with the right collection, folder, variable, and mode. Names that don't follow it import as plain CSS (no variable created).

`lumos-foundation.css` already conforms throughout. **Keep the names intact** when customizing — change values, not names — and use the same convention for any new token you introduce.

## Name → location (collection / folder / variable)

```
--_<collection>---<folder>--<...>--<name>: <value>;
   │            │     └── '--' = nested folder (becomes '/')
   │            └── '---' = collection → variable name
   └── leading '_' = start a collection   |   no '_' = base collection
'-' (single dash) = a space inside one segment
```

| Token | Meaning | Example |
|---|---|---|
| leading `_` (`--_…`) | start a **collection** | `--_theme---…` → collection **theme** |
| `---` (three dashes) | collection → variable name | `--_theme---background` |
| `--` (two dashes) | nested **folder** (`/` in panel) | `--_theme---button--primary--background` → `button/primary` |
| `-` (one dash) | **space** within one segment | `viewport-max` → "viewport max" |
| no leading `_` | **base** collection | `--site--viewport-max` |

⚠️ **One dash vs two:** `button-primary` = one folder named "button primary"; `button--primary` = two nested folders "button" → "primary". The `--` is what creates nesting.

How the foundation's tokens map:

| CSS variable | collection | folder | variable |
|---|---|---|---|
| `--site--gutter` | base | site | gutter |
| `--_theme---background` | theme | — | background |
| `--_theme---button-primary--background` | theme | button primary | background |
| `--_spacing---space--4` | spacing | space | 4 |

## Type comes from the VALUE, not the name

The name decides location; the value decides the Webflow type.

- Color — `#2b6fff`, `rgb()`, `hsl()`, `oklch()`
- Size — `16px`, `1.5rem`, `80%`, `1fr`, `200ms`, `45deg`
- Number — `12`, `1.4`, `600`
- FontFamily — `"Inter", sans-serif`
- Reference — a pure `var(--x)` (see below)
- Raw / fluid — `clamp() / calc() / min() / max() / color-mix()`

Use a value that matches the intended type (e.g. `16px`, not `16`, for a size).

## Variable modes

A mode is a class **`.u-<collection-slug>-<mode>`** that redefines that collection's variables. `<collection-slug>` = the collection name lowercased with spaces → `-` (collection "text style" → `text-style`).

- `.u-theme-dark` → theme collection, mode **Dark**
- `.u-text-style-h2` → text style collection, mode **H2**
- `.u-button-style-secondary` → button style collection, mode **Secondary**
- `.u-trigger-active` → trigger collection, mode **Active** (`on`/`off` flip to `0`/`1`; Base in `:root`)
- `.u-state-active` → state collection, mode **Active** (`true`/`false` flip to `0`/`1`; Base in `:root`)

Rules:

- **Default mode** = the values defined in `:root` / `html` / `body`.
- **An override must DIFFER from the base.** A mode whose values equal the base is skipped and won't register.
- **Value-as-mode:** a collection set to a plain number per component — e.g. `.hero_layout.u-grid-above { --_column-count---value: 3; }` — makes the number the mode name.
- **Put the mode class on a real element** (`u-theme-*` on the section, `u-text-style-*` on the text) so the binding is emitted and not pruned on import.

## Reference & raw

- **Reference:** `--x: var(--y)` (pure) → a reference variable. The target `--y` must also be imported, or the reference is dropped.
- **Raw / fluid:** `clamp/calc/min/max/color-mix(...)` import as a raw value; any `var()` inside resolves to the referenced variable automatically. Just write the normal expression.

## Authoring checklist

- [ ] Collection: leading `_` (`--_<collection>---…`); no `_` = base.
- [ ] Variable name after `---`; folders via `--`; `-` = space.
- [ ] Default value in `:root` / `html` / `body`.
- [ ] Mode = `.u-<collection-slug>-<mode>` with values **different** from base.
- [ ] Mode class is present on a relevant element in the HTML.
- [ ] Value matches the intended type (e.g. `16px`, not `16`).

> Mirrors the naming spec of the HTML-to-Webflow import tool. Sections 2–3 of `lumos-foundation.css` already conform — customize values, keep names.
