# Chapter 2 — Build Your Own Typography System

Typography is not decoration. It is the primary tool through which a user interface communicates. The spacing between lines determines whether a paragraph feels cramped or comfortable. The ratio between heading sizes determines whether a hierarchy is clear or muddled. The choice between `em` and `rem` determines whether a component's text scales with its container or with the page. Getting these decisions right separates UIs that look professional from ones that look assembled from parts.

Most developers treat typography as an afterthought — set a font, pick some sizes, move on. The result is a page where the headings are too close in size to matter, the line height is tight enough to make long paragraphs painful to read, and the form inputs are in a different font from everything else. Building a typography system forces you to make these decisions deliberately, with a model for why each value is what it is.

This chapter builds a complete typography system from scratch: font stack selection, a modular type scale, responsive fluid sizing, and vertical rhythm. The monospace font choice for code output throughout the JavaScript book is a deliberate typographic decision. The type scale in Chapter 7's Test Framework output and Chapter 13's Debugger UI uses exactly this system. Understanding what you are building here explains every text sizing decision in those later chapters.

## The Problem

The default web typography is bad. The browser renders everything in a serif font (usually Times New Roman) at 16px with a line height around 1.2. A line height of 1.2 means consecutive lines of text are separated by only 20% of the font size — which feels cramped for body text. A single font size for all text means there is no visual hierarchy. No hierarchy means the reader cannot scan the page to find what they need.

The naive fix is to pick a font from Google Fonts, set `font-size: 16px` on the body, and manually size each heading with a number that looks right. The problem is that "looks right" is arbitrary. If you increase the base font size later, all your heading sizes are wrong. If you want to add a new heading level between `h2` and `h3`, there is no principled value to choose — you just guess.

The correct approach is a **type scale** — a mathematical ratio applied to a base size. Every size in the system is the base multiplied by the ratio raised to some power. Change the base or the ratio and the entire system rescales consistently. Add a new size and there is a principled value for it.

## Building It Step by Step

### v1 — Font Stack and Base Size

Before a type scale, you need a foundation: the font itself and the base size it scales from.

```css
/* v1: type-system.css */

/* The 62.5% trick: makes 1rem = 10px at default browser settings */
html {
  font-size: 62.5%;
}

body {
  font-size: 1.6rem;    /* 16px equivalent */
  font-family:
    'Inter',
    -apple-system,
    BlinkMacSystemFont,
    'Segoe UI',
    Roboto,
    'Helvetica Neue',
    Arial,
    sans-serif;
  line-height: 1.6;
  color: #1a1a2e;
}

/* Monospace stack — for code, terminals, data */
code, pre, kbd, samp {
  font-family:
    'JetBrains Mono',
    'Fira Code',
    'Cascadia Code',
    'Courier New',
    Courier,
    monospace;
  font-size: 0.9em;  /* monospace fonts render slightly larger at equal px */
}
```

The 62.5% trick on `html` deserves an explanation. Browser default font size is 16px. `62.5%` of 16 is 10. Setting `html { font-size: 62.5% }` makes `1rem` equal to `10px`. This means you can write `1.6rem` and immediately know you mean 16px, `2.4rem` means 24px, `0.8rem` means 8px — the arithmetic is trivial. You keep the user's default font size as the reference point (a user who bumped their browser to 20px will get `1rem = 12.5px`), but rem values map cleanly to pixel values you recognize.

Font stacks have a specific logic. The first font is the one you want. Everything after is a fallback if that font is not available. `-apple-system` and `BlinkMacSystemFont` are the system UI font on macOS and iOS. `Segoe UI` is the Windows system font. `Roboto` is Android. `Helvetica Neue` and `Arial` are old but universal. `sans-serif` is the browser's configured sans-serif, whatever that is. This stack gives you native UI fonts on every platform with no network request required.

### v2 — The Modular Type Scale

A type scale applies a constant ratio between each step. A ratio of 1.25 (the "Major Third") is clean and clear without being dramatic:

