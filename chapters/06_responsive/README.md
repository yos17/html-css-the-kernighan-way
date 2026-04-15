# Chapter 6 — Build Your Own Responsive Framework

Mobile-first design is not about making things smaller. It is about writing CSS for the most constrained environment first and progressively enhancing for larger screens. That inversion — small to large, not large to small — is the entire discipline in one sentence. When you internalize it, breakpoints become additive rather than corrective, and the entire responsive problem becomes tractable.

Understanding responsive design at this level is what separates developers who fight the cascade from those who use it. A media query with `min-width` adds styles; a media query with `max-width` overrides them. The browser's default cascade already goes from nothing to everything — mobile-first CSS goes with the grain. Desktop-first CSS fights it. Every time you see a CSS file full of `max-width` overrides, you're looking at someone who didn't understand this.

This chapter builds `responsive.css` from scratch: mobile-first media queries, fluid typography with `clamp()`, container queries, and a complete breakpoint system. You'll understand not just the syntax but the reasoning behind every decision modern CSS frameworks make.

## The Problem

You have a design that looks right on a 1440px monitor. Now make it work on a 375px phone. The naive approach:

```css
/* Desktop-first — the mistake */
.sidebar {
  width: 300px;
  float: left;
}

.content {
  margin-left: 320px;
}

/* Now override everything for mobile */
@media (max-width: 768px) {
  .sidebar {
    width: 100%;
    float: none;
  }
  .content {
    margin-left: 0;
  }
}
```

Every desktop-first stylesheet eventually becomes a pile of `max-width` overrides fighting the base styles. The bigger the project, the more overrides accumulate. Specificity wars break out. Someone adds `!important`. It becomes unmaintainable.

The second problem is fixed values. `font-size: 32px` on a 4K monitor looks fine. On a 320px phone it's comically large. On a 1920px monitor it's too small for a headline. You're writing three stylesheets mentally — for phone, tablet, and desktop — but shipping one. The `clamp()` function solves this with a single declaration.

The third problem is that media queries respond to the viewport, not the element. A card component that should be two-column at 600px wide is two-column whether it's in a 600px sidebar or a 1200px main area. Container queries fix this, and they're now in every major browser.

## Building It Step by Step

### v1: Media Queries — Mobile-First

Start with the correct mental model: write base styles for the smallest screen, then add complexity for larger screens.

```css
/* v1: mobile-first-basics.css */

/* Base styles — these apply everywhere */
.container {
  width: 100%;
  padding-inline: 1rem;
}

.columns {
  display: flex;
  flex-direction: column;   /* stack on mobile */
  gap: 1rem;
}

/* Tablet and up */
@media (min-width: 640px) {
  .columns {
    flex-direction: row;     /* side by side */
  }
}

/* Desktop and up */
@media (min-width: 1024px) {
  .container {
    max-width: 1200px;
    margin-inline: auto;
  }
}
```

Read this as prose: "By default, columns stack vertically. At 640px and wider, they go side by side. At 1024px and wider, the container gets a max-width and centers."

Each media query only *adds* information. Nothing gets overridden. The cascade works with you.

The breakpoint values are not magic numbers — they're the natural breakpoints of common viewport widths:

```
320px  — small phones (minimum reasonable width)
480px  — large phones
640px  — small tablets / landscape phones
768px  — tablets
1024px — laptops / small desktops
1280px — standard desktops
1536px — large desktops / wide monitors
```

Name them semantically in custom properties:

```css
/* Breakpoints as CSS custom properties (for documentation) */
:root {
  --bp-sm:  640px;
  --bp-md:  768px;
  --bp-lg:  1024px;
  --bp-xl:  1280px;
  --bp-2xl: 1536px;
}
```

Custom properties can't be used inside `@media` queries (media features don't accept `var()`), but they document your breakpoint system and can be used in JavaScript.

### v2: Fluid Typography and Spacing with `clamp()`

`clamp(minimum, preferred, maximum)` returns the middle value unless it's smaller than the minimum or larger than the maximum.

