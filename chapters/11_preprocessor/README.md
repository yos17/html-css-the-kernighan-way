# Chapter 11 — Build Your Own CSS Preprocessor Concepts

Sass solved real problems. In 2006, CSS had no variables, no nesting, no reuse mechanisms at all. Sass gave you `$primary: blue`, nested rules that mirrored your HTML structure, `@mixin` for reusable blocks, `darken()` and `lighten()` for color math, and `@import` to split your CSS into multiple files. These were genuine ergonomic improvements, and Sass dominated front-end CSS for a decade as a result.

Modern CSS has absorbed nearly all of these features. Custom properties replaced Sass variables — and exceeded them by being runtime-dynamic. Native CSS nesting shipped in all major browsers in 2023. `calc()`, `min()`, `max()`, and `clamp()` replaced Sass math. `@layer` replaced `@import` for cascade management, but with actual cascade semantics instead of just concatenation. `color-mix()` and `color-contrast()` do what `darken()` and `lighten()` did, but correctly in perceptual color spaces.

This chapter builds a CSS architecture that demonstrates every Sass feature's native CSS equivalent, side by side. You will understand what preprocessors *do* mechanically — what compiled output they produce — and you will know exactly when a preprocessor is still worth adding to your build pipeline versus when native CSS is sufficient.

## The Problem

The naive approach is to treat preprocessors as a superset of CSS that you always add to projects. Most developers who use Sass today are using it for two things: variables (which CSS now does better) and nesting (which CSS now has natively). The rest of the Sass feature set — mixins, extends, functions — goes mostly unused. Pulling in a build step, a dependency, a compiler, and its version constraints for two features that CSS now provides natively is not a good trade.

The second naive approach is to abandon preprocessors entirely and write flat CSS. That also fails, because flat CSS at scale has a genuine specificity and organization problem that neither variables nor nesting solves on its own. The real answer is `@layer` — a cascade management tool that is categorically more powerful than anything a preprocessor offers, because it operates at the cascade level, not the output level.

The real problem is that most CSS written today — preprocessor or not — does not have a layering strategy. Rules fight each other through specificity, `!important` appears as a band-aid, and adding a new component breaks existing ones. Understanding `@layer`, `@scope`, `:is()`, and `:where()` gives you the tools to write CSS that scales without conflict, whether or not you use a preprocessor.

## Building It Step by Step

### v1: Variables and Nesting

The first Sass feature most people use is variables. Here is the comparison:

```css
/* ─── Variables ─── */

/* Sass */
$primary-color: #2563eb;
$font-size-base: 16px;
$border-radius: 6px;

.button {
  background: $primary-color;
  font-size: $font-size-base;
  border-radius: $border-radius;
}

/* Compiled Sass output */
.button {
  background: #2563eb;
  font-size: 16px;
  border-radius: 6px;
}

/* Native CSS equivalent */
:root {
  --color-primary: #2563eb;
  --font-size-base: 16px;
  --radius: 6px;
}

.button {
  background: var(--color-primary);
  font-size: var(--font-size-base);
  border-radius: var(--radius);
}
```

The compiled Sass output and the native CSS output look similar, but they are fundamentally different in behavior. The compiled output is dead — `#2563eb` is a static string. The custom property is alive — changing `--color-primary` on `:root` with JavaScript updates every consumer instantly. Sass variables are strictly inferior to CSS custom properties for anything you want to change at runtime.

Nesting is where the compiled output makes the difference visible:

```css
/* ─── Nesting ─── */

/* Sass */
.card {
  background: $surface;
  
  .title {
    font-size: 1.25rem;
  }
  
  &:hover {
    box-shadow: 0 4px 12px rgba(0,0,0,0.1);
  }
  
  &.featured {
    border-left: 3px solid $primary-color;
  }
}

/* Sass compiled output — always flat CSS */
.card { background: #f8fafc; }
.card .title { font-size: 1.25rem; }
.card:hover { box-shadow: 0 4px 12px rgba(0,0,0,0.1); }
.card.featured { border-left: 3px solid #2563eb; }

/* Native CSS nesting (identical to Sass, ships in all browsers 2023+) */
.card {
  background: var(--color-surface);

  .title {           /* same as .card .title */
    font-size: 1.25rem;
  }

  &:hover {          /* & means .card */
    box-shadow: 0 4px 12px rgba(0,0,0,0.1);
  }

  &.featured {       /* .card.featured */
    border-left: 3px solid var(--color-primary);
  }
}
```

