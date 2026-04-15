# Chapter 2 — Build Your Own Type Scale

Typography is the first thing a visitor perceives about a page. Before they read a word, the ratio of heading to body text, the spacing between lines, and the relationship between sizes communicate whether a page is professional, playful, authoritative, or amateurish. In this chapter you build a fluid type scale from scratch — a system of six sizes derived from a single ratio, each of which grows and shrinks fluidly with the viewport using `clamp()`. You'll understand exactly how Tailwind's `text-sm`, `text-lg`, `text-2xl` work, and why they're defined the way they are.

---

## The Problem

The naive approach to type sizing is to pick pixel values that look right on your screen:

```css
h1 { font-size: 32px; }
h2 { font-size: 24px; }
h3 { font-size: 20px; }
body { font-size: 16px; }
```

Three problems:

1. **Arbitrary numbers**: why 32? why not 36? There's no system — each size is a guess.
2. **No responsiveness**: on a phone, 32px headings are too large. On a wide monitor, they feel small. You need media queries for every size.
3. **Accessibility**: a user who increases their browser's default font size (a common accessibility accommodation) gets no benefit — pixel sizes don't scale with the browser setting.

A type scale solves all three:

```
A scale is a ratio applied repeatedly from a base size.
Base: 1rem (browser default, respects user's font size preference)
Ratio: 1.25 (Major Third — harmonious but not extreme)

Scale:
  xs:   base ÷ ratio²   = 0.64rem  ~10px
  sm:   base ÷ ratio    = 0.80rem  ~13px
  base: 1rem             = 1rem     ~16px
  lg:   base × ratio    = 1.25rem  ~20px
  xl:   base × ratio²   = 1.563rem ~25px
  2xl:  base × ratio³   = 1.953rem ~31px
  3xl:  base × ratio⁵   = 3.052rem ~49px
```

---

## Building It Step by Step

### v1 — Static Scale with `rem`

Start with the scale as pure custom properties. Every size is derived from the base and the ratio:

```css
/* v1: static scale — sizes relative to user's browser default */
:root {
  --ratio: 1.25;

  --text-xs:   0.64rem;    /* 1 ÷ 1.25² */
  --text-sm:   0.80rem;    /* 1 ÷ 1.25  */
  --text-base: 1rem;       /* baseline  */
  --text-lg:   1.25rem;    /* 1 × 1.25  */
  --text-xl:   1.563rem;   /* 1 × 1.25² */
  --text-2xl:  1.953rem;   /* 1 × 1.25³ */
  --text-3xl:  3.052rem;   /* 1 × 1.25⁵ */
}

h1 { font-size: var(--text-3xl); }
h2 { font-size: var(--text-2xl); }
h3 { font-size: var(--text-xl); }
p  { font-size: var(--text-base); }
```

`rem` means "root em" — it's relative to the `<html>` element's font size, which defaults to the browser's setting (typically 16px). If a user sets their browser default to 20px, every `rem` size scales up proportionally. Pixel sizes don't do this.

### v2 — Fluid Scale with `clamp()`

A static scale looks good on one viewport width. At small screens, the large sizes are too big. At large screens, they could be bigger. `clamp()` makes each size fluid:

```css
/* v2: fluid scale — sizes grow with viewport between min and max */
:root {
  /* clamp(MIN, PREFERRED, MAX)
     PREFERRED is a viewport-relative expression that
     slides between MIN and MAX as the viewport changes. */

  --text-sm:   clamp(0.80rem,  0.75rem + 0.25vw,  0.90rem);
  --text-base: clamp(1rem,     0.95rem + 0.25vw,  1.125rem);
  --text-lg:   clamp(1.125rem, 1rem    + 0.5vw,   1.375rem);
  --text-xl:   clamp(1.25rem,  1rem    + 1vw,     1.75rem);
  --text-2xl:  clamp(1.5rem,   1rem    + 2vw,     2.25rem);
  --text-3xl:  clamp(2rem,     1rem    + 4vw,     3.5rem);
}
```

At 320px viewport: `--text-3xl` = `2rem` (the minimum). At 1200px viewport: approaches `3.5rem`. In between: linearly interpolated. No media queries needed.

