# Chapter 1 — Build Your Own Reset/Normalize

Every browser ships with a default stylesheet. Chrome, Firefox, Safari, and Edge all have one, and they disagree on the details. Headings get different margins. Lists get different padding. Buttons render with different fonts. Form inputs inherit no font at all. The first time you write CSS without a reset, you spend an afternoon debugging why your heading looks fine in Chrome and wrong in Firefox. The reset exists to prevent that afternoon.

Building your own reset forces you to understand what defaults you are overriding and why. Grabbing normalize.css from a CDN is easy; knowing which of its 400 lines are doing work you actually need is harder. This chapter builds a reset from first principles, starting with the single most important rule in CSS and expanding outward until you have a minimal, production-ready stylesheet you understand completely.

This reset is the foundation every other chapter in this book uses. When the Typography chapter's headings look right and the Layout chapter's boxes land where you expect, it is because of exactly the rules built here. When Chapter 6's jQuery demo and Chapter 13's Debugger look consistent across browsers, it is because of this reset.

## The Problem

Open a browser and write this HTML with no stylesheet:

```html
<h1>Title</h1>
<ul>
  <li>Item one</li>
  <li>Item two</li>
</ul>
<button>Click me</button>
<input type="text" placeholder="Type here">
```

What you get is a collection of browser decisions you did not make. The `<h1>` has a top and bottom margin the browser chose. The `<ul>` has left padding and list-style bullets the browser chose. The `<button>` has a border, background, and font that differ between browsers. The `<input>` inherits no font from its parent — it uses the browser's UI font, which is not your page font.

The naive approach is to reset things as you discover them: add `margin: 0` when a margin surprises you, remove `list-style` when a bullet bothers you. This is slow, inconsistent, and leaves gaps you will not find until your page is in production. The correct approach is to reset everything at the start and add back exactly what you want.

