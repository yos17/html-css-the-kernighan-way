# Chapter 7 — Build Your Own Component Library

Every UI framework you've ever used — Bootstrap, Tailwind, Material UI, Chakra — is a component library. Buttons, cards, badges, alerts: these are not mysteries. They are a small set of CSS patterns applied consistently to HTML elements. Once you've built them from scratch, you will see exactly what every framework is doing. The magic evaporates. What remains is clarity.

Building a component library teaches you three things simultaneously: how to structure CSS for reuse, how to use custom properties as a design token system, and what the browser actually does when you apply `:hover`, `:focus`, or `:disabled`. These are not theoretical concepts. They are the foundation of every interactive UI ever built on the web.

The components in this chapter form a coherent system: buttons, cards, badges, and alerts share the same token vocabulary, so changing a single custom property ripples through every component correctly. This is how design systems at scale actually work — not through documentation, but through structured CSS.

## The Problem

The naive button:

```css
.btn {
  background: blue;
  color: white;
  padding: 8px 16px;
  border: none;
  cursor: pointer;
}

.btn-danger {
  background: red;
}
```

Three things are already wrong. First, `.btn-danger` can't exist independently — it requires `.btn` on the element, but nothing in the CSS enforces that. Second, the `:focus` state is missing, so keyboard users have no visual feedback. Third, there's no `:disabled` state, so disabled buttons look active. The simplest button has at least five states: default, hover, active, focus, and disabled. Every variant multiplies that.

The second problem is color coupling. `background: blue` is a dead end. When you need a secondary button, you write more colors. When the brand changes, you change colors everywhere. Custom properties as design tokens solve this — but only if you structure them as a system, not as one-off variables.

The third problem is naming. Without a convention, you get `.btn-primary`, `.button-primary`, `.primary-button`, `.primaryBtn` in the same codebase. BEM (Block Element Modifier) solves naming by making the structure of CSS class names mirror the structure of HTML components.

## Building It Step by Step

### v1: Buttons with States and Variants

Start with a button that handles all five states correctly, then add variants.

```css
/* v1: button with full state handling */
.btn {
  /* Layout */
  display: inline-flex;
  align-items: center;
  justify-content: center;
  gap: 0.5rem;

  /* Spacing */
  padding: 0.5rem 1.25rem;

  /* Typography */
  font-family: inherit;
  font-size: 0.875rem;
  font-weight: 500;
  line-height: 1.5;
  white-space: nowrap;

  /* Appearance */
  border: 1px solid transparent;
  border-radius: 6px;
  cursor: pointer;
  text-decoration: none;

  /* Transition */
  transition: background-color 0.15s, border-color 0.15s,
              color 0.15s, box-shadow 0.15s, opacity 0.15s;

  /* Prevent text selection on rapid clicking */
  user-select: none;
}

/* Primary variant */
.btn--primary {
  background: #2563eb;
  color: #ffffff;
  border-color: #2563eb;
}

.btn--primary:hover {
  background: #1d4ed8;
  border-color: #1d4ed8;
}

.btn--primary:active {
  background: #1e40af;
  border-color: #1e40af;
}

/* Focus — NEVER remove this. It's an accessibility requirement. */
.btn:focus-visible {
  outline: 2px solid #a8d8ea;
  outline-offset: 2px;
}

/* Disabled state */
.btn:disabled,
.btn[aria-disabled="true"] {
  opacity: 0.5;
  cursor: not-allowed;
  pointer-events: none;
}
```

`:focus-visible` is the correct selector for focus rings. Unlike `:focus`, it only shows when the element was focused via keyboard or another non-pointer mechanism — so clicking a button doesn't leave a ring, but tabbing to it does.

### v2: Cards, Badges, and Composition

Cards and badges introduce the modifier pattern and the concept of component composition.

