---
name: lumos-skill
description: 'Build responsive layouts with the Lumos Framework by Timothy Ricks — in Webflow OR in a plain HTML/CSS/JS project built from scratch. Use this skill whenever the user asks about Webflow layouts, HTML structure, responsive design, Lumos classes, Lumos utility classes, components, sections, grids, flexbox, spacing, typography, or color themes — and ALSO when building a vanilla HTML/CSS/JS site, landing page, or component from scratch, a static site, or a layout "without Webflow" where the user wants Lumos conventions (utility + component classes, fluid sizing, a breakpointless grid, and Webflow-breakpoint responsiveness). Trigger when the user mentions "Lumos", "u-" classes, "breakpointless", fluid sizing, or framework patterns — even if they don''t say "Lumos Framework" or "Webflow" — and whenever the work involves building a responsive section, hero, nav, or page in raw HTML/CSS/JS.'
---

# Lumos Framework — Responsive Layouts

Docs: https://lumos.timothyricks.com/

## Modes

This skill works in two modes. Figure out which one applies, then follow the shared rules below — they are identical in both modes. Only the foundation and a few Webflow-specific workarounds differ.

**Webflow / Lumos project (existing).** The default when the user is inside a Webflow site or an existing Lumos project, or pastes Lumos markup to extend. The foundation — every `u-*` utility and every `--_*` variable — is already defined by the project. Output is pasted in, so **never** redefine utilities, variables, resets, `:root`, or `body`. All rules apply, including the ones tagged _(Webflow only)_.

**Vanilla project (from scratch).** Use when the user asks to build with plain HTML/CSS/JS, "from scratch", a landing page or static site, or a layout "without Webflow" — i.e. there is no existing Lumos project. The foundation does **not** exist yet, so generate it first:

1. Copy `assets/lumos-foundation.css` into the project (e.g. `css/lumos-foundation.css`) and link it once in `<head>` **before** any component styles. It provides the reset, the design tokens (`:root` + `u-theme-*` classes), every `u-*` utility, and the responsive + trigger/state systems the components depend on.
2. Build components exactly as the shared rules below describe — identical class naming and techniques — with one change: collect component CSS into `css/styles.css` (linked after the foundation) and component JS into `js/main.js` (loaded once before `</body>`), instead of the inline per-component `<style>`/`<script>` blocks the rules show. The markup and class names are unchanged; only where the CSS and JS live differs. See `references/vanilla-mode.md`.
3. The token values in the foundation are starting defaults. Point the user to the clearly-marked **section 2** of the file to set brand colors, spacing, type scale, and fonts.

In vanilla mode, the rules tagged _(Webflow only)_ are **dropped** — they exist solely to work around Webflow Designer behavior that isn't present:

