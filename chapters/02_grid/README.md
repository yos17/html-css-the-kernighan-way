# Chapter 2 — Build Your Own Grid System

Bootstrap's grid is downloaded millions of times a day. Developers wrap divs in `col-md-6` without thinking about what that class does. In this chapter you build a 12-column responsive grid from scratch — the same system Bootstrap uses, minus 10,000 lines of history. The grid is about 40 lines of CSS. Writing it will teach you CSS Grid, the `fr` unit, named template areas, and how "responsive columns" actually work.

---

## The Problem

A layout engine needs to solve two problems: how to divide horizontal space, and how to stack things when the screen gets narrow. The naive approach uses `float` (Bootstrap 3 and earlier) or inline-block, which requires clearfix hacks and is full of rounding bugs. CSS Grid solves both problems with native browser support:

```
The old float approach (Bootstrap 3):
──────────────────────────────────────────────────────────
.col-6 { float: left; width: 50%; }
.clearfix::after { content: ''; display: table; clear: both; }

Problems:
  • floats collapse their parent — need clearfix hack
  • columns must be the same height or layout breaks
  • responsive requires percentage math (41.666666%)
  • equal-height columns require negative margin hacks

The CSS Grid approach:
──────────────────────────────────────────────────────────
.row { display: grid; grid-template-columns: repeat(12, 1fr); }
.col-6 { grid-column: span 6; }

Result:
  • no floats, no clearfix
  • columns are equal height automatically
  • gap property replaces margin math
  • repeat() and fr handle all the arithmetic
```

---

## Building It Step by Step

### v1 — A Fixed Two-Column Layout

Start with the simplest case: two equal columns.

```css
/* v1: hard-coded two columns */
.row {
  display: grid;
  grid-template-columns: 1fr 1fr;  /* two equal columns */
  gap: 1rem;
}
```

`1fr` means "one fraction of available space." With two columns, each gets half. With three `1fr` columns, each gets a third. The `fr` unit does the math you used to do with percentages.

### v2 — The 12-Column System

A 12-column grid is flexible because 12 divides evenly by 1, 2, 3, 4, 6, and 12 — every common layout is expressible:

```css
/* v2: 12-column grid with span classes */
.row {
  display: grid;
  grid-template-columns: repeat(12, 1fr);
  gap: 1.5rem;
}

/* span classes — how many columns this element takes */
.col-1  { grid-column: span 1;  }
.col-2  { grid-column: span 2;  }
.col-3  { grid-column: span 3;  }
.col-4  { grid-column: span 4;  }
.col-6  { grid-column: span 6;  }
.col-8  { grid-column: span 8;  }
.col-9  { grid-column: span 9;  }
.col-12 { grid-column: span 12; }
```

`repeat(12, 1fr)` is shorthand for writing `1fr` twelve times. `grid-column: span 6` means "this element spans 6 of those columns." Two `span 6` elements fill a row exactly.

### v3 — Responsive Breakpoints

The real power: redefine column spans at different screen widths.

```css
/* v3: responsive with breakpoints */

/* Mobile first: full width by default */
[class^="col-"] { grid-column: span 12; }

/* Tablet (≥ 768px) */
@media (min-width: 768px) {
  .col-md-1  { grid-column: span 1;  }
  .col-md-2  { grid-column: span 2;  }
  .col-md-3  { grid-column: span 3;  }
  .col-md-4  { grid-column: span 4;  }
  .col-md-6  { grid-column: span 6;  }
  .col-md-8  { grid-column: span 8;  }
  .col-md-12 { grid-column: span 12; }
}

/* Desktop (≥ 1024px) */
@media (min-width: 1024px) {
  .col-lg-1  { grid-column: span 1;  }
  .col-lg-2  { grid-column: span 2;  }
  .col-lg-3  { grid-column: span 3;  }
  .col-lg-4  { grid-column: span 4;  }
  .col-lg-6  { grid-column: span 6;  }
  .col-lg-8  { grid-column: span 8;  }
  .col-lg-12 { grid-column: span 12; }
}
```

