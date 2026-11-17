---
id: m02-06-responsive
module: m02-the-web
slug: responsive-design
title: "Responsive Design: Viewports, Breakpoints and Units"
order: 6
duration_min: 15
is_free: false
mission: false
volatility: low
last_reviewed: 2026-11-16
---

## Key learnings
- Design for a range of widths rather than for two named devices
- Explain what the viewport meta tag does and what happens without it
- Choose between px, rem, % and viewport units for a given job
- Say why mobile-first produces less CSS and fewer surprises than desktop-first
- Name the five responsive red flags that appear in almost every AI first draft

## Lesson

### The pain

A team's canteen app looks excellent on the presenter's laptop. A mentor opens it on her two-year-old Android at 360 x 800 and the page arrives as a tiny postcard in the middle of the screen, because the HTML has no viewport meta tag and the phone assumed a 980px desktop. She pinch-zooms and now the page is wider than the screen. The "Add to order" button is 22 pixels tall — half a fingertip — and she taps "Remove" by accident.

None of this is a design failure. Each is a two-line fix nobody checked for, because everybody tested on the machine they built on.

### The idea

**You are not designing for a phone and a laptop. You are designing for a continuous range of widths, and the layout should be correct at every one of them — including widths no device has yet.**

The practical version is **mobile-first**: write the narrowest layout as the default, then add rules that apply only as the screen gets wider. That produces less CSS and makes the default favour the users least able to tolerate a broken page.

### The analogy

You cannot build one serving window for short people and a second for tall people. You set the counter height so a first-year of 150cm and a final-year of 190cm are both served without crouching or reaching. Notice which end you design from: set it for the tallest and the short student is stuck; set it for the shortest and everybody else manages. That is mobile-first.

**Where the analogy breaks:** the canteen knows its range — nobody 400cm tall is coming. Your page will open on a 320px phone, a 1440px laptop, a television, a watch, and a window somebody dragged to 180px wide. Hence flexible rules, not a list of known devices.

### Picture it

```
  320px     360px        768px            1024px        1440px+
  ├──────────┼─────────────┼────────────────┼──────────────┤
  │ 1 column │ 1 column    │ 2 columns      │ 3 columns    │
  │ stacked  │ stacked     │ side nav       │ wider gutters│
  ├──────────┴─────────────┴────────────────┴──────────────┤
  │  DEFAULT (no media query needed)                       │
  │  ───────────▶ add rules as the screen gets wider ───▶  │
  └────────────────────────────────────────────────────────┘
```

The narrow layout is the default that needs no media query at all; each breakpoint above it adds only the rules that the extra space makes possible.

### Where it lives

One line in the `<head>` decides whether anything else in this chapter matters:

```
<meta name="viewport" content="width=device-width, initial-scale=1" />
```

Without it a phone pretends its screen is 980 CSS pixels wide, renders the layout at that size and scales the result down, so your media queries never fire and every font is unreadable. That is the postcard effect, and the first thing to check when a page looks "zoomed out".

Responsive behaviour is entirely client-side in M00's four boxes: the server sends one page and the browser arranges it. Detecting the device on the server and sending a different page — which an AI sometimes suggests — needs a device database and breaks on every new phone. This chapter sits on chapter 5's tools: an `auto-fit` grid is responsive before any media query.

### Words to own

| Term | Plain meaning | Example |
|---|---|---|
| Viewport | The visible area of the page in the browser | 360 x 800 on a phone |
| Viewport meta tag | Tells a phone to use its real width | `width=device-width` |
| CSS pixel | The unit CSS works in, not a hardware pixel | A 360px-wide phone |
| Device pixel ratio | How many hardware pixels per CSS pixel | 3x on many Androids |
| Breakpoint | A width at which your rules change | 768px |
| Media query | A rule that applies only above or below a width | `@media (min-width: 768px)` |
| Mobile-first | Narrow layout as the default, add rules upward | `min-width` queries |
| `rem` | A multiple of the root font size, so it respects user settings | `1.125rem` |
| `%` | A share of the parent's size | `width: 100%` |
| `vw` / `vh` | A hundredth of the viewport's width / height | `50vw` |
| `clamp()` | A value with a floor, a preference and a ceiling | Fluid font sizes |
| Tap target | The area a finger can reliably hit | At least 44 x 44 CSS px |

### Tradeoffs and gotchas

**Pixel font sizes break the user's own accessibility settings.** A student who set a larger default font size gets no benefit from `font-size: 14px`, because px is absolute; `rem` scales with that setting.

**Too many breakpoints means the layout is not flexible.** Three is plenty. Seven, including 420px and 480px, is a rigid layout being patched wherever it happens to break.