```css
/* v2: card, badge, alert */

/* ── Card ── */
.card {
  background: var(--card-bg, #1a1a2e);
  border: 1px solid var(--card-border, #2a2a40);
  border-radius: 8px;
  overflow: hidden;
}

.card__header {
  padding: 1rem 1.25rem;
  border-bottom: 1px solid var(--card-border, #2a2a40);
}

.card__body {
  padding: 1.25rem;
}

.card__footer {
  padding: 0.75rem 1.25rem;
  border-top: 1px solid var(--card-border, #2a2a40);
}

/* ── Badge ── */
.badge {
  display: inline-flex;
  align-items: center;
  padding: 0.15rem 0.5rem;
  font-size: 0.75rem;
  font-weight: 600;
  line-height: 1.4;
  border-radius: 999px;
  border: 1px solid transparent;
}

.badge--default  { background: #1a1a3e; color: #a8d8ea; border-color: #2a2a50; }
.badge--success  { background: #052e16; color: #4ade80; border-color: #166534; }
.badge--warning  { background: #431407; color: #fb923c; border-color: #7c2d12; }
.badge--danger   { background: #450a0a; color: #f87171; border-color: #7f1d1d; }

/* ── Alert ── */
.alert {
  display: flex;
  gap: 0.75rem;
  padding: 0.875rem 1rem;
  border-radius: 8px;
  border: 1px solid transparent;
  font-size: 0.875rem;
  line-height: 1.5;
}

.alert__icon  { font-size: 1rem; flex-shrink: 0; margin-top: 0.1rem; }
.alert__body  { flex: 1; }
.alert__title { font-weight: 600; margin-bottom: 0.25rem; }

.alert--info    { background: #0c1a2e; color: #93c5fd; border-color: #1e3a5f; }
.alert--success { background: #052e16; color: #86efac; border-color: #14532d; }
.alert--warning { background: #1c1003; color: #fcd34d; border-color: #78350f; }
.alert--error   { background: #1c0505; color: #fca5a5; border-color: #7f1d1d; }
```

BEM names make component structure legible: `.card__header` immediately tells you it's the `header` element *inside* a `card` block. `.badge--success` is the `success` modifier of a `badge`. This isn't ceremony — it's machine-readable documentation.

### v3: The Token System

Custom properties as design tokens give you a single source of truth for every visual decision. Change one token and every component that uses it updates.

```css
/* v3: token-driven component system */
:root {
  /* Color palette */
  --color-primary-500: #3b82f6;
  --color-primary-600: #2563eb;
  --color-primary-700: #1d4ed8;
  --color-danger-500:  #ef4444;
  --color-danger-600:  #dc2626;
  --color-success-500: #22c55e;
  --color-success-600: #16a34a;

  /* Semantic tokens — components reference these, not palette directly */
  --btn-primary-bg:    var(--color-primary-600);
  --btn-primary-hover: var(--color-primary-700);
  --btn-primary-text:  #ffffff;

  --btn-danger-bg:     var(--color-danger-600);
  --btn-danger-hover:  var(--color-danger-500);
  --btn-danger-text:   #ffffff;

  /* Surface tokens */
  --surface-1: #0d0d1a;
  --surface-2: #111122;
  --surface-3: #1a1a2e;
  --surface-4: #16213e;

  /* Border */
  --border-color: #2a2a40;
  --radius-sm:  4px;
  --radius-md:  6px;
  --radius-lg:  8px;
  --radius-pill: 999px;

  /* Focus ring */
  --focus-ring: 0 0 0 2px #0d0d1a, 0 0 0 4px #a8d8ea;
}

/* Now buttons reference tokens */
.btn--primary {
  background:   var(--btn-primary-bg);
  color:        var(--btn-primary-text);
  border-color: var(--btn-primary-bg);
}

.btn--primary:hover {
  background:   var(--btn-primary-hover);
  border-color: var(--btn-primary-hover);
}

.btn:focus-visible {
  outline: none;
  box-shadow: var(--focus-ring);
}
```

The two-level token system (palette tokens → semantic tokens → component references) is how every production design system works. The palette defines the colors. Semantic tokens name their *purpose*. Components reference semantic tokens. When you need a dark mode, you only change the semantic tokens — the components update automatically.

## The Complete Program

