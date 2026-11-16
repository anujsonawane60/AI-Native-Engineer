---
id: m02-04-css-cascade
module: m02-the-web
slug: css-cascade-box-model
title: "CSS: Selectors, the Cascade and the Box Model"
order: 4
duration_min: 25
is_free: false
mission: false
volatility: low
last_reviewed: 2026-11-16
---

## Key learnings
- Explain why a rule you can see in the file is not the rule the browser used
- Read the Computed panel to find which declaration actually won, and why
- Name the four layers of the box model and where unexpected space comes from
- Say what `!important` costs a team six weeks later
- Spot the three CSS red flags that make every future change harder

## Lesson

### The pain

"Make the Order button green."

The AI adds a rule. You reload. The button is still blue. You say "it is still blue". The AI adds `!important`. Now it is green — and so is the Cancel button, because the selector was too broad. You ask it to fix that. It adds a more specific selector with another `!important`. Three weeks later the stylesheet has forty `!important` declarations, nobody can change anything without breaking something else, and every instruction you give produces a new override instead of a correction.

The original problem took four seconds to diagnose and nobody looked. Another rule, written later in the file by a different component, was setting the colour. The browser was not ignoring your rule; it was obeying a rule you did not know existed.

### The idea

**CSS is a pile of rules that frequently disagree, and the browser resolves every disagreement with a fixed, mechanical ranking — so "my rule is not working" almost always means "another rule outranked it".**

The ranking, simplified to what you actually need: inline styles beat ID selectors, which beat class selectors, which beat element selectors. Within the same rank, the rule written last wins. `!important` jumps the whole queue, which is why it is both tempting and poisonous.

### The analogy

The canteen's pricing notices, pinned to the board in layers.

A notice at the top says "all items ₹20". Below it, a second says "South Indian items ₹30". Taped over both, a third says "Today only: thali ₹60". A staff member serving a thali reads all three and applies the most specific one. A student asking for a vada reads the first two and pays ₹30.

Now someone writes "₹25 — FINAL, DO NOT CHANGE" in red marker across the whole board. That is `!important`. It works today. It also means the next three people who need to change a price have to write in bigger red marker, and within a month the board is unreadable and nobody knows the real price of anything.

**Where the analogy breaks:** canteen staff use judgement — they can tell that "thali ₹60" obviously means the thali and not the plate it comes on. The browser has no judgement at all. It computes a specificity score, compares numbers, and applies the winner even when the result is visibly absurd. There is no step where anything notices that the outcome is silly.

### Picture it

```
            ┌─────────────────────────────────┐
            │          margin                 │  space OUTSIDE,
            │   ┌─────────────────────────┐   │  transparent
            │   │        border           │   │
            │   │   ┌─────────────────┐   │   │
            │   │   │     padding     │   │   │  space INSIDE,
            │   │   │   ┌─────────┐   │   │   │  takes background
            │   │   │   │ content │   │   │   │
            │   │   │   └─────────┘   │   │   │
            │   │   └─────────────────┘   │   │
            │   └─────────────────────────┘   │
            └─────────────────────────────────┘
```

Every element is four nested rectangles: the content, padding around it on the inside, a border, and margin pushing other elements away on the outside.

### Why that diagram matters more than it looks

Two of the most common "the layout is slightly wrong" bugs live in it.

**Padding versus margin.** Padding is inside the element and is painted with its background colour; margin is outside and is always transparent. So a card with a white background and 16px of padding has a comfortable white gap inside its edge, while the same card with 16px of margin has a gap *between* cards. Confuse them and the fix looks right in one place and wrong in three others.

**Where the width goes.** By default, a declared width of 300px means the *content* is 300px, and padding and border are added on top — so a 300px card with 16px padding and a 1px border actually occupies 334px. On a 360px phone, two of those will not fit where you expected. Setting `box-sizing: border-box` makes a declared width mean the whole box, which is what almost everyone intends. This is why almost every real project sets it once globally.

### Where it lives

CSS lives in the client box from M00, alongside the HTML it styles. It arrives as its own file (or files), which means it is one more thing to download before anything looks right — a cost chapter 16 will measure.

CSS does nothing on its own. It selects elements that HTML named (chapter 1), which is why chapter 1's semantic markup pays off here: `main > section > ul > li` is only writable if those elements exist. Layout, built on the box model, is chapter 5. Making values changeable in one place rather than forty is chapter 7. And M03 covers what you should be choosing in the first place — this chapter is only the mechanism.

### Words to own

| Term | Plain meaning | Example |
|---|---|---|
| Selector | The pattern that picks which elements a rule applies to | `.dish-card h3` |
| Declaration | One property and value pair | `color: green;` |
| Rule | A selector plus its block of declarations | `.btn { color: green; }` |
| Cascade | The process that decides which rule wins a conflict | Specificity, then order |
| Specificity | The score a selector gets, deciding its rank | An id beats a class |
| Inheritance | Some properties pass down to children automatically | `color`, `font-family` |
| Padding | Space inside the element, under its background | Breathing room in a card |
| Margin | Transparent space pushing neighbours away | Gap between cards |
| `box-sizing` | Whether width means content or the whole box | `border-box` |
| Margin collapse | Two vertical margins merging into the larger one | 16px + 24px = 24px |
| Computed value | The value that actually ended up being used | What DevTools shows |

