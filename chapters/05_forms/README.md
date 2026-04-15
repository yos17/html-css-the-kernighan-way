# Chapter 5 — Build Your Own Form Framework

Forms are the hardest part of CSS. Inputs, selects, checkboxes, and radios each behave differently across browsers, and none of them inherit fonts by default. Validation states need to work visually *and* accessibly. Focus indicators need to be visible to keyboard users without annoying mouse users. In this chapter you build a complete form framework from scratch — styled inputs, validation feedback, custom checkboxes and radios, and accessible error messages. Every decision connects to a real accessibility requirement.

---

## The Problem

Browser default form elements are inconsistent and largely unstyled. The naive approach styles only the happy path:

```css
/* Naive: styles happy path, forgets everything else */
input {
  border: 1px solid #ccc;
  padding: 8px;
  border-radius: 4px;
}
```

This misses: focus state (keyboard accessibility), invalid state (validation feedback), disabled state (non-interactive indication), the relationship between labels and inputs (screen reader association), and the fact that `<select>`, `<textarea>`, `<checkbox>`, and `<input>` each need different treatment.

---

## Building It Step by Step

### v1 — Base Input

```css
/* v1: accessible base input */

.form-control {
  display:       block;
  width:         100%;
  padding:       0.5rem 0.75rem;
  font-size:     0.9375rem;
  font-family:   inherit;
  line-height:   1.5;
  color:         var(--color-text);
  background:    var(--color-surface);
  border:        1.5px solid var(--color-border);
  border-radius: var(--radius);
  appearance:    none;           /* remove browser chrome */
  transition:    border-color 0.15s, box-shadow 0.15s;
}

/* Focus: visible ring for keyboard users */
.form-control:focus {
  outline:      none;
  border-color: #4361ee;
  box-shadow:   0 0 0 3px rgb(67 97 238 / 0.2);
}

/* Disabled */
.form-control:disabled {
  background: #f1f5f9;
  color:      var(--color-muted);
  cursor:     not-allowed;
}
```

### v2 — Validation States

```css
/* v2: valid/invalid visual feedback */

/* Invalid: shown after user interaction */
.form-control:user-invalid,
.form-control.is-invalid {
  border-color: #dc2626;
}
.form-control:user-invalid:focus,
.form-control.is-invalid:focus {
  box-shadow: 0 0 0 3px rgb(220 38 38 / 0.2);
}

/* Valid */
.form-control:user-valid,
.form-control.is-valid {
  border-color: #16a34a;
}
.form-control:user-valid:focus,
.form-control.is-valid:focus {
  box-shadow: 0 0 0 3px rgb(22 163 74 / 0.2);
}

/* Feedback text */
.form-feedback {
  display:    block;
  font-size:  0.8rem;
  margin-top: 0.3rem;
}
.form-feedback--invalid { color: #dc2626; }
.form-feedback--valid   { color: #16a34a; }
.form-feedback--hint    { color: var(--color-muted); }
```

### v3 — Custom Checkbox and Radio

The hardest part: `<input type="checkbox">` can't be styled like a regular element. The trick is to visually hide the real input and draw a custom one using its `<label>`:

```css
/* v3: custom checkbox */

.checkbox-label {
  display:     inline-flex;
  align-items: center;
  gap:         0.5rem;
  cursor:      pointer;
  user-select: none;
}

/* Hide the real checkbox visually, keep it in the DOM for accessibility */
.checkbox-input {
  appearance: none;
  -webkit-appearance: none;
  width:         1.125rem;
  height:        1.125rem;
  border:        1.5px solid var(--color-border);
  border-radius: 3px;
  background:    white;
  cursor:        pointer;
  flex-shrink:   0;
  transition:    all 0.15s;
}

.checkbox-input:checked {
  background:   #4361ee;
  border-color: #4361ee;
}

/* The checkmark via clip-path */
.checkbox-input:checked::after {
  content:      '';
  display:      block;
  width:        0.35rem;
  height:       0.6rem;
  border:       2px solid white;
  border-top:   none;
  border-left:  none;
  transform:    rotate(45deg) translate(2px, -1px);
  margin:       0 auto;
}

/* Focus ring */
.checkbox-input:focus-visible {
  outline:        2px solid #4361ee;
  outline-offset: 2px;
}
```

---

## The Complete Program

`forms.html` — a complete form with text inputs, selects, checkboxes, radios, a textarea, and real-time validation feedback.

---

## Walkthrough

### The `<label>` Contract

Every form control needs an associated label. There are three ways to associate them:

```html
<!-- Method 1: label wraps the input -->
<label>
  Email address
  <input type="email">
</label>

<!-- Method 2: for/id association -->
<label for="email">Email address</label>
<input type="email" id="email">

<!-- Method 3: aria-label (when no visible label) -->
<input type="search" aria-label="Search the site">
```

Methods 1 and 2 create the same accessible relationship. Method 3 is for cases where a visual label would be redundant (like a search box with a search button right next to it). Never use `placeholder` as a substitute for a label — it disappears when the user types and has poor color contrast.