Native CSS nesting is syntactically almost identical to Sass nesting. The one difference is that native CSS nesting requires `&` or a nested rule that starts with a selector symbol (`>`, `+`, `~`, `.`, `#`, `:`), not a plain element name without `&`. In practice: `& p` not just `p` inside a rule. Modern browsers handle bare element names too, but `&` is safer for now.

### v2: Mixins and Functions

Sass mixins are reusable blocks of CSS declarations. Native CSS does not have an exact equivalent, but the combination of `@layer`, custom properties, and utility classes covers the majority of mixin use cases.

```css
/* ─── Mixins ─── */

/* Sass mixin */
@mixin flex-center {
  display: flex;
  align-items: center;
  justify-content: center;
}

@mixin truncate($width: 100%) {
  width: $width;
  overflow: hidden;
  text-overflow: ellipsis;
  white-space: nowrap;
}

/* Usage */
.hero { @include flex-center; }
.card-title { @include truncate(200px); }

/* Compiled Sass output */
.hero { display: flex; align-items: center; justify-content: center; }
.card-title { width: 200px; overflow: hidden; text-overflow: ellipsis; white-space: nowrap; }

/* Native CSS approach 1: utility classes (no build step needed) */
.flex-center { display: flex; align-items: center; justify-content: center; }
.truncate    { overflow: hidden; text-overflow: ellipsis; white-space: nowrap; }

/* Native CSS approach 2: @layer component + custom properties for parameters */
@layer components {
  [data-truncate] {
    --truncate-width: 100%;
    width: var(--truncate-width);
    overflow: hidden;
    text-overflow: ellipsis;
    white-space: nowrap;
  }
}
/* Usage: <div data-truncate style="--truncate-width: 200px"> */
```

For Sass functions that do math, `calc()` and friends are the native equivalent:

```css
/* ─── Functions ─── */

/* Sass */
.container { width: $base-width * 1.5; }
.text { font-size: clamp(#{$min-font}, 3vw, #{$max-font}); }

/* Native CSS — calc(), min(), max(), clamp() */
.container { width: calc(var(--base-width) * 1.5); }
.text { font-size: clamp(0.875rem, 3vw, 1.25rem); }

/* Sass color functions */
.button:hover { background: darken($primary, 10%); }

/* Native CSS — color-mix() */
.button:hover {
  background: color-mix(in oklch, var(--color-primary) 80%, black);
}

/* color-contrast() — no Sass equivalent at all */
.dynamic-text {
  color: color-contrast(var(--color-surface) vs white, black);
}
```

`color-mix()` in `oklch` color space is actually better than `darken()` — it produces perceptually uniform results rather than the color-space artifacts that Sass's `darken()` produces for saturated colors.

### v3: @layer, @scope, :is(), :where()

This is where native CSS definitively surpasses what any preprocessor can offer. These are runtime cascade tools — they change how the browser resolves specificity conflicts, not just how you write CSS.

