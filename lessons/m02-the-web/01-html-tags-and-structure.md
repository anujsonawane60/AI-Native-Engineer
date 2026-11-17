---
id: m02-01-html-tags
module: m02-the-web
slug: html-tags-and-structure
title: "HTML: Tags, Elements, Attributes and Semantic Structure"
order: 1
duration_min: 15
is_free: false
mission: false
volatility: low
last_reviewed: 2026-11-16
---

## Key learnings
- Name the parts of a page precisely enough to direct a change to one of them
- Tell the difference between a tag, an element and an attribute
- Explain why a `<button>` is not a clickable `<div>`, in terms of real users
- Read a page's heading outline and say whether its structure is honest
- Spot the four markup red flags that appear in almost every AI first draft

## Lesson

### The pain

A team asks an AI builder for a canteen menu page. It arrives, it looks fine, and three things are quietly broken: a classmate using a screen reader hears "clickable, clickable, clickable" because every button is a `<div>` with a click handler; a classmate on a laptop presses Tab and never reaches "Order"; and when the team says "make the dish names bigger", the AI enlarges the prices, because in the markup the name and the price are both `<div class="text">`.

The page was not badly designed. It was badly *named*. Nothing in it says what anything is, so no browser, screen reader, AI or teammate can tell the parts apart.

### The idea

**HTML gives every part of a page a name, and the name you choose is a promise that browsers, screen readers and search engines act on.**

A `<button>` is not a box that happens to look like a button. It is a declaration: this is a control, it is focusable, Enter and Space activate it, and a screen reader announces it as a button. Choose `<div>` instead and you get none of that, and somebody has to rebuild each behaviour by hand — badly.

### The analogy

A bad canteen board is one font, one column: thali, rice, dal, 60, 20, 25, closed, samosa. Every character is legible and nobody can find the thali price. A good board has a big "SOUTH INDIAN" heading and rows with the name left and the price right. Same information — but it tells you what each piece of text *is*. Semantic HTML is that board.

**Where the analogy breaks:** the board only has to satisfy human eyes, and a human infers from position that ₹60 is a price. HTML is also read by things that cannot see it at all — a screen reader, Google's crawler, your own JavaScript hunting for "the submit button", and the AI you are directing. For those readers position means nothing, and the name is the only information there is.

### Picture it

```
  <html>
    └── <body>
          ├── <header>        the canteen name and logo
          ├── <nav>           links to Menu, Orders, About
          ├── <main>          the one thing this page is for
          │     ├── <h1>      "Today's Menu"
          │     └── <section>
          │           ├── <h2>       "South Indian"
          │           └── <ul>
          │                 ├── <li>  dish + price + button
          │                 └── <li>  dish + price + button
          └── <footer>        contact, timings
```

An HTML document is a nest of named boxes: `<body>` holds the landmarks, `<main>` holds the page's actual purpose, headings describe the sections under them, and a list of dishes is genuinely a list.

### Where it lives

HTML sits in the client box from M00's four boxes — what arrives in the browser and what the browser draws. You may write it, a component may produce it (chapter 15) or a server may assemble it (chapter 16), but the markup that reaches the browser is always HTML, which is why reading it survives every framework change. What depends on it:

- CSS (chapter 4) selects elements by name and attribute, so sloppy markup makes styling fragile
- JavaScript (chapter 8) has to find an element before it can change it
- M03's accessibility chapter is largely the consequences of the choices you make here

### Words to own

| Term | Plain meaning | Example |
|---|---|---|
| Tag | The angle-bracket marker that opens or closes a thing | `<h1>` and `</h1>` |
| Element | The opening tag, the content and the closing tag together | `<h1>Today's Menu</h1>` |
| Attribute | Extra information on an element, written `name="value"` | `src`, `href`, `alt`, `class` |
| Nesting | Elements placed inside other elements | An `<li>` inside a `<ul>` |
| Semantic element | One whose name describes its role, not its look | `<nav>`, `<main>`, `<button>` |
| Void element | An element with no closing tag and no content | `<img>`, `<input>`, `<br>` |
| `id` | A name unique on the page, for one element | `id="order-form"` |
| `class` | A shared label, reusable on many elements | `class="dish-card"` |
| `alt` | Text that replaces an image when it cannot be seen | `alt="Plate of two samosas"` |
| Heading level | `<h1>` to `<h6>`, describing document hierarchy | `<h2>` under an `<h1>` |

