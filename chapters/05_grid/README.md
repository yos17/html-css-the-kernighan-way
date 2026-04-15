# Chapter 5 — Build Your Own Grid System

CSS Grid is the most powerful layout system ever added to CSS. Before it arrived, web developers spent a decade wrestling with floats, clearfixes, and table-based layouts — all hacks trying to solve a problem the language wasn't designed to handle. Grid solves it at the language level. A two-dimensional coordinate system, axis-aligned placement, named areas, and automatic item distribution: these aren't library features, they're the browser itself doing layout.

The reason to build a grid system yourself, rather than reaching for Bootstrap or a framework, is that every grid framework is just a thin vocabulary layer over the same underlying CSS Grid properties. When you understand `grid-template-columns`, `fr` units, and `auto-placement`, you understand all of them. Bootstrap's `.col-md-6` is `grid-column: span 6` with extra steps. Once you've built the system, you'll never be confused by a framework's grid docs again.

This chapter builds `grid-system.css` from first principles: a 12-column grid with named areas, responsive auto-fill columns, and the full mental model of how the browser's grid algorithm works. By the end, you'll be able to layout any page without a framework.

## The Problem

You need a two-column layout: sidebar on the left, content on the right. The naive approach:

```css
/* The float hack — don't do this */
.sidebar {
  float: left;
  width: 25%;
}
.content {
  float: left;
  width: 75%;
}
.container::after {
  content: '';
  display: block;
  clear: both;
}
```

This works until it doesn't. Equal-height columns require more hacks. Reordering on mobile requires JavaScript. Centering vertically is painful. And none of this is semantic — the CSS has nothing to do with layout concepts, it's just fighting the float spec.

Flexbox improved things enormously, but it's fundamentally one-dimensional. It handles a row of items, or a column of items. When you try to build a grid with Flexbox, you end up managing row-breaks manually, fighting `flex-wrap`, and losing control of alignment across rows. The layout becomes the browser's problem after you're done fighting it.

CSS Grid thinks in two dimensions from the start. You define rows and columns explicitly, then place items anywhere in that coordinate space. Or you define the structure and let the browser place items automatically — and the algorithm is smart enough to be useful.

## Building It Step by Step

### v1: Grid Basics

Start with the three properties you need to understand first: `display: grid`, `grid-template-columns`, and `gap`.

```css
/* v1: grid-basics.css */
.grid {
  display: grid;
  grid-template-columns: 1fr 1fr 1fr;
  gap: 1rem;
}
```

```html
<div class="grid">
  <div class="cell">One</div>
  <div class="cell">Two</div>
  <div class="cell">Three</div>
  <div class="cell">Four</div>
  <div class="cell">Five</div>
  <div class="cell">Six</div>
</div>
```

That's a three-column grid. Six children auto-place into two rows. The `fr` unit — fractional unit — means "one fraction of the available space." Three `1fr` columns each get one-third.

Notice what you didn't write: no floats, no clearfix, no widths, no margins. The browser does the layout.

`gap` (previously `grid-gap`) sets the gutter between all cells. It's shorthand for `row-gap` and `column-gap`.

To make column spans work, use `grid-column`:

```css
.cell--wide {
  grid-column: span 2;
}
```

`span 2` means "take up 2 column tracks." The auto-placement algorithm will fit it in, moving other items around as needed.

### v2: The 12-Column Grid

A 12-column grid is the standard because 12 is divisible by 2, 3, 4, and 6 — giving you halves, thirds, quarters, and sixths with clean math.

```css
/* v2: 12-column grid */
.grid-12 {
  display: grid;
  grid-template-columns: repeat(12, 1fr);
  gap: 1rem;
}

/* Column spans */
.col-1  { grid-column: span 1;  }
.col-2  { grid-column: span 2;  }
.col-3  { grid-column: span 3;  }
.col-4  { grid-column: span 4;  }
.col-6  { grid-column: span 6;  }
.col-8  { grid-column: span 8;  }
.col-9  { grid-column: span 9;  }
.col-10 { grid-column: span 10; }
.col-12 { grid-column: span 12; }
```