```css
/* ─── @layer ─── */

/*
  Layers declared first lose to layers declared later,
  regardless of source order or specificity.
  This makes utilities always win over components, 
  which always win over base styles.
*/

@layer base, components, utilities;

@layer base {
  a { color: blue; }                  /* specificity: 0,0,1 */
}

@layer components {
  .nav a { color: var(--color-primary); }  /* specificity: 0,1,1 */
}

@layer utilities {
  .text-red { color: red; }           /* specificity: 0,1,0 */
}

/*
  Result: .text-red always wins, even though .nav a has higher specificity.
  Unlayered CSS wins over all layers.
*/

/* ─── @scope ─── */

/* Limit styles to a component's subtree — no Sass equivalent */
@scope (.card) {
  /* Only applies to .title inside .card, not .title anywhere */
  .title { font-size: 1.1rem; color: var(--color-text); }
}

@scope (.sidebar) {
  /* A different .title inside .sidebar */
  .title { font-size: 0.9rem; color: var(--color-text-muted); }
}

/* ─── :is() and :where() ─── */

/* Sass would generate separate rules */
.article h1, .article h2, .article h3,
.section h1, .section h2, .section h3 {
  font-weight: 600;
}

/* Native :is() — one rule, takes the highest specificity of its arguments */
:is(.article, .section) :is(h1, h2, h3) {
  font-weight: 600;
}

/* :where() — zero specificity, safe for overriding */
:where(.article, .section) :where(h1, h2, h3) {
  font-weight: 600;     /* can be overridden by any rule */
}
```

The specificity difference between `:is()` and `:where()` is the critical point. `:is(.article, .section) h1` has specificity `(0,1,1)` — the class selector counts. `:where(.article, .section) h1` has specificity `(0,0,1)` — `:where()` contributes zero specificity regardless of what is inside it. Use `:is()` when you want the normal specificity. Use `:where()` in resets and base layers where you want zero specificity so consumers can always override.

## The Complete Program

The full architecture is in `preprocessor.html` as an embedded `<style>` block. Here is the complete CSS listing:

```css
/* CSS architecture — native equivalents of all Sass features */

/* ─── @layer declaration order ─── */
@layer reset, tokens, base, components, utilities;

/* ─── @property declarations ─── */
@property --hue {
  syntax: '<number>';
  inherits: true;
  initial-value: 221;
}

/* ─── Layer: reset ─── */
@layer reset {
  *, *::before, *::after {
    box-sizing: border-box;
    margin: 0;
    padding: 0;
  }
}

/* ─── Layer: tokens (variables — the Sass $var equivalent) ─── */
@layer tokens {
  :root {
    /* Primitive */
    --blue-500: hsl(217 91% 60%);
    --blue-600: hsl(221 83% 53%);
    --blue-700: hsl(224 76% 48%);
    --gray-50:  hsl(210 40% 98%);
    --gray-900: hsl(222 47% 11%);

    /* Semantic */
    --color-primary:       var(--blue-600);
    --color-primary-hover: var(--blue-700);
    --color-surface:       var(--gray-50);
    --color-text:          var(--gray-900);

    /* Spacing */
    --space-1: 4px; --space-2: 8px; --space-3: 12px;
    --space-4: 16px; --space-6: 24px; --space-8: 32px;

    /* Typography */
    --fs-sm: 0.875rem; --fs-md: 1rem; --fs-lg: 1.25rem;

    /* Shape */
    --radius: 6px;
  }
}

/* ─── Layer: base (element defaults — low specificity via :where) ─── */
@layer base {
  :where(body) {
    font-family: system-ui, sans-serif;
    font-size: var(--fs-md);
    line-height: 1.6;
    color: var(--color-text);
    background: var(--color-surface);
  }

  /* Nesting — equivalent to Sass nesting */
  :where(a) {
    color: var(--color-primary);
    text-decoration: underline;

    &:hover { color: var(--color-primary-hover); }
    &:focus-visible { outline: 2px solid var(--color-primary); }
  }

  /* :is() for grouped selectors */
  :is(h1, h2, h3, h4, h5, h6) {
    line-height: 1.2;
    font-weight: 600;
    color: var(--color-text);
  }
}

/* ─── Layer: components ─── */
@layer components {
  .btn {
    /* Component tokens — parameterized like a Sass mixin */
    --btn-bg:      var(--color-primary);
    --btn-color:   white;
    --btn-border:  transparent;
    --btn-radius:  var(--radius);
    --btn-px:      var(--space-4);
    --btn-py:      var(--space-2);

    display: inline-flex;
    align-items: center;
    gap: var(--space-2);
    background: var(--btn-bg);
    color: var(--btn-color);
    border: 1px solid var(--btn-border);
    border-radius: var(--btn-radius);
    padding: var(--btn-py) var(--btn-px);
    font-size: var(--fs-sm);
    cursor: pointer;
    transition: background 150ms, color 150ms, border-color 150ms;

    /* Nesting */
    &:hover {
      background: color-mix(in oklch, var(--btn-bg) 85%, black);
    }

    /* Variants — override component tokens, not semantic tokens */
    &.ghost {
      --btn-bg:     transparent;
      --btn-color:  var(--color-primary);
      --btn-border: var(--color-primary);
      &:hover {
        --btn-bg: var(--color-primary);
        --btn-color: white;
      }
    }

    &.sm { --btn-px: var(--space-3); --btn-py: var(--space-1); font-size: 0.8rem; }
    &.lg { --btn-px: var(--space-6); --btn-py: var(--space-3); font-size: var(--fs-md); }
  }

  /* @scope — component isolation (no Sass equivalent) */
  @scope (.card-component) {
    :scope { /* the .card-component itself */
      background: var(--color-surface);
      border: 1px solid var(--color-border, #e2e8f0);
      border-radius: calc(var(--radius) * 2);
      padding: var(--space-6);
    }

    .title {
      font-size: var(--fs-lg);
      margin-bottom: var(--space-2);
    }

    .body {
      font-size: var(--fs-sm);
      color: color-mix(in oklch, var(--color-text) 60%, transparent);
    }
  }
}

/* ─── Layer: utilities (always win — equivalent to !important class in Sass) ─── */
@layer utilities {
  /* Truncate — the Sass mixin as a utility */
  .truncate {
    overflow: hidden;
    text-overflow: ellipsis;
    white-space: nowrap;
    width: var(--truncate-w, 100%);
  }

  /* Flex utilities */
  .flex-center { display: flex; align-items: center; justify-content: center; }
  .flex-col    { display: flex; flex-direction: column; }
  .flex-wrap   { flex-wrap: wrap; }
  .gap-2       { gap: var(--space-2); }
  .gap-4       { gap: var(--space-4); }

  /* Spacing utilities */
  .p-4         { padding: var(--space-4); }
  .p-6         { padding: var(--space-6); }
  .mt-4        { margin-top: var(--space-4); }

  /* Text */
  .text-sm     { font-size: var(--fs-sm); }
  .text-muted  { color: color-mix(in oklch, currentColor 55%, transparent); }

  /* calc() — Sass math equivalent */
  .w-half      { width: calc(50% - var(--space-2)); }
  .fluid-text  { font-size: clamp(var(--fs-sm), 2.5vw, var(--fs-lg)); }
}
```

## Walkthrough

### Sass Variable vs CSS Custom Property — Compile Time vs Runtime

The most important distinction to understand:

```
SASS VARIABLE                    CSS CUSTOM PROPERTY
─────────────────────────────    ─────────────────────────────
$primary: #2563eb;               :root { --primary: #2563eb; }

Resolved at:  compile time       Resolved at:  runtime
Lives in:     generated CSS      Lives in:     DOM
Inherits?     no                 Inherits?     yes, by default
JavaScript?   cannot read/write  Can read via getComputedStyle()
                                 Can write via style.setProperty()
Transitions?  no                 Yes, if typed with @property
Dark mode?    requires recompile  Change on :root, done

Sass compiled output:
  .btn { background: #2563eb; }     ← dead value baked in

CSS custom property output:
  .btn { background: var(--primary); }  ← live, changeable
```

The only remaining advantage of Sass variables is that they produce smaller CSS output — no `var()` call, just the value. For performance-critical CSS, this matters marginally. For everything else, custom properties are strictly better.

### How Sass Nesting Compiles

Understanding the compiled output is what lets you predict performance and specificity:

