# Chapter 1 — Build Your Own CSS Reset

Every browser ships with a default stylesheet. Before your first rule is parsed, Chrome has already set `<h1>` to `2em` bold, `<body>` to `8px` margin, `<ul>` to `40px` padding-left, and `<button>` to whatever the operating system's native button style looks like. These defaults differ across browsers, differ across operating systems, and differ from what you actually want. Your CSS has to fight them on every project. In this chapter you build a CSS reset from scratch — the 30-odd rules that establish a clean, predictable baseline before any real styling begins.

---

## The Problem

Open a blank HTML file in Chrome and Firefox side by side. Without any CSS, the heading sizes differ. The form element fonts differ. The `<table>` spacing differs. The `<button>` appearance differs. The list indentation differs.

The naive approach is to fix these one at a time, in your component CSS, as you discover them:

```css
/* scattered across multiple files */
.nav ul { padding: 0; list-style: none; }
.sidebar h2 { font-size: 1.2rem; }
.card button { font-family: inherit; }
```

The problem: you're fixing the same browser default in a dozen different places. When a new developer adds a `<ul>` to a component, they get the browser's 40px left padding back. The fix is to establish the baseline *once*, at the top of the cascade, before any component styles run.

That's what a CSS reset does. Three questions drive every rule:

```
What is the browser's default for this element?   →  browser stylesheet
Is that default ever what we want?                 →  almost never
What baseline makes components easiest to build?   →  reset to that
```

---

## Building It Step by Step

### v1 — The Box Model Fix

The single most impactful reset rule. Browsers default to `box-sizing: content-box`, which means width and height don't include padding or borders. This makes layout math unintuitive:

```css
/* v1: box-sizing reset — the most important rule in any stylesheet */
*,
*::before,
*::after {
  box-sizing: border-box;
}
```

With `content-box` (the default):
```
element width = content width + padding + border
width: 200px + padding: 20px = 240px actual width  ← surprising
```

With `border-box` (what we want):
```
element width = total rendered width, including padding and border
width: 200px + padding: 20px = 200px actual width  ← predictable
```

The `*` selector with `*::before` and `*::after` covers every element and both pseudo-elements. The `*` selector has zero specificity — any rule in your stylesheet can override it.

### v2 — Margin, Padding, and Typography Reset

Browser default margins and padding are inconsistent and almost never the values you want in a component system:

```css
/* v2: remove all default spacing and normalize typography */
*,
*::before,
*::after {
  box-sizing: border-box;
  margin: 0;
  padding: 0;
}

body {
  line-height: 1.5;
  -webkit-font-smoothing: antialiased;
}

h1, h2, h3, h4, h5, h6 {
  font-size: inherit;
  font-weight: inherit;
}

ul, ol {
  list-style: none;
}

img, picture, video, canvas, svg {
  display: block;
  max-width: 100%;
}

input, button, textarea, select {
  font: inherit;
}
```

`font-size: inherit` on headings removes the `2em`/`1.5em`/etc. sizing — your type scale will set these deliberately, not accidentally. `font: inherit` on form elements is perhaps the most-forgotten rule: browsers default form element fonts to the OS system font, not your body font.

### v3 — Sensible Defaults

The reset isn't just zeroing things out — it also establishes sensible baselines:

```css
/* v3: complete reset with sensible defaults */
*,
*::before,
*::after {
  box-sizing: border-box;
  margin: 0;
  padding: 0;
}

:root {
  /* Prevent text size adjustment on mobile orientation change */
  -webkit-text-size-adjust: 100%;
  /* Set consistent tab size */
  tab-size: 4;
}

body {
  min-height: 100vh;          /* body at least fills viewport */
  line-height: 1.5;
  -webkit-font-smoothing: antialiased;
  text-rendering: optimizeSpeed;
}

h1, h2, h3, h4, h5, h6 {
  font-size: inherit;
  font-weight: inherit;
}

ul[role="list"], ol[role="list"] {
  list-style: none;            /* only reset lists that are explicitly lists */
}

img, picture, video, canvas, svg {
  display: block;              /* remove bottom gap from inline images */
  max-width: 100%;             /* images never overflow their container */
}

input, button, textarea, select {
  font: inherit;               /* form elements use page font, not OS default */
}

p, h1, h2, h3, h4, h5, h6 {
  overflow-wrap: break-word;   /* prevent long words from causing overflow */
}

#root, #__next {
  isolation: isolate;          /* create a new stacking context for JS frameworks */
}
```