- No combo-class-purging workaround. Combo classes may be CSS-only, so the `_hidden u-display-none` placeholder for dynamically-applied combo classes is **not required**. (The hidden-template clone pattern is still recommended for JS-appended markup — it's clean, not a workaround.)
- No empty-div `padding: 0` fix (Webflow's default element padding doesn't exist).
- No reliance on `.w-*` runtime classes.

When generating a full page, see `references/vanilla-mode.md` for the project skeleton, an HTML boilerplate, token customization, and fidelity notes. The foundation's variable names follow the Webflow Variables import convention so a vanilla project imports back into Webflow as native variables — keep the names intact and follow `references/webflow-variable-naming.md` when adding or customizing tokens.

---

## Rules

### Output

- Vanilla HTML, CSS, JS only
- Only write CSS for component classes, their container queries, and Webflow `@media` breakpoints — never for `u-*` utilities
- No CSS resets, `:root` definitions, `body`/`page_wrap` styles, or utility redefinitions
- **Never style the bare `body` selector.** Webflow can't paste a `<body>`, so the import tool turns the body's class into a wrapper `<div>` — only class-based styles survive. All body-level styles (font, color, background) live on the `page_wrap` wrapper class in the foundation (the threshold responsive containers live on `u-container`). In vanilla output wrap the page in `<body class="page_wrap">` (see `references/vanilla-mode.md`)
- No `px` for sizing/layout — default to `rem`; text `max-width` uses `ch` via a `u-max-width-*ch` utility (see Typography). **Two breakpoint exceptions:** the `u-grid-*` container-query thresholds use `em` (`62`/`48`/`30em`), and Webflow `@media` breakpoints use Webflow's exact px (`991`/`767`/`479`) so the import maps them to Designer breakpoints (see Responsive)
- Class-only selectors — no tag names, IDs, data attributes, or descendant selectors
- No fallback values in `var()`
- No inline `style=""` — put styles in `<style>` block
- `<style>` first child inside `_wrap`, `<script>` last child:
  ```html
  <section class="hero_wrap u-section">
    <style>
      /* ... */
    </style>
    <!-- component markup -->
    <script>
      /* ... */
    </script>
  </section>
  ```
  _(Webflow only — in vanilla mode there are no inline `<style>`/`<script>` blocks; component CSS goes in `css/styles.css` and component JS in `js/main.js`. Markup, class names, and JS scoping are unchanged — only the location differs.)_
- **Author every visual style yourself, even when a third-party library ships its own CSS.** If a component uses a library that serves its own stylesheet (sliders, carousels, date pickers, lightboxes, etc.), still write the appearance as your own component CSS rather than relying on the library's served styles. On a Webflow export the library's runtime CSS isn't visible or editable in the Designer — only the styles you author carry over, so the component must look correct from your CSS alone (treat the library's stylesheet as behavior/positioning scaffolding, not the source of the look). **This applies to every class the library touches, not just the obvious ones.** It's easy to rename the library's top-level/slide classes to your own and stop there while leaving one deeper hook (a dynamically-injected sub-element the library creates at runtime, a disabled/locked state class, a pagination fill) still keyed to the library's literal class name and still relying on its CSS for that one piece. Check the library's config API for a rename option for **every** class it applies (most libraries expose one per hook, not just for the top-level wrapper/slide) — rename all of them to your own component classes, author each one's CSS yourself, and confirm afterward that zero of the library's own class names remain as CSS selectors anywhere in your stylesheet (only in the library's own `<link>`/`<script>` tag)
- No `::before`/`::after` — use a `<div>` with a class
- No `<em>` tag for italic — use `font-style: italic` in CSS on a `<span>` with a component class
- Use `<div>` for text elements, not `<span>` — only use `<span>` inside headings (`<h1>`–`<h6>`) or paragraphs (`<p>`)
- Empty divs (decorative, spacers) need `padding: 0` — Webflow adds default padding _(Webflow only)_
- `background-color` not `background`. `overflow: clip` not `overflow: hidden`
- When setting `color` alongside `background-color`, apply both to the same element — don't put `background-color` on a parent and `color` on a child. The element with the background should own its text color
- `var(--border-width--main)` for all border widths
- Always use `--_theme---*` variables for colors. When a design specifies a color outside the theme system, map it to the closest theme variable. **Never use hex codes anywhere — not in CSS, comments, or explanatory text.** Don't tell the user to map hex values — just use the closest `--_theme---*` variable directly
- `img`/`video` default to `object-fit: cover` (foundation) — don't re-add `object-fit`. Don't size a bare `img`; use the image-wrapper pattern below
- **Every image and asset needs alt text.** Every `<img>` must carry an `alt` attribute — descriptive alt for meaningful images (`alt="Team reviewing analytics dashboard"`), empty `alt=""` **plus** `aria-hidden="true"` for purely decorative ones. Never omit `alt` or leave a placeholder like `alt="..."` in real output. Other assets that convey meaning need an accessible name too: a meaningful inline SVG gets a `<title>` or `aria-label` (decorative SVGs get `aria-hidden="true"` — see Class Naming); a `<video>` needs a text alternative (caption/label). The `_img_wrap` pattern's `<img>` still owns the `alt`
- **Images use a relative wrapper + an absolutely-filled `img`.** Put the `<img>` in a `_img_wrap` div that defines the box (`position: relative` + an `aspect-ratio`, or sized by the layout); the `<img>` fills it with `position: absolute; width: 100%; height: 100%` (object-fit cover is already the default). The skeleton loading color goes on the wrapper:
  ```html
  <div class="hero_img_wrap">
    <img class="hero_img" src="..." alt="Descriptive text for the image" loading="lazy" />
  </div>
  ```
  ```css
  .hero_img_wrap {
    position: relative;
    width: 100%;
    aspect-ratio: 3 / 2;
  }
  .hero_img {
    position: absolute;
    width: 100%;
    height: 100%;
    /* object-fit: cover is the foundation default */
  }
  ```
  **`aspect-ratio` must always be paired with an explicit `width` or `height` on the same element — never `aspect-ratio` alone.** Safari doesn't compute `aspect-ratio` correctly on an element whose size is otherwise only implied (e.g. a grid/flex child sized entirely by its track, with no explicit `width`). Chrome/Firefox tolerate this; Safari doesn't. Always give the element a `width: 100%` (or explicit `height`) alongside its `aspect-ratio` — e.g. `.card_founder_wrap { width: 100%; aspect-ratio: 545 / 526; }` — and sweep every `aspect-ratio` declaration in a build for a paired width/height before calling it done
- Icons/logos next to text need `flex-shrink: 0`
- Square icons/logos: `width` + `aspect-ratio: 1/1` — not `width` + `height`
- Logos need `object-fit: contain` (overrides default `cover`)
- Input elements must have `font-size` no smaller than `1rem` — below `1rem` triggers auto-zoom on iOS
- SVGs get their own component class. Stroke attributes in CSS, not inline. `stroke-width: var(--border-width--main)`. `stroke: currentColor`
- **For icons that just need the standard stroke treatment, use the foundation's `[data-svg-stroke]` attribute instead of writing a one-line stroke rule per icon.** Writing `.icon_name_path { stroke: currentColor; stroke-width: var(--border-width--main); fill: none; }` for every single icon component (arrows, chevrons, nav glyphs, …) is pure repetition when they all resolve to the same declaration. Put `data-svg-stroke` on the `<svg>` (covers every `<path>` inside) or on one `<path>` directly; color comes from `color` on the icon's own component class (`currentColor` inherits the section/button color automatically — set `color` explicitly only when an icon needs to diverge, e.g. an inverted badge). The icon still keeps its own component class for anything that DOES need per-icon customizing (size, a non-standard stroke-width via `--svg-stroke-width`); the attribute only replaces the repetitive default-stroke rule, not the class
- Respect `prefers-reduced-motion` for complex animations — not needed for simple CSS transitions
- Accessibility: tabs need `role="tablist"`, `role="tab"`, `role="tabpanel"`, `aria-controls`, `aria-selected`, `aria-labelledby`. Accordions need `aria-expanded`, `aria-controls`. Keyboard nav (`Enter`, `Space`, arrows). Screen reader–only text: use `u-sr-only` class

### JavaScript

- Wrap each component's logic in a named init function — select the component root with `document.querySelector`, then early-return if it's absent so the function is safe to call on any page:
  ```js
  const initHeroWrap = () => {
    // 1. Select the component root as the scope
    const wrap = document.querySelector(".hero_wrap");
    // 2. Early-return validation
    if (!wrap) return;
    // 3. Target child elements by data attribute, never class
    const button = wrap.querySelector("[data-hero-button]");
  };
  ```
  Name it `init` + the component in PascalCase (`.hero_wrap` → `initHeroWrap`). If a component repeats on the page, select all and iterate inside the same function, early-returning when none exist: `const cards = document.querySelectorAll(".card_wrap"); if (!cards.length) return; cards.forEach((wrap) => { /* wrap.querySelector(...) */ });`
- One entry point: collect every component's init into a single `initFunction()`, called once on `DOMContentLoaded`. Inits that measure layout or play reveal animations go inside `document.fonts.ready.then()` so web fonts load first:
  ```js
  const initFunction = () => {
    initNavWrap();
    initHeroWrap();
    initServicesWrap();

    document.fonts.ready.then(() => {
      initRevealWrap();
      initWhyWrap();
    });
  };

  document.addEventListener("DOMContentLoaded", () => {
    initFunction();
  });
  ```
  No per-component init guard — each init runs exactly once from the single entry point.
- **Placement by mode.** _Vanilla_: every init function, `initFunction()`, and the single `DOMContentLoaded` live in `js/main.js`. _Webflow_: each component's inline `<script>` is standalone — it defines its own init function and calls it in its own `DOMContentLoaded` (separate embeds can't share one `initFunction()`). A component that may be placed multiple times in Webflow keeps the `querySelectorAll().forEach()` + `dataset.scriptInitialized` guard instead of a top-level named const, since duplicate embeds would otherwise redeclare it _(Webflow only)_.
- `const`/`let` only, no `var`. No ALL_CAPS names
- **Select elements only by custom `data-*` attributes — never by class** (and never by `id`). Classes are for styling and can be renamed or purged; `data-*` are stable JS hooks. Name them `data-[component]-[element]` (e.g. `data-tabs-link`, `data-hero-button`). The **one exception is the scoping anchor** — select the component wrap by its `_wrap` class — and `document.querySelector` is used only to grab that wrap (or, rarely, reach outside the component; comment it). Everything inside the wrap is queried via `data-*`, scoped to the wrap
- Only `.is-active` for toggling state — no `.is-visible`, `.is-open`, etc.
- JS-appended elements: never hard-code HTML strings. Place a hidden template inside the component wrapped in `.[component]_hidden.u-display-none`. JS clones from the template:
  ```html
  <div class="tabs_hidden u-display-none" data-tabs-hidden>
    <div class="tabs_toast" data-tabs-toast>
      <span class="tabs_toast_text u-text-style-small" data-tabs-toast-text></span>
    </div>
  </div>
  ```
  ```javascript
  const template = wrap.querySelector("[data-tabs-toast]");
  const toast = template.cloneNode(true);
  toast.querySelector("[data-tabs-toast-text]").textContent = message;
  wrap.querySelector("[data-tabs-list]").appendChild(toast);
  ```
  Never use `innerHTML`, `createElement`, or template literal HTML for elements with visual structure
