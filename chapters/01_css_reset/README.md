# Chapter 1 — Build Your Own CSS Reset

Let's start at the very beginning. If you've never written HTML or CSS before, this chapter is for you.

This chapter does three beginner jobs at once:

- it explains what HTML is
- it explains what CSS is
- it shows why browsers do visual things before you write any CSS yourself

---

## What Is HTML?

HTML is the language that tells a browser *what is on a page*. Every webpage you've ever seen is built from HTML. Here's the simplest possible webpage:

```html
<!DOCTYPE html>
<html>
  <body>
    Hello, world!
  </body>
</html>
```

Save this as `hello.html` and open it in a browser. You'll see the words "Hello, world!" — that's it. You just wrote HTML.

HTML uses **tags** to mark up content. A tag is a word inside angle brackets:

```
<h1>This is a heading</h1>
```

- `<h1>` is the **opening tag** — it says "start of a heading"
- `</h1>` is the **closing tag** — it says "end of a heading"
- Everything between them is the content

There are tags for headings (`<h1>` through `<h6>`), paragraphs (`<p>`), links (`<a>`), images (`<img>`), buttons (`<button>`), and many more. Each tag has a meaning. `<h1>` means "most important heading". `<p>` means "paragraph". `<ul>` means "unordered list" (bullets).

---

## What Is CSS?

A helpful beginner mental model is:

- HTML creates boxes and meaning
- CSS changes how those boxes look and behave visually

CSS is the language that tells a browser *how things should look*. Without CSS, every webpage looks like a plain text document. With CSS, you can change colors, sizes, fonts, spacing — everything visual.

Here's how to add CSS to an HTML file, using a `<style>` tag:

```html
<!DOCTYPE html>
<html>
<head>
  <style>
    h1 {
      color: blue;
      font-size: 32px;
    }
  </style>
</head>
<body>
  <h1>Hello, world!</h1>
</body>
</html>
```

**Try it**: paste this into a file called `test.html` and open it. The heading should be blue.

A CSS rule has two parts:

```
h1 {
  color: blue;
}
│    └── declaration: property + value
└────── selector: which elements to style
```

- The **selector** (`h1`) picks which HTML elements to style
- The **declaration block** (`{ color: blue; }`) says what to do to them
- Each declaration is a `property: value;` pair

---

## How HTML and CSS Connect

There are three ways to connect CSS to HTML:

**1. Inline styles** (worst — only for quick tests):
```html
<h1 style="color: blue;">Hello</h1>
```

**2. A `<style>` tag** (good for single files — this is what we use in this book):
```html
<head>
  <style>
    h1 { color: blue; }
  </style>
</head>
```

**3. An external `.css` file** (best for real projects):
```html
<head>
  <link rel="stylesheet" href="styles.css">
</head>
```

In this book, every chapter is a single `.html` file with a `<style>` tag. You can open it directly in any browser — no server, no tools required.

---

## The Problem: Browser Defaults

This is one of the first big “aha” moments in CSS.

If something looks strange, it is not always because you wrote bad CSS. Sometimes it is because the browser already styled the element before you touched it.

Here's something surprising: before you write a single line of CSS, your page already has styles. Every browser ships with a **default stylesheet** — a set of CSS rules built into the browser itself.

These defaults exist so that raw HTML looks *somewhat* readable even without any CSS. For example, the browser's built-in rules make `<h1>` large and bold, `<a>` links blue and underlined, and `<ul>` lists indented with bullet points.

The problem is that these defaults are **inconsistent across browsers** and almost never what you actually want when building a real UI:

```
Chrome default for <h1>:
  font-size: 2em     (that's 32px if body is 16px)
  font-weight: bold
  margin-top: 0.67em
  margin-bottom: 0.67em

Firefox default for <h1>:
  font-size: 2em     (same)
  font-weight: bold
  margin-block-start: 0.67em  ← slightly different property name
  margin-block-end: 0.67em

Safari default for <button>:
  Uses the macOS native button style — looks nothing like Chrome's button
```

If you don't reset these defaults, your page will look different in different browsers, and every component you build has to fight the browser's assumptions.

