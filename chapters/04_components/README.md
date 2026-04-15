# Chapter 4 — Build Your Own Component Library

Bootstrap has 12,000 lines of CSS. Most of those lines define *components* — buttons, cards, badges, alerts, modals. Developers import all 12,000 lines to get a blue button. In this chapter you build the same component system from scratch, and you'll see that a button is about 15 lines, a card is 8 lines, and a badge is 5. The insight is how modifier classes compose: a single base class defines structure, modifier classes swap in color or size, and compound selectors handle states. That pattern is the entire component model.

---

## The Problem

A component needs three things: a base style, variants (color, size), and states (hover, focus, disabled). The naive approach hard-codes everything:

```css
/* Naive: copy-paste per variant — doesn't scale */
.btn-primary {
  padding: 0.5rem 1rem;
  border: none;
  border-radius: 0.375rem;
  background: #4361ee;
  color: white;
  cursor: pointer;
}

.btn-secondary {
  padding: 0.5rem 1rem;  /* duplicated */
  border: none;          /* duplicated */
  border-radius: 0.375rem;  /* duplicated */
  background: #64748b;
  color: white;
  cursor: pointer;       /* duplicated */
}
```

The BEM (Block, Element, Modifier) pattern solves this by separating structure from appearance:

```css
/* BEM: base defines structure, modifiers swap appearance */
.btn { padding: 0.5rem 1rem; border: none; border-radius: 0.375rem; cursor: pointer; }
.btn--primary   { background: #4361ee; color: white; }
.btn--secondary { background: #64748b; color: white; }
```

---

## Building It Step by Step

### v1 — Button Base + Variants

```css
/* v1: button with color variants */

.btn {
  display:       inline-flex;
  align-items:   center;
  gap:           0.375rem;
  padding:       0.5rem 1rem;
  border:        1px solid transparent;
  border-radius: 0.375rem;
  font-family:   inherit;
  font-size:     0.875rem;
  font-weight:   500;
  line-height:   1;
  cursor:        pointer;
  text-decoration: none;
  transition:    all 0.15s;
  white-space:   nowrap;
  user-select:   none;
}

.btn:focus-visible {
  outline:        2px solid;
  outline-offset: 2px;
}

.btn:disabled, .btn[disabled] {
  opacity: 0.5;
  cursor:  not-allowed;
}

/* Variants */
.btn--primary   { background: #4361ee; color: white; border-color: #4361ee; }
.btn--secondary { background: #64748b; color: white; border-color: #64748b; }
.btn--success   { background: #16a34a; color: white; border-color: #16a34a; }
.btn--danger    { background: #dc2626; color: white; border-color: #dc2626; }
.btn--outline   { background: transparent; color: #4361ee; border-color: #4361ee; }
.btn--ghost     { background: transparent; color: #4361ee; border-color: transparent; }

/* Sizes */
.btn--sm { padding: 0.25rem 0.625rem; font-size: 0.75rem; }
.btn--lg { padding: 0.75rem 1.5rem;   font-size: 1rem; }
```

### v2 — Card

```css
/* v2: card with header, body, footer */

.card {
  background:    var(--color-surface);
  border:        1px solid var(--color-border);
  border-radius: 0.5rem;
  overflow:      hidden;
}

.card__header {
  padding:       1rem 1.25rem;
  border-bottom: 1px solid var(--color-border);
  font-weight:   600;
}

.card__body {
  padding: 1.25rem;
}

.card__footer {
  padding:    1rem 1.25rem;
  border-top: 1px solid var(--color-border);
  background: #f8fafc;
}

/* Variants */
.card--raised {
  box-shadow: 0 4px 6px -1px rgb(0 0 0 / 0.1), 0 2px 4px -2px rgb(0 0 0 / 0.1);
}

.card--clickable {
  cursor:     pointer;
  transition: box-shadow 0.2s, transform 0.2s;
}
.card--clickable:hover {
  box-shadow: 0 10px 15px -3px rgb(0 0 0 / 0.1);
  transform:  translateY(-2px);
}
```

