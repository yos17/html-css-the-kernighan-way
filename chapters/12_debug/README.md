# Chapter 12 — Build Your Own CSS Debug Tool

Every CSS bug comes down to one of five things: unexpected inheritance, a specificity conflict, a collapsed dimension, overflow clipping, or stacking context confusion. That is the complete list. CSS does not have mysterious undefined behavior — it has a deterministic cascade, a precise box model, and a well-specified stacking algorithm. When something looks wrong, the cause is always in one of those five categories, and the fix requires understanding which one you are dealing with.

This chapter builds `debug.css`, a CSS debugging toolkit. Starting from the classic `* { outline: 1px solid red }` one-liner, we will build a systematic visual layer that makes invisible structure visible — collapsed elements, overflowing content, negative margins, z-index relationships, and stacking contexts. Then we build a JavaScript-powered inspector: hover any element to see its computed box model, its inherited properties, its specificity chain, and the CSS rules applying to it — the same information the browser's DevTools provides, embedded directly in your page.

The skill you are building is not "how to fix CSS." It is how to diagnose CSS — how to look at a broken layout and immediately identify which of the five categories the bug belongs to, then apply the right investigation technique. Debugging CSS without this mental model is guessing. With it, it is engineering.

## The Problem

The naive approach to CSS debugging is trial and error: change a value, refresh, change it back, try something else. This works for trivial problems and fails for everything else. A collapsed height that is caused by a parent with `overflow: hidden` creating a new block formatting context — you cannot fix that by guessing at heights. A z-index that does not work because the element's parent has a lower stacking context — you cannot fix that by increasing `z-index` to 9999. Guessing escalates the problem rather than solving it.

The second naive approach is to immediately open the browser's DevTools and poke at computed styles without a framework for what you are looking for. DevTools is powerful, but it surfaces data without a diagnostic process. You see a hundred computed properties — which one is wrong? You see a specificity chain — but is the specificity the problem, or is it inheritance? Without a mental model, DevTools is a pile of numbers.

The real problem is that most CSS developers have never built a complete mental model of the box model, the stacking context, and the formatting context. These are the foundational mechanisms that determine how every element is sized and positioned. Understanding them precisely — not approximately — is what separates developers who diagnose CSS bugs systematically from developers who fix them by accident.

## Building It Step by Step

### v1: The Classic Debug Pattern — Outlines

The oldest CSS debugging technique: add a visible stroke to every element. Always `outline`, never `border`.

```css
/* v1: debug.css — the classic pattern */

/* The one-liner */
* { outline: 1px solid red !important; }

/* Why outline, not border:
   border affects box model — it adds to width/height (or inward with border-box)
   outline is outside the border box — it does not affect layout AT ALL */

/* Better: color-coded by depth */
.debug * { outline: 1px solid hsla(0,   90%, 60%, 0.5); }  /* 1 deep — red */
.debug * * { outline: 1px solid hsla(60,  90%, 60%, 0.5); }  /* 2 deep — yellow */
.debug * * * { outline: 1px solid hsla(120, 90%, 60%, 0.5); }  /* 3 deep — green */
.debug * * * * { outline: 1px solid hsla(200, 90%, 60%, 0.5); }  /* 4 deep — blue */
.debug * * * * * { outline: 1px solid hsla(280, 90%, 60%, 0.5); }  /* 5 deep — violet */

/* Box model visualization classes */
.debug-box-model * {
  background-clip: content-box !important;
  outline: 2px solid rgba(255, 100, 100, 0.6) !important;  /* border */
}

/* Highlight specific box model regions */
.debug-margins * {
  background: rgba(255, 165, 0, 0.1) !important;           /* orange = margin */
  outline: 1px dashed orange !important;
}

.debug-padding * {
  background: rgba(0, 200, 100, 0.15) !important;           /* green = padding */
  outline: 1px dashed green !important;
}
```

The `outline` vs `border` distinction is not a preference. It is a fact: `outline` is drawn outside the border box, does not participate in the document flow, and does not affect layout. Adding `outline: 1px solid red` to a precisely sized element will not break it. Adding `border: 1px solid red` to an element with `box-sizing: content-box` will.

### v2: Systematic Debug Layer

A more targeted debug layer: visual indicators for the specific problems that are hardest to see.

