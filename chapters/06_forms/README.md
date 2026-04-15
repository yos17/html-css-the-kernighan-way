# Chapter 8 — Build Your Own Form Framework

Forms are where HTML becomes practical. Login forms, search boxes, contact forms, checkout pages, all of them depend on the same small set of elements. Beginners often style forms to look right without understanding what the browser already gives them for free. This chapter focuses on those built-in features first.

The simplest beginner rule of forms is: use the right HTML first, then style it.

If you use the right HTML elements, the browser already knows a lot: how labels connect to inputs, how keyboard focus should move, and how simple validation should work. Good form CSS starts by keeping that built-in behavior intact.

## The Problem

A lot of form frustration comes from fighting the browser instead of using what the browser already knows about forms.

A beginner often writes something like this:

```html
<div class="field">
  <span class="label">Email</span>
  <input class="input" type="text" placeholder="Enter your email">
</div>
```

Three things go wrong here:

1. `<span>` is not a real label, so the browser cannot connect it properly to the input.
2. `type="text"` throws away helpful browser behavior like email validation and better mobile keyboards.
3. There is no clear feedback when the value is wrong.

The correct version uses one line of HTML the browser already understands:

```html
<label for="email">Email</label>
<input id="email" type="email" required>
```

The `for`/`id` association activates click-to-focus, screen reader announcements, and the browser's label-to-input connection in the accessibility tree. `type="email"` activates email keyboard on mobile, format validation, and autofill. `required` activates the `:required` pseudo-class and native validation. No JavaScript, no framework.

In plain English: correct HTML gives you a lot of useful behavior for free.

## Building It Step by Step

### v1: Label/Input Pairing and Focus States

The foundation: correctly associated labels, properly typed inputs, and visible focus states.

```css
/* v1: form-basics.css */
.form-group {
  display: flex;
  flex-direction: column;
  gap: 0.375rem;
  margin-bottom: 1.25rem;
}

.form-label {
  font-size: 0.875rem;
  font-weight: 500;
  color: #c8c8d8;
}

.form-label--required::after {
  content: ' *';
  color: #f87171;
  font-weight: 400;
}

.form-input {
  width: 100%;
  padding: 0.5rem 0.75rem;
  background: #111122;
  color: #c8c8d8;
  border: 1px solid #2a2a40;
  border-radius: 6px;
  font-family: inherit;
  font-size: 0.875rem;
  line-height: 1.5;
  transition: border-color 0.15s, box-shadow 0.15s;
  -webkit-appearance: none;
  appearance: none;
}

.form-input::placeholder {
  color: #4a4a6a;
}

.form-input:focus {
  outline: none;
  border-color: #a8d8ea;
  box-shadow: 0 0 0 3px rgba(168, 216, 234, 0.2);
}
```

```html
<div class="form-group">
  <label class="form-label form-label--required" for="email">Email</label>
  <input class="form-input" type="email" id="email" name="email" required
         placeholder="you@example.com" autocomplete="email">
</div>
```

`-webkit-appearance: none; appearance: none` removes OS-level styling from inputs, giving you a consistent baseline across browsers.

The `:focus` ring uses `box-shadow` instead of `outline` for the same reason as buttons — the double-ring technique works on any background.

### v2: Validation States

This is one of the places where CSS starts to feel very practical. The browser already knows whether the input is valid. CSS can react to that state visually.

CSS has pseudo-classes for every state in the browser's form validation lifecycle.

```css
/* v2: validation styling */

/* Valid input — after the user has interacted */
.form-input:user-valid {
  border-color: #16a34a;
}

.form-input:user-invalid {
  border-color: #dc2626;
  box-shadow: 0 0 0 3px rgba(220, 38, 38, 0.15);
}

/* Error message — hidden by default */
.form-error {
  font-size: 0.8rem;
  color: #f87171;
  display: none;
  gap: 0.25rem;
  align-items: flex-start;
}

/* Show error when the input is invalid AND has been touched */
.form-input:user-invalid ~ .form-error {
  display: flex;
}

/* Required indicator in label */
.form-input:required ~ .form-label::after,
.form-label--required::after {
  content: ' *';
  color: #f87171;
}
```

`:user-valid` and `:user-invalid` are the correct selectors. The older `:valid` and `:invalid` apply immediately on page load — required fields are `:invalid` before the user has touched them, which looks like an error state on a fresh form. `:user-valid`/`:user-invalid` only apply after the user has actually interacted with the field.

```html
<div class="form-group">
  <label class="form-label" for="email">Email</label>
  <input class="form-input" type="email" id="email" required>
  <div class="form-error">
    <span>✕</span>
    <span>Please enter a valid email address.</span>
  </div>
</div>
```