```css
/* v2: fluid typography */
h1 {
  font-size: clamp(1.5rem, 4vw, 3rem);
}
```

At 375px viewport: `4vw = 15px`. That's less than `1.5rem (24px)`, so the result is `24px`.
At 1200px viewport: `4vw = 48px`. That's more than `3rem (48px)`, so the result is `48px`.
At 750px viewport: `4vw = 30px`. Between bounds, so the result is `30px`.

The `preferred` value is typically a `vw` (viewport width) value. Calculate it with this formula:

```
preferred vw = (max_size - min_size) / (max_viewport - min_viewport) × 100

Example: h1 from 24px at 375px viewport to 48px at 1200px viewport
= (48 - 24) / (1200 - 375) × 100
= 24 / 825 × 100
= 2.9vw
```

A complete fluid type scale:

```css
:root {
  --text-xs:   clamp(0.75rem, 1.5vw, 0.875rem);
  --text-sm:   clamp(0.875rem, 2vw, 1rem);
  --text-base: clamp(1rem, 2.5vw, 1.125rem);
  --text-lg:   clamp(1.125rem, 3vw, 1.25rem);
  --text-xl:   clamp(1.25rem, 3.5vw, 1.5rem);
  --text-2xl:  clamp(1.5rem, 4vw, 2rem);
  --text-3xl:  clamp(1.875rem, 5vw, 2.5rem);
  --text-4xl:  clamp(2.25rem, 6vw, 3.5rem);
}
```

Apply the same principle to spacing:

```css
:root {
  --space-sm:  clamp(0.5rem, 1vw, 0.75rem);
  --space-md:  clamp(1rem, 2vw, 1.5rem);
  --space-lg:  clamp(1.5rem, 4vw, 3rem);
  --space-xl:  clamp(2rem, 6vw, 5rem);
}
```

Now your spacing and typography are proportional at every viewport width, with no jarring jumps at breakpoints.

### v3: Container Queries and `@layer`

Container queries respond to an element's *containing block*, not the viewport. Define a containment context, then query it:

```css
/* v3: container queries */

/* Mark the container */
.card-wrapper {
  container-type: inline-size;
  container-name: card;
}

/* Query the container, not the viewport */
@container card (min-width: 400px) {
  .card {
    display: grid;
    grid-template-columns: 120px 1fr;
  }
}
```

The card is single-column when its *container* is narrow, two-column when it's wide — regardless of the viewport. Put the same card in a narrow sidebar and a wide main area; it adapts to each context independently.

`@layer` (cascade layers) controls the order of style precedence across your entire stylesheet:

```css
/* Declare layer order — lower = lower priority */
@layer reset, base, components, utilities;

@layer reset {
  *, *::before, *::after { box-sizing: border-box; }
}

@layer base {
  body { font-size: 1rem; line-height: 1.6; }
}

@layer components {
  .card { padding: 1rem; border-radius: 8px; }
}

@layer utilities {
  .text-center { text-align: center; }
  .mt-4 { margin-top: 1rem; }
}
```

Styles in later-declared layers beat earlier layers, *regardless of specificity*. A utility class in the `utilities` layer beats a component style in `components` even if the component selector is more specific. This is how Tailwind CSS works — it wins because utilities are in the highest layer.

## The Complete Program

See `responsive.html` for the interactive demo. Below is the complete `responsive.css`:

```css
/* responsive.css — Chapter 6 */
/* A mobile-first responsive framework with fluid type, container queries, and @layer */

/* ─── Layer declaration ──────────────────────────────────── */
@layer reset, base, layout, components, utilities;

/* ─── Reset ──────────────────────────────────────────────── */
@layer reset {
  *, *::before, *::after { box-sizing: border-box; }
  html { -webkit-text-size-adjust: 100%; }
  body { margin: 0; }
  img, video { max-width: 100%; display: block; }
}

/* ─── Custom properties ──────────────────────────────────── */
@layer base {
  :root {
    /* Breakpoints (for reference — not usable in @media) */
    --bp-sm:  640px;
    --bp-md:  768px;
    --bp-lg:  1024px;
    --bp-xl:  1280px;

    /* Fluid type scale */
    --text-xs:   clamp(0.75rem,  1.5vw,  0.875rem);
    --text-sm:   clamp(0.875rem, 2vw,    1rem);
    --text-base: clamp(1rem,     2.5vw,  1.125rem);
    --text-lg:   clamp(1.125rem, 3vw,    1.25rem);
    --text-xl:   clamp(1.25rem,  3.5vw,  1.5rem);
    --text-2xl:  clamp(1.5rem,   4vw,    2rem);
    --text-3xl:  clamp(1.875rem, 5vw,    2.5rem);
    --text-4xl:  clamp(2.25rem,  6vw,    3.5rem);

    /* Fluid spacing */
    --space-xs:  clamp(0.25rem, 0.5vw,  0.5rem);
    --space-sm:  clamp(0.5rem,  1vw,    0.75rem);
    --space-md:  clamp(1rem,    2vw,    1.5rem);
    --space-lg:  clamp(1.5rem,  4vw,    3rem);
    --space-xl:  clamp(2rem,    6vw,    5rem);
  }

  body {
    font-size: var(--text-base);
    line-height: 1.6;
  }
}

/* ─── Layout ─────────────────────────────────────────────── */
@layer layout {
  .container {
    width: 100%;
    padding-inline: var(--space-md);
  }

  @media (min-width: 640px)  { .container { max-width: 640px;  margin-inline: auto; } }
  @media (min-width: 768px)  { .container { max-width: 768px;  } }
  @media (min-width: 1024px) { .container { max-width: 1024px; } }
  @media (min-width: 1280px) { .container { max-width: 1280px; } }

  /* Responsive columns with CSS Grid */
  .columns {
    display: grid;
    grid-template-columns: 1fr;
    gap: var(--space-md);
  }

  @media (min-width: 640px) {
    .columns-2 { grid-template-columns: repeat(2, 1fr); }
    .columns-3 { grid-template-columns: repeat(3, 1fr); }
  }

  @media (min-width: 1024px) {
    .columns-4 { grid-template-columns: repeat(4, 1fr); }
  }
}

/* ─── Components ─────────────────────────────────────────── */
@layer components {
  /* Card that responds to its container */
  .card-wrapper {
    container-type: inline-size;
    container-name: card;
  }

  .card {
    padding: var(--space-md);
    border-radius: 8px;
  }

  @container card (min-width: 400px) {
    .card--media {
      display: grid;
      grid-template-columns: 120px 1fr;
      gap: var(--space-md);
      align-items: start;
    }
  }
}

/* ─── Utilities ──────────────────────────────────────────── */
@layer utilities {
  /* Display */
  .hidden  { display: none !important; }
  .block   { display: block !important; }
  .flex    { display: flex  !important; }
  .grid    { display: grid  !important; }

  /* Responsive show/hide */
  @media (max-width: 639px)  { .hide-mobile  { display: none !important; } }
  @media (min-width: 640px)  { .show-mobile  { display: none !important; } }
  @media (max-width: 1023px) { .hide-tablet  { display: none !important; } }
  @media (min-width: 1024px) { .show-tablet  { display: none !important; } }

  /* Fluid type utilities */
  .text-xs   { font-size: var(--text-xs);   }
  .text-sm   { font-size: var(--text-sm);   }
  .text-base { font-size: var(--text-base); }
  .text-lg   { font-size: var(--text-lg);   }
  .text-xl   { font-size: var(--text-xl);   }
  .text-2xl  { font-size: var(--text-2xl);  }
  .text-3xl  { font-size: var(--text-3xl);  }
  .text-4xl  { font-size: var(--text-4xl);  }

  /* Fluid spacing utilities */
  .gap-sm { gap: var(--space-sm); }
  .gap-md { gap: var(--space-md); }
  .gap-lg { gap: var(--space-lg); }

  .p-sm { padding: var(--space-sm); }
  .p-md { padding: var(--space-md); }
  .p-lg { padding: var(--space-lg); }
}
```

## Walkthrough