```css
/* v2: add the type scale */

/* Base = 1.6rem (16px). Ratio = 1.25 (Major Third) */
/* Formula: base × ratio^n */

/* n = -1: 1.6 × 1.25^-1 = 1.28rem ≈ 12.8px */
.text-xs,
small,
figcaption {
  font-size: 1.28rem;
  line-height: 1.5;
}

/* n = 0: 1.6rem = 16px — body text */
body,
p,
.text-base {
  font-size: 1.6rem;
  line-height: 1.6;
}

/* n = 1: 1.6 × 1.25 = 2.0rem = 20px */
h6,
.text-lg {
  font-size: 2.0rem;
  font-weight: 600;
  line-height: 1.4;
}

/* n = 2: 1.6 × 1.25^2 = 2.5rem = 25px */
h5,
.text-xl {
  font-size: 2.5rem;
  font-weight: 600;
  line-height: 1.35;
}

/* n = 3: 1.6 × 1.25^3 = 3.125rem = 31.25px */
h4,
.text-2xl {
  font-size: 3.125rem;
  font-weight: 600;
  line-height: 1.3;
}

/* n = 4: 1.6 × 1.25^4 = 3.906rem ≈ 39px */
h3,
.text-3xl {
  font-size: 3.906rem;
  font-weight: 700;
  line-height: 1.25;
}

/* n = 5: 1.6 × 1.25^5 = 4.883rem ≈ 49px */
h2,
.text-4xl {
  font-size: 4.883rem;
  font-weight: 700;
  line-height: 1.2;
}

/* n = 6: 1.6 × 1.25^6 = 6.104rem ≈ 61px */
h1,
.text-5xl {
  font-size: 6.104rem;
  font-weight: 700;
  line-height: 1.15;
}
```

Notice two things about line height. First, it decreases as font size increases. Large text needs less relative line height because it already has visual separation. A 61px heading at `line-height: 1.6` would have 37px of space between lines — far too much. Second, the relationship is intentional: at body size (1.6) a line height of 1.6 is correct for long-form reading. At display sizes (6rem+) a line height of 1.15 is correct for headers.

### v3 — Fluid Type and Vertical Rhythm

The final version uses `clamp()` to make type sizes responsive without media queries, and adds spacing utilities based on a vertical rhythm.

```css
/* v3: type-system.css — complete version */

html { font-size: 62.5%; }

body {
  font-family:
    'Inter', -apple-system, BlinkMacSystemFont,
    'Segoe UI', Roboto, 'Helvetica Neue', Arial, sans-serif;
  font-size: 1.6rem;
  line-height: 1.6;
  color: #1a1a2e;
  /* Max line length for readability: ~70 characters */
  /* Applied to prose containers, not global */
}

/* Prose container: constrain line length */
.prose {
  max-width: 68ch;  /* ch = width of "0" glyph; 68ch ≈ 70 characters */
}

/* Fluid type with clamp(min, preferred, max) */
/* preferred uses viewport width: 1vw ≈ 1% of viewport */

h1 {
  /* Minimum 3.2rem (32px), grows with viewport, max 6.1rem (61px) */
  font-size: clamp(3.2rem, 5vw + 1rem, 6.104rem);
  font-weight: 700;
  line-height: 1.15;
  letter-spacing: -0.02em;  /* tight tracking on large type */
}

h2 {
  font-size: clamp(2.8rem, 4vw + 0.8rem, 4.883rem);
  font-weight: 700;
  line-height: 1.2;
  letter-spacing: -0.01em;
}

h3 {
  font-size: clamp(2.2rem, 3vw + 0.5rem, 3.906rem);
  font-weight: 600;
  line-height: 1.25;
}

h4 {
  font-size: clamp(1.8rem, 2vw + 0.3rem, 3.125rem);
  font-weight: 600;
  line-height: 1.3;
}

h5 {
  font-size: clamp(1.6rem, 1.5vw + 0.2rem, 2.5rem);
  font-weight: 600;
  line-height: 1.35;
}

h6 {
  font-size: clamp(1.4rem, 1vw + 0.2rem, 2.0rem);
  font-weight: 600;
  line-height: 1.4;
}

body, p { font-size: 1.6rem; line-height: 1.6; }
small    { font-size: 1.28rem; line-height: 1.5; }

/* Vertical rhythm — spacing in multiples of base line height */
/* base line height: 1.6rem × 1.6 = 2.56rem */

:root {
  --rhythm: 2.56rem;  /* base line-height × body font-size */
}

h1, h2, h3, h4, h5, h6 {
  margin-top: calc(var(--rhythm) * 2);
  margin-bottom: calc(var(--rhythm) * 0.5);
}

p + p {
  margin-top: calc(var(--rhythm) * 0.5);
}

/* Code */
code, pre, kbd, samp {
  font-family: 'JetBrains Mono', 'Fira Code', 'Cascadia Code',
               'Courier New', Courier, monospace;
}

code { font-size: 0.9em; }

pre {
  font-size: 1.4rem;
  line-height: 1.5;
  overflow-x: auto;
}
```

