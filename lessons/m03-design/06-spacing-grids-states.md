---
id: m03-06-spacing-grids-states
module: m03-design
slug: spacing-grids-states
title: "Spacing, Grids and Component States"
order: 6
duration_min: 15
is_free: false
mission: false
volatility: low
last_reviewed: 2026-11-16
---

## Key learnings
- Use one spacing scale so every gap on every screen is a deliberate choice
- Group things with proximity before reaching for boxes, lines or shadows
- Specify all eight states of an interactive component, not just the resting one
- Size tap targets for a real thumb on a real phone
- Catch the two spacing failures AI produces most: random gaps and crowded taps

## Lesson

### The pain

On a team's cart screen the gap between a dish and its price is 14 px and the gap between dishes is 16 px, so each price reads as belonging to the dish below and a student reads the vada pav as ₹40. "Remove" sits 6 px from "Pay now", and two testers paid when they meant to remove.

### The idea

**Space is how you say what belongs together, and a component is not designed until all its states are.**

Proximity groups more strongly than a border or a shadow, and an interactive element is eight things sharing a shape.

### The analogy

At the canteen counter, plates and glasses sit in one cluster with a clear gap before the chai urn. Nobody drew a line — the gap does the work. The urn's tap has states too: empty, ready, mid-pour, and "wait two minutes", which the staff say because the urn cannot.

**Where the analogy breaks:** the counter is arranged once for one room. Your layout must hold at 320 px and on a tablet, so you work from a *scale* of gaps, not by eye.

### Picture it

```
  WRONG: gaps fight the meaning     RIGHT: gaps carry the meaning
┌────────────────────────────┐    ┌────────────────────────────┐
│ Vada pav                   │    │ Vada pav                   │
│                      14px  │    │ Rs 15              4px     │
│ Rs 15                      │    │                            │
│                      16px  │    │                     24px   │
│ Misal pav                  │    │                            │
│                      14px  │    │ Misal pav                  │
│ Rs 40                      │    │ Rs 40              4px     │
└────────────────────────────┘    └────────────────────────────┘
   price reads as belonging          price clearly belongs to
   to the dish BELOW it              the dish ABOVE it
```

On the left the gap inside a group is almost as large as the gap between groups, so the eye assigns each price to the wrong dish; on the right the inside gap is much smaller than the between gap, and the grouping is unambiguous without a single border.

Pick a base unit — 4 px — and allow only multiples of it:

| Token | Value | Typical use |
|---|---|---|
| xs | 4 px | Label to its value, icon to its text |
| sm | 8 px | Inside a button, between tight items |
| md | 16 px | Padding inside a card, between fields |
| lg | 24 px | Between cards, between groups |
| xl | 32 px | Between sections |
| 2xl | 48 px | Above and below a page heading |

Two laws: the inside gap is *clearly* smaller than the between gap, and no improvised values — 13 px means somebody nudged it. Say which you mean: padding is inside an element, margin outside. On mobile a grid mostly means a 16 px page margin on every screen.

Every interactive component has eight states:

| State | What it says | Common failure |
|---|---|---|
| Default | This is available | — |
| Hover | You are over it | Only matters on desktop |
| Focus | Keyboard is here now | Removed entirely — breaks chapter 9 |
| Active / pressed | Your tap registered | Missing, so taps feel dead |
| Loading / pending | It is working, wait | Missing, so users double-tap |
| Disabled | Not available, and why | Looks identical to default |
| Error | It failed, here is what to do | A number instead of a sentence |
| Success | It worked | Flashes too fast to notice |

The focus ring is never removed; disabled says why at 3:1 contrast ("Pay now — add an item first"); loading changes the text *and* blocks a second tap; an error says "your money was not deducted — try again or pay at the counter."

Tap targets are 44×44 px (Apple) or 48×48 dp (Google) — a fingertip, not the icon — with 8 px between them, more when one is destructive.

### Where it lives

Spacing and states are the second half of hi-fi.

- **Chapter 7** — these numbers freeze into tokens
- **Chapter 8** — consistent space is what makes generated work look intentional
- **Chapter 9** — needs focus and disabled states to be real
- **M11** — one scale across screens is evidence of direction

### Words to own

