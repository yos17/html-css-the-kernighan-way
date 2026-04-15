# Chapter 10 — Build Your Own Theme System

CSS custom properties are not Sass variables with different syntax. A Sass variable is resolved at compile time, baked into the output CSS as a static value, and gone. A CSS custom property is a live value that exists at runtime, inherits through the DOM, responds to JavaScript, and updates every consumer simultaneously when you change it on an ancestor element. That difference is not a detail — it is the entire reason that every serious design system built after 2018 uses CSS custom properties as its foundation.

This chapter builds `theme-system.css`, a complete design token and theme system. You will learn the three-layer token hierarchy — primitive, semantic, component — and why that structure is not over-engineering but the minimum necessary to make a theme system that is both flexible and consistent. You will implement light and dark mode using two different mechanisms: the `prefers-color-scheme` media query for automatic OS-level switching, and a `data-theme` attribute for explicit user control. And you will use the `@property` rule to type your custom properties, which unlocks animated gradients and transition interpolation that regular custom properties cannot do.

The skill you are building is not "how to do dark mode." It is how to think about color and spacing as a system — a set of relationships between values that can be swapped out as a unit, where every component automatically inherits the right values without knowing anything about the specific theme.

## The Problem

The naive approach to theming is to define colors directly in component styles. Your button has `background: #2563eb`. Your card has `border: 1px solid #e2e8f0`. When you want dark mode, you write a media query and repeat every color, reversed. You end up with twice as many color declarations, tightly coupled to each component. Adding a third theme means tripling everything.

The second naive approach is Sass variables. These are better than hardcoded hex values — you can change `$color-primary` in one place — but they are compile-time constants. You cannot change a Sass variable at runtime without re-generating the CSS. You cannot animate a transition between Sass variable values. And you cannot let JavaScript read or modify them.

The real problem is that colors in a UI are not independent values. They form a system: a primary color implies a hover state, a disabled state, a text-on-primary color, a ring color. A surface color implies text-on-surface, border-on-surface, and input-on-surface. When you define colors as isolated hex values, you break those relationships. When you define them as a typed hierarchy of custom properties, the relationships are encoded in the CSS itself.

## Building It Step by Step

### v1: CSS Custom Properties — The Basics

CSS custom properties (officially "CSS custom properties for cascading variables") are declared with a `--` prefix on any element and consumed with `var()`.

```css
/* v1: theme-system.css — primitives only */

/* Primitive tokens: raw values, no semantics */
:root {
  /* Color palette */
  --blue-50:  hsl(214 100% 97%);
  --blue-100: hsl(214 95% 93%);
  --blue-200: hsl(213 97% 87%);
  --blue-300: hsl(212 96% 78%);
  --blue-400: hsl(213 94% 68%);
  --blue-500: hsl(217 91% 60%);
  --blue-600: hsl(221 83% 53%);
  --blue-700: hsl(224 76% 48%);
  --blue-800: hsl(226 71% 40%);
  --blue-900: hsl(224 64% 33%);

  --gray-50:  hsl(210 40% 98%);
  --gray-100: hsl(210 40% 96%);
  --gray-200: hsl(214 32% 91%);
  --gray-300: hsl(213 27% 84%);
  --gray-400: hsl(215 20% 65%);
  --gray-500: hsl(215 16% 47%);
  --gray-600: hsl(215 19% 35%);
  --gray-700: hsl(215 25% 27%);
  --gray-800: hsl(217 33% 17%);
  --gray-900: hsl(222 47% 11%);

  /* Spacing scale */
  --space-1:  4px;
  --space-2:  8px;
  --space-3:  12px;
  --space-4:  16px;
  --space-6:  24px;
  --space-8:  32px;
  --space-12: 48px;
  --space-16: 64px;

  /* Typography */
  --font-size-xs: 0.75rem;
  --font-size-sm: 0.875rem;
  --font-size-md: 1rem;
  --font-size-lg: 1.125rem;
  --font-size-xl: 1.25rem;
  --font-size-2xl: 1.5rem;

  /* Border radius */
  --radius-sm: 3px;
  --radius-md: 6px;
  --radius-lg: 12px;
  --radius-full: 9999px;
}
```