### Mobile-First Cascade

The cascade is the order in which CSS rules are applied. `min-width` media queries add to the cascade; `max-width` queries override it.

```
Mobile-first (correct):

  Base CSS          ─────────────────────────────────► applies everywhere
  @media (min-width: 640px)  { ... }  ──────────────► adds styles at 640px+
  @media (min-width: 1024px) { ... }  ──────────────► adds styles at 1024px+

  CSS reads like a story: start simple, add complexity.
  Nothing overrides anything. Pure addition.

Desktop-first (incorrect):

  Base CSS          ─────────────────────────────────► desktop styles
  @media (max-width: 1023px) { ... }  ──────────────► override for tablet
  @media (max-width: 639px)  { ... }  ──────────────► override for mobile

  Reads backwards. Every breakpoint cancels the previous one.
  Overrides accumulate. Specificity fights break out.
```

The practical difference compounds at scale. A mobile-first stylesheet can add 20 breakpoint blocks with zero conflicts. A desktop-first stylesheet's 20 override blocks will conflict in creative ways.

### Breakpoint Scale

```
Viewport width scale (common device ranges):

  ┌────────────────────────────────────────────────────────────┐
  │ 320px                                                      │
  │ ├── Small phones (iPhone SE, Galaxy S)                     │
  │                                                            │
  │ 480px                                                      │
  │ ├── Large phones (iPhone 14 Pro Max, landscape phones)     │
  │                                                            │
  │ 640px  ← --bp-sm                                          │
  │ ├── Small tablets, landscape phones                        │
  │                                                            │
  │ 768px  ← --bp-md                                          │
  │ ├── iPad (portrait), medium tablets                        │
  │                                                            │
  │ 1024px ← --bp-lg                                          │
  │ ├── iPad (landscape), small laptops                        │
  │                                                            │
  │ 1280px ← --bp-xl                                          │
  │ ├── Standard laptops and desktops                          │
  │                                                            │
  │ 1536px ← --bp-2xl                                         │
  │ └── Large monitors, wide desktops                          │
  └────────────────────────────────────────────────────────────┘

These are guidelines. Real breakpoints should come from your content,
not from device dimensions. Add a breakpoint when the layout breaks,
not when a new device category starts.
```

### The `clamp()` Formula

`clamp(min, preferred, max)` is a three-parameter function. The `preferred` value is typically a `vw` unit that makes the value grow with the viewport.

```
clamp(MIN, PREFERRED, MAX)

      MIN ──── if preferred < min, return min
      MAX ──── if preferred > max, return max
      PREFERRED ── otherwise, return preferred

Example: clamp(1rem, 4vw, 3rem)

Viewport:  300px  → 4vw = 12px (0.75rem) → below min → return 1rem (16px)
Viewport:  500px  → 4vw = 20px (1.25rem) → in range  → return 1.25rem
Viewport:  800px  → 4vw = 32px (2rem)    → in range  → return 2rem
Viewport: 1200px  → 4vw = 48px (3rem)    → at max    → return 3rem
Viewport: 1600px  → 4vw = 64px (4rem)    → above max → return 3rem

                     1rem              3rem
      ──────────────┬──────────────────┬──────────────────
       clamped min  │  growing with vw │  clamped max
                   400px            1200px
```

Calculate the `vw` value precisely:

```
preferred_vw = (max_size_px - min_size_px) / (max_vp_px - min_vp_px) × 100

For h1: 24px at 400px viewport, 48px at 1200px viewport:
= (48 - 24) / (1200 - 400) × 100
= 24 / 800 × 100
= 3vw

Result: clamp(1.5rem, 3vw, 3rem)
```

### Viewport Units: `vh`, `dvh`, `svh`, `lvh`

`vh` seems simple: 1% of the viewport height. But on mobile browsers, the viewport height changes when the address bar hides on scroll. This breaks full-height sections.