### Tradeoffs and gotchas

**`!important` is a debt, not a fix.** It wins today and it removes your ability to win tomorrow by normal means. The only honest uses are overriding a third-party stylesheet you cannot edit, and utility classes designed for it. If an AI reaches for `!important`, the correct instruction is: "remove it and tell me which rule was actually overriding mine."

**Vertical margins collapse and horizontal ones do not.** Two stacked elements with 16px bottom and 24px top margin end up 24px apart, not 40px. Beginners add more margin, get no change, and add more again. The honest fix is usually `gap` on the parent (chapter 5), which never collapses.

**Specificity wars make a codebase un-directable.** Once selectors are three and four levels deep, every change needs a still-more-specific selector, and the instruction "make this heading smaller" stops having a small answer. Flat, class-based selectors keep the stylesheet something you can still direct in week ten.

**Inline styles beat your stylesheet.** An AI that writes `style="color: blue"` directly on the element has put that value beyond the reach of every stylesheet rule except `!important`. Watch for it, especially in generated components.

**Inheritance is not universal.** `color` and `font-family` pass down to children; `border` and `padding` do not. Expecting the wrong one wastes a lot of time.

**The cascade is not the only ranking.** Where two rules tie on specificity, the later one wins — and "later" includes the order your stylesheets are linked in. A rule that works locally and fails in production is sometimes just a changed file order.

### Verify it

The Computed panel is the whole verification story for this chapter, and almost nobody new uses it.

1. **Ask the browser which rule won.** Inspect the element, open the Styles panel, and look for your declaration. If it is shown struck through, it lost. Directly next to the winner is the file and line it came from — that is your real answer, in four seconds.
2. **Read the Computed tab for the final value.** Styles shows every rule that was considered; Computed shows the single value in force. When the two disagree with your expectation, Computed is the truth.
3. **Count the `!important` declarations.** Search the stylesheet. Zero is the target on new work. Any number above three in a young project means the cascade is already being fought rather than used.
4. **Hover the box model diagram in DevTools.** It shows content, padding, border and margin as measured numbers. Mystery space is almost always margin you forgot or padding you assumed.
5. **Check for inline styles in generated markup.** Search for `style="`. Each one is a value no stylesheet can later change.
6. **Change one thing and count what moved.** After any spacing change, look at two other screens. Margin changes travel further than people expect.

Red flags to name in AI output: new `!important` declarations; selectors more than three levels deep; inline `style` attributes; a fix that adds a rule rather than correcting the rule that was wrong; `position: absolute` used to nudge something into place; and fixed pixel heights on anything containing text, which breaks the moment the text is longer or the user zooms.

The brief line that prevents most of this, under CONSTRAINTS: "do not use `!important`, inline styles, or selectors deeper than two levels; if my rule is being overridden, tell me which rule is overriding it instead of adding a stronger one."

## Walkthrough
1. Open a page of your own, or any page, and inspect a button. In the Styles panel, find the `color` declaration that is in force.
2. Scroll down the Styles panel and count how many *other* `color` declarations are listed and struck through. Note where each came from — file name and line, shown on the right.
3. Switch to the Computed tab and find `color`. Expand it. The browser is now telling you the final value and which rule produced it. Compare this with your guess from step 1.
4. Still on that button, tick and untick a declaration's checkbox in the Styles panel. Watch the page change live. You are now testing without changing any file.
5. Hover over the box model diagram at the bottom of the Styles panel. Write down the four numbers: content size, padding, border, margin.
6. Change the padding live to twice its value and watch what moves. Then undo it, change the margin to twice its value, and watch what moves instead. Describe the difference in one sentence.
7. Find an element with a declared `width` and check its `box-sizing` in Computed. If it is `content-box`, add up width plus padding plus border and compare that total against the measured box.
8. Search the page's stylesheet for `!important`. Count the hits. For any one of them, try to work out from the Styles panel what it was fighting.
9. Search the rendered markup for `style="`. For each hit, note what property is hardcoded there and why that is now hard to change.
10. Write a three-line report: which rule won on your button and why, where the unexplained space in the layout comes from, and the one CSS habit in this page you would ban in your own project.