The `var()` function takes a custom property name and an optional fallback:

```css
.button {
  background: var(--color-primary, blue); /* fallback if --color-primary is unset */
  color: var(--color-on-primary, white);
}
```

Fallbacks can be chained: `var(--first, var(--second, var(--third, solid-default)))`. The browser walks the chain until it finds a defined value.

### v2: Semantic Tokens and Dark Mode

Primitive tokens define the palette. Semantic tokens assign meaning — `--color-primary` is built from a primitive, `--color-surface` is built from a primitive, but consumers use the semantic name, never the primitive.

```css
/* v2: theme-system.css — semantic tokens + dark mode */

/* Light theme (default) */
:root {
  /* Semantic color tokens */
  --color-primary:        var(--blue-600);
  --color-primary-hover:  var(--blue-700);
  --color-primary-active: var(--blue-800);
  --color-on-primary:     var(--gray-50);

  --color-secondary:      var(--gray-600);
  --color-on-secondary:   var(--gray-50);

  --color-surface:        var(--gray-50);
  --color-surface-raised: white;
  --color-surface-sunken: var(--gray-100);

  --color-text:           var(--gray-900);
  --color-text-muted:     var(--gray-600);
  --color-text-disabled:  var(--gray-400);

  --color-border:         var(--gray-200);
  --color-border-strong:  var(--gray-400);

  --color-focus-ring:     var(--blue-400);

  --color-success: hsl(142 71% 45%);
  --color-warning: hsl(38 92% 50%);
  --color-error:   hsl(0 84% 60%);
}

/* Dark theme — same semantic names, different primitive sources */
@media (prefers-color-scheme: dark) {
  :root {
    --color-primary:        var(--blue-400);
    --color-primary-hover:  var(--blue-300);
    --color-primary-active: var(--blue-200);
    --color-on-primary:     var(--gray-900);

    --color-secondary:      var(--gray-400);
    --color-on-secondary:   var(--gray-900);

    --color-surface:        var(--gray-900);
    --color-surface-raised: var(--gray-800);
    --color-surface-sunken: hsl(222 47% 8%);

    --color-text:           var(--gray-50);
    --color-text-muted:     var(--gray-400);
    --color-text-disabled:  var(--gray-600);

    --color-border:         var(--gray-700);
    --color-border-strong:  var(--gray-500);

    --color-focus-ring:     var(--blue-400);
  }
}

/* Explicit data-theme attribute overrides the media query */
[data-theme="light"] {
  --color-primary:        var(--blue-600);
  --color-surface:        var(--gray-50);
  --color-text:           var(--gray-900);
  --color-border:         var(--gray-200);
  /* ... all light values ... */
}

[data-theme="dark"] {
  --color-primary:        var(--blue-400);
  --color-surface:        var(--gray-900);
  --color-text:           var(--gray-50);
  --color-border:         var(--gray-700);
  /* ... all dark values ... */
}
```

The `data-theme` attribute gives you explicit control. The user can override the OS preference. JavaScript sets `document.documentElement.setAttribute('data-theme', 'dark')` and every component on the page updates instantly — no CSS regeneration, no re-render, no flash.

### v3: @layer, Component Tokens, and @property

The final layer of the system: component-level tokens that inherit from semantic tokens, `@layer` for cascade isolation, and `@property` for typed custom properties.