The `clamp(min, preferred, max)` function is the key to fluid type. The preferred value is a viewport-relative expression. At a narrow viewport the minimum kicks in; at a wide viewport the maximum caps it. Between those extremes the size grows smoothly with the viewport width, with no media query breakpoints needed.

The `68ch` max-width on `.prose` containers is based on typography research: the optimal line length for reading is 60–80 characters. The `ch` unit is the width of the `0` glyph in the current font, which makes `68ch` a font-size-relative measure that scales correctly when the user changes their font size.

## The Complete Program

The complete type system is in `typography.html`. The CSS listing:

```css
/* type-system.css — HTML & CSS: The Kernighan Way */

html { font-size: 62.5%; }

body {
  font-family:
    'Inter', -apple-system, BlinkMacSystemFont,
    'Segoe UI', Roboto, 'Helvetica Neue', Arial, sans-serif;
  font-size: 1.6rem;
  line-height: 1.6;
}

code, pre, kbd, samp {
  font-family: 'JetBrains Mono', 'Fira Code', 'Cascadia Code',
               'Courier New', Courier, monospace;
}

code { font-size: 0.9em; }

.prose { max-width: 68ch; }

h1 { font-size: clamp(3.2rem, 5vw + 1rem, 6.104rem); font-weight: 700; line-height: 1.15; letter-spacing: -0.02em; }
h2 { font-size: clamp(2.8rem, 4vw + 0.8rem, 4.883rem); font-weight: 700; line-height: 1.2; letter-spacing: -0.01em; }
h3 { font-size: clamp(2.2rem, 3vw + 0.5rem, 3.906rem); font-weight: 600; line-height: 1.25; }
h4 { font-size: clamp(1.8rem, 2vw + 0.3rem, 3.125rem); font-weight: 600; line-height: 1.3; }
h5 { font-size: clamp(1.6rem, 1.5vw + 0.2rem, 2.5rem); font-weight: 600; line-height: 1.35; }
h6 { font-size: clamp(1.4rem, 1vw + 0.2rem, 2.0rem); font-weight: 600; line-height: 1.4; }

small  { font-size: 1.28rem; line-height: 1.5; }

:root { --rhythm: 2.56rem; }

h1, h2, h3, h4, h5, h6 {
  margin-top: calc(var(--rhythm) * 2);
  margin-bottom: calc(var(--rhythm) * 0.5);
}

p + p { margin-top: calc(var(--rhythm) * 0.5); }
```

## Walkthrough

### em vs rem — The Parent Dependency

`em` and `rem` are both relative units, but they are relative to different things.

```
em: relative to the current element's own font-size
    (or, for font-size itself, the parent's font-size)

rem: relative to the root element's (html) font-size

┌─── html: font-size 62.5% (= 10px at default) ──────────────────┐
│                                                                 │
│  ┌─── body: font-size 1.6rem ─────────────────────────────┐    │
│  │  1.6rem = 1.6 × 10px = 16px                            │    │
│  │                                                         │    │
│  │  ┌─── article: font-size 1.2em ───────────────────┐    │    │
│  │  │  1.2em = 1.2 × 16px = 19.2px                   │    │    │
│  │  │                                                 │    │    │
│  │  │  ┌─── p: font-size 0.8em ──────────────────┐   │    │    │
│  │  │  │  0.8em = 0.8 × 19.2px = 15.36px         │   │    │    │
│  │  │  │  (em compounds: parent was already 1.2×) │   │    │    │
│  │  │  └─────────────────────────────────────────┘   │    │    │
│  │  │                                                 │    │    │
│  │  │  ┌─── p: font-size 0.8rem ─────────────────┐   │    │    │
│  │  │  │  0.8rem = 0.8 × 10px = 8px              │   │    │    │
│  │  │  │  (rem ignores the 1.2× parent)           │   │    │    │
│  │  │  └─────────────────────────────────────────┘   │    │    │
│  │  └─────────────────────────────────────────────────┘    │    │
│  └─────────────────────────────────────────────────────────┘    │
└─────────────────────────────────────────────────────────────────┘
```