```css
/* v2: debug.css — systematic indicators */

/* ─── Collapsed elements ─── */
/* Zero-height elements are invisible but take up space */
[debug-mode] *:not([hidden]) {
  min-height: 1px !important;  /* force visible even if collapsed */
}

/* Flag elements that have content but zero computed height */
[debug-mode] .dbg-collapsed {
  outline: 3px dashed hsl(0 80% 60%) !important;
  background: hsla(0, 80%, 60%, 0.1) !important;
}
[debug-mode] .dbg-collapsed::before {
  content: '⚠ collapsed (0px height)';
  display: block;
  font-size: 10px;
  background: hsl(0 80% 60%);
  color: white;
  padding: 2px 6px;
}

/* ─── Overflow indicators ─── */
[debug-mode] .dbg-overflow {
  outline: 3px solid hsl(40 90% 55%) !important;
}
[debug-mode] .dbg-overflow::after {
  content: '⚠ overflow';
  position: fixed;
  background: hsl(40 90% 55%);
  color: black;
  font-size: 10px;
  padding: 2px 6px;
  z-index: 99999;
  pointer-events: none;
}

/* ─── Negative margin warning ─── */
[debug-mode] .dbg-negative-margin {
  outline: 3px dotted hsl(280 80% 65%) !important;
  background: hsla(280, 80%, 65%, 0.1) !important;
}

/* ─── Stacking context visualization ─── */
/* Stacking context creators: opacity<1, transform, filter, isolation,
   z-index on positioned, will-change */
[debug-mode] .dbg-stacking-context {
  outline: 2px solid hsl(200 80% 55%) !important;
  outline-offset: 2px;
}
[debug-mode] .dbg-stacking-context::before {
  content: '📦 stacking context';
  position: absolute;
  top: 0; left: 0;
  background: hsl(200 80% 55%);
  color: white;
  font-size: 9px;
  padding: 1px 5px;
  z-index: 99999;
  pointer-events: none;
}

/* ─── Z-index heat map ─── */
[debug-mode] [style*="z-index"] {
  outline: 2px solid hsl(200 80% 55%);
}
```

The stacking context detection is the most valuable part of this layer. Most `z-index` bugs are actually stacking context containment bugs — an element with `z-index: 100` that lives inside a parent with `z-index: 1` will never appear above an element that lives at the root stacking context, regardless of its `z-index` value. Making stacking contexts visible makes these bugs diagnosable in seconds.

### v3: JavaScript-Powered Inspector

A hover inspector that reads computed styles and presents them in a structured overlay — the same information as DevTools, embedded in the page.