- Don't add classes solely for JS (Webflow purges classes without CSS) — add a `data-*` attribute for JS targeting instead of an extra class or DOM-order reliance
- Screen size checks: `getComputedStyle` not `window.innerWidth`
- Mobile interaction override: `transform: unset !important;` inside a Webflow mobile breakpoint (`@media screen and (max-width: 767px)`) or `@media (hover: none)`

### Class Naming

- Component class first, then utilities: `<h2 class="hero_title u-text-style-h2">`
- **Treat the utility as a combo class — scope component CSS with it.** When an element has a component class plus a utility class, write its CSS as the combo `.component_class.u-utility-class { }`, never bare `.component_class { }`. The combo (specificity 0,2,0) outranks the utility (0,1,0), so the component's own declarations win while every property it doesn't set still comes from the utility. E.g. `.hero_title.u-text-style-h1 { font-weight: var(--_typography---font--primary-medium); }` keeps all h1 styling but overrides just the weight. Joining with one utility class is enough — it already beats any single-class utility selector regardless of source order. If the element carries several utilities, join with the `u-text-style-*` one
- **Audit for this systematically — the overlap is easy to miss when it isn't the "obvious" property.** Don't scope only the combos you already suspect are broken. For every component class that co-occurs with a utility (`u-section`, `u-container`, `u-theme-*`, `u-button-group`, `u-text-style-*`, `u-max-width-*ch`, …), check whether the bare rule sets ANY property the utility also sets — `background-color`/`color` (`u-theme-*`, `u-section`), `flex`/`z-index` (`u-container`), `justify-content`/`padding-*` (`u-section`), `font-weight`/`margin-bottom` (`u-text-style-*`), `max-width` (`u-max-width-*ch`), `gap` (`u-button-group`). A real Lumos build found nine such unscoped overlaps in one project — most were invisible locally because the component's own stylesheet happened to load after the foundation, which only works by accident. Three outcomes per overlap, don't default to "scope it": (1) the values genuinely differ → scope it as a combo; (2) the values are identical → delete the redundant bare declaration instead (the utility already provides it — don't restate it); (3) the bare declaration is a stray hardcoded value that duplicates what a utility already owns (e.g. a literal `max-width` on an element that also carries `u-max-width-*ch`) → delete it entirely, the utility is the single source of truth
- **A component rule scoped as a combo must ALSO have its `@media` overrides re-scoped to the same combo.** Raising a bare rule's specificity to `.component.u-utility` also raises the bar for anything meant to override it later — including your own responsive breakpoints. `@media (max-width: 991px) { .component { padding-top: X; } }` (still bare, 0,1,0) will silently stop applying once the non-media base rule becomes `.component.u-utility` (0,2,0), because the non-media rule now outranks it regardless of the media condition. Whenever you scope a base rule, grep for that class inside every `@media` block and scope those overrides identically, or the breakpoint quietly breaks
- **This is separate from — and in addition to — Webflow-import fidelity (see the button rule below): the combo you author must match the exact class stack that will exist on the element, not just beat the utility on specificity**
- Underscores separate parts: `[component]_[type]_[element]`
- Broadest type first → specific → element: `card_testimonial_title`, `cta_secondary_visual_img`
- Preferred names: `_title` not `_heading`, `_text` not `_paragraph`, `_img` not `_image`
- `_wrap` marks component/subcomponent start: `tabs_wrap` (component), `tabs_link_wrap` (subcomponent)
- Max 3 underscores. Deeper nesting starts a new subcomponent: `cta_secondary_icon_wrap` not `cta_secondary_visual_icon_wrap`
- Hyphens only for multi-word parts: `tabs_link_wrap` not `tabs_link-wrap`
- Utilities: `u-` prefix. Combo classes: `.is-reversed`, `.is-1`, `.is-active`
- Combo classes always scoped: `.hero_card_wrap.is-reversed { }` not `.is-reversed { }`
- Combo classes must exist in the HTML — Webflow removes unused classes. If a combo class is only applied dynamically (e.g. JS toggling `.is-active`) **and has CSS styles defined for it** (e.g. `.tabs_link_wrap.is-active { ... }`), place an element with it inside a `[component]_hidden u-display-none` div. If the combo class has no CSS (JS only uses it for querying/targeting), no `_hidden` placeholder is needed. If a `_hidden` div already exists (for clone templates), reuse it _(Webflow only — in vanilla mode CSS-only combo classes are fine and need no `_hidden` placeholder)_
- Every element must have a component class — no bare `<span>`, `<div>`, or `<a>` with only a utility class
- **Reusable UI atoms get one shared class, reused everywhere — not a per-section copy.** Buttons, links, badges, tags, and other small repeated elements are defined **once** as a single shared component class (`button_wrap`, `link_wrap`, `badge_wrap`) and reused across every section — don't restyle the same thing under a fresh `hero_button` / `cta_button` class each time. Express differences as scoped `.is-*` variant combo classes (`.button_wrap.is-small`, `.card_wrap.is-reversed`); the variant must appear in the HTML (Webflow purges unused combos). (Buttons' colour variants are the `.u-button-style-*` mode utilities — see Buttons.) This applies only to small reusable atoms — layout/content components (hero, card, section) stay per-component
- Interactive elements (`<a>`, `<button>`) that act as component roots must end in `_wrap`
- **Links and buttons only wrap — they never hold text directly.** The interactive element is the `_wrap`, with a child text `<div>` inside: `<a class="button_wrap"><div class="button_text">Label</div></a>`. Padding, `background-color`, `border`, and `color` live on `_wrap`; the text child **inherits** the color, so `_text` never re-sets it — `_text` carries only text styling (size/weight/letter-spacing) and animation. Exceptions: inline links inside a paragraph stay `<a>text</a>` (can't nest a `<div>` in prose), and a real `<button>` element takes a `<span>` child instead of `<div>` (button content must be inline)
- Any element containing children with component classes must end in `_wrap`
- SVG `<path>` and `<line>` elements need their own component class for stroke styling — named as siblings to the SVG, not nested: `enterprise_button_path` not `enterprise_button_svg_path`
- SVG inside a subcomponent starts a new subcomponent name: `enterprise_button_svg` not `enterprise_button_arrow_svg`
- Decorative SVGs need `aria-hidden="true"`
- Direct parents of text elements should not be `display: flex` — flex prevents margin collapsing. Use `display: block` or no display override
- Buttons must be wrapped in a div with `u-button-group`: `<div class="hero_actions u-button-group">`

### Sections & Containers

```html
<section class="[name]_wrap u-section">
  <div class="[name]_contain u-container">
    <div class="[name]_layout">...</div>
  </div>
</section>
```

- `u-section`: `display: flex; flex-flow: column; background-color: var(--_theme---background); color: var(--_theme---text); padding-top/bottom: var(--_spacing---section-space--main)`
- `u-container`: `max-width: var(--max-width--main); width: calc(100% - var(--site--margin) * 2); margin-inline: auto; flex: 1; container-type: inline-size; container-name: threshold-large threshold-medium threshold-small` — centers and bounds the content width and is the named responsive container (hosts all three thresholds). It has no `display`/`gap` of its own; put layout on the `_layout` child
- **Never apply layout directly on `u-container`** — it has `container-type: inline-size`, so `@container` rules affect its children, not itself. Always use a child `_layout` div:
  ```css
  /* CORRECT — `u-grid-above` on the _layout div supplies the grid; just set the column count */
  .hero_layout.u-grid-above {
    --_column-count---value: 12;
  }
  /* WRONG — layout on the container itself */
  .hero_contain {
    display: grid;
    grid-template-columns: repeat(12, minmax(0, 1fr));
  }
  ```
- **The site header is its own top-level component, separate from every section.** Place `<header class="header_wrap">` as a direct child of `page_wrap`, a sibling of the `<section>` elements (normally the first child) — never inside a `u-section` and never wrapping one. The top-level `<header>` is the `banner` landmark, so the primary navbar lives **inside** it as a `<nav>` (a navbar is introductory/navigational content — it belongs in `<header>` for correct semantics). The header doesn't take `u-section` (it's a bar, not a content section), but use an inner `header_contain u-container` for content width:
  ```html
  <body class="page_wrap">
    <header class="header_wrap">
      <div class="header_contain u-container">
        <a class="header_logo_wrap" href="/">...</a>
        <nav class="header_nav_wrap">...</nav>
      </div>
    </header>
    <section class="hero_wrap u-section">...</section>
    <section class="features_wrap u-section">...</section>
  </body>
  ```
  This is for the primary site nav only — secondary navs (footer nav, in-page table of contents, breadcrumbs) are standalone `<nav>` and don't need a `<header>`.