| Term | Plain meaning | Example |
|---|---|---|
| Spacing scale | The fixed list of gaps you allow | 4 / 8 / 16 / 24 / 32 / 48 |
| Padding | Space inside an element | Between button edge and its text |
| Margin | Space outside an element | Between two buttons |
| Proximity | Closeness as a grouping signal | Price 4 px under its dish |
| Gutter | The gap between grid columns | 16 px |
| Page margin | Space at the left and right edge | 16 px on every screen |
| Tap target | The tappable area, not the visible icon | 48×48 dp around a 20 px icon |
| State | A component's condition at a moment | Disabled, loading, error |
| Focus ring | The outline showing keyboard position | Never remove it |
| Optical alignment | Looking aligned rather than measuring aligned | A triangle nudged right |

### Tradeoffs and gotchas

**Tight spacing is not density.** For density, remove elements rather than shrink gaps.

**Borders are not substitutes for space.** Remove them; if the grouping still reads, they were never needed.

**Hover is nearly irrelevant.** Traffic is touch, so get *pressed* and *focus* right first.

**Disabled buttons can be a trap.** If the user cannot tell why, an enabled button that explains on tap is kinder.

### Verify it

Six checks, on a real phone.

1. **Measure two gaps per group.** An inside gap over half the between gap means the grouping is lying.
2. **List every distinct gap value**; anything off your scale was improvised.
3. **Check the left edge**: everything starts at the page margin.
4. **Tap everything with your thumb, standing, one-handed.** Two mis-hits means too small or too close.
5. **Force all eight states and photograph each**: network off, empty form, tab for a focus ring, empty cart. A state you cannot produce does not exist.
6. **Measure the destructive action's distance from the nearest safe one.**

Red flags: gaps like 13 px or 22 px; every group in a bordered card; no focus ring; a disabled state identical to default; no pending state on submit; AI calling spacing "clean" instead of naming values.

## Walkthrough
1. Screenshot one data-heavy screen of your product on a real phone and open it in any image viewer where you can read pixel distances.
2. Pick one group — a dish and its price, or a label and its value — and measure the gap inside it. Then measure the gap to the next group. Write both numbers down.
3. If the inside gap is not clearly smaller than the between gap, you have found the cause of every "it feels cluttered" comment your team has made.
4. List every distinct gap value visible on the screen. Count how many are not multiples of 4. Those are the improvised ones.
5. Write your spacing scale as six named steps with values, and write your page margin as a single number used on every screen.
6. Lay a straight edge down the left side of the screenshot. Note every element that does not start at the page margin.
7. Pick the most important interactive component on the screen and write out all eight states, including exactly what text the disabled and error states show.
8. Force each state on a real device: empty cart, network off, form submitted blank, keyboard tabbed through. Photograph each one. List the states you could not produce.
9. Brief AI with your scale, your page margin and your eight-state table, and ask it to apply them without changing layout, copy or colour. Demand that it list which values it changed.
10. Re-test with your thumb while standing, one-handed. Record every mis-tap and measure the gap between the two controls involved.

## Talk to AI
```prompt
CONTEXT:      I am a student founder directing AI to build my product. I
              do not write code. Users are on budget Android phones in
              India, screens 320 to 412px wide, used one-handed while
              walking, often on slow 4G. Some interactions cost money.

              The screen, region by region top to bottom, with the exact
              text on every element: [paste]
              The groups that belong together: [e.g. "dish name + price
              + veg mark are one group; each dish is a separate group"]
              The destructive actions on this screen: [list them]

GOAL:         A spacing system and a complete set of component states
              for this screen, so that grouping is unambiguous and no
              interaction can be mis-tapped or double-submitted.

REQUIREMENTS: 1. A spacing scale of exactly six named steps, all
                 multiples of 4px, stated as a table with where each is
                 used
              2. One page margin value used on every screen, stated once
              3. For each group I named: the gap INSIDE it and the gap
                 BETWEEN it and its neighbour, with the inside gap no
                 more than half the between gap
              4. All EIGHT states for every interactive element:
                 default, hover, focus, pressed, loading, disabled,
                 error, success - with the exact words shown in the
                 disabled, loading and error states
              5. Tap target sizes in dp for every control, including the
                 invisible padding around small icons
              6. The minimum distance between each destructive action
                 and the nearest safe control

CONSTRAINTS:  All spacing from the scale, no improvised values. Minimum
              48x48dp tap targets. Minimum 8px between adjacent targets
              and much more around destructive ones. Focus rings must
              never be removed. Disabled states must keep 3:1 contrast
              and must state why they are disabled. Must hold at 320px
              wide with no horizontal scrolling. Do not change my
              layout order, my copy or my colours.

NON-GOALS:    No new components. No animation or transitions. No
              gradients or shadows - solve grouping with space first.
              No dark mode. Do not add borders or cards to create
              grouping that spacing should carry.

ACCEPTANCE:   - Every gap on the screen maps to a named step in my scale
              - Every group's inside gap is at most half its between gap
              - Each of the eight states is described distinctly, and
                disabled is visibly different from default
              - The loading state changes the button text, shows
                progress AND makes the control untappable
              - The error wording tells the user what happened and
                whether their money moved
              - A final list: every element you could not assign a
                state to, and what you need from me

ASK:          First tell me which TWO groupings on this screen are most
              likely to be misread by a user, and why, using only
              spacing as your evidence. Then propose the system.
```