`repeat(12, 1fr)` is equivalent to writing `1fr 1fr 1fr 1fr 1fr 1fr 1fr 1fr 1fr 1fr 1fr 1fr`. Use `repeat()`.

You can also set explicit start and end lines:

```css
.sidebar {
  grid-column: 1 / 4;   /* start at line 1, end at line 4 = 3 columns */
}
.content {
  grid-column: 4 / 13;  /* start at line 4, end at line 13 = 9 columns */
}
```

Grid lines are numbered from 1 (not 0). A 12-column grid has 13 vertical lines. Negative numbers count from the end: `grid-column: 1 / -1` spans the full width.

### v3: Named Areas, Auto-Fill, and Responsive Grid

Named template areas are Grid's killer feature. You draw the layout in ASCII art and the browser obeys:

```css
/* v3: named areas + responsive */
.page-layout {
  display: grid;
  grid-template-columns: 250px 1fr;
  grid-template-rows: auto 1fr auto;
  grid-template-areas:
    "header  header"
    "sidebar content"
    "footer  footer";
  min-height: 100vh;
  gap: 0;
}

.page-header  { grid-area: header;  }
.page-sidebar { grid-area: sidebar; }
.page-content { grid-area: content; }
.page-footer  { grid-area: footer;  }
```

The `grid-template-areas` string is a visual map of your layout. Every row is a string. Every word is a cell name. Repeated names form a rectangle — an area. A `.` is an empty cell.

For responsive columns that adjust automatically, use `auto-fill` with `minmax()`:

```css
/* Responsive card grid — no media queries needed */
.card-grid {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(250px, 1fr));
  gap: 1.5rem;
}
```

This creates as many 250px-minimum columns as will fit. On a 1200px container: 4 columns. On a 600px container: 2 columns. On a 300px container: 1 column. No media queries.

## The Complete Program

The full `grid.html` demo file demonstrates all three versions interactively. Below is the complete `grid-system.css` listing:

```css
/* grid-system.css — Chapter 5 */
/* A 12-column CSS Grid system with named areas and responsive auto-fill */

/* ─── Reset ─────────────────────────────────────────────── */
*, *::before, *::after {
  box-sizing: border-box;
}

/* ─── Container ──────────────────────────────────────────── */
.container {
  width: 100%;
  max-width: 1200px;
  margin-inline: auto;
  padding-inline: 1rem;
}

/* ─── 12-Column Grid ─────────────────────────────────────── */
.grid {
  display: grid;
  grid-template-columns: repeat(12, 1fr);
  gap: var(--grid-gap, 1rem);
}

/* Column spans */
.col-1  { grid-column: span 1;  }
.col-2  { grid-column: span 2;  }
.col-3  { grid-column: span 3;  }
.col-4  { grid-column: span 4;  }
.col-5  { grid-column: span 5;  }
.col-6  { grid-column: span 6;  }
.col-7  { grid-column: span 7;  }
.col-8  { grid-column: span 8;  }
.col-9  { grid-column: span 9;  }
.col-10 { grid-column: span 10; }
.col-11 { grid-column: span 11; }
.col-12 { grid-column: span 12; }

/* Column offsets — skip N columns */
.offset-1  { grid-column-start: 2;  }
.offset-2  { grid-column-start: 3;  }
.offset-3  { grid-column-start: 4;  }
.offset-4  { grid-column-start: 5;  }
.offset-6  { grid-column-start: 7;  }

/* Responsive: collapse to 1 column on small screens */
@media (max-width: 640px) {
  .grid > * {
    grid-column: span 12 !important;
  }
}

/* ─── Named Template Areas ───────────────────────────────── */
.page-layout {
  display: grid;
  grid-template-columns: 240px 1fr;
  grid-template-rows: 60px 1fr 50px;
  grid-template-areas:
    "header  header"
    "sidebar content"
    "footer  footer";
  min-height: 100vh;
}

.page-header  { grid-area: header;  }
.page-sidebar { grid-area: sidebar; }
.page-content { grid-area: content; }
.page-footer  { grid-area: footer;  }

@media (max-width: 768px) {
  .page-layout {
    grid-template-columns: 1fr;
    grid-template-rows: auto;
    grid-template-areas:
      "header"
      "content"
      "sidebar"
      "footer";
  }
}

/* ─── Auto-Fill Card Grid ────────────────────────────────── */
.card-grid {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(240px, 1fr));
  gap: 1.5rem;
}

/* ─── Auto-Fit Card Grid ─────────────────────────────────── */
.card-grid--fit {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(240px, 1fr));
  gap: 1.5rem;
}

/* ─── Dense Auto-Placement ───────────────────────────────── */
.grid--dense {
  grid-auto-flow: dense;
}

/* ─── Centered Single Item ───────────────────────────────── */
.grid--center {
  display: grid;
  place-items: center;
}

/* ─── Full-bleed row inside a grid ──────────────────────────*/
.full-bleed {
  grid-column: 1 / -1;
}
```