Use `rem` for font sizes that should be consistent across the page regardless of nesting. Use `em` for values that should scale with the local font size — padding, border-radius, icons that live inline with text.

### Type Scale Ratios

The mathematical relationship between scale steps:

```
Base: 1.6rem (16px)   Ratio: 1.25 (Major Third)

Step   Multiplier      rem         px (approx)   Usage
────   ──────────      ────        ───────────   ──────────────
 -1    ÷ 1.25          1.280rem    12.8px        caption, small
  0    ×  1 (base)     1.600rem    16.0px        body text
  1    × 1.25          2.000rem    20.0px        h6, lead text
  2    × 1.25²         2.500rem    25.0px        h5
  3    × 1.25³         3.125rem    31.3px        h4
  4    × 1.25⁴         3.906rem    39.1px        h3
  5    × 1.25⁵         4.883rem    48.8px        h2
  6    × 1.25⁶         6.104rem    61.0px        h1

Other common ratios:
  1.067 — Minor Second: barely noticeable difference
  1.125 — Major Second: subtle
  1.200 — Minor Third: gentle
  1.250 — Major Third: clear (this chapter)
  1.333 — Perfect Fourth: strong hierarchy
  1.414 — Augmented Fourth: dramatic
  1.500 — Perfect Fifth: very dramatic
```

A 1.25 ratio is the sweet spot for most UI work. It creates visible hierarchy without the h1 dwarfing everything else on the page.

### The `clamp()` Function

`clamp(minimum, preferred, maximum)` returns the preferred value, clamped between minimum and maximum.

```
clamp(3.2rem, 5vw + 1rem, 6.104rem)

Viewport width: 320px
  5vw = 16px = 1.6rem
  preferred = 1.6rem + 1rem = 2.6rem
  clamp returns: 3.2rem (minimum kicks in)

Viewport width: 768px
  5vw = 38.4px = 3.84rem
  preferred = 3.84rem + 1rem = 4.84rem
  clamp returns: 4.84rem (in range)

Viewport width: 1400px
  5vw = 70px = 7rem
  preferred = 7rem + 1rem = 8rem
  clamp returns: 6.104rem (maximum kicks in)

┌────────────────────────────────────────────────────────────────┐
│                                                                │
│  6.1rem ────────────────────────────────────────■■■■■■■■■■■   │
│                                           ╱                    │
│                                    ╱                           │
│  3.2rem ■■■■■■■■■■──────────╱                                  │
│         │         │         │         │         │              │
│        320px     480px     640px     960px    1280px           │
│        (min)                                  (max)            │
└────────────────────────────────────────────────────────────────┘
```

The viewport-relative preferred value (`5vw + 1rem`) creates a slope. The `+ 1rem` term is an offset that controls the slope's starting point. Without it, the fluid range would start at zero.

### Vertical Rhythm

Vertical rhythm is the consistent spacing of text baselines. When rhythm is maintained, the page feels organized and the eye flows naturally. When it breaks, the page feels cluttered.

```
With rhythm (base unit = 2.56rem):

┌───────────────────────────────────────────┐  ← margin: 2×rhythm
│  Heading                                  │
│                                           │  ← margin: 0.5×rhythm
├ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ┤  baseline
│  First paragraph of text that spans      │
│  multiple lines in a comfortable way     │
│                                           │
├ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ┤  baseline
│  Second paragraph, consistently spaced  │
│  from the first                          │
└───────────────────────────────────────────┘

Without rhythm (arbitrary spacing):

┌───────────────────────────────────────────┐
│  Heading                                  │
├──────────────────────────────────────────┤  ← 15px? 20px?
│  First paragraph                          │

│
│                                           │  ← 8px? who knows
│  Second paragraph — baseline chaos       │
└───────────────────────────────────────────┘
```

Perfect vertical rhythm (every text baseline on a grid) is essentially impossible in web typography because different font sizes at different line heights will rarely align. The pragmatic goal is consistent spacing ratios, not pixel-perfect baselines. The `--rhythm` custom property gives you a single reference point for all vertical spacing.

### Font Loading Strategies

