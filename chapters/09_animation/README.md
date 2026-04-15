# Chapter 9 — Build Your Own Animation Library

CSS animations are not magic. They are a keyframe interpolation engine sitting on top of a timing function, driven by a compositing layer in your GPU. Once you understand what the browser actually does when you write `transition: transform 300ms ease-out`, the whole subject becomes mechanical and predictable. You stop guessing which properties are safe to animate and start knowing — because you understand the rendering pipeline they run through.

This chapter builds `animate.css`, a self-contained animation library in the style of Animate.css. We start with transitions — the simplest kind of motion, triggered by state changes — then add transforms, which let you move, rotate, and scale elements without touching layout. Finally we build `@keyframes` animations: named, loopable, controllable motion sequences. By the end you will have a library of reusable animation classes and a clear mental model for adding motion to any UI.

The skill you are building here is not "how to make things bounce." It is how to think about motion as data — timing, easing, distance, and fill mode — and how to keep animations on the compositor thread so they never drop a frame.

## The Problem

The naive approach to animation is to change a CSS property with JavaScript — update `element.style.left` on every `requestAnimationFrame`. This works, but it forces the browser to recalculate layout on every frame. Layout is expensive. On a mobile device with a constrained GPU, this is where jank comes from.

The second naive approach is to use CSS `transition` on everything and call it a day. That works until you need a looping animation, or an animation that doesn't correspond to a user interaction, or a multi-step animation with more than two states. `transition` can only move between two values. For anything more complex, you need `@keyframes`.

The real problem is not syntax — it is understanding which approach to use, which CSS properties are safe to animate, and what "safe" means. Animating `width` triggers layout. Animating `background-color` triggers paint. Animating `transform` and `opacity` triggers neither — they are handled entirely by the compositor. That is the distinction that separates smooth 60fps animations from janky ones, and it is what this chapter teaches you to use systematically.

## Building It Step by Step

### v1: CSS Transitions

A transition is the simplest animation primitive: when a CSS property changes, instead of snapping to the new value, interpolate smoothly over a duration.

```css
/* v1: animate.css — transitions only */

/* Base transition class */
.transition {
  transition-property: all;
  transition-duration: 300ms;
  transition-timing-function: ease;
}

/* Duration utilities */
.duration-75   { transition-duration: 75ms; }
.duration-100  { transition-duration: 100ms; }
.duration-150  { transition-duration: 150ms; }
.duration-200  { transition-duration: 200ms; }
.duration-300  { transition-duration: 300ms; }
.duration-500  { transition-duration: 500ms; }
.duration-700  { transition-duration: 700ms; }
.duration-1000 { transition-duration: 1000ms; }

/* Timing function utilities */
.ease-linear    { transition-timing-function: linear; }
.ease-in        { transition-timing-function: cubic-bezier(0.4, 0, 1, 1); }
.ease-out       { transition-timing-function: cubic-bezier(0, 0, 0.2, 1); }
.ease-in-out    { transition-timing-function: cubic-bezier(0.4, 0, 0.2, 1); }

/* Property-specific transitions */
.transition-opacity   { transition-property: opacity; }
.transition-transform { transition-property: transform; }
.transition-colors    { transition-property: color, background-color, border-color; }

/* Triggered by .is-active, :hover, :focus — use in combination */
.fade {
  opacity: 1;
  transition: opacity 300ms ease;
}
.fade.is-hidden {
  opacity: 0;
}
```

This is the foundation. The key insight is that `transition` is *declarative* — you declare what should animate, and the browser triggers it automatically whenever the property changes. You do not write animation loops. You toggle classes.

```html
<!-- v1 usage -->
<button class="btn transition-colors duration-200" onclick="this.classList.toggle('active')">
  Toggle
</button>
```

### v2: CSS Transforms

Transforms are the workhorse of CSS animation. They operate on a separate compositing layer, which means they are nearly free from a performance perspective. The browser never has to recalculate layout or repaint when you animate a transform.