```css
/* v3: theme-system.css — complete system */

@layer tokens, components, utilities;

/* Component tokens: specific to one component family, inherit from semantic */
@layer components {
  .button {
    --button-bg:          var(--color-primary);
    --button-bg-hover:    var(--color-primary-hover);
    --button-text:        var(--color-on-primary);
    --button-radius:      var(--radius-md);
    --button-padding-x:   var(--space-4);
    --button-padding-y:   var(--space-2);
    --button-font-size:   var(--font-size-sm);

    background: var(--button-bg);
    color: var(--button-text);
    border-radius: var(--button-radius);
    padding: var(--button-padding-y) var(--button-padding-x);
    font-size: var(--button-font-size);
    border: none;
    cursor: pointer;
    transition: background 150ms ease;
  }
  .button:hover {
    --button-bg: var(--button-bg-hover);
  }

  /* Variant: override the component token, not the semantic token */
  .button.secondary {
    --button-bg:       var(--color-surface-raised);
    --button-bg-hover: var(--color-surface-sunken);
    --button-text:     var(--color-text);
    border: 1px solid var(--color-border);
  }

  /* Local override: consumer overrides a component token */
  .danger-zone .button {
    --color-primary:       var(--color-error);
    --color-primary-hover: hsl(0 84% 50%);
  }
}

/* @property: typed custom properties with syntax validation */
@property --hue-shift {
  syntax: '<number>';
  inherits: false;
  initial-value: 0;
}

@property --gradient-angle {
  syntax: '<angle>';
  inherits: false;
  initial-value: 0deg;
}

/* Typed properties can be transitioned! Regular custom properties cannot. */
.animated-gradient {
  background: linear-gradient(
    var(--gradient-angle),
    hsl(220 80% 60%),
    hsl(280 80% 60%)
  );
  transition: --gradient-angle 600ms ease;
}
.animated-gradient:hover {
  --gradient-angle: 180deg;
}
```

The `@property` rule is what unlocks animated gradients. A regular CSS custom property is an opaque string — the browser cannot interpolate between `0deg` and `180deg` because it does not know they are angles. With `@property` declaring `syntax: '<angle>'`, the browser knows exactly how to interpolate.

## The Complete Program

The full system is in `theme.html` as an embedded `<style>` block. Here is the complete CSS listing:

```css
/* theme-system.css — complete */

@layer tokens, base, components, utilities;

@layer tokens {

  /* ─── Primitive tokens ─── */

  :root {
    /* Blues */
    --blue-100: hsl(214 95% 93%);
    --blue-200: hsl(213 97% 87%);
    --blue-300: hsl(212 96% 78%);
    --blue-400: hsl(213 94% 68%);
    --blue-500: hsl(217 91% 60%);
    --blue-600: hsl(221 83% 53%);
    --blue-700: hsl(224 76% 48%);
    --blue-800: hsl(226 71% 40%);

    /* Grays */
    --gray-50:  hsl(210 40% 98%);
    --gray-100: hsl(210 40% 96%);
    --gray-200: hsl(214 32% 91%);
    --gray-300: hsl(213 27% 84%);
    --gray-400: hsl(215 20% 65%);
    --gray-500: hsl(215 16% 47%);
    --gray-600: hsl(215 19% 35%);
    --gray-700: hsl(215 25% 27%);
    --gray-800: hsl(217 33% 17%);
    --gray-900: hsl(222 47% 11%);

    /* Spacing */
    --space-1: 4px;   --space-2: 8px;    --space-3: 12px;
    --space-4: 16px;  --space-6: 24px;   --space-8: 32px;
    --space-12: 48px; --space-16: 64px;

    /* Typography */
    --font-size-xs: 0.75rem;  --font-size-sm: 0.875rem;
    --font-size-md: 1rem;     --font-size-lg: 1.125rem;
    --font-size-xl: 1.25rem;  --font-size-2xl: 1.5rem;

    /* Borders */
    --radius-sm: 3px; --radius-md: 6px;
    --radius-lg: 12px; --radius-full: 9999px;
  }

  /* ─── Light theme (semantic tokens) ─── */
  :root,
  [data-theme="light"] {
    --color-primary:        var(--blue-600);
    --color-primary-hover:  var(--blue-700);
    --color-primary-active: var(--blue-800);
    --color-on-primary:     var(--gray-50);

    --color-surface:        var(--gray-50);
    --color-surface-raised: white;
    --color-surface-sunken: var(--gray-100);

    --color-text:           var(--gray-900);
    --color-text-muted:     var(--gray-500);
    --color-text-disabled:  var(--gray-400);

    --color-border:         var(--gray-200);
    --color-border-strong:  var(--gray-400);
    --color-focus-ring:     var(--blue-400);

    --color-success: hsl(142 71% 45%);
    --color-warning: hsl(38 92% 50%);
    --color-error:   hsl(0 84% 60%);
  }

  /* ─── Dark theme ─── */
  @media (prefers-color-scheme: dark) { :root:not([data-theme]) {
    --color-primary:        var(--blue-400);
    --color-primary-hover:  var(--blue-300);
    --color-primary-active: var(--blue-200);
    --color-on-primary:     var(--gray-900);

    --color-surface:        var(--gray-900);
    --color-surface-raised: var(--gray-800);
    --color-surface-sunken: hsl(222 47% 8%);

    --color-text:           var(--gray-50);
    --color-text-muted:     var(--gray-400);
    --color-text-disabled:  var(--gray-600);

    --color-border:         var(--gray-700);
    --color-border-strong:  var(--gray-500);
  }}

  [data-theme="dark"] {
    --color-primary:        var(--blue-400);
    --color-primary-hover:  var(--blue-300);
    --color-on-primary:     var(--gray-900);
    --color-surface:        var(--gray-900);
    --color-surface-raised: var(--gray-800);
    --color-surface-sunken: hsl(222 47% 8%);
    --color-text:           var(--gray-50);
    --color-text-muted:     var(--gray-400);
    --color-border:         var(--gray-700);
    --color-border-strong:  var(--gray-500);
  }

  [data-theme="high-contrast"] {
    --color-primary:        hsl(213 100% 60%);
    --color-primary-hover:  hsl(213 100% 70%);
    --color-on-primary:     black;
    --color-surface:        black;
    --color-surface-raised: hsl(0 0% 8%);
    --color-surface-sunken: hsl(0 0% 4%);
    --color-text:           white;
    --color-text-muted:     hsl(0 0% 80%);
    --color-border:         hsl(0 0% 40%);
    --color-border-strong:  white;
    --color-success: hsl(120 100% 50%);
    --color-error:   hsl(0 100% 60%);
  }
}

@layer base {
  *, *::before, *::after { box-sizing: border-box; margin: 0; padding: 0; }
  html { background: var(--color-surface); color: var(--color-text); transition: background 200ms, color 200ms; }
}

@layer components {
  .btn {
    display: inline-flex; align-items: center; gap: var(--space-2);
    background: var(--color-primary); color: var(--color-on-primary);
    border: 1px solid transparent; border-radius: var(--radius-md);
    padding: var(--space-2) var(--space-4);
    font-size: var(--font-size-sm); cursor: pointer;
    transition: background 150ms ease, color 150ms ease, border-color 150ms ease;
  }
  .btn:hover    { background: var(--color-primary-hover); }
  .btn.ghost    { background: transparent; color: var(--color-primary); border-color: var(--color-primary); }
  .btn.ghost:hover { background: var(--color-primary); color: var(--color-on-primary); }
  .btn.subtle   { background: var(--color-surface-sunken); color: var(--color-text); border-color: var(--color-border); }
  .btn.subtle:hover { border-color: var(--color-border-strong); }

  .card-theme {
    background: var(--color-surface-raised);
    border: 1px solid var(--color-border);
    border-radius: var(--radius-lg);
    padding: var(--space-6);
  }

  .badge-theme {
    display: inline-flex; align-items: center; gap: var(--space-1);
    padding: 2px 8px; border-radius: var(--radius-full);
    font-size: var(--font-size-xs); font-weight: 600;
  }
  .badge-theme.success { background: color-mix(in srgb, var(--color-success) 15%, transparent); color: var(--color-success); }
  .badge-theme.warning { background: color-mix(in srgb, var(--color-warning) 15%, transparent); color: var(--color-warning); }
  .badge-theme.error   { background: color-mix(in srgb, var(--color-error)   15%, transparent); color: var(--color-error); }

  .input-theme {
    width: 100%; padding: var(--space-2) var(--space-3);
    background: var(--color-surface-sunken); color: var(--color-text);
    border: 1px solid var(--color-border); border-radius: var(--radius-md);
    font-size: var(--font-size-sm); outline: none;
    transition: border-color 150ms ease, box-shadow 150ms ease;
  }
  .input-theme:focus {
    border-color: var(--color-primary);
    box-shadow: 0 0 0 3px color-mix(in srgb, var(--color-focus-ring) 30%, transparent);
  }
}

@layer utilities {
  .text-primary { color: var(--color-primary); }
  .text-muted   { color: var(--color-text-muted); }
  .bg-surface   { background: var(--color-surface); }
  .bg-raised    { background: var(--color-surface-raised); }
  .border-theme { border: 1px solid var(--color-border); }
}

/* Typed custom property for animated gradient */
@property --gradient-angle {
  syntax: '<angle>';
  inherits: false;
  initial-value: 135deg;
}
```