```
SASS NESTING                     COMPILED CSS
───────────────────────────────  ──────────────────────────────
.nav {                           .nav { ... }
  background: white;
                                 .nav ul { list-style: none; }
  ul { list-style: none; }
                                 .nav ul li a {
  ul li a {                        color: blue;
    color: blue;                 }
  }
                                 .nav ul li a:hover {
  ul li a:hover {                  color: darkblue;
    color: darkblue;             }
  }
}

Specificity of .nav ul li a: (0, 1, 3)  ← high, hard to override
Specificity of .nav ul li a:hover: (0, 1, 3) + pseudo = (0, 1, 3)

Better with native nesting + :is():
.nav {
  & :is(ul, ol) { list-style: none; }
  & a { color: blue; }           /* specificity: (0,1,1) */
  & a:hover { color: darkblue; } /* specificity: (0,1,1) */
}
```

Deep nesting generates high-specificity selectors. High-specificity selectors are hard to override without even higher specificity or `!important`. The rule: never nest more than 3 levels deep. Sass had this problem too — it just made it easier to accidentally create it.

### The @layer Cascade

`@layer` is the most important CSS feature of the last decade. It gives you explicit control over the cascade order, independent of specificity.

```
WITHOUT @layer — specificity determines winner:
  ┌──────────────────────────────────────────────────┐
  │  a { color: blue; }         specificity: 0,0,1   │
  │  .nav a { color: red; }     specificity: 0,1,1   │  ← WINS
  │  .text-blue { color: blue; } specificity: 0,1,0   │  ← LOSES to .nav a
  └──────────────────────────────────────────────────┘
  .text-blue cannot override .nav a without !important

WITH @layer:
  @layer base, components, utilities;

  @layer base       { a { color: blue; }           /* 0,0,1 */ }
  @layer components { .nav a { color: red; }        /* 0,1,1 */ }
  @layer utilities  { .text-blue { color: blue; }   /* 0,1,0 */ }

  ┌──────────────────────────────────────────────────────────┐
  │  Layer order: base < components < utilities              │
  │  Result: .text-blue WINS because utilities layer > all   │
  │  Specificity is irrelevant between layers                │
  └──────────────────────────────────────────────────────────┘

UNLAYERED CSS beats all layers:
  .override { color: green; }  /* wins over every layer */
```

The layer declaration order — `@layer base, components, utilities` — is what sets the priority. A layer declared later beats a layer declared earlier, regardless of the specificity of rules inside them.

### :is() vs :where() — Specificity Mechanics

```
SELECTOR                          SPECIFICITY
──────────────────────────────    ─────────────
.article h1                       (0, 1, 1)
.article h2                       (0, 1, 1)

:is(.article) h1                  (0, 1, 1)  ← class counts
:is(.article, #main) h1           (1, 0, 1)  ← #main is id, takes highest!

:where(.article) h1               (0, 0, 1)  ← :where contributes 0
:where(.article, #main) h1        (0, 0, 1)  ← still 0, #main ignored

Rule: :is() takes the specificity of its MOST SPECIFIC argument.
      :where() always contributes zero specificity.

Use :is() to simplify grouped selectors without changing behavior.
Use :where() in base/reset layers so everything is easily overridable.
```

### @scope — What Preprocessors Cannot Do

CSS `@scope` limits where rules apply in the DOM tree. No preprocessor can produce this — it is a runtime cascade feature.

```
Without @scope:
  .title { font-size: 1.5rem; }   /* global — applies everywhere */
  .card .title { ... }             /* scoped via descendant — brittle */
  .sidebar .title { ... }          /* another override needed */

With @scope:
  @scope (.card) {
    .title { font-size: 1.5rem; }  /* only inside .card */
  }
  @scope (.sidebar) {
    .title { font-size: 1rem; }    /* only inside .sidebar */
  }

  DOM:
  ┌── .card ──────────────────────┐
  │  ┌── .title ─┐                │  ← gets 1.5rem from @scope(.card)
  │  └───────────┘                │
  └───────────────────────────────┘
  ┌── .sidebar ───────────────────┐
  │  ┌── .title ─┐                │  ← gets 1rem from @scope(.sidebar)
  │  └───────────┘                │
  └───────────────────────────────┘

  No descendant selectors. No specificity conflict. No Sass equivalent.
```