```css
/* v2: animate.css — adding transforms */

/* 2D transform utilities */
.translate-x-0    { transform: translateX(0); }
.translate-x-full { transform: translateX(100%); }
.-translate-x-full { transform: translateX(-100%); }
.translate-y-full { transform: translateY(100%); }
.-translate-y-full { transform: translateY(-100%); }

.scale-0   { transform: scale(0); }
.scale-50  { transform: scale(0.5); }
.scale-75  { transform: scale(0.75); }
.scale-90  { transform: scale(0.9); }
.scale-100 { transform: scale(1); }
.scale-105 { transform: scale(1.05); }
.scale-110 { transform: scale(1.1); }
.scale-125 { transform: scale(1.25); }

.rotate-0   { transform: rotate(0deg); }
.rotate-45  { transform: rotate(45deg); }
.rotate-90  { transform: rotate(90deg); }
.rotate-180 { transform: rotate(180deg); }
.-rotate-45 { transform: rotate(-45deg); }

/* 3D transforms — require perspective on parent */
.rotate-x-90 { transform: rotateX(90deg); }
.rotate-y-90 { transform: rotateY(90deg); }

.perspective-sm { perspective: 500px; }
.perspective-md { perspective: 800px; }
.perspective-lg { perspective: 1200px; }

/* Transform origin */
.origin-center { transform-origin: center; }
.origin-top    { transform-origin: top; }
.origin-bottom { transform-origin: bottom; }
.origin-left   { transform-origin: left; }
.origin-right  { transform-origin: right; }

/* Backface visibility for card flips */
.backface-hidden { backface-visibility: hidden; }

/* GPU compositing hint */
.will-change-transform  { will-change: transform; }
.will-change-opacity    { will-change: opacity; }

/* Combined slide utilities — transition + transform */
.slide-in-left {
  transform: translateX(-100%);
  opacity: 0;
  transition: transform 300ms ease-out, opacity 300ms ease-out;
}
.slide-in-left.is-visible {
  transform: translateX(0);
  opacity: 1;
}

.slide-in-right {
  transform: translateX(100%);
  opacity: 0;
  transition: transform 300ms ease-out, opacity 300ms ease-out;
}
.slide-in-right.is-visible {
  transform: translateX(0);
  opacity: 1;
}

.slide-in-up {
  transform: translateY(20px);
  opacity: 0;
  transition: transform 300ms ease-out, opacity 300ms ease-out;
}
.slide-in-up.is-visible {
  transform: translateY(0);
  opacity: 1;
}
```

The transforms give you a vocabulary of motion. The transitions turn that vocabulary into smooth animation. Together they cover the majority of UI motion — drawer slides, modal entrances, button press feedback, card reveals.

### v3: @keyframes Animations

For looping animations, multi-step animations, and motion that does not correspond to a state change, use `@keyframes`. A keyframe animation has a name, a set of waypoints, and a set of playback controls.

