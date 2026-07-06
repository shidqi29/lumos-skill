# Changelog

All notable changes to the **Lumos Skill** are recorded here.

The newest entry is always at the **top**, grouped by date (`YYYY-MM-DD`). Within a
date, changes are grouped as **Added**, **Changed**, **Removed**, or **Fixed**.
When a change ships, add it under today's date (create the date heading if it's the
first change of the day).

## 2026-07-06

### Added

- **`flex: 1` vs `height: 100%` for filling a `min-height` parent** (Layout) — a flex child needing to fill its parent should use `flex: 1`/`.u-flex-grow`, not `height: 100%`, whenever the parent's height comes from `min-height` (the 100svh full-bleed hero is the common case). `min-height` only sets a floor and the box can grow taller if content needs it, so it isn't a definite height a percentage reliably resolves against — `flex: 1` fills the flex container's actual available space regardless of how that height was established. Distilled from ENEX Bonn (2026-07-06).

## 2026-07-03

### Added

- **Combo-scoping systematic audit.** New guidance to check every component class that co-occurs with a utility for ANY overlapping property, not just the visually-obvious one — with a three-way decision (genuinely differs → scope; identical value → delete the redundant bare declaration; stray hardcoded value duplicating a utility's job → delete entirely). Plus a gotcha: scoping a base rule to a combo also requires re-scoping its `@media` overrides to the same combo, or the breakpoint silently stops firing.
- **Webflow-import combo-chain rule (Buttons).** A one-off button/component combo selector must include the full class chain in order (base + utility-mode + custom combo) — skipping the utility-mode class in the middle beats the utility in the browser but isn't recognized by the HTML-to-Webflow import tool, silently dropping the style on import.
- **Asymmetric grid layouts** (8/4, 7/5, 9/3, …) — use a 12-column `u-grid-above` + `grid-column: span N` per child, not a forced 2-column grid. Stacks automatically below the collapse threshold since flex ignores `grid-column`.
- **Fixed-count list/card groups** should size by content (`height: auto`), not a fixed height + `flex: 1 0 0` children (which can collapse/vanish on reflow at narrower widths).
- **`aspect-ratio` must be paired with an explicit `width` or `height`** — Safari doesn't compute it correctly otherwise (Chrome/Firefox tolerate it).
- **`[data-svg-stroke]` / `[data-svg-fill]` foundation attribute helpers** — shared default stroke/fill treatment for icons, replacing a repetitive one-line CSS rule per icon that just wants the standard look.
- **`[hidden]` vs `display` specificity gotcha** (Trigger & State) — a component's own `display` declaration can beat the native `hidden` attribute's `display: none` at equal specificity; scope as `.component[hidden]` or drive show/hide through the state system instead.
- **Multi-page vanilla projects** (`references/vanilla-mode.md`): a new section covering the shared `global/` folder (foundation + shared components, linked by every page before that page's own styles) and per-page-named JS entry functions (`initHomeFunction`, not a generic `initFunction` repeated in every page).
- **State manager Webflow custom-code requirement** (`references/vanilla-mode.md`): the Trigger & State manager block doesn't survive the HTML-to-Webflow import as native global CSS — it must be pasted into Site Settings → Custom Code → Head once per site, or every `data-trigger`/`data-state`/`.is-active` goes inert post-import.
- **Motion tokens in the foundation** (`--ease-default`, `--duration-default`) — one shared easing/duration so transitions feel consistent project-to-project.
- **Third-party-library principle strengthened** (Output): rename and self-author EVERY class hook a library touches, not just the top-level/slide classes — check the library's config API for a rename option per hook (disabled/locked states, dynamically-injected sub-elements, etc.) and confirm zero of the library's own class names remain as CSS selectors.

### Fixed

- **Motion/duration token naming corrected.** `references/webflow-variable-naming.md` previously listed `200ms` as a safe "Size" type value; in practice a `transition-timing-function` has no representable Webflow Variable type at all, and a plain reusable duration doesn't reliably import as a usable Size variable either. Both must use plain single-dash naming (`--ease-default`, not `--ease--default`) so the import tool leaves them as custom CSS instead of attempting a Variable it can't correctly represent.
- **Trigger/State: clarified the resting-value semantics.** The token defaults (`--_trigger---on: 1`, `--_state---true: 1`) are the RESTING values — the manager flips them on hover/activation. Added an explicit note that in `color-mix(A·on, B·off)` the **first term is the resting look and the second applies on hover/active** (matching the button base: `background` first, `background-hover` second), so the defaults aren't misread as "the value when active" (which inverts hover/rest).

### Changed

- **Buttons: choose a style mode with contrast against the section theme.** Documented that the style modes are theme-relative — in the light theme `u-button-style-tertiary`/`-quaternary` are white-on-white (they exist for dark sections); use `u-button-style-secondary` for an outline button on a light section. Sanity-check each button against its section's theme.

## 2026-07-01

### Added

- **Alt text is now mandatory for every image and asset.** Every `<img>` must carry an `alt` attribute — descriptive for meaningful images, empty `alt=""` + `aria-hidden="true"` for decorative ones. Meaningful inline SVGs/videos need an accessible name too. Added as an Output rule and an anti-pattern; the image-wrapper example no longer ships a placeholder `alt="..."`.

### Fixed

- **State manager now truly complete in `lumos-foundation.css`.** Added the two missing canonical Lumos trigger selectors: `.wf-design-mode [data-trigger~="preview"]` (the documented `preview` trigger was previously inert — the selector activating it was absent) and `[data-trigger~="hover-if-clickable"]`. Documented both trigger types in the Trigger & State System section.

## 2026-06-30

### Added

- Composable grid placement utilities — `u-column-start-*` / `u-column-span-*` and `u-row-start-*` / `u-row-span-*`. Start line and span compose (e.g. `u-column-start-3 u-column-span-4`); `u-column-span-full` (`1 / -1`) and `u-column-span-indent` (`2 / -2`) are absolute spans.

### Changed

- **Responsive is now two systems.** The `u-grid-*` grid stays breakpointless (container queries at the 62 / 48 / 30em thresholds, reacting to container width); every other element's responsiveness uses Webflow `@media` breakpoints (`max-width` 991 / 767 / 479px) so it imports onto the Designer's native breakpoints instead of a custom embed.
- Button colors (`background-color`, `color`, `border`) live on the `_wrap`; the `_text` child inherits the color and never re-sets it.
- Line-length limits use a `u-max-width-*ch` utility directly on the text element — no flex-column wrapper required for the limit itself.
- Documented the trigger/state manager as part of the **global foundation CSS** (components only read the flipped `--_state---*` / `--_trigger---*` variables; without the manager block they do nothing).
- Author component styles yourself even when a third-party library ships its own CSS, so the appearance shows up in the Webflow Designer on export.
- Documented `column-count` as a value-as-mode collection: base mode 1 (`:root`), page default 12 (`page_wrap`), kept as `--_column-count---value: N` in component CSS.

### Removed

- Native Webflow dropdown (`.w-dropdown`). Dropdowns are now built as normal accessible components (component classes, `data-*` hooks, `.is-active` + the trigger/state system, ARIA + keyboard support).

## 2026-06-29

### Added

- Button-style variable modes (`u-button-style-primary` / `-secondary` / `-tertiary` / `-quaternary`) and the `--_button-style---*` alias that buttons read, plus mode-export guidance for vanilla → Webflow imports.
- Trigger/state "Active" variable modes (`u-trigger-active`, `u-state-active`) so the runtime trigger/state systems also export as Webflow variable modes.
- Native Webflow dropdown support _(removed 2026-06-30)_.

### Changed

- Reconciled `lumos-foundation.css` to the real Webflow CSS export — authentic reset, theme classes, `u-section` / `u-container`, type scale, and the imported utility classes.
- Theme now cascades from `page_wrap`; **Light is the base mode** (no class needed). A section gets `u-theme-*` only to override the page default.
- All three named threshold containers (`threshold-large` / `-medium` / `-small`) live on `u-container`, so queries react to the column area's width.
- Standardized every `color-mix()` on the `in srgb` color space.
- JavaScript targets elements by `data-*` attributes, never by class — only the component `_wrap` is selected by class, as the scope anchor.
- Aligned all `SKILL.md` rules and references to the reconciled foundation.

## 2026-06-26

### Changed

- The site header is its own top-level component (the `banner` landmark) and wraps the primary `<nav>`; the navbar is separated from the page sections.
- Reusable UI atoms (buttons, links, badges) use one shared component class with scoped `.is-*` variants instead of per-section copies.
- Links and buttons only wrap text (a `_text` child div); images use a relative `_img_wrap` with an absolutely-filled `img`; the `a` underline reset moved into the foundation.

## 2026-06-25

### Added

- Vanilla mode ships component CSS/JS in external `css/` + `js/` files with a single `initFunction()` entry point on `DOMContentLoaded`.
- Combo-class scoping rule and the `references/webflow-variable-naming.md` reference.

### Changed

- Switched the responsive approach to the threshold / container-query system.
- Moved body-level styles onto `page_wrap` so they survive the HTML-to-Webflow import (Webflow can't paste a bare `<body>`).

## 2026-06-24

### Added

- Initial Lumos skill and README.
- Root CSS foundation imported from the Webflow project.
- Repository docs: README rewritten around skill usage, `.gitignore` added.
