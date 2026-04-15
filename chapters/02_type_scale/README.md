# Chapter 2 — Build Your Own Type Scale

Text is usually the first thing people notice on a page. If the text is too small, too large, or inconsistent, the whole page feels wrong. In this chapter, you will build a simple type scale: a small set of text sizes that work well together. The goal is not fancy typography. The goal is to stop guessing and start using a system.

If you are a beginner, keep the chapter goal simple: make headings, body text, and small text feel related instead of random.

---

## Before We Start: What Is `font-size`?

Open the HTML file from Chapter 1 and add one line of CSS:

```html
<style>
  p {
    font-size: 24px;
  }
</style>
```

That's a font size. `px` means **pixels** — screen pixels. On a typical monitor, `16px` is the default browser font size for body text. At that size, most people can read comfortably. `24px` is 50% bigger.

**Try it now**: write a `<p>` tag with some text. Change `font-size` between `10px`, `16px`, `24px`, and `48px`. See how the text grows.

### What's wrong with `px` for fonts?

Pixels feel straightforward. But they have a serious problem.

Most browsers have an **accessibility setting** that lets users make all text larger. Someone with low vision might set their default font size to `20px` instead of `16px`. When you use `font-size: 16px`, that user's preference is *ignored* — the text stays at 16px no matter what. You've overridden their accessibility setting.

```
Browser default font size setting: 20px  (user set this)
Your CSS: font-size: 16px  ← ignores user preference
Result: text is 16px — user can't read it comfortably
```

### The solution: `rem`

A good beginner way to think about `rem` is: it is a size that respects the browser’s default text size instead of hard-forcing your own.

`rem` stands for **root em**. It means "relative to the root font size" — the `<html>` element's font size, which is the browser's default (typically 16px).

```
Browser default font size: 16px

1rem   = 16px   (same as default)
1.5rem = 24px   (1.5 × 16)
2rem   = 32px   (2 × 16)
0.75rem= 12px   (0.75 × 16)
```

With `rem`, if a user sets their browser default to `20px`:

```
Browser font size: 20px

1rem   = 20px   (scales up automatically!)
1.5rem = 30px
2rem   = 40px
```

Your text scales with their preference. That's accessibility.

---

## The Problem

The deeper lesson here is that visual systems are easier to manage than isolated values. Type scale is your first example of that.

Beginners often write CSS like this:

```css
h1 { font-size: 36px; }
h2 { font-size: 28px; }
h3 { font-size: 22px; }
p  { font-size: 16px; }
```

Three problems show up quickly:

1. **The numbers are guesses**. Why `36px`? Why not `32px`? There is no rule behind it.
2. **It does not adapt well**. A size that looks fine on your laptop may feel too large on a phone.
3. **Fixed pixels ignore user preferences**. If someone increases their browser text size for readability, fixed pixel values fight against that.

A **type scale** solves this by giving your text sizes a simple relationship. Instead of picking every size by feel, you start with one base size and grow from there:

```
Start with: base = 1rem (scales with browser preference)
            ratio = 1.25 (each step is 25% bigger)

Sizes going up:
  1rem × 1.25⁰ = 1.000rem  ← body text
  1rem × 1.25¹ = 1.250rem  ← slightly larger
  1rem × 1.25² = 1.563rem  ← medium heading
  1rem × 1.25³ = 1.953rem  ← large heading
  1rem × 1.25⁵ = 3.052rem  ← hero heading

Sizes going down:
  1rem ÷ 1.25¹ = 0.800rem  ← small text
  1rem ÷ 1.25² = 0.640rem  ← captions
```

The sizes have a built-in relationship — they're harmonious rather than arbitrary.

---

## Building It Step by Step

### v1 — A Simple Static Scale

Let's start with the simplest version: a set of size variables using `rem`.

First, a new concept: **CSS custom properties** (also called CSS variables). They let you name a value and reuse it:

```css
:root {
  --text-base: 1rem;       /* define the variable */
  --text-lg:   1.25rem;
  --text-xl:   1.563rem;
}

p  { font-size: var(--text-base); }   /* use the variable */
h2 { font-size: var(--text-xl); }
```

`--text-base` is the variable name (always starts with `--`). `var(--text-base)` reads its value.