```javascript
/* v3: The CSS inspector */

class CSSInspector {
  constructor() {
    this.panel = null;
    this.active = false;
    this.lastTarget = null;
  }

  enable() {
    this.active = true;
    this.panel = this.createPanel();
    document.body.appendChild(this.panel);
    document.addEventListener('mousemove', this.onMove.bind(this));
    document.addEventListener('keydown', e => {
      if (e.key === 'Escape') this.disable();
    });
  }

  disable() {
    this.active = false;
    if (this.panel) this.panel.remove();
    document.removeEventListener('mousemove', this.onMove.bind(this));
  }

  onMove(e) {
    const target = document.elementFromPoint(e.clientX, e.clientY);
    if (!target || target === this.panel || this.panel.contains(target)) return;
    if (target === this.lastTarget) return;
    this.lastTarget = target;
    this.inspect(target, e);
  }

  inspect(el, e) {
    const computed = getComputedStyle(el);
    const rect = el.getBoundingClientRect();

    this.panel.innerHTML = this.renderBoxModel(el, computed, rect)
      + this.renderDimensions(el, computed, rect)
      + this.renderStackingContext(computed)
      + this.renderInherited(computed)
      + this.renderSelector(el);

    // Position panel avoiding viewport edges
    const px = Math.min(e.clientX + 16, window.innerWidth - 320);
    const py = Math.min(e.clientY + 16, window.innerHeight - 400);
    this.panel.style.left = px + 'px';
    this.panel.style.top  = py + 'px';
  }

  renderBoxModel(el, computed, rect) {
    const mt = parseFloat(computed.marginTop);
    const mr = parseFloat(computed.marginRight);
    const mb = parseFloat(computed.marginBottom);
    const ml = parseFloat(computed.marginLeft);
    const bt = parseFloat(computed.borderTopWidth);
    const br = parseFloat(computed.borderRightWidth);
    const bb = parseFloat(computed.borderBottomWidth);
    const bl = parseFloat(computed.borderLeftWidth);
    const pt = parseFloat(computed.paddingTop);
    const pr = parseFloat(computed.paddingRight);
    const pb = parseFloat(computed.paddingBottom);
    const pl = parseFloat(computed.paddingLeft);
    const w = rect.width;
    const h = rect.height;

    return `<div class="dbg-section">
      <div class="dbg-label">Box Model</div>
      <div class="box-model-viz">
        <div class="bm-margin">
          margin: ${mt}↑ ${mr}→ ${mb}↓ ${ml}←
          <div class="bm-border">
            border: ${bt}↑ ${br}→ ${bb}↓ ${bl}←
            <div class="bm-padding">
              padding: ${pt}↑ ${pr}→ ${pb}↓ ${pl}←
              <div class="bm-content">
                ${Math.round(w - bl - br - pl - pr)}×${Math.round(h - bt - bb - pt - pb)}
              </div>
            </div>
          </div>
        </div>
      </div>
    </div>`;
  }

  renderDimensions(el, computed, rect) {
    const tagName = el.tagName.toLowerCase();
    const id = el.id ? '#' + el.id : '';
    const cls = [...el.classList].slice(0, 3).map(c => '.' + c).join('');

    return `<div class="dbg-section">
      <div class="dbg-label">Dimensions</div>
      <div class="dbg-row"><span>offsetWidth</span><span>${el.offsetWidth}px</span></div>
      <div class="dbg-row"><span>offsetHeight</span><span>${el.offsetHeight}px</span></div>
      <div class="dbg-row"><span>scrollWidth</span><span>${el.scrollWidth}px</span></div>
      <div class="dbg-row"><span>display</span><span>${computed.display}</span></div>
      <div class="dbg-row"><span>position</span><span>${computed.position}</span></div>
    </div>`;
  }

  renderStackingContext(computed) {
    const creators = [];
    if (parseFloat(computed.opacity) < 1) creators.push(`opacity: ${computed.opacity}`);
    if (computed.transform !== 'none') creators.push('transform');
    if (computed.filter !== 'none') creators.push(`filter: ${computed.filter}`);
    if (computed.isolation === 'isolate') creators.push('isolation: isolate');
    if (computed.willChange !== 'auto') creators.push(`will-change: ${computed.willChange}`);
    if (computed.zIndex !== 'auto' && computed.position !== 'static')
      creators.push(`z-index: ${computed.zIndex}`);

    const isCtx = creators.length > 0;

    return `<div class="dbg-section">
      <div class="dbg-label">Stacking</div>
      <div class="dbg-row">
        <span>z-index</span>
        <span>${computed.zIndex}</span>
      </div>
      <div class="dbg-row">
        <span>stacking context</span>
        <span style="color:${isCtx ? '#a8d8ea' : '#7a7a9a'}">${isCtx ? '✓ YES' : 'no'}</span>
      </div>
      ${creators.map(c => `<div class="dbg-row dbg-reason"><span></span><span>${c}</span></div>`).join('')}
    </div>`;
  }

  renderInherited(computed) {
    const props = ['color', 'font-family', 'font-size', 'font-weight', 'line-height'];
    return `<div class="dbg-section">
      <div class="dbg-label">Computed</div>
      ${props.map(p => `
        <div class="dbg-row">
          <span>${p}</span>
          <span>${computed[p.replace(/-([a-z])/g, m => m[1].toUpperCase())]}</span>
        </div>`).join('')}
    </div>`;
  }

  renderSelector(el) {
    const tag = el.tagName.toLowerCase();
    const id  = el.id ? `#${el.id}` : '';
    const cls = [...el.classList].slice(0, 4).map(c => `.${c}`).join('');
    return `<div class="dbg-section">
      <div class="dbg-label">Element</div>
      <div class="dbg-row"><span>tag</span><span>&lt;${tag}&gt;</span></div>
      ${id ? `<div class="dbg-row"><span>id</span><span>${id}</span></div>` : ''}
      ${cls ? `<div class="dbg-row"><span>classes</span><span>${cls}</span></div>` : ''}
    </div>`;
  }

  createPanel() {
    const p = document.createElement('div');
    p.id = 'css-inspector-panel';
    p.style.cssText = `
      position: fixed; z-index: 2147483647;
      width: 300px; max-height: 80vh; overflow-y: auto;
      background: #0d0d1a; border: 1px solid #a8d8ea;
      border-radius: 6px; font-family: 'Courier New', monospace;
      font-size: 11px; color: #c8c8d8; box-shadow: 0 8px 32px rgba(0,0,0,0.6);
      pointer-events: none;
    `;
    return p;
  }
}
```

## The Complete Program

The full toolkit is in `debug.html`. Here is the complete `debug.css` listing:

```css
/* debug.css — v3 complete */