## Walkthrough

### Grid Lines, Tracks, Cells, and Areas

Before you can use Grid fluently, you need precise vocabulary. The browser's devtools use these terms. The spec uses these terms. You should too.

```
Grid Lines — the dividing lines that form the grid structure
             numbered from 1, both horizontally and vertically

     1    2    3    4    5
  1  ┌────┬────┬────┬────┐
     │    │    │    │    │  ← Row Track (horizontal)
  2  ├────┼────┼────┼────┤
     │    │    │    │    │
  3  ├────┼────┼────┼────┤
     │    │    │    │    │
  4  └────┴────┴────┴────┘
     ↑
     Column Track (vertical)

     Each intersection rectangle = a Grid Cell
     A rectangular group of cells = a Grid Area
```

- **Grid line**: a dividing line. Column lines run vertically; row lines run horizontally.
- **Grid track**: the space between two adjacent lines. Column tracks are vertical bands; row tracks are horizontal bands.
- **Grid cell**: the intersection of one row track and one column track. The smallest unit.
- **Grid area**: any rectangular group of one or more cells. Named areas must be rectangles.

### The `fr` Unit — Fractional Space

The `fr` unit distributes free space after fixed sizes are removed.

```
Container: 900px wide, gap: 20px

grid-template-columns: 200px 1fr 2fr;

Step 1: Remove fixed sizes and gaps
  900px - 200px (fixed) - 20px (gap) - 20px (gap) = 660px free

Step 2: Count fractional units
  1fr + 2fr = 3fr total

Step 3: Distribute
  1fr = 660px / 3 = 220px
  2fr = 660px / 3 × 2 = 440px

Final:
  ┌──────────┬──────────────┬────────────────────────────┐
  │  200px   │    220px     │           440px            │
  │  (fixed) │    (1fr)     │           (2fr)            │
  └──────────┴──────────────┴────────────────────────────┘
```

`fr` only distributes *free* space. If a `1fr` column's content is wider than 220px, it will take more space and the other `fr` columns adjust. To prevent this, use `minmax(0, 1fr)` instead of `1fr` — this allows the column to shrink to zero.

### Explicit Grid vs. Implicit Grid

When grid items are placed outside the explicitly defined tracks, the browser creates *implicit* tracks automatically.

```
grid-template-columns: 1fr 1fr 1fr;
grid-template-rows: 100px;   ← only one explicit row

Items 1-3 go in the explicit row (100px tall)
Items 4-6 overflow into implicit rows

  ┌─────────┬─────────┬─────────┐
  │    1    │    2    │    3    │  ← explicit row: 100px
  ├─────────┼─────────┼─────────┤
  │    4    │    5    │    6    │  ← implicit row: auto height
  └─────────┴─────────┴─────────┘
```

