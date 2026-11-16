---
id: m02-01-html-tags
module: m02-the-web
slug: html-tags-and-structure
title: "HTML: Tags, Elements, Attributes and Semantic Structure"
order: 1
duration_min: 25
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

A team asks an AI builder for a canteen menu page. It arrives, it looks fine, and three things are quietly broken.

A classmate who uses a screen reader opens it and hears "clickable, clickable, clickable, clickable" — because every button is a `<div>` with a click handler attached. A second classmate on a laptop tries to tab to the "Order" control and the key does nothing, because only real interactive elements are reachable by keyboard. And when the team says "make the dish names bigger", the AI changes the price text instead, because in the markup the dish name and the price are indistinguishable: both are `<div class="text">`.

The page was not badly designed. It was badly *named*. Nothing in it says what anything is, so neither a browser, nor a screen reader, nor the AI, nor the team can tell the parts apart.

### The idea

**HTML gives every part of a page a name, and the name you choose is a promise that browsers, screen readers and search engines act on.**

A `<button>` is not a box that happens to look like a button. It is a declaration: this is a control, it is focusable, Enter and Space activate it, and a screen reader should announce it as a button. You get all of that free by choosing the right name. Choose `<div>` and you get none of it, and then somebody has to rebuild each behaviour by hand — badly.

### The analogy

Picture the printed board above the canteen counter.

A bad board is one font, one size, one column: thali, rice, dal, sambar, 60, 20, 25, 30, closed, samosa. Every character on it is legible. Nobody can find the thali price.

A good board has a big heading "SOUTH INDIAN", item rows where the name sits left and the price sits right, and a stamp over the ones that are finished. Same information. The difference is that the board tells you what each piece of text *is*, so your eye can jump straight to the part you want. Semantic HTML is that board.

**Where the analogy breaks:** the canteen board only ever has to satisfy human eyes, and a human can work out from position and context that ₹60 is a price. HTML is also read by things that cannot see it at all — a screen reader announcing the page out loud, Google's crawler deciding what the page is about, your own JavaScript looking for "the submit button", and the AI you are directing. For those readers, position means nothing and the name is the only information there is.

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

HTML sits in the client box from M00's four boxes — it is what arrives in the browser and what the browser draws. In a plain page you write it directly. In a framework (chapter 15) a component produces it, and in a server-rendered app (chapter 16) the server assembles it before sending. Either way the markup that reaches the browser is HTML, which is why reading it is the skill that survives every framework change.

Three later modules depend on this chapter being solid. CSS (chapter 4) selects elements by their names and attributes, so sloppy markup makes styling fragile. JavaScript (chapter 8) finds elements before changing them. And M03's accessibility chapter is largely the consequences of the choices you make here.

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

**Div soup is the default failure.** Left to itself, an AI will produce a page built entirely from `<div>` and `<span>`, because those always work and never complain. It renders correctly, so nothing tells you it is wrong until a keyboard user or a screen-reader user arrives. The fix is to ask for it: name the landmarks you want.

**Headings are structure, not font size.** Picking `<h3>` because it "looks the right size" is the most common honest mistake. Heading levels are how a screen-reader user navigates a page — they jump heading to heading, the way you scan with your eyes. Skipping from `<h1>` to `<h4>` tells them a level of structure exists that does not. If a heading is the wrong size, that is a CSS problem.

**One `<h1>` per page, and it should match what the page is for.** Three `<h1>` elements means the page claims three main subjects.

**A `<div>` with a click handler is a trap with no warning label.** It is not keyboard-focusable, Enter does nothing, a screen reader does not announce it as a control, and it has no disabled state. All four are invisible in a screenshot, which is exactly why they survive review.

**`<b>` versus `<strong>`, `<i>` versus `<em>`.** The second of each pair means "this matters" and is announced with emphasis; the first only means "draw it bold". Prefer the meaning.

**Attributes are where accessibility actually lives.** `alt` on images, `for` on labels, `lang` on `<html>`. An AI will frequently emit `alt=""` or `alt="image"` to satisfy the rule without doing the work. Empty `alt` is correct for a purely decorative image and wrong for a photo of the dish you are selling.

### Verify it

Four checks, none of which need you to read the whole file.

1. **Tab through the page.** Press Tab repeatedly from the top. Every control you can click with a mouse must be reachable, must show a visible focus ring, and must activate with Enter. If a button is skipped, it is not a `<button>`.
2. **Read the heading outline.** In DevTools (chapter 3) or by searching the markup for `<h`, list the headings in order. Is there exactly one `<h1>`? Do the levels descend without skipping? Does the outline read like a sensible table of contents for this page?
3. **Check every image's `alt`.** For each `<img>`, ask what a person who cannot see it would need. "Plate of two samosas, ₹20" is useful. `alt="image"`, `alt="img1"` and a missing `alt` are all failures.
4. **Ask what each element claims to be.** Search the markup for `onclick` on anything that is not a `<button>` or `<a>`. Count the `<div>` elements; if there is no `<main>`, no `<nav>` and no `<header>` anywhere, the AI skipped structure entirely.

Red flags to name out loud when you push back: a page with zero semantic landmarks; `<div onclick>`; multiple `<h1>` elements; `alt="image"`; a "link" with no `href`, which is a `<a>` that cannot be opened in a new tab or copied; and tables used to lay out a page rather than to show tabular data.

The useful instruction to add to any brief, from M00's Director's Brief template, is a constraint and an acceptance criterion rather than a request: "use semantic HTML landmarks and real `<button>` and `<a>` elements for anything interactive" under CONSTRAINTS, and "every control is reachable and activatable by keyboard alone" under ACCEPTANCE.

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