`var()` syntax: `var(--variable-name)`.

`:root` is a selector that matches the `<html>` element — it's the highest point in the CSS inheritance tree. Custom properties defined on `:root` are available everywhere on the page.

Here's the full static scale:

```css
/* v1: a static type scale using CSS custom properties and rem */
:root {
  --text-xs:   0.64rem;    /* extra small: captions, fine print */
  --text-sm:   0.80rem;    /* small: secondary text */
  --text-base: 1rem;       /* base: body text */
  --text-lg:   1.25rem;    /* large: lead text */
  --text-xl:   1.563rem;   /* extra large: h3-level headings */
  --text-2xl:  1.953rem;   /* 2x large: h2-level headings */
  --text-3xl:  3.052rem;   /* 3x large: h1, hero headings */
}
```

These numbers come from repeatedly multiplying `1rem` by `1.25` (the ratio).

**Try it**: create a heading at `var(--text-3xl)` and body text at `var(--text-base)`. The relationship between them should feel balanced — not too dramatic, not too subtle.

### v2 — Make It Fluid with `clamp()`

`clamp()` looks scary the first time you see it, but the idea is simple: never go below this size, prefer this flexible size, and never go above that size.

The static scale looks good on one screen width. But on a phone, `3.052rem` (about 49px) for a heading is too large. On a wide monitor, it might feel small.

`clamp()` makes a value fluid — it slides between a minimum and maximum as the viewport changes:

```
clamp(MINIMUM, PREFERRED, MAXIMUM)
```

- Below the minimum: value stays at MINIMUM
- Above the maximum: value stays at MAXIMUM
- In between: value follows the PREFERRED formula

```css
/* v2: fluid sizing with clamp() */
:root {
  --text-base: clamp(1rem,  0.95rem + 0.25vw,  1.125rem);
  --text-xl:   clamp(1.25rem, 1rem  + 1vw,     1.75rem);
  --text-3xl:  clamp(2rem,  1rem    + 4vw,     3.5rem);
}
```

What is `vw`? It means **1% of the viewport width**. So `4vw` at a 400px viewport = 16px. At a 1200px viewport, `4vw` = 48px.

**Reading `clamp(2rem, 1rem + 4vw, 3.5rem)` out loud**:
- "Minimum size is 2rem (about 32px)"
- "Preferred: 1rem plus 4% of the viewport width — this grows with the screen"
- "Maximum size is 3.5rem (about 56px)"

At a 320px mobile screen: `1rem + 4vw = 16px + 12.8px ≈ 29px` → clamped to `32px` (minimum)
At a 1200px desktop: `1rem + 4vw = 16px + 48px = 64px` → clamped to `56px` (maximum)

**Try it**: open `type-scale.html` and drag the viewport width slider. Watch the sizes change fluidly.

### v3 — Add Line Heights and Letter Spacing

Font size alone isn't enough. A complete type system also controls:

- **Line height**: the space between lines of text
- **Letter spacing**: the space between individual letters
- **Font weight**: how bold the text is

```css
/* v3: complete type system */
:root {
  /* ── Sizes ────────────────────────────────────── */
  --text-xs:   clamp(0.64rem,  0.60rem + 0.2vw,  0.75rem);
  --text-sm:   clamp(0.80rem,  0.75rem + 0.25vw, 0.90rem);
  --text-base: clamp(1rem,     0.95rem + 0.25vw, 1.125rem);
  --text-lg:   clamp(1.125rem, 1rem    + 0.5vw,  1.375rem);
  --text-xl:   clamp(1.25rem,  1rem    + 1vw,    1.75rem);
  --text-2xl:  clamp(1.5rem,   1rem    + 2vw,    2.25rem);
  --text-3xl:  clamp(2rem,     1rem    + 4vw,    3.5rem);

  /* ── Line heights ──────────────────────────────── */
  --leading-tight:   1.25;   /* for headings */
  --leading-normal:  1.5;    /* for body text */
  --leading-relaxed: 1.625;  /* for long reading */

  /* ── Letter spacing ─────────────────────────────── */
  --tracking-tight:  -0.03em;  /* tighter: large headings */
  --tracking-normal:  0em;     /* default */
  --tracking-wide:    0.05em;  /* looser: small labels */

  /* ── Font weights ────────────────────────────────── */
  --font-normal:  400;
  --font-medium:  500;
  --font-semibold:600;
  --font-bold:    700;
}
```

