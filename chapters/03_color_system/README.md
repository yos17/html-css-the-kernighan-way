# Chapter 3 — Build Your Own Color System

Beginners often choose colors one by one: one blue for a button, another blue for a link, another blue for a border. That works for five minutes, then the stylesheet becomes hard to control. In this chapter, you will build a simple color system so related colors stay related and the page feels consistent.

---

## The Problem

The naive approach: pick colors as you go, defining them wherever you use them.

```css
/* scattered across 8 files */
.button-primary   { background: #3b82f6; }
.link             { color: #2563eb; }
.border-focus     { border-color: #1d4ed8; }
.text-muted       { color: #6b7280; }
.bg-surface       { background: #f9fafb; }
.error            { color: #ef4444; }
.success          { color: #22c55e; }
```

Three problems appear quickly:

1. **The colors have no structure**. You know they are all blue, but the code does not say that clearly.
2. **Changes are painful**. If you want a different brand color later, you must hunt through the whole stylesheet.
3. **Dark mode becomes messy**. Every hard-coded color has to be replaced by hand.

A color system fixes this by doing three simple things:

- define colors in one place
- give them names based on their job
- reuse those names everywhere else

That way, one change at the top can update the whole page:

```
Raw palette:  blue-500: hsl(217 91% 60%)
                         ↓
Semantic:     --color-primary: var(--blue-500)
                         ↓
Usage:        .button { background: var(--color-primary) }
```

---

## Building It Step by Step

### v1 — A Static HSL Palette

HSL (Hue, Saturation, Lightness) is the right format for a palette because you can reason about it:

```css
/* v1: palette defined in HSL — readable and adjustable */
:root {
  /* Blue family — same hue (217), varying lightness */
  --blue-50:  hsl(217 91% 97%);
  --blue-100: hsl(217 91% 93%);
  --blue-200: hsl(217 91% 86%);
  --blue-300: hsl(217 91% 74%);
  --blue-400: hsl(217 91% 67%);
  --blue-500: hsl(217 91% 60%);   /* base */
  --blue-600: hsl(217 91% 50%);
  --blue-700: hsl(217 91% 40%);
  --blue-800: hsl(217 91% 30%);
  --blue-900: hsl(217 91% 20%);

  /* Neutral family — low saturation */
  --gray-50:  hsl(220 14% 97%);
  --gray-100: hsl(220 14% 93%);
  --gray-200: hsl(220 13% 86%);
  --gray-300: hsl(220 12% 73%);
  --gray-400: hsl(220 11% 60%);
  --gray-500: hsl(220 10% 46%);
  --gray-600: hsl(220 11% 36%);
  --gray-700: hsl(220 12% 26%);
  --gray-800: hsl(220 13% 18%);
  --gray-900: hsl(220 14% 10%);

  /* Status colors */
  --green-500: hsl(142 71% 45%);
  --yellow-500: hsl(45 93% 47%);
  --red-500:   hsl(0 72% 51%);
}
```

By keeping hue constant and varying lightness, the palette has an inherent harmony. The 50–900 scale mirrors Tailwind's convention: low numbers are light, high numbers are dark.

### v2 — Semantic Tokens

The palette is raw material. Semantic tokens give names to the *roles* colors play:

```css
/* v2: semantic layer — names describe function, not appearance */
:root {
  /* ── Backgrounds ─────────────────────────────── */
  --color-bg:         var(--gray-50);
  --color-bg-subtle:  var(--gray-100);
  --color-bg-raised:  hsl(0 0% 100%);      /* cards, modals */

  /* ── Borders ─────────────────────────────────── */
  --color-border:        var(--gray-200);
  --color-border-strong: var(--gray-300);

  /* ── Text ────────────────────────────────────── */
  --color-text:          var(--gray-900);
  --color-text-muted:    var(--gray-500);
  --color-text-disabled: var(--gray-300);

  /* ── Interactive ─────────────────────────────── */
  --color-primary:          var(--blue-500);
  --color-primary-hover:    var(--blue-600);
  --color-primary-active:   var(--blue-700);
  --color-primary-text:     hsl(0 0% 100%);   /* text on primary bg */

  /* ── Status ──────────────────────────────────── */
  --color-success:  var(--green-500);
  --color-warning:  var(--yellow-500);
  --color-error:    var(--red-500);
}
```

