# HTML & CSS: The Kernighan Way

**Learn HTML and CSS by building small, real pages from scratch.**

---

## What This Book Is

This book is for beginners.

If HTML and CSS still feel confusing, that is normal. The web has many small rules, and most tutorials throw too much at you at once. This book takes the opposite approach: one file, one idea, one working result at a time.

You will learn by building pages in the browser, not by memorizing long lists of tags and properties.

## How This Book Works

Each chapter builds one small tool or system:

1. we start with the problem
2. we build the simplest version that works
3. we improve it step by step
4. we explain what each new line is doing

The goal is not to impress you with clever CSS. The goal is to make the code readable enough that you can change it yourself.

## What You Need

Only two things:

1. **A browser**
2. **A text editor** like VS Code

No build tools. No npm. No frameworks. Each example is a single `.html` file you can open directly in a browser.

## The Main Learning Path

Right now this repository has some rough edges and overlapping chapter folders. For a beginner, use this path first:

| Chapter | Title | What You Learn |
|---------|-------|----------------|
| 1 | [Build Your Own CSS Reset](chapters/01_css_reset/) | What HTML is, what CSS is, and why browsers add default styles |
| 2 | [Build Your Own Type Scale](chapters/02_type_scale/) | How text size works, and why `rem` is better than fixed pixels |
| 3 | [Build Your Own Color System](chapters/03_color_system/) | How to choose and organize colors without guessing every value |
| 4 | [Build Your Own Layout Engine](chapters/04_layout_engine/) | How flexbox and grid place elements on a page |
| 7 | [Build Your Own Animation Library](chapters/07_animations/) | How transitions and keyframes create motion |
| 8 | [Build Your Own Form Framework](chapters/08_forms/) | How to build forms that are clear and usable |
| 12 | [Build Your Own CSS Debug Tool](chapters/12_debug/) | How to inspect layout and fix CSS problems systematically |

## Before You Start

If you are brand new, keep these ideas in mind:

- **HTML** says what something is
- **CSS** says how it looks
- the browser already applies some default styles
- most CSS becomes easier once you can see the box model, spacing, and layout axes clearly

You do not need to understand everything on the first read. Open the HTML files, change values, and watch what happens.

## Getting Started

```bash
git clone https://github.com/yos17/html-css-the-kernighan-way.git
cd html-css-the-kernighan-way
```

Then start here:

- read `chapters/01_css_reset/README.md`
- open the chapter HTML file in your browser
- change one thing at a time

That is the whole method.

---

*"The only way to learn a new programming language is by writing programs in it."*
— Brian W. Kernighan & Dennis M. Ritchie, *The C Programming Language* (1978)
