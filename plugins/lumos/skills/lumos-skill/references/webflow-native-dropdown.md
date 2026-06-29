# Webflow native dropdown

Read this whenever the layout needs a dropdown / menu (nav dropdown, account menu, "more" menu, etc.). **Don't hand-roll a dropdown** — use Webflow's native dropdown structure so it behaves identically in Webflow and in a from-scratch project, and so it plugs into the trigger/state system.

The native component is three nested pieces:

- `.w-dropdown` — the wrapper (position context).
- `.w-dropdown-toggle` — the click/hover target that opens the list. A `<div role="button" tabindex="0">`, **not** an `<a>` (it doesn't navigate). This is the one sanctioned exception to "use `<a>` for triggers".
- `.w-dropdown-list` — the menu, a `<nav>` holding the links. Hidden until the toggle (and list) get the `.w--open` class.

Open state is the `.w--open` class on the toggle and the list. The foundation's state manager already listens to `.w--open` via `data-state="open"`, so style open/close reactions through the normal trigger/state system — never select `.w--open` yourself in component CSS.

## Markup

Keep the `.w-*` structural classes and add your own component classes alongside them for styling. Links and the toggle label follow the usual text-wrapper rule (`_text` div inside).

```html
<div class="dropdown_wrap w-dropdown" data-hover="false" data-delay="0">
  <div class="dropdown_toggle w-dropdown-toggle" data-state="open" data-trigger="hover focus">
    <div class="dropdown_toggle_text">Menu</div>
    <div class="dropdown_icon w-icon-dropdown-toggle" aria-hidden="true"></div>
  </div>
  <nav class="dropdown_list w-dropdown-list">
    <a href="#" class="dropdown_link w-inline-block">
      <div class="dropdown_link_text">Link one</div>
    </a>
    <a href="#" class="dropdown_link w-inline-block">
      <div class="dropdown_link_text">Link two</div>
    </a>
  </nav>
</div>
```

- `data-hover="true"` opens on hover (desktop); `data-delay="N"` is the close delay in ms. Leave `data-hover="false"` for click-only.
- The ARIA wiring (`id`, `aria-controls`, `aria-haspopup="menu"`, `aria-expanded`, `role`, `tabindex`, `aria-labelledby`) is added at runtime — by Webflow in Webflow mode, by the init below in vanilla mode — so the authored markup stays clean.
- `data-state="open"` on the toggle lets you animate the icon / restyle the toggle while open through the state variables (see below). Drop it if the toggle needs no open-state styling.

## Styling

All visuals go on your component classes — the `.w-*` classes only carry structure. Example: lay the list out and react to open state without ever selecting `.w--open`:

```css
.dropdown_list.w-dropdown-list {
  display: flex;
  flex-flow: column;
  gap: var(--_spacing---space--2);
  padding: var(--_spacing---space--3);
  background-color: var(--_theme---background-2);
  border-radius: var(--radius--main);
}
/* Icon rotates while open — reads the state variable the toggle's
   data-state="open" flips, never `.w--open` directly */
.dropdown_icon {
  transition: transform 200ms;
  transform: rotate(calc(180deg * var(--_state---true)));
}
```

The list's show/hide itself is handled by the structural base (`.w-dropdown-list` is `display: none`, `.w-dropdown-list.w--open` is `display: block`) — don't reimplement it.

## Webflow mode

Emit the **markup only**. Webflow's runtime supplies the base `.w-dropdown` CSS and the entire open/close/keyboard interaction (the dropdown script ships in `webflow.js` on every published Webflow site). So:

- Don't write the `.w-dropdown` structural CSS.
- Don't include any dropdown JS — Webflow runs it.
- Only add your component-class styling.

## Vanilla mode

There's no Webflow runtime, so two things come from the skill instead:

1. **Base CSS** — already in `lumos-foundation.css` (the `.w-dropdown` / `.w-dropdown-list` / `.w--open` / `.w-inline-block` block). Nothing to add.
2. **Behavior** — add this init to `js/main.js` and call it from `initFunction()`. It mirrors Webflow's dropdown: click/hover open, click-outside + Escape close, `aria-expanded`, and arrow / Home / End keyboard navigation. It toggles the native `.w--open` class (the deliberate exception to the `.is-active`-only rule, so it matches the base CSS and the `data-state="open"` listener).

```js
const initDropdown = () => {
  const dropdowns = document.querySelectorAll(".w-dropdown");
  if (!dropdowns.length) return;

  dropdowns.forEach((wrap, index) => {
    const toggle = wrap.querySelector(".w-dropdown-toggle");
    const list = wrap.querySelector(".w-dropdown-list");
    if (!toggle || !list) return;

    // ARIA wiring, the way Webflow's runtime does it
    const toggleId = toggle.id || `w-dropdown-toggle-${index}`;
    const listId = list.id || `w-dropdown-list-${index}`;
    toggle.id = toggleId;
    list.id = listId;
    toggle.setAttribute("aria-controls", listId);
    toggle.setAttribute("aria-haspopup", "menu");
    toggle.setAttribute("aria-expanded", "false");
    list.setAttribute("aria-labelledby", toggleId);
    if (toggle.tagName !== "BUTTON") {
      toggle.setAttribute("role", "button");
      if (!toggle.hasAttribute("tabindex")) toggle.setAttribute("tabindex", "0");
    }

    const links = [...list.querySelectorAll("a")];
    links.forEach((link) => {
      if (!link.hasAttribute("tabindex")) link.setAttribute("tabindex", "0");
    });

    const hoverEnabled = wrap.dataset.hover === "true";
    const delay = Number(wrap.dataset.delay) || 0;
    let closeTimer;

    const isOpen = () => toggle.classList.contains("w--open");
    const open = () => {
      window.clearTimeout(closeTimer);
      toggle.classList.add("w--open");
      list.classList.add("w--open");
      toggle.setAttribute("aria-expanded", "true");
    };
    const close = () => {
      toggle.classList.remove("w--open");
      list.classList.remove("w--open");
      toggle.setAttribute("aria-expanded", "false");
    };

    toggle.addEventListener("click", () => {
      isOpen() ? close() : open();
    });

    toggle.addEventListener("keydown", (event) => {
      if (event.key === "Enter" || event.key === " ") {
        event.preventDefault();
        isOpen() ? close() : open();
      } else if (event.key === "ArrowDown" && !isOpen()) {
        event.preventDefault();
        open();
        links[0]?.focus();
      }
    });

    list.addEventListener("keydown", (event) => {
      if (!isOpen()) return;
      const current = links.indexOf(document.activeElement);
      if (event.key === "Escape") {
        close();
        toggle.focus();
      } else if (event.key === "ArrowDown") {
        event.preventDefault();
        links[Math.min(links.length - 1, current + 1)]?.focus();
      } else if (event.key === "ArrowUp") {
        event.preventDefault();
        links[Math.max(0, current - 1)]?.focus();
      } else if (event.key === "Home") {
        event.preventDefault();
        links[0]?.focus();
      } else if (event.key === "End") {
        event.preventDefault();
        links[links.length - 1]?.focus();
      }
    });

    if (hoverEnabled) {
      wrap.addEventListener("mouseenter", open);
      wrap.addEventListener("mouseleave", () => {
        closeTimer = window.setTimeout(close, delay);
      });
    }

    // Close on outside click / focus leaving the dropdown
    document.addEventListener("click", (event) => {
      if (isOpen() && !wrap.contains(event.target)) close();
    });
    wrap.addEventListener("focusout", (event) => {
      if (isOpen() && !wrap.contains(event.relatedTarget)) close();
    });
  });
};
```

```js
const initFunction = () => {
  initDropdown();
  // ...other component inits
};
```