### `:user-invalid` vs `.is-invalid`

```css
/* :user-invalid — shows error ONLY after the user has interacted with the field */
/* Great for native HTML5 validation (required, pattern, type="email") */
input:user-invalid { border-color: red; }

/* .is-invalid — shows error when YOU decide (JS-driven validation) */
/* Necessary for custom validation logic */
.is-invalid { border-color: red; }
```

Use `:user-invalid` for native constraints (required, pattern, email format). Use `.is-invalid` when you control the validation yourself. Combine both for the best user experience: show native errors automatically, show custom errors when your API returns a specific failure.

### `aria-describedby` for Error Messages

Screen readers won't read your `.form-feedback` text automatically — you need to link it to the input:

```html
<div class="form-group">
  <label for="email">Email</label>
  <input type="email" id="email"
         aria-describedby="email-error"
         aria-invalid="true"
         class="form-control is-invalid">
  <span id="email-error" class="form-feedback form-feedback--invalid" role="alert">
    Please enter a valid email address.
  </span>
</div>
```

`aria-describedby` links the input to the error. `aria-invalid="true"` tells screen readers this field has an error. `role="alert"` causes screen readers to announce the message immediately when it appears.

### Custom `<select>` Arrow

`<select>` is one of the hardest elements to style. `appearance: none` removes the native arrow, then you draw your own with a background image:

```css
.form-select {
  appearance:       none;
  background-image: url("data:image/svg+xml,%3Csvg xmlns='http://www.w3.org/2000/svg' width='12' height='12' viewBox='0 0 12 12'%3E%3Cpath fill='%2364748b' d='M6 8L1 3h10z'/%3E%3C/svg%3E");
  background-repeat:   no-repeat;
  background-position: right 0.75rem center;
  padding-right:       2.5rem;  /* space for the arrow */
}
```

The SVG is inlined as a data URI — no external file needed.

### `fieldset` and `legend`

Group related inputs (like radio buttons) inside a `<fieldset>` with a `<legend>`. This is a semantic HTML requirement, not just visual grouping:

```html
<fieldset>
  <legend>Preferred contact method</legend>
  <label><input type="radio" name="contact" value="email"> Email</label>
  <label><input type="radio" name="contact" value="phone"> Phone</label>
  <label><input type="radio" name="contact" value="sms"> SMS</label>
</fieldset>
```

Screen readers announce "Preferred contact method" before each radio option — "Preferred contact method, Email" — so users know what they're choosing between.

---

## Guided Try It — Add a Character Counter

**The goal**: a `<textarea>` that shows remaining characters as the user types.

**Step 1 — HTML structure**

```html
<div class="form-group">
  <label for="bio">Bio</label>
  <textarea id="bio" class="form-control" maxlength="160"
            rows="3" placeholder="Tell us about yourself"></textarea>
  <div class="form-footer">
    <span class="form-feedback form-feedback--hint" id="bio-hint">
      Write a short bio for your profile.
    </span>
    <span class="char-count" id="bio-count">160</span>
  </div>
</div>
```

**Step 2 — CSS for the counter**

```css
.form-footer {
  display:         flex;
  justify-content: space-between;
  margin-top:      0.3rem;
}

.char-count {
  font-size:  0.8rem;
  color:      var(--color-muted);
  font-family: var(--font-mono);
  transition: color 0.2s;
}

.char-count.warning { color: #ca8a04; }
.char-count.danger  { color: #dc2626; font-weight: 600; }
```

**Step 3 — JavaScript**

```javascript
const textarea = document.getElementById('bio');
const counter  = document.getElementById('bio-count');
const max      = parseInt(textarea.getAttribute('maxlength'));

textarea.addEventListener('input', () => {
  const remaining = max - textarea.value.length;
  counter.textContent = remaining;
  counter.className = 'char-count' +
    (remaining < 20 ? ' warning' : '') +
    (remaining < 5  ? ' danger'  : '');
});
```

**Think about it**: the `maxlength` attribute prevents typing past the limit at the HTML level. But should the counter show `remaining` or `used/max`? Twitter uses `remaining` (counts down), GitHub uses `used/max`. Which is more useful when you're close to the limit?

---

## Exercises

1. **Floating label**: A text input where the label starts inside the field (like a placeholder) and floats up when the field is focused or has a value. Use CSS `:placeholder-shown` to detect whether the field has a value.

2. **Password strength meter**: A password input with a strength indicator (weak/fair/strong/very strong). Use a progress bar (from Chapter 4) whose color and width change based on password entropy. Calculate entropy in JavaScript.

3. **Custom file upload**: The native `<input type="file">` is ugly. Hide it with `opacity: 0; position: absolute` and trigger it via a styled button. Show the selected filename in a custom display element. Handle drag-and-drop to the drop zone.

4. **Toggle switch**: A custom on/off toggle that looks like a mobile switch. Built on top of `<input type="checkbox">` — no JavaScript required, only CSS. Accessible because it uses the real checkbox under the hood.