```
Mobile browser — address bar visible:
  ┌────────────────┐  ─┐
  │ address bar    │   │ browser UI (e.g. 80px)
  ├────────────────┤  ─┤
  │                │   │
  │   visible      │   │ vh = this full height (includes hidden UI)
  │   content      │   │ svh = small viewport (UI visible)
  │                │   │ dvh = dynamic (changes with UI)
  │                │   │ lvh = large viewport (UI hidden)
  └────────────────┘  ─┘

Mobile browser — address bar hidden (scrolled):
  ┌────────────────┐  ─┐
  │                │   │
  │   visible      │   │ lvh = this full height
  │   content      │   │ dvh = matches current state
  │                │   │ svh = still the smaller value
  │                │   │
  └────────────────┘  ─┘

Rule: use dvh for "full-height" mobile elements. It tracks the actual
visible height as the browser UI appears and disappears.
```

```css
/* Old, broken on mobile */
.hero { height: 100vh; }

/* Correct: use dvh with vh fallback */
.hero {
  height: 100vh;           /* fallback for browsers without dvh */
  height: 100dvh;          /* dynamic viewport height — use this */
}
```

### Container Queries vs. Media Queries

```
Media query: responds to the VIEWPORT

  Viewport: 800px wide
  ┌─────────────────────────────────────┐
  │                                     │
  │  .sidebar (250px)  .main (500px)    │
  │  ┌──────────────┐  ┌─────────────┐  │
  │  │ .card        │  │ .card       │  │
  │  │ [narrow]     │  │ [wide]      │  │
  │  └──────────────┘  └─────────────┘  │
  └─────────────────────────────────────┘

  @media (min-width: 640px) targets the viewport (800px ✓)
  BOTH cards get the "wide" layout — even the sidebar card!

Container query: responds to the CONTAINER

  @container (min-width: 400px) targets the containing element
  sidebar card container is 250px → narrow layout ✓
  main card container is 500px → wide layout ✓

  Same component, correct behavior in each context.
```

```css
/* Container query setup */
.card-wrapper {
  container-type: inline-size;   /* watch inline (horizontal) size */
  container-name: card;          /* optional: name it for targeting */
}

/* The component responds to its own container */
@container card (min-width: 400px) {
  .card { display: grid; grid-template-columns: 120px 1fr; }
}

/* No name — responds to nearest container ancestor */
@container (min-width: 300px) {
  .card-title { font-size: 1.25rem; }
}
```

### `@layer` — Cascade Layers

Without layers, specificity determines which rule wins. Layers add a layer *above* specificity: higher layers always win, regardless of selector specificity.

```
Without @layer: specificity decides
  .card .title         specificity: 0-2-0  ← wins
  .text-xl             specificity: 0-1-0

With @layer:
  @layer base, components, utilities;

  @layer components { .card .title { ... } }   layer: components
  @layer utilities  { .text-xl { ... } }       layer: utilities

  utilities layer > components layer → .text-xl wins
  regardless of specificity

Layer order (declared in @layer statement):
  ┌────────────────────────────────────────────────┐
  │ reset       lowest priority — browser overrides│
  │ base        body defaults, type, color         │
  │ layout      structural containers and grids    │
  │ components  reusable UI patterns               │
  │ utilities   single-purpose overrides           │ highest priority
  └────────────────────────────────────────────────┘
```

Unlayered styles (not inside any `@layer`) win over *all* layered styles. This means third-party CSS loaded without layers beats all your layered code. Wrap third-party imports in a low-priority layer to prevent this:

```css
@layer reset, third-party, base, components, utilities;

@layer third-party {
  @import url("some-library.css");
}
```

## Guided Try It — Build a Responsive Navigation Bar

A navigation bar is the canonical responsive component: horizontal on desktop, hamburger-toggled on mobile. Let's build it purely with CSS, no JavaScript for the layout.

### Step 1: Mobile layout — stacked, hidden by default

```html
<nav class="nav">
  <div class="nav__brand">Brand</div>
  <input type="checkbox" id="nav-toggle" class="nav__toggle-input">
  <label for="nav-toggle" class="nav__toggle">☰</label>
  <ul class="nav__menu">
    <li><a href="#" class="nav__link">Home</a></li>
    <li><a href="#" class="nav__link">About</a></li>
    <li><a href="#" class="nav__link">Work</a></li>
    <li><a href="#" class="nav__link">Contact</a></li>
  </ul>
</nav>
```