There are two philosophies here. The **reset** philosophy (Eric Meyer's original reset.css) removes everything: all margins, all padding, all font sizes, all list decorations. You start from zero and build up. The **normalize** philosophy (normalize.css) instead makes browsers agree without removing useful defaults. Both are valid. This chapter builds a hybrid: opinionated removal of the things that cause bugs, with sensible preservation of the things that help.

## Building It Step by Step

### v1 — The One Rule That Matters

Before anything else, fix the box model. This is the single most important rule you will ever write in CSS.

```css
/* v1: micro-reset.css */
*,
*::before,
*::after {
  box-sizing: border-box;
}
```

This rule applies to every element and every pseudo-element. Here is what it changes.

The default box model is `content-box`. When you write `width: 200px; padding: 20px; border: 2px solid`, the element renders at 244px wide — 200px content, plus 20px padding on each side, plus 2px border on each side. The declared width does not match the rendered width. This is a constant source of layout bugs.

With `border-box`, the width you declare is the width you get. Padding and border are included in that number. `width: 200px; padding: 20px; border: 2px solid` renders at exactly 200px. The content area shrinks to 156px to accommodate the padding and border, but the box stays at 200px.

The `*` selector catches every element. The `::before` and `::after` selectors catch pseudo-elements, which are generated content boxes that also default to `content-box`.

Add the second most important pair of rules:

```css
/* v1 continued */
*,
*::before,
*::after {
  box-sizing: border-box;
  margin: 0;
  padding: 0;
}
```

Setting `margin: 0` and `padding: 0` on everything removes the browser's spacing decisions. Now you add spacing deliberately, only where you need it. Nothing is hidden. Nothing surprises you.

### v2 — Element-Specific Resets

The universal selector handles the box model. Now fix the elements that have their own problems.

```css
/* v2: micro-reset.css */
*,
*::before,
*::after {
  box-sizing: border-box;
  margin: 0;
  padding: 0;
}

/* Typography */
h1, h2, h3, h4, h5, h6 {
  font-size: inherit;
  font-weight: inherit;
}

/* Lists */
ul, ol {
  list-style: none;
}

/* Links */
a {
  color: inherit;
  text-decoration: inherit;
}

/* Media */
img, video, canvas, svg {
  display: block;
  max-width: 100%;
}

/* Form elements */
button, input, select, textarea {
  font: inherit;
  color: inherit;
  background: none;
  border: none;
  padding: 0;
  cursor: pointer;
}

textarea {
  resize: vertical;
}

/* Tables */
table {
  border-collapse: collapse;
  border-spacing: 0;
}
```

Walk through each group.

**Headings**: Browsers render `<h1>` through `<h6>` with different `font-size` and `font-weight` values. Setting both to `inherit` makes headings plain text by default. You add size and weight back through your typography system (Chapter 2), not through browser defaults. This means your headings are styled consistently whether they appear in an article, a sidebar, or a dialog.

**Lists**: The default list has `list-style: disc` or `list-style: decimal` plus left padding to indent the bullets. Removing both means `<ul>` and `<ol>` are just vertical stacks of items. You add bullets back when the context calls for them, typically a class like `.prose ul` rather than globally.

**Links**: By default, `<a>` elements are bright blue and underlined. Neither is necessarily wrong, but both should be your choice, not the browser's. `color: inherit` and `text-decoration: inherit` make links look like their surrounding text until you style them explicitly.

**Media**: Images are `inline` by default, which means they sit on the text baseline and get a small gap underneath. `display: block` eliminates that gap. `max-width: 100%` means images never overflow their containers — a baseline responsive behavior you almost always want.

**Form elements**: This is the most important group after the box model. `font: inherit` is the critical rule — browsers apply a separate UI font to form elements that does not inherit from the page. Without this rule, your `<button>` and `<input>` will use the system UI font while everything else uses your declared font. `background: none; border: none` removes the visual affordances so you can style them from scratch.

### v3 — Cross-Browser Normalization and Sensible Defaults

The final version adds rules that fix specific cross-browser inconsistencies and add defaults that almost every project needs.

```css
/* v3: micro-reset.css — complete version */

/* Box model */
*,
*::before,
*::after {
  box-sizing: border-box;
  margin: 0;
  padding: 0;
}

/* Root */
html {
  -webkit-text-size-adjust: 100%;
  tab-size: 4;
  scroll-behavior: smooth;
}

body {
  min-height: 100vh;
  line-height: 1.5;
  -webkit-font-smoothing: antialiased;
  text-rendering: optimizeLegibility;
}

/* Typography */
h1, h2, h3, h4, h5, h6 {
  font-size: inherit;
  font-weight: inherit;
}

p, h1, h2, h3, h4, h5, h6 {
  overflow-wrap: break-word;
}

/* Lists */
ul, ol {
  list-style: none;
}

/* Links */
a {
  color: inherit;
  text-decoration: inherit;
}

/* Media */
img, picture, video, canvas, svg {
  display: block;
  max-width: 100%;
}

img, video {
  height: auto;
}

/* Form elements */
button, input, select, textarea, optgroup {
  font: inherit;
  color: inherit;
  background: none;
  border: none;
  padding: 0;
}

button, [type="button"], [type="reset"], [type="submit"] {
  cursor: pointer;
  -webkit-appearance: button;
}

button:disabled {
  cursor: default;
}

textarea {
  resize: vertical;
}

input[type="search"] {
  -webkit-appearance: none;
  outline-offset: -2px;
}

/* Tables */
table {
  border-collapse: collapse;
  border-spacing: 0;
}

/* Interactive */
summary {
  cursor: pointer;
}

/* Accessibility */
[hidden] {
  display: none !important;
}

:focus-visible {
  outline: 2px solid currentColor;
  outline-offset: 2px;
}
```

The additions in v3 each earn their place.

`-webkit-text-size-adjust: 100%` prevents iOS Safari from automatically inflating font sizes when the device rotates to landscape. Without this rule, your carefully sized text will grow on rotation and your layout will break.

`min-height: 100vh` on `body` ensures the body element fills the viewport even with little content. This prevents the background color from ending halfway down a short page.

`-webkit-font-smoothing: antialiased` applies subpixel antialiasing on macOS WebKit. `text-rendering: optimizeLegibility` enables kerning and ligatures. Both make text look sharper on modern displays.

`overflow-wrap: break-word` prevents long words and URLs from overflowing their containers. Without this, a long URL in a paragraph will push the text off the side of the page on mobile.

The `:focus-visible` rule is the accessibility reset. Browsers removed the default focus outline in some contexts because it looked bad, but the outline is essential for keyboard navigation. `:focus-visible` applies the outline only when the element was focused via keyboard, not mouse — preserving the visual affordance for keyboard users without showing it on mouse click.

`[hidden] { display: none !important }` enforces the HTML `hidden` attribute. Some CSS libraries accidentally override this attribute by setting `display: block` on elements, breaking the semantic hiding behavior. The `!important` here is deliberate and correct.

## The Complete Program

The complete reset is in `reset.html`. The CSS listing that constitutes the `micro-reset.css` library:

```css
/* micro-reset.css — HTML & CSS: The Kernighan Way */

*,
*::before,
*::after {
  box-sizing: border-box;
  margin: 0;
  padding: 0;
}

html {
  -webkit-text-size-adjust: 100%;
  tab-size: 4;
  scroll-behavior: smooth;
}

body {
  min-height: 100vh;
  line-height: 1.5;
  -webkit-font-smoothing: antialiased;
  text-rendering: optimizeLegibility;
}

h1, h2, h3, h4, h5, h6 {
  font-size: inherit;
  font-weight: inherit;
}

p, h1, h2, h3, h4, h5, h6 {
  overflow-wrap: break-word;
}

ul, ol {
  list-style: none;
}

a {
  color: inherit;
  text-decoration: inherit;
}

img, picture, video, canvas, svg {
  display: block;
  max-width: 100%;
}

img, video {
  height: auto;
}

button, input, select, textarea, optgroup {
  font: inherit;
  color: inherit;
  background: none;
  border: none;
  padding: 0;
}

button, [type="button"], [type="reset"], [type="submit"] {
  cursor: pointer;
  -webkit-appearance: button;
}

button:disabled {
  cursor: default;
}

textarea {
  resize: vertical;
}

input[type="search"] {
  -webkit-appearance: none;
  outline-offset: -2px;
}

table {
  border-collapse: collapse;
  border-spacing: 0;
}

summary {
  cursor: pointer;
}

[hidden] {
  display: none !important;
}

:focus-visible {
  outline: 2px solid currentColor;
  outline-offset: 2px;
}
```

## Walkthrough

### The Box Model

Every element in CSS is a rectangular box. The box has four layers:

```
┌─────────────────────────────────────────────────┐
│                    margin                       │
│   ┌─────────────────────────────────────────┐   │
│   │                 border                  │   │
│   │   ┌─────────────────────────────────┐   │   │
│   │   │             padding             │   │   │
│   │   │   ┌─────────────────────────┐   │   │   │
│   │   │   │                         │   │   │   │
│   │   │   │        content          │   │   │   │
│   │   │   │                         │   │   │   │
│   │   │   └─────────────────────────┘   │   │   │
│   │   └─────────────────────────────────┘   │   │
│   └─────────────────────────────────────────┘   │
└─────────────────────────────────────────────────┘
```

The `width` and `height` properties target the content box by default (`box-sizing: content-box`). Padding and border are added on top:

```
╔══════════════════════════════════════════════════════╗
║  content-box (browser default — avoid this)          ║
║                                                      ║
║  CSS:  width: 200px; padding: 20px; border: 2px      ║
║                                                      ║
║  ┌──────────────────────────────────────────────┐    ║
║  │  border (2px)                                │    ║
║  │  ┌────────────────────────────────────────┐  │    ║
║  │  │  padding (20px)                        │  │    ║
║  │  │  ┌──────────────────────────────────┐  │  │    ║
║  │  │  │  content  200px                  │  │  │    ║
║  │  │  └──────────────────────────────────┘  │  │    ║
║  │  └────────────────────────────────────────┘  │    ║
║  └──────────────────────────────────────────────┘    ║
║                                                      ║
║  Rendered = 200 + 20 + 20 + 2 + 2 = 244px           ║
║  Your declared width ≠ actual rendered width         ║
╚══════════════════════════════════════════════════════╝

╔══════════════════════════════════════════════════════╗
║  border-box (use this always)                        ║
║                                                      ║
║  CSS:  width: 200px; padding: 20px; border: 2px      ║
║                                                      ║
║  ┌──────────────────────────────────────────────┐    ║
║  │  border (2px)                                │    ║
║  │  ┌────────────────────────────────────────┐  │    ║
║  │  │  padding (20px)                        │  │    ║
║  │  │  ┌──────────────────────────────────┐  │  │    ║
║  │  │  │  content  156px                  │  │  │    ║
║  │  │  └──────────────────────────────────┘  │  │    ║
║  │  └────────────────────────────────────────┘  │    ║
║  └──────────────────────────────────────────────┘    ║
║                 │◄────── 200px ──────►│              ║
║  Rendered = 200px. Always.                           ║
╚══════════════════════════════════════════════════════╝
```

`border-box` is not a preference. It is the correct mental model for layout. When you write `width: 50%`, you want the element to take up half its container — including its padding. Without `border-box`, two 50%-wide elements with any padding will not fit side by side.

### The Specificity Cascade

CSS rules are resolved by specificity. Each selector has a score calculated from three components:

```
Specificity: (ID count, Class count, Type count)

┌─────────────────────────────────┬────────────────┐
│  Selector                       │  Score         │
├─────────────────────────────────┼────────────────┤
│  a                              │  0, 0, 1       │
│  .button                        │  0, 1, 0       │
│  #header                        │  1, 0, 0       │
│  div.container                  │  0, 1, 1       │
│  #nav .item a                   │  1, 1, 1       │
│  [type="text"]                  │  0, 1, 0       │
│  :hover                         │  0, 1, 0       │
│  ::before                       │  0, 0, 1       │
│  *                              │  0, 0, 0       │
└─────────────────────────────────┴────────────────┘
```

Higher specificity wins. When specificities are equal, the later rule wins. The universal selector `*` has specificity `(0, 0, 0)` — lower than any element. This is why a reset using `*` is safe: any rule you write for a specific element overrides it.

```
┌────────────────────────────────────────────────────────┐
│  Reset rule:        * { margin: 0 }         0,0,0      │
│  Your rule:         p { margin: 1em }       0,0,1  ← wins
│  Your class rule:   .text { margin: 0.5em } 0,1,0  ← wins
│  Your ID rule:      #content { margin: 2em }1,0,0  ← wins
└────────────────────────────────────────────────────────┘
```

The reset always loses. That is the design.

### Inheritance

Some CSS properties inherit from parent to child; most do not. Typography properties inherit: `font-family`, `font-size`, `color`, `line-height`. Layout properties do not: `margin`, `padding`, `border`, `width`, `display`.

```
body { font-family: 'Courier New'; color: #c8c8d8; }

┌─── body ──────────────────────────────────────────────┐
│  font-family: 'Courier New'  ← declared               │
│  color: #c8c8d8              ← declared               │
│                                                       │
│  ┌─── p ─────────────────────────────────────────┐    │
│  │  font-family: 'Courier New'  ← INHERITED       │    │
│  │  color: #c8c8d8              ← INHERITED       │    │
│  │  margin: 0                   ← NOT inherited   │    │
│  └───────────────────────────────────────────────┘    │
│                                                       │
│  ┌─── button ────────────────────────────────────┐    │
│  │  font-family: system-ui  ← NOT inherited       │    │
│  │    (browser UA stylesheet sets it explicitly)  │    │
│  │  ← This is why font: inherit matters           │    │
│  └───────────────────────────────────────────────┘    │
└───────────────────────────────────────────────────────┘
```

The `button` case is why `font: inherit` matters in the reset. Browsers set `font-family` explicitly on form elements in their user-agent stylesheet, which overrides the inherited value. `font: inherit` is a shorthand that sets every font sub-property to `inherit`, which beats the UA stylesheet.

### The Inline Image Gap

Images are `inline` by default. That means they sit on the text baseline, and below the baseline is the descender space — where letters like `g`, `p`, and `y` hang. Images have no descenders, but they still get that space:

```
Before reset (img is inline):

┌─────────────────────────────────────────────┐  ← container
│  ┌───────────────────────────────────┐       │
│  │                                   │       │
│  │            <img>                  │       │
│  │                                   │       │
│  └───────────────────────────────────┘       │
│  baseline ─────────────────────────────      │
│                              ↑ 4px gap       │
└─────────────────────────────────────────────┘

After reset (img { display: block }):

┌─────────────────────────────────────────────┐  ← container
│  ┌───────────────────────────────────┐       │
│  │                                   │       │
│  │            <img>                  │       │
│  │                                   │       │
│  └───────────────────────────────────┘       │
└─────────────────────────────────────────────┘
  Gap: gone.
```

This 4px gap causes confusing bugs in card components and image galleries. `display: block` on all media elements eliminates it permanently.

### Reset vs. Normalize Philosophy

```
reset.css approach:
┌──────────────────────────────────────────────────────┐
│  Before: browser defaults (messy, inconsistent)      │
│  After:  everything is zero                          │
│  Build:  add back everything deliberately            │
│                                                      │
│  Pro: total control, no hidden defaults              │
│  Con: you must style everything, including           │
│       <abbr>, <kbd>, <samp>, <var>, <mark>           │
└──────────────────────────────────────────────────────┘

normalize.css approach:
┌──────────────────────────────────────────────────────┐
│  Before: browser defaults (messy, inconsistent)      │
│  After:  browsers agree on the same defaults         │
│  Build:  style differences from the agreed baseline  │
│                                                      │
│  Pro: preserves useful defaults like <pre>, <code>   │
│  Con: 400 lines you may not understand               │
└──────────────────────────────────────────────────────┘

micro-reset.css (this chapter):
┌──────────────────────────────────────────────────────┐
│  Opinionated hybrid:                                 │
│  - Reset things that cause layout bugs               │
│    (box model, form fonts, image gap, list bullets)  │
│  - Normalize things that fix cross-browser gaps      │
│    (text-size-adjust, font-smoothing, focus-visible) │
│  - Ignore obscure elements you will never use        │
│  Result: ~40 lines you understand completely         │
└──────────────────────────────────────────────────────┘
```

The choice between them depends on context. A design system needs a full reset for total control. A documentation site benefits from normalize's preserved prose semantics. The micro-reset is the right choice for application UIs — the kind of thing the JavaScript book demos use.

## Guided Try It — Add a Print Reset

Web pages are often printed, and print stylesheets are consistently neglected. A print reset ensures your page looks presentable when someone hits Ctrl+P.

**Step 1**: Add a `@media print` block at the end of your reset. Start by hiding navigation and controls:

```css
@media print {
  nav,
  .no-print,
  button,
  [role="navigation"] {
    display: none !important;
  }
}
```

**Step 2**: Reset colors for print. Dark backgrounds use a lot of ink and often disappear on non-color printers:

```css
@media print {
  nav,
  .no-print,
  button,
  [role="navigation"] {
    display: none !important;
  }

  *,
  *::before,
  *::after {
    background: white !important;
    color: black !important;
    box-shadow: none !important;
    text-shadow: none !important;
  }
}
```

**Step 3**: Handle links. On paper, URLs are invisible. Print them after the link text using a pseudo-element:

```css
@media print {
  nav,
  .no-print,
  button,
  [role="navigation"] {
    display: none !important;
  }

  *,
  *::before,
  *::after {
    background: white !important;
    color: black !important;
    box-shadow: none !important;
    text-shadow: none !important;
  }

  a[href]::after {
    content: " (" attr(href) ")";
    font-size: 0.8em;
    opacity: 0.7;
  }

  /* Don't print URLs for internal anchors or JS pseudo-links */
  a[href^="#"]::after,
  a[href^="javascript:"]::after {
    content: "";
  }
}
```

The last two selectors suppress URL printing for internal anchor links (`#section`) and JavaScript pseudo-links, since those mean nothing on paper.

**Think about it**: The `!important` declarations in the print reset are unusual — the rest of the reset avoids `!important`. Why is it appropriate here? What does it tell you about when `!important` is a legitimate tool rather than a hack?

## Exercises

1. **The Specificity Calculator**: Write a JavaScript function in the browser console that takes a CSS selector string and returns its specificity score as `[ids, classes, types]`. Test it on: `a`, `.nav-link`, `#header .nav a:hover`, `input[type="checkbox"]`, `::before`.

2. **The Inheritance Inspector**: Without using DevTools, predict which of these properties inherit to a `<span>` inside a `<div>`: `color`, `background-color`, `font-size`, `border`, `display`, `opacity`, `cursor`, `pointer-events`. Then verify using DevTools. Write down any surprises and explain why they behave as they do.

3. **Form Element Font Fix**: Write a test page with five form elements: `<input type="text">`, `<input type="number">`, `<select>`, `<textarea>`, and `<button>`. Set the page's `font-family` to a clearly distinctive font like `'Georgia', serif`. Apply ONLY `font: inherit` to form elements and screenshot the result. Then apply the full v2 reset and screenshot again. Document what changes.

4. **Minimal Normalize**: Study the normalize.css philosophy. Identify 10 rules you consider most important for a modern project targeting only evergreen browsers (Chrome 90+, Firefox 88+, Safari 14+). Write those 10 rules with comments explaining what each fixes. Compare with the micro-reset — what does a full normalize do that the micro-reset intentionally omits?

5. **The Reset Audit**: Take a web page you have built. Open DevTools and look at the Styles panel for five different elements. For each element, find one rule from the user-agent stylesheet that your reset overrides. Document what the browser default is and what your reset changes it to. Then find one browser default you are NOT overriding — and decide whether you should be.

## Solutions

### Exercise 1 — Specificity Calculator

```javascript
function calculateSpecificity(selector) {
  let ids = 0, classes = 0, types = 0;

  // Work on a copy, stripping pseudo-element markers
  let s = selector.replace(/::[a-z-]+/gi, ' ');

  // Count and remove ID selectors
  s = s.replace(/#[a-zA-Z_-][a-zA-Z0-9_-]*/g, () => { ids++; return ' '; });

  // Count and remove class, attribute, and pseudo-class selectors
  s = s.replace(/\.[a-zA-Z_-][a-zA-Z0-9_-]*/g, () => { classes++; return ' '; });
  s = s.replace(/\[[^\]]+\]/g, () => { classes++; return ' '; });
  s = s.replace(/:[a-zA-Z-]+(\([^)]*\))?/g, () => { classes++; return ' '; });

  // Count remaining type selectors (letters only, not *, combinators, or spaces)
  const typeMatches = s.match(/\b[a-zA-Z][a-zA-Z0-9]*\b/g) || [];
  types = typeMatches.length;

  return [ids, classes, types];
}

// Test cases
console.log(calculateSpecificity('a'));
// → [0, 0, 1]

console.log(calculateSpecificity('.nav-link'));
// → [0, 1, 0]

console.log(calculateSpecificity('#header .nav a:hover'));
// → [1, 2, 1]

console.log(calculateSpecificity('input[type="checkbox"]'));
// → [0, 1, 1]

console.log(calculateSpecificity('::before'));
// → [0, 0, 0]  — pseudo-elements stripped before counting

// Compare two selectors
function specificityCmp(a, b) {
  const sa = calculateSpecificity(a);
  const sb = calculateSpecificity(b);
  for (let i = 0; i < 3; i++) {
    if (sa[i] !== sb[i]) return sb[i] - sa[i];
  }
  return 0; // equal
}

const selectors = ['a', '.nav-link', '#header .nav a:hover', '#main'];
console.log([...selectors].sort(specificityCmp));
// → ["#header .nav a:hover", "#main", ".nav-link", "a"]
```

### Exercise 2 — Inheritance Answers

```
Properties that INHERIT (text properties):
  color           ✓  parent color flows to all children
  font-size       ✓  em values computed relative to inherited size
  cursor          ✓  pointer cursor on parent means pointer on children

Properties that DO NOT INHERIT (box/layout properties):
  background-color  ✗  transparent by default, not inherited
  border            ✗  layout properties never inherit
  display           ✗  each element establishes its own display context

Surprise cases:
  opacity:         ✗  does NOT technically inherit via CSS cascade,
                      but affects all descendants through compositing;
                      you cannot give a child opacity: 1 to escape a
                      parent's opacity: 0.5

  pointer-events:  ✓  DOES inherit — children of pointer-events:none
                      are also non-interactive, unless you explicitly
                      set pointer-events: auto on them
```

### Exercise 3 — Form Element Font Fix

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <style>
    body {
      font-family: 'Georgia', serif;
      font-size: 18px;
      padding: 2rem;
      line-height: 1.6;
    }

    .panel {
      margin-bottom: 2rem;
      padding: 1.5rem;
      border: 1px solid #ccc;
    }

    label { display: block; margin-bottom: 0.5rem; color: #666; font-size: 0.9em; }
    .form-row { margin-bottom: 1rem; }

    /* Toggle this block to see the fix */
    .with-reset button,
    .with-reset input,
    .with-reset select,
    .with-reset textarea {
      font: inherit;
      display: block;
      margin-top: 0.25rem;
      padding: 0.4rem;
      border: 1px solid #ccc;
    }
  </style>
</head>
<body>
  <div class="panel">
    <h2>Without font: inherit</h2>
    <p>This text is in Georgia. The form elements below will use system UI font.</p>
    <div class="form-row">
      <label>Text input:</label>
      <input type="text" value="Type something here">
    </div>
    <div class="form-row">
      <label>Select:</label>
      <select><option>Select an option</option></select>
    </div>
    <div class="form-row">
      <label>Textarea:</label>
      <textarea rows="2">Textarea content</textarea>
    </div>
    <div class="form-row">
      <button>A button</button>
    </div>
  </div>

  <div class="panel with-reset">
    <h2>With font: inherit</h2>
    <p>This text is in Georgia. The form elements below will match.</p>
    <div class="form-row">
      <label>Text input:</label>
      <input type="text" value="Type something here">
    </div>
    <div class="form-row">
      <label>Select:</label>
      <select><option>Select an option</option></select>
    </div>
    <div class="form-row">
      <label>Textarea:</label>
      <textarea rows="2">Textarea content</textarea>
    </div>
    <div class="form-row">
      <button>A button</button>
    </div>
  </div>
</body>
</html>
```

The difference is immediate: without the reset, form elements render in San Francisco (macOS) or Segoe UI (Windows), visually breaking the typographic consistency of the page.

### Exercise 4 — Minimal Normalize for Evergreen Browsers

```css
/*
 * Minimal Normalize for Evergreen Browsers (Chrome 90+, Firefox 88+, Safari 14+)
 * 10 most important rules from the normalize philosophy
 */

/* 1. Prevent font inflation on iOS orientation change */
html {
  -webkit-text-size-adjust: 100%;
}

/* 2. Correct font weight for <b> and <strong> in Chrome/Safari */
b, strong {
  font-weight: bolder;
}

/* 3. Prevent font-size from changing in code blocks */
code, kbd, samp, pre {
  font-family: monospace, monospace;
  font-size: 1em;
}

/* 4. Fix <sub>/<sup> affecting line height */
sub, sup {
  font-size: 75%;
  line-height: 0;
  position: relative;
  vertical-align: baseline;
}
sub { bottom: -0.25em; }
sup { top: -0.5em; }

/* 5. Correct cursor on <abbr title="..."> */
abbr[title] {
  text-decoration: underline dotted;
  cursor: help;
}

/* 6. Prevent <hr> from having odd box-sizing in Firefox */
hr {
  box-sizing: content-box;
  height: 0;
  overflow: visible;
}

/* 7. Remove inner border/padding on Firefox buttons */
::-moz-focus-inner {
  border-style: none;
  padding: 0;
}

/* 8. Restore focus styles suppressed by rule 7 */
:-moz-focusring {
  outline: 1px dotted ButtonText;
}

/* 9. Correct color for <fieldset> in all browsers */
fieldset {
  border: 1px solid #a0a0a0;
  margin: 0 2px;
  padding: 0.35em 0.75em 0.625em;
}

/* 10. Fix legend display in Edge */
legend {
  box-sizing: border-box;
  color: inherit;
  display: table;
  max-width: 100%;
  white-space: normal;
}
```

The micro-reset handles rules 1 and 7–8 through broader strokes. Rules 2–6 and 9–10 cover specific elements that the micro-reset leaves to the project. A full normalize covers ~400 lines including Safari 4 bugs; these 10 rules cover the cases that still appear in 2024.

### Exercise 5 — Reset Audit Guide

Open DevTools on any page. In the Styles panel, grey/italic rules with "user agent stylesheet" labels are browser defaults. Crossed-out rules are overridden.

```
Common findings:

<body>:
  Browser default: margin: 8px
  Micro-reset:     margin: 0  (via * { margin: 0 })

<h1>:
  Browser default: font-size: 2em; font-weight: bold; margin-block: 0.67em
  Micro-reset:     font-size: inherit; font-weight: inherit; margin: 0

<ul>:
  Browser default: list-style-type: disc; padding-inline-start: 40px
  Micro-reset:     list-style: none; padding: 0

<a>:
  Browser default: color: -webkit-link; text-decoration: underline
  Micro-reset:     color: inherit; text-decoration: inherit

<button>:
  Browser default: font-family: -apple-system (varies by OS)
  Micro-reset:     font: inherit

NOT being overridden (and shouldn't be):
  <table>: display: table   ← correct, tables should still be tables
  <thead>: display: table-header-group ← structural, keep it
  <h1> in section: browser adjusts font-size to 1.5em
                   ← you are NOT overriding this if you used h1 { font-size: inherit }
                   ← the inherit means it now depends on YOUR body font-size
```

## What You Learned

| Concept | Key point |
|---------|-----------|
| `box-sizing: border-box` | Declared width equals rendered width; padding and border come out of the content area, not in addition to it |
| `box-sizing: content-box` | Browser default; padding and border are added to declared width, making layout arithmetic hard |
| Universal selector `*` | Specificity (0,0,0) — overridden by any real selector; safe for resets |
| Specificity scoring | ID=100, class/attribute/pseudo-class=10, type/pseudo-element=1; higher number wins |
| Inheritance | Typography properties inherit; layout properties do not; form elements block font inheritance |
| `font: inherit` on form elements | Browsers explicitly set font on inputs/buttons in the UA stylesheet, breaking inheritance; `font: inherit` overrides the UA rule |
| Inline image gap | `<img>` is inline by default and gets descender space (4px); `display: block` eliminates it |
| Reset philosophy | Remove all defaults, build up; maximum control, maximum work required |
| Normalize philosophy | Make browsers agree on the same defaults; preserve useful semantic styling |
| `:focus-visible` | Focus outline for keyboard users only, not mouse clicks; critical for accessibility |
| `overflow-wrap: break-word` | Prevents long URLs and unbroken strings from overflowing containers |
| `[hidden]` with `!important` | Enforces the HTML `hidden` attribute even when libraries accidentally set `display: block` |

## Building with Claude

**Bad prompt:**
> "Write me a CSS reset"

This returns either a copy of Eric Meyer's 2011 reset or a generic snippet that may not match your needs. You get no explanation of what each rule does or why.

**Good prompt:**
> "I'm building a web application that uses a custom design system with a monospace font and a dark color palette. Write a minimal CSS reset (under 50 lines) that: (1) sets box-sizing: border-box universally, (2) zeroes margin and padding on everything, (3) makes form elements inherit the page font via font: inherit, (4) removes list bullets and link underlines, (5) sets display: block on img/video, and (6) fixes the iOS text-size-adjust issue. For each rule, add a one-line comment explaining what browser behavior it corrects. Do not include resets for elements I am unlikely to use in application UI: abbr, acronym, blockquote decorations, table captions, fieldset borders."

The good prompt specifies constraints (under 50 lines), requirements (6 specific behaviors), format (inline comments), and exclusions (semantic HTML elements irrelevant to application UI). The result will be something you can read, understand, and maintain — not a cargo-culted block of CSS.