/* ─── 1. Global outline mode ─── */
[debug-outlines] * {
  outline: 1px solid hsla(0, 90%, 60%, 0.6) !important;
}
[debug-outlines] * * {
  outline: 1px solid hsla(60, 90%, 60%, 0.6) !important;
}
[debug-outlines] * * * {
  outline: 1px solid hsla(120, 90%, 60%, 0.6) !important;
}
[debug-outlines] * * * * {
  outline: 1px solid hsla(200, 90%, 60%, 0.6) !important;
}
[debug-outlines] * * * * * {
  outline: 1px solid hsla(280, 90%, 60%, 0.6) !important;
}

/* ─── 2. Layout violations ─── */
[debug-layout] img:not([alt]) {
  outline: 4px solid red !important;
}
[debug-layout] img[alt=""] {
  outline: 4px solid orange !important;
}
[debug-layout] :empty:not(input):not(hr):not(br):not(img):not(svg) {
  outline: 2px dashed hsl(280 80% 65%) !important;
  min-height: 8px !important;
}

/* ─── 3. Overflow flag ─── */
[debug-overflow] * {
  max-height: none !important;
  overflow: visible !important;
}
```

## Walkthrough

### The Computed Box Model

The browser has three different "widths" for an element. Confusing them is the source of many layout bugs.

```
┌─────────────────────────────────────────────────────────────────────┐
│  offsetWidth — total visible width including border                  │
│  ┌──────────────────────────────────────────────────────────────┐   │
│  │  margin (not included in offsetWidth)                        │   │
│  │  ┌────────────────────────────────────────────────────────┐  │   │
│  │  │  BORDER WIDTH                                          │  │   │
│  │  │  ┌──────────────────────────────────────────────────┐  │  │   │
│  │  │  │  padding                                         │  │  │   │
│  │  │  │  ┌────────────────────────────────────────────┐  │  │  │   │
│  │  │  │  │  content                                   │  │  │  │   │
│  │  │  │  │                                            │  │  │  │   │
│  │  │  │  │  clientWidth = content + padding           │  │  │  │   │
│  │  │  │  │  scrollWidth = content (even if overflow)  │  │  │  │   │
│  │  │  │  └────────────────────────────────────────────┘  │  │  │   │
│  │  │  └──────────────────────────────────────────────────┘  │  │   │
│  │  └────────────────────────────────────────────────────────┘  │   │
│  └──────────────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────────────┘

offsetWidth  = border + padding + content width
clientWidth  = padding + content width (no border, no scrollbar)
scrollWidth  = content width including overflow (no padding, no border)

scrollWidth > clientWidth  →  content is overflowing
```

The most common confusion: `offsetWidth` includes border, `clientWidth` does not. When you want "how wide is the content area including padding," use `clientWidth`. When you want "how wide is the element on screen," use `offsetWidth`. When you want "how wide is the scrollable content," use `scrollWidth`.

### Why outline Does Not Affect Layout

`outline` is drawn outside the border box. It is not part of the box model at all.

```
Without outline:
  ┌──────────────────┐
  │  margin          │
  │  ┌────────────┐  │
  │  │  border    │  │  ← offsetWidth includes this
  │  │  ┌──────┐  │  │
  │  │  │  p   │  │  │
  │  │  └──────┘  │  │
  │  └────────────┘  │
  └──────────────────┘

With outline: 2px solid red (outline-offset: 0)
               ┌────────────┐   ← outline drawn HERE, outside border box
             ┌──────────────┐
             │  border      │  ← offsetWidth unchanged
             │  ┌──────┐    │
             │  │  p   │    │
             │  └──────┘    │
             └──────────────┘