### color-mix() vs Sass darken()/lighten()

Sass `darken($color, 10%)` and `lighten($color, 10%)` operate in the HSL color space. HSL lightness is not perceptually uniform — `darken(blue, 10%)` produces a very different perceived change than `darken(yellow, 10%)`. `color-mix()` in `oklch` is perceptually uniform.

```
Sass:    darken(hsl(217 91% 60%), 10%)  →  hsl(217 91% 50%)
         (shifts lightness in HSL — can shift perceived hue for saturated colors)

CSS:     color-mix(in oklch, hsl(217 91% 60%) 80%, black)
         (mixes toward black in perceptual space — consistent lightness step)

color-mix() also does things Sass cannot:
  color-mix(in oklch, blue 50%, red)        /* perceptual blend */
  color-mix(in srgb, var(--primary) 20%, transparent)  /* tinting */
```

## Guided Try It — Build a Layered Button System

A button with variants is the canonical test of a layered CSS architecture. The goal: base styles never override variants, utilities never override user intent, and everything is overridable without `!important`.

### Step 1: Declare the layers

```css
@layer reset, base, components, utilities;
```

The declaration order is the priority order. Right side wins.

### Step 2: Base button in the components layer

```css
@layer components {
  .btn {
    --btn-bg: var(--color-primary);
    --btn-fg: white;
    background: var(--btn-bg);
    color: var(--btn-fg);
    padding: 8px 16px;
    border-radius: 6px;
    border: none;
    cursor: pointer;
    font-size: 0.875rem;
    transition: background 150ms;

    &:hover {
      background: color-mix(in oklch, var(--btn-bg) 80%, black);
    }
  }

  .btn.danger {
    --btn-bg: hsl(0 84% 60%);
  }
}
```

### Step 3: Add a utility that overrides the component

```css
@layer utilities {
  .bg-green { --btn-bg: hsl(142 71% 45%); }
}
```

Because `utilities` is declared after `components` in the `@layer` declaration, `.bg-green` overrides `.btn.danger` even though `.btn.danger` has higher specificity. Layers defeat specificity.

Think about it: What happens if you use `!important` inside a layer? Research `@layer` and `!important` — the interaction is counterintuitive. `!important` inside a lower-priority layer actually wins over `!important` in a higher-priority layer. Why does the spec work this way?

## Exercises

**1.** Take this Sass snippet and convert it to native CSS with no build step. Use CSS nesting, custom properties, and `color-mix()` instead of Sass color functions. Verify the visual output is equivalent:
```scss
$bg: #1e293b;
$accent: #38bdf8;
.panel {
  background: $bg;
  border: 1px solid lighten($bg, 15%);
  .heading { color: $accent; }
  &:hover { background: darken($bg, 5%); }
}
```

**2.** Demonstrate `@layer` specificity independence: write a `.card .title` rule in a `components` layer with specificity `(0,2,1)`, and a `.title` rule in a `utilities` layer with specificity `(0,1,0)`. Show that the utilities rule wins. Then add an unlayered `.card .title` rule and show that it wins over the utilities layer.

**3.** Build a button system using component tokens (custom properties as parameters): a single `.btn` class parameterized with `--btn-bg`, `--btn-fg`, `--btn-radius`, `--btn-font-size`. Create three variants (primary, danger, ghost) that override only the component tokens, never the semantic tokens.

**4.** Use `@scope` to create two different `.card` components — `.product-card` and `.blog-card` — that each have a `.title`, `.body`, and `.footer`, but with different typography and spacing. The styles for `.title` inside a `.product-card` must not affect `.title` inside a `.blog-card`.

