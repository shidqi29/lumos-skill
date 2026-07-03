# Vanilla Mode — building a Lumos project from scratch

Read this when the user wants Lumos conventions in a plain HTML/CSS/JS project (no Webflow). The component-writing rules are unchanged from `SKILL.md`; this file only covers the project setup, the foundation, and what differs from Webflow.

## The idea

A Webflow Lumos project supplies the foundation for free — the reset, the design tokens, every `u-*` utility, and the responsive/trigger/state systems. A from-scratch project has none of that, so you ship `assets/lumos-foundation.css` once and then write component code exactly as you would in Webflow. The foundation makes every utility class and `--_*` variable the component code references actually resolve.

## Project skeleton

```
project/
├── index.html
├── css/
│   ├── lumos-foundation.css   ← copied from this skill's assets/, linked first
│   └── styles.css             ← all component styles
└── js/
    └── main.js                ← all component scripts
```

**Component CSS and JS live in external files, not inline.** The skill's per-component `<style>`/`<script>` blocks exist only so each component is a self-contained paste into the Webflow Designer — a vanilla project has real files, so that reason is gone. Collect every component's styles into `css/styles.css` (linked after the foundation) and every component's scripts into `js/main.js` (loaded once before `</body>`). Nothing else changes: class names, combo-class scoping, the `data-trigger`/`data-state` technique, and the JS init-guard/scoping pattern are all identical to Webflow mode — only the location of the CSS and JS differs.

## HTML boilerplate

`index.html` — markup only, no inline `<style>`/`<script>`:

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>Page title</title>
    <!-- Foundation FIRST, before any component styles -->
    <link rel="stylesheet" href="css/lumos-foundation.css" />
    <!-- Component styles AFTER the foundation -->
    <link rel="stylesheet" href="css/styles.css" />
  </head>
  <!-- Page default theme lives here: light needs nothing; a dark/brand-default
       design adds the combo class, e.g. <body class="page_wrap u-theme-dark"> -->
  <body class="page_wrap">
    <!-- Header (banner) wraps the primary nav; top-level, a sibling of the sections — never inside one -->
    <header class="header_wrap">
      <nav class="header_nav_wrap"></nav>
    </header>
    <!-- Sections inherit the page theme; add u-theme-* on a section only to override it -->
    <section class="hero_wrap u-section">
      <div class="hero_contain u-container">
        <div class="hero_layout u-grid-above">
          <div class="hero_content u-margin-trim">
            <h1 class="hero_title u-text-style-h1">Heading</h1>
            <p class="hero_text u-text-style-main">Body copy.</p>
          </div>
          <div class="hero_visual"></div>
        </div>
      </div>
    </section>
    <!-- Component scripts, loaded once at the end -->
    <script src="js/main.js"></script>
  </body>
</html>
```

`css/styles.css` — every component's styles (what would have been the hero's inline `<style>`):

```css
.hero_layout.u-grid-above {
  --_column-count---value: 2;
}
.hero_title.u-text-style-h1 {
  margin-bottom: var(--_text-style---margin-bottom);
}
.hero_text.u-text-style-main {
  margin-bottom: var(--_text-style---margin-bottom);
}
```

`js/main.js` — each component that needs JS gets a named init function, all collected into one `initFunction()` run once on `DOMContentLoaded` (full rules in `SKILL.md`). The hero above needs no JS, so `main.js` stays empty until a component does; the shape is:

```js
const initHeroWrap = () => {
  const wrap = document.querySelector(".hero_wrap");
  if (!wrap) return;
  // wrap.querySelector(...) — scope every query to the component
};

const initFunction = () => {
  initHeroWrap();
  // ...one call per component; font/layout-dependent inits go in document.fonts.ready.then()
};