- First section: use `--_spacing---section-space--page-top` for nav offset

### Layout

- Grids use `u-grid-above` (or `u-grid-below`) on the `_layout` div — the utility provides `display: grid`, the `--site--gutter` gutters, and `flex-flow: column`. Set the column count with `--_column-count---value` on the combo class (`.hero_layout.u-grid-above { --_column-count---value: 12; }`); override `grid-column-gap`/`grid-row-gap` there if needed. Don't write `display: grid` or `grid-template-columns` by hand. (The page default is 12, set on `page_wrap`; the base mode is 1. Setting the value per component registers that number as a column-count mode on a Webflow import.)
- **Replicating an asymmetric split (e.g. 8/4, 7/5, 9/3) is a 12-column grid + spans, not a 2-column grid.** Don't set `--_column-count---value: 2` and fight it into uneven proportions — set `--_column-count---value: 12` on the `_layout` and give each child `grid-column: span N` (spans summing to 12: `span 8` + `span 4`, etc.). This is more flexible than forcing two equal columns and stays responsive for free: below the collapse threshold (62em) `u-grid-above` flips to `display: flex !important; flex-flow: column`, and a flex container ignores `grid-column` entirely — so children stack full-width with no `@media` override needed. Author the desktop span once; mobile stacking is automatic
- **A section whose content is indented (not flush with the full container width) is still a 12-column `u-grid-above` wrapper — use `u-column-span-indent`.** When a design's content band is narrower than the container (e.g. content is 1115 of a 1440 container), don't hand-roll extra padding on the `_contain`/`_layout` — wrap in the normal 12-column grid and give the inner content `u-column-span-indent` (an absolute `2 / -2` span, ~10 of 12 columns). The indented block can itself be `u-grid-above` again for its own children (nesting is fine)
- **A list/card group with a fixed, known item count should size by content (`height: auto`), not a fixed height plus `flex: 1 0 0` children.** Forcing a fixed container height and letting flex children fill it with `flex: 1 0 0` looks fine at the authored breakpoint but breaks under reflow — a flex-basis child with `min-height: 0` (needed to prevent overflow) can collapse or vanish once the container's fixed height no longer fits the content at a narrower width. If the group's item count doesn't change, just let the container grow with its content and skip the fixed height and the `@media` overrides it would otherwise need — it's safe at every breakpoint automatically
- Grid columns: always `minmax(0, 1fr)` — never bare `1fr`: `repeat(2, minmax(0, 1fr))` not `1fr 1fr`
- **Place grid children with the placement utilities, not hand-written `grid-column`/`grid-row`.** Column start + span compose: `u-column-start-1…12` (start line, plus `u-column-start-auto`) with `u-column-span-1…12` (span N columns) — e.g. `u-column-start-3 u-column-span-4` starts at column 3 and spans 4. `u-column-span-full` (`1 / -1`) and `u-column-span-indent` (`2 / -2`) are absolute spans. Rows use `u-row-start-1…6` + `u-row-span-1…6`. Apply them on the grid child (a direct child of the `u-grid-*` `_layout`)
- If you ever set `display: grid` manually (outside `u-grid-*`), it must also have `flex-direction: column` so it stacks when collapsed — but prefer `u-grid-above`/`u-grid-below`, which already include this
- Direct grid children need `width: 100%`
- Content blocks: `width: 100%` + `align-self: start|center|end` for vertical alignment
- Two-column grids: `grid-row-gap: var(--_spacing---space--8)`
- DOM order + the `u-column-start-*`/`u-row-start-*` utilities for positioning — avoid `order`
- `@container` is for the `u-grid-*` system only; minimize custom container-query code (the grid collapse is automatic). Use Webflow `@media` breakpoints for non-grid responsiveness (see Responsive)

### Sizes & Spacing

- Spacing: `--_spacing---space--1` through `--8`. `space--1`/`space--2` too small for text — use `space--3`+
- Section padding: `--_spacing---section-space--none|small|main|large|page-top`
- Container widths: `--max-width--small: 50rem`, `--max-width--main: 90rem`, `--max-width--full: 100%`
- `--site--margin`, `--site--gutter`
- `--radius--small`, `--radius--main`, `--radius--round`
- `u-margin-trim` removes first child margin-top / last child margin-bottom
- Gap for lists/columns, margins for text elements
- Flex containers with multiple children should almost always have `gap` using a spacing variable — missing gap is a common oversight
- Prefer the `u-gap-*` utility (`u-gap-4`, `u-gap-gutter`, …) on flex/grid containers over writing `gap` in component CSS: it applies the gap **and** registers the value as a mode of the gap collection on a Webflow import. (Same idea for the other mode collections — `u-theme-*`, `u-text-style-*`, `u-button-style-*`, `u-alignment-*` export their modes only when the class is on a real element; see `references/webflow-variable-naming.md`.)

### Typography