**A CSS reset is the solution.** It's a small set of rules you put at the top of your stylesheet — before any other CSS — that clears out the browser defaults and establishes a clean, predictable baseline.

Think of it as clearing a whiteboard before you start drawing.

---

## Your First CSS Rule: Exploring What's There

Before we build the reset, let's see the browser defaults in action. Create this file and open it:

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Browser Defaults</title>
  <!-- No CSS at all! -->
</head>
<body>

  <h1>Heading Level 1</h1>
  <h2>Heading Level 2</h2>
  <h3>Heading Level 3</h3>

  <p>A paragraph of text. Notice the margin above and below.</p>

  <ul>
    <li>List item one</li>
    <li>List item two</li>
  </ul>

  <button>A Button</button>

</body>
</html>
```

Open this in your browser. You'll see:
- `<h1>` is big and bold
- `<p>` has space above and below it
- `<ul>` is indented with bullet points
- `<button>` looks like a native OS button

These are all coming from the browser's built-in stylesheet — you wrote zero CSS. Now we'll replace all of that with our own baseline.

In plain English: before your CSS starts, the browser already has opinions.

---

## Building It Step by Step

### v1 — The Most Important Rule: Box Sizing

The single most impactful reset rule is this one:

```css
*,
*::before,
*::after {
  box-sizing: border-box;
}
```

To understand why this matters, you need to know about the **box model**.

Every HTML element is a box. That box has four layers:

```
┌─────────────────────────────────────┐
│              MARGIN                 │  ← space outside the element
│  ┌───────────────────────────────┐  │
│  │            BORDER             │  │  ← the visible outline
│  │  ┌─────────────────────────┐  │  │
│  │  │         PADDING         │  │  │  ← space between border and content
│  │  │  ┌───────────────────┐  │  │  │
│  │  │  │      CONTENT      │  │  │  │  ← text, images, etc.
│  │  │  └───────────────────┘  │  │  │
│  │  └─────────────────────────┘  │  │
│  └───────────────────────────────┘  │
└─────────────────────────────────────┘
```

The browser default (`box-sizing: content-box`) means: when you set `width: 200px`, that's *only* the content area. Any padding or border you add makes the total element *wider* than 200px.

```css
/* content-box (browser default) — confusing: */
.box {
  width: 200px;
  padding: 20px;
  border: 5px solid black;
}
/* Actual rendered width: 200 + 20 + 20 + 5 + 5 = 250px — not 200! */
```

With `border-box` (what our reset sets), `width: 200px` means the *total* rendered width including padding and border:

```css
/* border-box (our reset) — predictable: */
.box {
  width: 200px;
  padding: 20px;    /* included in 200px */
  border: 5px solid;/* included in 200px */
}
/* Actual rendered width: 200px — exactly what you set */
```

**Try it**: create two divs, one with each box-sizing, set both to `width: 200px` with `padding: 20px`. Measure them — the content-box one will be wider.

The `*` selector means "every single element". `*::before` and `*::after` cover **pseudo-elements** — virtual elements that CSS can create (covered in later chapters). This one rule applies to everything on the page.

### v2 — Remove Default Spacing and Typography

Browsers add margin to headings, paragraphs, lists, and more. We remove it all:

```css
*,
*::before,
*::after {
  box-sizing: border-box;
  margin: 0;
  padding: 0;
}
```

Now *nothing* has default spacing. We'll add it back deliberately in our components.

Next, fix the headings. Browser defaults make `<h1>` through `<h6>` have different sizes and bold weight. We reset that too, so our type scale (Chapter 2) can set sizes deliberately:

```css
h1, h2, h3, h4, h5, h6 {
  font-size: inherit;    /* same size as body — we'll set these later */
  font-weight: inherit;  /* same weight as body — we'll set these later */
}
```

**What `inherit` means**: "use whatever value the parent element has." Since body has `font-size: 1rem` and `font-weight: 400`, all headings now default to those same values. They'll look the same as paragraph text until our type scale adds size and weight back.

And fix form elements. This is a commonly forgotten one:

```css
input, button, textarea, select {
  font: inherit;  /* use the page font, not the OS system font */
}
```

Without this rule, `<input>` and `<button>` use the operating system's default font (different on Windows, Mac, and Linux). `font: inherit` tells them to use the same font as the rest of the page.

**Common mistake**: forgetting `font: inherit` on form elements, then wondering why your button text looks different from your body text. This rule fixes it.

### v3 — Sensible Defaults

The reset isn't just removing things — it also adds sensible new baselines:

```css
/* ── Box sizing ──────────────────────────────────── */
*,
*::before,
*::after {
  box-sizing: border-box;
  margin: 0;
  padding: 0;
}