The sibling selector `~` connects the input to the error message that follows it. No JavaScript — the browser's validation state drives the CSS.

### v3: Custom Checkboxes, Radios, and Floating Labels

Custom form controls require hiding the native element while keeping it accessible and keyboard-navigable.

```css
/* v3: custom checkbox */
.checkbox-wrapper {
  display: flex;
  align-items: flex-start;
  gap: 0.625rem;
  cursor: pointer;
  line-height: 1.5;
}

/* Hide the native checkbox visually, but keep it in the accessibility tree */
.checkbox-input {
  appearance: none;
  -webkit-appearance: none;
  width: 1.125rem;
  height: 1.125rem;
  min-width: 1.125rem;
  background: #111122;
  border: 1.5px solid #2a2a40;
  border-radius: 4px;
  cursor: pointer;
  position: relative;
  transition: background 0.15s, border-color 0.15s;
  margin-top: 0.15rem; /* align with first line of label */
}

/* The checkmark — drawn with CSS */
.checkbox-input::after {
  content: '';
  position: absolute;
  display: none;
  left: 4px;
  top: 1px;
  width: 5px;
  height: 9px;
  border: 2px solid #fff;
  border-top: none;
  border-left: none;
  transform: rotate(45deg);
}

.checkbox-input:checked {
  background: #2563eb;
  border-color: #2563eb;
}

.checkbox-input:checked::after {
  display: block;
}

.checkbox-input:focus-visible {
  outline: none;
  box-shadow: 0 0 0 2px #0d0d1a, 0 0 0 4px #a8d8ea;
}

.checkbox-input:disabled {
  opacity: 0.45;
  cursor: not-allowed;
}

.checkbox-input:disabled ~ .checkbox-label {
  opacity: 0.45;
  cursor: not-allowed;
}
```

The native `<input type="checkbox">` remains in the DOM — it receives keyboard focus, `Tab` navigation, `Space` to toggle, and screen reader announcements. Only its visual appearance is replaced with CSS. Never remove the native element.

Floating label technique using `:placeholder-shown`:

```css
/* Floating label */
.form-group--float {
  position: relative;
  margin-bottom: 1.25rem;
}

.form-group--float .form-input {
  padding-top: 1.25rem;
  padding-bottom: 0.25rem;
}

.form-group--float .form-label {
  position: absolute;
  left: 0.75rem;
  top: 0.875rem;
  font-size: 0.875rem;
  color: #4a4a6a;
  pointer-events: none;
  transition: top 0.15s, font-size 0.15s, color 0.15s;
  transform-origin: left top;
}

/* When placeholder is shown (input is empty), label is in resting position */
/* When placeholder is NOT shown (input has value), label floats up */
.form-group--float .form-input:not(:placeholder-shown) ~ .form-label,
.form-group--float .form-input:focus ~ .form-label {
  top: 0.25rem;
  font-size: 0.7rem;
  color: #a8d8ea;
}
```

`:placeholder-shown` is true when the input is empty and the placeholder is visible. When the user types, the placeholder hides, `:placeholder-shown` becomes false, and the sibling label floats up. The input must have a `placeholder` attribute (even `placeholder=" "`) for this technique to work.

## The Complete Program

See `forms.html` for the interactive demo. Below is the complete `form-framework.css`:

```css
/* form-framework.css — Chapter 8 */
/* A complete form styling system with validation, custom controls, and accessibility */

/* ─── Custom Properties ──────────────────────────────────── */
:root {
  --form-bg:           #111122;
  --form-border:       #2a2a40;
  --form-border-focus: #a8d8ea;
  --form-border-valid: #16a34a;
  --form-border-error: #dc2626;
  --form-text:         #c8c8d8;
  --form-text-dim:     #7878a0;
  --form-placeholder:  #4a4a6a;
  --form-radius:       6px;
  --form-focus-ring:   0 0 0 3px rgba(168, 216, 234, 0.2);
  --form-error-ring:   0 0 0 3px rgba(220, 38, 38, 0.15);
}

/* ─── Form Group ─────────────────────────────────────────── */
.form-group {
  display: flex;
  flex-direction: column;
  gap: 0.375rem;
  margin-bottom: 1.25rem;
}

/* ─── Labels ─────────────────────────────────────────────── */
.form-label {
  font-size: 0.875rem;
  font-weight: 500;
  color: var(--form-text);
  cursor: default;
}

.form-label--required::after {
  content: ' *';
  color: #f87171;
  font-weight: 400;
}

/* ─── Text Inputs, Textarea, Select ─────────────────────── */
.form-input,
.form-textarea,
.form-select {
  width: 100%;
  padding: 0.5rem 0.75rem;
  background: var(--form-bg);
  color: var(--form-text);
  border: 1px solid var(--form-border);
  border-radius: var(--form-radius);
  font-family: inherit;
  font-size: 0.875rem;
  line-height: 1.5;
  transition: border-color 0.15s, box-shadow 0.15s;
  -webkit-appearance: none;
  appearance: none;
}

.form-input::placeholder,
.form-textarea::placeholder {
  color: var(--form-placeholder);
}

.form-input:focus,
.form-textarea:focus,
.form-select:focus {
  outline: none;
  border-color: var(--form-border-focus);
  box-shadow: var(--form-focus-ring);
}

/* Validation */
.form-input:user-valid,
.form-textarea:user-valid {
  border-color: var(--form-border-valid);
}

.form-input:user-invalid,
.form-textarea:user-invalid {
  border-color: var(--form-border-error);
  box-shadow: var(--form-error-ring);
}

/* Disabled */
.form-input:disabled,
.form-textarea:disabled,
.form-select:disabled {
  opacity: 0.5;
  cursor: not-allowed;
  background: #0d0d1a;
}

/* ─── Textarea ────────────────────────────────────────────── */
.form-textarea {
  min-height: 100px;
  resize: vertical;
}

/* ─── Select ──────────────────────────────────────────────── */
.form-select {
  padding-right: 2.5rem;
  background-image: url("data:image/svg+xml,%3Csvg xmlns='http://www.w3.org/2000/svg' width='16' height='16' viewBox='0 0 24 24'%3E%3Cpath fill='%237878a0' d='M7 10l5 5 5-5z'/%3E%3C/svg%3E");
  background-repeat: no-repeat;
  background-position: right 0.5rem center;
  cursor: pointer;
}

/* ─── Error message ───────────────────────────────────────── */
.form-error {
  font-size: 0.8rem;
  color: #f87171;
  display: none;
  gap: 0.3rem;
  align-items: flex-start;
  line-height: 1.4;
}

.form-input:user-invalid ~ .form-error,
.form-textarea:user-invalid ~ .form-error {
  display: flex;
}

/* ─── Helper text ─────────────────────────────────────────── */
.form-hint {
  font-size: 0.775rem;
  color: var(--form-text-dim);
  line-height: 1.4;
}

/* ─── Floating Label ──────────────────────────────────────── */
.form-group--float {
  position: relative;
  margin-bottom: 1.25rem;
}

.form-group--float .form-input {
  padding-top: 1.25rem;
  padding-bottom: 0.25rem;
}

.form-group--float .form-label {
  position: absolute;
  left: 0.75rem;
  top: 0.875rem;
  font-size: 0.875rem;
  color: var(--form-placeholder);
  pointer-events: none;
  transition: top 0.15s, font-size 0.15s, color 0.15s;
  margin: 0;
}

.form-group--float .form-input:not(:placeholder-shown) ~ .form-label,
.form-group--float .form-input:focus ~ .form-label {
  top: 0.25rem;
  font-size: 0.7rem;
  color: var(--form-border-focus);
}

/* ─── Custom Checkbox ─────────────────────────────────────── */
.checkbox-wrapper {
  display: flex;
  align-items: flex-start;
  gap: 0.625rem;
  cursor: pointer;
}

.checkbox-input {
  appearance: none; -webkit-appearance: none;
  width: 1.125rem; height: 1.125rem; min-width: 1.125rem;
  background: var(--form-bg);
  border: 1.5px solid var(--form-border);
  border-radius: 4px;
  cursor: pointer;
  position: relative;
  transition: background 0.15s, border-color 0.15s;
  margin-top: 0.15rem;
  flex-shrink: 0;
}

.checkbox-input::after {
  content: '';
  position: absolute;
  display: none;
  left: 4px; top: 1px;
  width: 5px; height: 9px;
  border: 2px solid #fff;
  border-top: none; border-left: none;
  transform: rotate(45deg);
}

.checkbox-input:checked { background: #2563eb; border-color: #2563eb; }
.checkbox-input:checked::after { display: block; }
.checkbox-input:focus-visible { outline: none; box-shadow: 0 0 0 2px #0d0d1a, 0 0 0 4px #a8d8ea; }
.checkbox-input:disabled { opacity: 0.45; cursor: not-allowed; }
.checkbox-input:disabled ~ .checkbox-label { opacity: 0.45; cursor: not-allowed; }
.checkbox-input:hover:not(:disabled) { border-color: #a8d8ea; }

/* ─── Custom Radio ────────────────────────────────────────── */
.radio-wrapper {
  display: flex;
  align-items: flex-start;
  gap: 0.625rem;
  cursor: pointer;
}

.radio-input {
  appearance: none; -webkit-appearance: none;
  width: 1.125rem; height: 1.125rem; min-width: 1.125rem;
  background: var(--form-bg);
  border: 1.5px solid var(--form-border);
  border-radius: 50%;
  cursor: pointer;
  position: relative;
  transition: background 0.15s, border-color 0.15s;
  margin-top: 0.15rem;
  flex-shrink: 0;
}

.radio-input::after {
  content: '';
  position: absolute;
  display: none;
  top: 50%; left: 50%;
  transform: translate(-50%, -50%);
  width: 6px; height: 6px;
  background: #fff;
  border-radius: 50%;
}

.radio-input:checked { background: #2563eb; border-color: #2563eb; }
.radio-input:checked::after { display: block; }
.radio-input:focus-visible { outline: none; box-shadow: 0 0 0 2px #0d0d1a, 0 0 0 4px #a8d8ea; }
.radio-input:disabled { opacity: 0.45; cursor: not-allowed; }
.radio-input:hover:not(:disabled) { border-color: #a8d8ea; }

/* ─── Fieldset / Legend ───────────────────────────────────── */
fieldset.form-fieldset {
  border: 1px solid var(--form-border);
  border-radius: 8px;
  padding: 1rem 1.25rem;
  margin-bottom: 1.25rem;
}

legend.form-legend {
  padding: 0 0.5rem;
  font-size: 0.875rem;
  font-weight: 600;
  color: var(--form-border-focus);
}
```

