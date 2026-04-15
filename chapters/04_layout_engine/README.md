# Chapter 4 — Build Your Own Layout Engine

Before CSS Grid and Flexbox, web layouts were built from floats and absolute positioning — hacks borrowed from print and desktop UI that were never designed for the web. Today's layout model is built for the web: Flexbox for one-dimensional alignment, CSS Grid for two-dimensional structure. In this chapter you build a layout library that exposes both systems as a composable set of utilities. By the end, you'll understand not just the syntax but *why* the axis model works the way it does, why `auto` margins absorb space, and how `fr` units distribute space without overflow.

---

## The Problem

The naive approach to layout is writing custom CSS for every container:

```css
.header    { display: flex; justify-content: space-between; align-items: center; }
.sidebar   { display: flex; flex-direction: column; gap: 16px; }
.card-grid { display: grid; grid-template-columns: repeat(3, 1fr); gap: 24px; }
.centered  { display: grid; place-items: center; }
```

This works, but the knowledge is trapped in component-specific CSS. When you look at `.card-grid`, you don't see a pattern — you see an incantation. Understanding layout means seeing that `justify-content: space-between` distributes space along the *main axis*, and `align-items: center` centers on the *cross axis*, and those axes flip when you change `flex-direction`. The two rules are the same rule applied to two different axes.

---

## Building It Step by Step

### v1 — Flexbox Utilities

The Flexbox model has one key concept: the main axis and the cross axis.

```
flex-direction: row (default)
   main axis →  [item] [item] [item]
   cross axis ↓

flex-direction: column
   main axis ↓
   [item]
   [item]
   cross axis →
```

`justify-content` controls the main axis. `align-items` controls the cross axis. Everything else is a variation:

```css
/* v1: flex utilities */

/* ── Containers ───────────────────────────────────── */
.flex   { display: flex; }
.flex-col { display: flex; flex-direction: column; }
.flex-wrap { flex-wrap: wrap; }

/* ── Main axis (justify-content) ──────────────────── */
.justify-start   { justify-content: flex-start; }
.justify-end     { justify-content: flex-end; }
.justify-center  { justify-content: center; }
.justify-between { justify-content: space-between; }
.justify-around  { justify-content: space-around; }
.justify-evenly  { justify-content: space-evenly; }

/* ── Cross axis (align-items) ─────────────────────── */
.items-start    { align-items: flex-start; }
.items-end      { align-items: flex-end; }
.items-center   { align-items: center; }
.items-baseline { align-items: baseline; }
.items-stretch  { align-items: stretch; }

/* ── Gap ──────────────────────────────────────────── */
.gap-1 { gap: 0.25rem; }
.gap-2 { gap: 0.5rem;  }
.gap-4 { gap: 1rem;    }
.gap-6 { gap: 1.5rem;  }
.gap-8 { gap: 2rem;    }
```

### v2 — Grid Utilities

CSS Grid is two-dimensional. The key concepts are tracks, lines, and placement:

```css
/* v2: grid utilities */

/* ── Containers ───────────────────────────────────── */
.grid      { display: grid; }
.grid-cols-1 { grid-template-columns: repeat(1, 1fr); }
.grid-cols-2 { grid-template-columns: repeat(2, 1fr); }
.grid-cols-3 { grid-template-columns: repeat(3, 1fr); }
.grid-cols-4 { grid-template-columns: repeat(4, 1fr); }
.grid-cols-6 { grid-template-columns: repeat(6, 1fr); }
.grid-cols-12{ grid-template-columns: repeat(12, 1fr); }

/* ── Auto-responsive grid (no media queries needed) ── */
.grid-auto-sm { grid-template-columns: repeat(auto-fill, minmax(200px, 1fr)); }
.grid-auto-md { grid-template-columns: repeat(auto-fill, minmax(280px, 1fr)); }
.grid-auto-lg { grid-template-columns: repeat(auto-fill, minmax(360px, 1fr)); }

/* ── Placement ────────────────────────────────────── */
.col-span-1  { grid-column: span 1; }
.col-span-2  { grid-column: span 2; }
.col-span-3  { grid-column: span 3; }
.col-span-full { grid-column: 1 / -1; }

/* ── Row gap / column gap ─────────────────────────── */
.gap-x-4 { column-gap: 1rem; }
.gap-y-4 { row-gap: 1rem; }
```