```css
/* v3: animate.css — @keyframes animations */

/* ─── Keyframe definitions ─── */

@keyframes fadeIn {
  from { opacity: 0; }
  to   { opacity: 1; }
}

@keyframes fadeOut {
  from { opacity: 1; }
  to   { opacity: 0; }
}

@keyframes slideUp {
  from {
    opacity: 0;
    transform: translateY(30px);
  }
  to {
    opacity: 1;
    transform: translateY(0);
  }
}

@keyframes slideDown {
  from {
    opacity: 1;
    transform: translateY(0);
  }
  to {
    opacity: 0;
    transform: translateY(30px);
  }
}

@keyframes bounceIn {
  0%   { opacity: 0; transform: scale(0.3); }
  50%  { opacity: 1; transform: scale(1.05); }
  70%  { transform: scale(0.9); }
  100% { transform: scale(1); }
}

@keyframes pulse {
  0%, 100% { transform: scale(1); opacity: 1; }
  50%       { transform: scale(1.05); opacity: 0.8; }
}

@keyframes spin {
  from { transform: rotate(0deg); }
  to   { transform: rotate(360deg); }
}

@keyframes wiggle {
  0%, 100% { transform: rotate(0deg); }
  20%      { transform: rotate(-10deg); }
  40%      { transform: rotate(10deg); }
  60%      { transform: rotate(-5deg); }
  80%      { transform: rotate(5deg); }
}

@keyframes shake {
  0%, 100% { transform: translateX(0); }
  20%      { transform: translateX(-10px); }
  40%      { transform: translateX(10px); }
  60%      { transform: translateX(-6px); }
  80%      { transform: translateX(6px); }
}

@keyframes heartbeat {
  0%, 100% { transform: scale(1); }
  14%      { transform: scale(1.3); }
  28%      { transform: scale(1); }
  42%      { transform: scale(1.3); }
  70%      { transform: scale(1); }
}

@keyframes float {
  0%, 100% { transform: translateY(0); }
  50%      { transform: translateY(-12px); }
}

@keyframes flash {
  0%, 50%, 100% { opacity: 1; }
  25%, 75%      { opacity: 0; }
}

@keyframes rubberBand {
  0%   { transform: scale(1, 1); }
  30%  { transform: scale(1.25, 0.75); }
  40%  { transform: scale(0.75, 1.25); }
  50%  { transform: scale(1.15, 0.85); }
  65%  { transform: scale(0.95, 1.05); }
  75%  { transform: scale(1.05, 0.95); }
  100% { transform: scale(1, 1); }
}

/* ─── Animation base: CSS custom properties for control ─── */

.animated {
  --duration: 1s;
  --delay: 0s;
  --repeat: 1;
  --easing: ease;
  --fill: both;

  animation-duration: var(--duration);
  animation-delay: var(--delay);
  animation-iteration-count: var(--repeat);
  animation-timing-function: var(--easing);
  animation-fill-mode: var(--fill);
}

/* ─── Named animation classes ─── */

.animate-fadeIn     { animation-name: fadeIn; }
.animate-fadeOut    { animation-name: fadeOut; }
.animate-slideUp    { animation-name: slideUp; }
.animate-slideDown  { animation-name: slideDown; }
.animate-bounceIn   { animation-name: bounceIn; }
.animate-pulse      { animation-name: pulse; }
.animate-spin       { animation-name: spin; }
.animate-wiggle     { animation-name: wiggle; }
.animate-shake      { animation-name: shake; }
.animate-heartbeat  { animation-name: heartbeat; }
.animate-float      { animation-name: float; }
.animate-flash      { animation-name: flash; }
.animate-rubberBand { animation-name: rubberBand; }

/* ─── Playback control utilities ─── */

.repeat-1        { --repeat: 1; }
.repeat-2        { --repeat: 2; }
.repeat-3        { --repeat: 3; }
.repeat-infinite { --repeat: infinite; }

.fill-none     { --fill: none; }
.fill-forwards { --fill: forwards; }
.fill-backwards{ --fill: backwards; }
.fill-both     { --fill: both; }

.direction-normal    { animation-direction: normal; }
.direction-reverse   { animation-direction: reverse; }
.direction-alternate { animation-direction: alternate; }

/* ─── Staggered animations via custom properties ─── */

.stagger-1  { --delay: calc(var(--stagger-base, 100ms) * 1); }
.stagger-2  { --delay: calc(var(--stagger-base, 100ms) * 2); }
.stagger-3  { --delay: calc(var(--stagger-base, 100ms) * 3); }
.stagger-4  { --delay: calc(var(--stagger-base, 100ms) * 4); }
.stagger-5  { --delay: calc(var(--stagger-base, 100ms) * 5); }

/* Or use nth-child with CSS custom properties for arbitrary lists */
.stagger-children > * {
  animation-delay: calc(var(--stagger-base, 80ms) * var(--i, 0));
}
```

The `animation-fill-mode: both` on `.animated` is critical. Without `forwards`, an animation snaps back to its initial state the moment it ends. Without `backwards`, there is a flash of the final state before a delayed animation starts. `both` handles both cases.

## The Complete Program

The full library is in `animation.html` as an embedded `<style>` block. Here is the complete CSS listing:

```css
/* animate.css — v3 complete */

/* ─── 1. Transitions ─── */

.transition {
  transition-property: color, background-color, border-color, text-decoration-color,
    fill, stroke, opacity, box-shadow, transform, filter, backdrop-filter;
  transition-duration: 300ms;
  transition-timing-function: cubic-bezier(0.4, 0, 0.2, 1);
}

.transition-none      { transition-property: none; }
.transition-all       { transition-property: all; }
.transition-opacity   { transition-property: opacity; }
.transition-transform { transition-property: transform; }
.transition-colors    { transition-property: color, background-color, border-color; }

.duration-75   { transition-duration: 75ms; }
.duration-100  { transition-duration: 100ms; }
.duration-150  { transition-duration: 150ms; }
.duration-200  { transition-duration: 200ms; }
.duration-300  { transition-duration: 300ms; }
.duration-500  { transition-duration: 500ms; }
.duration-700  { transition-duration: 700ms; }
.duration-1000 { transition-duration: 1000ms; }

.ease-linear  { transition-timing-function: linear; }
.ease-in      { transition-timing-function: cubic-bezier(0.4, 0, 1, 1); }
.ease-out     { transition-timing-function: cubic-bezier(0, 0, 0.2, 1); }
.ease-in-out  { transition-timing-function: cubic-bezier(0.4, 0, 0.2, 1); }

/* ─── 2. Transforms ─── */

.scale-0   { transform: scale(0); }
.scale-75  { transform: scale(0.75); }
.scale-90  { transform: scale(0.9); }
.scale-95  { transform: scale(0.95); }
.scale-100 { transform: scale(1); }
.scale-105 { transform: scale(1.05); }
.scale-110 { transform: scale(1.1); }
.scale-125 { transform: scale(1.25); }

.rotate-0   { transform: rotate(0deg); }
.rotate-45  { transform: rotate(45deg); }
.rotate-90  { transform: rotate(90deg); }
.rotate-180 { transform: rotate(180deg); }
.-rotate-45 { transform: rotate(-45deg); }
.-rotate-90 { transform: rotate(-90deg); }

.translate-x-full  { transform: translateX(100%); }
.-translate-x-full { transform: translateX(-100%); }
.translate-y-full  { transform: translateY(100%); }
.-translate-y-full { transform: translateY(-100%); }

.origin-center { transform-origin: center; }
.origin-top    { transform-origin: top center; }
.origin-bottom { transform-origin: bottom center; }
.origin-left   { transform-origin: left center; }
.origin-right  { transform-origin: right center; }

.perspective-sm { perspective: 500px; }
.perspective-md { perspective: 800px; }
.perspective-lg { perspective: 1200px; }
.backface-hidden { backface-visibility: hidden; }

.will-change-transform { will-change: transform; }
.will-change-opacity   { will-change: opacity; }
.will-change-auto      { will-change: auto; }

/* ─── 3. Slide utilities (transition + transform combined) ─── */

.slide-in-left {
  transform: translateX(-100%); opacity: 0;
  transition: transform 350ms cubic-bezier(0, 0, 0.2, 1),
              opacity 350ms ease;
}
.slide-in-left.is-visible { transform: translateX(0); opacity: 1; }

.slide-in-right {
  transform: translateX(100%); opacity: 0;
  transition: transform 350ms cubic-bezier(0, 0, 0.2, 1),
              opacity 350ms ease;
}
.slide-in-right.is-visible { transform: translateX(0); opacity: 1; }

.slide-in-up {
  transform: translateY(20px); opacity: 0;
  transition: transform 350ms cubic-bezier(0, 0, 0.2, 1),
              opacity 350ms ease;
}
.slide-in-up.is-visible { transform: translateY(0); opacity: 1; }

/* ─── 4. Keyframe definitions ─── */

@keyframes fadeIn    { from { opacity: 0; } to { opacity: 1; } }
@keyframes fadeOut   { from { opacity: 1; } to { opacity: 0; } }
@keyframes slideUp   { from { opacity: 0; transform: translateY(30px); } to { opacity: 1; transform: translateY(0); } }
@keyframes slideDown { from { opacity: 1; transform: translateY(0); } to { opacity: 0; transform: translateY(30px); } }
@keyframes bounceIn  {
  0%   { opacity: 0; transform: scale(0.3); }
  50%  { opacity: 1; transform: scale(1.05); }
  70%  { transform: scale(0.9); }
  100% { transform: scale(1); }
}
@keyframes pulse     { 0%, 100% { transform: scale(1); } 50% { transform: scale(1.05); } }
@keyframes spin      { from { transform: rotate(0deg); } to { transform: rotate(360deg); } }
@keyframes wiggle    {
  0%, 100% { transform: rotate(0); }
  20%      { transform: rotate(-10deg); }
  40%      { transform: rotate(10deg); }
  60%      { transform: rotate(-5deg); }
  80%      { transform: rotate(5deg); }
}
@keyframes shake     {
  0%, 100% { transform: translateX(0); }
  20%      { transform: translateX(-10px); }
  40%      { transform: translateX(10px); }
  60%      { transform: translateX(-6px); }
  80%      { transform: translateX(6px); }
}
@keyframes float     { 0%, 100% { transform: translateY(0); } 50% { transform: translateY(-12px); } }
@keyframes flash     { 0%, 50%, 100% { opacity: 1; } 25%, 75% { opacity: 0; } }
@keyframes rubberBand {
  0%   { transform: scale(1, 1); }
  30%  { transform: scale(1.25, 0.75); }
  40%  { transform: scale(0.75, 1.25); }
  50%  { transform: scale(1.15, 0.85); }
  65%  { transform: scale(0.95, 1.05); }
  75%  { transform: scale(1.05, 0.95); }
  100% { transform: scale(1, 1); }
}

/* ─── 5. Animation base ─── */

.animated {
  --duration: 1s;
  --delay: 0s;
  --repeat: 1;
  --easing: ease;
  --fill: both;
  animation-duration: var(--duration);
  animation-delay: var(--delay);
  animation-iteration-count: var(--repeat);
  animation-timing-function: var(--easing);
  animation-fill-mode: var(--fill);
}

.animate-fadeIn     { animation-name: fadeIn; }
.animate-fadeOut    { animation-name: fadeOut; }
.animate-slideUp    { animation-name: slideUp; }
.animate-bounceIn   { animation-name: bounceIn; }
.animate-pulse      { animation-name: pulse; }
.animate-spin       { animation-name: spin; }
.animate-wiggle     { animation-name: wiggle; }
.animate-shake      { animation-name: shake; }
.animate-float      { animation-name: float; }
.animate-flash      { animation-name: flash; }
.animate-rubberBand { animation-name: rubberBand; }

.repeat-infinite { animation-iteration-count: infinite; }
.fill-forwards   { animation-fill-mode: forwards; }

.stagger-1 { --delay: calc(var(--stagger-base, 100ms) * 1); }
.stagger-2 { --delay: calc(var(--stagger-base, 100ms) * 2); }
.stagger-3 { --delay: calc(var(--stagger-base, 100ms) * 3); }
.stagger-4 { --delay: calc(var(--stagger-base, 100ms) * 4); }
.stagger-5 { --delay: calc(var(--stagger-base, 100ms) * 5); }

@media (prefers-reduced-motion: reduce) {
  .animated,
  .transition,
  .slide-in-left,
  .slide-in-right,
  .slide-in-up {
    animation-duration: 0.01ms !important;
    animation-iteration-count: 1 !important;
    transition-duration: 0.01ms !important;
  }
}
```