With border: 2px solid red instead:
  ┌──────────────────────┐
  │  ┌─────────────────┐ │  ← border box is now 4px wider
  │  │  content + pad  │ │  ← layout shifts for all siblings
  │  └─────────────────┘ │
  └──────────────────────┘
```

Always use `outline` for debug visualization. `border` modifies layout. `outline` does not.

### Collapsed Margins — When and Why

Vertical margins collapse in three specific situations. This is intentional CSS behavior, not a bug — but it creates unexpected gaps and is a common debugging trap.

```
SITUATION 1: Adjacent siblings
  ┌────────────────────┐
  │  paragraph 1       │ margin-bottom: 20px
  └────────────────────┘
         ↕ 20px  ← NOT 40px! The 20px margins collapse into one.
  ┌────────────────────┐
  │  paragraph 2       │ margin-top: 20px
  └────────────────────┘

SITUATION 2: Parent and first/last child (no border/padding between them)
  ┌── .parent (margin-top: 0) ──────────────────────┐
  │  ┌── .child (margin-top: 30px) ──────────────┐  │
  │  │  content                                  │  │
  │  └───────────────────────────────────────────┘  │
  └─────────────────────────────────────────────────┘
  Result: .parent appears to have margin-top: 30px
          The child's margin "escapes" the parent.
  Fix: add any border, padding, or overflow ≠ visible to .parent

SITUATION 3: Empty elements
  An element with no height and equal top/bottom margins:
  margin-top: 20px + margin-bottom: 20px → becomes 20px, not 40px
```

Margin collapse does NOT happen:
- For horizontal margins (left/right)
- For flex or grid children
- When there is border or padding between parent and child
- When `overflow` is not `visible` on the parent
- For absolutely or fixed positioned elements

### overflow: hidden Creates a New Block Formatting Context

This is the most surprising CSS side effect. `overflow: hidden` does two things: clips content that overflows, and creates a new Block Formatting Context (BFC). The BFC is what prevents margin collapse with children and makes the element contain its floated children.

```
WITHOUT overflow: hidden (or other BFC trigger):
  ┌── .parent ─────────────────────────────────────────┐
  │  (height: 0 — does not contain the float!)         │
  └────────────────────────────────────────────────────┘
  ████████████ .float (height: 100px, floating)
  
  .parent does not wrap around .float — classic clearfix bug

WITH overflow: hidden (triggers BFC):
  ┌── .parent (BFC) ───────────────────────────────────┐
  │  ████████████ .float contained inside parent       │
  │  parent height = 100px (wraps the float)           │
  └────────────────────────────────────────────────────┘

Other BFC triggers (same effect on floats and margin collapse):
  overflow: auto | scroll    display: flow-root (cleanest)
  display: flex | grid       contain: layout | paint | strict
  position: absolute | fixed float ≠ none
```

`display: flow-root` is the modern, explicit way to create a BFC without the clipping side effect of `overflow: hidden`. Use it when you want "contain children and clear floats" without accidentally hiding overflow.

### The Stacking Context Checklist

Z-index only works within a stacking context. Elements in different stacking contexts cannot be interleaved — a child in a lower-priority stacking context can never appear above an element in a higher-priority one, regardless of z-index value.

```
STACKING CONTEXT CREATORS (complete list):
  ┌──────────────────────────────────────────────────────────────┐
  │  opacity < 1                                                 │
  │  transform ≠ none                                            │
  │  filter ≠ none                                               │
  │  backdrop-filter (any value)                                 │
  │  perspective ≠ none                                          │
  │  clip-path ≠ none                                            │
  │  mask / mask-image / mask-border (any value)                 │
  │  isolation: isolate                                          │
  │  mix-blend-mode ≠ normal                                     │
  │  will-change: transform | opacity | filter | etc.            │
  │  contain: layout | paint | strict | content                  │
  │  z-index ≠ auto on a positioned element (pos ≠ static)       │
  │  position: fixed or sticky (always creates stacking context) │
  └──────────────────────────────────────────────────────────────┘

THE z-index CONTAINMENT BUG:
  <div style="position:relative; z-index:1; transform:rotate(0)"> ← stacking context!
    <div style="z-index:9999">I am trapped inside my parent's context</div>
  </div>
  <div style="position:relative; z-index:2">
    I appear above everything in the first div, even z-index:9999
  </div>

  FIX: Remove the stacking context trigger from the parent,
       or move the high-z-index element out of the context.
