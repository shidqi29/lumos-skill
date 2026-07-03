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
- Size — `16px`, `1.5rem`, `80%`, `1fr`, `45deg`
- Number — `12`, `1.4`, `600`
- FontFamily — `"Inter", sans-serif`
- Reference — a pure `var(--x)` (see below)
- Raw / fluid — `clamp() / calc() / min() / max() / color-mix()`

Use a value that matches the intended type (e.g. `16px`, not `16`, for a size).

**Some values have no matching Webflow Variable type at all — a transition-timing-function (`cubic-bezier(...)`, `ease`) has no representable type, and a plain duration meant to stay a reusable motion token (`300ms`) does not reliably come across as a usable Size variable in practice.** Don't name these with the collection convention (`--_<collection>---<name>` or the base-collection `--<name>--<name>` form) expecting them to become native Variables — that invites the import tool to try and fail. Give them a **plain single-dash name with no collection separator** (`--ease-default`, `--duration-default`, not `--ease--default`/`--_motion---ease--default`) so the import tool recognizes no collection pattern and leaves them as plain CSS custom properties (they still work identically via `var()`, they just land in the site's custom CSS rather than the Variables panel). Reserve the collection-naming convention for values that DO have a real Webflow Variable type (color/size/number/font-family).

## Variable modes

A mode is a class **`.u-<collection-slug>-<mode>`** that redefines that collection's variables. `<collection-slug>` = the collection name lowercased with spaces → `-` (collection "text style" → `text-style`). On a vanilla→Webflow import, each such class becomes a **mode** of its collection.

`lumos-foundation.css` ships these mode classes — apply them in your markup and the modes come across on import:

| Collection | Variables | Mode classes (foundation) |
|---|---|---|
| theme | `--_theme---*` | **base = Light** (`:root`); `.u-theme-dark` / `.u-theme-brand` are the override modes; `.u-theme-light` just re-applies the base (so it folds into base on import, not a separate mode) |
| text style | `--_text-style---*` | `.u-text-style-display` / `-h1`…`-h6` / `-large` / `-medium` / `-main` / `-small` / `-tiny` |
| button style | `--_button-style---*` | `.u-button-style-primary` / `-secondary` / `-tertiary` / `-quaternary` |
| gap | `--_gap---size` | `.u-gap-0`…`-8` / `-gutter` / `-inherit` |
| alignment | `--_alignment---direction` | `.u-alignment-start` / `-center` / `-end` |
| trigger | `--_trigger---*` | `.u-trigger-active` (Base `on:1/off:0` → Active `on:0/off:1`) |
| state | `--_state---*` | `.u-state-active` (Base `true:1/false:0` → Active `true:0/false:1`) |

Rules:

- **Default mode** = the values defined in `:root` / `html` / `body` (e.g. theme→Light, button-style→Primary, trigger→Base).
- **An override must DIFFER from the base.** A mode whose values equal the base is skipped and won't register.
- **Value-as-mode:** a collection set to a plain number per component — e.g. `.hero_layout.u-grid-above { --_column-count---value: 3; }` — makes the number the mode name. The **column-count** collection works this way: base mode is 1 (`:root`), the page default is 12 (set on `.page_wrap`), and writing `--_column-count---value: N` on a component registers mode "N" — no utility class.
- **Put the mode class on a real element** so the binding is emitted and not pruned on import — `u-theme-*` on the section, `u-text-style-*` on the text, `u-button-style-*` on the button, `u-gap-*` on the flex/grid container, `u-alignment-*` on the content wrapper. A mode whose class never appears in the HTML won't import. (The `u-trigger-active`/`u-state-active` modes ride on the runtime state-manager rules rather than a static element — verify they survive a real import.)

## Reference & raw

- **Reference:** `--x: var(--y)` (pure) → a reference variable. The target `--y` must also be imported, or the reference is dropped.
- **Raw / fluid:** `clamp/calc/min/max/color-mix(...)` import as a raw value; any `var()` inside resolves to the referenced variable automatically. Just write the normal expression.

## Authoring checklist

- [ ] Does this value have a real Webflow Variable type (color/size/number/font-family)? If not (easing, a plain reusable duration) → single-dash name, skip the rest of this checklist.
- [ ] Collection: leading `_` (`--_<collection>---…`); no `_` = base.
- [ ] Variable name after `---`; folders via `--`; `-` = space.
- [ ] Default value in `:root` / `html` / `body`.
- [ ] Mode = `.u-<collection-slug>-<mode>` with values **different** from base.
- [ ] Mode class is present on a relevant element in the HTML.
- [ ] Value matches the intended type (e.g. `16px`, not `16`).

> Mirrors the naming spec of the HTML-to-Webflow import tool. Sections 2–3 of `lumos-foundation.css` already conform — customize values, keep names.