## Walkthrough

### Label-Input Association

The `for`/`id` connection is the most important HTML form primitive. It does three things:

```
Without for/id association:

  <span>Email</span>           ← not a label — no connection
  <input type="email">         ← standalone input

  Clicking "Email" text: nothing happens.
  Screen reader reads input: "edit text" — no context.
  Tab navigation: works, but no label announced.

With for/id association:

  <label for="email">Email</label>
  <input id="email" type="email">

  1. Click "Email" text → input receives focus
     ┌────────────────┐     ┌─────────────────┐
     │  Email label   │────▶│  email input    │
     └────────────────┘     └─────────────────┘
     (click anywhere on the label text)

  2. Screen reader: "Email, required, edit text, type: email"
     The accessible name of the input comes from the label.

  3. Mobile: tapping the label activates the input,
     which opens the correct keyboard type.
```

Alternative association — wrapping label (no `for`/`id` needed):

```html
<label>
  Email
  <input type="email" name="email">
</label>
```

The wrapping form works but is less flexible: you can't style the label and input independently, and it's harder to add error messages between them.

### `:placeholder-shown` and Floating Labels

The floating label is a pure CSS state machine based on one selector:

```
State machine for floating label:

  Input state: empty (placeholder visible)
    .form-input:placeholder-shown ~ .form-label
    label position: inside the input (resting)
    ┌─────────────────────────────┐
    │  Email                      │  ← label overlaps placeholder area
    └─────────────────────────────┘

  Input state: focused (placeholder hidden by focus indicator)
    .form-input:focus ~ .form-label
    label position: floated to top-left
    ┌─────────────────────────────┐
    │Email                        │  ← label shrinks and rises
    │ ________________________    │
    └─────────────────────────────┘

  Input state: has value (placeholder no longer shown)
    .form-input:not(:placeholder-shown) ~ .form-label
    label position: floated (same as focused)
    ┌─────────────────────────────┐
    │Email                        │  ← stays floated even when unfocused
    │ alice@example.com           │
    └─────────────────────────────┘

The ~ combinator means "sibling that comes AFTER".
The label must come AFTER the input in the HTML for ~ to work.
```

Critical detail: the `<input>` must come *before* the `<label>` in the HTML for the `~` sibling combinator to work. This is the opposite of normal form structure. The visual order is controlled by CSS `position: absolute` on the label.

### Custom Checkbox Anatomy

Never build a fake checkbox with `<div>` and JavaScript. The native `<input type="checkbox">` provides keyboard handling, form submission, accessibility, and browser autofill for free.

```
Custom checkbox — what's really happening:

HTML:
  <label class="checkbox-wrapper">
    <input type="checkbox" class="checkbox-input">
    <span class="checkbox-label">Accept terms</span>
  </label>

CSS layers:
  1. appearance: none           → removes browser styling
  2. width/height: 1.125rem     → sets the visible box size
  3. background/border          → draws the custom box
  4. ::after                    → draws the checkmark when :checked
  5. :checked                   → changes bg to blue, shows ::after

Keyboard behavior (FREE from the browser):
  Tab    → focuses the input
  Space  → toggles checked state
  Label click → toggles checked state

Accessibility (FREE from the browser):
  role="checkbox" — announced automatically
  aria-checked    — state announced automatically
  Tab order       — included in natural tab sequence

The input is VISIBLE to assistive technology (not display:none).
Only the visual presentation is changed with CSS.
```