See `components.html` for the interactive showcase. Below is the complete `components.css`:

```css
/* components.css — Chapter 7 */
/* A minimal component library with BEM naming and token-based theming */

/* ─── Design Tokens ──────────────────────────────────────── */
:root {
  /* Palette */
  --blue-400:  #60a5fa;
  --blue-500:  #3b82f6;
  --blue-600:  #2563eb;
  --blue-700:  #1d4ed8;

  --green-400: #4ade80;
  --green-600: #16a34a;

  --amber-400: #fbbf24;
  --amber-600: #d97706;

  --red-400:   #f87171;
  --red-500:   #ef4444;
  --red-600:   #dc2626;

  --gray-400:  #9ca3af;
  --gray-600:  #4b5563;

  /* Semantic */
  --color-primary:   var(--blue-600);
  --color-primary-h: var(--blue-700);
  --color-danger:    var(--red-600);
  --color-danger-h:  var(--red-500);
  --color-success:   var(--green-600);
  --color-ghost-h:   rgba(168,216,234,0.08);

  /* Surfaces */
  --s1: #0d0d1a;
  --s2: #111122;
  --s3: #1a1a2e;
  --s4: #16213e;

  /* Borders + radius */
  --border:     #2a2a40;
  --r-sm:  4px;
  --r-md:  6px;
  --r-lg:  8px;
  --r-pill: 999px;

  /* Focus */
  --focus-ring: 0 0 0 2px var(--s1), 0 0 0 4px #a8d8ea;

  /* Typography */
  --font-mono: 'Courier New', monospace;
}

/* ─── Button Base ────────────────────────────────────────── */
.btn {
  display: inline-flex;
  align-items: center;
  justify-content: center;
  gap: 0.5rem;
  padding: 0.5rem 1.25rem;
  font-family: var(--font-mono);
  font-size: 0.875rem;
  font-weight: 500;
  line-height: 1.5;
  white-space: nowrap;
  text-decoration: none;
  border: 1px solid transparent;
  border-radius: var(--r-md);
  cursor: pointer;
  user-select: none;
  transition: background 0.15s, border-color 0.15s,
              color 0.15s, box-shadow 0.15s, opacity 0.15s;
}

.btn:focus-visible {
  outline: none;
  box-shadow: var(--focus-ring);
}

.btn:disabled,
.btn[aria-disabled="true"] {
  opacity: 0.45;
  cursor: not-allowed;
  pointer-events: none;
}

/* ─── Button Variants ────────────────────────────────────── */
.btn--primary {
  background: var(--color-primary);
  color: #fff;
  border-color: var(--color-primary);
}
.btn--primary:hover  { background: var(--color-primary-h); border-color: var(--color-primary-h); }
.btn--primary:active { filter: brightness(0.9); }

.btn--secondary {
  background: var(--s3);
  color: #c8c8d8;
  border-color: var(--border);
}
.btn--secondary:hover  { background: var(--s4); border-color: #a8d8ea; color: #a8d8ea; }
.btn--secondary:active { filter: brightness(0.9); }

.btn--danger {
  background: var(--color-danger);
  color: #fff;
  border-color: var(--color-danger);
}
.btn--danger:hover  { background: var(--color-danger-h); border-color: var(--color-danger-h); }
.btn--danger:active { filter: brightness(0.9); }

.btn--ghost {
  background: transparent;
  color: #a8d8ea;
  border-color: transparent;
}
.btn--ghost:hover { background: var(--color-ghost-h); }

.btn--outline {
  background: transparent;
  color: #a8d8ea;
  border-color: #a8d8ea;
}
.btn--outline:hover { background: rgba(168,216,234,0.1); }

/* ─── Button Sizes ───────────────────────────────────────── */
.btn--sm { padding: 0.25rem 0.75rem; font-size: 0.8rem; border-radius: var(--r-sm); }
.btn--lg { padding: 0.75rem 1.75rem; font-size: 1rem; border-radius: var(--r-lg); }

/* ─── Card ───────────────────────────────────────────────── */
.card {
  background: var(--s3);
  border: 1px solid var(--border);
  border-radius: var(--r-lg);
  overflow: hidden;
}

.card__header {
  padding: 1rem 1.25rem;
  border-bottom: 1px solid var(--border);
  font-weight: 600;
  color: #c8c8d8;
}

.card__body {
  padding: 1.25rem;
  color: #9898b8;
  font-size: 0.875rem;
  line-height: 1.6;
}

.card__footer {
  padding: 0.75rem 1.25rem;
  border-top: 1px solid var(--border);
  display: flex;
  gap: 0.5rem;
  align-items: center;
}

.card--interactive {
  cursor: pointer;
  transition: border-color 0.15s, transform 0.15s, box-shadow 0.15s;
}
.card--interactive:hover {
  border-color: #a8d8ea;
  transform: translateY(-2px);
  box-shadow: 0 4px 16px rgba(0,0,0,0.4);
}

/* ─── Badge ──────────────────────────────────────────────── */
.badge {
  display: inline-flex;
  align-items: center;
  gap: 0.25rem;
  padding: 0.15rem 0.5rem;
  font-size: 0.75rem;
  font-weight: 600;
  line-height: 1.4;
  border-radius: var(--r-pill);
  border: 1px solid transparent;
  white-space: nowrap;
}

.badge--default  { background: #1a1a3e; color: #a8d8ea; border-color: #2a2a50; }
.badge--primary  { background: #1e3a5f; color: #93c5fd; border-color: #1e40af; }
.badge--success  { background: #052e16; color: #4ade80; border-color: #166534; }
.badge--warning  { background: #431407; color: #fb923c; border-color: #7c2d12; }
.badge--danger   { background: #450a0a; color: #f87171; border-color: #7f1d1d; }
.badge--neutral  { background: #1f2937; color: #9ca3af; border-color: #374151; }

/* ─── Alert ──────────────────────────────────────────────── */
.alert {
  display: flex;
  gap: 0.75rem;
  padding: 0.875rem 1rem;
  border-radius: var(--r-lg);
  border: 1px solid transparent;
  font-size: 0.875rem;
  line-height: 1.5;
}

.alert__icon  { font-size: 1.1rem; flex-shrink: 0; line-height: 1.5; }
.alert__body  { flex: 1; min-width: 0; }
.alert__title { font-weight: 600; margin-bottom: 0.2rem; }
.alert__close {
  background: none; border: none; cursor: pointer;
  padding: 0; font-size: 1rem; line-height: 1; opacity: 0.7; flex-shrink: 0;
}
.alert__close:hover { opacity: 1; }

.alert--info    { background: #0c1a2e; color: #93c5fd; border-color: #1e3a5f; }
.alert--success { background: #052e16; color: #86efac; border-color: #14532d; }
.alert--warning { background: #1c0f03; color: #fcd34d; border-color: #78350f; }
.alert--error   { background: #1c0505; color: #fca5a5; border-color: #7f1d1d; }

/* ─── currentColor theming ───────────────────────────────── */
.alert__close { color: currentColor; }
.alert__icon  { color: currentColor; }
```