**`100vh` is a lie on mobile.** The address bar overlaps the viewport, so a `100vh` element is taller than the visible area and the page scrolls 60-ish pixels for no reason. Use `100dvh`.

**Hiding content on small screens is not responsive design.** `display: none` below 768px is how a mobile version loses the price filter mobile users need most. Rearrange rather than delete.

**Tap targets are measurable, not a feeling.** 44 x 44 CSS pixels is the floor, and the gap to the neighbour matters as much — two correct buttons 2px apart still produce mis-taps. That is how "Pay ₹240" next to "Cancel" gets dangerous on a bumpy bus.

### Verify it

1. **Check the viewport meta tag first.** Missing, `maximum-scale=1` or `user-scalable=no` are all failures; the last two block the pinch zoom some users read with.
2. **Sweep the width, do not sample it.** Drag slowly from 1440px to 320px and note every width where something clips, overlaps or scrolls sideways — tablets and split-screen windows live between your favourite two sizes.
3. **Test 320px specifically.** The narrowest width worth supporting, and the one that exposes fixed widths.
4. **Zoom to 200% at desktop width.** The page should reflow, not break.
5. **Measure the tap targets.** Under 44px in the box model is a finding, and so is the gap to the neighbour.
6. **Open it on the cheapest Android in your group, on mobile data.** The step people skip, and the only one that tells the truth.
7. **Check nothing was hidden.** Compare mobile against desktop feature by feature.

Red flags: no viewport meta tag · `user-scalable=no` · `font-size` in px throughout · more than four breakpoints · `display: none` as the mobile strategy · `100vh` on a main section · a fixed px `width` on a container · media queries written only with `max-width`, which is desktop-first in disguise.

## Walkthrough
1. Open any Indian consumer site on a laptop — a food delivery app, a ticket booking site, your college portal. View source and search for `viewport`. Record exactly what the tag says, or that it is absent.
2. Open the device toolbar and set the width to 1440px. Screenshot the layout and note the number of columns and the overall structure.
3. Now drag the viewport width slowly down towards 320px. Do not jump. Write down every width at which the layout visibly changes — those are the site's real breakpoints.
4. Keep going and record the first width at which anything breaks: text clipping, elements overlapping, or a horizontal scrollbar appearing.
5. At 360px, inspect the primary call-to-action button and read its height and width from the DevTools box model. Compare against the 44 x 44 floor. Then measure the gap to the nearest other tappable thing.
6. Inspect a paragraph of body text and check in the Computed tab whether its `font-size` resolves from `px` or `rem`. Then change your browser's default font size to "Very large" in settings and reload to see whether the text responds.
7. Back at desktop width, zoom the browser to 200% and reload. Note what breaks here that did not break at 360px.
8. Compare the 360px view against the 1440px view feature by feature. List anything that exists on desktop and is missing on mobile, and judge whether the omission looks deliberate.
9. Search the page's stylesheet for `@media` and count the queries. Note whether they use `min-width` (mobile-first) or `max-width` (desktop-first).
10. Open the same page on the cheapest Android phone you can borrow, on mobile data rather than Wi-Fi, and compare your DevTools findings against what you actually experience. Write 100 words on the gap.

## Talk to AI
```prompt
CONTEXT:      I direct AI rather than writing CSS. My product is a canteen
              ordering app for students in Nashik. Roughly 85% of traffic is
              budget Android phones at 360px wide, a few at 320px; the rest
              is laptops at 1366px and 1440px. Many users are outdoors in
              bright sunlight. Some have set a larger default font size in
              their browser. The main actions are "Add to order" and a
              "Pay" button that triggers a UPI payment.

GOAL:         A responsive strategy for this product that I can hold the
              whole team and the AI agent to, plus the minimum CSS
              foundation that enforces it.

REQUIREMENTS: 1. The exact viewport meta tag I should use, and what each
                 part of it does
              2. A mobile-first breakpoint set — no more than three
                 breakpoints — with a one-line justification for each
                 chosen width based on my actual traffic
              3. A rule for which unit to use for what: font sizes,
                 spacing, container widths, and anything full-height.
                 Include why, not just what
              4. Minimum tap-target size and minimum spacing between
                 adjacent tap targets, stated as numbers, with specific
                 attention to the Pay button sitting next to Cancel
              5. A short written list of things we will never do, e.g.
                 hiding content below a breakpoint

CONSTRAINTS:  Plain CSS, no framework. Mobile-first only: media queries use
              min-width, never max-width. No font sizes in px. No fixed
              container widths in px. No 100vh on a main section. Pinch
              zoom must never be disabled. Nothing may be hidden on small
              screens to make a layout fit.

NON-GOALS:    No colours, type choices or visual design. No dark mode. No
              JavaScript-based device detection, and no server-side device
              detection — tell me plainly why both are wrong here.

ACCEPTANCE:   - No horizontal scrolling anywhere between 320px and 1440px
              - Body text grows when I change the browser's default font
                size to "Very large"
              - At 200% browser zoom the page reflows rather than breaking
              - The Pay button measures at least 44x44 CSS pixels and sits
                at least 12px clear of any other tappable element
              - Every feature available at 1440px is reachable at 360px
              - Counting @media blocks in the stylesheet gives three or
                fewer

ASK:          First, before any code, propose two candidate breakpoint sets
              for my specific traffic mix, with the tradeoffs of each, and
              recommend one with your reasoning. Then list the units you
              would use for each of my five categories as a table. Wait for
              my approval before writing any CSS.
```

