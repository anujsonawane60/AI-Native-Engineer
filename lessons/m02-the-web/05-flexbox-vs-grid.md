---
id: m02-05-flex-grid
module: m02-the-web
slug: flexbox-vs-grid
title: "Layout: Flexbox vs Grid, and When to Use Each"
order: 5
duration_min: 25
is_free: false
mission: false
volatility: low
last_reviewed: 2026-11-16
---

## Key learnings
- Choose flexbox or grid from the shape of the problem, not from habit
- Say what "one direction" versus "two directions" actually means in a layout
- Explain why absolute positioning as a layout tool breaks on a 360px phone
- Use the DevTools layout overlays to see the container the AI actually built
- Name the four stress tests that break most AI-generated layouts

## Lesson

### The pain

A team's menu page looks right on the laptop it was built on. On a ₹8,000 Android it falls apart in three separate ways.

The navigation bar, built with absolute positioning, has the logo sitting on top of the menu links. The dish cards, given a fixed `width: 300px` each in a row, overflow the 360px screen so the page scrolls sideways and the price column is off-screen. And one dish has a long name — "Mysore Masala Dosa with Extra Chutney" — which pushes its card taller than the others, so the row of "Add" buttons is now a ragged staircase and the cheapest dish's button is below the fold.

Each of these is a layout tool used for a job it was not built for. None of them would have survived four minutes of testing at 360px.

### The idea

**Flexbox lays things out along one direction; grid lays things out along two. Pick by the shape of the problem and most layout bugs never happen.**

A navigation bar is one direction: things in a row, with space distributed between them. A photo gallery where items must line up in both rows *and* columns is two directions. If you find yourself fighting a layout, the usual cause is that the problem has two directions and the tool has one, or the reverse.

### The analogy

The canteen at lunchtime, in two parts.

The queue at the counter is flexbox. It is one line. You can decide whether people bunch at the front, spread evenly, or leave a standard gap between them, and whether short and tall people are aligned by their heads or their feet. But it is still, fundamentally, a line — and when there are too many people the line either gets squashed or wraps around the corner.

The thali tray is grid. It has fixed compartments: rice here, two sabzis there, a small round one for pickle. Rows and columns both matter, and every item has a slot that exists whether or not anything is in it. You can also say "the rice compartment gets twice the space of the pickle one" and have that hold true regardless of what you put in them.

**Where the analogy breaks:** a steel tray's compartments are a fixed size forever. Grid tracks can be fractions that recompute — "three equal columns" means equal at 360px and equal at 1440px, and "as many columns of at least 160px as will fit" means two on a phone and six on a laptop, with no breakpoint written anywhere. A tray cannot do that, and that flexibility is most of why grid is worth learning.

### Picture it

```
  FLEXBOX - one direction                GRID - two directions
  ┌────────────────────────────┐         ┌──────┬──────┬──────┐
  │ [logo]      [a] [b] [c]    │         │  1   │  2   │  3   │
  └────────────────────────────┘         ├──────┼──────┼──────┤
    items distribute along the           │  4   │  5   │  6   │
    main axis; the cross axis            ├──────┴──────┼──────┤
    only aligns them                     │  7 (spans 2)│  8   │
                                         └─────────────┴──────┘
                                           rows AND columns are
                                           declared up front
```

Flexbox arranges items along a single line and aligns them across it; grid declares rows and columns in advance, and items can be placed in or across those tracks.

### How to choose, in one question

Ask: **do the items need to line up with anything in the other direction?**

A row of nav links does not — nothing below them has to align with link number two. Flexbox.

A card's internal layout — photo on the left, name and price stacked on the right — does not need cross-alignment either. Flexbox.

A page where the sidebar, the header and the content area must all share consistent edges does. Grid.

A responsive set of dish cards where every card's "Add" button should sit at the same height as its neighbours' does. Grid, or flexbox plus a rule that stretches the items.

In real projects you use both, nested: grid for the page skeleton, flexbox inside each region. That is not indecision; it is the two tools doing their own jobs.

### Where it lives

Layout is pure client-box work from M00 — the server sends the same HTML and CSS regardless of screen size, and the browser does all the arranging. That is a genuinely important point: a layout that adapts needs no server involvement and no extra request, which is why CSS layout is cheaper than any JavaScript solution to the same problem.

This chapter builds directly on chapter 4's box model, because `gap`, padding and margin are what produce the spacing inside any layout. Chapter 6 adds the screen-size dimension: the same grid with different track counts at different widths. M03 covers what good spacing and hierarchy actually look like — here we only cover the mechanism for achieving them.

### Words to own

| Term | Plain meaning | Example |
|---|---|---|
| Flex container | The parent that lays its children out in a line | The nav bar |
| Main axis | The direction flex items flow along | Across, for a row |
| Cross axis | The perpendicular direction, used for alignment | Vertical, for a row |
| `justify-content` | How space is distributed along the main axis | Logo left, links right |
| `align-items` | How items line up across the cross axis | Vertically centred |
| `gap` | Fixed space between items, which never collapses | 16px between cards |
| `flex-wrap` | Whether a too-long line breaks onto a second line | Cards wrapping at 360px |
| Grid track | One row or one column of a grid | The middle column |
| `fr` unit | A fraction of the leftover space | `1fr 2fr` = one third, two thirds |
| `minmax()` | A track that is at least X and at most Y | `minmax(160px, 1fr)` |
| `auto-fit` | Fit as many tracks as will go, then stop | Responsive cards, no breakpoints |