## Walkthrough

### BEM Naming

BEM (Block Element Modifier) is a naming convention that makes CSS class names self-documenting.

```
Structure:
  .block              The component root
  .block__element     A part of the block
  .block--modifier    A variant or state of the block

Examples:
  .card               → the card component
  .card__header       → the header part of a card
  .card__body         → the body part of a card
  .card--interactive  → a card variant that's clickable

  .btn                → the button component
  .btn--primary       → the primary variant
  .btn--sm            → the small size modifier
  .btn__icon          → an icon element inside a button

In HTML:
  <div class="card card--interactive">
    <div class="card__header">
      Title <span class="badge badge--success">New</span>
    </div>
    <div class="card__body">Content</div>
    <div class="card__footer">
      <button class="btn btn--primary btn--sm">Action</button>
    </div>
  </div>

The class names tell you the entire component hierarchy.
No need to read the HTML structure.
```

The double-underscore and double-hyphen separators are load-bearing. `.card-header` could be a standalone component or a card element — you can't tell. `.card__header` can only be a card element. The convention is precise.

### CSS Cascade Order for Component States

States must be applied in a specific order to work correctly. The standard is LVHFA: Link, Visited, Hover, Focus, Active.

```
For buttons and interactive elements, the cascade order:

  1. Base (no pseudo-class)
     .btn { background: blue; }
     Sets the resting state.

  2. Hover
     .btn:hover { background: darkblue; }
     Applied when mouse is over the element.

  3. Focus
     .btn:focus-visible { box-shadow: 0 0 0 2px white, 0 0 0 4px blue; }
     Applied when element has keyboard focus.
     :focus-visible — only when focused by keyboard/non-pointer.
     :focus          — always when focused (including mouse click).

  4. Active
     .btn:active { filter: brightness(0.9); }
     Applied while the element is being clicked.
     Note: :active must come AFTER :hover and :focus in the cascade.

  5. Disabled
     .btn:disabled { opacity: 0.45; cursor: not-allowed; }
     Applied via HTML attribute. pointer-events: none prevents hover/active.

Specificity is equal for all pseudo-classes (0-1-1 for .btn:hover).
Order in the stylesheet determines which one wins when multiple apply.
```