Now `.button { background: var(--color-primary) }` — it uses the role, not the raw color. If the brand color changes from blue to purple, you update one variable in the palette and one alias in the semantic layer. Everything else is untouched.

### v3 — Dark Mode with `prefers-color-scheme`

The semantic layer is what makes dark mode trivial:

```css
/* v3: dark mode swaps the semantic tokens — component CSS is unchanged */

/* Light mode (default) */
:root {
  --color-bg:         var(--gray-50);
  --color-bg-raised:  hsl(0 0% 100%);
  --color-border:     var(--gray-200);
  --color-text:       var(--gray-900);
  --color-text-muted: var(--gray-500);
  --color-primary:    var(--blue-500);
}

/* Dark mode: override the tokens, not the components */
@media (prefers-color-scheme: dark) {
  :root {
    --color-bg:         var(--gray-900);
    --color-bg-raised:  var(--gray-800);
    --color-border:     var(--gray-700);
    --color-text:       var(--gray-50);
    --color-text-muted: var(--gray-400);
    --color-primary:    var(--blue-400);   /* lighter blue for dark bg */
  }
}
```

Zero component CSS changes. The button that says `background: var(--color-primary)` automatically uses blue-500 in light mode and blue-400 in dark mode.

---

## The Complete Program

`color-system.html` — open in your browser. Use the hue slider to regenerate the entire palette from a single hue. Toggle dark mode to see the semantic token swap.

```html
<!DOCTYPE html>
<html lang="en">
<head><meta charset="UTF-8"><title>Build Your Own Color System — Chapter 3</title>
</head>
<body>
<style>
/* ═══════════════════════════════════════════════════════════════════════
   THE LIBRARY: miniColorSystem
   ═══════════════════════════════════════════════════════════════════════ */
:root {
  /* ── Raw palette ──────────────────────────────── */
  --hue:        217;
  --blue-50:    hsl(var(--hue) 91% 97%);
  --blue-500:   hsl(var(--hue) 91% 60%);
  --blue-400:   hsl(var(--hue) 91% 67%);
  --blue-600:   hsl(var(--hue) 91% 50%);
  --gray-50:    hsl(220 14% 97%);
  --gray-900:   hsl(220 14% 10%);

  /* ── Semantic tokens (light mode) ─────────────── */
  --color-bg:       var(--gray-50);
  --color-text:     var(--gray-900);
  --color-primary:  var(--blue-500);
}
@media (prefers-color-scheme: dark) {
  :root {
    --color-bg:      var(--gray-900);
    --color-text:    var(--gray-50);
    --color-primary: var(--blue-400);
  }
}
</style>
</body>
</html>
```

---

## Walkthrough

### HSL — Hue, Saturation, Lightness

HSL is the color model that maps to how humans think about color:

```
hsl(HUE  SATURATION  LIGHTNESS)
     ↑        ↑           ↑
  0–360     0–100%      0–100%
  angle    vivid→gray  dark→light

hsl(217  91%  60%)   →  a medium blue
     |    |    └── 60% lightness: neither too dark nor too light
     |    └────── 91% saturation: vivid, not gray
     └─────────── 217°: blue on the color wheel
```

Why HSL over hex?

```
Hex:  #3b82f6  ← what is this? can I darken it by 10%?
HSL:  hsl(217 91% 60%)  ← blue, vivid, medium. darken: hsl(217 91% 50%)
```

With HSL, you can build a palette by varying just the lightness value while keeping hue and saturation constant. This guarantees harmony.

### The 50–900 Scale

```
--blue-50:   97% lightness  ← almost white, tint for backgrounds
--blue-100:  93%            ← subtle hover state
--blue-200:  86%            ← borders, accents
--blue-300:  74%            ← disabled state color
--blue-400:  67%            ← dark-mode primary
--blue-500:  60%            ← BASE: primary action color
--blue-600:  50%            ← hover state
--blue-700:  40%            ← active/pressed state
--blue-800:  30%            ← dark text on light background
--blue-900:  20%            ← almost black, high contrast text
```