### v3 — Badge and Alert

```css
/* v3: badge (inline label) and alert (block message) */

/* Badge */
.badge {
  display:       inline-flex;
  align-items:   center;
  padding:       0.2rem 0.5rem;
  border-radius: 9999px;  /* pill shape */
  font-size:     0.7rem;
  font-weight:   600;
  line-height:   1;
  white-space:   nowrap;
}

.badge--primary { background: #dbeafe; color: #1d4ed8; }
.badge--success { background: #dcfce7; color: #15803d; }
.badge--warning { background: #fef9c3; color: #a16207; }
.badge--danger  { background: #fee2e2; color: #b91c1c; }
.badge--neutral { background: #f1f5f9; color: #475569; }

/* Alert */
.alert {
  display:       flex;
  gap:           0.75rem;
  padding:       1rem 1.25rem;
  border-radius: 0.5rem;
  border:        1px solid transparent;
  font-size:     0.875rem;
}

.alert--info    { background: #eff6ff; border-color: #bfdbfe; color: #1e40af; }
.alert--success { background: #f0fdf4; border-color: #bbf7d0; color: #166534; }
.alert--warning { background: #fefce8; border-color: #fde68a; color: #92400e; }
.alert--danger  { background: #fef2f2; border-color: #fecaca; color: #991b1b; }
```

---

## The Complete Program

`components.html` — the full component library rendered in a browser. Includes buttons, cards, badges, alerts, and an avatar component.

---

## Walkthrough

### BEM Naming

BEM stands for Block, Element, Modifier. It's a naming convention that makes CSS structure visible in the HTML:

```
Block:    .btn         — the standalone component
Element:  .card__body  — a part of a component (double underscore)
Modifier: .btn--lg     — a variant (double dash)

Examples:
  .nav               ← block
  .nav__link         ← element: a link inside nav
  .nav__link--active ← modifier: the active link

  .card              ← block
  .card__header      ← element
  .card__body        ← element
  .card--raised      ← modifier: raised variant
```

Not every project uses BEM literally, but the mental model is universal: *base class handles structure, modifier classes swap appearance.*

### Compound Selectors for States

Hover, focus, and active states use compound selectors (no space = same element):

```css
/* Different from .btn .active (would mean "descendant named .active") */
.btn:hover  { /* same element, hovered */ }
.btn:focus  { /* same element, focused */ }
.btn.active { /* same element with both .btn AND .active classes */ }
.btn--primary:hover { /* a primary button being hovered */ }
```

### The `focus-visible` vs `focus` Distinction

```css
/* :focus triggers on ALL focus events — including mouse clicks */
.btn:focus { outline: 2px solid blue; }

/* :focus-visible only triggers when focus is from keyboard navigation */
.btn:focus-visible { outline: 2px solid blue; }
```

Use `:focus-visible` for visible focus rings. This way, keyboard users see the outline; mouse users don't get the distracting blue ring after clicking.

### `currentColor` for Icon Tinting

When a button contains an SVG icon, use `currentColor` so the icon inherits the button's text color automatically:

```css
.btn svg {
  width:  1em;
  height: 1em;
  fill:   currentColor;  /* inherits color from .btn's color property */
}
```

When you change `.btn--danger` to red text, the icon turns red too — no extra rules needed.

### Box Shadow Anatomy

```
box-shadow: 0  4px  6px  -1px  rgb(0 0 0 / 0.1);
            │   │    │     │        │
            │   │    │     │        └── color with opacity
            │   │    │     └────────── spread (negative = shrink shadow)
            │   │    └──────────────── blur radius
            │   └───────────────────── y offset (positive = down)
            └───────────────────────── x offset

Multiple shadows (comma-separated, first wins on overlap):
box-shadow:
  0 10px 15px -3px rgb(0 0 0 / 0.1),   /* large shadow */
  0  4px  6px -4px rgb(0 0 0 / 0.1);   /* small tight shadow */
```