Now `<div class="col-md-6 col-lg-4">` is full-width on mobile, half-width on tablet, one-third on desktop.

---

## The Complete Program

`grid.html` — open it in your browser. Resize the window to see responsive columns snap at each breakpoint. Toggle the grid overlay to see column lines.

---

## Walkthrough

### The `fr` Unit

`fr` stands for "fraction of remaining space." It's calculated after fixed sizes and gaps are removed:

```
Available width:   900px
Gap between cols:  1rem × 11 = 176px  (11 gaps between 12 columns)
Remaining:         900 - 176 = 724px
Each 1fr:          724 / 12 = ~60.3px

grid-column: span 6  →  6 × 60.3 + 5 × 16 = ~442px
```

You never need to know the actual pixel values — that's the point. The browser calculates everything.

### How `span` Works

`grid-column: span N` is shorthand for `grid-column: auto / span N`. It means "start wherever auto places me, and span N tracks":

```
12-column grid:
 1   2   3   4   5   6   7   8   9  10  11  12
 ┌───┬───┬───┬───┬───┬───┬───┬───┬───┬───┬───┬───┐
 │     col-4 (span 4)    │       col-8 (span 8)       │
 └───┴───┴───┴───┴───┴───┴───┴───┴───┴───┴───┴───┘

 ┌───┬───┬───┬───┬───┬───┬───┬───┬───┬───┬───┬───┐
 │  col-3  │  col-3  │  col-3  │  col-3  │
 └───┴───┴───┴───┴───┴───┴───┴───┴───┴───┴───┴───┘

 ┌───┬───┬───┬───┬───┬───┬───┬───┬───┬───┬───┬───┐
 │              col-12 (full width)                   │
 └───┴───┴───┴───┴───┴───┴───┴───┴───┴───┴───┴───┘
```

Items that don't fit on the current row automatically wrap to the next.

### Named Template Areas

For fixed layouts (header/sidebar/main/footer), named areas are clearer than span numbers:

```css
.layout {
  display: grid;
  grid-template-columns: 250px 1fr;
  grid-template-rows: auto 1fr auto;
  grid-template-areas:
    "header  header"
    "sidebar main  "
    "footer  footer";
  min-height: 100vh;
}

.header  { grid-area: header;  }
.sidebar { grid-area: sidebar; }
.main    { grid-area: main;    }
.footer  { grid-area: footer;  }
```

The visual layout in `grid-template-areas` mirrors what you see in the browser — the name appears in the cell it occupies. Dots (`.`) mark empty cells.

### `auto-fit` and `auto-fill`

For card grids where you don't care about exact column counts, `auto-fit` creates as many columns as fit:

```css
/* As many 200px-minimum columns as fit — no media queries needed */
.card-grid {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(200px, 1fr));
  gap: 1rem;
}
```

`minmax(200px, 1fr)` means "each column is at least 200px wide, and expands to fill available space." When the viewport narrows below 400px, the two columns automatically collapse to one. No `@media` needed.

### The Offset Pattern

To skip columns (whitespace before an element), use `grid-column-start`:

```css
/* Start at column 3, span 8 — "centered" 8-column block */
.centered-content {
  grid-column: 3 / span 8;
}

/* Or use the offset class pattern */
.offset-2 { grid-column-start: 3; }  /* start at column 3 = skip 2 */
.offset-4 { grid-column-start: 5; }
```

---

## Guided Try It — Build a Masonry-Style Layout

**The goal**: a "Pinterest-style" layout where items have different heights but pack tightly, with no gaps.

**The reality**: CSS Grid doesn't do true masonry (that's complex JS). But it can do a close approximation with `grid-row: span N`:

**Step 1 — Set up a grid with many small rows**