document.addEventListener("DOMContentLoaded", () => {
  initFunction();
});
```

## Multi-page projects

The single-page skeleton above generalizes directly, with two changes once a project grows past one HTML file.

**One shared `global/` folder, never a per-page copy of the foundation or shared components:**

```
project/
├── global/
│   ├── lumos-foundation.css   ← ONE copy, shared by every page
│   └── global.css             ← shared components: header, footer, buttons, anything
│                                  reused across pages — linked by every page, second
├── home/
│   ├── index.html
│   ├── css/styles.css         ← Home's unique sections ONLY
│   └── js/main.js
├── about/
│   ├── index.html
│   ├── css/styles.css         ← About's unique sections ONLY
│   └── js/main.js
└── …
```

Every page's `<head>` links in this order — foundation, then shared components, then that page's own unique styles:

```html
<link rel="stylesheet" href="../global/lumos-foundation.css" />
<link rel="stylesheet" href="../global/global.css" />
<link rel="stylesheet" href="css/styles.css" />
```

A component used on more than one page (header, footer, buttons, a shared CTA banner) is styled **once** in `global.css` — never copy its rules into each page's own `styles.css`. A page's own `styles.css` holds only the sections unique to that page. This is the same "never redefine what's already provided" discipline the skill applies to the foundation itself, extended to shared components across pages.

**Name each page's entry function for that page, not a generic `initFunction`.** Each page still has exactly one entry point called once on `DOMContentLoaded` (the "One entry point" JS rule is unchanged — nothing shares across pages, since each page loads its own separate `main.js`), but name it for the page: `initHomeFunction`, `initAboutFunction`, not a bare `initFunction` repeated identically in every page's file. This makes it unambiguous which page's script you're looking at when several pages' files are open side by side, and avoids an identical top-level name across files that could otherwise be confused if scripts were ever bundled or compared:

```js
// about/js/main.js
const initAboutFunction = () => {
  initHeaderWrap();
  initFoundersWrap();
};