The 100-step numbers are borrowed from Tailwind and communicate relative brightness at a glance. `blue-50` is obviously lighter than `blue-900`. Your team knows what `var(--blue-300)` means without checking the definition.

### CSS Custom Property Cascade

Custom properties cascade like any other CSS property. This makes the two-layer system work:

```
Layer 1: Raw palette (never changes)
  :root { --blue-500: hsl(217 91% 60%); }

Layer 2: Semantic tokens (swapped per theme)
  :root { --color-primary: var(--blue-500); }
  @media (prefers-color-scheme: dark) {
    :root { --color-primary: var(--blue-400); }
  }

Layer 3: Components (never touch color directly)
  .button { background: var(--color-primary); }
```

When dark mode is active, the `@media` block overrides `--color-primary` in `:root`. The button still says `var(--color-primary)` — it just resolves to a different value.

### Contrast and Accessibility

WCAG (Web Content Accessibility Guidelines) defines minimum contrast ratios:

```
Normal text (< 18pt / < 14pt bold):  4.5:1 minimum
Large text  (≥ 18pt / ≥ 14pt bold):  3:1   minimum
UI components and icons:              3:1   minimum
```

Contrast ratio is calculated from relative luminance. For our palette:

```
--blue-500 on white:   4.56:1  ← passes AA for normal text
--blue-600 on white:   5.89:1  ← passes AAA for large text
--blue-500 on gray-900: 10.2:1 ← excellent
white text on blue-500: 4.56:1 ← passes AA
```

The rule of thumb: use shades 600+ for text on white backgrounds. Use shades 50–100 for backgrounds that will have dark text. Never guess — use a contrast checker.

### `color-mix()` — Tinting in Pure CSS

Modern CSS has `color-mix()` for creating tints and shades without Sass:

```css
/* Mix any color with white or black */
.card-hover {
  /* 10% blue mixed into white */
  background: color-mix(in srgb, var(--color-primary) 10%, white);
}

.button-pressed {
  /* 20% darker than primary */
  background: color-mix(in srgb, var(--color-primary), black 20%);
}
```

This replaces Sass `lighten()` and `darken()` with a browser-native function that works with custom properties.

---

## Guided Try It — Generate a Complementary Accent Color

**The goal**: add a complementary accent color derived mathematically from the primary hue. The complement of a hue is `(hue + 180) % 360`.

**Why this is useful**: a complementary accent adds visual interest without creating a discordant palette. It's used for highlights, badges, and call-to-action contrasts.

**Step 1 — Define the accent palette**

```css
:root {
  --hue-primary: 217;
  --hue-accent:  calc(var(--hue-primary) + 180);   /* 397 → wraps to 37: yellow-orange */

  --accent-400: hsl(var(--hue-accent) 85% 65%);
  --accent-500: hsl(var(--hue-accent) 85% 55%);
  --accent-600: hsl(var(--hue-accent) 85% 45%);
}
```

**Step 2 — Add semantic accent tokens**

```css
:root {
  --color-accent:       var(--accent-500);
  --color-accent-hover: var(--accent-600);
  --color-on-accent:    hsl(0 0% 10%);   /* dark text on warm accent */
}

@media (prefers-color-scheme: dark) {
  :root {
    --color-accent:       var(--accent-400);
    --color-on-accent:    hsl(0 0% 10%);
  }
}
```

**Step 3 — Use the accent for secondary CTAs**

```css
.button-accent {
  background: var(--color-accent);
  color: var(--color-on-accent);
}
.button-accent:hover {
  background: var(--color-accent-hover);
}
```

**Think about it**: `calc(var(--hue-primary) + 180)` produces `397` for a primary hue of `217`. CSS `hsl()` automatically wraps hue values that exceed 360 (so `397deg = 37deg`). What would you get if `--hue-primary` were `90`? Compute the complement and check: is yellow (90°) and violet (270°) a good pairing for a UI? Why or why not?

---

## Exercises

1. **Add a triadic palette**: The triadic colors are at `+120°` and `+240°` from the primary hue. Add `--hue-triadic-1` and `--hue-triadic-2` using `calc()`, then define 3–4 shades of each. Create semantic tokens: `--color-info` (triadic-1) and `--color-success` (triadic-2).

