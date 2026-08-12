# Changelog

All notable changes to the **Lumos Skill** are recorded here.

The newest entry is always at the **top**, grouped by date (`YYYY-MM-DD`). Within a
date, changes are grouped as **Added**, **Changed**, **Removed**, or **Fixed**.
When a change ships, add it under today's date (create the date heading if it's the
first change of the day).

## 2026-08-12

### Fixed

- 🔴 **`u-container` / `-small` / `-full` now carry ONE container name (`threshold-large`) instead of all three** (`assets/lumos-foundation.css`). Declaring `threshold-large threshold-medium threshold-small` together reads like "hosts all three thresholds", but an element answering to several names is still **one container** — so all three `@container` queries resolved against the same box, and because the three threshold blocks are **identical rule sets** the widest matching condition always won:
  - `.u-grid-above` → flex when `width < 62em`, making `< 48em` and `< 30em` dead code;
  - `.u-grid-below` → flex when `width >= 30em` instead of `>= 62em`.

  **Measured, not inferred — and `u-grid-below` was worse than "wrong breakpoint": it never worked at all.** Probing the shipped file at container widths 1200 / 900 / 700 / 560 / 440px, `u-grid-below` computed `display: flex` at **every one of them**; it only becomes a grid below 30em (440px of container), which no real layout reaches. After the fix it is a grid below 62em and stacked above it, which is what the name means. `u-grid-above` measures **identically before and after** (grid above 62em, flex below) — which is exactly why this survived: the utility everyone actually uses was already correct.

  So `threshold-medium` and `threshold-small` were unreachable on a `u-container`. Nothing errors, nothing overflows, no sweep catches it. **To use the medium or small threshold, declare a nested container with the `u-threshold-medium` / `u-threshold-small` utilities — that is what they are for.**

  ⚠️ **Breaking in two narrow ways.** (1) Any component that wrote `@container threshold-medium|small` expecting it to resolve against `u-container` — those rules now match no container and never apply; move them to `threshold-large` at the width they actually meant, or add the matching `u-threshold-*` utility to a wrapper between the container and the element. (2) Anything that had `u-grid-below` on it now genuinely becomes a grid below 62em, where before it was always stacked — check those layouts. Existing projects pin their skill version and are unaffected until they re-sync. From GTM-24.

### Added

- 🔴 **`u-container` carries `flex: 1`, so `justify-content` on the section does nothing** (Rules §Sections & Containers). The container is the section's only flex child and absorbs every spare pixel of the column, so `justify-content: center` on `.[name]_wrap` has nothing left to distribute — the content stays at the top of a container that is simply taller, with no error and no visual clue the rule was ignored. Vertical centring goes on the **container**: `display: flex; flex-flow: column; justify-content: center`. Measured on a real hero: content sat 90px above the comp before, within 4px after. Bites any full-height band. From GTM-24 (three separate heroes).
- 🔴 **The foundation's `li { display: block }` removes the list-item box, so a list renders with NO marker — and `list-style-type` is not the fix** (Rules §Typography). A block box generates no marker whatever `list-style-*` says, and it drops the list semantics assistive tech reads. Put `display: list-item` back on the item's own class. **Fails silently and completely**: no error, no warning, the glyphs are simply nowhere. From GTM-24, and it bites every CMS rich-text block.
- 🔴 **A custom list marker must be a `background-image` on the `<li>`, never `list-style-image`** (Rules §Typography). A marker box is pinned **bottom-edge to the text baseline** and no property moves it, so a 20px glyph on a 24px line sits ~6px above the text's optical centre. Padding inside the SVG does not help: making the artwork taller shifts the glyph **up** whichever end the blank is added to, because it is the box's bottom that is pinned — measured against a three-variant fixture (20×20, 20×26 blank-bottom, 20×26 blank-top → item heights 27 / 33 / 33). A background is *positioned* rather than aligned, so it can be centred at `(line-height − icon) / 2`; wrapped lines then align under the **text** rather than under the icon. Also the more Webflow-native of the two — `list-style-image` is not exposed in the Designer, a background on "All List Items" plus "List style: None" is. From GTM-24.
- **Every `u-grid-*` threshold measures its nearest named container, which is normally the PAGE container — not the column the grid sits in** (Rules §Responsive). A two-up pair inside an 8-column form column still collapses on the page container's 62em (~1112px of viewport at default margins), long after that column ran out of room for two fields. **A pair that has to collapse on its own width is a wrapping flex row, not a grid**: `flex-flow: wrap` + `flex: 1 1 <basis>` + `min-width: 0`. Reach for `u-grid-*` when the thing collapsing is a full-width band. From GTM-24 (Contact and Schnellanfrage forms).