document.addEventListener("DOMContentLoaded", () => {
  initAboutFunction();
});
```

## The page wrapper (`page_wrap`)

Webflow can't paste a `<body>` tag — the HTML-to-Webflow import tool wraps the page in a `<div>` that inherits the body's class, so anything styled on the bare `body` selector is lost. Because of that, **all body-level styling lives on the `.page_wrap` class** in the foundation, never on `body`:

- base font, color, and background
- `min-height: 100vh` so the background fills the viewport

(The threshold responsive containers are **not** on `page_wrap` — they live on `u-container` so queries react to the column area's width; see Responsive.)

So wrap every section in a single `<body class="page_wrap">`. On import it becomes `<div class="page_wrap">` and every style carries over unchanged. Never add a `body { … }` rule; put page-level styles on `.page_wrap`.

## What differs from Webflow mode

These Webflow-only workarounds are **dropped** — they only exist to satisfy the Webflow Designer:

- **Inline `<style>`/`<script>` blocks are gone.** They exist only to make each component a self-contained paste for the Webflow Designer. In a vanilla project, component CSS lives in `css/styles.css` and component JS in `js/main.js` (see *Project skeleton* above). Class names, combo-class scoping, and the JS init-guard/scoping pattern are unchanged.
- **Combo classes can be CSS-only.** Webflow purges classes not present in the canvas, which is why dynamically-toggled combo classes (e.g. `.tabs_link_wrap.is-active`) normally need a `_hidden u-display-none` placeholder. In vanilla CSS nothing is purged, so define them directly and skip the placeholder. Still use the hidden-template clone pattern for JS-appended markup — that's a clean technique, not a workaround.
- **No empty-div `padding: 0`.** Webflow injects default padding into empty elements; the browser doesn't. Omit the fix unless you actually want padding.
- **No `.w-*` classes.** Anything keyed to `.w-richtext`, `.w-dyn-item`, `.w-condition-invisible`, `.wf-design-mode`, etc. is Webflow runtime only and has been stripped from the foundation.

Everything else is identical: class naming, fluid `rem`/`ch`/`em` sizing, the dual responsive system (the breakpointless `u-grid-*` container-query grid + Webflow `@media` breakpoints for every other element — see *Responsive* in `SKILL.md`), the `data-trigger`/`data-state` technique, scalable visual compositions, and the accessibility requirements.

## Customizing tokens

All brand values live in **section 2** of `lumos-foundation.css`. Keep the variable names; change the values:

- **Colors** — edit the `:root` theme block and the `.u-theme-light` / `.u-theme-dark` / `.u-theme-brand` classes. Set the page default theme on `page_wrap` (light needs no class; dark/brand → `u-theme-*` combo on `page_wrap`); add `u-theme-*` to a section only to override that default.
- **Spacing** — `--_spacing---space--1…8` and the fluid `--_spacing---section-space--*`.
- **Type scale** — the `.u-text-style-*` rules use `clamp()` for breakpointless sizing; adjust min/max as needed.
- **Fonts** — set `--_typography---family--primary` (and load the webfont in `<head>` if it isn't a system font). Weight tokens are `--_typography---font--primary-regular|medium|bold`.
- **Radii, borders, focus, nav height** — the remaining tokens in section 2.

**Keep the variable names exactly as-is.** They follow the Webflow Variables naming convention, so the next section works.

## Importing back into Webflow (variables)

A vanilla project built this way can be imported into Webflow with the HTML-to-Webflow tool, and its CSS custom properties come across as **native Webflow Variables** — correct collection, folder, variable, and mode — because the foundation's names follow the import convention:

- `--_<collection>---<name>` → a collection variable (`--_theme---background` → collection *theme*, variable *background*)
- `--` inside the name → a nested folder (`--_theme---button-primary--background` → `button primary/background`)
- `-` → a space; a name with no leading `_` → the base collection (`--site--gutter`)
- `.u-<collection-slug>-<mode>` classes (`u-theme-dark`, `u-text-style-h2`) → **variable modes**; defaults come from `:root`/`html`/`body`, and an override only registers if it differs from the base

So: don't rename collections, don't inline hardcoded values where a token exists, and keep applying the `u-theme-*` / `u-text-style-*` mode classes on real elements (a mode class absent from the HTML gets pruned on import). New tokens must use the same naming to import. Full rules and checklist: `references/webflow-variable-naming.md`.

**The Trigger & State manager block does not survive the import as native Webflow global CSS — it must be pasted back in as custom code, or every `data-trigger`/`data-state`/`.is-active` in the imported project goes silently inert.** The manager (the `[data-state]`/`[data-trigger]` selectors that flip `--_state---*`/`--_trigger---*`) lives in `lumos-foundation.css`, but the HTML-to-Webflow import brings across component styles, not this file wholesale as Webflow's own equivalent global stylesheet — a freshly-imported Webflow site has no page where that block already exists unless the site was already a Lumos project before the import. After importing, paste the manager block into that Webflow site's **Site Settings → Custom Code → Head** (its selectors are too complex for the Designer's style panel to author natively). Do this once per site, not per page. Until it's pasted in, hover/focus/active states will look completely static in the published site even though everything renders correctly in the Designer's own preview only after a real publish with the custom code in place.

## Fidelity notes

The foundation is assembled from three layers with different provenance:

1. **Reset + global + state-manager** — taken from a real Lumos global style. Faithful; edit rarely.
2. **Design tokens** — reconstructed defaults chosen to look reasonable, not to match any specific brand. Expect to customize them.
3. **Utilities + responsive system** — reconstructed to satisfy the contracts the skill emits (`u-section`, `u-container`, `u-text-style-*`, the `threshold-*` grid/order utilities, etc.).

For 1:1 parity with a specific Webflow project, replace layers 2 and 3 with that project's exported **Variables** (`:root` custom properties) and **utility-class CSS**. Because the variable names match, this is a drop-in swap — paste them over sections 2 and 3 and component output is unaffected.

Responsive behavior runs through **two systems**. The `u-grid-*` grid stays breakpointless: the utilities (`u-grid-above`/`u-grid-below`, `u-order-unset-*`, `u-all-unset-*`) plus named `@container threshold-large|medium|small` queries at the global style's own `62 / 48 / 30em` breakpoints, scoped to `u-container` (reacting to the column area's width, not the page). Every other element's responsiveness is hand-written in component CSS as Webflow `@media` breakpoints (`max-width: 991 / 767 / 479px`) so it imports onto the Designer's native breakpoints instead of a custom embed. (The older `--_responsive---*` / `--flex-*` keyword-variable system has been removed from the foundation.)