/* ── Root defaults ───────────────────────────────── */
:root {
  /* Prevents mobile browsers from zooming text on rotation */
  -webkit-text-size-adjust: 100%;
}

/* ── Body defaults ───────────────────────────────── */
body {
  /* body always fills the whole viewport, even if page is short */
  min-height: 100vh;

  /* comfortable line spacing for all text — 1.5 means 1.5× the font size */
  line-height: 1.5;

  /* makes text crisper on Mac/iPhone screens */
  -webkit-font-smoothing: antialiased;
}

/* ── Typography ──────────────────────────────────── */
h1, h2, h3, h4, h5, h6 {
  font-size: inherit;
  font-weight: inherit;
}

/* ── Lists ───────────────────────────────────────── */
/* Only remove bullets from lists that have role="list"
   (see the Walkthrough for why) */
ul[role="list"],
ol[role="list"] {
  list-style: none;
}

/* ── Images and media ────────────────────────────── */
img, picture, video, canvas, svg {
  /* block removes the mysterious gap below inline images */
  display: block;
  /* images never overflow their container */
  max-width: 100%;
}

/* ── Form elements ───────────────────────────────── */
input, button, textarea, select {
  font: inherit;
}

/* ── Overflow ────────────────────────────────────── */
p, h1, h2, h3, h4, h5, h6 {
  /* long words won't break the layout horizontally */
  overflow-wrap: break-word;
}
```

This is the complete reset. Every rule here has a specific reason. The walkthrough explains each one.

---

## The Complete Program

`css-reset.html` — open it in your browser. Click "Toggle Reset" to see what each rule changes. Click a category to filter the rules and see what they do.

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

:root { -webkit-text-size-adjust: 100%; }

body  {
  min-height: 100vh;
  line-height: 1.5;
  -webkit-font-smoothing: antialiased;
}

h1, h2, h3, h4, h5, h6 {
  font-size: inherit;
  font-weight: inherit;
}

ul[role="list"], ol[role="list"] { list-style: none; }

img, picture, video, canvas, svg {
  display: block;
  max-width: 100%;
}

input, button, textarea, select { font: inherit; }

p, h1, h2, h3, h4, h5, h6 { overflow-wrap: break-word; }
</style>
</body>
</html>
```

---

## Walkthrough

### Selectors — How CSS Picks Elements

You've seen `h1 { }` — that's a **type selector**: it matches every `<h1>` element on the page.

There are several kinds of selectors:

```css
/* Type selector — matches the element by its tag name */
h1 { color: red; }       /* all <h1> elements */
p  { color: blue; }      /* all <p> elements */

/* Class selector — matches elements with class="..." */
.button { background: blue; }    /* <button class="button"> */
.big    { font-size: 2rem; }     /* any element with class="big" */

/* ID selector — matches the one element with id="..." */
#header { position: sticky; }   /* <div id="header"> */

/* Universal selector — matches every element */
* { box-sizing: border-box; }

/* Combining selectors */
h1, h2, h3 { font-weight: bold; }   /* commas = "and also" */
.card p    { color: gray; }         /* space = "inside" — <p> inside .card */
```

You'll mostly use **type selectors** and **class selectors**. ID selectors (`#`) are powerful but hard to override (more on that below).

### Specificity — When Two Rules Conflict

What happens when two CSS rules target the same element?

```css
h1 { color: red; }
h1 { color: blue; }
```

The later rule wins — that's **source order**. But what about:

```css
h1     { color: red; }   /* type selector */
.title { color: blue; }  /* class selector */
```

```html
<h1 class="title">Which color?</h1>
```

The answer is **blue** — even if `h1` came first. That's because class selectors have higher **specificity** than type selectors. Specificity is a score that determines which rule wins:

```
Selector type          Score
──────────────────────────────────────────────────────
Type selector (h1)     (0, 0, 1)  ← lowest
Class selector (.btn)  (0, 1, 0)  ← beats type
ID selector (#header)  (1, 0, 0)  ← beats class
Inline style           (ignore)   ← beats everything
```

Higher score wins. When scores tie, the later rule wins.

**Why our reset uses `*`**: the universal selector has zero specificity — score `(0, 0, 0)`. That means *any* rule you write later can override the reset. This is intentional: the reset is a floor, not a ceiling.

**Common mistake**: writing `#myId { color: red; }` and wondering why a later `.class { color: blue; }` doesn't work. IDs beat classes. Use classes.

### The Cascade — The "C" in CSS

"Cascading" means rules flow down from multiple sources. The browser combines them in this order:

```
1. Browser defaults (lowest priority)
       ↓
2. Your stylesheet (author styles)
       ↓
3. Inline styles (style="...")
       ↓
4. !important declarations (highest — use sparingly)
```

Your reset sits at level 2 and wins over browser defaults (level 1). Any of your component styles also sit at level 2, so specificity and source order decide between them.

### Inheritance — What Passes Down

Some CSS properties automatically pass from a parent element to its children. This is called **inheritance**:

```html
<body style="color: darkblue; font-family: Georgia, serif;">
  <p>This text is darkblue in Georgia.</p>
  <div>
    <span>This too — it inherits from body.</span>
  </div>
</body>
```

Properties that **do** inherit (they pass to children):
- `color`, `font-family`, `font-size`, `font-weight`, `line-height`
- `text-align`, `letter-spacing`, `word-spacing`

Properties that **do not** inherit (each element sets its own):
- `background`, `border`, `margin`, `padding`
- `width`, `height`, `display`, `position`

**Why `font: inherit` on form elements?** `font-family` and `font-size` are inherited — they *should* reach `<input>` and `<button>`. But the browser's default stylesheet overrides them with the OS system font. `font: inherit` forces the browser's override away and lets inheritance work normally.

### Why `display: block` on Images

Here's a strange browser behavior worth understanding. In HTML, elements are either **block** or **inline** by default:

- **Block elements** (`<div>`, `<p>`, `<h1>`) take up the full width and stack vertically
- **Inline elements** (`<span>`, `<a>`, `<img>`) flow with text, side by side

Images (`<img>`) are inline by default. Inline elements sit on a **text baseline** — the invisible line that text rests on. Below the baseline, there's a small gap reserved for text descenders (the tails on letters like `g`, `p`, `y`).

This gap appears below inline images as mysterious white space:

```
Without reset:                  With display: block:
┌────────────────┐              ┌────────────────┐
│                │              │                │
│     image      │              │     image      │
│                │              │                │
└────────────────┘              └────────────────┘
            ↑ gap here          (gap is gone)
```

Setting `display: block` removes the image from inline text flow. The gap disappears.

**Try it**: put an `<img>` inside a `<div>` with a background color. Without `display: block`, you'll see a few pixels of the background below the image. With `display: block`, it's gone.

### The List Accessibility Rule

```css
ul[role="list"],
ol[role="list"] {
  list-style: none;
}
```

Why not just `ul { list-style: none; }`?