## Walkthrough

### The Rendering Pipeline

Before animating anything, understand what the browser does to put pixels on screen. There are four stages:

```
┌─────────────────────────────────────────────────────────────┐
│                   RENDERING PIPELINE                         │
├──────────┬──────────┬──────────┬──────────────────────────  │
│  STYLE   │  LAYOUT  │  PAINT   │       COMPOSITE            │
│          │          │          │                             │
│ Resolve  │ Calc box │ Fill in  │ GPU assembles layers        │
│ all CSS  │ positions│ pixels   │ into final frame            │
│ values   │ & sizes  │          │                             │
└──────────┴──────────┴──────────┴─────────────────────────── ┘
     ▲           ▲          ▲               ▲
     │           │          │               │
  cheap       EXPENSIVE   MEDIUM          free
                                        (GPU only)
```

Changing `width`, `height`, `margin`, `padding`, `top`, `left`, `display` — any of these triggers layout. The browser must recalculate the position of every affected element and its neighbors. This cascades.

Changing `color`, `background-color`, `border-color`, `box-shadow` — these skip layout but trigger paint. The browser redraws the pixels for the affected region.

Changing `transform` and `opacity` — these skip both. The element lives on its own compositor layer. The GPU applies the transform as a matrix multiplication. No layout, no paint. This is why you should always animate `transform` instead of `top`/`left` and `opacity` instead of `visibility`/`display`.

### Easing Functions

Every animation has a timing function that maps time to progress. Linear is a straight line. The others are cubic Bezier curves.

```
ease-in-out                ease-out               ease-in
  progress                 progress               progress
  1.0 │         ╭──        1.0 │      ╭───        1.0 │            ╭
      │       ╱             │    ╱               │          ╱
  0.5 │     ╱               │  ╱                 0.5 │        ╱
      │   ╱                 │╱                   │      ╱
  0.0 └───────── time       └────────── time     0.0 └──╱────── time
      0.0       1.0             0.0    1.0             0.0     1.0

linear                     cubic-bezier(0.68, -0.55, 0.27, 1.55)
  progress                   progress
  1.0 │          ╱           1.2 │        ╭──
      │        ╱                 │      ╱
  0.5 │      ╱              0.5 │    ╱         (overshoot!)
      │    ╱                    │  ╱
  0.0 └───────── time       0.0 └╱───────── time
```