## 2026-08-07

### Added

- **An exact proportional split between padded items is grid `fr`, not `flex-grow` with `flex-basis: 0`** (Rules §Layout + Anti-patterns) — with `box-sizing: border-box` a flex item can never be narrower than its own padding, so that floor comes out of the pool **before** the grow ratio is applied. Measured: two panels with 40px padding asked for 615:412 came back **600:428**, and `min-width: 0` did not help, because the floor is padding rather than content. `grid-template-columns: minmax(0, 615fr) minmax(0, 412fr)` divides the free space after the gap and knows nothing about item padding. From Qubes & Co (CTA plates).
- **An explicit `gap` on a `justify-content: space-between` container is a floor, not a spacing** (Rules §Layout + Anti-patterns) — space-between already distributes the leftover; the gap only bites when that leftover is *smaller*, and then it pushes the container past the size the comp fixed. A `space--9` gap over a 31.2px leftover took a 479px panel to 506 and dragged the whole row with it. In a column whose height the design fixes, write no gap. From Qubes & Co.
- **Paint order among siblings must be stated once any of them is transformed** (Rules §Layout + Anti-patterns) — a `transform` creates a stacking context, so a rotated sibling paints in a later phase than its un-transformed neighbours and lands on top of them regardless of DOM order. In an overlapping card fan that buried the un-rotated card's icon under its neighbour's corner, with nothing in the CSS to point at. If the design stacks in source order, give the un-transformed ones an explicit `z-index`. From Qubes & Co (benefits fan at mobile).
- **Reserve an out-of-flow child's footprint with `aspect-ratio` on the parent instead of re-forcing the grid** (Rules §Layout + Anti-patterns) — when a design keeps two blocks overlapping at mobile rather than stacking them, the readable build is parent `position: relative` + `aspect-ratio`, child `position: absolute`. `min-height: auto` still applies, so the parent grows if the copy outgrows the ratio — the stack-on-overflow fallback comes free. Re-forcing `display: grid` over the framework's `display: flex !important` collapse means answering an `!important` with another one, which leaves two rules insisting at each other. From Qubes & Co (hero at 390).
- **Class names must be unique across the SITE in a multi-page project — put the page in the `{variation}` slot.** A Webflow class name is one global object for the whole site, so `hero_wrap` cannot mean something different on Home and About; sections every page has become `hero_home_*` / `hero_about_*`, sections only one page has keep their plain name. Recorded with the reason it is dangerous: **it is invisible in vanilla.** Each page loads its own stylesheet, all pages render correctly, and no sweep, render or pixel diff reports anything — the collision only exists after import, where the last page overwrites the others. Includes the one-pass audit (parse each page stylesheet's selectors, intersect the class names, require an empty intersection). Distilled from Qubes & Co, where it found 16 collisions across three pages, every one of them in the hero, because Figma names them all `hero_wrap`.
- **The 3-underscore cap is a forcing function, not a style preference.** Adding the page scope is what usually pushes a deep name over it; the answer is never a fourth underscore but promoting the inner part to its own subcomponent (`hero_set_visual_shadow_wrap` → `setstack_shadow_wrap`). Hitting the cap is the signal that a subcomponent wants a name.
- **A grid seam belongs to the cell that comes AFTER it** — its leading edge IS the seam. Hung on the preceding cell, the line draws down the outside of the grid. Extended to the collapsed case: when wrappers become `display: contents`, the seams must move onto the cells, since a `display: contents` box paints no border and is not a containing block.
- **Prefer a border owned by each cell over absolutely-positioned rules with computed offsets.** When a comp staggers its rules, per-cell edges make the stagger fall out for free (the columns are genuinely different heights) and the container bleed comes free with them; percentage-of-section offsets only hold at the height the section happens to be.

### Changed

- **`--duration--default` in `assets/lumos-foundation.css` now reads the override hook** — `var(--default-duration, 0.4s)` instead of a literal `300ms`, so a project can retune every transition by defining `--default-duration` once. This aligns the skill's own asset with the decision already taken on 2026-07-30 and applied to the Fig-2-Web mirror; the two had diverged since, with the skill left on the old literal. ⚠️ **The default moves 300ms → 400ms**, deliberately matching `--nav-duration`. To keep 300ms, change the fallback and leave the hook.
- The image-wrapper rule now says explicitly: **never put the sizing on the `<img>` itself — an `aspect-ratio` there loses to the image's own `width`/`height` attributes.** A shot sized directly on the `<img>` came out 362 wide (correct) and 1024 tall (its intrinsic height). Flagged as easiest to miss inside a scalable visual composition, where children read as artwork rather than as content images.
- Five matching anti-patterns added.
## 2026-08-06

### Added

- **A library's `overflow: hidden` root is also its min-content reset** (Output §third-party libraries + Anti-patterns) — a scroll container's min-content size is `0`, and that zero is what stops N non-shrinking `width: 100%` slides from contributing their combined min-content to whatever sizes the root. Lumos prefers `overflow: clip`, which is **not** a scroll container, so making that swap on a slider root silently closes a sizing cycle — no error, the browser just resolves at its maximum length. Measured: a grid column at `33,554,406px` and a page `18,755,333px` tall from removing one declaration. Swap `hidden` → `clip` on an element whose children can't shrink and you must add `min-width: 0` back. From Qubes & Co (Swiper on a Webflow Collection List).
- **Check a rename target isn't reserved by the foundation before handing it to a library** (Output §third-party libraries + Anti-patterns) — a bare `.is-active` is part of the State Manager and flips `--_state---true`/`--_state---false` for the element and everything inside it. Used as a slider's `slideActiveClass` it flips every stateful descendant of the active item only, which is near-undebuggable. The "only `.is-active` for state" rule covers state *you* manage, not a class the library stamps at runtime; namespace it (`is-slide-active`) and treat `.is-current` / `.is-open` the same. From Qubes & Co.

## 2026-08-05

### Added

- **`flex-flow: wrap`, never `flex-flow: row wrap`** (Output + Anti-patterns) — `row` is already the initial `flex-direction`, so the two-value form only restates the default while reading as a deliberate direction change, and on a Webflow import it lands as an explicit direction setting rather than the plain "wrap children" toggle the Designer exposes. `flex-flow: column wrap` stays valid (there `column` is real information), as do the direction-only `flex-flow: row` / `flex-flow: column`. DoD sweep: `grep -n 'flex-flow: row wrap'` must return nothing. Operator directive (Shidqi, Qubes & Co slicing).
- **State combos belong on the BASE class, not stacked on a variant combo** (Class Naming + Anti-patterns) — when a component already carries variant combos (`.card_wrap.is-1`, `.is-2`, `.is-3`), write the active state once against the base as `.card_wrap.is-active` so it covers every variant automatically. Authoring `.card_wrap.is-1.is-active`, `.card_wrap.is-2.is-active`, … means one combo per variant to build and maintain in the Designer (plus a `_hidden` placeholder each), silently misses any variant you forget, and breaks when the item count changes. Corollary: JS toggles `.is-active` on the element carrying the base class — that element is the state root, and its children read `var(--_state---*)` instead of getting their own `.is-active`. Operator directive (Shidqi, Qubes & Co slicing).

## 2026-07-08

### Added

- **Text-style collection is capped at 12 modes** (Typography) — Webflow limits a variable collection's modes, and each `u-text-style-*` class = one mode on import, so the foundation ships exactly the canonical 12 (`display`, `h1`–`h6`, `large`, `medium`, `main`, `small`, `tiny`) and you must never add a 13th. A design with >12 distinct sizes maps onto the 12 by pixel size (rename, don't add); off-scale one-offs stay component-CSS `rem` (no mode). From Stadtform (a foundation had grown to 15 text styles → broke a clean Webflow import).
- **Grid-first layout is now stated as a hard default** (Layout) — every real multi-column layout is `u-grid-above` + column-spans, never `display:flex`+`flex:1`/fixed `rem` widths; keep flex only for visual compositions, compact positioned groups, single-child centering, form-field rows, and non-collapsing label|value tables. Also spells out the mechanical swap (drop `flex:1`, add `width:100%`, delete the `@media flex-flow:column`). From Stadtform grid-first refactor (operator directive).