## Walkthrough

### Design Token Hierarchy

The three-layer model is not optional — it is the minimum structure that makes a theme system maintainable.

```
LAYER 1: PRIMITIVE TOKENS
  (raw values, no meaning)
  ┌──────────────────────────────────────┐
  │  --blue-500: hsl(217 91% 60%)        │
  │  --gray-900: hsl(222 47% 11%)        │
  │  --space-4:  16px                    │
  └──────────────────────────────────────┘
           │           │
           ▼           ▼
LAYER 2: SEMANTIC TOKENS
  (meaning, no component specifics)
  ┌──────────────────────────────────────┐
  │  --color-primary: var(--blue-500)    │
  │  --color-surface: var(--gray-50)     │
  │  --color-text:    var(--gray-900)    │
  └──────────────────────────────────────┘
           │           │
           ▼           ▼
LAYER 3: COMPONENT TOKENS
  (local overrides, still semantic)
  ┌──────────────────────────────────────┐
  │  .btn { --btn-bg: var(--color-primary) } │
  │  .card { --card-bg: var(--color-surface-raised) } │
  └──────────────────────────────────────┘
```

This indirection is the point. To switch themes, you only change Layer 2. Layer 1 and Layer 3 do not change. Components never reference primitives directly — they reference semantics, and semantics are swapped as a unit when the theme changes.

### How var() Fallbacks Work

The fallback chain in `var()` is evaluated at computed-value time, not definition time.

```
var(--first, var(--second, var(--third, solid-value)))
         │              │              │
         ▼              ▼              ▼
     defined?       defined?       defined?
         │              │              │
        yes ───────►  value        yes ──► value
         │                              │
        no  ──────────────────────────► no ──► solid-value
```

If `--first` is defined anywhere in the ancestor chain, it wins. If not, `--second` is tried. If neither is defined, the literal `solid-value` is used. The key point: "defined" means defined on this element or any ancestor — CSS custom properties inherit by default.

### prefers-color-scheme vs data-theme

These are two independent mechanisms for dark mode, and you want both.

```
OS PREFERENCE                USER PREFERENCE
(prefers-color-scheme)       (data-theme attribute)

┌─────────────────┐          ┌──────────────────────┐
│ macOS dark mode │          │ <html data-theme="dark"> │
│ Windows dark mode│         │ set by JavaScript     │
│ Android dark mode│         │ persisted in localStorage │
└─────────────────┘          └──────────────────────┘
         │                              │
         ▼                              ▼
  @media query                  Attribute selector
  responds automatically        explicit user choice
                                overrides media query
```

The correct implementation: use `prefers-color-scheme` as the default, and `[data-theme]` as an explicit override. When `data-theme` is set, it wins. When it is not set, the media query applies. Use `:root:not([data-theme])` inside the media query to express this:

```css
@media (prefers-color-scheme: dark) {
  :root:not([data-theme]) {
    /* only applies when no explicit theme is set */
  }
}
```