2. **Create a status color system**: Define a complete set of status tokens — success, warning, error, and info — each with three variants: `--color-{status}` (solid), `--color-{status}-subtle` (tint for backgrounds), and `--color-{status}-text` (for readable text on the subtle background). Test: create an alert `<div>` for each status that uses these tokens.

3. **Build a contrast checker function**: Write a JavaScript function `contrastRatio(hex1, hex2)` that computes the WCAG contrast ratio between two hex colors. Formula: `(L1 + 0.05) / (L2 + 0.05)` where L1 > L2 are the relative luminances. Relative luminance for an sRGB value: linearize each channel (`c <= 0.04045 ? c/12.92 : ((c+0.055)/1.055)^2.4`), then `L = 0.2126R + 0.7152G + 0.0722B`.

4. **Add `color-scheme` to `:root`**: The `color-scheme` property tells the browser which color scheme an element supports. Add `color-scheme: light dark` to `:root`. Now form elements, scrollbars, and the browser chrome will automatically switch between light and dark appearances alongside your CSS variables.

5. **Create a "brand color" generator**: Write a JavaScript function that takes a single hue value (0–360) and generates a complete 50–900 palette for both a primary and a neutral (gray-tinted) family. Output should be a `<style>` block of CSS custom properties ready to paste into a stylesheet. Hint: for the neutral family, keep saturation at 10–15% to give a faint tint of the primary hue.

---

## Solutions

### Exercise 1 — Triadic palette

```css
:root {
  --hue-primary:    217;
  --hue-triadic-1:  calc(217 + 120);   /* = 337, a magenta/pink */
  --hue-triadic-2:  calc(217 + 240);   /* = 457 → 97, a yellow-green */

  /* Info: triadic-1 (magenta-pink) */
  --info-300: hsl(var(--hue-triadic-1) 80% 74%);
  --info-500: hsl(var(--hue-triadic-1) 80% 60%);
  --info-700: hsl(var(--hue-triadic-1) 80% 40%);

  /* Success: triadic-2 (yellow-green) */
  --success-300: hsl(var(--hue-triadic-2) 75% 70%);
  --success-500: hsl(var(--hue-triadic-2) 75% 50%);
  --success-700: hsl(var(--hue-triadic-2) 75% 35%);

  --color-info:    var(--info-500);
  --color-success: var(--success-500);
}
```

### Exercise 2 — Status color system

```css
:root {
  /* Success */
  --color-success:        hsl(142 71% 45%);
  --color-success-subtle: hsl(142 71% 95%);
  --color-success-text:   hsl(142 71% 25%);

  /* Warning */
  --color-warning:        hsl(45 93% 47%);
  --color-warning-subtle: hsl(45 93% 95%);
  --color-warning-text:   hsl(45 93% 25%);

  /* Error */
  --color-error:          hsl(0 72% 51%);
  --color-error-subtle:   hsl(0 72% 96%);
  --color-error-text:     hsl(0 72% 30%);

  /* Info */
  --color-info:           hsl(217 91% 60%);
  --color-info-subtle:    hsl(217 91% 95%);
  --color-info-text:      hsl(217 91% 30%);
}

.alert {
  padding: 12px 16px;
  border-radius: 6px;
  border-left: 4px solid;
}
.alert-success {
  background: var(--color-success-subtle);
  border-color: var(--color-success);
  color: var(--color-success-text);
}
.alert-error {
  background: var(--color-error-subtle);
  border-color: var(--color-error);
  color: var(--color-error-text);
}
```

### Exercise 3 — Contrast ratio calculator

