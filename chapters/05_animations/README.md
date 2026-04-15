# Chapter 7 — Build Your Own Animation Library

Animation can look mysterious at first, but most web animation is just a few CSS ideas repeated in different ways. In this chapter, you will build small animations from scratch so you can see the pieces clearly: transitions, keyframes, and transforms.

A good beginner goal here is not “make the craziest animation possible.” It is “make motion feel intentional and understandable.”

---

## The Problem

A beginner-friendly rule is: if CSS can describe the motion, prefer CSS first.

A beginner might try to animate by changing styles with JavaScript over and over:

```javascript
// Naive: manual animation loop
let opacity = 0;
const el = document.getElementById('box');
const interval = setInterval(() => {
  opacity += 0.05;
  el.style.opacity = opacity;
  if (opacity >= 1) clearInterval(interval);
}, 16);
```

That works poorly. The code is noisy, the browser has less room to optimize, and the result can feel jumpy. CSS animation is often simpler because you describe the motion once, then let the browser handle it.

---

## Building It Step by Step

### v1 — CSS Transitions

Transitions are the easiest place to start because they animate between two states you already understand, like normal and hover.

Transitions animate a property from its current value to a new one when a state change occurs (hover, class addition, etc.):

```css
/* v1: transitions for hover states */

.fade {
  opacity:    1;
  transition: opacity 0.2s ease;
}
.fade:hover { opacity: 0.7; }

/* The shorthand: transition: property duration timing-function delay */
.slide-btn {
  transform:  translateX(0);
  transition: transform 0.3s cubic-bezier(0.34, 1.56, 0.64, 1);  /* spring */
}
.slide-btn:hover { transform: translateX(4px); }
```

`transition` watches for changes on the specified property and smooths the change. Without `transition`, the change would be instant.

### v2 — `@keyframes` Animations

Keyframes define a named sequence of states. A class applies the animation:

```css
/* v2: keyframe animations */

@keyframes fadeIn {
  from { opacity: 0; }
  to   { opacity: 1; }
}

@keyframes slideInUp {
  from { transform: translateY(20px); opacity: 0; }
  to   { transform: translateY(0);    opacity: 1; }
}

@keyframes shake {
  0%, 100% { transform: translateX(0); }
  25%       { transform: translateX(-8px); }
  75%       { transform: translateX(8px); }
}

/* Apply with utility classes */
.animate-fade-in    { animation: fadeIn    0.3s ease both; }
.animate-slide-up   { animation: slideInUp 0.4s ease both; }
.animate-shake      { animation: shake     0.5s ease; }
```

The `animation` shorthand: `name duration timing-function delay iteration-count direction fill-mode`.

`both` fill mode is almost always what you want: it applies the first keyframe before the animation starts and holds the last keyframe after it ends.

### v3 — The Library

Add delays, durations, and the full set:

```css
/* v3: full animation library */

/* Duration modifiers */
.duration-75  { animation-duration: 75ms;  }
.duration-150 { animation-duration: 150ms; }
.duration-300 { animation-duration: 300ms; }
.duration-500 { animation-duration: 500ms; }
.duration-700 { animation-duration: 700ms; }
.duration-1000{ animation-duration: 1000ms;}

/* Delay modifiers */
.delay-75  { animation-delay: 75ms;  }
.delay-150 { animation-delay: 150ms; }
.delay-300 { animation-delay: 300ms; }
.delay-500 { animation-delay: 500ms; }
.delay-700 { animation-delay: 700ms; }

/* Easing */
.ease-linear { animation-timing-function: linear; }
.ease-in     { animation-timing-function: cubic-bezier(0.4, 0, 1, 1); }
.ease-out    { animation-timing-function: cubic-bezier(0, 0, 0.2, 1); }
.ease-bounce { animation-timing-function: cubic-bezier(0.34, 1.56, 0.64, 1); }

/* Repeat */
.animate-infinite { animation-iteration-count: infinite; }

/* Performance hint */
.animate-gpu { will-change: transform, opacity; }
```

---

## The Complete Program

`animations.html` — a gallery of all animations. Click any card to replay. Toggle reduced-motion mode to see the accessibility fallback.

---

## Walkthrough

### The Transform Coordinate System

This section is often where beginners slow down. That is okay. The important first step is just learning that transforms move or reshape the element without changing the normal document flow the same way layout properties do.

Transforms operate on the element's local coordinate system, centered at `transform-origin` (default: center center):

```
translateX(n):   move right (+) or left (-)
translateY(n):   move down  (+) or up   (-)
translateZ(n):   move toward (+) or away (-) from viewer

scale(n):        grow (>1) or shrink (<1)
scaleX(n):       stretch/squish horizontally
scaleY(n):       stretch/squish vertically

rotate(n):       rotate clockwise (positive degrees)
rotateX(n):      flip on horizontal axis (3D)
rotateY(n):      flip on vertical axis (3D)

skewX(n):        shear horizontally
skewY(n):        shear vertically
```