```
System fonts (this chapter's stack):
  Pro: zero network request, instant, matches OS conventions
  Con: different appearance on Windows vs macOS vs iOS vs Android
  Use for: application UIs, dashboards, developer tools

Web fonts via @font-face:
  Pro: consistent appearance everywhere
  Con: network request; FOUT/FOIT (flash of unstyled/invisible text)
  Use for: marketing sites, brand-critical designs

font-display values:
  auto    — browser decides (usually same as block)
  block   — invisible until loaded, then swap (FOIT)
  swap    — show fallback immediately, swap when loaded (FOUT)
  fallback— short block, then fallback, swap if fast enough
  optional— short block, then fallback, no swap (best UX)

Recommendation for body text: font-display: optional
  Users who load fast get your font.
  Users on slow connections never see a flash.
  The page never reflows to accommodate a late font.
```

## Guided Try It — Add a Prose Component

A typography system without a prose context class is incomplete. This step builds `.prose` — a container that applies all the typographic details appropriate for long-form text.

**Step 1**: Add the base prose container:

```css
.prose {
  max-width: 68ch;
  color: inherit;
}
```

**Step 2**: Add heading and paragraph spacing within prose context:

```css
.prose h1,
.prose h2,
.prose h3,
.prose h4,
.prose h5,
.prose h6 {
  margin-top: calc(var(--rhythm) * 2);
  margin-bottom: calc(var(--rhythm) * 0.5);
}

.prose h1:first-child,
.prose h2:first-child {
  margin-top: 0;
}

.prose p {
  margin-bottom: var(--rhythm);
}

.prose p + p {
  margin-top: 0;
}
```

**Step 3**: Restore the typographic details that the reset stripped away — list bullets, link underlines — but only inside prose:

```css
.prose ul,
.prose ol {
  padding-left: 1.5em;
  margin-bottom: var(--rhythm);
}

.prose ul { list-style: disc; }
.prose ol { list-style: decimal; }

.prose li + li {
  margin-top: 0.4em;
}

.prose a {
  color: #4a90d9;
  text-decoration: underline;
  text-underline-offset: 0.15em;
  text-decoration-thickness: 1px;
}

.prose a:hover {
  text-decoration-thickness: 2px;
}

.prose strong { font-weight: 700; }
.prose em     { font-style: italic; }

.prose code {
  font-size: 0.875em;
  background: rgba(0,0,0,0.06);
  padding: 0.1em 0.3em;
  border-radius: 3px;
}

.prose pre {
  overflow-x: auto;
  padding: 1.2em 1.5em;
  background: rgba(0,0,0,0.05);
  border-radius: 4px;
  margin-bottom: var(--rhythm);
}

.prose pre code {
  background: none;
  padding: 0;
  font-size: 1em;
}

.prose blockquote {
  border-left: 3px solid currentColor;
  padding-left: 1em;
  font-style: italic;
  opacity: 0.8;
  margin-bottom: var(--rhythm);
}
```

**Think about it**: The `.prose` class restores list bullets and link underlines that the reset removed. This seems like undoing work. Why is it better to remove them globally and restore them in context, rather than keeping them globally and removing them where you don't want them?

## Exercises

1. **Scale Calculator**: Write a JavaScript function `typeScale(base, ratio, steps)` that returns an array of font sizes. `typeScale(16, 1.25, 7)` should return `[16, 20, 25, 31.25, 39.0625, 48.828125, 61.03515625]`. Then convert those values to rem (assuming the 62.5% trick, so 1rem = 10px).

2. **em vs rem Cascade**: Build a test page that demonstrates em compounding. Create three nested `<div>` elements, each with `font-size: 1.2em`. Without calculating, predict what the font-size of the innermost div will be in pixels (assume a 16px root). Then verify with DevTools. Repeat with `font-size: 1.2rem` on each — what changes?

3. **Fluid Type Range**: For an h1 that should be `24px` at `320px` viewport and `56px` at `1200px` viewport, calculate the `clamp()` expression. The formula for the preferred value slope is: `(max_size - min_size) / (max_vw - min_vw) × 100vw + (min_size - slope × min_vw)`. Show your work.

4. **Custom Property Type Scale**: Rewrite the type scale using CSS custom properties so that changing `--type-base` and `--type-ratio` on `:root` recalculates the entire scale. (Hint: CSS `calc()` supports multiplication. Use `calc(var(--type-base) * var(--step-6))` where `--step-6` is the precomputed multiplier for step 6.)