```javascript
function hexToRgb(hex) {
  const r = parseInt(hex.slice(1, 3), 16) / 255;
  const g = parseInt(hex.slice(3, 5), 16) / 255;
  const b = parseInt(hex.slice(5, 7), 16) / 255;
  return [r, g, b];
}

function linearize(c) {
  return c <= 0.04045
    ? c / 12.92
    : Math.pow((c + 0.055) / 1.055, 2.4);
}

function relativeLuminance(hex) {
  const [r, g, b] = hexToRgb(hex).map(linearize);
  return 0.2126 * r + 0.7152 * g + 0.0722 * b;
}

function contrastRatio(hex1, hex2) {
  const L1 = relativeLuminance(hex1);
  const L2 = relativeLuminance(hex2);
  const lighter = Math.max(L1, L2);
  const darker  = Math.min(L1, L2);
  return (lighter + 0.05) / (darker + 0.05);
}

// Tests:
contrastRatio('#3b82f6', '#ffffff').toFixed(2);  // "4.56"  — AA pass for normal text
contrastRatio('#1d4ed8', '#ffffff').toFixed(2);  // "7.30"  — AAA pass
contrastRatio('#93c5fd', '#ffffff').toFixed(2);  // "1.87"  — fail
```

### Exercise 4 — `color-scheme`

```css
:root {
  color-scheme: light dark;
  /* ... all other tokens ... */
}
```

With `color-scheme: light dark`, the browser will:
- Use a white background and black text for the root by default
- Switch `<input>`, `<select>`, `<textarea>`, scrollbars, and other UA-styled elements to dark variants when dark mode is active
- Expose the `light-dark()` CSS function for conditional values:
  ```css
  color: light-dark(black, white);  /* black in light, white in dark */
  ```

### Exercise 5 — Brand color generator

```javascript
function generatePalette(hue) {
  const lightnesses = [97, 93, 86, 74, 67, 60, 50, 40, 30, 20];
  const steps = [50, 100, 200, 300, 400, 500, 600, 700, 800, 900];

  let css = ':root {\n';

  // Primary family (high saturation)
  steps.forEach((step, i) => {
    css += `  --primary-${step}: hsl(${hue} 91% ${lightnesses[i]}%);\n`;
  });

  // Neutral family (low saturation, tinted with hue)
  steps.forEach((step, i) => {
    css += `  --gray-${step}: hsl(${hue} 12% ${lightnesses[i]}%);\n`;
  });

  css += '\n  /* Semantic tokens */\n';
  css += `  --color-primary: var(--primary-500);\n`;
  css += `  --color-bg:      var(--gray-50);\n`;
  css += `  --color-text:    var(--gray-900);\n`;
  css += `  --color-border:  var(--gray-200);\n`;
  css += '}\n';

  return css;
}

// Test: generatePalette(217) produces a blue-based system
// generatePalette(142) produces a green-based system
console.log(generatePalette(260));  // purple brand
```

---

## What You Learned

| Concept | Key point |
|---------|-----------|
| HSL color model | Hue (0–360°) + Saturation (0–100%) + Lightness (0–100%); human-readable |
| 50–900 palette scale | Convention: 50 is lightest, 900 is darkest; matches Tailwind naming |
| Raw palette vs semantic tokens | Palette = raw values; semantic = roles. Components use semantic, never raw |
| Custom property cascade | Dark mode overrides semantic tokens at `:root` level; components unchanged |
| `prefers-color-scheme` | Media query that detects OS dark mode preference |
| `color-scheme` property | Tells the browser which modes an element supports; adjusts UA styles |
| `color-mix()` | Browser-native tinting/shading; replaces Sass `lighten()`/`darken()` |
| WCAG contrast ratio | `(L1+0.05)/(L2+0.05)` ≥ 4.5:1 for AA normal text; use 600+ on white |
| Complementary hue | `(hue + 180) % 360`; CSS `hsl()` auto-wraps values > 360 |
| `calc()` with custom properties | `calc(var(--hue) + 180)` — arithmetic with tokens |

---

## Building with Claude

Bad prompt:
> "What colors should I use for my website?"

Good prompt:
> "I'm building a CSS color system with a two-layer structure: a raw palette of HSL custom properties (e.g. `--blue-500: hsl(217 91% 60%)`) and a semantic layer (e.g. `--color-primary: var(--blue-500)`). My dark mode override is in `@media (prefers-color-scheme: dark)`. The problem: I need `--color-primary-text` (the text color to use *on top of* a primary-colored background) to automatically switch between white and near-black depending on lightness. In dark mode, `--color-primary` is a lighter blue (67% lightness) where dark text is more readable. Can I conditionally set `--color-primary-text` to white vs dark using `@media` in the semantic layer, or do I need a different approach?"