Transforms compose from right to left: `transform: rotate(45deg) translateX(100px)` first moves 100px to the right *in the rotated coordinate system*, then rotates. The order matters.

### Easing Functions

An easing function controls the acceleration curve of the animation:

```
linear:          ──────────── constant speed
ease:            ──╮──── slow start, fast middle, slow end (default)
ease-in:         ──────╮ slow start, fast end
ease-out:        ╮───── fast start, slow end
ease-in-out:     ──╮──╮ slow start, slow end

cubic-bezier(x1, y1, x2, y2):  custom curve
  spring: cubic-bezier(0.34, 1.56, 0.64, 1)  ← overshoots, then snaps back
  snappy: cubic-bezier(0.25, 0.46, 0.45, 0.94)
```

The four numbers are the control points of a cubic Bezier curve. X values must be 0–1 (time). Y values can exceed 0–1 to create overshoot (the "spring" effect).

### `will-change` — Hardware Acceleration

```css
/* Tell the browser to promote this element to its own GPU layer */
.animated-el {
  will-change: transform, opacity;
}
```

This hints to the browser that `transform` and `opacity` on this element will change soon. The browser allocates a GPU compositing layer for it. Animations on composited layers skip layout and paint — they're handled entirely by the GPU compositor, which is why they're smooth.

**Don't overuse**: every composited layer uses GPU memory. Setting `will-change` on everything wastes memory and can slow non-animated performance. Add it only to elements that actually animate.

### `prefers-reduced-motion`

Some users configure their OS to reduce animation because it causes motion sickness or vestibular disorders. CSS has a media query for this:

```css
/* Default: animate */
.animate-fade-in {
  animation: fadeIn 0.3s ease both;
}

/* Respect user's OS preference */
@media (prefers-reduced-motion: reduce) {
  .animate-fade-in {
    animation: none;   /* or a simpler instant version */
  }
}

/* Or: disable all animations at once */
@media (prefers-reduced-motion: reduce) {
  *, *::before, *::after {
    animation-duration:   0.01ms !important;
    animation-iteration-count: 1 !important;
    transition-duration:  0.01ms !important;
  }
}
```

This is a WCAG 2.1 accessibility requirement, not a nice-to-have.

### The Stagger Pattern

Animate a list of items with increasing delays to create a cascading effect:

```javascript
// JavaScript stagger: set delay via CSS custom property
document.querySelectorAll('.list-item').forEach((el, i) => {
  el.style.setProperty('--stagger', i);
});
```

```css
.list-item {
  animation: slideInUp 0.4s ease both;
  animation-delay: calc(var(--stagger) * 60ms);
}
```

Item 0 starts at 0ms, item 1 at 60ms, item 2 at 120ms, etc.

---

## Guided Try It — Build a Skeleton Loading Screen

**The goal**: a "skeleton" loading state — grey rectangles that pulse while content loads.

**Step 1 — The pulse keyframe**

```css
@keyframes pulse {
  0%, 100% { opacity: 1;   }
  50%       { opacity: 0.4; }
}
```

**Step 2 — Skeleton components**

```css
.skeleton {
  background:    #e2e8f0;
  border-radius: 4px;
  animation:     pulse 1.5s ease-in-out infinite;
}

.skeleton-text   { height: 1em;    width: 100%; margin-bottom: 0.5em; }
.skeleton-title  { height: 1.5em;  width: 60%; }
.skeleton-avatar { width: 48px; height: 48px; border-radius: 50%; }
.skeleton-image  { width: 100%; aspect-ratio: 16/9; }
```

**Step 3 — Stagger the pulses**

```css
/* Offset each skeleton element's pulse so they don't all blink together */
.skeleton:nth-child(1) { animation-delay: 0ms;    }
.skeleton:nth-child(2) { animation-delay: 150ms;  }
.skeleton:nth-child(3) { animation-delay: 300ms;  }
```

**Think about it**: skeleton loaders use `opacity` for the pulse, not `background-color`. Why? `opacity` changes are composited (GPU-handled) while `background-color` changes trigger repaint. The `opacity` pulse is always smooth; the `background-color` version can jank on slow devices.

---

## Exercises

1. **Build `bounceIn`**: An element that scales from 0 to 1.1 (overshoot) then settles at 1. Use `@keyframes` with four stops: 0% (scale 0), 60% (scale 1.1), 80% (scale 0.95), 100% (scale 1).

2. **Build `typewriter`**: Text that appears character by character. Use `overflow: hidden`, `white-space: nowrap`, and animate `width` from 0 to the text width using `steps(n, end)` timing — one step per character.

3. **Scroll-triggered animation**: Elements that animate into view when they enter the viewport. Use `IntersectionObserver` to add an `.is-visible` class when the element becomes visible. Define animations on `.animate-on-scroll.is-visible`.

4. **Staggered list**: A `<ul>` of items that slide in from the left, each delayed by 60ms from the previous. Use `--stagger` CSS custom property and `calc()` for the delay. Items should stay hidden (opacity 0, translateX -20px) before the animation.