The `ul[role="list"]` selector is intentional. The WCAG accessibility guidelines say that removing `list-style` from a `<ul>` causes screen readers to stop announcing it as a list. By only resetting lists that have `role="list"`, you tell both the browser and screen readers that this is a navigational/functional list, not a document list.

---

## The Complete Program

`css-reset.html` — open it in your browser. Toggle "Show Reset" to see what changes. Click each category to see the before/after for that property group.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Build Your Own CSS Reset — Chapter 1</title>
  <!-- styles in css-reset.html -->
</head>
<body>
<style>
/* ═══════════════════════════════════════════════════════════════════════
   THE LIBRARY: miniReset
   ═══════════════════════════════════════════════════════════════════════ */

*,
*::before,
*::after {
  box-sizing: border-box;
  margin: 0;
  padding: 0;
}

:root { -webkit-text-size-adjust: 100%; tab-size: 4; }
body  { min-height: 100vh; line-height: 1.5; -webkit-font-smoothing: antialiased; }

h1, h2, h3, h4, h5, h6 { font-size: inherit; font-weight: inherit; }
ul[role="list"], ol[role="list"] { list-style: none; }
img, picture, video, canvas, svg { display: block; max-width: 100%; }
input, button, textarea, select  { font: inherit; }
p, h1, h2, h3, h4, h5, h6        { overflow-wrap: break-word; }
#root, #__next                    { isolation: isolate; }
</style>
</body>
</html>
```

---

## Walkthrough

### The Cascade Algorithm

"Cascade" is the C in CSS. It's the algorithm that decides which rule wins when two rules target the same element and property. The algorithm has four steps, applied in order:

```
1. Origin and importance
   Browser stylesheet < Author stylesheet < Author !important < Browser !important
   (You rarely touch this — just know your rules beat the browser's.)

2. Specificity
   The weight of the selector. Higher specificity wins.
   [inline styles] > [ID selectors] > [class/attr/pseudo-class] > [element/pseudo-element]

3. Source order
   When specificity ties, the later rule wins.

4. Inheritance
   Some properties inherit from parent to child automatically.
   Others don't. (See "Inheritance" section below.)
```

Our reset wins over browser defaults because it's in the author stylesheet (step 1), and uses `*` which has zero specificity — any rule you write later can override it (step 2).

### The Box Model

Every element is a rectangular box made of four layers:

```
┌─────────────────────────────────────────┐  ← margin edge
│               margin                    │
│  ┌───────────────────────────────────┐  │  ← border edge
│  │            border                 │  │
│  │  ┌─────────────────────────────┐  │  │  ← padding edge
│  │  │          padding            │  │  │
│  │  │  ┌───────────────────────┐  │  │  │  ← content edge
│  │  │  │       content         │  │  │  │
│  │  │  └───────────────────────┘  │  │  │
│  │  └─────────────────────────────┘  │  │
│  └───────────────────────────────────┘  │
└─────────────────────────────────────────┘
```

`content-box` (browser default): `width` applies to the *content* box only. Padding and border add on top.

`border-box` (our reset): `width` applies to the *border* box. Padding and border are subtracted from the content area instead. Your specified `width` is the total rendered width.

Why does this matter? With `content-box`, a grid of 4 items `width: 25%` breaks if you add `padding: 10px`. With `border-box`, it doesn't — padding is absorbed inside the 25%.

### Specificity

Specificity is a three-number score written as `(A, B, C)`:

```
A = number of ID selectors (#header, #nav)
B = number of class, attribute, pseudo-class selectors (.btn, [type], :hover)
C = number of element and pseudo-element selectors (div, h1, ::before)

Selector              Score    Wins against
─────────────────────────────────────────────────────────────
*                     (0,0,0)  Nothing — lowest possible
div                   (0,0,1)  * only
.button               (0,1,0)  div, * — class beats element
div.button            (0,1,1)  .button alone
#header               (1,0,0)  Everything above
#header .button       (1,1,0)  #header alone
style="color:red"     —        Beats all selectors (inline wins)
```

The `*` in our reset has score `(0,0,0)` — it can be overridden by *anything*. This is intentional. The reset is a floor, not a ceiling.

### Inheritance

Some CSS properties automatically pass from parent to child. Others don't.

**Inherited by default** (propagate down the tree):
- Typography: `font-family`, `font-size`, `font-weight`, `line-height`, `color`
- Text: `text-align`, `text-transform`, `white-space`, `word-spacing`
- Lists: `list-style`, `list-style-type`
- Visibility: `visibility` (not `display`), `cursor`

**Not inherited by default** (only apply to the element they're set on):
- Box model: `width`, `height`, `margin`, `padding`, `border`
- Background: `background`, `background-color`
- Layout: `display`, `position`, `float`, `overflow`
- Visual: `box-shadow`, `opacity`, `transform`

This is why `font: inherit` on form elements is needed. `font-family` and `font-size` are inherited properties — they *should* pass from `body` to every child, including form elements. But browsers override this inheritance for `<input>` and `<button>` with a browser stylesheet rule of higher specificity (the OS system font rule). `font: inherit` explicitly resets that override.

### Why `display: block` on Images

Inline elements (like `<img>` by default) sit on a text baseline. The text baseline doesn't sit at the bottom of the line — there's a small gap between the baseline and the bottom of the line box (for descenders like `g` and `p`). This creates a mysterious gap below inline images that CSS beginners spend hours debugging.

`display: block` removes the element from inline flow entirely. The gap disappears.

```css
/* Before reset: mysterious 4px gap below every image */
img { /* display: inline (default) */ }

/* After reset: no gap */
img { display: block; }
```

### The Accessibility List Selector

```css
ul[role="list"], ol[role="list"] { list-style: none; }
```

The `[role="list"]` attribute selector only matches elements that have `role="list"` in the HTML. This means:

```html
<!-- This list keeps its bullets (document content): -->
<ul>
  <li>First item</li>
  <li>Second item</li>
</ul>

<!-- This list loses its bullets (navigation/UI): -->
<ul role="list">
  <li><a href="/home">Home</a></li>
  <li><a href="/about">About</a></li>
</ul>
```

When you add `role="list"`, you're explicitly telling both the browser and screen readers: "this is a functional list, not a document list." The `list-style: none` reset is safe on that basis.

---

## Guided Try It — Add a Print Stylesheet

**The goal**: add a `@media print` block to the reset that removes backgrounds, reduces colors, and ensures links show their URLs when printed.

**Why this is useful**: print stylesheets are forgotten by nearly every web developer, but they're the entire interface between your page and a user's PDF export or physical printout.

**Step 1 — Hide decorative elements**

```css
/* Add to your reset: */
@media print {
  *,
  *::before,
  *::after {
    background: transparent !important;
    color: black !important;
    box-shadow: none !important;
    text-shadow: none !important;
  }
}
```

The `!important` is acceptable here because print styles intentionally override everything else — this is one of the rare legitimate uses.

**Step 2 — Show link URLs**

```css
@media print {
  a[href]::after {
    content: " (" attr(href) ")";
    font-size: 0.85em;
    color: #555;
  }

  /* Skip internal and javascript: links */
  a[href^="#"]::after,
  a[href^="javascript:"]::after {
    content: "";
  }
}
```

The `attr(href)` function in `content` reads the value of the `href` attribute and inserts it as text. `a[href^="#"]` is an attribute selector that matches links starting with `#` (anchor links).

**Step 3 — Prevent bad page breaks**

```css
@media print {
  h1, h2, h3 { page-break-after: avoid; }
  p           { orphans: 3; widows: 3; }
  table       { page-break-inside: avoid; }
}
```

`page-break-after: avoid` prevents a page break immediately after a heading. `orphans` and `widows` control how many lines must remain at the bottom/top of a page before a paragraph is split.

**Think about it**: the `@media print` block goes at the *bottom* of the reset. Why? Consider specificity and source order: all our print rules need to override the page styles. Since print rules use `!important` for color/background, source order doesn't matter for those — but for layout rules like `page-break-after`, order matters. By putting the print block last, you guarantee it wins on anything that's a specificity tie.

---

## Exercises

1. **Add `border-collapse` to tables**: Tables have `border-collapse: separate` by default, which creates double borders between cells. Add a rule to the reset that sets `border-collapse: collapse` and `border-spacing: 0` on all tables. Test: create a `<table>` with borders and see the difference.

2. **Add a focus-visible ring**: Some resets remove `:focus` outlines (`:focus { outline: none }`), which is an accessibility disaster. Instead, add a rule that keeps focus styles for keyboard navigation but hides them for mouse clicks. Use `:focus-visible` with a `2px solid` outline in a visible color.

3. **Normalize button appearance**: Browsers style `<button>` with a platform-native appearance. Add a rule targeting `button` that removes `appearance`, sets `cursor: pointer`, and inherits `color` from the parent. Why is `cursor: pointer` needed? (Hint: `default` is the browser's default for buttons in some browsers.)

4. **Add a `prefers-reduced-motion` block**: Some users have vestibular disorders or motion sensitivity. Add a `@media (prefers-reduced-motion: reduce)` block that sets `animation-duration: 0.01ms`, `animation-iteration-count: 1`, and `transition-duration: 0.01ms` on `*`. This disables all motion when the user has requested reduced motion in their OS settings.

5. **Design your own "opinionated reset"**: A reset removes browser defaults. An *opinionated reset* also adds sensible base styles. Design a 15–20 rule opinionated reset that includes: body font stack with system fonts (`-apple-system, BlinkMacSystemFont, "Segoe UI", sans-serif`), `line-height: 1.6` on body, `1rem` paragraph font size, sensible heading sizes using a 1.25 modular scale (ems only, no px), and a `:root` with a `font-size: 100%` baseline. The goal: every page using this reset should look like a readable document without any additional CSS.

---

## Solutions

### Exercise 1 — Table border collapse

```css
table {
  border-collapse: collapse;
  border-spacing: 0;
}
```

`border-collapse: collapse` merges adjacent cell borders into a single border. `border-spacing: 0` has no visible effect once `collapse` is set (they're mutually exclusive), but it's good practice to zero it out so the table behaves predictably if someone later sets `border-collapse: separate`.

```html
<!-- Test: -->
<table style="border: 1px solid black">
  <tr>
    <td style="border: 1px solid black; padding: 8px">Cell 1</td>
    <td style="border: 1px solid black; padding: 8px">Cell 2</td>
  </tr>
</table>
<!-- Without reset: double border between cells -->
<!-- With reset: single border between cells -->
```

### Exercise 2 — Focus-visible ring

```css
/* Remove focus ring for mouse/touch users */
:focus:not(:focus-visible) {
  outline: none;
}

/* Keep focus ring for keyboard users */
:focus-visible {
  outline: 2px solid #005fcc;
  outline-offset: 2px;
}
```

`:focus-visible` is a pseudo-class that applies when the browser determines the focus indicator should be visible — typically when the user navigated with a keyboard (Tab key). For mouse clicks, it doesn't apply, so `:focus:not(:focus-visible)` removes the outline for those users. This is the correct accessibility pattern: never remove focus styles entirely, only suppress them when they're not needed.

### Exercise 3 — Button appearance

```css
button {
  appearance: none;
  -webkit-appearance: none;
  cursor: pointer;
  color: inherit;
  background: transparent;
  border: none;
}
```

`appearance: none` removes the OS-native button look. `cursor: pointer` is needed because some browsers default to `cursor: default` on buttons (the arrow cursor) rather than `cursor: pointer` (the hand). `color: inherit` is required because color is an inherited property, but button elements in some browsers have a browser stylesheet rule that overrides inheritance with the OS button text color.

### Exercise 4 — Reduced motion

```css
@media (prefers-reduced-motion: reduce) {
  *,
  *::before,
  *::after {
    animation-duration: 0.01ms !important;
    animation-iteration-count: 1 !important;
    transition-duration: 0.01ms !important;
    scroll-behavior: auto !important;
  }
}
```

`0.01ms` rather than `0` because some JavaScript libraries check whether `animation-duration` is `0` to determine if animation is disabled and behave differently. `0.01ms` is imperceptibly fast but not `0`. `scroll-behavior: auto` disables smooth scrolling, which can cause motion sickness for users with vestibular disorders.

### Exercise 5 — Opinionated reset

```css
/* ── Opinionated Reset ──────────────────────────────────── */

*,
*::before,
*::after {
  box-sizing: border-box;
  margin: 0;
  padding: 0;
}

:root {
  font-size: 100%;          /* 1rem = browser default (typically 16px) */
  -webkit-text-size-adjust: 100%;
  tab-size: 4;
}

body {
  font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto,
               Helvetica, Arial, sans-serif, "Apple Color Emoji";
  font-size: 1rem;
  line-height: 1.6;
  min-height: 100vh;
  -webkit-font-smoothing: antialiased;
}

h1 { font-size: 2.441em; }    /* 1.25^5 */
h2 { font-size: 1.953em; }    /* 1.25^4 */
h3 { font-size: 1.563em; }    /* 1.25^3 */
h4 { font-size: 1.25em;  }    /* 1.25^2 */
h5 { font-size: 1em;     }    /* 1.25^1 */
h6 { font-size: 0.8em;   }    /* 1.25^0 */

h1, h2, h3, h4, h5, h6 {
  font-weight: 700;
  line-height: 1.25;
  margin-bottom: 0.5em;
}

p  { margin-bottom: 1em; }
p:last-child { margin-bottom: 0; }

img, picture, video, canvas, svg {
  display: block;
  max-width: 100%;
}

input, button, textarea, select { font: inherit; }
button { cursor: pointer; }

:focus-visible {
  outline: 2px solid #005fcc;
  outline-offset: 2px;
}

@media (prefers-reduced-motion: reduce) {
  *, *::before, *::after {
    animation-duration: 0.01ms !important;
    transition-duration: 0.01ms !important;
  }
}
```

The `em`-based heading sizes in `h1`–`h6` are relative to the *element's* font size, not the root. Because we set `font-size: inherit` in nothing (this is the opinionated reset that sets heading sizes), the ems compound from the body `1rem`. The `1.25` ratio is the "Major Third" scale — a common choice for readable text that doesn't get too extreme at the top end.

---

## What You Learned

| Concept | Key point |
|---------|-----------|
| The cascade | Origin → specificity → source order → inheritance. Browser styles lose to author styles. |
| `box-sizing: border-box` | Makes `width` the *total* rendered width including padding and border |
| `*` selector | Zero specificity — any rule you write overrides it |
| `content-box` vs `border-box` | Default causes padding to expand elements; `border-box` absorbs it |
| Specificity score | `(IDs, classes/attrs/pseudo-classes, elements)` — higher wins |
| Inheritance | Typography properties inherit; layout/box properties don't |
| `font: inherit` on form elements | Browsers override font inheritance for inputs/buttons; this restores it |
| `display: block` on `<img>` | Removes the inline baseline gap below images |
| `:focus-visible` | Shows focus ring for keyboard, hides it for mouse — the correct accessibility pattern |
| `ul[role="list"]` | Only resets list style when the list has semantic role="list" markup |
| `@media (prefers-reduced-motion)` | Disables animations/transitions for users who need it |
| `@media print` | Entirely separate stylesheet for print/PDF output |

---

## Building with Claude

Bad prompt:
> "How do I fix button styles in CSS?"

Good prompt:
> "I'm building a CSS reset in plain CSS (no frameworks). I have `font: inherit` on form elements and `appearance: none` on buttons. A `<button>` inside a `<form>` still shows the OS system border in Safari. My reset rule is `button { appearance: none; -webkit-appearance: none; border: none; }`. Is there a Safari-specific property I'm missing, or is something in my cascade overriding the reset? The button has no class or id — it's a plain `<button>Submit</button>`."

The good prompt shows your exact rules, names the browser, describes the visual symptom, and asks a specific question. Claude's answer will be about *your* button rule — not a generic tutorial on button styling.