### v3 — Full System with Line Heights and Spacing

A type scale isn't just sizes. Line height, letter spacing, and the spacing between type elements all need to be part of the system:

```css
/* v3: complete type system */
:root {
  /* ── Sizes ─────────────────────────────────── */
  --text-xs:   clamp(0.64rem,  0.60rem + 0.2vw,  0.75rem);
  --text-sm:   clamp(0.80rem,  0.75rem + 0.25vw, 0.90rem);
  --text-base: clamp(1rem,     0.95rem + 0.25vw, 1.125rem);
  --text-lg:   clamp(1.125rem, 1rem    + 0.5vw,  1.375rem);
  --text-xl:   clamp(1.25rem,  1rem    + 1vw,    1.75rem);
  --text-2xl:  clamp(1.5rem,   1rem    + 2vw,    2.25rem);
  --text-3xl:  clamp(2rem,     1rem    + 4vw,    3.5rem);

  /* ── Line heights ──────────────────────────── */
  --leading-none:   1;      /* headings, logos */
  --leading-tight:  1.25;   /* display text */
  --leading-snug:   1.375;  /* subheadings */
  --leading-normal: 1.5;    /* body copy */
  --leading-relaxed:1.625;  /* long-form reading */
  --leading-loose:  2;      /* captions, footnotes */

  /* ── Letter spacing ─────────────────────────── */
  --tracking-tight:  -0.05em;
  --tracking-normal:  0em;
  --tracking-wide:    0.05em;
  --tracking-widest:  0.15em;

  /* ── Font weight ────────────────────────────── */
  --font-normal:  400;
  --font-medium:  500;
  --font-semibold:600;
  --font-bold:    700;
}

/* Apply the system: */
h1 {
  font-size: var(--text-3xl);
  font-weight: var(--font-bold);
  line-height: var(--leading-tight);
  letter-spacing: var(--tracking-tight);
}

p {
  font-size: var(--text-base);
  font-weight: var(--font-normal);
  line-height: var(--leading-normal);
  letter-spacing: var(--tracking-normal);
}
```

---

## The Complete Program

`type-scale.html` — open in your browser. Drag the viewport slider to see the fluid scaling in real time. Click any size token to copy it.

```html
<!DOCTYPE html>
<html lang="en">
<head><meta charset="UTF-8"><title>Build Your Own Type Scale — Chapter 2</title>
<!-- styles in type-scale.html -->
</head>
<body>
<style>
/* ═══════════════════════════════════════════════════════════════════════
   THE LIBRARY: miniTypeScale
   ═══════════════════════════════════════════════════════════════════════ */
:root {
  --text-xs:   clamp(0.64rem,  0.60rem + 0.2vw,  0.75rem);
  --text-sm:   clamp(0.80rem,  0.75rem + 0.25vw, 0.90rem);
  --text-base: clamp(1rem,     0.95rem + 0.25vw, 1.125rem);
  --text-lg:   clamp(1.125rem, 1rem    + 0.5vw,  1.375rem);
  --text-xl:   clamp(1.25rem,  1rem    + 1vw,    1.75rem);
  --text-2xl:  clamp(1.5rem,   1rem    + 2vw,    2.25rem);
  --text-3xl:  clamp(2rem,     1rem    + 4vw,    3.5rem);

  --leading-tight:  1.25;
  --leading-normal: 1.5;
  --leading-loose:  2;

  --tracking-tight:   -0.05em;
  --tracking-normal:   0em;
  --tracking-widest:   0.15em;
}
</style>
</body>
</html>
```

---

## Walkthrough

### `rem` vs `em` vs `px`

Three unit options for font sizes — each with different reference points:

```
px  — absolute pixels. Ignores browser default. Ignores user preferences.
      h1 { font-size: 32px } → always 32px, even if user needs larger text.

em  — relative to the CURRENT element's font-size.
      Cascades: body 16px → div 1.5em (24px) → p 1.5em (36px!) — compounds!

rem — relative to the ROOT element's font-size (usually browser default: 16px).
      Doesn't compound. 1.5rem is always 1.5 × root.
      Respects user's browser font-size setting.
```

