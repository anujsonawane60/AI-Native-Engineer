---
id: m03-02-wireframes
module: m03-design
slug: wireframes-lofi-to-hifi
title: "Wireframes: Lo-Fi to Hi-Fi"
order: 2
duration_min: 15
is_free: false
mission: false
volatility: low
last_reviewed: 2026-11-16
---

## Key learnings
- Decide layout and priority on paper before any colour or font exists
- Move through the four fidelity levels in order, and know what each one is for
- Describe a wireframe in words precisely enough for AI to build it
- Draw the empty, loading, error and full states of every screen, not just the pretty one
- Recognise when high fidelity is hiding an unresolved layout problem

## Lesson

### The pain

A team spends two days making their canteen app beautiful: rounded cards, a gradient, a hero thali image. Then someone asks "where do I see that I already have three items in the cart?" There is nowhere, and fixing it moves the header, breaks the hero and regenerates the screen. Two days of polish undone by a question a pencil sketch would have caught in ninety seconds.

### The idea

**A wireframe decides what goes where and what matters most, deliberately before it decides what anything looks like.**

Raise fidelity one level at a time, and only when the current level has no unanswered questions left:

- **0 paper** — ten minutes, a pen, one frame per step of your chapter-1 flow
- **1 lo-fi** — grey boxes at correct sizes; tests hierarchy and nothing else
- **2 mid-fi** — real dish names and real ₹ prices; exists to break your layout
- **3 hi-fi** — type, colour, states. A layout question still open here means go back a level, not solve it in paint

### The analogy

Before the canteen is built, someone draws the room on paper: counter here, seating there, cashier where the queue bends. No tiles, no paint, no typography — the drawing answers one question, whether the room works when sixty people arrive at once.

**Where the analogy breaks:** rearranging a canteen means breaking concrete, so paper is obviously worth it. Your interface regenerates in forty seconds, which tempts everyone to skip to hi-fi. Regenerating is cheap but *deciding* is not, and a hi-fi screen hides the decision under attractive surface until you stop seeing it.

### Picture it

```
 LEVEL 0        LEVEL 1          LEVEL 2            LEVEL 3
 paper          lo-fi            mid-fi             hi-fi
┌────────┐    ┌──────────┐    ┌──────────┐      ┌──────────┐
│ ▭▭▭    │    │ ████     │    │ Today    │      │ Today    │
│ ▭▭▭▭▭▭ │───▶│ ▭▭▭▭▭▭▭▭ │───▶│ Vada pav │ ───▶ │ Vada pav │
│ [▭▭]   │    │ ▭▭▭▭     │    │ ₹15      │      │ ₹15  ●   │
│        │    │ [ BTN  ] │    │ [Order]  │      │ [Order]  │
└────────┘    └──────────┘    └──────────┘      └──────────┘
  layout       hierarchy       real words         type,
  only         + sizes         + real data        colour,
                                                  states
```

Each level answers one question and hands the screen on: paper decides what is on it, lo-fi what is biggest, mid-fi swaps lorem ipsum for real words and prices, hi-fi adds type, colour and states.

You describe the screen and AI draws it, so write regions top to bottom with purpose and relative size, then name the one primary action.

```
SCREEN: Today (first screen after open)
 1 Header, fixed, small: campus name left, cart count right
 2 Strip, one line: "Open till 4:30 pm" or "Closed"
 3 Primary block, largest on screen: "Order again - Vada pav
   x2, Rs 30" with one big button
 4 List, scrolls: dish name + price, sold-out greyed and
   labelled, never hidden
PRIMARY ACTION: the Order again button; all else quieter
```

Naming one primary action is the instruction AI most needs and least often gets. Without it every button comes back the same size.

### Where it lives

Wireframes sit between the chapter-1 flow and the visual design of chapters 5 to 7.

- **Chapter 4** — lo-fi frames are what you test; sketches get honest criticism, polished mocks get polite praise
- **M02** — a wireframe is the decision your HTML structure expresses
- **M11** — your frames side by side are the checkpoint evidence

### Words to own

| Term | Plain meaning | Example |
|---|---|---|
| Fidelity | How finished a design looks | Paper sketch versus hi-fi mock |
| Wireframe | Layout and priority without visual styling | Grey boxes, correct sizes |
| Mockup | A hi-fi still image of the screen | Final colours and type |
| Prototype | A clickable version you can walk through | Three screens linked |
| Hierarchy | What the eye notices, in order | Price bigger than description |
| Primary action | The one thing you most want tapped | "Order again" |
| Empty state | The screen with no data in it, done well | "No orders yet" plus menu |
| Lorem ipsum | Placeholder text standing in for real words | Dangerous past lo-fi |
| Above the fold | Visible before any scrolling | Top 600px on a 5-inch phone |
| Annotation | A note on a frame explaining behaviour | "Sticks on scroll" |