**5.** Demonstrate `color-mix()` vs HSL `darken` equivalents: create a color palette generator that, given a single `--hue` value, generates a 10-step scale using `color-mix(in oklch, hsl(var(--hue) 80% 50%) X%, black)` for the dark steps and `color-mix(in oklch, hsl(var(--hue) 80% 50%) X%, white)` for the light steps. Compare visually to a Sass-style `hsl(var(--hue) 80% X%)` lightness scale.

## Solutions

### Exercise 1 — Sass to Native CSS

```css
/* Native CSS equivalent */
:root {
  --bg: hsl(215 28% 17%);
  --accent: hsl(199 89% 61%);
}

.panel {
  background: var(--bg);
  border: 1px solid color-mix(in oklch, var(--bg) 70%, white);

  & .heading {
    color: var(--accent);
  }

  &:hover {
    background: color-mix(in oklch, var(--bg) 90%, black);
  }
}
```

The key differences: `lighten($bg, 15%)` becomes `color-mix(in oklch, var(--bg) 70%, white)` (70% original + 30% white ≈ lightening by ~15% in perceptual terms). `darken($bg, 5%)` becomes `color-mix(in oklch, var(--bg) 90%, black)`.

### Exercise 2 — @layer Specificity Independence

```html
<style>
  @layer components, utilities;

  @layer components {
    .card .title {            /* specificity: (0,2,1) */
      color: red;
    }
  }

  @layer utilities {
    .title {                  /* specificity: (0,1,0) */
      color: blue;            /* WINS because utilities layer > components layer */
    }
  }

  /* Unlayered CSS — always beats all layers */
  .card .title {              /* specificity: (0,2,1) */
    color: green;             /* WINS over everything */
  }
</style>
<div class="card">
  <h2 class="title">This is green (unlayered wins)</h2>
</div>
```

Remove the unlayered rule, and the title becomes blue (utilities wins). Add `@layer unlayered { .card .title { color: green; } }` and the title becomes blue again because it is now in a layer, and utilities is declared last.

### Exercise 3 — Button System with Component Tokens

```css
@layer components {
  .btn {
    /* Component token defaults */
    --btn-bg:       var(--color-primary);
    --btn-fg:       white;
    --btn-radius:   6px;
    --btn-fs:       0.875rem;
    --btn-border:   transparent;

    display: inline-flex;
    align-items: center;
    gap: 6px;
    background: var(--btn-bg);
    color: var(--btn-fg);
    border: 1px solid var(--btn-border);
    border-radius: var(--btn-radius);
    padding: 6px 16px;
    font-size: var(--btn-fs);
    cursor: pointer;
    transition: background 150ms, color 150ms;

    &:hover {
      background: color-mix(in oklch, var(--btn-bg) 82%, black);
    }
  }

  /* Variants override component tokens only */
  .btn.danger {
    --btn-bg: hsl(0 84% 60%);
  }

  .btn.ghost {
    --btn-bg:     transparent;
    --btn-fg:     var(--color-primary);
    --btn-border: var(--color-primary);

    &:hover {
      --btn-bg: var(--color-primary);
      --btn-fg: white;
    }
  }

  /* Size modifiers override component tokens only */
  .btn.sm { --btn-fs: 0.75rem; --btn-radius: 4px; padding: 3px 10px; }
  .btn.lg { --btn-fs: 1rem;    --btn-radius: 8px; padding: 10px 24px; }
}
```

### Exercise 4 — @scope Component Isolation

```css
@scope (.product-card) {
  :scope {
    padding: 16px;
    border-radius: 8px;
    background: var(--color-surface);
  }
  .title  { font-size: 1.1rem; font-weight: 700; margin-bottom: 4px; }
  .body   { font-size: 0.85rem; color: var(--color-text-muted); }
  .footer { margin-top: 12px; display: flex; justify-content: space-between; }
}

@scope (.blog-card) {
  :scope {
    padding: 24px;
    border-left: 3px solid var(--color-primary);
  }
  .title  { font-size: 1.4rem; font-weight: 400; font-style: italic; }
  .body   { font-size: 1rem; line-height: 1.7; }
  .footer { margin-top: 16px; font-size: 0.8rem; color: var(--color-text-muted); }
}
```