Control implicit row height with `grid-auto-rows`:

```css
.grid {
  grid-template-columns: repeat(3, 1fr);
  grid-auto-rows: 200px;        /* all implicit rows are 200px */
}
```

Or use `minmax()` for flexible implicit rows:

```css
grid-auto-rows: minmax(100px, auto);  /* at least 100px, grows with content */
```

### `auto-fill` vs. `auto-fit`

These two values look identical until you have fewer items than columns. The difference is fundamental.

```
Container: 900px   minmax(200px, 1fr)
So: 4 columns fit. But we only have 2 items.

auto-fill:
  ┌──────┬──────┬──────┬──────┐
  │  1   │  2   │ (e) │ (e) │
  └──────┴──────┴──────┴──────┘
  Empty columns EXIST and take up space.
  Items do NOT expand to fill.

auto-fit:
  ┌──────────────┬──────────────┐
  │      1       │      2       │
  └──────────────┴──────────────┘
  Empty columns are COLLAPSED to 0px.
  Items DO expand to fill available space.
```

Use `auto-fill` when you want items at a consistent size and empty space on the right is fine — think a photo gallery.

Use `auto-fit` when you want items to always fill the full row — think a navigation bar or a set of stats cards.

### Grid Template Areas

The `grid-template-areas` property is a ASCII-art representation of your layout. The browser parses it literally.

```
grid-template-areas:
  "header  header  header"
  "sidebar content content"
  "footer  footer  footer";

Visual representation:
  ┌──────────────────────────────┐
  │           header             │
  ├────────┬─────────────────────┤
  │        │                     │
  │sidebar │      content        │
  │        │                     │
  ├────────┴─────────────────────┤
  │           footer             │
  └──────────────────────────────┘

Rules:
  - Every row must have the same number of cells
  - Named areas must be rectangular (no L-shapes)
  - Use a dot (.) for an empty cell
  - Repeating a name across cells creates one area
```

Named areas automatically create named lines. If you have an area called `sidebar`, the browser creates `sidebar-start` and `sidebar-end` lines on both axes. You can use those in `grid-column` and `grid-row`:

```css
.sidebar { grid-area: sidebar; }
/* equivalent to: */
.sidebar {
  grid-column: sidebar-start / sidebar-end;
  grid-row: sidebar-start / sidebar-end;
}
```

### Flexbox vs. Grid — When to Use Each

This question comes up constantly. The answer is precise.

```
Flexbox: one-dimensional layout
  ┌────┬────┬────┬────┐
  │    │    │    │    │  ← one row of items
  └────┴────┴────┴────┘
  Items flow in one direction.
  The container responds to item sizes.
  Best for: nav bars, button groups, centering, card internals.

Grid: two-dimensional layout
  ┌────┬────┬────┐
  │    │    │    │  ← row 1
  ├────┼────┼────┤
  │    │    │    │  ← row 2
  └────┴────┴────┘
  You define the structure first, items fill it.
  Best for: page layouts, card grids, forms, any 2D arrangement.
```

The practical rule: use Grid for the page macro-layout (header, sidebar, main, footer) and for item grids (cards, photos, results). Use Flexbox for the micro-layout of components themselves (icon + text in a button, items in a navbar).

They compose well together. A grid cell can be a flex container. A flex item can be a grid container.

## Guided Try It — Add a Masonry-Style Dense Grid

CSS Grid's auto-placement algorithm can fill in gaps that shorter items leave behind. This creates a dense packing behavior. Let's build it.

### Step 1: Create a basic auto-fill grid with items of varying heights

```html
<div class="masonry-grid">
  <div class="card card--tall">Tall card</div>
  <div class="card">Normal card</div>
  <div class="card card--short">Short card</div>
  <div class="card card--tall">Tall card</div>
  <div class="card">Normal card</div>
  <div class="card card--short">Short card</div>
  <div class="card">Normal card</div>
</div>
```