### Focus Indicators — Accessibility

Removing the focus ring is one of the most common accessibility mistakes in CSS.

```css
/* The crime: */
*:focus { outline: none; }
/* or */
.btn:focus { outline: 0; }

/* This makes the site impossible to navigate by keyboard.
   Screen reader users, motor-impaired users, and power users
   all rely on visible focus indicators. */

/* The correct approach: */
.btn:focus-visible {
  outline: none;                          /* remove default browser ring */
  box-shadow: 0 0 0 2px #0d0d1a,         /* inner ring — matches background */
              0 0 0 4px #a8d8ea;          /* outer ring — visible on any background */
}

/* This gives you a custom focus ring that:
   1. Looks great (double-ring technique)
   2. Works on dark and light backgrounds
   3. Only shows for keyboard navigation (:focus-visible)
   4. Doesn't appear on mouse clicks */
```

The double-ring technique (inner ring matching background + outer colored ring) makes the focus indicator visible on *any* background color. A single outline works on some backgrounds but disappears on others.

### The `currentColor` Keyword

`currentColor` is the current value of the CSS `color` property. It enables single-property theming.

```
Without currentColor:
  .alert--info { color: #93c5fd; }
  .alert__icon { color: #93c5fd; }  ← must repeat the color
  .alert__close { color: #93c5fd; } ← must repeat again

  If the color changes, update 3 places.

With currentColor:
  .alert--info { color: #93c5fd; }
  .alert__icon  { color: currentColor; } ← inherits parent's color
  .alert__close { color: currentColor; } ← inherits parent's color

  Change one property, all elements update.

currentColor can be used in:
  background, border-color, fill, stroke, outline, box-shadow...

Example — icon that matches its parent:
  .btn__icon {
    fill: currentColor;   /* SVG icon fills with button text color */
  }

Example — border that matches text:
  .badge {
    border: 1px solid currentColor;  /* border tracks text color */
    opacity: 0.6;                    /* slightly transparent */
  }
```

### CSS Logical Properties

Logical properties replace physical directions with writing-mode-aware equivalents. This matters for internationalization: Arabic and Hebrew read right-to-left, and physical `left`/`right` breaks those layouts.

```
Physical → Logical:

  margin-top      → margin-block-start
  margin-bottom   → margin-block-end
  margin-left     → margin-inline-start
  margin-right    → margin-inline-end

  padding-top     → padding-block-start
  border-left     → border-inline-start
  left            → inset-inline-start
  width           → inline-size
  height          → block-size

In a left-to-right language (English):
  inline = horizontal
  block  = vertical

In a right-to-left language (Arabic):
  inline-start = right side
  inline-end   = left side

Practical: use for padding and margin on text elements.
Keep physical properties for purely decorative spacing.

.card__body {
  padding-block:  1rem;    /* top and bottom */
  padding-inline: 1.25rem; /* left and right, or right and left in RTL */
}
```