The checkmark is drawn entirely with CSS using a border technique:

```
::after pseudo-element — the checkmark:

  width: 5px; height: 9px;
  border: 2px solid white;
  border-top: none;
  border-left: none;
  transform: rotate(45deg);

  Before rotation:    After rotation:
   ___                     /
  |   |       →           /|
  |___|                  /_|
       right+bottom
       borders only
```

### Form Validation States

```
Browser validation pseudo-classes:

  :valid        — value satisfies all constraints
  :invalid      — value does NOT satisfy constraints
  :required     — has the required attribute
  :optional     — does NOT have required attribute
  :placeholder-shown — placeholder is currently displayed
  :checked      — checkbox/radio is checked

  New (correct):
  :user-valid   — like :valid, but only after user interaction
  :user-invalid — like :invalid, but only after user interaction

Why :user-valid instead of :valid:
  On page load, <input type="email" required> is immediately :invalid.
  This makes every required empty field look like an error before
  the user has typed anything.

  :user-invalid only applies after the user has:
    - Typed in the field and moved away
    - Attempted to submit the form

  ┌──────────────────────────────────────────────────────────┐
  │ Page loads                                               │
  │   :invalid = true   ← would show error immediately      │
  │   :user-invalid = false ← correct — user hasn't typed   │
  │                                                          │
  │ User types invalid email and blurs                       │
  │   :invalid = true                                        │
  │   :user-invalid = true ← now show the error             │
  │                                                          │
  │ User corrects to valid email                             │
  │   :valid = true                                          │
  │   :user-valid = true ← show success indicator           │
  └──────────────────────────────────────────────────────────┘
```

### `fieldset` and `legend` — Grouping and Accessibility

`<fieldset>` and `<legend>` provide semantic grouping that translates directly to accessibility.

```
Without fieldset/legend:

  ○ Male
  ○ Female
  ○ Non-binary

  Screen reader: "Male, radio button, 1 of 3"
  The user doesn't know what they're choosing.

With fieldset/legend:

  <fieldset>
    <legend>Gender</legend>
    <label><input type="radio" name="gender" value="m"> Male</label>
    <label><input type="radio" name="gender" value="f"> Female</label>
    <label><input type="radio" name="gender" value="nb"> Non-binary</label>
  </fieldset>

  Screen reader: "Gender group. Male, radio button, 1 of 3."
  ┌─ Gender ──────────────────────┐
  │  ○ Male                       │
  │  ○ Female                     │
  │  ○ Non-binary                 │
  └───────────────────────────────┘

The legend text is prepended to every input's accessible name.
Use fieldset for every group of related checkboxes or radio buttons.
```

`<fieldset disabled>` disables *all* form controls within it at once — a single attribute change instead of adding `disabled` to every input.

### Tab Order and Keyboard Navigation

```
Natural tab order — the browser's default:

  The browser tabs through focusable elements in DOM order:
  1. Inputs, buttons, selects, textareas
  2. Links with href
  3. Elements with tabindex="0"

  Tab order should match visual reading order.
  If it doesn't, keyboard users get confused.

  ┌─────────────┬─────────────┐
  │  [1] Name   │  [3] Phone  │  ← Tab 1 → 3 jumps columns
  ├─────────────┼─────────────┤    instead of going row-by-row
  │  [2] Email  │  [4] Submit │    This is a two-column layout
  └─────────────┴─────────────┘    problem — use flex/grid row order

tabindex values:
  tabindex="-1"  → focusable by script, not by Tab key
  tabindex="0"   → focusable by Tab in DOM order
  tabindex="1+"  → explicit order — AVOID. Creates maintenance nightmare.

Never use positive tabindex values. They override the natural order
and become impossible to maintain. Fix your DOM order instead.
```

## Guided Try It — Add a Password Strength Indicator

Password inputs benefit from live feedback. Let's build a CSS-only strength bar that responds to input state.

### Step 1: Base input with toggle visibility

```html
<div class="form-group">
  <label class="form-label" for="password">Password</label>
  <div class="input-wrapper" style="position:relative;">
    <input class="form-input" type="password" id="password"
           pattern="(?=.*[A-Z])(?=.*[0-9])(?=.*[^A-Za-z0-9]).{8,}"
           required>
    <button type="button" class="input-icon-btn" id="togglePwd"
            aria-label="Show password" aria-pressed="false">
      👁
    </button>
  </div>
  <div class="form-hint">
    Minimum 8 characters with uppercase, number, and symbol.
  </div>
</div>
```