```css
.masonry-grid {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(200px, 1fr));
  gap: 1rem;
}

.card { padding: 1rem; background: #1a1a2e; }
.card--tall  { height: 200px; }
.card--short { height: 80px;  }
/* Default height: ~120px from padding */
```

Right now, shorter items leave visible gaps below them because the grid places items in document order.

### Step 2: Enable dense auto-placement

```css
.masonry-grid {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(200px, 1fr));
  grid-auto-rows: 80px;   /* unit rows */
  gap: 1rem;
  grid-auto-flow: dense;  /* fill gaps with later items */
}

.card--tall  { grid-row: span 3; }  /* 3 × 80px = 240px */
.card--short { grid-row: span 1; }  /* 1 × 80px = 80px  */
.card        { grid-row: span 2; }  /* 2 × 80px = 160px */
```

Now `grid-auto-flow: dense` tells the browser: when placing items, go back and fill any gaps if a later item will fit. Items may appear out of order visually, but the layout will be denser.

### Step 3: Add a "featured" card that spans both columns and rows

```css
.card--featured {
  grid-column: span 2;
  grid-row: span 2;
  background: #16213e;
  font-size: 1.25rem;
}
```

```html
<div class="masonry-grid">
  <div class="card card--featured">Featured</div>
  <div class="card card--tall">Tall</div>
  <div class="card">Normal</div>
  <div class="card card--short">Short</div>
  <!-- more items... -->
</div>
```

The featured card spans a 2×2 area. Dense placement fills the remaining cells with whatever fits.

**Think about it**: `grid-auto-flow: dense` may reorder items visually from their DOM order. Why might this be a problem for screen readers and keyboard navigation? What CSS property can you set to ensure visual order matches DOM order?

## Exercises

1. **Two-Column Article Layout**: Build a layout with a 700px `max-width` main content area and a fixed 260px sidebar. The sidebar should be on the right. On screens below 768px, the sidebar should stack below the content. Use `grid-template-areas`.

2. **Holy Grail Layout**: The classic web layout: header, footer that stick to the top and bottom, a wide content area in the middle, and two sidebars flanking it. All five regions should be named grid areas. The two sidebars should be 200px each. The layout should be responsive: stack to a single column on mobile.

3. **Photo Gallery with Auto-Fill**: Create a responsive photo gallery where each photo is a `div` with a `padding-top: 100%` (aspect ratio trick) to make square cells. Use `auto-fill` with `minmax(150px, 1fr)`. Add a "featured" class that makes a photo span 2×2. The gallery should work at any viewport width without media queries.

4. **CSS Grid Subgrid**: Research the `subgrid` keyword (baseline-align content across nested grids). Create a card layout where each card has a title, body, and footer. Without subgrid, the card footers don't align across cards. With `grid-template-rows: subgrid`, they do. Implement both versions and show the difference.

5. **Grid-Based Form Layout**: Create a form where labels and inputs are aligned using a two-column grid. The label column is `auto` width (fits the text), the input column is `1fr`. Some fields (like a textarea or a full-width error message) should span both columns with `grid-column: 1 / -1`. The form should be responsive.

## Solutions

### Exercise 1: Two-Column Article Layout

```html
<!DOCTYPE html>
<html>
<head>
<style>
.article-layout {
  display: grid;
  grid-template-columns: 1fr 260px;
  grid-template-areas:
    "content sidebar";
  max-width: 1000px;
  margin: 0 auto;
  gap: 2rem;
  padding: 1rem;
}

.article-content { grid-area: content; }
.article-sidebar { grid-area: sidebar; }

@media (max-width: 768px) {
  .article-layout {
    grid-template-columns: 1fr;
    grid-template-areas:
      "content"
      "sidebar";
  }
}
</style>
</head>
<body>
<div class="article-layout">
  <main class="article-content">
    <h1>Main Content</h1>
    <p>Article body text goes here...</p>
  </main>
  <aside class="article-sidebar">
    <h2>Sidebar</h2>
    <p>Related links...</p>
  </aside>
</div>
</body>
</html>
```