```css
.nav {
  display: flex;
  flex-wrap: wrap;
  align-items: center;
  justify-content: space-between;
  padding: 0.75rem 1rem;
  background: #111122;
}

.nav__toggle-input { display: none; }  /* hidden checkbox */

.nav__menu {
  display: none;              /* hidden by default on mobile */
  width: 100%;
  flex-direction: column;
  list-style: none;
  padding: 0;
  margin: 0;
}

/* Show menu when checkbox is checked */
.nav__toggle-input:checked ~ .nav__menu {
  display: flex;
}
```

### Step 2: Tablet and up — horizontal layout, no hamburger

```css
@media (min-width: 640px) {
  .nav__toggle { display: none; }   /* hide hamburger */

  .nav__menu {
    display: flex !important;        /* always visible */
    flex-direction: row;
    width: auto;
    gap: 0.5rem;
  }
}
```

### Step 3: Fluid spacing and transitions

```css
.nav__link {
  display: block;
  padding: clamp(0.4rem, 1vw, 0.6rem) clamp(0.75rem, 2vw, 1.25rem);
  color: #c8c8d8;
  text-decoration: none;
  border-radius: 4px;
  transition: background 0.15s, color 0.15s;
}

.nav__link:hover {
  background: rgba(168, 216, 234, 0.1);
  color: #a8d8ea;
}
```

The hamburger toggle works with zero JavaScript: the `<label>` toggles the hidden `<input type="checkbox">`, and the CSS `~` sibling selector reveals the menu when the checkbox is checked.

**Think about it**: This CSS-only hamburger toggle has an accessibility problem — the menu state isn't communicated to screen readers. The `aria-expanded` attribute on the toggle button should reflect the open/closed state. How would you add JavaScript to update `aria-expanded` without changing the CSS layout logic?

## Exercises

1. **Fluid Type Scale Generator**: Create a CSS file that implements a complete fluid type scale using `clamp()`. The scale should have 8 sizes from `--text-xs` to `--text-4xl`. At 375px viewport, the smallest is 12px and the largest is 36px. At 1280px viewport, the smallest is 14px and the largest is 56px. Calculate the exact `vw` values for each size using the formula from this chapter.

2. **Breakpoint-Annotated Layout**: Build a three-section layout (full-width banner, two-column content, full-width footer). At mobile: everything stacks. At 640px: the two-column section goes side by side. At 1024px: the content column has a max-width and centers. Add a small fixed indicator in the corner that shows the current breakpoint label (xs/sm/md/lg) using CSS-only `content` + `counter` tricks.

3. **Container-Aware Card**: Build a `.card--media` component with an image, title, and description. When its container is less than 400px, it's single-column (image above text). When its container is 400px–700px, it's two-column (image left, text right). When its container is over 700px, it's two-column with a larger image ratio. Demonstrate this in a three-column layout where the card appears in different-sized columns.

4. **Layer Architecture**: Take a stylesheet with specificity conflicts — a utility class `.hidden { display: none }` that loses to a component `.modal .content { display: block }` — and restructure it using `@layer` so utilities always win without using `!important`. Then add a third-party library import that would normally override your styles, and wrap it in a low-priority layer.

5. **Responsive Table**: HTML tables notoriously break on mobile. Build a responsive table where on mobile, each table row becomes a card. Use `data-label` attributes on `<td>` elements and CSS `content: attr(data-label)` to show labels in the mobile card view. No JavaScript. The `<table>` element itself becomes a block, and `<tr>` becomes a card with `display: grid`.

## Solutions

### Exercise 1: Fluid Type Scale Generator