### `:hover`, `:active`, `:focus`, `:focus-visible`, `:disabled`

```
Pseudo-class reference:

  :hover        Applied when pointer device is over the element.
                Does NOT apply when element is focused by keyboard.
                ┌──────────────────────────────────────────┐
                │ Mouse over element → :hover is true      │
                │ Mouse elsewhere   → :hover is false      │
                └──────────────────────────────────────────┘

  :active       Applied while element is being activated (mousedown,
                or while Enter/Space is held on a button).
                Very brief — just the moment of click.

  :focus        Applied whenever element has focus: keyboard OR click.
                ┌──────────────────────────────────────────┐
                │ Click on button → :focus is true         │
                │ Tab to button  → :focus is true          │
                │ Click elsewhere → :focus is false        │
                └──────────────────────────────────────────┘

  :focus-visible Applied only when focus came from keyboard
                 or other non-pointer source. The browser determines
                 this based on the "last input modality."
                 Use this for visible focus rings.

  :disabled     Applied when the HTML disabled attribute is present.
                Works on: input, button, select, textarea, fieldset.
                Does NOT work on div, a, span.
                Use aria-disabled + pointer-events: none for those.
```

## Guided Try It — Add an Icon Button Variant

Many component libraries have icon-only buttons: square, with just an icon and an accessible label. Let's build one.

### Step 1: Create the base icon button

```css
.btn--icon {
  padding: 0.5rem;           /* equal padding for square shape */
  width: 2.25rem;
  height: 2.25rem;
  border-radius: var(--r-md);
}
```

```html
<button class="btn btn--secondary btn--icon" aria-label="Delete item">
  <!-- SVG icon or text character -->
  ✕
</button>
```

The `aria-label` is mandatory — there's no visible text, so screen readers need the label attribute.

### Step 2: Add a tooltip on hover using CSS only

```css
.btn--icon {
  position: relative;
}

/* Tooltip */
.btn--icon::after {
  content: attr(aria-label);
  position: absolute;
  bottom: calc(100% + 6px);
  left: 50%;
  transform: translateX(-50%);
  background: #1a1a3e;
  color: #c8c8d8;
  font-size: 0.7rem;
  padding: 0.25rem 0.5rem;
  border-radius: 4px;
  white-space: nowrap;
  border: 1px solid #2a2a40;
  pointer-events: none;
  opacity: 0;
  transition: opacity 0.15s;
}

.btn--icon:hover::after {
  opacity: 1;
}
```

The tooltip reads from `aria-label` via `content: attr(aria-label)`. The label serves double duty: accessibility for screen readers, and visible tooltip for sighted users.

### Step 3: Add loading state

```css
.btn--loading {
  position: relative;
  color: transparent !important;  /* hide text while loading */
  pointer-events: none;
}

.btn--loading::after {
  content: '';
  position: absolute;
  width: 1rem;
  height: 1rem;
  border: 2px solid rgba(255,255,255,0.3);
  border-top-color: #fff;
  border-radius: 50%;
  animation: spin 0.6s linear infinite;
}

@keyframes spin {
  to { transform: rotate(360deg); }
}
```

```javascript
button.classList.add('btn--loading');
button.setAttribute('aria-busy', 'true');
// ... async work ...
button.classList.remove('btn--loading');
button.removeAttribute('aria-busy');
```

`aria-busy="true"` tells screen readers the button is doing work. The text becoming transparent (but still in the DOM) preserves the button's width — it doesn't jump smaller during loading.

**Think about it**: The loading spinner uses `::after`. But step 2's tooltip also uses `::after`. If you put both on the same button, the second one overwrites the first. CSS only allows two pseudo-elements per element (`::before` and `::after`). How would you solve this conflict without adding a wrapper element?

## Exercises

1. **Button Group**: Create a `.btn-group` component where multiple buttons are joined edge-to-edge, sharing borders. The leftmost button has rounded left corners; the rightmost has rounded right corners; middle buttons have no rounding. The active/selected button in the group should have a different background. Hint: use CSS `:first-child`, `:last-child`, and negative `margin-inline-start` to merge borders.