5. **Measure Audit**: Take any long-form webpage (a blog post, a documentation page, an article). Measure the line length in characters (count the characters in a typical full line). Is it within the 60–80 character optimal range? If not, what would you change and how? Then implement the fix with CSS `max-width` using the `ch` unit.

## Solutions

### Exercise 1 — Scale Calculator

```javascript
function typeScale(base, ratio, steps) {
  const sizes = [];
  for (let i = 0; i < steps; i++) {
    sizes.push(base * Math.pow(ratio, i));
  }
  return sizes;
}

// Returns pixel values
const px = typeScale(16, 1.25, 7);
console.log(px);
// [16, 20, 25, 31.25, 39.0625, 48.828125, 61.03515625]

// Convert to rem (with 62.5% trick, 1rem = 10px)
function pxToRem(px, rootPx = 10) {
  return px / rootPx;
}

const rems = px.map(v => pxToRem(v).toFixed(3) + 'rem');
console.log(rems);
// ["1.600rem", "2.000rem", "2.500rem", "3.125rem", "3.906rem", "4.883rem", "6.105rem"]

// Generate CSS custom properties
function generateScale(base, ratio, steps, rootPx = 10) {
  const sizes = typeScale(base, ratio, steps);
  return sizes.map((px, i) => {
    const rem = (px / rootPx).toFixed(3);
    return `  --text-${i}: ${rem}rem; /* ${px.toFixed(1)}px */`;
  }).join('\n');
}

console.log(':root {\n' + generateScale(16, 1.25, 7) + '\n}');
```

### Exercise 2 — em Compounding

```html
<!DOCTYPE html>
<html>
<head>
<style>
  html { font-size: 16px; }  /* root: 16px */

  /* em version — compounds */
  .em-test { border: 1px solid blue; padding: 0.5em; margin-bottom: 1rem; }
  .em-test .em-test { font-size: 1.2em; }  /* 16 × 1.2 = 19.2px */
  .em-test .em-test .em-test { font-size: 1.2em; } /* 19.2 × 1.2 = 23.04px */
  /* third level: 23.04 × 1.2 = 27.648px */

  /* rem version — does not compound */
  .rem-test { border: 1px solid red; padding: 0.5em; margin-bottom: 1rem; }
  .rem-test .rem-test { font-size: 1.2rem; }  /* 16 × 1.2 = 19.2px — always */
  /* third level: still 19.2px */
</style>
</head>
<body>
  <div class="em-test">Level 1 (em) — 16px
    <div class="em-test">Level 2 — 19.2px
      <div class="em-test">Level 3 — 23.04px
        <div class="em-test">Level 4 — 27.648px (em compounds!)</div>
      </div>
    </div>
  </div>

  <div class="rem-test">Level 1 (rem) — 16px
    <div class="rem-test">Level 2 — 19.2px
      <div class="rem-test">Level 3 — still 19.2px
        <div class="rem-test">Level 4 — still 19.2px (rem anchors to root)</div>
      </div>
    </div>
  </div>
</body>
</html>
```

### Exercise 3 — Fluid Type Calculation

```
Goal: 24px at 320px viewport, 56px at 1200px viewport

Step 1: Calculate the slope
  slope = (max_size - min_size) / (max_vw - min_vw)
  slope = (56 - 24) / (1200 - 320)
  slope = 32 / 880
  slope = 0.03636...
  slope as vw = 3.636vw

Step 2: Calculate the offset
  offset = min_size - slope × min_vw
  offset = 24 - 0.03636 × 320
  offset = 24 - 11.636
  offset = 12.364px
  offset as rem (÷10) = 1.236rem

Step 3: Assemble clamp()
  clamp(2.4rem, 3.636vw + 1.236rem, 5.6rem)

Verify at 320px:
  3.636% × 320 = 11.636px + 12.36px = 24px ✓

Verify at 1200px:
  3.636% × 1200 = 43.636px + 12.36px = 56px ✓

CSS:
h1 { font-size: clamp(2.4rem, 3.636vw + 1.236rem, 5.6rem); }
```

### Exercise 4 — Custom Property Type Scale