`rem` is the right unit for a type scale. It's predictable (no compounding), it respects user preferences, and the math is simple: `1.25rem` is always `1.25 × root`.

### `clamp()` — Three Arguments

```
clamp(MINIMUM, PREFERRED, MAXIMUM)
         ↑          ↑          ↑
    never smaller  fluid    never larger
    than this      value    than this
```

The PREFERRED is usually a `vw` (viewport width) expression:

```css
font-size: clamp(1rem, 0.95rem + 0.25vw, 1.125rem);
```

At 320px viewport:  `0.95rem + 0.25 × 3.2px = 0.95rem + 0.8px ≈ 1rem`   → clamps to `1rem`
At 1200px viewport: `0.95rem + 0.25 × 12px  = 0.95rem + 3px   ≈ 1.14rem` → approaches `1.125rem`

The `vw` unit is `1/100` of the viewport width. So `1vw` at 1200px = 12px.

### Modular Scale — The Math

A modular scale is a geometric sequence: each value is the previous multiplied by a fixed ratio.

```
ratio = 1.25 (Major Third in musical intervals)

base = 1rem

1rem × 1.25⁰ = 1.000rem  → --text-base
1rem × 1.25¹ = 1.250rem  → --text-lg
1rem × 1.25² = 1.563rem  → --text-xl
1rem × 1.25³ = 1.953rem  → --text-2xl
1rem × 1.25⁴ = 2.441rem
1rem × 1.25⁵ = 3.052rem  → --text-3xl

Going down (dividing instead of multiplying):
1rem ÷ 1.25¹ = 0.800rem  → --text-sm
1rem ÷ 1.25² = 0.640rem  → --text-xs
```

Common ratios and their names:
```
1.067 — Minor Second  (very subtle — caption to body)
1.125 — Major Second  (tight, editorial)
1.200 — Minor Third   (good for dense UIs)
1.250 — Major Third   (default, readable web)
1.333 — Perfect Fourth (headings stand out well)
1.414 — Augmented Fourth / √2
1.500 — Perfect Fifth  (dramatic contrast)
1.618 — Golden Ratio   (very dramatic)
```

### Line Height

Line height controls the vertical space between lines of text. The optimal range for readability:

```
--leading-tight:  1.25  ← headings (few lines, large size)
--leading-normal: 1.5   ← body copy (multiple lines)
--leading-relaxed:1.625 ← long-form reading
```

Why no units? `line-height: 1.5` (unitless) means 1.5× the *current* font size. `line-height: 1.5rem` means 1.5× the *root* font size. For text, you almost always want unitless — the leading should scale with the text size.

```css
/* Unitless: scales with font size */
h1 { font-size: 3rem; line-height: 1.25; }
/* line-height = 3rem × 1.25 = 3.75rem */

p  { font-size: 1rem; line-height: 1.5; }
/* line-height = 1rem × 1.5  = 1.5rem */
```

### Letter Spacing

Letter spacing is relative to the font size using `em`:

```css
--tracking-tight:   -0.05em;  /* Bring letters closer — large headings */
--tracking-normal:   0em;     /* Default — body text */
--tracking-wide:     0.05em;  /* Open — medium UI labels */
--tracking-widest:   0.15em;  /* All-caps abbreviations, tags */
```

Using `em` means the spacing scales proportionally with font size. A `0.05em` gap on a 3rem heading is much larger in pixels than `0.05em` on 1rem body text — which is exactly what you want.

---

## Guided Try It — Add a Fluid Spacing Scale

**The goal**: extend the type scale with a spacing scale — padding, margin, and gap values that use the same harmonic ratios as the type.

**Why this is useful**: when type and spacing use the same ratios, layouts feel coherent. Tailwind's spacing scale and type scale aren't identical, but they're both derived from a consistent base unit.

**Step 1 — Define the spacing tokens**