### Exercise 2: Holy Grail Layout

```html
<!DOCTYPE html>
<html>
<head>
<style>
.holy-grail {
  display: grid;
  grid-template-columns: 200px 1fr 200px;
  grid-template-rows: auto 1fr auto;
  grid-template-areas:
    "header  header  header"
    "left    content right"
    "footer  footer  footer";
  min-height: 100vh;
  gap: 0;
}

.hg-header  { grid-area: header;  padding: 1rem; background: #1a1a3e; }
.hg-left    { grid-area: left;    padding: 1rem; background: #12122e; }
.hg-content { grid-area: content; padding: 1rem; }
.hg-right   { grid-area: right;   padding: 1rem; background: #12122e; }
.hg-footer  { grid-area: footer;  padding: 1rem; background: #1a1a3e; }

@media (max-width: 768px) {
  .holy-grail {
    grid-template-columns: 1fr;
    grid-template-areas:
      "header"
      "content"
      "left"
      "right"
      "footer";
  }
}
</style>
</head>
<body>
<div class="holy-grail">
  <header class="hg-header">Header</header>
  <nav    class="hg-left">Left Nav</nav>
  <main   class="hg-content">Main Content</main>
  <aside  class="hg-right">Right Sidebar</aside>
  <footer class="hg-footer">Footer</footer>
</div>
</body>
</html>
```

### Exercise 3: Photo Gallery with Auto-Fill

```html
<!DOCTYPE html>
<html>
<head>
<style>
.gallery {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(150px, 1fr));
  grid-auto-rows: 150px;
  gap: 0.5rem;
  padding: 1rem;
}

.photo {
  background: #1a1a3e;
  border-radius: 4px;
  overflow: hidden;
}

.photo--featured {
  grid-column: span 2;
  grid-row: span 2;
  background: #2a2a5e;
}
</style>
</head>
<body>
<div class="gallery">
  <div class="photo photo--featured"></div>
  <div class="photo"></div>
  <div class="photo"></div>
  <div class="photo"></div>
  <div class="photo"></div>
  <div class="photo"></div>
  <div class="photo photo--featured"></div>
  <div class="photo"></div>
  <div class="photo"></div>
</div>
</body>
</html>
```

### Exercise 4: Subgrid Card Alignment

```html
<!DOCTYPE html>
<html>
<head>
<style>
/* Without subgrid: footers don't align */
.cards-no-subgrid {
  display: grid;
  grid-template-columns: repeat(3, 1fr);
  gap: 1rem;
}

.card {
  display: flex;
  flex-direction: column;
  background: #1a1a3e;
  padding: 1rem;
  border-radius: 8px;
}

.card-body { flex: 1; }
.card-footer { margin-top: auto; padding-top: 1rem; border-top: 1px solid #333; }

/* With subgrid: footers align perfectly */
.cards-subgrid {
  display: grid;
  grid-template-columns: repeat(3, 1fr);
  gap: 1rem;
}

.cards-subgrid .card {
  display: grid;
  grid-template-rows: auto 1fr auto;
  grid-row: span 3;
}

.cards-subgrid {
  grid-template-rows: auto;
}
</style>
</head>
<body>
<div class="cards-no-subgrid">
  <div class="card">
    <h3>Short Title</h3>
    <p class="card-body">A short body.</p>
    <div class="card-footer">Footer</div>
  </div>
  <div class="card">
    <h3>A Much Longer Title That Wraps</h3>
    <p class="card-body">A longer body with more text inside it.</p>
    <div class="card-footer">Footer</div>
  </div>
  <div class="card">
    <h3>Title</h3>
    <p class="card-body">Medium body text here.</p>
    <div class="card-footer">Footer</div>
  </div>
</div>
</body>
</html>
```