The answer involves screen readers — software that reads webpages aloud for users with visual impairments. When VoiceOver (macOS/iOS's screen reader) encounters a `<ul>` styled with `list-style: none`, it stops announcing it *as a list*. From VoiceOver's perspective, if there are no bullets, it's not a list.

The `[role="list"]` attribute tells both the browser and assistive technology: "I'm deliberately making this look non-list-like, but it *is* a list semantically." With this attribute present, VoiceOver continues to announce it correctly.

Rule of thumb:
- Navigation menus, tags, card grids → add `role="list"`, then remove bullets
- Article content, instructions → leave the `<ul>` alone; keep its bullets

---

## Guided Try It — Add a Focus Ring

**The goal**: add a visible focus indicator so keyboard users can see which element is currently selected.

**Why this is important**: not everyone uses a mouse. Many users navigate with the keyboard (Tab to move, Enter to activate). Without a visible focus indicator, they have no idea where they are on the page. Removing `:focus` outlines is one of the most common accessibility mistakes.

**What is `:focus`?** It's a **pseudo-class** — a selector that matches an element based on its *state*, not its type or class. `:focus` matches whichever element currently has keyboard focus.

**Step 1 — See the browser default first**

```html
<button>Click me</button>
<a href="#">A link</a>
<input type="text" placeholder="Type here">
```

Press Tab on your keyboard to move focus between them. See the browser's default focus ring (usually a blue outline or a dotted border).

**Step 2 — The wrong way (never do this)**

```css
/* DANGER: never do this — it removes focus for keyboard users */
* { outline: none; }
:focus { outline: none; }
```

Some websites remove the focus ring because it looks "ugly". This makes the site unusable for keyboard-only users.

**Step 3 — The right way**

```css
/* Keep focus ring for keyboard navigation, hide it for mouse clicks */
:focus:not(:focus-visible) {
  outline: none;
}

/* Show a clear, styled focus ring when navigating by keyboard */
:focus-visible {
  outline: 2px solid #005fcc;
  outline-offset: 2px;
}
```

`:focus-visible` is a smarter version of `:focus`. The browser shows it when the user *navigates with the keyboard* but hides it when they click with the mouse. Best of both worlds.

**Try it**: tab through some elements after adding this rule. The focus ring shows while keyboard-navigating but not when clicking.

---

## Exercises

1. **Explore browser defaults**: Create an HTML file with one each of `<h1>`, `<h2>`, `<h3>`, `<p>`, `<ul>`, `<button>`, and `<input>`. Open it with no CSS. Use your browser's DevTools (F12 → Inspect → Computed tab) to see what `font-size`, `margin`, and `padding` the browser is applying to each element. Write down three defaults that surprised you.

2. **Feel the box model**: Create a `<div>` with `width: 200px`, `padding: 30px`, and `border: 10px solid black`. First use the browser default `box-sizing: content-box` — measure its actual width in DevTools. Then add `box-sizing: border-box` — measure again. Write down the difference and explain why.

3. **Add table reset rules**: Tables (`<table>`) have two browser defaults that cause problems: `border-collapse: separate` (which creates double borders between cells) and `border-spacing: 2px`. Add rules to the reset that set `border-collapse: collapse` and `border-spacing: 0`. Create a test table with borders to see the difference.

4. **Add a button reset**: The `<button>` element still looks like an OS native button after our reset because we didn't reset `appearance`. Add `.btn-reset { appearance: none; background: transparent; border: none; cursor: pointer; }`. Now style a button that looks like a link using this reset class.

5. **Build your own opinionated reset**: Start with the reset from v3. Add five more rules that set defaults *you personally prefer* — things like a base font-family, a `line-height` you like, or a sensible default `color`. Comment each rule explaining your choice. This becomes your personal starting point for every project.

---

## Solutions

### Exercise 1 — Explore browser defaults

Example findings (Chrome, macOS):

```
h1:  font-size: 32px, font-weight: bold, margin-top: 21px, margin-bottom: 21px
h2:  font-size: 24px, font-weight: bold, margin-top: 19px
p:   font-size: 16px, margin-top: 16px, margin-bottom: 16px
ul:  padding-left: 40px, list-style-type: disc
button: font-family: -apple-system (not body font!), padding: 1px 6px
input: font-family: -apple-system, border: 2px inset
```

The most surprising for most beginners: `<button>` and `<input>` use a completely different font family than `<p>`. That's why `font: inherit` is in the reset.

### Exercise 2 — Box model

```html
<div id="box" style="width:200px; padding:30px; border:10px solid black">
  I'm a box
</div>
```

Without `box-sizing: border-box`:
```
Actual width = 200 (content) + 30 (left pad) + 30 (right pad)
             + 10 (left border) + 10 (right border)
             = 280px
```

With `box-sizing: border-box`:
```
Actual width = 200px (border and padding are subtracted from content area)
Content area = 200 - 30 - 30 - 10 - 10 = 120px wide
But the outer box = exactly 200px
```

### Exercise 3 — Table reset

```css
table {
  border-collapse: collapse;
  border-spacing: 0;
}
```

Test:
```html
<table style="border: 1px solid black">
  <tr>
    <td style="border: 1px solid black; padding: 8px">Cell A</td>
    <td style="border: 1px solid black; padding: 8px">Cell B</td>
  </tr>
</table>
```

Without reset: double border between cells (each cell's border + adjacent cell's border = 2px gap).
With reset: single 1px border between cells.

### Exercise 4 — Button reset

```css
.btn-reset {
  appearance: none;        /* remove OS native styling */
  -webkit-appearance: none;/* same, for older Safari */
  background: transparent; /* no background */
  border: none;            /* no border */
  cursor: pointer;         /* hand cursor on hover */
  color: inherit;          /* use page text color */
  font: inherit;           /* use page font */
}

/* Now style it as a link */
.link-button {
  color: #005fcc;
  text-decoration: underline;
  padding: 0;
}
.link-button:hover { color: #0036a3; }
```

```html
<button class="btn-reset link-button">This looks like a link</button>
```

### Exercise 5 — Opinionated reset

```css
/* Opinionated reset — personal preferences */

*, *::before, *::after {
  box-sizing: border-box;
  margin: 0;
  padding: 0;
}

:root {
  /* I prefer a system font stack — it looks native on every OS */
  font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto,
               sans-serif;
  font-size: 100%;           /* respect browser font size preference */
  color-scheme: light dark;  /* support dark mode in browser UI */
}

body {
  font-size: 1rem;
  line-height: 1.6;          /* I find 1.6 more comfortable than 1.5 */
  color: hsl(220 14% 10%);   /* near-black, not pure black */
  background: hsl(220 14% 97%); /* off-white, not pure white */
  min-height: 100vh;
  -webkit-font-smoothing: antialiased;
}

/* Comfortable heading sizes — Minor Third scale (1.2) */
h1 { font-size: 1.728rem; font-weight: 700; line-height: 1.2; }
h2 { font-size: 1.440rem; font-weight: 600; line-height: 1.25; }
h3 { font-size: 1.200rem; font-weight: 600; line-height: 1.3; }

h1, h2, h3, h4, h5, h6 { margin-bottom: 0.5em; }
p { margin-bottom: 1em; }
p:last-child { margin-bottom: 0; }  /* no extra space after the last paragraph */

img, picture, video, canvas, svg { display: block; max-width: 100%; }
input, button, textarea, select   { font: inherit; }

:focus-visible { outline: 2px solid hsl(217 91% 50%); outline-offset: 2px; }
```

---

## What You Learned

| Concept | Key point |
|---------|-----------|
| HTML tag | Text inside `<>` that marks up content. Opening `<p>` + closing `</p>` wraps content. |
| CSS rule | selector `{ property: value; }` — picks elements and styles them |
| Browser defaults | Every browser has built-in CSS; resets remove it for a clean slate |
| Box model | margin → border → padding → content; four layers around every element |
| `box-sizing: border-box` | Makes `width` include padding and border — predictable layout math |
| `*` selector | Matches every element; zero specificity — anything overrides it |
| Specificity | Type (0,0,1) < class (0,1,0) < ID (1,0,0) < inline |
| Cascade | Browser → author → inline → `!important`; your rules beat browser defaults |
| Inheritance | `color`, `font-*` pass to children; `margin`, `padding`, `background` do not |
| `font: inherit` | Form elements override font inheritance; this restores it |
| `display: block` on `<img>` | Removes the invisible gap below inline images |
| `:focus-visible` | Focus ring for keyboard users; hidden for mouse clicks — always include this |

---

## Building with Claude

Bad prompt:
> "How do I style my website?"

Good prompt:
> "I'm building a CSS reset in a single `.html` file. I set `box-sizing: border-box` on `*` and `font: inherit` on form elements, but my `<button>` in Safari still uses the OS font rather than my body font. My `<body>` has `font-family: 'Inter', sans-serif`. Is `font: inherit` not enough for Safari buttons, or is something in my specificity overriding it? Here's my current rule: `input, button, textarea, select { font: inherit; }`"

The good prompt shows your exact code, names the browser where the problem occurs, and asks one specific question. You'll get a targeted answer, not a tutorial.