## Stuck? Ask AI
```prompt
Explain the difference between px, rem, %, vw and clamp() by giving me one job that each is genuinely the best choice for and one job where using it would be a mistake. Then give me eight real sizing decisions from a mobile app and ask me which unit I would pick for each, without telling me the answers first.
```

```prompt
My page appears as a tiny shrunken version of the desktop layout when I open it on a phone, with everything proportional but unreadably small. Explain exactly what the browser is doing and why, what single line fixes it, and what each part of that line means. Then tell me three other symptoms that have the same cause.
```

```prompt
I keep being told "mobile-first" but I do not understand why the order matters if the end result looks the same. Walk me through the same two-column-to-one-column layout written mobile-first and desktop-first, compare how much CSS each needs, and explain which users suffer when the browser fails to apply a media query in each case.
```

## Quiz

### Q1 · L3
A page opens on a phone as a tiny but perfectly proportioned version of the desktop layout. What is the cause?
- (a) The CSS file failed to load
- (b) The viewport meta tag is missing, so the phone assumed a ~980px desktop page and scaled it down ✅
- (c) The breakpoints were written with `min-width` instead of `max-width`
- (d) The phone's device pixel ratio is 3x
> Without the tag the browser never uses the real screen width, so no media query fires. It is one line in the `<head>` and it is the first thing to check for this symptom.

### Q2 · L3
Why is `font-size: 14px` on body text worse than `font-size: 0.875rem`?
- (a) px renders more slowly than rem
- (b) px ignores the user's own default font-size setting, so someone who needs larger text gets nothing ✅
- (c) rem works on more browsers
- (d) px is not supported inside media queries
> `rem` is a multiple of the root font size, which the user controls in their browser settings. An absolute px value silently overrides an accessibility choice somebody made deliberately.

### Q3 · L4
An AI delivers a mobile layout that uses `display: none` below 768px to hide the price filter. What is wrong with this?
- (a) Nothing — mobile screens cannot fit a filter
- (b) `display: none` is slow on mobile devices
- (c) Mobile users lose a feature they need most, when the correct answer is to rearrange it ✅
- (d) It will also hide the filter on tablets in landscape
> Hiding is not responding. Mobile is the majority of this product's traffic, so the device with the least room is being given the fewest features — exactly backwards.

### Q4 · L4
You tested at 360px and at 1440px and everything looked fine. A user reports a broken layout. What did your testing method miss?
- (a) Nothing; the user's browser must be faulty
- (b) The widths in between, where tablets and split-screen windows live ✅
- (c) The page needed a hard reload
- (d) The device pixel ratio of the user's phone
> Sampling two widths tests two widths. Dragging the viewport slowly from 1440px to 320px finds the exact width where something first breaks, which is almost never one of the two you picked.

### Q5 · L5
Your "Pay ₹240" button measures 40 x 36 CSS pixels and sits 3px from the Cancel button. Both are inside the 320px layout and nothing overlaps. Is this acceptable?
- (a) Yes — nothing overlaps and both fit on screen
- (b) No — both the target size and the spacing are below the usable floor, and the consequence is a mis-tapped payment ✅
- (c) Yes, provided the buttons have different colours
- (d) No, but only because of the width; 36px tall is fine
> Fitting on screen is not the same as being hittable. 44 x 44 is the usual floor, and adjacency matters as much as size — a small Pay button 3px from Cancel is a money bug on a moving bus.

## Go deeper
- [MDN: Responsive design](https://developer.mozilla.org/en-US/docs/Learn_web_development/Core/CSS_layout/Responsive_Design)
- [web.dev: Responsive web design basics](https://web.dev/articles/responsive-web-design-basics)