### Tradeoffs and gotchas

**Div soup is the default failure.** Left alone, an AI builds the whole page from `<div>` and `<span>`, because those always render and never complain. Nothing tells you it is wrong until a keyboard or screen-reader user arrives, so you have to ask for landmarks by name.

**Headings are structure, not font size.** Picking `<h3>` because it "looks right" makes the outline lie — screen-reader users navigate by jumping heading to heading. Wrong size is a CSS problem. One `<h1>` per page; three `<h1>` elements claim three main subjects.

**A `<div>` with a click handler is a trap with no warning label.** Not focusable, Enter does nothing, not announced as a control, no disabled state — all four invisible in a screenshot, which is why they survive review.

**Meaning beats appearance in pairs.** `<strong>` and `<em>` mean "this matters" and are announced with emphasis; `<b>` and `<i>` only say "draw it bold".

**Attributes are where accessibility actually lives** — `alt` on images, `for` on labels, `lang` on `<html>`. An AI often emits `alt="image"` to satisfy the rule without doing the work. Empty `alt` is correct for a decorative image and wrong for a photo of the dish you are selling.

### Verify it

1. **Tab through the page.** Every mouse-clickable control must be reachable, show a focus ring and fire on Enter; a skipped button is not a `<button>`.
2. **Read the heading outline.** List the headings in order: exactly one `<h1>`, levels descending without gaps, reading like a table of contents.
3. **Check every `alt`.** "Plate of two samosas, ₹20" is useful; `alt="image"`, `alt="img1"` and a missing `alt` are failures.
4. **Search for `onclick`.** Any hit on something that is not a `<button>` or `<a>` is a control keyboard users cannot use.
5. **Count the landmarks.** No `<main>`, `<nav>` or `<header>` anywhere means the AI skipped structure entirely.

Red flags: zero semantic landmarks · `<div onclick>` · multiple `<h1>` elements · `alt="image"` · an `<a>` with no `href` · tables used for layout rather than tabular data. Put the fix in the brief as a rule, not a wish: "real `<button>` and `<a>` for anything interactive" under CONSTRAINTS, "every control reachable by keyboard alone" under ACCEPTANCE.

## Walkthrough
1. Open any page you did not build — your college website, a food delivery site, the canteen's Instagram page in a browser. Right-click and choose "View page source".
2. Find the `<html>` tag at the top and look for a `lang` attribute on it. Note whether it is there. That one attribute tells a screen reader which language to pronounce.
3. Search the source for `<main`. Then search for `<nav`, `<header` and `<footer`. Write down which landmarks exist. Many real sites have none.
4. Search for `<h1`. Count the matches. Then list every heading in order and write the outline out on paper as an indented list.
5. Search for `<img`. Pick three and read their `alt` attributes aloud. Decide for each whether a blind user would understand what the image was doing there.
6. Now put the page away and tab through it with the keyboard only, from the top. Try to reach the main call-to-action button and press Enter. Note where the focus ring disappears or where Tab skips something you can see.
7. Search the source for `onclick`. For each hit, look at what element it is attached to. Every `onclick` on a `<div>` or `<span>` is a control that keyboard users cannot use.
8. Write a three-line structural critique of the page: what it gets right, the single worst markup decision, and who it hurts.
9. Open an AI chat and paste your critique. Ask it to confirm or correct each point and to explain any you got wrong, without rewriting the page.
10. Finally, write the two sentences you would add to a Director's Brief — one CONSTRAINT and one ACCEPTANCE criterion — that would have prevented the worst problem you found.