```css
:root {
  --type-base: 1.6;    /* in rem */
  --type-ratio: 1.25;

  /* Precomputed multipliers: ratio^n */
  --step--1: 0.8;      /* 1 ÷ 1.25 */
  --step-0:  1;
  --step-1:  1.25;     /* 1.25^1 */
  --step-2:  1.5625;   /* 1.25^2 */
  --step-3:  1.953125; /* 1.25^3 */
  --step-4:  2.441406; /* 1.25^4 */
  --step-5:  3.051758; /* 1.25^5 */
  --step-6:  3.814697; /* 1.25^6 */

  /* Computed sizes */
  --text-xs:  calc(var(--type-base) * var(--step--1) * 1rem);
  --text-sm:  calc(var(--type-base) * var(--step-0)  * 1rem);
  --text-md:  calc(var(--type-base) * var(--step-1)  * 1rem);
  --text-lg:  calc(var(--type-base) * var(--step-2)  * 1rem);
  --text-xl:  calc(var(--type-base) * var(--step-3)  * 1rem);
  --text-2xl: calc(var(--type-base) * var(--step-4)  * 1rem);
  --text-3xl: calc(var(--type-base) * var(--step-5)  * 1rem);
  --text-4xl: calc(var(--type-base) * var(--step-6)  * 1rem);
}

/* Apply the scale */
small { font-size: var(--text-xs); }
body  { font-size: var(--text-sm); }
h6    { font-size: var(--text-md); }
h5    { font-size: var(--text-lg); }
h4    { font-size: var(--text-xl); }
h3    { font-size: var(--text-2xl); }
h2    { font-size: var(--text-3xl); }
h1    { font-size: var(--text-4xl); }

/* Now: change --type-base from 1.6 to 1.8 and the entire scale shifts */
/* Change --type-ratio from 1.25 to 1.333 for a more dramatic scale */
/* Note: CSS calc() cannot compute powers, so multipliers are precomputed */
```

### Exercise 5 — Measure Audit

```css
/* Typical findings: most websites have no max-width on text, producing
   lines 120+ characters wide on large monitors — nearly double the optimal.

   The fix is straightforward: */

.article-content,
.blog-post,
.prose {
  max-width: 68ch;   /* ~70 chars — optimal for body text */
  margin-left: auto;
  margin-right: auto;
}

/* For larger text (24px+), optimal line length is shorter (~50-60 chars) */
.lead-text {
  max-width: 55ch;
}

/* For UI components, line length is less critical — don't constrain */
.card,
.navbar,
.sidebar {
  /* no max-width based on ch */
}

/* Verify your ch value: */
/* In DevTools console: */
/* document.createElement('span').style.font = getComputedStyle(document.querySelector('.prose')).font */
/* That element's width at 1ch is the character reference width */
```

## What You Learned

| Concept | Key point |
|---------|-----------|
| 62.5% trick | `html { font-size: 62.5% }` makes 1rem = 10px; rem values map cleanly to px values you recognize |
| rem vs em | rem is relative to root; em is relative to current element (and compounds in nesting) |
| Modular type scale | Base size × ratio^n; change one number and the whole system rescales consistently |
| 1.25 ratio | Major Third — clear hierarchy without extreme size differences |
| Line height decreases with size | Body text needs 1.6; display headings need 1.15; large type has inherent visual separation |
| `clamp(min, preferred, max)` | Fluid type that scales smoothly with viewport width; no media query breakpoints needed |
| `ch` unit | Width of the "0" glyph; `68ch` max-width gives optimal 60–80 character line length |
| System font stack | Zero network request, native OS appearance; right choice for application UIs |
| Vertical rhythm | Consistent spacing in multiples of `line-height × font-size`; creates visual order |
| `.prose` context class | Restore typographic details (bullets, underlines) in long-form reading contexts only |

## Building with Claude

**Bad prompt:**
> "What font should I use for my website?"

This returns a list of popular fonts and generic advice about pairing serifs with sans-serifs. It does not help you build a system.

**Good prompt:**
> "I'm building a type scale for a developer tool UI (dark background, monospace-heavy, technical content). I want: (1) a system font stack that works on macOS, Windows, and Linux with no web font request, (2) a modular scale with 1.25 ratio and 1.6rem base (assuming html { font-size: 62.5% }), (3) fluid h1 and h2 using clamp() that go from roughly 28px at 320px viewport to 56px at 1280px, (4) a separate monospace stack for code blocks, (5) a --rhythm custom property for vertical spacing. Output clean CSS I can paste directly into my reset file, with one comment per rule explaining the decision."