```css
/* fluid-type-scale.css */
:root {
  /* Formula: vw = (max_px - min_px) / (max_vp - min_vp) * 100 */
  /* Viewport range: 375px to 1280px */
  /* (max_px - min_px) / (1280 - 375) * 100 = (diff / 905) * 100 */

  /* --text-xs:  12px → 14px,  diff=2,   vw = (2/905)*100 = 0.22vw  ≈ 0.5vw  (clipped) */
  --text-xs:   clamp(0.75rem, 0.5vw + 0.6rem, 0.875rem);

  /* --text-sm:  14px → 16px,  diff=2,   vw ≈ 0.22vw */
  --text-sm:   clamp(0.875rem, 0.5vw + 0.7rem, 1rem);

  /* --text-base:16px → 18px,  diff=2,   vw ≈ 0.22vw */
  --text-base: clamp(1rem, 0.5vw + 0.8rem, 1.125rem);

  /* --text-lg:  18px → 20px,  diff=2,   vw ≈ 0.22vw */
  --text-lg:   clamp(1.125rem, 0.5vw + 0.9rem, 1.25rem);

  /* --text-xl:  20px → 24px,  diff=4,   vw = (4/905)*100 = 0.44vw */
  --text-xl:   clamp(1.25rem, 0.5vw + 1rem, 1.5rem);

  /* --text-2xl: 24px → 32px,  diff=8,   vw = (8/905)*100 = 0.88vw */
  --text-2xl:  clamp(1.5rem, 1vw + 1.1rem, 2rem);

  /* --text-3xl: 28px → 44px,  diff=16,  vw = (16/905)*100 = 1.77vw */
  --text-3xl:  clamp(1.75rem, 2vw + 1.1rem, 2.75rem);

  /* --text-4xl: 36px → 56px,  diff=20,  vw = (20/905)*100 = 2.21vw */
  --text-4xl:  clamp(2.25rem, 3vw + 1rem, 3.5rem);
}
```

### Exercise 2: Breakpoint Indicator

```html
<style>
  body::after {
    content: 'xs';
    position: fixed;
    bottom: 1rem;
    right: 1rem;
    background: #1a1a3e;
    color: #a8d8ea;
    padding: 0.25rem 0.5rem;
    border: 1px solid #a8d8ea;
    border-radius: 4px;
    font-family: monospace;
    font-size: 0.75rem;
    z-index: 9999;
  }

  @media (min-width: 640px)  { body::after { content: 'sm'; } }
  @media (min-width: 768px)  { body::after { content: 'md'; } }
  @media (min-width: 1024px) { body::after { content: 'lg'; } }
  @media (min-width: 1280px) { body::after { content: 'xl'; } }
</style>
```

### Exercise 3: Container-Aware Card

```html
<style>
  .three-col {
    display: grid;
    grid-template-columns: 1fr 2fr 3fr;
    gap: 1rem;
  }

  .col { container-type: inline-size; container-name: col; }

  .card--media { display: grid; }

  .card__image {
    background: #1a1a3e;
    aspect-ratio: 16/9;
    border-radius: 4px;
  }

  /* Narrow container: stacked */
  .card--media { grid-template-columns: 1fr; }

  /* Medium container: 1:2 split */
  @container col (min-width: 400px) {
    .card--media {
      grid-template-columns: 1fr 2fr;
      gap: 1rem;
      align-items: start;
    }
    .card__image { aspect-ratio: 1/1; }
  }

  /* Wide container: 1:3 split */
  @container col (min-width: 700px) {
    .card--media { grid-template-columns: 1fr 3fr; }
  }
</style>

<div class="three-col">
  <div class="col"><div class="card--media"><div class="card__image"></div><div><h3>Title</h3><p>Text</p></div></div></div>
  <div class="col"><div class="card--media"><div class="card__image"></div><div><h3>Title</h3><p>Text</p></div></div></div>
  <div class="col"><div class="card--media"><div class="card__image"></div><div><h3>Title</h3><p>Text</p></div></div></div>
</div>
```

### Exercise 4: Layer Architecture

```css
@layer reset, third-party, base, components, utilities;

/* Third-party — wrapped in low-priority layer */
@layer third-party {
  /* @import "some-library.css"; */
  .modal .content { display: block; background: red; }
}

@layer components {
  .modal .content { padding: 1rem; }
}

/* Utilities beat everything — even .modal .content */
@layer utilities {
  .hidden { display: none; }  /* no !important needed */
}
```