### Changed

- **Foundation customization: no duplicate variables** (Modes §Vanilla, point 4) — customize by editing the placeholder token value in-place; never leave the same variable defined twice with different values (a second override `:root` block that redefines a foundation token is a bug — last wins, dead placeholder confuses editors, imports ambiguously). Overwrite the placeholder or make a genuinely new name. From Stadtform (foundation had 23 vars defined twice with conflicting values).

## 2026-07-06

### Added

- **`flex: 1` vs `height: 100%` for filling a `min-height` parent** (Layout) — a flex child needing to fill its parent should use `flex: 1`/`.u-flex-grow`, not `height: 100%`, whenever the parent's height comes from `min-height` (the 100svh full-bleed hero is the common case). `min-height` only sets a floor and the box can grow taller if content needs it, so it isn't a definite height a percentage reliably resolves against — `flex: 1` fills the flex container's actual available space regardless of how that height was established. Distilled from ENEX Bonn (2026-07-06).
- **Vanilla-mode token overrides: same name, not a parallel prefix** (`references/vanilla-mode.md`) — when a project's real value differs from the foundation's placeholder default, redeclare the *same* custom property (section 2 already loads after the defaults) instead of adding a project-prefixed variant name alongside the untouched original. A parallel namespace leaves two font-size/spacing scales in the same file with no signal which one components should actually use. Distilled from Stadtform (2026-07-06) — a `sf-` prefix had crept across nearly every token in section 2, including ones that turned out to be exact-value duplicates of existing defaults.
- **Vanilla-mode token overrides: check for hidden dependents before overriding** (`references/vanilla-mode.md`) — some tokens are load-bearing beyond their obvious use. `--_typography---font-size--text-main` looks scoped to `u-text-style-main`, but the foundation's own default also points `page_wrap`'s base font-size (`--_text-style---font-size`) at it. Overriding `text-main` with a project's large/prominent-paragraph value (correctly, per the rule above) can silently inflate the whole page's inherited base font-size — invisible to `rem`-based CSS but not to anything in `em`, including the `u-grid-above`/`u-grid-below` `@container` thresholds. Distilled from Stadtform (2026-07-06): the grid silently ran in flex mode at every desktop width (1440–2560px) until traced back to this.

### Fixed

- **Reverted the 2026-07-03 single-dash decision for `--ease-default`/`--duration-default`.** That entry called single-dash naming deliberate so the Webflow import tool wouldn't attempt to represent a transition-timing-function/duration as a Variable it has no type for. Real-world use in Stadtform (2026-07-06) settled on double-dash (`--ease--default`, `--duration--default`) instead. Foundation and any project still on the single-dash names should move to double-dash.

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