```

The most common accidental stacking context trigger is `transform: translateZ(0)` or `will-change: transform` added for animation performance. Adding either creates a stacking context, which can trap children with high z-index values.

### Browser DevTools Computed Panel

The browser's computed styles panel tells you the resolved value of every property — after the cascade has been applied, after inheritance, after `var()` resolution. How to read it:

```
Computed panel shows:
  property: resolved-value   ← the final value applied to this element

  If you click the value, it shows:
  ┌─ Which rule set this value ─────────────────────────────────────┐
  │  selector (specificity)  filename.css:line                      │
  │  ↳  .other-selector      other.css:42  (overridden, struck out)  │
  └──────────────────────────────────────────────────────────────────┘

  Struck-through rules were overridden. They applied but lost.

  "Inherited from: .parent-class" means the value came via
  inheritance, not from a rule targeting this element directly.

  The value shows "ua stylesheet" for browser default styles.
```

When debugging a specificity conflict, look for struck-through values in the computed panel. The rule you want is losing to one with higher specificity. The fix is either to increase the specificity of your rule, decrease the specificity of the competing rule, or use `@layer` to establish explicit priority.

## Guided Try It — Diagnose a Broken Layout

You have a modal dialog that appears behind the page header despite having `z-index: 1000`. This is the classic stacking context containment bug.

### Step 1: Enable the inspector

```javascript
const inspector = new CSSInspector();
inspector.enable();
```

Hover the modal. The inspector panel shows "stacking context: YES" on an ancestor element. The reason: one of the ancestor elements has `transform: translateZ(0)` — a common GPU promotion trick that accidentally creates a stacking context.

### Step 2: Identify the containing stacking context

```javascript
function findStackingContextAncestors(el) {
  const results = [];
  let node = el.parentElement;
  while (node && node !== document.body) {
    const cs = getComputedStyle(node);
    const creates =
      parseFloat(cs.opacity) < 1 ||
      cs.transform !== 'none' ||
      cs.filter !== 'none' ||
      cs.isolation === 'isolate' ||
      cs.willChange !== 'auto' ||
      (cs.zIndex !== 'auto' && cs.position !== 'static') ||
      cs.position === 'fixed' ||
      cs.position === 'sticky';
    if (creates) {
      results.push({ el: node, reason: getContextReason(cs) });
    }
    node = node.parentElement;
  }
  return results;
}
```

### Step 3: Fix options

```
Option A: Remove the stacking context trigger from the ancestor
  .hero { transform: translateZ(0); }  →  delete this line
  .hero { will-change: transform; }    →  will-change: auto

Option B: Move the modal outside the stacking context
  document.body.appendChild(modalEl);  // portal pattern

Option C: Explicit isolation
  .modal-container { isolation: isolate; }
  // Creates an intentional stacking context for the modal,
  // separate from the hero's context