2. **Dismissible Alert**: Make the `.alert` component dismissible with a close button inside it. The close button should be positioned at the top-right of the alert. When clicked, the alert should shrink its height to 0 with a transition, then `display: none`. Implement the transition without JavaScript by using `max-height` animation. (Hint: transitioning `max-height` is the CSS-only way to animate height to 0.)

3. **Token Theming**: Take the component library and add a full theme-switch capability. Define a `[data-theme="light"]` variant that changes all `--s1` through `--s4` tokens and `--border` to light equivalents. Both buttons and cards should reflect the theme without changing their CSS rules — only the token values change. Toggle the theme with a JavaScript button that sets `document.documentElement.dataset.theme`.

4. **Progress Badge**: Create a `.badge--progress` variant that shows a percentage value with a small circular progress indicator using `conic-gradient`. The badge should accept a `--progress` custom property (0–100) and fill a circle accordingly. Example: `style="--progress: 75"` shows a 75% filled circle.

5. **Card Skeleton Loader**: Create a `.card--skeleton` modifier where the card's content is replaced by animated shimmer placeholders. Use `background: linear-gradient` + `background-size` + `animation` to create a shimmer effect that sweeps left-to-right. The skeleton should match the structure of a real card (title placeholder, body lines, footer).

## Solutions

### Exercise 1: Button Group

```css
.btn-group {
  display: inline-flex;
}

.btn-group .btn {
  border-radius: 0;
  margin-inline-start: -1px;  /* merge borders */
  position: relative;
}

.btn-group .btn:first-child {
  border-radius: var(--r-md) 0 0 var(--r-md);
  margin-inline-start: 0;
}

.btn-group .btn:last-child {
  border-radius: 0 var(--r-md) var(--r-md) 0;
}

.btn-group .btn:hover,
.btn-group .btn:focus-visible {
  z-index: 1;  /* bring to front so borders show */
}

.btn-group .btn.active,
.btn-group .btn[aria-pressed="true"] {
  background: var(--s4);
  color: var(--accent);
  border-color: var(--accent);
  z-index: 1;
}
```

```html
<div class="btn-group" role="group" aria-label="View mode">
  <button class="btn btn--secondary active" aria-pressed="true">Grid</button>
  <button class="btn btn--secondary" aria-pressed="false">List</button>
  <button class="btn btn--secondary" aria-pressed="false">Table</button>
</div>
```

### Exercise 2: Dismissible Alert

```css
.alert--dismissible {
  max-height: 200px;
  overflow: hidden;
  transition: max-height 0.3s ease, opacity 0.3s ease,
              padding 0.3s ease, margin 0.3s ease;
}

.alert--dismissed {
  max-height: 0;
  opacity: 0;
  padding-block: 0;
  margin-block: 0;
  border-width: 0;
}
```

```javascript
document.querySelectorAll('.alert__close').forEach(btn => {
  btn.addEventListener('click', () => {
    const alert = btn.closest('.alert');
    alert.classList.add('alert--dismissed');
    alert.addEventListener('transitionend', () => {
      alert.style.display = 'none';
    }, { once: true });
  });
});
```

### Exercise 3: Token Theming

```css
:root {
  --s1: #0d0d1a;
  --s2: #111122;
  --s3: #1a1a2e;
  --s4: #16213e;
  --border: #2a2a40;
  --text: #c8c8d8;
  --text-dim: #7878a0;
}

[data-theme="light"] {
  --s1: #f8f9fa;
  --s2: #ffffff;
  --s3: #f1f3f5;
  --s4: #e9ecef;
  --border: #dee2e6;
  --text: #212529;
  --text-dim: #6c757d;
}
```

```javascript
const toggle = document.querySelector('#theme-toggle');
toggle.addEventListener('click', () => {
  const current = document.documentElement.dataset.theme;
  document.documentElement.dataset.theme = current === 'light' ? '' : 'light';
});
```