```css
.input-wrapper { position: relative; }
.input-wrapper .form-input { padding-right: 2.5rem; }
.input-icon-btn {
  position: absolute; right: 0.5rem; top: 50%;
  transform: translateY(-50%);
  background: none; border: none; cursor: pointer;
  color: var(--form-text-dim); padding: 0.25rem;
  font-size: 0.875rem;
}
```

### Step 2: Strength bar using CSS attribute patterns

```css
.strength-bar {
  display: grid;
  grid-template-columns: repeat(4, 1fr);
  gap: 4px;
  height: 4px;
  margin-top: 0.5rem;
}

.strength-segment {
  background: var(--form-border);
  border-radius: 2px;
  transition: background 0.2s;
}

/* Driven by data-strength attribute set by JavaScript */
[data-strength="1"] .strength-segment:nth-child(1) { background: #dc2626; }
[data-strength="2"] .strength-segment:nth-child(-n+2) { background: #ea580c; }
[data-strength="3"] .strength-segment:nth-child(-n+3) { background: #ca8a04; }
[data-strength="4"] .strength-segment { background: #16a34a; }
```

### Step 3: JavaScript to calculate strength and set attribute

```javascript
const input = document.getElementById('password');
const bar   = document.getElementById('strengthBar');
const toggleBtn = document.getElementById('togglePwd');

function calcStrength(val) {
  let score = 0;
  if (val.length >= 8)              score++;
  if (/[A-Z]/.test(val))            score++;
  if (/[0-9]/.test(val))            score++;
  if (/[^A-Za-z0-9]/.test(val))     score++;
  return score;
}

input.addEventListener('input', () => {
  const strength = calcStrength(input.value);
  bar.dataset.strength = input.value.length > 0 ? strength : '';
});

toggleBtn.addEventListener('click', () => {
  const isText = input.type === 'text';
  input.type = isText ? 'password' : 'text';
  toggleBtn.setAttribute('aria-pressed', String(!isText));
  toggleBtn.setAttribute('aria-label', isText ? 'Show password' : 'Hide password');
});
```

The strength indicator is driven by a `data-strength` attribute on the bar container. CSS reads that attribute with attribute selectors (`[data-strength="3"]`). JavaScript only calculates the score and sets the attribute — the visual presentation is entirely CSS.

**Think about it**: The `pattern` attribute on the password input triggers `:invalid` if the password doesn't match. But showing a red border while the user is still typing their password is aggressive. How would you use `:user-invalid` and `:focus` together to show the error only after the user has finished typing and moved away from the field?

## Exercises

1. **Multi-Step Form**: Build a two-step form where Step 1 collects name and email, and Step 2 collects a password and terms acceptance. Show a progress indicator (Step 1 of 2). The "Next" button validates Step 1 before showing Step 2. Use JavaScript to toggle visibility between steps, but all styling (active step, progress, validation) should be CSS.

2. **Search Input with Clear Button**: Build a search input that shows a clear (×) button only when it has content. Use the CSS `:placeholder-shown` selector: when the placeholder is shown (input is empty), the clear button is hidden; when the placeholder is hidden (input has content), the clear button appears. The clear button should be positioned inside the input using `position: absolute`.

3. **Custom Select with Chevron**: Style a `<select>` element to remove the native OS arrow and replace it with a custom SVG chevron using `background-image`. The select should have the same focus ring as other inputs. Ensure the select looks identical in Chrome, Firefox, and Safari (hint: `appearance: none` plus explicit padding-right for the icon area).

4. **Accessible Toggle Switch**: Build a toggle switch (iOS-style) using only `<input type="checkbox">` + CSS. No JavaScript, no divs. The switch should show a sliding circle that moves left (off) to right (on) when toggled. Use the `::before` pseudo-element for the track and `::after` for the thumb. Include `:focus-visible` for keyboard users and `:disabled` styling.

5. **Form Layout with Grid**: Build a full contact form that uses CSS Grid for layout. The form has: Name (full width), Email and Phone (two columns), Subject (full width), Message textarea (full width, min 6 rows), and Submit/Reset buttons (right-aligned). All labels should be above their inputs. The grid should collapse to single-column on screens below 600px.

## Solutions

### Exercise 1: Multi-Step Form

