# HTML & CSS: The Kernighan Way

**Learn HTML and CSS by building real tools — then building the tools that built them.**

---

## The Approach

Brian Kernighan taught a generation of programmers with a simple idea: *you learn to program by writing programs*. Not by memorizing syntax. Not by reading theory. You learn by building something real, understanding why it works, and making it your own.

This book takes that idea through two phases:

**Chapters 1–4: The Language** — fundamental HTML and CSS through building real tools. The cascade and box model, typography, color, layout. Each chapter builds a working stylesheet from scratch.

**Chapters 5–12: Build Your Own X** — each chapter builds a real CSS library or system from scratch. Tailwind's utility layer. Bootstrap's component model. Animate.css. A responsive grid. A dark mode engine. A form library. CSS-in-JS. A design token system. You use these tools every day; now you'll understand exactly how they work at the rule level.

**The premise**: the best way to understand a CSS framework is to build one. A grid system is just `display: grid` with named column classes. A theme system is CSS custom properties plus a `data-theme` attribute. Dark mode is a two-line media query and a property swap. These aren't magic — they're patterns, and once you see the pattern, you can build anything.

## How Chapters Work

Every chapter follows the same structure:

1. **Intro paragraph** — the concept and why you're building it
2. **The Problem** — what you're solving and why the naive approach fails
3. **Building It Step by Step** — v1, v2, v3: each version adds one concept
4. **The Complete Program** — the full stylesheet or system, readable in one sitting
5. **Walkthrough** — concept by concept, with code, ASCII diagrams, and explanations
6. **Guided Try It** — one extension, walked through step by step
7. **Exercises** — 5 progressively harder problems
8. **Solutions** — complete working code for every exercise
9. **What You Learned** — a summary table of every key concept
10. **Building with Claude** — a bad prompt and a good prompt

## What You Need

Two things:

1. **A web browser** — Chrome, Firefox, Safari, or Edge. You already have one.
2. **A text editor** — [VS Code](https://code.visualstudio.com/) is free and excellent.

No build tools. No preprocessors. No npm. Every program is a single `.html` file you open in a browser. CSS lives in a `<style>` tag. Logic (where needed) lives in a `<script>` tag.

## Table of Contents

### Part 1: The Language

| Chapter | Title | You Build | Concepts |
|---------|-------|-----------|----------|
| 1 | [Build Your Own CSS Reset](chapters/01_css_reset/) | A minimal stylesheet eliminating browser inconsistencies | Cascade, specificity, inheritance, box model, browser defaults |
| 2 | [Build Your Own Type Scale](chapters/02_type_scale/) | A fluid, ratio-based typographic system | `rem`, `clamp()`, viewport units, modular scale, custom properties |
| 3 | [Build Your Own Color System](chapters/03_color_system/) | A semantic color palette with HSL | `hsl()`, custom properties, color relationships, contrast, dark mode |
| 4 | [Build Your Own Layout Engine](chapters/04_layout_engine/) | A grid + flexbox utility library | `display: grid`, `display: flex`, `fr`, `auto-fill`, `minmax()` |

### Part 2: Build Your Own X

| Chapter | Title | You Build | Key Concepts |
|---------|-------|-----------|--------------|
| 5 | [Build Your Own Utility Framework](chapters/05_utility_framework/) | ~100 single-purpose CSS classes (Tailwind-style) | Utility-first CSS, class composition, pseudo-class utilities |
| 6 | [Build Your Own Component Library](chapters/06_component_library/) | Button, Card, Badge, Alert — styled from scratch | BEM methodology, component states, CSS custom property theming |
| 7 | [Build Your Own Animation Library](chapters/07_animation_library/) | 12 named animations (fade, slide, bounce, shake…) | `@keyframes`, `animation`, timing functions, `prefers-reduced-motion` |
| 8 | [Build Your Own Responsive System](chapters/08_responsive_system/) | 5-breakpoint grid with container query support | Media queries, container queries, mobile-first, fluid layouts |
| 9 | [Build Your Own Dark Mode System](chapters/09_dark_mode/) | Light/dark theme with OS sync and manual toggle | `prefers-color-scheme`, custom property cascade, JS + CSS |
| 10 | [Build Your Own Form Library](chapters/10_form_library/) | Complete form controls with validation states | `appearance`, `:focus-visible`, `:valid`/`:invalid`, custom checkboxes |
| 11 | [Build Your Own CSS-in-JS](chapters/11_css_in_js/) | A styled-components–style library in ~100 lines of JS | Tagged templates, `CSSStyleSheet`, `adoptedStyleSheets`, dynamic styles |
| 12 | [Build Your Own Design System](chapters/12_design_system/) | A complete token-based design system | Design tokens, systematic design, single source of truth, CSS architecture |

## Code Conventions

All code uses **modern CSS** (broadly supported in 2024):

- CSS custom properties (`--spacing-4: 1rem`) — no Sass variables
- `clamp()`, `min()`, `max()` — intrinsic sizing over media queries where possible
- `container queries` — component-level responsiveness
- `:is()`, `:where()`, `:has()` — modern selector logic
- `gap` for spacing in flex and grid — no margin hacks
- `aspect-ratio` — no padding-top percentage tricks
- `@layer` — explicit cascade management where specificity matters

Every program is a **single HTML file** with a `<style>` block and, where needed, a `<script>` block. Not how production stylesheets are organized — but the fastest path from an idea to a rendered page, with nothing between you and the CSS.

## Building with Claude

Every chapter ends with a **"Building with Claude"** section showing two prompts: a bad one and a good one. The difference is specificity.

The bad prompt asks a general question. The good prompt shows your exact CSS, describes the visual result you're seeing, and asks a specific question about the property or rule that isn't working. That level of detail is what gets you past "here's an overview" and into a real technical conversation about your actual stylesheet.

General rule: paste the relevant CSS, describe what you expect vs. what you see, and ask the specific question. That's the formula.

## Getting Started

```bash
git clone https://github.com/yos17/html-css-the-kernighan-way.git
cd html-css-the-kernighan-way
```

Open `chapters/01_css_reset/` and read the README. Open `css-reset.html` in your browser. Start there and work forward. By chapter 12, you'll have built a complete design system from scratch.

---

*"The only way to learn a new programming language is by writing programs in it."*
— Brian W. Kernighan & Dennis M. Ritchie, *The C Programming Language* (1978)