## Talk to AI
```prompt
CONTEXT:      I am learning to read and direct HTML rather than write it. I
              am building a single page for a college canteen in Nashik.
              Users are students on budget Android phones, often on slow 4G.
              Some classmates use screen readers; some use a keyboard only.

GOAL:         A menu page whose MARKUP is structured well enough that I can
              later point at any single part of it and ask for a change to
              exactly that part.

REQUIREMENTS: 1. Proper document landmarks: header, nav, main, footer
              2. Exactly one h1, and heading levels that descend with no gaps
              3. Six dishes as a real list, each with a name, a price in
                 rupees, a photo and an "Add" control
              4. Every interactive thing is a real button or a real link
                 with an href, never a div with a click handler
              5. Every image has an alt attribute that describes the dish
              6. A lang attribute on the html element

CONSTRAINTS:  Plain HTML only for now, no framework, no CSS beyond the bare
              minimum, no JavaScript. Keep it under 120 lines so I can read
              all of it. Do not use tables for layout.

NON-GOALS:    No styling work. No cart. No payment. No login. Do not add a
              build step, a package file or any dependency.

ACCEPTANCE:   - Every control is reachable by Tab and activates with Enter
              - The heading outline reads as a sensible table of contents
              - With images blocked, every dish is still identifiable from
                its alt text alone
              - Searching the file for "onclick" returns nothing
              - There is exactly one h1 element

ASK:          First, before writing anything, list the HTML elements you
              intend to use and what each one is for, as a tree. Tell me
              which choices are genuinely debatable and what the
              alternative would be. Then wait for me to approve the tree.
              After I approve it, output the HTML and a short list of every
              decision where you chose meaning over appearance.
```

## Stuck? Ask AI
```prompt
Explain the difference between a tag, an element and an attribute using a non-computer analogy, then give me ten short snippets of HTML and ask me to label each part. Do not tell me the answers until I have attempted all ten.
```

```prompt
Here is some HTML a tool generated for me: [paste it]. Do not rewrite it. Instead, act as an accessibility reviewer and tell me, in a table: every element whose name does not match its role, who it hurts and how, and what the correct element would be. Rank the problems by how many users they affect.
```

```prompt
I keep being told to use semantic HTML but I do not understand what I actually lose by using divs for everything, since the page looks the same. Walk me through exactly what a screen reader announces for a div with a click handler versus a real button, and what a keyboard user experiences in each case. Be concrete about the keypresses.
```

## Quiz

### Q1 · L2
What is an attribute?
- (a) The text a user sees inside an element
- (b) Extra information written on an element as `name="value"` ✅
- (c) The closing half of a tag pair
- (d) A CSS rule attached to a tag
> Attributes qualify an element: `src` says where an image lives, `href` where a link goes, `alt` what the image shows. The element says what the thing is; attributes say the specifics.

### Q2 · L3
A page's "Order" control is a `<div>` with a click handler. It works perfectly when you test it with a mouse. What is actually broken?
- (a) Nothing — if it works with a mouse it is fine
- (b) It is not keyboard-focusable, Enter does nothing, and screen readers do not announce it as a control ✅
- (c) It will render at the wrong size on mobile
- (d) The browser will refuse to send the click to the server
> All four failures are invisible in a screenshot, which is why they survive review. A real `<button>` supplies focus, keyboard activation, the correct announcement and a disabled state for free.

### Q3 · L3
An AI gives you a page where a section title is marked `<h4>` because `<h2>` "looked too big". What should you change?
- (a) Accept it — heading tags are about size
- (b) Change it to `<h2>` and fix the size in CSS ✅
- (c) Change it to a `<div>` with a large font
- (d) Add a second `<h1>` above it to balance the outline
> Heading levels are the document's structure and are how screen-reader users navigate. Appearance is CSS's job. Picking a level for its font size makes the outline lie about the page.

### Q4 · L4
You are reviewing AI-generated markup for a dish card. Which finding is the most serious for your actual users?
- (a) The dish name is in a `<span>` rather than an `<h3>`
- (b) The photo has `alt="image"` ✅
- (c) The price uses `<b>` instead of `<strong>`
- (d) The card uses two nested `<div>` elements where one would do
> A useless `alt` means a blind student cannot tell what the dish is — the information is simply gone. The other three are real but milder: wrong emphasis, a weaker outline, and harmless extra nesting.

### Q5 · L4
Which line belongs in the CONSTRAINTS section of a brief to prevent div soup, rather than in GOAL?
- (a) "Build a canteen menu page"
- (b) "Make it look modern and clean"
- (c) "Use semantic landmarks and real button and anchor elements for anything interactive" ✅
- (d) "The page should be accessible"
> CONSTRAINTS are limits on *how* it may be built, and must be specific enough to check. "Accessible" is an intention; naming the elements required is a rule you can verify by searching the file.

## Go deeper
- [MDN: Structuring content with HTML](https://developer.mozilla.org/en-US/docs/Learn_web_development/Core/Structuring_content)
- [MDN: HTML elements reference](https://developer.mozilla.org/en-US/docs/Web/HTML/Element)