### Tradeoffs and gotchas

**Absolute positioning is not a layout tool.** It takes an element out of the flow and pins it to coordinates, which means nothing else knows it exists. It is correct for a dropdown panel, a tooltip or a badge on a corner. It is wrong for a nav bar, a card row or anything whose size depends on its content — and it is the single most common reason an AI-generated layout collapses at 360px.

**Fixed pixel widths inside a flex row will overflow.** `width: 300px` on three cards needs 900px plus gaps. On a 360px phone the row either overflows or squashes. Use `flex: 1` or a grid with `minmax()` so the items negotiate rather than demand.

**Grid for a nav bar is overkill; flexbox for a page skeleton is a fight.** Both work. Both produce more CSS than needed and more confusion for whoever changes it next.

**Deeply nested flex containers become undebuggable.** Six levels of flex, each with its own `justify-content`, and nobody can say why something is 3px off. If the AI gives you five nested flex wrappers for a card, the honest instruction is "flatten this; the card has at most two layout containers".

**`gap` versus margins.** `gap` applies only *between* items, never outside the first or last, and never collapses. Margins on children produce edge gaps you then have to cancel. Prefer `gap`.

**Height is harder than width.** `height: 100%` and `height: 100vh` behave differently from their width counterparts, and `100vh` on mobile browsers includes space the browser toolbar covers — so a "full screen" section is taller than the screen and the page scrolls by 60px for no visible reason. Chapter 6 revisits this.

### Verify it

Layout verification is almost entirely about stress, not inspection. A layout that looks right with the AI's sample content tells you very little.

1. **Turn on the layout overlay.** In DevTools, elements that are flex or grid containers show a small badge next to them in the Elements panel. Click it to see the tracks and gaps drawn on the page. This instantly tells you which tool the AI actually used, which is often not the one it said it used.
2. **Shrink to 320px and look for a horizontal scrollbar.** Any horizontal scroll on a content page is a bug. 320px is the narrowest you should support; 360px is where most of your users are.
3. **Stress test one: the long item.** Replace one dish name with "Mysore Masala Dosa with Extra Chutney and Sambar". Does the card grow gracefully, does the text clip, or does the row break?
4. **Stress test two: the count.** The AI built it with six cards. Try one card. Try twelve. Try one with an empty list. A grid with `auto-fit` handles all four; a hand-placed layout usually handles exactly six.
5. **Stress test three: the missing image.** Block images in DevTools and reload. Layouts that depend on an image's intrinsic size collapse here.
6. **Stress test four: zoom to 200%.** Browser zoom is how a lot of people with low vision read. A layout built from fixed pixels in absolute positions falls apart; a flexible one simply reflows.
7. **Count the layout containers.** Inspect one card and count how many nested flex or grid parents it has. More than three is a smell worth asking about.

Red flags in AI layout output: `position: absolute` on anything structural; fixed pixel widths on items in a row; `height` in pixels on anything containing text; five or more nested wrapper `<div>` elements for one card; a media query at every 50px; and `overflow: hidden` added to make an overflow problem invisible rather than fixing it. That last one is the layout equivalent of `!important`.

## Walkthrough
1. Open any shopping or food-delivery site on a laptop and inspect its main product grid. In the Elements panel, look for the grid or flex badge on the container and click it.
2. With the overlay on, read the layout off the page: how many columns, how wide the gaps, whether the tracks are equal. Write it down.
3. Walk up the tree from one card to the page body, noting every element that carries a flex or grid badge. Count them. That is the layout depth of this page.
4. Now inspect the site's top navigation bar and identify which tool it uses. Decide, from the "does it need to line up in the other direction" question, whether you agree with the choice.
5. Open the device toolbar, set 360 x 800, and reload. Count the columns now. Then look for whether a media query did that or whether the grid itself adapted — check the Styles panel for the container.
6. Drag the viewport down to 320px. Note the exact width at which anything breaks, clips or starts scrolling sideways.
7. Using the Elements panel, edit one product title to something three times longer. Watch what happens to its card and to its neighbours. Screenshot the result.
8. Delete all but one product card in the Elements panel and look at the layout with a single item. Then duplicate one card several times and look at it with far more items than intended.
9. Zoom the browser to 200% and reload. Note what breaks that did not break at 360px, and what that tells you about fixed pixel sizing.
10. Write a short verdict: which tool this page used where, one place you would have chosen differently and why, and the exact viewport width at which it first breaks.