`auto-fill` with `minmax(200px, 1fr)` creates an automatically responsive grid. At 600px viewport: 3 columns of ≥200px each. At 400px: 2 columns. At 300px: 1 column. No media queries.

### v3 — Composition and Centering

The most useful patterns compose `flex` and `grid` together:

```css
/* v3: composition utilities */

/* ── Full centering ───────────────────────────────── */
.place-center   { display: grid; place-items: center; }   /* grid shorthand */
.flex-center    { display: flex; align-items: center; justify-content: center; }

/* ── Stack layout ─────────────────────────────────── */
.stack { display: flex; flex-direction: column; }
.stack > * + * { margin-top: var(--stack-gap, 1rem); }   /* lobotomized owl */

/* ── Cluster layout ───────────────────────────────── */
.cluster {
  display: flex;
  flex-wrap: wrap;
  gap: var(--cluster-gap, 0.75rem);
  align-items: center;
}

/* ── Sidebar layout ───────────────────────────────── */
.with-sidebar {
  display: flex;
  flex-wrap: wrap;
  gap: var(--sidebar-gap, 1rem);
}
.with-sidebar > .sidebar { flex-basis: 200px; flex-grow: 1; }
.with-sidebar > .main    { flex-basis: 0; flex-grow: 999; min-width: 50%; }

/* ── Holy grail layout ────────────────────────────── */
.page-layout {
  display: grid;
  grid-template:
    "header header header" auto
    "nav    main   aside"  1fr
    "footer footer footer" auto
    / 200px 1fr 200px;
  min-height: 100vh;
}
```

The `with-sidebar` pattern is particularly powerful: the sidebar takes its natural width, the main content grows to fill the rest — and if there isn't enough room (under `min-width: 50%`), it wraps to a full-width column. Responsive without a media query.

---

## The Complete Program

`layout-engine.html` — open in your browser. Use the interactive builder to compose layout classes and see the result in real time.

```html
<!DOCTYPE html>
<html lang="en">
<head><meta charset="UTF-8"><title>Build Your Own Layout Engine — Chapter 4</title>
<!-- styles in layout-engine.html -->
</head>
<body>
<style>
/* ═══════════════════════════════════════════════════════════════════════
   THE LIBRARY: miniLayout
   ═══════════════════════════════════════════════════════════════════════ */

/* ── Flex ───────────────────────────────────────────────────────────── */
.flex          { display: flex; }
.flex-col      { display: flex; flex-direction: column; }
.flex-wrap     { flex-wrap: wrap; }
.flex-1        { flex: 1; }
.flex-auto     { flex: 1 1 auto; }
.flex-none     { flex: none; }
.flex-shrink-0 { flex-shrink: 0; }

.justify-start   { justify-content: flex-start; }
.justify-end     { justify-content: flex-end; }
.justify-center  { justify-content: center; }
.justify-between { justify-content: space-between; }
.justify-around  { justify-content: space-around; }

.items-start    { align-items: flex-start; }
.items-end      { align-items: flex-end; }
.items-center   { align-items: center; }
.items-baseline { align-items: baseline; }
.items-stretch  { align-items: stretch; }

/* ── Grid ───────────────────────────────────────────────────────────── */
.grid         { display: grid; }
.grid-cols-1  { grid-template-columns: repeat(1, 1fr); }
.grid-cols-2  { grid-template-columns: repeat(2, 1fr); }
.grid-cols-3  { grid-template-columns: repeat(3, 1fr); }
.grid-cols-4  { grid-template-columns: repeat(4, 1fr); }
.grid-auto-sm { grid-template-columns: repeat(auto-fill, minmax(160px, 1fr)); }
.grid-auto-md { grid-template-columns: repeat(auto-fill, minmax(240px, 1fr)); }
.col-span-2   { grid-column: span 2; }
.col-span-full{ grid-column: 1 / -1; }
.place-center { display: grid; place-items: center; }

/* ── Gap ────────────────────────────────────────────────────────────── */
.gap-1 { gap: 0.25rem; } .gap-2 { gap: 0.5rem; }
.gap-4 { gap: 1rem; }    .gap-6 { gap: 1.5rem; }
.gap-8 { gap: 2rem; }

/* ── Composition ────────────────────────────────────────────────────── */
.cluster { display: flex; flex-wrap: wrap; gap: 0.75rem; align-items: center; }
.stack   { display: flex; flex-direction: column; gap: 1rem; }
.center  { margin-inline: auto; max-width: var(--center-max, 65ch); }
</style>
</body>
</html>
```