### Tradeoffs and gotchas

Every screen that shows data has at least four states, and AI hands you one unless you ask.

| State | When it shows | Canteen example |
|---|---|---|
| Empty | No data yet, correctly | First-time user, no past orders |
| Loading | Data is on its way | Slow 4G, menu still arriving |
| Error | The request failed | No network, or the server is down |
| Full | The normal case | Six dishes, three in cart |
| Edge | Real data at its ugliest | 40-character dish name, ₹1,250, all sold out |

**The empty state has the highest return.** A blank rectangle says "broken"; "No orders yet — here's today's menu" plus a button says "working, and here is what to do".

**Hi-fi too early is expensive twice.** Redoing it costs time, and looking finished stops criticism. A polished screen gets "nice!"; a sketch gets "where's the cart?"

**Lorem ipsum lies about your layout.** "Misal Pav with extra farsan" is four lines, and ₹1,250 is wider than ₹15.

**Annotate behaviour a still image cannot show** — "header sticks on scroll", "tapping a sold-out dish does nothing". Unannotated frames get interpreted, and not by you.

### Verify it

Run these five checks on every wireframe AI returns.

1. **The squint test.** Shrink it to a thumbnail; the first thing you can still make out must be your primary action.
2. **Count the primary actions** — exactly one per screen. Three equal buttons means AI could not tell what mattered.
3. **Ask for the other states explicitly**: "show this screen with no data, loading, and after a failed request."
4. **Paste in your worst real content** — longest dish name, ₹1,250, a 30-character user name. Overflow means it was built for lorem ipsum.
5. **Check lo-fi stayed lo-fi.** Any colour, shadow or icon means the layout decision is buried under styling you never evaluated.

Red flags: every element the same size; placeholder text in a mid-fi frame; no empty state; a design that works at exactly one screen width.

## Walkthrough
1. Take the primary flow you wrote in chapter 1 and count its steps. Draw that many empty rectangles on paper, one per step, and label each with the step name.
2. For each frame, write down the three to five things that must be on it, in order of importance. Decide the order before you draw anything.
3. Sketch the first frame six different ways in ten minutes. Force yourself to the sixth — the useful idea is usually the fourth or fifth. Do not erase, just draw again.
4. Pick one sketch and mark the single primary action with a circle. If you cannot choose, the screen is doing two jobs and needs splitting.
5. Write the frame as a text wireframe: numbered regions top to bottom, each with purpose and relative size, plus one line naming the primary action.
6. Brief AI from that text and demand lo-fi only — grey boxes, no colour, no icons, no font selection. Reject anything styled and restate the constraint.
7. Squint at what comes back, at thumbnail size. Confirm the primary action is still the most visible thing. If not, write one precise correction naming what should be larger and what should be quieter.
8. Replace every placeholder with real content from your own product — longest name you have, a four-digit price, a real person's name. Note everything that breaks.
9. Ask explicitly for the empty, loading and error states of this screen, and write what each one should say in your own words before you look at the result.
10. Annotate the final frame with three behaviour notes a still image cannot show, then save all your frames side by side. That sequence is your evidence for the M11 checkpoint.