5. **Animated loading spinner**: A spinner that uses only CSS — no border tricks, but a proper arc that grows and shrinks. Use two nested elements with different animation durations and directions to create a complex spinning effect.

---

## Solutions

### Exercise 1 — `bounceIn`

```css
@keyframes bounceIn {
  0%   { transform: scale(0);    opacity: 0; }
  60%  { transform: scale(1.1);  opacity: 1; }
  80%  { transform: scale(0.95); }
  100% { transform: scale(1); }
}

.animate-bounce-in {
  animation: bounceIn 0.6s cubic-bezier(0.34, 1.56, 0.64, 1) both;
}
```

### Exercise 2 — Typewriter effect

```css
@keyframes typing {
  from { width: 0; }
  to   { width: 100%; }
}

@keyframes blink {
  50% { border-color: transparent; }
}

.typewriter {
  overflow:    hidden;
  white-space: nowrap;
  border-right: 2px solid currentColor;
  width:       fit-content;

  animation:
    typing 3s steps(30, end) both,  /* 30 = number of characters */
    blink  0.75s step-end infinite; /* cursor blink */
}
```

### Exercise 3 — Scroll-triggered animation

```css
/* Default: hidden */
.animate-on-scroll {
  opacity:   0;
  transform: translateY(20px);
  transition: opacity 0.5s ease, transform 0.5s ease;
}

/* Visible: animated in */
.animate-on-scroll.is-visible {
  opacity:   1;
  transform: translateY(0);
}
```

```javascript
const observer = new IntersectionObserver(
  (entries) => entries.forEach(entry => {
    if (entry.isIntersecting) {
      entry.target.classList.add('is-visible');
      observer.unobserve(entry.target);  /* only animate once */
    }
  }),
  { threshold: 0.1 }
);

document.querySelectorAll('.animate-on-scroll').forEach(el => observer.observe(el));
```

### Exercise 4 — Staggered list

```css
.stagger-item {
  opacity:   0;
  transform: translateX(-20px);
  animation: slideInLeft 0.4s ease both;
  animation-delay: calc(var(--i, 0) * 60ms);
}

@keyframes slideInLeft {
  to { opacity: 1; transform: translateX(0); }
}
```

```javascript
document.querySelectorAll('.stagger-item').forEach((el, i) => {
  el.style.setProperty('--i', i);
});
```

### Exercise 5 — Complex spinner

```css
.spinner {
  width:         40px; height: 40px;
  border-radius: 50%;
  border:        3px solid #e2e8f0;
  border-top-color: #4361ee;
  animation:     spin 0.8s linear infinite;
}

/* More complex: arc that grows/shrinks */
.spinner-arc {
  width:  40px; height: 40px;
  animation: rotate 1.4s linear infinite;
}

.spinner-arc::before {
  content:      '';
  display:      block;
  width:        100%; height: 100%;
  border-radius: 50%;
  border:        3px solid transparent;
  border-top-color: #4361ee;
  animation:    stretch 1.4s ease-in-out infinite;
}

@keyframes rotate  { to { transform: rotate(360deg); } }
@keyframes stretch {
  0%   { transform: rotate(-135deg); }
  50%  { transform: rotate(135deg);  }
  100% { transform: rotate(225deg);  }
}
```

---

## What You Learned

| Concept | Key point |
|---------|-----------|
| `transition` | Smooths property changes triggered by state changes (hover, class) |
| `@keyframes` | Names a sequence of states; referenced by `animation-name` |
| `animation` shorthand | `name duration timing delay iterations direction fill-mode` |
| `fill-mode: both` | Apply first keyframe before start; hold last keyframe after end |
| `transform` | Moves, scales, rotates without affecting layout flow |
| `will-change` | Promotes element to GPU layer; use sparingly |
| Easing functions | `cubic-bezier(x1,y1,x2,y2)` — control points for acceleration curve |
| `prefers-reduced-motion` | OS accessibility setting; disable/simplify animations when set |
| Stagger with `--i` | `calc(var(--i) * 60ms)` for cascading delays |
| `IntersectionObserver` | Trigger animations when elements enter viewport |
| Compositor thread | GPU handles `opacity` + `transform` without touching main thread |

---

## Building with Claude

Bad prompt:
> "How do I make CSS animations?"

Good prompt:
> "I'm building an animation library. I have `@keyframes slideInUp { from { transform: translateY(20px); opacity: 0 } to { transform: translateY(0); opacity: 1 } }` with `.animate-slide-up { animation: slideInUp 0.4s ease both }`. I want to trigger this when a button is clicked — remove the class, force a reflow, then re-add it. I use `el.classList.remove('animate-slide-up'); el.offsetWidth; el.classList.add('animate-slide-up')`. The `offsetWidth` reflow trick works in Chrome but I've read it's not in the spec — is there a cleaner way to restart a CSS animation that works reliably cross-browser? Should I use the Web Animations API `el.getAnimations()[0].cancel()` + `.play()` instead?"