## Stuck? Ask AI
```prompt
Explain the difference between padding and margin using a physical object, not a web page. Then give me six descriptions of layout problems in plain language and ask me, one at a time, whether each is a padding problem or a margin problem. Do not tell me the answers in advance.
```

```prompt
Here is one button in my app and what it does: [describe it, including what happens when it is tapped and what could go wrong]. Write out all eight of its states and the exact words the user sees in each. For the disabled, loading and error states, explain what a user would wrongly conclude if that state were missing.
```

```prompt
Here are all the gap values I measured on one screen: [list them]. Tell me which ones are improvised rather than from a scale, group them into the smallest sensible scale of six steps, and tell me which single gap change would most improve how grouped the screen feels. Do not redesign the screen.
```

## Quiz

### Q1 · L4
A dish name sits 14 px above its price, and 16 px above the next dish. Why do students misread the prices?
- (a) The font size of the price is too small
- (b) The inside gap is nearly as large as the between gap, so grouping is ambiguous ✅
- (c) There is no border separating the dishes
- (d) The prices should be right-aligned
> Proximity is the strongest grouping signal. When the gap inside a group approaches the gap between groups, the eye assigns elements to the wrong group — and a border would only paper over it.

### Q2 · L3
Which state is most responsible for preventing duplicate orders?
- (a) Hover
- (b) Loading, with the control made untappable ✅
- (c) Success
- (d) Disabled
> Without immediate feedback that the tap registered, users re-tap. The loading state must change the text, show progress and block further taps — all three.

### Q3 · L2
Your icon is 20 px. What should the tap target be?
- (a) 20 px — matching the visible icon
- (b) At least 44 to 48 px, achieved by padding around the icon ✅
- (c) 32 px, as a compromise between looks and usability
- (d) It depends only on the screen width
> The target is sized for a fingertip, not for the artwork. Padding grows the tappable area without changing how the icon looks.

### Q4 · L4
AI removes the focus ring from your buttons because it "looks cleaner". What is the consequence?
- (a) Nothing meaningful — focus rings are a desktop legacy
- (b) Keyboard and switch-access users can no longer tell where they are on the page ✅
- (c) The buttons become harder to tap on touchscreens
- (d) The hover state stops working
> The focus ring is the only indicator of keyboard position. Removing it is a hard accessibility failure, covered in chapter 9, and it is one of the most common things AI strips.

### Q5 · L5
Your spacing audit finds gaps of 6, 13, 14, 16, 17 and 22 px on one screen. What is the right correction brief?
- (a) Change the 13 px gap to 12 px and leave the rest
- (b) State a six-step scale of multiples of 4 and ask for it to be reapplied screen-wide, listing what changed ✅
- (c) Add borders around each group so the gaps matter less
- (d) Reduce every gap by 2 px for consistency
> Fixing gaps one at a time rebuilds the same mess next session. The durable fix is to name the system, apply it everywhere, and demand a list of what changed so you can review it.

## Go deeper
- [Material Design: Accessibility — touch target sizing](https://m3.material.io/foundations/designing/structure)
- [Nielsen Norman Group: Proximity and the Gestalt principle of grouping](https://www.nngroup.com/articles/gestalt-proximity/)