### CSS Custom Properties Are Live

This is the key behavioral difference from Sass variables. Setting a custom property updates every element that consumes it — synchronously, in the same frame.

```
document.documentElement.style
  .setProperty('--color-primary', 'hsl(280 80% 60%)');

          ┌──── html ─────────────────────┐
          │ --color-primary: hsl(280 ...) │
          │                               │
          │  ┌── .btn ───┐  ┌── .link ──┐ │
          │  │ bg: PRIMARY│  │ fg: PRIMARY│ │
          │  └───────────┘  └───────────┘ │
          └───────────────────────────────┘

Both .btn and .link update in the same frame.
No React re-render. No CSS regeneration. Just a
property change that cascades through the DOM.
```

### The @property Rule

Regular custom properties are untyped strings. The browser cannot interpolate between them. `@property` gives them a type.

```css
/* Without @property: cannot be transitioned */
:root { --angle: 0deg; }
.el { background: linear-gradient(var(--angle), red, blue); }
.el:hover { --angle: 180deg; }
/* Result: instantaneous snap, no transition */

/* With @property: can be transitioned */
@property --angle {
  syntax: '<angle>';
  inherits: false;
  initial-value: 0deg;
}
.el { transition: --angle 600ms ease; }
.el:hover { --angle: 180deg; }
/* Result: smooth gradient rotation over 600ms */
```

The `inherits: false` declaration means the property does not inherit from parent elements — it starts fresh at its `initial-value` on each element. Use `inherits: true` for theme tokens that should cascade.

### HSL Color Model for Theming

Use HSL, not hex. `hsl(220 80% 60%)` is readable: 220° on the color wheel (blue), 80% saturated, 60% lightness. Generating a complete palette becomes arithmetic.

```
HSL(220, 80%, X%) palette where X varies:

  X=97%  ████  --blue-50   (near white)
  X=93%  ████  --blue-100
  X=87%  ████  --blue-200
  X=78%  ████  --blue-300
  X=68%  ████  --blue-400
  X=60%  ████  --blue-500  (reference)
  X=53%  ████  --blue-600
  X=48%  ████  --blue-700
  X=40%  ████  --blue-800
  X=33%  ████  --blue-900  (near black)

Switching from blue to purple: change hue from 220 to 280.
The entire scale shifts coherently with a single number change.
```

For dark mode, the general rule is: swap light (`>70%` lightness) for dark (`<25%`) and mid-range for mid-range. The hue and saturation stay roughly the same — the darkness comes from adjusting lightness only.

## Guided Try It — Runtime Color Customizer

A runtime color customizer lets users pick a primary hue and applies it immediately. This is the practical proof that CSS custom properties are live.

### Step 1: The HSL structure

```css
:root {
  --hue: 221;  /* the single knob */

  --color-primary:       hsl(var(--hue) 83% 53%);
  --color-primary-hover: hsl(var(--hue) 76% 45%);
  --color-on-primary:    white;
}
```

### Step 2: The color picker input

```html
<label>
  Primary hue:
  <input type="range" id="hue-picker" min="0" max="360" value="221">
  <span id="hue-value">221°</span>
</label>
```

### Step 3: JavaScript wires them together

```javascript
document.getElementById('hue-picker').addEventListener('input', function() {
  document.documentElement.style.setProperty('--hue', this.value);
  document.getElementById('hue-value').textContent = this.value + '°';
});
```

That is the complete customizer. One JavaScript property write, and every button, link, focus ring, badge, and input on the page updates its color because they all consume `--color-primary`, which consumes `--hue`.

Think about it: The `--hue` variable is set on `:root` via JavaScript. Components consume `--color-primary`, which references `--hue`. If you move `--hue` to a `<section>` instead of `:root`, only components inside that section change. Why? This is CSS variable inheritance. How would you use this scoping to build a section of a page with a different accent color, while the rest uses the default?

## Exercises