5. **Multi-step form**: A form broken into three steps with a progress indicator. Only one step is visible at a time. Validate each step before allowing Next. On the final step, show a summary of all collected values before Submit.

---

## Solutions

### Exercise 1 — Floating label

```css
.float-group {
  position: relative;
  padding-top: 0.75rem;
}

.float-input {
  width:         100%;
  padding:       0.625rem 0.75rem;
  border:        1.5px solid var(--color-border);
  border-radius: var(--radius);
  font-size:     0.9375rem;
  font-family:   inherit;
  background:    white;
  appearance:    none;
}

.float-label {
  position:   absolute;
  left:       0.75rem;
  top:        1.375rem;  /* vertically centered in input */
  font-size:  0.9375rem;
  color:      var(--color-muted);
  pointer-events: none;
  transition: all 0.15s ease;
  transform-origin: left top;
  background: white;
  padding:    0 0.25rem;
}

/* Float up when focused OR when placeholder is not shown (= has value) */
.float-input:focus           + .float-label,
.float-input:not(:placeholder-shown) + .float-label {
  transform:  translateY(-1.4rem) scale(0.8);
  color:      #4361ee;
}

.float-input:focus { outline: none; border-color: #4361ee; box-shadow: 0 0 0 3px rgb(67 97 238/0.2); }
```

```html
<div class="float-group">
  <input type="text" class="float-input" id="name" placeholder=" ">
  <label class="float-label" for="name">Full Name</label>
</div>
<!-- Note: placeholder=" " (a space) is required for :placeholder-shown to work -->
```

### Exercise 2 — Password strength

```javascript
function getStrength(password) {
  let score = 0;
  if (password.length >= 8)  score++;
  if (password.length >= 12) score++;
  if (/[A-Z]/.test(password)) score++;
  if (/[0-9]/.test(password)) score++;
  if (/[^A-Za-z0-9]/.test(password)) score++;
  return score;  // 0-5
}

const labels  = ['', 'Weak', 'Fair', 'Good', 'Strong', 'Very strong'];
const colors  = ['', '#dc2626', '#ca8a04', '#ca8a04', '#16a34a', '#15803d'];
const widths  = ['0%', '20%', '40%', '60%', '80%', '100%'];

input.addEventListener('input', () => {
  const s = getStrength(input.value);
  bar.style.setProperty('--value', widths[s]);
  bar.style.setProperty('--bar-color', colors[s]);
  label.textContent = labels[s];
});
```

### Exercise 4 — Toggle switch (CSS only)

```css
.toggle { position: relative; display: inline-block; width: 44px; height: 24px; }
.toggle input { opacity: 0; width: 0; height: 0; }

.toggle-track {
  position:      absolute;
  inset:         0;
  background:    #cbd5e1;
  border-radius: 9999px;
  cursor:        pointer;
  transition:    background 0.2s;
}

.toggle-track::after {
  content:       '';
  position:      absolute;
  width:         18px; height: 18px;
  left:          3px; top: 3px;
  background:    white;
  border-radius: 50%;
  transition:    transform 0.2s;
  box-shadow:    0 1px 3px rgb(0 0 0/0.2);
}

input:checked + .toggle-track              { background: #4361ee; }
input:checked + .toggle-track::after       { transform: translateX(20px); }
input:focus-visible + .toggle-track        { outline: 2px solid #4361ee; outline-offset: 2px; }
```

```html
<label class="toggle" aria-label="Enable notifications">
  <input type="checkbox">
  <span class="toggle-track"></span>
</label>
```

---

## What You Learned

| Concept | Key point |
|---------|-----------|
| `appearance: none` | Removes browser-native UI chrome from form elements |
| `for`/`id` association | Links `<label>` to `<input>` — required for screen readers |
| `:user-invalid` | Shows invalid state only after user interaction (not on page load) |
| `aria-describedby` | Links error message text to the input that it describes |
| `aria-invalid` | Tells screen readers an input has a validation error |
| `role="alert"` | Causes screen readers to announce content immediately when it appears |
| Custom checkbox | `appearance: none` + `::after` for checkmark; real input stays for accessibility |
| `fieldset` + `legend` | Groups related inputs; screen readers prefix each input with the legend |
| `:placeholder-shown` | True when the placeholder is visible (= field has no value) |
| `data:` URI for SVG | Inline SVG as background-image — no external file needed |
| Focus ring | `box-shadow: 0 0 0 3px` — visible, doesn't affect layout, customizable color |

---

## Building with Claude

Bad prompt:
> "How do I style a checkbox in CSS?"

Good prompt:
> "I'm building a custom checkbox using `appearance: none` on `<input type='checkbox'>` with an `::after` pseudo-element for the checkmark. The checkmark is a rotated border trick: `border: 2px solid white; border-top: none; border-left: none; transform: rotate(45deg)`. Problem: the checkmark is slightly off-center — it sits too low and to the right. I've tried `margin: auto`, `position: absolute` with `top/left: 50%`, and `translate`. Each has different off-by-a-pixel issues. What's the most reliable way to center a pseudo-element inside a fixed-size checkbox that works across Chrome, Firefox, and Safari?"