- Headings: `u-text-style-display`, `u-text-style-h1`–`h6`
- Paragraphs: `u-text-style-large`, `u-text-style-main`, `u-text-style-small`
- Tag (`h1`–`h6`) is semantic; utility controls visual size
- Font weight: use `--_typography---font--primary-regular`, `--_typography---font--primary-medium`, or `--_typography---font--primary-bold` — never raw numeric weights like `400`, `500`, `700`
- Letter spacing: use `--_typography---letter-spacing--tight` (-0.03em) — never raw values like `-0.03em` or `-1px`
- Space text vertically with `gap` on the flex `_content` (a spacing variable), or `margin-bottom: var(--_text-style---margin-bottom)` on each text element when `_content` is a block
- A block `_content` that relies on text margins should have `u-margin-trim` (drops the first child's top / last child's bottom margin); a flex `_content` spaced with `gap` doesn't need it
- **Headings & paragraphs use the `u-text-style-*` utility directly on the element** for sizing — `<h1 class="hero_title u-text-style-h1">`, `<p class="hero_text u-text-style-main">`. Don't wrap them. (`c-heading`/`c-paragraph` and `u-heading`/`u-text` are reserved for rich-text fields and the Webflow heading/paragraph component — never for direct headings/paragraphs.)
- **Limit line length with a `u-max-width-*ch` utility applied directly on the text element** (`u-max-width-20ch` … `u-max-width-80ch`) — `<p class="hero_text u-text-style-small u-max-width-40ch">`. Never write a `max-width` for text in CSS, and **don't introduce a wrapper just to constrain width** — the utility works directly on the heading/paragraph itself, no flex-column child structure needed for the limit
- **The flex-column `_content` is only for spacing and/or centering multiple stacked text elements — not for the width limit.** When several text elements (and maybe a `u-button-group`) stack, make `_content` a **flex column** spaced with `gap`, and **center by adding `u-alignment-center` to `_content`** (it sets `text-align`, `justify-content`, and `align-items` to center) — the width-limited text then centers as a flex child:
  ```html
  <div class="hero_content u-alignment-center">
    <h1 class="hero_title u-text-style-h2 u-max-width-20ch"></h1>
    <p class="hero_text u-text-style-small u-max-width-40ch"></p>
  </div>
  ```
  ```css
  .hero_content {
    display: flex;
    flex-flow: column;
    gap: var(--_spacing---space--4);
  }
  ```
  For a left-aligned block, omit `u-alignment-center`. Omit `u-max-width-*ch` when no line limit is wanted. A single text element that just needs a line-length cap gets the utility directly — no `_content` wrapper at all
- Reduced-opacity text: `color: color-mix(in srgb, currentColor 80%, transparent)` — adjust percentage as needed. Never use the `opacity` property to fade text

### Buttons