Applying the system:

```css
h1 {
  font-size:      var(--text-3xl);
  font-weight:    var(--font-bold);
  line-height:    var(--leading-tight);
  letter-spacing: var(--tracking-tight);
}

p {
  font-size:   var(--text-base);
  font-weight: var(--font-normal);
  line-height: var(--leading-normal);
}

.label {
  font-size:      var(--text-sm);
  font-weight:    var(--font-semibold);
  letter-spacing: var(--tracking-wide);
  text-transform: uppercase;
}
```

**Try it**: apply these to a page with a heading, a paragraph, and a label. Adjust the line heights — notice how `1.25` feels cramped for body text but right for headings, and `1.625` feels spacious.

---

## The Complete Program

`type-scale.html` — open in your browser. Three views:
- **Tokens**: see all sizes at the current viewport width
- **Specimen**: see each size in context (hero, heading, body, caption)
- **Scale**: compare the sizes visually as bars

Drag the "Root" slider to simulate a user with a larger browser font size — watch all sizes scale proportionally.

---

## Walkthrough

### `rem` vs `em` vs `px`

Three units, three reference points:

```
px  = absolute pixels
      font-size: 16px  → always 16 pixels
      Doesn't scale with browser preferences. Avoid for font sizes.

em  = relative to the CURRENT element's font-size
      If body is 16px, and p has font-size: 1.5em → p is 24px
      If that p has a span with font-size: 1.5em → span is 36px (compounds!)
      Cascades and compounds. Useful for padding/margin on components,
      but dangerous for font-size nesting.

rem = relative to the ROOT font-size (the <html> element)
      If browser default is 16px:
        1.5rem = 24px  (always, no matter how deep in the HTML)
      If user sets browser to 20px:
        1.5rem = 30px  (scales automatically!)
      Doesn't compound. Best choice for font sizes.
```

**Rule of thumb**: use `rem` for font sizes. Use `em` for spacing that should scale with the element's font (like `padding: 0.5em` on a button — the padding grows with the button's text). Use `px` sparingly, only for things that should never scale (like a 1px border).

### How `clamp()` Works

```
clamp(MIN, PREFERRED, MAX)

     MIN = the smallest the value ever gets
MAX = the largest the value ever gets
     PREFERRED = how it grows in between

When viewport is narrow:   value = MAX(MIN, PREFERRED) → usually MIN
When viewport is wide:     value = MIN(MAX, PREFERRED) → usually MAX
In between:                value = PREFERRED (grows linearly)
```

The PREFERRED is usually written as `Xrem + Yvw` — a fixed base plus a fraction of the viewport:

```css
clamp(1rem,  0.95rem + 0.25vw,  1.125rem)
      │      │         │         │
      │      └─ fixed  └─ grows  │
      │        base    w/ screen └─ maximum
      └─ minimum
```

At `320px` wide:  `0.95rem + 0.25 × 3.2px = 0.95rem + 0.8px ≈ 1rem`  → min (1rem)
At `1200px` wide: `0.95rem + 0.25 × 12px  = 0.95rem + 3px ≈ 1.14rem` → between min and max
At `1600px` wide: `0.95rem + 0.25 × 16px  = 0.95rem + 4px ≈ 1.2rem`  → max (1.125rem)

### Modular Scale — The Math

A **modular scale** is a geometric sequence — each step is the previous one multiplied by a fixed ratio.

Why does this produce harmonious sizes? Because the sizes have a mathematical relationship to each other. Music theory uses the same idea: notes that are pleasing together share simple frequency ratios.

Common ratios for type scales:

```
Ratio  Name              Effect
─────────────────────────────────────────────────────────────
1.067  Minor Second      Very subtle — barely any contrast
1.125  Major Second      Gentle — good for dense UIs
1.200  Minor Third       Moderate — works well in tight spaces
1.250  Major Third       Standard — the default in this book
1.333  Perfect Fourth    Good contrast — headings stand out
1.414  Augmented Fourth  Dramatic — bold, editorial feel
1.500  Perfect Fifth     Very dramatic — large displays
1.618  Golden Ratio      Extreme — headlines vs. body very different
```