**1.** Add a `[data-theme="solarized"]` theme using the Solarized color palette: base background `hsl(192 100% 11%)`, base content `hsl(44 87% 94%)`, primary `hsl(205 69% 49%)`. Map every semantic token from this chapter to a Solarized value. Test that toggling `data-theme="solarized"` on `<html>` updates all components.

**2.** Build a `@property`-based animated border. Declare `--border-hue` as a typed `<number>` property. Use it in `border-color: hsl(var(--border-hue) 80% 60%)`. Transition `--border-hue` from 0 to 360 on `:focus`, with a 400ms duration. Verify it animates smoothly (not just snapping).

**3.** Implement a `prefers-contrast: more` variant. Use the `@media (prefers-contrast: more)` query to increase text contrast and border width for users who need it. Test by enabling "Increase Contrast" in your OS accessibility settings (macOS: System Settings → Accessibility → Display).

**4.** Build a design token inspector: a floating panel that reads all CSS custom properties defined on `:root` using `getComputedStyle(document.documentElement)` and displays them in groups (colors, spacing, typography). Show the resolved value and a color swatch for color tokens. Use `CSS.supports()` to detect `@property` support.

**5.** Implement theme persistence: save the user's chosen theme to `localStorage` on change, and restore it on page load before the first render (to prevent flash of wrong theme). The key is that the script must run synchronously before the `<body>` is rendered — place it in `<head>` without `defer` or `async`.

## Solutions

### Exercise 1 — Solarized Theme

```css
[data-theme="solarized"] {
  /* Solarized primitive overrides */
  --sol-base03: hsl(192 100% 11%);
  --sol-base02: hsl(192 81% 14%);
  --sol-base01: hsl(194 14% 40%);
  --sol-base00: hsl(195 11% 46%);
  --sol-base0:  hsl(186 8% 55%);
  --sol-base1:  hsl(180 7% 60%);
  --sol-base2:  hsl(44 87% 94%);
  --sol-base3:  hsl(44 87% 97%);
  --sol-blue:   hsl(205 69% 49%);
  --sol-cyan:   hsl(175 59% 40%);
  --sol-green:  hsl(68 100% 30%);
  --sol-red:    hsl(1 71% 52%);

  /* Semantic mapping */
  --color-primary:        var(--sol-blue);
  --color-primary-hover:  var(--sol-cyan);
  --color-on-primary:     var(--sol-base3);

  --color-surface:        var(--sol-base03);
  --color-surface-raised: var(--sol-base02);
  --color-surface-sunken: hsl(192 100% 9%);

  --color-text:           var(--sol-base1);
  --color-text-muted:     var(--sol-base01);
  --color-text-disabled:  var(--sol-base00);

  --color-border:         var(--sol-base02);
  --color-border-strong:  var(--sol-base01);

  --color-success: var(--sol-green);
  --color-warning: hsl(45 100% 55%);
  --color-error:   var(--sol-red);
}
```

### Exercise 2 — Animated Border via @property

```css
@property --border-hue {
  syntax: '<number>';
  inherits: false;
  initial-value: 220;
}

.fancy-input {
  border: 2px solid hsl(var(--border-hue) 80% 60%);
  transition: --border-hue 400ms ease;
  outline: none;
  padding: 8px 12px;
  border-radius: 6px;
  background: var(--color-surface-sunken);
  color: var(--color-text);
}

.fancy-input:focus {
  --border-hue: 580;  /* 220 + 360 = same hue, but interpolates through the spectrum */
}
```

The key: animating from `220` to `580` (= 220 + 360) sweeps through the full color wheel. Without `@property`, this would not animate at all.

### Exercise 3 — prefers-contrast

```css
@media (prefers-contrast: more) {
  :root {
    --color-text:          black;
    --color-text-muted:    hsl(0 0% 25%);
    --color-border:        hsl(0 0% 30%);
    --color-border-strong: black;
    --color-surface:       white;
    --color-surface-raised: hsl(0 0% 97%);
  }

  /* Force visible focus rings */
  :focus-visible {
    outline: 3px solid var(--color-primary) !important;
    outline-offset: 2px !important;
  }

  /* Remove subtle borders and rely on strong contrast */
  .card-theme {
    border-width: 2px;
  }
}
```