## Talk to AI
```prompt
CONTEXT:      I direct AI to write CSS; I do not write it myself. My project
              is a canteen ordering app used by students in Nashik on budget
              Android phones, mostly at 360px wide, often in bright sunlight.
              The stylesheet is young and I want to keep it directable for
              the next six months, with three teammates and an AI agent all
              touching it.

GOAL:         A styling convention for this project, plus the stylesheet
              foundations that enforce it, so that six weeks from now a
              one-sentence visual instruction still has a one-line answer.

REQUIREMENTS: 1. A global reset or baseline, with box-sizing set so a
                 declared width means the whole box, and an explanation of
                 why that matters at 360px
              2. A selector convention: flat class names, maximum nesting
                 depth, and what names should describe
              3. A spacing approach that avoids margin-collapse surprises
              4. A short written rule for when, if ever, !important is
                 allowed in this project
              5. A worked example: one dish card, styled under these rules,
                 with every declaration justified in a comment-free list
                 alongside the code

CONSTRAINTS:  Plain CSS only, no framework and no build step yet. No inline
              style attributes. No selector deeper than two levels. No
              !important anywhere in my own code. No fixed pixel heights on
              anything containing text. Must be legible in sunlight, so
              state the contrast ratios you are targeting.

NON-GOALS:    Do not introduce a CSS framework, a preprocessor or a
              component library. Do not restyle the whole app. Do not
              discuss dark mode or design tokens yet.

ACCEPTANCE:   - Searching the stylesheet for "!important" returns nothing
              - Searching the markup for 'style="' returns nothing
              - No selector has more than two levels
              - A declared card width of 300px measures 300px in the
                DevTools box model, including padding and border
              - Doubling the card's padding changes only space inside the
                card, and I can predict that before I try it

ASK:          Before writing anything, propose two different selector and
              spacing conventions for this project, with the tradeoffs of
              each, and say which you would pick for a three-person team
              working with an AI agent and why. No code until I choose.
```

## Stuck? Ask AI
```prompt
I wrote a CSS rule and it is not taking effect, but I can see it in the file. Explain the cascade to me as a ranking I can apply by hand: list what beats what, in order, with a one-line example of each level. Then give me six pairs of conflicting rules and ask me which one wins and why, without telling me the answers first.
```

```prompt
Explain the difference between padding and margin using an analogy that is not about boxes or picture frames. Then explain what margin collapse is, why it exists, and give me three layouts where it would surprise me. Tell me what to use instead of vertical margins to avoid it.
```

```prompt
Here is some CSS a tool generated for me: [paste it]. Do not improve it. Instead, audit it: list every !important, every inline style, every selector deeper than two levels, and every fixed pixel height on text. For each one, tell me what specific future change it makes harder, and what the correct alternative would have been.
```

## Quiz

### Q1 · L2
In the box model, which layer is painted with the element's background colour?
- (a) Margin
- (b) Padding ✅
- (c) Both margin and padding equally
- (d) Neither — only the content area is
> Padding is space inside the element, so the background extends through it. Margin is outside the element and is always transparent, which is why a "gap" that should have been coloured usually means padding was needed.

### Q2 · L3
Your rule `.btn { color: green; }` appears in the Styles panel struck through. What does that mean, and what should you do?
- (a) The file failed to load; re-link the stylesheet
- (b) Another rule outranked it; read which rule won and correct that one ✅
- (c) The selector is invalid; rewrite it
- (d) Add `!important` to make it win
> Struck through means considered and beaten. DevTools names the winning rule's file and line right next to it, so the fix is a correction to the real cause rather than a stronger override.

### Q3 · L3
A card is declared `width: 300px` with `16px` padding and a `1px` border, under default `box-sizing`. How wide is the box on screen?
- (a) 300px
- (b) 317px
- (c) 334px ✅
- (d) 332px
> By default `width` sizes the content only, so padding and border are added: 300 + 16 + 16 + 1 + 1. On a 360px phone that difference is the whole bug, which is why projects set `box-sizing: border-box` globally.

### Q4 · L4
An AI "fixes" your colour problem by adding `!important`. Why is this the wrong outcome even though the page now looks right?
- (a) `!important` is slower for the browser to process
- (b) The real conflicting rule is still there, and every future change now needs an even stronger override ✅
- (c) `!important` only works in some browsers
- (d) It will be stripped out when the site is deployed
> The symptom is gone and the cause is intact. Each `!important` removes a layer of normal control, so within weeks the stylesheet can only be changed by escalation — which is what makes a codebase un-directable.

### Q5 · L5
Two stacked cards have `margin-bottom: 16px` and `margin-top: 24px`. The gap measures 24px, not 40px. A teammate wants to add more margin. What is the better instruction?
- (a) Increase the bottom margin to 40px
- (b) Add `!important` to the top margin
- (c) Remove the vertical margins and set `gap` on the parent container ✅
- (d) Wrap each card in an extra `<div>` to stop the collapse
> Adjacent vertical margins collapse to the larger value, so adding more produces unpredictable results. `gap` on the parent never collapses, so the spacing you declare is the spacing you get.

## Go deeper
- [MDN: Handling conflicts and the cascade](https://developer.mozilla.org/en-US/docs/Learn_web_development/Core/Styling_basics/Handling_conflicts)
- [MDN: The box model](https://developer.mozilla.org/en-US/docs/Learn_web_development/Core/Styling_basics/Box_model)