`ease-out` starts fast and slows at the end — good for elements entering the screen. `ease-in` starts slow and accelerates — good for elements leaving. `ease-in-out` is smooth in both directions — good for state changes. `cubic-bezier` with values outside 0–1 on the y axis creates overshoot (bounce).

### The will-change Property

`will-change: transform` tells the browser to promote the element to its own compositor layer *before* the animation starts. Without it, the browser promotes the layer at the start of the animation — causing a one-frame jitter.

```
Without will-change:
┌─────────────┐          ┌─────────────┐
│  Main layer │  frame 1 │  Main layer │ frame 2
│  ┌───────┐  │  ───►    │             │
│  │ elem  │  │          │  ┌───────┐  │ (promoted — 1 frame delay)
│  └───────┘  │          │  │ elem  │  │
└─────────────┘          └─────────────┘

With will-change: transform:
┌─────────────┐  ┌──────┐   Same two layers exist from the start.
│  Main layer │  │ elem │   GPU composites immediately. No jitter.
└─────────────┘  └──────┘
```

Use `will-change` judiciously. Every promoted layer consumes GPU memory. Apply it to elements that you *know* will animate, remove it when the animation is done. Never apply it to `*` or large containers.

### @keyframes vs transition — When to Use Each

```
TRANSITION                          @KEYFRAMES
──────────────────────────────────  ────────────────────────────────────
Triggered by state change            Triggered by class assignment or
(hover, class toggle, focus)         JavaScript

Two states: from current → to new    Any number of waypoints: 0%–100%

Reversible: removing the trigger     Does not reverse unless you
reverses the animation               use animation-direction: alternate

Cannot loop                          Can loop: animation-iteration-count

Examples:                            Examples:
  button hover color change            loading spinner
  drawer slide in/out                  pulse notification badge
  modal fade                           floating animation
  dropdown expand                      bounce entrance
```

The rule is simple: if the animation corresponds to a binary state change (open/closed, hover/not-hover, active/inactive), use `transition`. If it is standalone motion — looping, multi-step, or not tied to a user interaction — use `@keyframes`.

### animation-fill-mode: forwards

This is the most common source of animation bugs. By default, when a `@keyframes` animation ends, the element snaps back to its pre-animation state.

```
Without fill-mode:
  t=0         t=300ms       t=301ms
  ┌───────┐   ┌───────┐     ┌───────┐
  │opacity│   │opacity│     │opacity│
  │  = 0  │──►│  = 1  │────►│  = 0  │  SNAP BACK
  └───────┘   └───────┘     └───────┘
                 (end of animation)

With animation-fill-mode: forwards:
  t=0         t=300ms       t=301ms
  ┌───────┐   ┌───────┐     ┌───────┐
  │opacity│   │opacity│     │opacity│
  │  = 0  │──►│  = 1  │────►│  = 1  │  HOLDS FINAL VALUE
  └───────┘   └───────┘     └───────┘
```

Always use `animation-fill-mode: both` on entry animations. `both` combines `forwards` (holds final state) and `backwards` (applies initial state during delay). The `.animated` class in our library sets `--fill: both` by default.

### Staggered Animations

Staggered animations — where a list of elements animates in sequence — are a powerful pattern. The naive approach sets a different `animation-delay` on each element. The modern approach uses CSS custom properties so the delay is computed, not hardcoded.

```
Items animate in sequence:

  item 1: delay = 100ms × 1 = 100ms   ──────╮
  item 2: delay = 100ms × 2 = 200ms         ├─ each 100ms later
  item 3: delay = 100ms × 3 = 300ms         │
  item 4: delay = 100ms × 4 = 400ms   ──────╯

Using --i custom property set by JavaScript:

  for each item at index i:
    item.style.setProperty('--i', i)

  CSS: animation-delay: calc(80ms * var(--i));
```

This approach is composable: you set `--stagger-base` on the parent to control the overall speed, and the children compute their individual delays automatically.

## Guided Try It — Add a Card Flip Animation

A 3D card flip is the classic showcase for CSS 3D transforms. The pattern — two faces, a container with `perspective`, `backface-visibility: hidden` on each face — appears everywhere from login forms to product displays.

### Step 1: HTML structure

```html
<div class="card-scene">        <!-- perspective container -->
  <div class="card-flip">       <!-- the element that rotates -->
    <div class="card-front">Front</div>
    <div class="card-back">Back</div>
  </div>
</div>
```