---

## Guided Try It — Add a Loading Spinner State

**The goal**: `.btn--loading` adds a spinning animation inside the button, disables interaction, and replaces the button text with "Loading…"

**Step 1 — Define the spinner as a pseudo-element**

```css
.btn--loading {
  position: relative;
  color:     transparent;     /* hide text without removing it */
  pointer-events: none;       /* prevent clicks while loading */
}

.btn--loading::after {
  content:      '';
  position:     absolute;
  width:        1em; height:  1em;
  border:       2px solid currentColor;
  border-top-color: transparent;
  border-radius: 50%;
  animation:    spin 0.6s linear infinite;

  /* Center in button */
  top:  50%; left: 50%;
  transform: translate(-50%, -50%);
}

@keyframes spin {
  to { transform: translate(-50%, -50%) rotate(360deg); }
}
```

**Step 2 — The `color: transparent` trick**

Setting `color: transparent` hides the text visually without removing it from the DOM. Screen readers can still read it. The button width doesn't collapse. The `::after` pseudo-element uses `currentColor` on its border — but since `color` is now `transparent`, the border inherits that, which is wrong. Fix it:

```css
.btn--loading::after {
  border-color: white;           /* override for light-text buttons */
  border-top-color: transparent;
}
```

**Step 3 — Pair with JavaScript**

```javascript
document.querySelectorAll('.btn[data-action]').forEach(btn => {
  btn.addEventListener('click', async () => {
    btn.classList.add('btn--loading');
    await simulateWork(1500);
    btn.classList.remove('btn--loading');
  });
});
```

---

## Exercises

1. **Add `.btn--icon`**: A variant for icon-only buttons (no text). It should be square, with equal padding all around. Add a `title` attribute for accessibility. Test with a Unicode symbol like `✕` or `☰`.

2. **Build a `.tag` component**: Like a badge but dismissible. The tag should have an `×` button on the right. When clicked, the tag removes itself from the DOM. Use `::before` for the tag dot marker.

3. **Build a `.progress` bar**: A progress bar component with width controlled by a CSS custom property: `<div class="progress" style="--value: 65%">`. The fill is `width: var(--value)`. Add smooth transition when the value changes.

4. **Build an `.avatar` stack**: A row of overlapping circular avatars. Each avatar overlaps the previous by 8px using a negative `margin-inline-start`. The stack should work for 2–6 avatars. Add a "+N more" badge for overflow.

5. **Build a `.chip` component**: A small, rounded, interactive element used for filters/tags. It should have a selected state (`chip--selected`), a removable state (with × button), and a disabled state. All states should be keyboard-accessible.

---

## Solutions

### Exercise 1 — `btn--icon`

```css
.btn--icon {
  padding:      0.5rem;      /* equal on all sides */
  aspect-ratio: 1;           /* force square */
  justify-content: center;
}

/* Sizes still work: */
.btn--icon.btn--sm { padding: 0.375rem; }
.btn--icon.btn--lg { padding: 0.75rem; }
```

### Exercise 2 — `.tag` component

```css
.tag {
  display:       inline-flex;
  align-items:   center;
  gap:           0.25rem;
  padding:       0.25rem 0.5rem;
  background:    #f1f5f9;
  border:        1px solid #e2e8f0;
  border-radius: 4px;
  font-size:     0.8rem;
}

.tag::before {
  content:       '';
  width:         6px; height: 6px;
  border-radius: 50%;
  background:    #94a3b8;
}

.tag-dismiss {
  background:  none;
  border:      none;
  cursor:      pointer;
  color:       #94a3b8;
  padding:     0;
  font-size:   1rem;
  line-height: 1;
}
.tag-dismiss:hover { color: #dc2626; }
```

```javascript
document.querySelectorAll('.tag-dismiss').forEach(btn => {
  btn.addEventListener('click', () => btn.closest('.tag').remove());
});
```