### Exercise 4 — Token Inspector

```javascript
function inspectTokens() {
  const styles = getComputedStyle(document.documentElement);
  const all = [];

  // CSS custom properties are not enumerable; parse the stylesheet
  for (const sheet of document.styleSheets) {
    try {
      for (const rule of sheet.cssRules) {
        if (rule.selectorText === ':root') {
          const text = rule.cssText;
          const matches = text.matchAll(/--[\w-]+/g);
          for (const m of matches) {
            const name = m[0];
            const value = styles.getPropertyValue(name).trim();
            all.push({ name, value });
          }
        }
      }
    } catch (e) { /* cross-origin */ }
  }

  return all;
}

function isColorToken(name, value) {
  return name.includes('color') ||
         /^hsl|^rgb|^#[0-9a-f]/.test(value);
}
```

### Exercise 5 — Theme Persistence (No FOUC)

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <!-- Must be synchronous, in <head>, before any content renders -->
  <script>
    (function() {
      const stored = localStorage.getItem('theme');
      if (stored) {
        document.documentElement.setAttribute('data-theme', stored);
      } else if (window.matchMedia('(prefers-color-scheme: dark)').matches) {
        document.documentElement.setAttribute('data-theme', 'dark');
      }
    })();
  </script>
  <link rel="stylesheet" href="theme-system.css">
</head>
```

```javascript
// Theme switcher
function setTheme(theme) {
  document.documentElement.setAttribute('data-theme', theme);
  localStorage.setItem('theme', theme);
}

function clearTheme() {
  document.documentElement.removeAttribute('data-theme');
  localStorage.removeItem('theme');
  // Falls back to prefers-color-scheme
}
```

The IIFE in `<head>` is the only way to set the theme before the browser renders any content. Even `DOMContentLoaded` is too late — the browser will paint the default theme for one frame before the script runs.

## What You Learned

| Concept | Key point |
|---|---|
| CSS custom properties | Live, inheritable runtime values — not compile-time constants like Sass variables |
| Design token hierarchy | Primitive → Semantic → Component. Components never reference primitives directly |
| var() fallback chain | `var(--a, var(--b, fallback))` — walks the chain until a defined value is found |
| prefers-color-scheme | Media query for OS-level dark mode preference |
| data-theme attribute | Explicit user override; use `:root:not([data-theme])` inside media queries |
| @layer | Declares cascade order explicitly; layers declared first lose to layers declared later |
| @property | Typed custom properties — unlocks transition interpolation and syntax validation |
| HSL color model | `hsl(hue saturation% lightness%)` — arithmetic palette generation, easy dark mode |
| color-mix() | Native CSS function: `color-mix(in srgb, blue 30%, transparent)` for tinted backgrounds |
| Theme persistence | Store in localStorage, restore in a synchronous `<head>` script to prevent FOUC |

## Building with Claude

Bad prompt:
> "Give me a CSS dark mode"

This produces a `@media (prefers-color-scheme: dark)` block that hardcodes specific colors with no token system, no JavaScript switchability, and no structure you can extend.

Good prompt:
> "I need a CSS custom property theme system for a web app. Build it with three layers: primitive tokens (raw HSL palette values), semantic tokens (--color-primary, --color-surface, --color-text etc. that reference primitives), and component tokens (per-component variables that reference semantics). Implement both light and dark themes using data-theme attribute selectors so JavaScript can switch them. Include @property declarations for --gradient-angle and --hue so they can be transitioned. Show the JavaScript snippet to persist the user's choice in localStorage with no flash of unstyled content."

This produces a complete, layered system with the right architecture rather than a pile of color overrides.

---

*The dark/light theme toggle in Chapter 11's Redux demo and Chapter 13's Debugger are built on exactly this theme system. Every color in the JavaScript book's demos is a CSS custom property from this system.*