### Step 2: Base CSS

```css
.card-scene {
  perspective: 800px;           /* gives depth to child transforms */
  width: 200px;
  height: 280px;
}

.card-flip {
  width: 100%;
  height: 100%;
  position: relative;
  transform-style: preserve-3d;  /* children live in 3D space */
  transition: transform 600ms cubic-bezier(0.4, 0, 0.2, 1);
}

.card-flip.is-flipped {
  transform: rotateY(180deg);
}

.card-front,
.card-back {
  position: absolute;
  inset: 0;
  backface-visibility: hidden;   /* hide when facing away */
  display: flex;
  align-items: center;
  justify-content: center;
}

.card-back {
  transform: rotateY(180deg);    /* start facing away */
}
```

### Step 3: Trigger with JavaScript

```javascript
document.querySelector('.card-scene').addEventListener('click', () => {
  document.querySelector('.card-flip').classList.toggle('is-flipped');
});
```

Think about it: `transform-style: preserve-3d` is set on `.card-flip`, not on the individual faces. Why does moving it to `.card-front` or `.card-back` break the effect? What does `preserve-3d` actually control in the rendering pipeline?

## Exercises

**1.** Add a `@keyframes typewriter` animation that simulates text appearing letter by letter using `width: 0` to `width: 100%` and `overflow: hidden`. Add a blinking cursor using a separate animation on a pseudo-element.

**2.** Create a loading spinner using only CSS: a circle with a partially-visible arc that spins continuously. Use `border` with `border-color: transparent` on three sides and your accent color on one. Use `animation-timing-function: linear` and `animation-iteration-count: infinite`.

**3.** Build a "notification badge" animation: a small red circle that pulses (scales from 1 to 1.3 and back) indefinitely, but only when it has an `.unread` class. When the `.unread` class is removed, the animation should stop immediately (no abrupt snap). Research `animation-play-state`.

**4.** Implement a staggered list entrance: a `<ul>` where each `<li>` fades in from `translateY(20px)`, with each item starting 80ms after the previous. Use JavaScript to set `--i` custom properties, then use CSS `calc()` for the delay. The entrance should trigger when the list enters the viewport — use `IntersectionObserver`.

**5.** Build a page transition system: when the user clicks a link, the current page fades out (`fadeOut`, 200ms), then the new page fades in (`fadeIn`, 200ms) after a 200ms delay. Use `animation-fill-mode: forwards` to hold the final state. Implement it with `History.pushState` and class toggles — no full page reload.

## Solutions

### Exercise 1 — Typewriter Effect

```css
@keyframes typewriter {
  from { width: 0; }
  to   { width: 100%; }
}

@keyframes blink-cursor {
  0%, 100% { border-right-color: currentColor; }
  50%       { border-right-color: transparent; }
}

.typewriter {
  display: inline-block;
  overflow: hidden;
  white-space: nowrap;
  border-right: 2px solid currentColor;
  width: 0;
  animation:
    typewriter 3s steps(30, end) forwards,
    blink-cursor 0.7s step-end infinite;
}
```

```html
<p class="typewriter">The quick brown fox jumps over the lazy dog.</p>
```

The `steps(30, end)` timing function is key — it makes the animation jump in discrete steps rather than interpolating smoothly. Use the character count for the step number. The cursor blink uses `step-end` so it switches on exact frames without interpolation.

### Exercise 2 — Loading Spinner

```css
@keyframes spin-arc {
  from { transform: rotate(0deg); }
  to   { transform: rotate(360deg); }
}

.spinner {
  display: inline-block;
  width: 40px;
  height: 40px;
  border-radius: 50%;
  border: 4px solid transparent;
  border-top-color: #a8d8ea;
  animation: spin-arc 700ms linear infinite;
}

/* Two-arc variant with more visual interest */
.spinner-dual {
  display: inline-block;
  width: 40px;
  height: 40px;
  border-radius: 50%;
  border: 4px solid rgba(168, 216, 234, 0.2);
  border-top-color: #a8d8ea;
  border-right-color: #a8d8ea;
  animation: spin-arc 700ms linear infinite;
}
```

### Exercise 3 — Notification Badge

```css
@keyframes pulse-badge {
  0%, 100% { transform: scale(1); }
  50%       { transform: scale(1.3); }
}

.badge {
  display: inline-block;
  width: 12px;
  height: 12px;
  border-radius: 50%;
  background: #e74c3c;
  animation: pulse-badge 1s ease-in-out infinite;
  animation-play-state: paused;   /* off by default */
}

.badge.unread {
  animation-play-state: running;  /* on when .unread present */
}
```