### Exercise 3 — `.progress` bar

```css
.progress {
  background:    #e2e8f0;
  border-radius: 9999px;
  height:        0.5rem;
  overflow:      hidden;
}

.progress::after {
  content:        '';
  display:        block;
  height:         100%;
  width:          var(--value, 0%);
  background:     #4361ee;
  border-radius:  inherit;
  transition:     width 0.4s ease;
}
```

```html
<div class="progress" style="--value: 65%"></div>
```

```javascript
/* Update: */
el.style.setProperty('--value', '80%');
/* Transition fires automatically */
```

### Exercise 4 — `.avatar` stack

```css
.avatar-stack {
  display:  flex;
  flex-direction: row-reverse;  /* reverse so first avatar is on top */
}

.avatar {
  width:         36px; height: 36px;
  border-radius: 50%;
  border:        2px solid white;
  overflow:      hidden;
  margin-inline-start: -8px;   /* overlap */
  background:    #4361ee;
  color:         white;
  display:       flex;
  align-items:   center;
  justify-content: center;
  font-size:     0.7rem;
  font-weight:   700;
}

.avatar-stack .avatar:last-child { margin-inline-start: 0; }

.avatar-overflow {
  background:  #e2e8f0;
  color:       #64748b;
  font-size:   0.65rem;
}
```

### Exercise 5 — `.chip` component

```css
.chip {
  display:        inline-flex;
  align-items:    center;
  gap:            0.25rem;
  padding:        0.3rem 0.75rem;
  background:     #f1f5f9;
  border:         1px solid #e2e8f0;
  border-radius:  9999px;
  font-size:      0.8rem;
  cursor:         pointer;
  transition:     all 0.15s;
  user-select:    none;
}

.chip:hover          { background: #e2e8f0; }
.chip:focus-visible  { outline: 2px solid #4361ee; outline-offset: 2px; }
.chip--selected      { background: #dbeafe; border-color: #93c5fd; color: #1d4ed8; }
.chip--selected:hover{ background: #bfdbfe; }
.chip:disabled, .chip[aria-disabled="true"] { opacity: 0.5; cursor: not-allowed; }

.chip-remove {
  background:  none; border: none; cursor: pointer;
  color:       inherit; padding: 0; font-size: 0.9rem; line-height: 1;
  opacity:     0.6;
}
.chip-remove:hover { opacity: 1; }
```

---

## What You Learned

| Concept | Key point |
|---------|-----------|
| BEM | Block / Element (`__`) / Modifier (`--`): separates structure from appearance |
| Compound selectors | `.btn:hover` = same element; `.btn .icon` = descendant |
| `:focus-visible` | Focus ring only for keyboard users, not mouse clicks |
| `inline-flex` | Button behaves inline in text flow but has flex alignment internally |
| `currentColor` | Inherits the element's `color` value — great for icon tinting |
| `pointer-events: none` | Disables all mouse/touch interaction on an element |
| `color: transparent` | Hides text visually but keeps DOM structure and width |
| CSS custom property as param | `--value: 65%` passed inline; consumed by `var(--value)` in the rule |
| `box-shadow` syntax | x-offset y-offset blur spread color — multiple shadows comma-separated |
| `aspect-ratio: 1` | Forces square regardless of content or container |

---

## Building with Claude

Bad prompt:
> "How do I make buttons in CSS?"

Good prompt:
> "I'm building a CSS component library with a `.btn` base class and modifiers like `.btn--primary` and `.btn--danger`. I use `color: transparent` + `::after` spinner for the loading state. Problem: when I add `.btn--loading` to a `.btn--outline` button (which has `background: transparent; color: #4361ee; border: 1px solid #4361ee`), the spinner inherits `currentColor` = transparent instead of the border color. I can't use `border-color: white` because outline buttons have a dark background context. How do I make the spinner color adapt to whether the button has a light or dark background?"