## Talk to AI
```prompt
CONTEXT:      I am designing one screen of a campus food-ordering app with
              my team of three. Users are students on budget Android
              phones, screens around 5 inches, often on slow 4G and often
              outdoors in bright sunlight. I am at the wireframe stage and
              I want layout decisions, not visual design.

GOAL:         A LOW-FIDELITY wireframe of the screen described below, in
              which the priority of every element is unmistakable.

MY TEXT WIREFRAME:
SCREEN: Today - the first screen after opening the app
 1 Header, fixed, small: campus name left, cart count right
 2 Status strip, one line: "Open till 4:30 pm" or "Closed now"
 3 Primary block, the largest thing on screen: reorder the
   user's last order, showing items and total in rupees, with
   one large button
 4 Dish list, scrolls: each row is name + price in rupees;
   sold-out dishes are greyed and labelled, never hidden
 5 No bottom navigation on this screen
PRIMARY ACTION: the reorder button. Everything else must be
   visibly quieter than it.

REQUIREMENTS: 1. Lo-fi only: grey blocks, placeholder bars for text
              2. Correct relative sizes, so the reorder block clearly
                 dominates
              3. Produce FOUR versions of this screen: full, empty
                 (no past orders), loading, and failed-to-load
              4. Annotate each frame with behaviour a still image cannot
                 show
              5. State in words which element you expect to be noticed
                 first, second and third

CONSTRAINTS:  No colour beyond greys. No icons. No font choices. No
              gradients, shadows or rounded decoration. Must work at
              320px wide with no horizontal scrolling. One primary
              action only.

NON-GOALS:    No visual design. No brand. No animation. Do not add
              search, filters, ratings, categories or any feature not
              listed above. Do not replace my region order.

ACCEPTANCE:   - At thumbnail size, the reorder block is still the most
                visible thing
              - Exactly one element reads as the primary action
              - The empty state tells a first-time user what to do next,
                rather than showing a blank area
              - The loading state shows the shape of the content, not
                just a centred spinner
              - The error state says what failed and offers a retry
              - Nothing in any frame uses colour, icons or a chosen font

ASK:          First describe TWO different ways to arrange these five
              regions, with the tradeoff of each for a rushed student on
              slow 4G, and say which you would choose and why. Do not
              produce any frames until I pick one.
```

## Stuck? Ask AI
```prompt
Here is my screen description: [paste it]. Do not design anything. Instead list every state this screen can be in that I have not described — empty, loading, error, partial data, and anything specific to this screen. For each, tell me in one sentence what the user should see and why, then tell me which state I am most likely to forget.
```

```prompt
I keep being told my designs have no visual hierarchy but I cannot see it myself. Give me a step-by-step method for checking hierarchy on a screenshot using nothing but my own eyes and a phone — no tools, no plugins. Then give me three questions to ask about any screen that will reveal a hierarchy problem.
```

```prompt
Here is the real content from my product: [paste the longest name, the largest price, the longest user name, and one empty list]. Tell me, specifically, what each of these will do to a typical mobile card layout, and what decision I need to make in advance for each one. Do not write any code or CSS.
```

## Quiz

### Q1 · L3
Why is a lo-fi wireframe better than a polished mockup for showing to test users?
- (a) It is faster to produce, so you can show more of them
- (b) People criticise a sketch honestly and praise a polished screen politely ✅
- (c) Lo-fi designs load faster on slow connections
- (d) Users cannot understand high-fidelity designs
> Polish reads as "decided", so feedback softens into compliments. A sketch invites the blunt question — "where's the cart?" — which is the feedback you actually need.

### Q2 · L2
What question is the mid-fidelity level specifically for?
- (a) What colours and fonts should we use?
- (b) Does the layout survive real content — long names, big numbers, empty lists? ✅
- (c) Which animation should the button have?
- (d) How many users will tap this?
> Mid-fi exists to break your layout with reality. Lorem ipsum is uniformly short and tidy; "Misal Pav with extra farsan" and ₹1,250 are not.

### Q3 · L4
You asked for a wireframe and received a screen with a purple gradient, rounded cards and an icon set. What is the actual cost?
- (a) None — it saves a step later
- (b) The layout decision you were isolating is now buried under styling you did not evaluate ✅
- (c) It will take longer to load in the browser
- (d) The colours will be wrong for your brand
> Each fidelity level makes one kind of mistake cheap. Skipping straight to styling means the layout was never judged on its own, and nobody will question it once it looks finished.

### Q4 · L3
A new user opens your app for the first time and sees a blank rectangle where past orders would be. What has gone wrong?
- (a) A data-loading bug
- (b) The empty state was never designed, so the first screen any new user sees reads as broken ✅
- (c) The user needs onboarding screens
- (d) Nothing — there genuinely are no orders
> The data is correct; the design is missing. The empty state is the highest-return state to design because every new user starts there.

### Q5 · L5
Your team is three days into hi-fi polish when someone asks where the cart count appears. There is no good answer. What does this tell you about your process, not your design?
- (a) The team needs a better design tool
- (b) Fidelity was raised while a layout question was still unanswered ✅
- (c) The cart feature should be cut
- (d) The brief should have specified a colour palette earlier
> You only raise fidelity when the current level has no open questions. An unresolved placement question at hi-fi means three days of styling has to be redone around it.

## Go deeper
- [Nielsen Norman Group: Paper prototyping as a core tool](https://www.nngroup.com/articles/paper-prototyping/)
- [web.dev: Designing for slow networks and real devices](https://web.dev/learn/performance/)