`animation-play-state: paused` freezes the animation at its current position — the element does not snap back to its initial state. When `.unread` is removed, the animation pauses wherever it is in the cycle, which means no visible jump.

### Exercise 4 — Staggered Viewport Entrance

```css
.stagger-list li {
  opacity: 0;
  transform: translateY(20px);
  animation: slideUp 400ms cubic-bezier(0, 0, 0.2, 1) both;
  animation-play-state: paused;
  animation-delay: calc(80ms * var(--i, 0));
}

.stagger-list.is-visible li {
  animation-play-state: running;
}
```

```javascript
document.querySelectorAll('.stagger-list li').forEach((item, i) => {
  item.style.setProperty('--i', i);
});

const observer = new IntersectionObserver(entries => {
  entries.forEach(entry => {
    if (entry.isIntersecting) {
      entry.target.classList.add('is-visible');
      observer.unobserve(entry.target); // animate once
    }
  });
}, { threshold: 0.1 });

document.querySelectorAll('.stagger-list').forEach(list => {
  observer.observe(list);
});
```

### Exercise 5 — Page Transitions

```css
@keyframes page-exit  { from { opacity: 1; } to { opacity: 0; } }
@keyframes page-enter { from { opacity: 0; } to { opacity: 1; } }

.page {
  animation-fill-mode: forwards;
  animation-duration: 200ms;
}

.page.is-exiting {
  animation-name: page-exit;
  animation-timing-function: ease-in;
}

.page.is-entering {
  animation-name: page-enter;
  animation-timing-function: ease-out;
  animation-delay: 200ms;
  opacity: 0;  /* hold initial state during delay */
}
```

```javascript
function navigate(url) {
  const page = document.querySelector('.page');
  page.classList.add('is-exiting');

  page.addEventListener('animationend', () => {
    history.pushState({}, '', url);
    page.innerHTML = fetchPageContent(url); // simplified
    page.classList.remove('is-exiting');
    page.classList.add('is-entering');
    page.addEventListener('animationend', () => {
      page.classList.remove('is-entering');
    }, { once: true });
  }, { once: true });
}
```

## What You Learned

| Concept | Key point |
|---|---|
| Rendering pipeline | Style → Layout → Paint → Composite. Animating `transform` and `opacity` skips Layout and Paint entirely |
| transition | Interpolates between two states triggered by a CSS property change. Cannot loop |
| @keyframes | Named multi-step animations with full playback control. Use for loops and standalone motion |
| timing functions | Cubic Bezier curves mapping time to progress. `ease-out` for entrances, `ease-in` for exits |
| transform | 2D and 3D transforms run on the compositor. Always prefer `translateX` over `left` |
| animation-fill-mode | `forwards` holds final state. `backwards` applies initial state during delay. Use `both` |
| will-change | Promotes element to its own compositor layer before animation. Use sparingly |
| staggered animations | Set `--i` custom property per element; use `calc()` to derive delay |
| animation-play-state | Pause and resume animations without snap-back |
| prefers-reduced-motion | Always add a reduced-motion media query to respect accessibility preferences |

## Building with Claude

Bad prompt:
> "Give me a CSS animation that bounces"

This produces a generic keyframe with no control parameters, no timing function rationale, no fill mode, and no consideration for the rendering pipeline.

Good prompt:
> "I need a CSS @keyframes animation for a card entrance. The card should start at scale(0.8) and opacity 0, overshoot to scale(1.02) at 70%, then settle at scale(1) opacity 1. The animation should use animation-fill-mode: both so it holds its final state, run once on page load, and respect prefers-reduced-motion. Show me the @keyframes block and the class that applies it, and explain why you chose the easing function."

This produces useful code because it specifies the exact motion (overshoot), the fill mode requirement, the trigger conditions, the accessibility requirement, and asks for explanation. Claude will give you a `cubic-bezier` with the right overshoot characteristics, explain why `fill-mode: both` matters here, and write the reduced-motion override correctly.

---

*The loading spinner in Chapter 9's Promise demo is a CSS keyframe animation. The slide-in panels in Chapter 10's Router use transitions on `transform: translateX`. The expanding/collapsing sections in Chapter 13's Debugger use max-height transitions.*