```css
:root {
  /* Base: 0.25rem (4px at default root size) */
  --space-1:  0.25rem;   /*  4px */
  --space-2:  0.5rem;    /*  8px */
  --space-3:  0.75rem;   /* 12px */
  --space-4:  1rem;      /* 16px */
  --space-5:  1.25rem;   /* 20px */
  --space-6:  1.5rem;    /* 24px */
  --space-8:  2rem;      /* 32px */
  --space-10: 2.5rem;    /* 40px */
  --space-12: 3rem;      /* 48px */
  --space-16: 4rem;      /* 64px */
  --space-20: 5rem;      /* 80px */
  --space-24: 6rem;      /* 96px */
}
```

**Step 2 — Make them fluid with `clamp()`**

```css
:root {
  --space-4:  clamp(0.75rem,  0.5rem  + 1vw, 1rem);
  --space-8:  clamp(1.5rem,   1rem    + 2vw, 2rem);
  --space-16: clamp(3rem,     2rem    + 4vw, 4rem);
}
```

**Step 3 — Use them in components**

```css
.card {
  padding: var(--space-6);
  gap: var(--space-4);
  margin-bottom: var(--space-8);
}

section {
  padding-block: var(--space-16);
}
```

**Think about it**: fluid spacing and fluid type interact. If the heading text shrinks on small screens, should the space around it also shrink? Usually yes — but not always. A call-to-action button might want consistent padding regardless of viewport. How would you create a "fixed" version of each token (e.g., `--space-4-fixed: 1rem`) alongside the fluid version?

---

## Exercises

1. **Add a `--text-4xl` size**: Extend the scale with a fourth extra-large size — `clamp(2.5rem, 1rem + 6vw, 5rem)`. Test: use it for a hero heading and resize the viewport.

2. **Create a monospace scale**: Add a separate set of size tokens for `<code>` elements. Monospace fonts tend to look larger than proportional fonts at the same size, so a monospace scale should use approximately 90% of the corresponding size: `--code-base: calc(var(--text-base) * 0.9)`. Define `--code-sm`, `--code-base`, and `--code-lg`.

3. **Calculate the fluid `vw` midpoint**: For `clamp(1rem, X + Yvw, 1.125rem)`, find the `X` and `Y` values that produce exactly `1rem` at 320px and exactly `1.125rem` at 1200px. Use algebra: `1rem = X + Y×3.2px`, `1.125rem = X + Y×12px`. If `1rem = 16px`, solve for `X` (in rem) and `Y`.

4. **Add optical size adjustments**: Large headings need tighter letter spacing to look right; small text needs looser. Create a mixin-style approach using CSS custom properties: define `--h1-tracking: -0.02em`, `--h2-tracking: -0.01em`, `--body-tracking: 0em`, and apply them to the appropriate heading levels.

5. **Build a type specimen page**: Using only the type scale tokens (no hardcoded sizes), create an HTML page that demonstrates every size in context: a hero section with `--text-3xl`, a subheading in `--text-xl`, body paragraphs in `--text-base`, captions in `--text-sm`, footnotes in `--text-xs`. Apply appropriate line heights and letter spacing from the scale. No hardcoded values — every size must use a custom property.

---

## Solutions

### Exercise 1 — `--text-4xl`

```css
:root {
  --text-4xl: clamp(2.5rem, 1rem + 6vw, 5rem);
}

/* Usage: */
.hero-heading {
  font-size: var(--text-4xl);
  font-weight: 800;
  line-height: 1.1;
  letter-spacing: -0.03em;
}
```

At 320px: `1rem + 6×3.2px = 1rem + 19.2px ≈ 2.2rem` → clamped to `2.5rem`
At 1200px: `1rem + 6×12px = 1rem + 72px ≈ 5.5rem` → clamped to `5rem`

### Exercise 2 — Monospace scale

```css
:root {
  /* Monospace looks ~10% larger, so reduce by 10% */
  --code-xs:   calc(var(--text-xs)   * 0.9);
  --code-sm:   calc(var(--text-sm)   * 0.9);
  --code-base: calc(var(--text-base) * 0.9);
  --code-lg:   calc(var(--text-lg)   * 0.9);
}

code, pre, kbd {
  font-family: 'Courier New', Courier, monospace;
  font-size: var(--code-base);
  line-height: var(--leading-relaxed);
}
```

### Exercise 3 — Fluid midpoint calculation