```html
<style>
  .step { display: none; }
  .step.active { display: block; }

  .progress {
    display: flex; gap: 0.5rem; margin-bottom: 1.5rem;
  }
  .progress-step {
    flex: 1; height: 4px; border-radius: 2px;
    background: #2a2a40; transition: background 0.3s;
  }
  .progress-step.done { background: #2563eb; }
  .progress-step.active { background: #a8d8ea; }
</style>

<form id="multi-form">
  <div class="progress">
    <div class="progress-step active" id="prog1"></div>
    <div class="progress-step" id="prog2"></div>
  </div>

  <div class="step active" id="step1">
    <div class="form-group">
      <label class="form-label form-label--required" for="ms-name">Name</label>
      <input class="form-input" type="text" id="ms-name" required minlength="2">
    </div>
    <div class="form-group">
      <label class="form-label form-label--required" for="ms-email">Email</label>
      <input class="form-input" type="email" id="ms-email" required>
    </div>
    <button type="button" class="btn btn--primary" id="nextBtn">Next →</button>
  </div>

  <div class="step" id="step2">
    <div class="form-group">
      <label class="form-label form-label--required" for="ms-pw">Password</label>
      <input class="form-input" type="password" id="ms-pw" required minlength="8">
    </div>
    <label class="checkbox-wrapper">
      <input type="checkbox" class="checkbox-input" required>
      <span class="checkbox-label">I accept the terms</span>
    </label>
    <div style="margin-top:1rem;display:flex;gap:0.5rem;">
      <button type="button" class="btn btn--secondary" id="backBtn">← Back</button>
      <button type="submit" class="btn btn--primary">Submit</button>
    </div>
  </div>
</form>

<script>
  document.getElementById('nextBtn').addEventListener('click', () => {
    const step1Inputs = document.querySelectorAll('#step1 input');
    const allValid = [...step1Inputs].every(i => i.checkValidity());
    if (allValid) {
      document.getElementById('step1').classList.remove('active');
      document.getElementById('step2').classList.add('active');
      document.getElementById('prog1').classList.replace('active', 'done');
      document.getElementById('prog2').classList.add('active');
    } else {
      step1Inputs.forEach(i => i.reportValidity());
    }
  });
  document.getElementById('backBtn').addEventListener('click', () => {
    document.getElementById('step2').classList.remove('active');
    document.getElementById('step1').classList.add('active');
    document.getElementById('prog1').classList.replace('done', 'active');
    document.getElementById('prog2').classList.remove('active');
  });
</script>
```

### Exercise 2: Search with Clear Button

```html
<style>
  .search-wrapper { position: relative; }

  .search-clear {
    position: absolute; right: 0.5rem; top: 50%; transform: translateY(-50%);
    background: none; border: none; cursor: pointer;
    color: var(--form-text-dim); padding: 0.25rem; font-size: 0.875rem;
    opacity: 0; pointer-events: none; transition: opacity 0.15s;
  }

  .search-input:not(:placeholder-shown) ~ .search-clear {
    opacity: 1; pointer-events: auto;
  }
</style>

<div class="search-wrapper">
  <input class="form-input search-input" type="search"
         placeholder="Search..." id="search-field">
  <button type="button" class="search-clear"
          onclick="document.getElementById('search-field').value='';
                   document.getElementById('search-field').focus();"
          aria-label="Clear search">×</button>
</div>
```

### Exercise 3: Custom Select with Chevron

```css
.form-select {
  width: 100%;
  padding: 0.5rem 2.5rem 0.5rem 0.75rem;
  background: var(--form-bg);
  color: var(--form-text);
  border: 1px solid var(--form-border);
  border-radius: var(--form-radius);
  font-family: inherit;
  font-size: 0.875rem;
  line-height: 1.5;
  cursor: pointer;
  /* Remove ALL native styling */
  -webkit-appearance: none;
  -moz-appearance: none;
  appearance: none;
  /* Custom chevron */
  background-image: url("data:image/svg+xml,%3Csvg xmlns='http://www.w3.org/2000/svg' width='16' height='16' viewBox='0 0 24 24'%3E%3Cpath fill='%237878a0' d='M7 10l5 5 5-5z'/%3E%3C/svg%3E");
  background-repeat: no-repeat;
  background-position: right 0.625rem center;
  background-size: 16px;
}

.form-select:focus {
  outline: none;
  border-color: var(--form-border-focus);
  box-shadow: var(--form-focus-ring);
}
```

### Exercise 4: Toggle Switch

```css
.toggle-input {
  appearance: none; -webkit-appearance: none;
  position: relative;
  width: 44px; height: 24px;
  border-radius: 12px;
  background: var(--form-border);
  cursor: pointer;
  transition: background 0.2s;
  flex-shrink: 0;
}

/* Thumb */
.toggle-input::after {
  content: '';
  position: absolute;
  top: 3px; left: 3px;
  width: 18px; height: 18px;
  border-radius: 50%;
  background: #fff;
  transition: transform 0.2s, background 0.2s;
  box-shadow: 0 1px 3px rgba(0,0,0,0.4);
}

.toggle-input:checked {
  background: #2563eb;
}

.toggle-input:checked::after {
  transform: translateX(20px);
}

.toggle-input:focus-visible {
  outline: none;
  box-shadow: 0 0 0 2px #0d0d1a, 0 0 0 4px #a8d8ea;
}

.toggle-input:disabled {
  opacity: 0.45; cursor: not-allowed;
}
```