```css
.masonry {
  display: grid;
  grid-template-columns: repeat(3, 1fr);
  grid-template-rows: repeat(20, 40px);   /* many small rows */
  gap: 1rem;
  align-items: start;
}
```

**Step 2 — Assign different row spans to items**

```css
.tall   { grid-row: span 4; }   /* 4 × 40px = 160px */
.medium { grid-row: span 3; }   /* 3 × 40px = 120px */
.short  { grid-row: span 2; }   /* 2 × 40px =  80px */
```

**Step 3 — Use `grid-auto-rows` instead of fixed row count**

```css
.masonry {
  display: grid;
  grid-template-columns: repeat(3, 1fr);
  grid-auto-rows: 40px;   /* generates rows on demand */
  gap: 0.5rem;
}
```

**Think about it**: CSS Grid places items in document order, left-to-right then top-to-bottom. If a tall item is followed by a short item, the short item can't "float up" to fill the gap — that space stays empty. The real masonry requires JavaScript to manually position items. When would you accept the gap vs. use JS?

---

## Exercises

1. **Implement `col-offset-*` classes**: Add `.col-offset-1` through `.col-offset-6` that push an element right by N columns using `grid-column-start`. Test: `<div class="col-6 col-offset-3">` should appear centered.

2. **Build a holy grail layout**: Using `grid-template-areas`, create a page with header (full width), left sidebar (200px), main content (1fr), right sidebar (200px), and footer (full width). Make it stack to a single column on mobile.

3. **`auto-fit` vs `auto-fill`**: Create two card grids side by side — one with `auto-fit` and one with `auto-fill`. Fill the first grid with 3 cards and the second with 3 cards. Explain in a comment what happens to the empty columns when there aren't enough cards.

4. **Equal-height cards**: In a 3-column grid, put cards with different text lengths. Verify they're equal height (they should be — that's Grid's default). Now try the same with `float: left; width: 33%` and document what breaks.

5. **Subgrid**: Use `grid-template-columns: subgrid` to align nested card content (icon, title, body, action button) across all cards in a row, even when the cards have different content lengths. This requires each card's children to share the parent grid's column tracks.

---

## Solutions

### Exercise 1 — `col-offset-*`

```css
/* Offset classes: push element right by N columns */
.col-offset-1 { grid-column-start: 2;  }
.col-offset-2 { grid-column-start: 3;  }
.col-offset-3 { grid-column-start: 4;  }
.col-offset-4 { grid-column-start: 5;  }
.col-offset-6 { grid-column-start: 7;  }

/* Usage: */
/* <div class="row">
     <div class="col-6 col-offset-3">Centered content</div>
   </div> */

/* col-offset-3 starts at column 4, col-6 spans to column 9.
   Columns 1-3 and 10-12 are empty = centered appearance */
```

### Exercise 2 — Holy grail layout

```css
.holy-grail {
  display: grid;
  grid-template-columns: 200px 1fr 200px;
  grid-template-rows: auto 1fr auto;
  grid-template-areas:
    "header  header  header "
    "sidebar main    aside  "
    "footer  footer  footer ";
  min-height: 100vh;
}

.header  { grid-area: header;  }
.sidebar { grid-area: sidebar; }
.main    { grid-area: main;    }
.aside   { grid-area: aside;   }
.footer  { grid-area: footer;  }

/* Mobile: single column */
@media (max-width: 767px) {
  .holy-grail {
    grid-template-columns: 1fr;
    grid-template-areas:
      "header "
      "main   "
      "sidebar"
      "aside  "
      "footer ";
  }
}
```

### Exercise 3 — `auto-fit` vs `auto-fill`

```css
/* auto-fill: creates as many columns as fit, even if empty */
.grid-fill {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(150px, 1fr));
  gap: 1rem;
}

/* auto-fit: collapses empty columns, stretches filled ones */
.grid-fit {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(150px, 1fr));
  gap: 1rem;
}

/*
  With 3 cards in a 600px container:
  auto-fill: 3 cards + 1 empty column (4 total defined)
             cards don't stretch to full width
  auto-fit:  3 cards, empty columns collapse
             cards stretch to fill all available width

  Rule of thumb: use auto-fit when you want cards to fill the row.
  Use auto-fill when you want consistent column sizing regardless of count.
*/
```