### Exercise 4: Progress Badge

```css
.badge--progress {
  --progress: 0;
  gap: 0.4rem;
  padding-inline-start: 0.3rem;
}

.badge--progress::before {
  content: '';
  display: inline-block;
  width: 14px;
  height: 14px;
  border-radius: 50%;
  background: conic-gradient(
    currentColor calc(var(--progress) * 1%),
    rgba(255,255,255,0.2) calc(var(--progress) * 1%)
  );
  flex-shrink: 0;
}
```

```html
<span class="badge badge--primary badge--progress" style="--progress: 75">75%</span>
```

### Exercise 5: Card Skeleton Loader

```css
.card--skeleton .card__header,
.card--skeleton .card__body,
.card--skeleton .card__footer {
  pointer-events: none;
}

.skeleton-line {
  height: 0.875rem;
  border-radius: 4px;
  margin-bottom: 0.5rem;
  background: linear-gradient(
    90deg,
    var(--s4) 25%,
    var(--s3) 50%,
    var(--s4) 75%
  );
  background-size: 200% 100%;
  animation: shimmer 1.5s infinite;
}

.skeleton-line:last-child { margin-bottom: 0; }
.skeleton-line--short { width: 40%; }
.skeleton-line--medium { width: 70%; }

@keyframes shimmer {
  0%   { background-position: 200% 0; }
  100% { background-position: -200% 0; }
}
```

```html
<div class="card card--skeleton">
  <div class="card__header">
    <div class="skeleton-line skeleton-line--medium"></div>
  </div>
  <div class="card__body">
    <div class="skeleton-line"></div>
    <div class="skeleton-line"></div>
    <div class="skeleton-line skeleton-line--short"></div>
  </div>
  <div class="card__footer">
    <div class="skeleton-line skeleton-line--short"></div>
  </div>
</div>
```

## What You Learned

| Concept | Key point |
|---|---|
| BEM | `.block`, `.block__element`, `.block--modifier` — naming that encodes component structure |
| Component states | Five states every interactive element needs: default, hover, active, focus, disabled |
| `:focus-visible` | Use instead of `:focus` for rings — only shows on keyboard focus, not mouse clicks |
| Focus ring technique | Double box-shadow (inner matches bg + outer colored) works on any background |
| `outline: none` | Never apply without a `:focus-visible` replacement — it disables keyboard navigation |
| `currentColor` | References the element's `color` value; enables single-property theming of icons and borders |
| Design tokens | Two-level system: palette tokens define colors; semantic tokens define purpose; components reference semantic tokens |
| CSS logical properties | `padding-inline`, `margin-block` etc. work correctly in both LTR and RTL layouts |
| `:disabled` | HTML attribute — works on form elements. Use `aria-disabled` + `pointer-events: none` for other elements |
| Cascade order for states | Base → hover → focus → active. Order matters because specificity is equal; last declaration wins |
| `pointer-events: none` | On disabled elements, prevents all mouse interaction including `:hover` |

## Building with Claude

**Bad prompt:**
> "Add a loading state to my button component"

This could mean a dozen different things. Does the button stay the same size? Does the text change? Is there a spinner? Does it disable while loading? You'll get a guess.

**Good prompt:**
> "I have a `.btn` component with `.btn--primary` and `.btn--secondary` variants. Add a `.btn--loading` modifier class that: (1) replaces the button text with a CSS-only spinning ring using `::after` (I'm not using `::after` for anything else on this component), (2) preserves the button's current width so layout doesn't shift, (3) disables pointer events while loading, (4) works on both primary and secondary variants without duplicating the animation CSS. Show only the CSS additions, not the full component."

This prompt specifies the existing structure, what the modifier must do (four precise requirements), a constraint (`::after` is available), and the desired scope. You get a precise, drop-in answer.

**Connection to the JS book**: The button components in Chapter 6's jQuery demo and the alert notifications in Chapter 11's Redux demo use exactly these button and alert components. Chapter 7's test runner uses badge-style pass/fail indicators — the `.badge--success` and `.badge--danger` variants show test results inline.