```

Think about it: `position: fixed` creates a stacking context on every element, but it does not respect a transformed ancestor. A `position: fixed` element inside a `transform ≠ none` parent is actually positioned relative to that parent, not the viewport. Why does this happen? What does it tell you about the relationship between the coordinate system and the stacking context?

## Exercises

**1.** Write a `findCollapsedMargins()` JavaScript function that walks the DOM and identifies sibling elements where adjacent margins are collapsing (i.e., both have non-zero `margin-bottom`/`margin-top` that are collapsing, not flowing). Flag the pairs visually with an orange outline.

**2.** Build a `z-index map`: a floating overlay that, when activated, shows a labeled indicator on every element with a non-auto `z-index`, color-coded by value range (0–9 = gray, 10–99 = blue, 100–999 = orange, 1000+ = red). Each indicator shows the element's selector and z-index value.

**3.** Implement a `specificity calculator` function: given a CSS selector string, return its specificity as `[a, b, c]` where `a` = id count, `b` = class/attribute/pseudo-class count, `c` = element/pseudo-element count. Handle `:is()` (takes max specificity of arguments), `:not()`, `:where()` (always zero), and `*` (zero specificity).

**4.** Build a `BFC detector`: a function that returns all Block Formatting Context creators in the DOM — elements whose `overflow`, `display`, `position`, `float`, or `contain` property creates a BFC. Display them as highlighted regions with a label showing which property triggered the BFC.

**5.** Create a "layout shift debugger": a `PerformanceObserver` that watches for layout shift entries (`layout-shift`), then highlights the shifted elements with a red flash and logs the shift score, the impacted nodes, and which element triggered the shift. Use `entry.sources` to identify the elements involved.

## Solutions

### Exercise 1 — Collapsed Margin Finder

```javascript
function findCollapsedMargins() {
  const elements = [...document.querySelectorAll('*')];
  const pairs = [];

  for (let i = 0; i < elements.length - 1; i++) {
    const a = elements[i];
    const b = elements[i + 1];

    // Only adjacent siblings collapse
    if (a.parentElement !== b.parentElement) continue;

    const csA = getComputedStyle(a);
    const csB = getComputedStyle(b);

    const marginBottom = parseFloat(csA.marginBottom);
    const marginTop    = parseFloat(csB.marginTop);

    if (marginBottom > 0 && marginTop > 0) {
      // Check if they are in normal flow (not flex/grid children)
      const parent = getComputedStyle(a.parentElement);
      if (parent.display !== 'flex' && parent.display !== 'grid') {
        pairs.push({ a, b, marginBottom, marginTop,
          collapsed: Math.max(marginBottom, marginTop) });
        a.style.outline = '2px dashed orange';
        b.style.outline = '2px dashed orange';
      }
    }
  }
  return pairs;
}
```

### Exercise 2 — Z-Index Map

```javascript
function buildZIndexMap() {
  const positioned = [...document.querySelectorAll('*')].filter(el => {
    const cs = getComputedStyle(el);
    return cs.position !== 'static' && cs.zIndex !== 'auto';
  });

  const overlay = document.createElement('div');
  overlay.style.cssText = 'position:fixed;inset:0;pointer-events:none;z-index:2147483647';

  positioned.forEach(el => {
    const rect = el.getBoundingClientRect();
    const z = parseInt(getComputedStyle(el).zIndex);
    const color = z >= 1000 ? '#e74c3c' : z >= 100 ? '#f39c12' : z >= 10 ? '#3498db' : '#888';

    const badge = document.createElement('div');
    badge.style.cssText = `
      position:absolute;
      left:${rect.left}px; top:${rect.top}px;
      background:${color}; color:white;
      font:bold 10px monospace; padding:2px 5px;
      border-radius:3px; z-index:2147483647;
      pointer-events:none;
    `;
    badge.textContent = `z:${z}`;
    overlay.appendChild(badge);
  });

  document.body.appendChild(overlay);
  return overlay;
}
```

### Exercise 3 — Specificity Calculator

```javascript
function calcSpecificity(selector) {
  let a = 0, b = 0, c = 0;

  // Remove :where() — zero specificity
  selector = selector.replace(/:where\([^)]*\)/g, '');

  // Count IDs
  a += (selector.match(/#[\w-]+/g) || []).length;
  selector = selector.replace(/#[\w-]+/g, '');

  // :not(), :is() — take max of arguments (simplified: count contents)
  const notIs = selector.matchAll(/:(?:not|is)\(([^)]+)\)/g);
  for (const m of notIs) {
    const inner = calcSpecificity(m[1]);
    b += inner[1]; c += inner[2];
  }
  selector = selector.replace(/:(?:not|is)\([^)]*\)/g, '');

  // Classes, attributes, pseudo-classes
  b += (selector.match(/\.[\w-]+|\[[^\]]+\]|:[\w-]+/g) || []).length;
  selector = selector.replace(/\.[\w-]+|\[[^\]]+\]|:[\w-]+/g, '');

  // Elements and pseudo-elements
  c += (selector.match(/(?:^|[\s>~+])[\w-]+|::[\w-]+/g) || []).length;

  // Universal selector: zero specificity
  return [a, b, c];
}