To compute the scale at ratio 1.25:
```
Base:   1rem
× 1.25: 1.250rem    → --text-lg
× 1.25: 1.563rem    → --text-xl  (1.250 × 1.25)
× 1.25: 1.953rem    → --text-2xl (1.563 × 1.25)
× 1.25: 2.441rem    (skip — too close to 3xl)
× 1.25: 3.052rem    → --text-3xl (2.441 × 1.25)

Going down:
÷ 1.25: 0.800rem    → --text-sm  (1 ÷ 1.25)
÷ 1.25: 0.640rem    → --text-xs  (0.800 ÷ 1.25)
```

### Line Height — No Units

Line height controls space between lines. The value `1.5` means "1.5 times the current font size":

```css
p {
  font-size: 1rem;    /* 16px */
  line-height: 1.5;   /* 24px — 1.5 × 16px */
}

h1 {
  font-size: 3rem;    /* 48px */
  line-height: 1.25;  /* 60px — 1.25 × 48px */
}
```

Notice: **no unit on the line-height number**. `line-height: 1.5` (unitless) vs. `line-height: 1.5rem` (with unit) is a significant difference:

- `line-height: 1.5` → 1.5× the *current* font-size (scales with font)
- `line-height: 1.5rem` → 1.5× the *root* font-size (doesn't scale)

For text, you almost always want the unitless version. Large text should have large line spacing; small text should have proportionally smaller spacing.

**Optimal ranges**:
- `1.1–1.3` for large headings (few words, big size — tight feels intentional)
- `1.4–1.6` for body copy (comfortable reading at paragraph length)
- `1.6–2.0` for dense technical text or small sizes

### Letter Spacing in `em`

Letter spacing (`letter-spacing` property) uses `em` — relative to the *current* font size:

```css
h1 { letter-spacing: -0.03em; }
/* On a 48px heading: -0.03 × 48 = -1.44px tighter per letter */

.badge { letter-spacing: 0.08em; }
/* On a 12px badge: 0.08 × 12 = 0.96px wider per letter */
```

Using `em` means the absolute spacing is always proportional to the font size. If you wrote `letter-spacing: -2px`, it would be way too tight on small text and barely noticeable on a giant heading. With `em`, it scales correctly.

**Common patterns**:
- Large headings often need `-0.02em` to `-0.04em` (tighter) — they look too spread-out at big sizes
- Body text needs `0em` — don't touch it
- All-caps labels need `0.05em` to `0.1em` (wider) — all-caps words look cramped without extra spacing

---

## Guided Try It — Add a Monospace Scale for Code

**The goal**: add a separate set of size tokens for `<code>` and `<pre>` elements. Monospace fonts render slightly larger visually than proportional fonts at the same pixel size, so code text should be about 90% of the corresponding regular size.

**Why this is useful**: every technical blog, documentation site, and developer tool needs to style code blocks. Getting the size right makes code readable without dominating the surrounding text.

**Step 1 — Add code size tokens**

```css
:root {
  /* Code sizes: ~90% of the corresponding text size */
  --code-sm:   calc(var(--text-sm)   * 0.9);
  --code-base: calc(var(--text-base) * 0.9);
  --code-lg:   calc(var(--text-lg)   * 0.9);
}
```

`calc()` is a CSS function that does math. `calc(var(--text-base) * 0.9)` means "90% of the text-base size". You can mix units: `calc(1rem + 4px)` works.

**Step 2 — Apply to code elements**

```css
code, kbd {
  font-family: 'Courier New', Courier, monospace;
  font-size: var(--code-base);
  background: hsl(220 14% 93%);
  padding: 0.15em 0.35em;
  border-radius: 3px;
}

pre {
  font-family: 'Courier New', Courier, monospace;
  font-size: var(--code-sm);   /* slightly smaller for code blocks */
  line-height: var(--leading-relaxed);
  padding: 1em;
  overflow-x: auto;
}
```

**Step 3 — Test it**

```html
<p>Use the <code>font-size</code> property to set text size.</p>

<pre>
:root {
  --text-base: 1rem;
}
</pre>
```

**Think about it**: why does `var(--code-base)` use `calc()` with `* 0.9` instead of just hardcoding a number like `0.9rem`? Because if someone changes `--text-base` (say, to make body text larger for an accessibility reason), the code font automatically scales with it. Hardcoding `0.9rem` would break that relationship.

---

## Exercises

1. **Convert a pixel design to rem**: You have a design spec with these pixel sizes: `h1: 40px`, `h2: 32px`, `h3: 24px`, `p: 16px`, `small: 13px`. Convert each to `rem` (assuming the browser default is 16px). Now write them as custom property tokens following the naming convention in this chapter.

2. **Build a fluid heading**: Use `clamp()` to make `--text-3xl` fluid between `2rem` at 320px viewport and `4rem` at 1400px viewport. Work out the formula: `clamp(2rem, Xrem + Yvw, 4rem)` by solving for X and Y. (Hint: at 320px, `X + Y×3.2 = 32px`; at 1400px, `X + Y×14 = 64px`.)

3. **Add a display size**: Add an eighth size, `--text-display`, for use in hero sections or landing page banners. It should be larger than `--text-3xl`. Use `clamp()` with a minimum of `3rem` and a maximum of `6rem`. Apply it to an `<h1>` on a demo page.

4. **Explore font stacks**: A font stack is a list of fonts as fallbacks: `font-family: 'Playfair Display', Georgia, serif`. If the first font isn't installed, the browser tries the next one. Create three font stacks: one for headlines (a serif or display font), one for body text (a sans-serif), and one for code (a monospace). Test each on a page of text.

5. **Build a complete type specimen**: Create an HTML page that demonstrates every token from the complete v3 system. For each size, show: the actual text at that size (use a sentence like "The quick brown fox"), the token name, and the computed pixel value at the default font size. Style it cleanly so it could serve as a design reference.

---

## Solutions

### Exercise 1 — Convert to rem

```
h1: 40px  ÷ 16px = 2.5rem   → --text-4xl (or add to existing scale)
h2: 32px  ÷ 16px = 2rem     → --text-3xl (close enough to 2.044rem)
h3: 24px  ÷ 16px = 1.5rem   → --text-2xl (close to 1.953rem)
p:  16px  ÷ 16px = 1rem     → --text-base
small: 13px ÷ 16px = 0.8125rem → --text-sm (close to 0.8rem)
```

```css
:root {
  --text-sm:   0.8125rem;   /* 13px */
  --text-base: 1rem;         /* 16px */
  --text-2xl:  1.5rem;       /* 24px */
  --text-3xl:  2rem;         /* 32px */
  --text-4xl:  2.5rem;       /* 40px */
}
```

### Exercise 2 — Fluid heading formula

Solve the system of equations:
```
At 320px:   X + Y × 3.2  = 32   (2rem = 32px at 16px root)
At 1400px:  X + Y × 14   = 64   (4rem = 64px)

Subtract:   Y × (14 - 3.2) = 64 - 32
            Y × 10.8 = 32
            Y ≈ 2.963

X = 32 - 2.963 × 3.2 ≈ 32 - 9.48 ≈ 22.52px ≈ 1.41rem

Result: clamp(2rem, 1.41rem + 2.963vw, 4rem)
```

### Exercise 3 — Display size

```css
:root {
  --text-display: clamp(3rem, 1rem + 8vw, 6rem);
}

.hero-title {
  font-size: var(--text-display);
  font-weight: 900;
  line-height: 1.05;
  letter-spacing: -0.04em;
}
```

At 320px:  `1rem + 8×3.2px = 1rem + 25.6px ≈ 2.6rem` → clamps to 3rem
At 1200px: `1rem + 8×12px = 1rem + 96px = 7rem` → clamps to 6rem

### Exercise 4 — Font stacks

```css
:root {
  /* Headline font: elegant serif with fallbacks */
  --font-headline: 'Playfair Display', 'Georgia', 'Times New Roman', serif;

  /* Body font: clean sans-serif, system-font based */
  --font-body: -apple-system, BlinkMacSystemFont, 'Segoe UI',
               'Roboto', 'Helvetica Neue', Arial, sans-serif;

  /* Code font: monospace with wide support */
  --font-code: 'JetBrains Mono', 'Fira Code', 'Courier New',
               Courier, monospace;
}

body { font-family: var(--font-body); }
h1, h2, h3 { font-family: var(--font-headline); }
code, pre, kbd { font-family: var(--font-code); }
```

### Exercise 5 — Type specimen page

```html
<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<title>Type Specimen</title>
<style>
:root {
  --text-xs:   clamp(0.64rem,  0.60rem + 0.2vw,  0.75rem);
  --text-sm:   clamp(0.80rem,  0.75rem + 0.25vw, 0.90rem);
  --text-base: clamp(1rem,     0.95rem + 0.25vw, 1.125rem);
  --text-lg:   clamp(1.125rem, 1rem    + 0.5vw,  1.375rem);
  --text-xl:   clamp(1.25rem,  1rem    + 1vw,    1.75rem);
  --text-2xl:  clamp(1.5rem,   1rem    + 2vw,    2.25rem);
  --text-3xl:  clamp(2rem,     1rem    + 4vw,    3.5rem);
}
*, *::before, *::after { box-sizing: border-box; margin: 0; padding: 0; }
body { font-family: Georgia, serif; padding: 2rem; color: #111; }
.sample { margin-bottom: 2rem; padding-bottom: 2rem; border-bottom: 1px solid #ddd; }
.meta { font-size: 0.75rem; color: #888; font-family: monospace; margin-top: 4px; }
</style>
</head>
<body>
<div class="sample">
  <div style="font-size:var(--text-3xl);font-weight:800;line-height:1.1">The quick brown fox</div>
  <div class="meta">--text-3xl · clamp(2rem, 1rem + 4vw, 3.5rem)</div>
</div>
<div class="sample">
  <div style="font-size:var(--text-2xl);font-weight:700;line-height:1.2">The quick brown fox</div>
  <div class="meta">--text-2xl · clamp(1.5rem, 1rem + 2vw, 2.25rem)</div>
</div>
<div class="sample">
  <div style="font-size:var(--text-base);line-height:1.6">
    Body text at --text-base. Comfortable for reading at paragraph length.
    Resize the viewport to see the fluid scaling.
  </div>
  <div class="meta">--text-base · clamp(1rem, 0.95rem + 0.25vw, 1.125rem)</div>
</div>
<div class="sample">
  <div style="font-size:var(--text-xs);line-height:1.4;color:#555">
    Fine print and footnotes at --text-xs.
  </div>
  <div class="meta">--text-xs · clamp(0.64rem, 0.60rem + 0.2vw, 0.75rem)</div>
</div>
</body>
</html>
```

---

## What You Learned

| Concept | Key point |
|---------|-----------|
| `font-size` | Controls text size; `px`, `rem`, `em` are common units |
| Why not `px` for fonts | Ignores user's browser font-size accessibility setting |
| `rem` | Relative to root (`<html>`) font size; respects user preferences; doesn't compound |
| `em` | Relative to *current element's* font size; compounds in nested elements |
| CSS custom property | `--name: value` defines a variable; `var(--name)` uses it |
| `:root` | Selector for the `<html>` element; variables defined here are globally available |
| Modular scale | Geometric sequence: each step = previous × ratio; produces harmonious sizes |
| `clamp(min, pref, max)` | Fluid value: grows with viewport between a minimum and maximum |
| `vw` unit | 1% of viewport width; used in `clamp()` to create viewport-responsive sizes |
| `line-height` unitless | Unitless value scales with font size; always use this for text |
| `letter-spacing` in `em` | Scales proportionally with font size |
| `calc()` | CSS arithmetic: `calc(var(--text-base) * 0.9)` |

---

## Building with Claude

Bad prompt:
> "How do I make my text responsive?"

Good prompt:
> "I'm building a fluid type scale with `clamp()` in plain CSS. My `--text-3xl` is `clamp(2rem, 1rem + 4vw, 3.5rem)`. At a 1400px viewport I want the maximum to be exactly `4rem`, but the clamp hits `3.5rem` first. If I change the max to `4rem`, the `vw` coefficient `4vw` reaches `3.5rem` already at 1200px and then the value is flat from 1200px to 1400px instead of continuing to grow. Should I increase the `vw` coefficient and lower the max, or is there a better approach for hitting a target value at a specific breakpoint?"