### Exercise 5: Grid-Based Form Layout

```html
<!DOCTYPE html>
<html>
<head>
<style>
.form-grid {
  display: grid;
  grid-template-columns: auto 1fr;
  gap: 0.75rem 1rem;
  align-items: baseline;
  max-width: 500px;
}

.form-grid label {
  text-align: right;
  font-weight: 500;
}

.form-grid input,
.form-grid textarea,
.form-grid select {
  width: 100%;
  padding: 0.5rem;
  background: #1a1a2e;
  border: 1px solid #333;
  color: #c8c8d8;
  border-radius: 4px;
}

.form-grid .full-width {
  grid-column: 1 / -1;
}

.form-grid .form-actions {
  grid-column: 2;
}

@media (max-width: 500px) {
  .form-grid {
    grid-template-columns: 1fr;
  }
  .form-grid label {
    text-align: left;
  }
  .form-grid .form-actions {
    grid-column: 1;
  }
}
</style>
</head>
<body>
<form class="form-grid">
  <label for="name">Name</label>
  <input type="text" id="name" />

  <label for="email">Email</label>
  <input type="email" id="email" />

  <label for="message">Message</label>
  <textarea id="message" rows="4"></textarea>

  <div class="full-width error">Please fill in all fields.</div>

  <div class="form-actions">
    <button type="submit">Submit</button>
  </div>
</form>
</body>
</html>
```

## What You Learned

| Concept | Key point |
|---|---|
| `display: grid` | Turns a container into a grid formatting context; direct children become grid items |
| `grid-template-columns` | Defines column track sizes; `repeat(12, 1fr)` creates a 12-column grid |
| `fr` unit | Distributes *free* space after fixed sizes; `minmax(0, 1fr)` to allow shrinking below content size |
| `gap` | Sets gutters between tracks; shorthand for `row-gap` and `column-gap` |
| `grid-column: span N` | Makes an item span N column tracks; `1 / -1` spans the full width |
| `grid-template-areas` | ASCII-art layout map; areas must be rectangular; `.` for empty cells |
| `grid-area` | Assigns an item to a named area; also shorthand for row/column start/end |
| `auto-fill` | Creates as many tracks as fit; empty tracks remain and take space |
| `auto-fit` | Creates as many tracks as fit; empty tracks collapse to 0 |
| `minmax(min, max)` | Sets a track's minimum and maximum size; core of responsive grids |
| Explicit grid | Tracks defined by `grid-template-columns/rows` |
| Implicit grid | Tracks auto-created when items overflow; sized by `grid-auto-rows/columns` |
| `grid-auto-flow: dense` | Fills gaps with later items; may reorder visual/DOM order |
| Flexbox vs. Grid | Flexbox for 1D component layout; Grid for 2D page/section layout |

## Building with Claude

**Bad prompt:**
> "Make my layout responsive with CSS grid"

This gives Claude no information about what the layout looks like, what "responsive" means to you, or what constraints exist. You'll get a generic example that may not apply.

**Good prompt:**
> "I have a CSS grid layout with `grid-template-columns: 250px 1fr` and `grid-template-areas: 'sidebar content'`. On screens below 768px, I want the sidebar to stack below the content, and I want to use `grid-template-areas` for the responsive state too. The sidebar should appear after the content in the mobile stacking order even though it comes first in the HTML. Write just the responsive CSS — I already have the base styles."

This prompt specifies the existing CSS, the exact breakpoint, the desired behavior, the stacking order requirement (which is non-obvious), and the scope of the answer. Claude will give you exactly the media query you need.

**Connection to the JS book**: The two-column layout in Chapter 13's Debugger (code panel on the left, output panel on the right) is a named grid template area. The card grid in Chapter 12's virtual DOM demo uses `auto-fill` for responsive columns that adjust without JavaScript.