---

## Walkthrough

### The Flexbox Axis Model

```
flex-direction: row (default)
──────────────────────────────────────────────────────
main axis ────────────────────────────────────────────→
│  ┌────────┐  ┌────────┐  ┌────────┐               │
│  │ item 1 │  │ item 2 │  │ item 3 │               │
│  └────────┘  └────────┘  └────────┘               │
cross axis ↓

justify-content controls spacing along the main axis (→)
align-items controls alignment on the cross axis (↓)


flex-direction: column
──────────────────────────────────────────────────────
main axis ↓         cross axis →
│  ┌────────────────────────────────────────────┐
│  │                 item 1                     │
│  └────────────────────────────────────────────┘
│  ┌────────────────────────────────────────────┐
│  │                 item 2                     │
│  └────────────────────────────────────────────┘

Now justify-content controls the vertical axis (↓)
and align-items controls the horizontal axis (→)
```

This is the central insight: `justify-content` and `align-items` don't mean "horizontal" and "vertical". They mean "main axis" and "cross axis". Switch `flex-direction` and they switch.

### `fr` Units — Fractional Space

`fr` is a unit unique to CSS Grid. It means "a fraction of the available space after fixed sizes have been subtracted":

```css
grid-template-columns: 200px 1fr 200px;

Available space calculation:
Total width:        1200px
Minus fixed:        200px + 200px = 400px
Available for fr:   1200px - 400px = 800px
1fr = 100% of remaining = 800px

Result: [200px] [800px] [200px]
```

Multiple `fr` units divide proportionally:

```css
grid-template-columns: 1fr 2fr 1fr;
/* 1200px total: 300px | 600px | 300px */

grid-template-columns: 1fr 1fr 1fr;
/* Equal thirds: 400px | 400px | 400px */
/* Same as: repeat(3, 1fr) */
```

### `auto-fill` vs `auto-fit`

Both create as many columns as fit. The difference appears with fewer items than columns:

```
Container width: 600px, minmax(200px, 1fr)

auto-fill creates columns even if empty:
[item1] [item2] [empty] [empty]
←200px→ ←200px→ ←200px→ ←empty→  ← no expansion

auto-fit collapses empty columns, then expands remaining:
[item1         ] [item2         ]
←    300px     → ←    300px     →  ← items expand to fill
```

Use `auto-fill` when you want a grid to maintain its column structure. Use `auto-fit` when you want items to expand and fill the available space.

### The Lobotomized Owl — `* + *`

```css
.stack > * + * { margin-top: 1rem; }
```

The `* + *` selector — called the "lobotomized owl" (it looks like one) — matches any element that is immediately preceded by another element. It adds spacing *between* items, not before the first or after the last:

```
Stack without owl:  Stack with owl:
┌──────────┐       ┌──────────┐
│  item 1  │       │  item 1  │
│          │       │          │
│  item 2  │       └──────────┘
│          │        1rem gap
│  item 3  │       ┌──────────┐
└──────────┘       │  item 2  │
                   └──────────┘
                    1rem gap
                   ┌──────────┐
                   │  item 3  │
                   └──────────┘
```

With `gap` in flexbox/grid, you can achieve the same effect more concisely. The owl is still useful for heterogeneous content where you don't control the immediate parent.

### Named Grid Areas

Grid areas give meaningful names to regions of a layout:

```css
.page {
  display: grid;
  grid-template-areas:
    "header header"
    "nav    main"
    "footer footer";
  grid-template-columns: 220px 1fr;
  grid-template-rows: auto 1fr auto;
}

.header { grid-area: header; }
.nav    { grid-area: nav; }
.main   { grid-area: main; }
.footer { grid-area: footer; }
```