```html
<label class="checkbox-wrapper">
  <input type="checkbox" class="toggle-input">
  <span class="checkbox-label">Enable notifications</span>
</label>
```

### Exercise 5: Form Layout with Grid

```css
.contact-form {
  display: grid;
  grid-template-columns: 1fr 1fr;
  gap: 1.25rem;
}

.contact-form .form-group { margin-bottom: 0; }

.contact-form .full-width {
  grid-column: 1 / -1;
}

.contact-form .form-actions {
  grid-column: 1 / -1;
  display: flex;
  justify-content: flex-end;
  gap: 0.5rem;
}

@media (max-width: 600px) {
  .contact-form { grid-template-columns: 1fr; }
  .contact-form .full-width,
  .contact-form .form-actions { grid-column: 1; }
}
```

```html
<form class="contact-form">
  <div class="form-group full-width">
    <label class="form-label form-label--required" for="cf-name">Name</label>
    <input class="form-input" type="text" id="cf-name" required>
  </div>
  <div class="form-group">
    <label class="form-label form-label--required" for="cf-email">Email</label>
    <input class="form-input" type="email" id="cf-email" required>
  </div>
  <div class="form-group">
    <label class="form-label" for="cf-phone">Phone</label>
    <input class="form-input" type="tel" id="cf-phone">
  </div>
  <div class="form-group full-width">
    <label class="form-label form-label--required" for="cf-subject">Subject</label>
    <input class="form-input" type="text" id="cf-subject" required>
  </div>
  <div class="form-group full-width">
    <label class="form-label form-label--required" for="cf-message">Message</label>
    <textarea class="form-textarea" id="cf-message" rows="6" required></textarea>
  </div>
  <div class="form-actions">
    <button type="reset"  class="btn btn--secondary">Reset</button>
    <button type="submit" class="btn btn--primary">Send Message</button>
  </div>
</form>
```

## What You Learned

| Concept | Key point |
|---|---|
| `for`/`id` association | Links `<label>` to `<input>` — enables click-to-focus, screen reader announcements, and mobile keyboard activation |
| `type` attribute | Always set the correct type — `email`, `tel`, `number`, `password`, `date`. Activates the right mobile keyboard and validation |
| `appearance: none` | Removes native OS styling from inputs, checkboxes, radios, and selects — your baseline for custom styling |
| `:user-valid`/`:user-invalid` | Use instead of `:valid`/`:invalid` — only apply after the user has interacted with the field |
| `:placeholder-shown` | True when input is empty and placeholder is visible — drives the floating label state machine |
| Custom checkbox technique | `appearance: none` + CSS `::after` checkmark — native element stays in DOM for keyboard/a11y |
| `fieldset` + `legend` | Semantic grouping for radio buttons and checkboxes — prepends group name to each input's accessible name |
| `fieldset disabled` | Disables all contained controls at once — simpler than setting `disabled` on each input |
| Natural tab order | Matches DOM order. Never use positive `tabindex` values. Fix the HTML structure if tab order is wrong |
| `~` sibling combinator | Selects elements *after* a target — essential for showing error messages after inputs without wrappers |
| `placeholder` attribute | Must be present (even as `placeholder=" "`) for `:placeholder-shown` floating label technique to work |

## Building with Claude

**Bad prompt:**
> "Style my form inputs to look nice"

This is aesthetics-only. It ignores validation states, accessibility, focus rings, disabled states, and mobile behavior. You'll get CSS that looks decent but breaks as soon as a user tries to submit an invalid form.

**Good prompt:**
> "I have a form with `<label for='email'>` and `<input type='email' id='email' required>`. Write CSS for: (1) a focus ring using `box-shadow` (not `outline`) that is visible on a dark background, (2) a valid state using `:user-valid` that shows a green border after the user types a valid email, (3) an invalid state using `:user-invalid` that shows a red border and makes a `.form-error` sibling element visible using `display: flex`, (4) a disabled state that shows `not-allowed` cursor. The form background is `#111122`. No JavaScript — CSS selectors only."

This specifies the HTML structure, the exact selectors to use, the four states to handle, a specific technique requirement, the background color for focus ring calculation, and the scope. You'll get production-ready CSS.

**Connection to the JS book**: The search form in Chapter 10's Router, the filter inputs in Chapter 13's Debugger, and the test config form in Chapter 7's Test Framework all use this form framework for consistent, accessible input styling. The Debugger's filter inputs use the `:user-invalid` pattern to show a red border on malformed regex patterns, purely through CSS matching the browser's native constraint validation.