## Talk to AI
```prompt
CONTEXT:      I direct AI rather than writing CSS. My project is a canteen
              ordering app. Primary device is a budget Android at 360px
              wide; some users are on 320px, a few on laptops at 1440px.
              There are two layout problems I need solved: the page skeleton
              (header, main content, footer) and a list of dish cards. Each
              dish card has a photo, a name, a price in rupees and an Add
              button.

GOAL:         A layout for both problems that survives real content rather
              than sample content, with the choice between flexbox and grid
              explicitly justified for each.

REQUIREMENTS: 1. For each of the two problems, state whether you are using
                 flexbox or grid and answer the question "do these items
                 need to line up in the other direction" to justify it
              2. The dish card list must show as many columns as fit, with
                 no media queries, and each card at least 160px wide
              3. Every card's Add button must sit at the same height as its
                 neighbours' regardless of how long the dish name is
              4. Spacing between items must use gap, not margins on
                 children
              5. Must work with 1 card, 6 cards and 40 cards, and with an
                 empty list

CONSTRAINTS:  Plain CSS, no framework. No position: absolute for anything
              structural. No fixed pixel widths on cards. No fixed pixel
              heights on anything containing text. No overflow: hidden used
              to hide a layout problem. Maximum two nested layout
              containers per card. Must not scroll horizontally at 320px.

NON-GOALS:    No colours, fonts or visual polish. No animation. No dark
              mode. Do not restructure the HTML beyond what the layout
              needs, and tell me if you do.

ACCEPTANCE:   - At 320px there is no horizontal scrollbar
              - A dish name of 45 characters does not clip, overlap or
                break the row
              - With 1 card the layout does not stretch that card across
                the whole screen
              - With 40 cards the columns stay the same width
              - Doubling the browser zoom to 200% reflows rather than
                breaking
              - The DevTools layout overlay shows the tool you claimed

ASK:          Before any code, give me a table with one row per layout
              problem: the tool, the one-sentence justification, and the
              main risk of that choice. Flag anything where a reasonable
              developer would pick the other tool. Wait for my approval.
```

## Stuck? Ask AI
```prompt
Explain the difference between flexbox and grid using an analogy that is not about queues, trays or newspapers. Then give me eight real layout problems — a nav bar, a photo gallery, a card's insides, a dashboard, and four more — and ask me which tool I would pick for each and why. Do not tell me the answers until I have attempted all eight.
```

```prompt
My layout looks correct on my laptop and breaks on a 360px phone. Give me an ordered diagnostic list of the eight most likely causes, cheapest check first. For each, tell me exactly what to look for in DevTools and what the correct fix is — not a patch that hides it.
```

```prompt
Explain what position: absolute actually does to an element, and why it is the right tool for a dropdown but the wrong tool for a nav bar. Then show me three layouts where an AI would plausibly reach for absolute positioning, and what it should have used instead.
```

## Quiz

### Q1 · L3
You need a navigation bar with a logo pinned left and three links pinned right. Which tool and why?
- (a) Grid, because a bar has rows and columns
- (b) Flexbox, because the items flow along one direction and nothing must align across it ✅
- (c) Absolute positioning, because the logo and links have fixed places
- (d) A table, because the items sit in a row
> Nothing below the bar has to line up with link number two, so the problem has one direction. Grid would work but costs more CSS; absolute positioning is what causes the logo to sit on top of the links at 360px.

### Q2 · L3
A dish card list uses `width: 300px` on each card inside a flex row. What happens on a 360px phone?
- (a) The cards shrink proportionally and all is well
- (b) The row overflows and the page scrolls sideways, or the cards squash ✅
- (c) The browser automatically wraps them onto new lines
- (d) Only the first card renders
> A fixed width is a demand, not a negotiation. Three 300px cards need 900px plus gaps. Items need `flex: 1` or a grid track like `minmax(160px, 1fr)` so they can adapt.

### Q3 · L4
Which requirement forces you towards grid rather than flexbox?
- (a) The items should sit in a row with even spacing
- (b) Each card's Add button must line up with the Add buttons in other columns ✅
- (c) The logo should sit at the far left
- (d) Items should be vertically centred in the bar
> Cross-direction alignment is the signature of a two-direction problem. The other three are single-line arrangements, which is exactly what flexbox is for.

### Q4 · L4
An AI-generated layout looks perfect with its six sample cards. What is the most informative single stress test?
- (a) Change the background colour
- (b) Replace one dish name with a 45-character name and look at the row ✅
- (c) Reload the page twice
- (d) Check that the CSS file is minified
> Sample content is uniform and real content is not. The long-name test exposes clipping, overlap, ragged button rows and fixed heights all at once, and it takes ten seconds in the Elements panel.

### Q5 · L5
Your layout overflows at 360px and the AI's fix is to add `overflow: hidden` to the container. Why should you reject it?
- (a) `overflow: hidden` is not supported on mobile browsers
- (b) It hides the symptom while the content is still too wide, so users simply cannot reach it ✅
- (c) It will break the page at desktop widths instead
- (d) It makes the page slower to render
> The price column did not get narrower; it got unreachable. This is the layout equivalent of `!important` — a change that removes your evidence of the bug without removing the bug.

## Go deeper
- [MDN: Flexbox](https://developer.mozilla.org/en-US/docs/Learn_web_development/Core/CSS_layout/Flexbox)
- [MDN: CSS grid layout](https://developer.mozilla.org/en-US/docs/Learn_web_development/Core/CSS_layout/Grids)