The visual ASCII map in `grid-template-areas` is the layout. Dots (`.`) represent empty cells. A region spanning multiple rows/columns must form a rectangle.

---

## Guided Try It — Build a Masonry-Style Card Grid

**The goal**: create a card grid where tall cards automatically occupy more vertical space without leaving gaps — like Pinterest.

**Why this is useful**: CSS Grid's `grid-auto-rows` property combined with `grid-row: span N` creates this effect without JavaScript.

**Step 1 — Create the grid with row granularity**

```css
.masonry-grid {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(240px, 1fr));
  grid-auto-rows: 8px;   /* tiny row height — cards span multiple rows */
  gap: 0 16px;           /* horizontal gap only; rows handle vertical */
}
```

**Step 2 — Span rows based on content height**

```javascript
// After rendering, measure each card and set its row span
function sizeMasonryItem(item) {
  const grid = item.parentElement;
  const rowHeight = parseInt(getComputedStyle(grid).gridAutoRows);
  const gap = parseInt(getComputedStyle(grid).gap) || 0;

  const itemHeight = item.firstElementChild.getBoundingClientRect().height;
  const rowSpan = Math.ceil((itemHeight + gap) / (rowHeight + gap));
  item.style.gridRowEnd = `span ${rowSpan}`;
}
```

**Step 3 — Call on load and resize**

```javascript
function runMasonry() {
  document.querySelectorAll('.masonry-item').forEach(sizeMasonryItem);
}
runMasonry();
window.addEventListener('resize', runMasonry);
```

**Think about it**: this masonry approach requires JavaScript to measure and span rows. CSS-only masonry was proposed (using `grid-template-rows: masonry`) but is only behind a flag in Firefox. What's the CSS-only workaround? (Hint: CSS columns — `column-count: 3` — creates a masonry effect but breaks document order. Does that matter for your use case?)

---

## Exercises

1. **Add `self-` alignment utilities**: Create `.self-start`, `.self-end`, `.self-center`, `.self-stretch` using `align-self`. These align a *single* flex or grid item independently of the container's `align-items`. Test: in a `.flex.items-center` container, one item with `.self-start` should stick to the top.

2. **Build a `grid-cols-auto` utility**: Create a class that automatically picks the right number of columns based purely on the `min-content` of the items — no `minmax()`, no specified minimum. Use `grid-template-columns: repeat(auto-fit, minmax(0, 1fr))`. When does this beat `minmax(200px, 1fr)`, and when does it lose?

3. **Create a `flex-split` layout**: A common UI pattern — one item pushes to the left, one to the right. Build this with a `.flex-split` class. The key: `margin-left: auto` on the right item pushes it to the far end of the main axis. No `justify-content: space-between` needed.

4. **Build a 12-column offset system**: Bootstrap-style offset classes let you push a grid item to the right: `.offset-3` moves an item 3 columns to the right. Build `.offset-1` through `.offset-6` using `grid-column-start`. Test: a grid with `grid-cols-12` where `.col-span-4.offset-4` centers a block.

5. **Create an `aspect-ratio` utility set**: Build `.aspect-square` (1/1), `.aspect-video` (16/9), `.aspect-portrait` (3/4), and `.aspect-wide` (21/9). Use `aspect-ratio` property. These should work on grid cells, images, and containers. Demo: a 4-column grid of image placeholders all locked to 16:9.

---

## Solutions

### Exercise 1 — `self-` alignment utilities

```css
.self-auto    { align-self: auto; }
.self-start   { align-self: flex-start; }
.self-end     { align-self: flex-end; }
.self-center  { align-self: center; }
.self-stretch { align-self: stretch; }
.self-baseline{ align-self: baseline; }

/* Test: */
/* <div class="flex items-center gap-4" style="height:120px;background:#111">
     <div style="background:#333;padding:8px">Normal (centered)</div>
     <div class="self-start" style="background:#333;padding:8px">Stuck top</div>
     <div class="self-end" style="background:#333;padding:8px">Stuck bottom</div>
   </div> */
```

### Exercise 2 — `grid-cols-auto`

```css
.grid-cols-auto {
  grid-template-columns: repeat(auto-fit, minmax(0, 1fr));
}
```