### Exercise 4 — Equal-height cards

```html
<!-- CSS Grid: equal height automatically -->
<div class="row">
  <div class="col-4" style="background:#e0f2fe;padding:1rem">Short text</div>
  <div class="col-4" style="background:#e0f2fe;padding:1rem">
    Much longer text that wraps to multiple lines and makes this card taller
  </div>
  <div class="col-4" style="background:#e0f2fe;padding:1rem">Medium text here</div>
</div>
<!-- All three are equal height: Grid's default align-items is stretch -->

<!-- Float approach: NOT equal height -->
<div style="overflow:hidden">
  <div style="float:left;width:33%;background:#fce7f3;padding:1rem">Short</div>
  <div style="float:left;width:33%;background:#fce7f3;padding:1rem">
    Much longer text that makes this taller — the other floats don't match
  </div>
  <div style="float:left;width:33%;background:#fce7f3;padding:1rem">Medium</div>
</div>
<!-- Heights differ: floats are independent boxes -->
```

### Exercise 5 — Subgrid

```css
/* Parent grid */
.card-row {
  display: grid;
  grid-template-columns: repeat(3, 1fr);
  gap: 1rem;
  align-items: start;
}

/* Each card participates in the parent grid AND defines its own rows */
.card {
  display: grid;
  grid-row: span 1;
  /* Subgrid: use the parent's column tracks for this card's children */
  grid-template-rows: auto 1fr auto auto;  /* icon, title, body, button */
}

/* Better: define card rows explicitly and use subgrid for columns */
.card-row {
  display: grid;
  grid-template-columns: repeat(3, 1fr);
  grid-template-rows: repeat(3, auto);  /* 3 cards × 4 rows each */
  gap: 1rem;
}

.card {
  display: grid;
  grid-template-rows: subgrid;    /* inherit parent row sizing */
  grid-row: span 4;               /* each card takes 4 rows */
}
/* Now all icons align, all titles align, all body text aligns,
   all buttons align — regardless of content length */
```

---

## What You Learned

| Concept | Key point |
|---------|-----------|
| `display: grid` | Creates a grid formatting context; children become grid items |
| `fr` unit | Fraction of available space after gaps and fixed sizes are removed |
| `repeat(n, track)` | Shorthand for repeating a track definition N times |
| `grid-column: span N` | Item spans N column tracks from its auto-placed start position |
| `gap` | Space between rows and columns; replaces margin hacks |
| `grid-template-areas` | Visual ASCII-art layout declaration; each name = a grid-area |
| `auto-fit` vs `auto-fill` | Both fill columns; `auto-fit` collapses empty tracks |
| `minmax(min, max)` | Column is at least min, at most max (commonly `minmax(200px, 1fr)`) |
| `grid-column-start` | Explicitly place an item at a column line |
| `subgrid` | Nested grid inherits parent's track sizes for alignment across cards |
| Equal height | Grid items in the same row are equal height by default (`align-items: stretch`) |

---

## Building with Claude

Bad prompt:
> "How do I make a responsive grid in CSS?"

Good prompt:
> "I'm building a 12-column CSS Grid system from scratch (no Bootstrap). I have `grid-template-columns: repeat(12, 1fr)` on `.row` and `.col-6 { grid-column: span 6 }` classes. For responsive behavior, I'm using `@media (min-width: 768px) { .col-md-6 { grid-column: span 6 } }`. The problem: on mobile, before any breakpoint applies, items with only `col-md-6` (no base `col-*` class) don't stack to full width — they stay at their natural content width instead of spanning all 12 columns. Should I add `grid-column: span 12` to `[class*='col-']` as a base rule, or handle it differently? What are the trade-offs?"