### Exercise 5 — color-mix() Palette vs HSL Lightness Scale

```css
:root { --hue: 221; }

/* oklch perceptual scale */
.oklch-scale {
  --c900: color-mix(in oklch, hsl(var(--hue) 80% 50%) 30%, black);
  --c800: color-mix(in oklch, hsl(var(--hue) 80% 50%) 50%, black);
  --c700: color-mix(in oklch, hsl(var(--hue) 80% 50%) 65%, black);
  --c600: color-mix(in oklch, hsl(var(--hue) 80% 50%) 80%, black);
  --c500: hsl(var(--hue) 80% 50%); /* reference */
  --c400: color-mix(in oklch, hsl(var(--hue) 80% 50%) 80%, white);
  --c300: color-mix(in oklch, hsl(var(--hue) 80% 50%) 60%, white);
  --c200: color-mix(in oklch, hsl(var(--hue) 80% 50%) 40%, white);
  --c100: color-mix(in oklch, hsl(var(--hue) 80% 50%) 20%, white);
  --c50:  color-mix(in oklch, hsl(var(--hue) 80% 50%) 8%, white);
}

/* HSL lightness scale (Sass-style) */
.hsl-scale {
  --c900: hsl(var(--hue) 80% 15%);
  --c800: hsl(var(--hue) 80% 25%);
  --c700: hsl(var(--hue) 80% 35%);
  --c600: hsl(var(--hue) 80% 43%);
  --c500: hsl(var(--hue) 80% 50%); /* same reference */
  --c400: hsl(var(--hue) 80% 62%);
  --c300: hsl(var(--hue) 80% 73%);
  --c200: hsl(var(--hue) 80% 83%);
  --c100: hsl(var(--hue) 80% 92%);
  --c50:  hsl(var(--hue) 80% 97%);
}
```

The visible difference is most pronounced at high saturation values. The oklch scale produces perceptually even steps. The HSL scale produces uneven perceived lightness jumps, especially in the blue and purple hue ranges.

## What You Learned

| Concept | Key point |
|---|---|
| Sass variables vs custom properties | Sass is compile-time; custom properties are runtime and live in the DOM |
| CSS nesting | Native in all browsers since 2023. Use `&` prefix for compound selectors |
| Sass compiled output | Nesting always compiles to flat CSS. Deep nesting → high specificity → hard to override |
| @layer | Explicit cascade order. Later-declared layers win regardless of specificity |
| Unlayered CSS | Always beats any layer. Use for one-off overrides |
| :is() specificity | Takes the specificity of its most specific argument |
| :where() specificity | Always zero. Use in resets so everything is easily overridable |
| @scope | Limits rule application to a DOM subtree. No preprocessor equivalent |
| color-mix() | Perceptually uniform blending. Better than Sass darken()/lighten() |
| When to still use Sass | Loops (@for, @each), complex logic (@if), or teams who need the familiar syntax |

## Building with Claude

Bad prompt:
> "Convert this Sass to CSS"

This produces a mechanical substitution: `$var` → `--var`, `@include` → inline declarations, with no @layer structure and no use of modern CSS features like `@scope` or `color-mix()`.

Good prompt:
> "Convert this Sass component to native CSS using @layer for cascade isolation. The component has a base class and three variants. Use CSS custom properties as component tokens so variants override only the token, not the base rule. Use CSS nesting for the :hover and :focus states. Use color-mix() in oklch instead of Sass darken() for the hover color. Show the @layer declaration and explain how it prevents the variants from needing higher specificity than the base."

This produces an architecture, not just a translation. Claude will layer the output correctly, use `color-mix()` with an explanation of the perceptual color space, and show the specificity arithmetic that makes the layer approach better than the Sass output.

---

*Understanding CSS nesting lets you read the component styles in any modern framework. The `@layer` cascade management used throughout the JavaScript book's demos ensures utility classes override component styles correctly.*