`minmax(0, 1fr)` vs `minmax(200px, 1fr)`:
- `minmax(0, 1fr)` — columns can shrink to zero; equal fractions always. Use when items should always be equal width.
- `minmax(200px, 1fr)` — columns have a minimum size; wraps to fewer columns before shrinking. Use for card grids where cards below 200px are unusable.

### Exercise 3 — `flex-split`

```css
/* The split is created by one child having margin-left: auto */
.flex-split {
  display: flex;
  align-items: center;
}

/* Any child with this class gets pushed right */
.ml-auto { margin-left: auto; }

/* Usage:
<nav class="flex-split">
  <a href="/">Logo</a>
  <a href="/login" class="ml-auto">Login</a>
</nav>
*/
```

`margin: auto` in Flexbox absorbs all available free space along the main axis. It doesn't just center — it pushes to the far end. A single `margin-left: auto` pushes one item to the right. Two items with `margin: auto` and no content between would push to opposite edges.

### Exercise 4 — 12-column offset system

```css
/* Assumes .grid-cols-12 container */
.offset-1  { grid-column-start: 2; }
.offset-2  { grid-column-start: 3; }
.offset-3  { grid-column-start: 4; }
.offset-4  { grid-column-start: 5; }
.offset-5  { grid-column-start: 6; }
.offset-6  { grid-column-start: 7; }

/* Centered: offset-4 + col-span-4 = starts at col 5, ends at col 9 */
/* <div class="grid grid-cols-12 gap-4">
     <div class="col-span-4 offset-4" style="background:#333">
       Centered content
     </div>
   </div> */
```

### Exercise 5 — `aspect-ratio` utilities

```css
.aspect-square   { aspect-ratio: 1 / 1; }
.aspect-video    { aspect-ratio: 16 / 9; }
.aspect-portrait { aspect-ratio: 3 / 4; }
.aspect-wide     { aspect-ratio: 21 / 9; }

/* For images inside aspect-ratio containers: */
.aspect-square img,
.aspect-video  img {
  width: 100%;
  height: 100%;
  object-fit: cover;
}

/* Demo: 4-column video grid */
/* <div class="grid grid-auto-sm gap-4">
     <div class="aspect-video" style="background:#1a1a2e">16:9</div>
     <div class="aspect-video" style="background:#1a1a2e">16:9</div>
     <div class="aspect-video" style="background:#1a1a2e">16:9</div>
     <div class="aspect-video" style="background:#1a1a2e">16:9</div>
   </div> */
```

---

## What You Learned

| Concept | Key point |
|---------|-----------|
| Flex main axis | Direction of `flex-direction`; `justify-content` controls distribution |
| Flex cross axis | Perpendicular to main; `align-items` controls alignment |
| `fr` unit | Fraction of *available* space after fixed sizes removed; unique to Grid |
| `auto-fill` vs `auto-fit` | `auto-fill` keeps empty columns; `auto-fit` collapses them so items expand |
| `minmax(200px, 1fr)` | Responsive columns with a floor — wraps before shrinking below minimum |
| `grid-column: span N` | Makes an item span N column tracks from its auto-placement position |
| `grid-column: 1 / -1` | Spans all columns; `-1` means "last grid line" |
| `place-items: center` | Grid shorthand for `align-items: center; justify-items: center` |
| `margin-left: auto` | Absorbs all free space on the main axis — pushes to the right in row flex |
| Lobotomized owl `* + *` | Adds margin between siblings, never before the first or after the last |
| Named grid areas | `grid-template-areas` visual map creates named regions |

---

## Building with Claude

Bad prompt:
> "How do I center a div in CSS?"

Good prompt:
> "I'm building a layout library in plain CSS. I have `.place-center { display: grid; place-items: center; }` which works for centering a child inside a parent. But I need a different pattern: centering the container itself horizontally on the page with `max-width: 65ch` and auto margins. I'm using `margin-inline: auto` which works fine. The question is: should I make this a separate `.center` utility class, or combine it with a width constraint like `.container`? My concern is that sometimes I want the max-width constraint without centering (a left-aligned constrained column), and sometimes I want centering without a max-width. How would Tailwind handle this split, and is that the right call for a smaller utility library?"