// Tests:
console.log(calcSpecificity('#nav .item'));         // [1, 1, 0]
console.log(calcSpecificity('.card .title'));        // [0, 2, 0]
console.log(calcSpecificity(':is(.a, .b) h1'));      // [0, 1, 1]
console.log(calcSpecificity(':where(.a, #id) h1')); // [0, 0, 1]
```

### Exercise 4 — BFC Detector

```javascript
function findBFCCreators() {
  const results = [];
  const all = document.querySelectorAll('*');

  all.forEach(el => {
    const cs = getComputedStyle(el);
    const reasons = [];

    if (!['visible', 'clip'].includes(cs.overflow))   reasons.push(`overflow:${cs.overflow}`);
    if (cs.display === 'flow-root')                    reasons.push('display:flow-root');
    if (['flex','grid','inline-flex','inline-grid']
        .includes(cs.display))                        reasons.push(`display:${cs.display}`);
    if (cs.float !== 'none')                          reasons.push(`float:${cs.float}`);
    if (['absolute','fixed'].includes(cs.position))   reasons.push(`position:${cs.position}`);
    if (cs.contain && cs.contain !== 'none')          reasons.push(`contain:${cs.contain}`);

    if (reasons.length) {
      results.push({ el, reasons });
      el.setAttribute('data-bfc', reasons.join(', '));
    }
  });

  return results;
}
```

### Exercise 5 — Layout Shift Debugger

```javascript
function initLayoutShiftDebugger() {
  if (!('PerformanceObserver' in window)) {
    console.warn('PerformanceObserver not available');
    return;
  }

  const observer = new PerformanceObserver(list => {
    for (const entry of list.getEntries()) {
      if (entry.entryType !== 'layout-shift' || entry.hadRecentInput) continue;

      console.group(`Layout Shift — score: ${entry.value.toFixed(4)}`);
      console.log('Time:', entry.startTime.toFixed(1) + 'ms');

      if (entry.sources) {
        entry.sources.forEach((source, i) => {
          console.log(`Source ${i}:`, source.node);
          console.log('  Previous rect:', source.previousRect);
          console.log('  Current rect:', source.currentRect);

          // Flash the shifted element
          if (source.node && source.node.style) {
            const orig = source.node.style.outline;
            source.node.style.outline = '3px solid red';
            setTimeout(() => { source.node.style.outline = orig; }, 1000);
          }
        });
      }
      console.groupEnd();
    }
  });

  observer.observe({ type: 'layout-shift', buffered: true });
  return observer;
}
```

## What You Learned

| Concept | Key point |
|---|---|
| outline vs border for debugging | `outline` is outside the box model and does not affect layout. Always use it for debug visualization |
| offsetWidth vs clientWidth vs scrollWidth | offsetWidth includes border; clientWidth includes padding but not border; scrollWidth is the full content width including overflow |
| Collapsed margins | Adjacent sibling margins collapse. Parent-child margins collapse unless border/padding/overflow separates them. Does not apply to flex/grid children |
| overflow: hidden creates BFC | Side effect beyond clipping: contains floats, prevents margin collapse with children. Use `display: flow-root` for the BFC without clipping |
| Stacking context checklist | opacity<1, transform, filter, isolation, will-change, z-index on positioned — all create new stacking contexts |
| z-index containment | Elements in different stacking contexts cannot interleave. A z-index:9999 in a low-priority context loses to z-index:1 in a higher context |
| transform creates stacking context | `transform: translateZ(0)` (GPU promotion trick) accidentally traps children in a stacking context |
| getComputedStyle | Returns resolved values — after cascade, inheritance, and var() resolution |
| position: fixed + transform | A fixed element inside a transformed ancestor is positioned relative to that ancestor, not the viewport |
| PerformanceObserver layout-shift | Can identify which elements caused Cumulative Layout Shift (CLS) with `entry.sources` |

## Building with Claude

Bad prompt:
> "Why is my z-index not working?"

This produces a generic list of z-index facts without diagnosing your specific problem. You need to provide context.

Good prompt:
> "My modal has z-index: 1000 and position: fixed, but it appears behind my sticky header which has z-index: 100. I have a hero section with transform: translateZ(0) for GPU promotion. Using the stacking context checklist from Chapter 12's debug tool, explain: which property on which element is creating a stacking context that contains the modal, why the z-index values are irrelevant in this case, and give me two concrete fixes — one that removes the stacking context and one that uses the portal pattern to move the modal outside the context."

This produces a diagnostic answer, not a generic explanation. Claude will identify `transform: translateZ(0)` as the stacking context creator, explain the context hierarchy, and give you concrete fixes for both approaches.

---

*Chapter 13's Debugger uses a JavaScript technique to inspect scope and call stack. This chapter is the CSS equivalent — building the inspection layer that makes CSS problems visible and diagnosable.*