- **Use `<a>` for buttons, not `<button>` — reserve `<button>` for real form submit controls only.** Any button-like element (link, CTA, dropdown toggle, JS-action trigger) is an `<a class="button_wrap">`; only an actual form submit uses `<button type="submit">`. This is why the text-wrapper rule notes `<button>` needs a `<span>` child — that case is rare. A JS-action `<a>` should keep keyboard/focus behavior (it already gets focus styles; add `role`/`aria-*` for non-navigation triggers like dropdowns)
- **One shared, reusable button class — `button_wrap` — defined once and used for every button on the site.** Don't create per-section button classes (`hero_button`, `cta_button`); reuse `button_wrap` and add `.is-*` variants for differences. No button utility — style on the `button_wrap` class with `data-trigger="hover focus"`
- Buttons must always include padding — default: `padding: var(--_spacing---space--3) var(--_spacing---space--5)`
- **All button colors live on `button_wrap`, never on `button_text`.** `background-color`, `color`, and `border` are set on the `_wrap` (see the base CSS below); the `_text` child inherits `color`, so re-declaring it there is redundant and breaks theme/trigger inheritance. `_text` only carries size/weight/letter-spacing
- Buttons read the **`--_button-style---*` alias** (`background`, `text`, `border`, plus each `*-hover`), which defaults to the primary button theme vars and re-resolves per theme class. Write the `color-mix` trigger declarations **once** on the base; color variants just remap the alias (below) — no need to rewrite the `color-mix`. Always `color-mix(in srgb, …)` (the foundation's color space), `--_trigger---on` first, `--_trigger---off` second:
- Base (primary):
  ```css
  .button_wrap {
    padding: var(--_spacing---space--3) var(--_spacing---space--5);
    background-color: color-mix(
      in srgb,
      var(--_button-style---background) calc(100% * var(--_trigger---on)),
      var(--_button-style---background-hover) calc(100% * var(--_trigger---off))
    );
    color: color-mix(
      in srgb,
      var(--_button-style---text) calc(100% * var(--_trigger---on)),
      var(--_button-style---text-hover) calc(100% * var(--_trigger---off))
    );
    border-color: color-mix(
      in srgb,
      var(--_button-style---border) calc(100% * var(--_trigger---on)),
      var(--_button-style---border-hover) calc(100% * var(--_trigger---off))
    );
    border-width: var(--border-width--main);
    transition: all 300ms;
  }
  ```
- **Color variants are `.u-button-style-*` mode utilities** — `u-button-style-secondary`, `-tertiary`, `-quaternary` (the base `button_wrap` is primary). The utility remaps `--_button-style---*`, so the base `color-mix` declarations resolve to the new colors automatically — and on a vanilla→Webflow import it registers as a **mode** of the button-style collection (so keep it on the button element). No component CSS needed for the color; the `.u-button-style-*` classes live in the foundation. Other (non-color) variants stay scoped `.is-*` combos:
  ```css
  /* Size variant — only overrides padding */
  .button_wrap.is-small {
    padding: var(--_spacing---space--2) var(--_spacing---space--4);
  }
  ```
- **Pick a button style with contrast against the SECTION's theme.** The style modes are theme-relative: in the light theme `u-button-style-tertiary` / `-quaternary` resolve to white background + white border + white text (they exist for dark sections) — on a light section they're invisible. For an outline button on a light section use `u-button-style-secondary` (transparent background + theme border + theme text). Always sanity-check each button's contrast against the theme of the section it sits in.
- **A one-off button variant's combo selector must include the FULL class chain, in order — never skip the utility-mode class sitting between the base and the combo.** A button with a bespoke look (e.g. a hero CTA whose border wraps only the text, not the icon) still carries all three classes — `<a class="button_wrap u-button-style-secondary hero_cta_wrap">` — and the CSS must be written as `.button_wrap.u-button-style-secondary.hero_cta_wrap { }`, not `.button_wrap.hero_cta_wrap { }`. On the HTML-to-Webflow import, the tool matches authored selectors against the literal class stack to reconstitute the combo; a selector that skips the middle utility class doesn't match any real element, and the override is silently dropped on import even though it renders correctly in the browser (browser cascade doesn't care about the gap — the import tool does). The same applies to a bare combo child with no base class: `.hero_cta_text { }` should be `.button_text.hero_cta_text { }`. Rule of thumb: the CSS selector = the element's full `class="…"` attribute, in the same order, every time you scope a combo — never a subset.
- Usage — reuse the same class, vary with combos, inside the `u-button-group`:
  ```html
  <div class="hero_actions u-button-group">
    <a class="button_wrap" href="#" data-trigger="hover focus">
      <div class="button_text">Primary</div>
    </a>
    <a class="button_wrap u-button-style-secondary" href="#" data-trigger="hover focus">
      <div class="button_text">Secondary</div>
    </a>
  </div>
  ```

### Dropdowns & menus

- Build a dropdown/menu as a normal accessible component (best practice) — **don't use Webflow's native `.w-dropdown`**. One component class per part (wrapper/toggle/list), `data-*` hooks for JS targeting, and toggle the open state with `.is-active` (read in CSS via the trigger/state system). The toggle is an `<a>` (or `<button type="button">`) with `aria-expanded` + `aria-controls`; the menu is a list. Keyboard: Enter/Space toggles, Esc closes, arrows move between items; close on outside click

### Color & Theming

- **Light is the theme collection's base mode** (the `:root` values) — so `u-theme-light` is never needed to *get* light; it only flips a section back to light under a dark/brand page
- **Theme cascades from `page_wrap` — don't put a theme class on every section.** The page's default theme lives on `page_wrap`: `<body class="page_wrap">` is light (the base mode — add nothing), while a dark- or brand-default design adds the combo class there: `<body class="page_wrap u-theme-dark">`. Every section/component inherits the `--_theme---*` vars automatically
- **Add `u-theme-*` to a section/card only to OVERRIDE the page default** — a section whose theme differs from the rest
- **Choosing the right theme**: `u-theme-light` for light/white backgrounds, `u-theme-dark` for dark/black, `u-theme-brand` for colored (green, blue, etc.). The header/nav inherits the page default like everything else — give it its own `u-theme-*` only when its visual context differs
- Theme variables: `--_theme---background`, `--_theme---text`, `--_theme---border`, `--_theme---background-2` (lighter shade of the section background — use for pill buttons, tags, badges, nav buttons, or any element needing subtle contrast)
- Links: `--_theme---text-link--border|text|border-hover|text-hover`
- Buttons: read the `--_button-style---*` alias (defaults to `--_theme---button-primary--*`); switch style by applying a `u-button-style-*` mode utility (`secondary`/`tertiary`/`quaternary`), which remaps the alias and, on Webflow import, registers as a button-style mode. **Buttons MUST always use the button variables for all colors (background, text, border)** — never `--_theme---text`/`--_theme---background` or the inverted-colors technique. The button variables are already configured per theme to produce the correct visual result
- **Don't set `color` on headings or paragraphs** unless the text color differs from the section's inherited `--_theme---text`
- **Inverted colors for decorative elements only**: when SVGs, icons, badges, or other non-interactive decorative elements use colors without a specific theme variable, invert the section's theme — `--_theme---text` for the element's background/fill, `--_theme---background` for foreground elements on top. **Never apply this to buttons** — buttons always use `--_theme---button-primary--*` or `--_theme---button-secondary--*` regardless of their visual appearance

### Responsive

**Two systems, split by element.** The container setup is unchanged either way — `u-container` (and `-small`/`-full`) keeps `container-type: inline-size; container-name: threshold-large threshold-medium threshold-small`.

1. **The `u-grid-*` grid system stays breakpointless (container queries).** `u-grid-above` / `u-grid-below` and their threshold companions `u-order-unset-*` / `u-all-unset-*` collapse and reset at the named threshold container queries — `threshold-large` (62em), `threshold-medium` (48em), `threshold-small` (30em) — reacting to the **container's** width. This is the *only* place the breakpointless system is used.
2. **Every other responsive change uses Webflow breakpoints (`@media`).** For anything that isn't the grid collapse — flex-direction, alignment, gap, sizing, show/hide, font-size, position, a non-grid reorder — write a `@media` query in that element's own component CSS at one of Webflow's four breakpoints (desktop-first, `max-width`, px to match Webflow exactly):

| Breakpoint | Media query | Applies |
|---|---|---|
| **Desktop** | *(none — base styles)* | the base; author the desktop layout here |
| **Tablet** | `@media screen and (max-width: 991px)` | ≤ 991px |
| **Mobile landscape** | `@media screen and (max-width: 767px)` | ≤ 767px |
| **Mobile portrait** | `@media screen and (max-width: 479px)` | ≤ 479px |

Desktop-first: the base (no query) is the desktop layout; override downward. Each smaller breakpoint inherits from the one above unless it overrides.

```css
/* Base = desktop */
.enterprise_header {
  display: flex;
  align-items: center;
}
/* Tablet and down */
@media screen and (max-width: 991px) {
  .enterprise_header {
    flex-direction: column;
    align-items: start;
  }
}
```

**Why two systems.** On a vanilla → Webflow import, `@media` queries written at Webflow's exact breakpoints map onto the Designer's **native breakpoint styles**, so they stay editable in the Designer like a hand-built Webflow site. Container queries have no Designer-breakpoint equivalent, so they land in a custom embed's CSS — far harder to maintain. The grid genuinely needs to react to its container's width (not the viewport), so it keeps container queries; everything else uses Webflow media queries so it imports cleanly into the breakpoint system.

#### Grid → stack (the breakpointless case)

Add `u-grid-above` to the `_layout` div. The utility supplies `display: grid`, the gutters (`--site--gutter`), and `flex-flow: column` — you only set the **column count** via `--_column-count---value` on the component combo class. Below the large threshold (62em) the container query collapses it to a vertical flex stack:

```html
<div class="hero_layout u-grid-above"></div>
```

```css
.hero_layout.u-grid-above {
  --_column-count---value: 2;
  /* optional — override the default gutters: */
  grid-column-gap: var(--_spacing---space--6);
  grid-row-gap: var(--_spacing---space--6);
}
```

Never write `display: grid` or `grid-template-columns` yourself — `u-grid-above` already does. `u-grid-below` is the inverse (stacked above the breakpoint, grid below) — rare. `u-order-unset-above`/`-below` drop a custom `order`, and `u-all-unset-above`/`-below` reset every property, above / below the threshold.

#### Rules

- **Webflow breakpoints are px** (`991` / `767` / `479`) — the one place px is correct, since they must match Webflow exactly. Sizing/layout *inside* a query still uses `rem`/`em`/`ch`, never px.
- The grid's container thresholds stay in **em** (`62` / `48` / `30em`); they fall at the same viewport boundaries (992 / 768 / 480px) but react to container width.
- **Use only these four breakpoints** — don't invent others. To vary a `u-grid-*` column count per breakpoint, set `--_column-count---value` inside a threshold container query (it's part of the grid system); everything else uses the Webflow `@media` breakpoints.
- For a finer, component-local query, an unnamed `@container (width < Xem)` against the nearest `container-type: inline-size` ancestor still works — but prefer Webflow `@media` for non-grid responsiveness so it imports to the Designer.

### Trigger & State System

Flips CSS variable values on a parent so children react without descendant selectors.

- **The manager block lives in the foundation global CSS — never per-component.** The `[data-state]` / `[data-trigger]` selectors that flip `--_state---*` / `--_trigger---*` (`.is-active`, the `data-state` listeners, and the hover/focus/mobile `data-trigger` rules) ship in `lumos-foundation.css` in vanilla mode and are part of the Lumos global style in Webflow. Components only **read** the flipped variables; they never redefine the manager. If that block isn't present in the global CSS, `data-state`/`data-trigger`/`.is-active` do nothing — the variables never flip

- `--_state---true: 1` / `--_state---false: 0` → flip when activated (`.is-active`, `data-state` match)
- `--_trigger---on: 1` / `--_trigger---off: 0` → flip on hover/focus (`data-trigger`)
- **These are the RESTING values; the manager flips them when active/hovered.** At rest `on`/`true` = 1 and `off`/`false` = 0 — so in `color-mix(A calc(100% * var(--_trigger---on)), B calc(100% * var(--_trigger---off)))` the **first term (A) is the resting appearance and the second (B) applies on hover/active** (the button base sets `--_button-style---background` first = rest, `--_button-style---background-hover` second = hover). For state likewise: the active look is multiplied by `--_state---false`, the resting look by `--_state---true`. Don't misread the defaults as "the value when active" — that inverts hover/rest.
- **Hidden by default, shown when active** → `var(--_state---false)`
- **Shown by default, hidden when active** → `var(--_state---true)`
- **If a toggled panel uses the native `hidden` attribute instead of the state-variable pattern above, its specificity is only 0,1,0 (an attribute selector) — a component rule setting `display` on that same element (e.g. `.tabs_panel { display: flex; }`) beats `[hidden] { display: none; }` at equal or higher specificity and the panel never actually hides.** Either drive show/hide entirely through `--_state---*` (preferred — no specificity fight exists), or if `hidden` is kept for its accessibility semantics, scope the override as `.tabs_panel[hidden] { display: none; }` so it wins
- `data-state` listeners: `checked` (`:checked`), `current` (`.w--current`), `open` (`.w--open`), `pressed` (`aria-pressed`), `expanded` (`aria-expanded`), `external` (`target="_blank"`)
- JS-driven interactives (tabs, sliders, accordions): toggle `.is-active`, not `data-state`
- `data-trigger` types: `hover`, `focus`, `preview` (forces the "on" state in the Webflow Designer preview so you can style it), `mobile`, `group`, `hover-other`, `focus-other`, `hover-if-clickable`
- **Never select `[data-state]`, `[data-trigger]`, or `.is-active` in CSS** — read variable values only:

  ```css
  /* CORRECT */
  .tabs_link_text {
    opacity: var(--_state---false);
  }
  .tabs_link_bar {
    transform: scaleY(calc(var(--_state---false)));
    transition: transform;
  }

  /* WRONG */
  .tabs_link.is-active .tabs_link_text {
    opacity: 1;
  }
  ```

- Don't wrap single variable in `calc()`: `opacity: var(--_state---false)` not `opacity: calc(var(--_state---false))`
- Ordering: `true`/`on` always first, `false`/`off` second in `color-mix()` and `calc()`. `color-mix()` uses the `in srgb` color space
- Patterns:
  ```css
  opacity: calc(1 - 0.4 * var(--_trigger---on));
  transform: scale(calc(1 + 0.2 * var(--_trigger---on)));
  transform: rotate(calc(45deg * var(--_trigger---on)));
  transform: translateX(calc(2rem * var(--_trigger---on)));
  ```

### Scalable Visual Compositions

For graphical/decorative visuals where elements float freely (overlapping cards, badges, images — not aligned on a grid). Scales proportionally like a single image. **Only for graphical elements, not entire sections.**

Structure: `_visual_wrap` → `_visual_inner` → absolutely positioned children.

```html
<div class="hero_visual_wrap">
  <div class="hero_visual_inner">
    <div class="hero_visual_card"></div>
    <div class="hero_visual_badge"></div>
  </div>
</div>
```

```css
.hero_visual_wrap {
  container-type: inline-size;
  width: 100%;
  aspect-ratio: 3 / 2;
  position: relative;
}

.hero_visual_inner {
  display: flex;
  justify-content: center;
  align-items: center;
  font-size: 1cqw;
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
}

/* Centered child: no top/left/bottom/right, offset from center with transform */
.hero_visual_card {
  position: absolute;
  transform: translateX(-10em) translateY(-3em);
  width: 30em;
  aspect-ratio: 4 / 2;
}

/* Corner-anchored child: anchor to a corner, offset with transform */
.hero_visual_badge {
  position: absolute;
  bottom: 0;
  right: 0;
  transform: translateX(-5em) translateY(-5em);
  width: 20em;
  aspect-ratio: 1 / 1;
}

.hero_visual_badge_text {
  font-size: 2em;
}
```

Key rules:

- `_visual_wrap`: `container-type: inline-size`, `position: relative`, `width: 100%`, `aspect-ratio` to define the canvas. Can use `max-width` if needed
- **The bounding box must tightly hug the content.** The `aspect-ratio` and any `max-width` on `_visual_wrap` should be sized so elements reach or nearly reach all edges — no large empty margins inside the composition. If elements only occupy the center, the aspect ratio is too tall/wide. Adjust `aspect-ratio`, element positions, and sizes until the outermost elements are close to the edges of the `_visual_wrap`. Think of the bounding box as shrink-wrapped around the content
- `_visual_inner`: `display: flex; justify-content: center; align-items: center` and `font-size: 1cqw` — the only place `cqw` appears. Everything else uses `em`
- Children use `position: absolute` positioned one of two ways:
  - **Centered** (default): omit `top`/`left`/`bottom`/`right` — centers via flex. Offset with `transform` in `em`
  - **Corner-anchored**: set one corner pair (`top: 0; left: 0`, `top: 0; right: 0`, `bottom: 0; left: 0`, or `bottom: 0; right: 0`), offset with `transform` in `em`
- **Never use percentage values** (`top: 50%`, `left: 30%`) — offsets are exclusively `transform` in `em`
- **Only wrap elements that have both a custom `font-size` AND need a `transform`** — because `em` in transforms resolves against the element's own `font-size`, not the parent's `1cqw`. Most children (images, cards, shapes) can have `position`, `transform`, and `width` directly — no extra wrapper needed. The wrapper is only for text elements where you set a `font-size` like `3em` and also need positional offset:

  ```css
  /* Needs a wrapper — has font-size AND transform */
  .hero_visual_heading_wrap {
    position: absolute;
    top: 0;
    left: 0;
    transform: translateX(5em) translateY(2em);
  }
  .hero_visual_heading {
    font-size: 3em;
  }

  /* No wrapper needed — no custom font-size */
  .hero_visual_card {
    position: absolute;
    transform: translateX(5em) translateY(2em);
    width: 30em;
  }
  ```

- Sizes (`width`, `aspect-ratio`), font sizes, and `border-radius` on children are in `em` — not `--radius--*` variables, which don't scale with the composition
- **No container queries by default** — because everything is in `em` relative to `1cqw`, the entire composition scales down proportionally as the container shrinks. Don't add `@container` rules to resize, reposition, or hide children at smaller sizes unless the user specifically asks for a different responsive behavior

## Centering

Never use `top: 50%; left: 50%; transform: translate(-50%, -50%)`. Instead, use flex on the parent and `position: absolute` on the child with no `top`/`left`/`bottom`/`right`:

```css
.parent {
  display: flex;
  justify-content: center;
  align-items: center;
}
.child {
  position: absolute;
}
```

This applies everywhere, not just visual compositions.

## Example

```html
<section class="hero_wrap u-section">
  <style>
    .hero_layout.u-grid-above {
      --_column-count---value: 2;
    }
    .hero_title.u-text-style-h1 {
      margin-bottom: var(--_text-style---margin-bottom);
    }
    .hero_text.u-text-style-main {
      margin-bottom: var(--_text-style---margin-bottom);
    }
  </style>
  <div class="hero_contain u-container">
    <div class="hero_layout u-grid-above">
      <div class="hero_content u-margin-trim">
        <h1 class="hero_title u-text-style-h1"></h1>
        <p class="hero_text u-text-style-main"></p>
      </div>
      <div class="hero_visual"></div>
    </div>
  </div>
</section>
```

## Anti-patterns

- `px` anywhere, `vw` for fonts
- `@media` at non-Webflow breakpoints — only Webflow's four (`991`/`767`/`479px` max-width, plus desktop base) are allowed; never invent custom breakpoints. Feature queries like `@media (hover: none)` are fine
- `:hover`, `:focus`, or any interactive pseudo-class in CSS — use `data-trigger` and `--_trigger---on`/`--_trigger---off`
- `.is-active`, `[data-state]`, `[data-trigger]` in CSS selectors
- The `--_responsive---*` / `--flex-medium` keyword-variable system — removed; use the `u-grid-*` container-query system for the grid and Webflow `@media` breakpoints for everything else
- Container queries for non-grid responsiveness — those land in a custom embed's CSS on Webflow import (hard to maintain); reserve `@container` for `u-grid-*` and use Webflow `@media` breakpoints elsewhere
- `@keyframes` with state/trigger selectors
- Fallback values in `var()`
- `false`/`off` before `true`/`on` in expressions
- Unscoped combo classes
- Bare `.component { }` CSS for an element that also has a utility class — scope it as the combo `.component.u-utility` so the component's declarations win over the utility
- Grid columns with bare `1fr` — always `minmax(0, 1fr)`
- `display: grid` or layout on `u-container` — use a child `_layout` div
- Site header/navbar nested inside a `<section>` or `u-section` — the `<header>` is a separate top-level component, a sibling of the sections under `page_wrap`
- Primary navbar as a bare top-level `<nav>` instead of wrapped in a top-level `<header>` (the banner landmark)
- Hand-writing `display: grid` / `grid-template-columns` on a `_layout` — use `u-grid-above`/`u-grid-below` and set `--_column-count---value` on the combo class
- Hand-writing `grid-column`/`grid-row` placement on a grid child — use `u-column-start-*`/`u-column-span-*` and `u-row-start-*`/`u-row-span-*` (they compose: start line + span)
- Hex color codes (`#ff0000`, `#333`) anywhere — CSS, comments, or prose. Never reference hex values
- Hardcoded colors (`white`, `black`, etc.) or border widths — always use `--_theme---*` variables
- `color` on headings/paragraphs that matches the section's inherited `--_theme---text` — redundant
- Hardcoded button colors — use `--_theme---button-primary--*` or `--_theme---button-secondary--*`
- `opacity` property to fade text — use `color-mix(in srgb, currentColor [%], transparent)`
- `innerHTML`, `createElement`, or template literal HTML in JS — use the `_hidden` clone pattern
- `getElementById`, `id` attributes, or JS targeting by `id`
- Selecting elements by class in JS (`wrap.querySelector(".tabs_link")`) — target by a `data-*` attribute instead (`[data-tabs-link]`); only the component wrap is selected by class, as the scoping anchor
- In vanilla mode: scattered `DOMContentLoaded` listeners or per-component `dataset.scriptInitialized` guards — collect inits into one `initFunction()` called once on a single `DOMContentLoaded`
- Combo classes in CSS but not in the HTML — Webflow purges them; put them in a `_hidden u-display-none` div _(Webflow only)_
- Text elements missing `margin-bottom: var(--_text-style---margin-bottom)`
- Text parent missing `u-margin-trim`
- Hardcoding `max-width: Nch` (or any `max-width`) on text — use a `u-max-width-*ch` utility instead
- Empty divs without `padding: 0` _(Webflow only)_
- Buttons without padding
- Per-section button/link classes (`hero_button`, `cta_button`) that restyle the same atom — reuse one shared `button_wrap`/`link_wrap` class with `.is-*` variants
- `<button>` for a link, CTA, or dropdown toggle — use `<a>`; reserve `<button>` for real form submit controls
- Link/button holding text directly (`<a class="button_wrap">Label</a>`) instead of wrapping a `_text` div — except inline prose links
- `color` (or `background-color`/`border`) set on `button_text` instead of `button_wrap` — the `_wrap` owns all button colors; the `_text` inherits `color`
- Constraining text width with a `max-width` in CSS or an extra wrapper instead of `u-max-width-*ch` directly on the text element
- Relying on a third-party library's served CSS for a component's appearance — author the styles yourself so they export to the Webflow Designer. Renaming only the library's top-level/slide classes and leaving a deeper hook (an injected sub-element, a disabled/locked state class) still on the library's literal class name and its CSS
- `aspect-ratio` on an element with no explicit `width`/`height` alongside it — Safari won't compute it correctly
- A combo selector that skips a utility-mode class sitting between the base class and the combo (e.g. `.button_wrap.hero_cta_wrap` when the element also carries `u-button-style-secondary`) — it beats the utility in the browser but the Webflow import tool won't recognize it as matching the element's real class stack, and drops the style
- Scoping a base rule to a combo (`.component.u-utility`) without re-scoping its `@media` overrides to the same combo — the breakpoint override silently stops firing once the base rule outranks it
- Scoping only the overlap you already suspect — audit every property a co-occurring utility sets, not just the visually-obvious one
- Native `hidden` attribute fighting a component's own `display` declaration on the same element — scope as `.component[hidden]` or drive show/hide through the state system instead
- Bare `img` sized directly instead of a relative `_img_wrap` wrapper + absolutely-filled `img`
- `<img>` missing an `alt` attribute, or shipping a placeholder `alt="..."` — every image needs real alt text (descriptive, or empty `alt=""` + `aria-hidden="true"` for decorative); other meaningful assets need an accessible name too
- Per-component `text-decoration: none` on links to kill the underline — the foundation already resets `a { text-decoration: none }` (only `a:not([class])` rich-text stays underlined); stripping it link-by-link is whack-a-mole and misses elements like the logo
- `<style>` not first child or `<script>` not last child inside `_wrap` _(Webflow only — vanilla mode keeps component CSS/JS in `css/styles.css` and `js/main.js`, not inline)_
- Percentage values on `top`/`left`/`bottom`/`right` for positioning — use corner anchors with `0` and `transform` in `em`
- `top: 50%; left: 50%; transform: translate(-50%, -50%)` — use flex centering on parent
- `transform` with `em` on an element with its own `font-size` in visual compositions — wrap in a positioning div