Given: minimum `1rem` at 320px, maximum `1.125rem` at 1200px. Using `16px = 1rem`:

```
At 320px:  X + Y × 3.2  = 16px    (1rem = 16px)
At 1200px: X + Y × 12   = 18px    (1.125rem = 16 × 1.125)

Subtract:  Y × (12 - 3.2) = 18 - 16
           Y × 8.8 = 2
           Y = 2 / 8.8 ≈ 0.2273

X = 16 - 0.2273 × 3.2 ≈ 16 - 0.727 ≈ 15.27px ≈ 0.954rem

Result: clamp(1rem, 0.954rem + 0.2273vw, 1.125rem)
```

### Exercise 4 — Optical size adjustments

```css
:root {
  --h1-tracking: -0.03em;
  --h2-tracking: -0.02em;
  --h3-tracking: -0.01em;
  --body-tracking: 0em;
  --small-tracking: 0.01em;
  --cap-tracking: 0.08em;    /* for all-caps labels */
}

h1 { letter-spacing: var(--h1-tracking); }
h2 { letter-spacing: var(--h2-tracking); }
h3 { letter-spacing: var(--h3-tracking); }
p  { letter-spacing: var(--body-tracking); }

.label-caps {
  font-size: var(--text-sm);
  letter-spacing: var(--cap-tracking);
  text-transform: uppercase;
  font-weight: 600;
}
```

### Exercise 5 — Type specimen page

```html
<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
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
  body { font-family: Georgia, serif; line-height: 1.5; padding: 2rem; max-width: 65ch; }

  .hero     { font-size: var(--text-3xl); line-height: 1.1; letter-spacing: -0.02em; font-weight: 800; margin-bottom: 0.5em; }
  .subhead  { font-size: var(--text-xl);  line-height: 1.3; font-weight: 400; color: #555; margin-bottom: 2em; }
  .body     { font-size: var(--text-base); line-height: 1.6; margin-bottom: 1em; }
  .caption  { font-size: var(--text-sm);  line-height: 1.4; color: #666; margin-bottom: 0.5em; }
  .footnote { font-size: var(--text-xs);  line-height: 1.4; color: #888; }
</style>
</head>
<body>
  <p class="hero">The quick brown fox</p>
  <p class="subhead">A fluid type system in pure CSS</p>
  <p class="body">Body text uses --text-base with line-height 1.6 for comfortable reading at paragraph length. Resize the viewport to see the fluid scaling.</p>
  <p class="caption">Caption text at --text-sm, used for image descriptions and secondary information.</p>
  <p class="footnote">Footnote text at --text-xs — smallest readable size, for legal copy and fine print.</p>
</body>
</html>
```

---

## What You Learned

| Concept | Key point |
|---------|-----------|
| `rem` unit | Relative to root font size; respects user browser settings; doesn't compound |
| `em` unit | Relative to *current* font size; compounds when nested |
| `px` for type | Ignores user font preferences — avoid for font sizes |
| Modular scale | Geometric sequence from a base × ratio; produces harmonious size relationships |
| `clamp(min, pref, max)` | Fluid sizing between two extremes; the preferred value is usually a `vw` expression |
| `vw` unit | 1/100 of viewport width; used in `clamp()` preferred values for fluid scaling |
| Unitless `line-height` | Scales with current font size; almost always correct for text |
| `letter-spacing` in `em` | Scales proportionally with font size; larger text gets more absolute spacing |
| Custom property for type | `var(--text-xl)` is a contract: a name, not a value |
| Major Third ratio (1.25) | Common modular scale ratio; harmonious without being extreme |

---

## Building with Claude

Bad prompt:
> "What font size should I use for headings?"

Good prompt:
> "I'm building a fluid type scale using `clamp()` and CSS custom properties. My `--text-3xl` is `clamp(2rem, 1rem + 4vw, 3.5rem)`. At 320px viewport this produces 2rem (correct — that's my minimum), but at 768px it produces `2.92rem` and I want it to be closer to `2.5rem` at that breakpoint. I'm using `1rem = 16px`. Should I adjust the `vw` coefficient or the fixed offset, and what's the formula for making the preferred value hit a specific size at a specific viewport width?"
