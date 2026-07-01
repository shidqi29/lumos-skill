# Changelog

All notable changes to the **Lumos Skill** are recorded here.

The newest entry is always at the **top**, grouped by date (`YYYY-MM-DD`). Within a
date, changes are grouped as **Added**, **Changed**, **Removed**, or **Fixed**.
When a change ships, add it under today's date (create the date heading if it's the
first change of the day).

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