Test: `.modal .content.hidden` will be hidden. Without layers, `.modal .content` (0-2-0) beats `.hidden` (0-1-0).

### Exercise 5: Responsive Table

```html
<style>
  .table-responsive { width: 100%; }

  .table-responsive th { text-align: left; padding: 0.5rem; background: #1a1a3e; }
  .table-responsive td { padding: 0.5rem; }
  .table-responsive tr:nth-child(even) td { background: #111122; }

  @media (max-width: 639px) {
    .table-responsive,
    .table-responsive tbody,
    .table-responsive tr,
    .table-responsive td {
      display: block;
      width: 100%;
    }
    .table-responsive thead { display: none; }
    .table-responsive tr {
      margin-bottom: 1rem;
      border: 1px solid #2a2a40;
      border-radius: 6px;
      overflow: hidden;
    }
    .table-responsive td {
      display: grid;
      grid-template-columns: 120px 1fr;
      gap: 0.5rem;
    }
    .table-responsive td::before {
      content: attr(data-label);
      font-weight: bold;
      color: #a8d8ea;
    }
  }
</style>

<table class="table-responsive">
  <thead>
    <tr><th>Name</th><th>Role</th><th>Status</th></tr>
  </thead>
  <tbody>
    <tr>
      <td data-label="Name">Alice</td>
      <td data-label="Role">Engineer</td>
      <td data-label="Status">Active</td>
    </tr>
    <tr>
      <td data-label="Name">Bob</td>
      <td data-label="Role">Designer</td>
      <td data-label="Status">Away</td>
    </tr>
  </tbody>
</table>
```

## What You Learned

| Concept | Key point |
|---|---|
| Mobile-first | Write base styles for small screens; use `min-width` queries to add complexity for larger screens |
| `min-width` vs `max-width` | `min-width` adds styles (progressive); `max-width` overrides them (regressive). Always prefer `min-width` |
| Breakpoints | Should come from your content breaking, not device categories. Common values: 640, 768, 1024, 1280px |
| `clamp(min, preferred, max)` | Returns `preferred` unless it's outside `[min, max]`; enables fluid scaling without media queries |
| `vw` in clamp | Use the formula `(max_size - min_size) / (max_vp - min_vp) × 100` to calculate the preferred vw value |
| `dvh` | Dynamic viewport height — tracks actual visible height as mobile browser UI shows/hides. Prefer over `vh` |
| Container queries | `container-type: inline-size` + `@container (min-width: N)` — responds to the element's container, not the viewport |
| `@layer` | Declares priority order for style blocks; later layers win regardless of specificity |
| Unlayered styles | Beat all layered styles — wrap third-party CSS in a low-priority `@layer` to prevent it overriding your code |
| Fluid spacing | Apply `clamp()` to spacing variables too, not just typography — creates consistent proportional spacing at every size |

## Building with Claude

**Bad prompt:**
> "Make my website responsive"

This is too vague to be useful. Responsive how? What breakpoints? What changes at each breakpoint?

**Good prompt:**
> "I have a desktop layout with `display: grid; grid-template-columns: 300px 1fr 300px` for a three-column layout. I need mobile-first responsive CSS for this component. At mobile (< 640px): single column, sidebars stack below main. At tablet (640px–1023px): two columns, right sidebar hidden. At desktop (1024px+): the current three-column layout. Use `@media (min-width: ...)` queries only, no `max-width`. The base styles should be the mobile layout. Give me only the responsive CSS, not the full component."

This tells Claude the current structure, the exact three breakpoint behaviors, the mobile-first requirement, and the scope. You'll get a precise, usable answer.

**Connection to the JS book**: The responsive breakpoints in Chapter 10's Router demo ensure the sidebar collapses on mobile. The fluid typography in the test framework output (Chapter 7) uses `clamp()` for readable output on any screen size — the terminal-style output stays readable from a phone to a 4K monitor without a single media query.
