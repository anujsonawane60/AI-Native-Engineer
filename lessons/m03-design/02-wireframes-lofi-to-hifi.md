---
id: m03-02-wireframes
module: m03-design
slug: wireframes-lofi-to-hifi
title: "Wireframes: Lo-Fi to Hi-Fi"
order: 2
duration_min: 25
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

A team spends two days getting their canteen app to look beautiful. Rounded cards, a soft gradient, a nice font, a hero image of a thali. They show it in the daily session and someone asks a simple question: "where do I see that I already have three items in the cart?"

There is nowhere. The cart lives behind a tab at the bottom that scrolls off. Fixing it means moving the header, which means the hero image no longer fits, which means the gradient breaks, which means re-generating most of the screen. Two days of polish, undone by one question that a pencil sketch would have surfaced in ninety seconds.

The opposite failure is just as common. Another team sketches for a week, produces thirty beautiful grey boxes, and never finds out that their main action is invisible in sunlight — because grey boxes have no contrast problem to find.

Fidelity is a dial. Turning it too fast and turning it too slowly both cost you days.

### The idea

**A wireframe decides what goes where and what matters most, deliberately before it decides what anything looks like.**

You raise fidelity one level at a time, and you only raise it when the current level has no unanswered questions left. Each level is designed to make a different kind of mistake cheap: a sketch makes layout mistakes cost a pencil stroke, a hi-fi mock makes contrast mistakes cost a token change.

### The analogy

Before the canteen is built, someone draws the room on paper. Counter here, seating there, hand-wash by the exit, cashier where the queue naturally bends. That drawing has no tiles, no paint, no menu board typography. It is answering one question: does the room work when sixty people arrive at once?

Once the layout is settled they choose tiles and paint and the board. Nobody chooses the tiles first and then tries to arrange the counter around them.

**Where the analogy breaks:** a canteen is built once and rearranging it means breaking concrete, so the paper stage is obviously worth it. Your interface can be regenerated in forty seconds, which tempts everyone to skip straight to high fidelity and iterate there. The trap is that regenerating is cheap but *deciding* is not — and a hi-fi screen hides the decision under a lot of attractive surface, so you stop seeing it.

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

Each level answers one question and then hands the screen on: paper decides what is on it, lo-fi decides what is biggest, mid-fi replaces lorem ipsum with real words and real prices, hi-fi adds type, colour and every interaction state.

### What each level is for

**Level 0 — paper.** Ten minutes, a pen, no device. You are deciding what exists on the screen at all and in what order down the page. Six sketches of the same screen is normal and healthy; the point of paper is that throwing one away costs nothing. Do this for every step in the flow you wrote in chapter 1 — one frame per step, so a three-step flow is three frames.

**Level 1 — lo-fi.** Grey boxes, correct sizes, correct positions. No colour, no real font, placeholder text. What you are testing here is hierarchy: squint at it, and the thing you notice first should be the most important thing on the screen. If the biggest element is a decorative image and the smallest is the price, you found a problem for free.

**Level 2 — mid-fi.** Replace every placeholder with real content. Real dish names, real prices in ₹, a real student's name, the longest name you can actually find. This level exists to break your layout, and it reliably does. "Misal Pav with extra farsan and kanda" is four lines, not one. A dish at ₹1,250 is wider than one at ₹15. An empty order history is a screen you had not drawn.

**Level 3 — hi-fi.** Now type, colour, spacing, icons, shadows and every state. This is where chapters 5 to 8 live. By the time you arrive, no layout question should remain open — if one does, go back down a level rather than solving it in paint.

### The four states nobody draws

Every screen that shows data has at least four states, and AI will hand you exactly one of them unless you ask.

| State | When it shows | Canteen example |
|---|---|---|
| Empty | No data yet, correctly | First-time user, no past orders |
| Loading | Data is on its way | Slow 4G, menu still arriving |
| Error | The request failed | No network, or the server is down |
| Full | The normal case | Six dishes, three in cart |
| Edge | Real data at its ugliest | 40-character dish name, ₹1,250, all sold out |

The empty state is the one with the highest return on effort, because it is the first screen a new user ever sees. A blank rectangle says "broken". The same rectangle with "No orders yet — here's today's menu" and a button says "working, and here is what to do". On a budget Android phone on slow 4G, the loading state is not a technicality either: it is what most of your users will look at for two seconds on every single visit, and M08 goes further into what that means on real devices.

### Describing a wireframe in words

You do not draw the hi-fi screen. You describe it, and AI draws it. That makes written wireframe language a core skill, and it has a shape.

Go top to bottom, name each region, state its purpose, state its size relative to its neighbours, and name the single primary action. Like this:

```
SCREEN: Today (first screen after open)
 1 Header, fixed, small: campus name left, cart count right
 2 Strip, one line: "Open till 4:30 pm" or "Closed"
 3 Primary block, largest thing on screen: "Order again -
   Vada pav x2, Rs 30" with one big button
 4 List, scrolls: today's dishes, name + price, sold-out
   dishes greyed and labelled, never hidden
 5 No bottom nav on this screen
PRIMARY ACTION: the Order again button. Everything else is
   quieter than it.
EMPTY: no past orders -> block 3 becomes "First time? Today's
   menu is below."
```

Two things make that brief work. It is ordered top to bottom, so there is no ambiguity about placement. And it names one primary action, which is the instruction AI most needs and least often receives — without it, every button comes back the same size and the screen has no hierarchy at all.

### Where it lives

Wireframes sit between the flow from chapter 1 and the visual design of chapters 5 to 7. They are the thing you put in front of your three test friends in chapter 4 — and lo-fi is deliberately better for that, because people criticise a sketch honestly and praise a polished mock politely. In M02 you learned how HTML structure and CSS layout actually work; a wireframe is the decision that structure then expresses. Your lo-fi frames are also the first real artefact for your M11 checkpoint, where you will be asked to show the design decisions behind the build.

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

**Hi-fi too early is the expensive mistake.** It is expensive twice: redoing it costs time, and looking finished stops people from criticising it. A polished screen gets "nice!" A sketch gets "where's the cart?" You need the second response.

**Lorem ipsum lies about your layout.** Placeholder text is uniformly short and tidy. Real Indian dish names, real ₹ amounts with four digits, and real student names are none of those things. Carrying placeholder text past level 1 guarantees a layout that breaks on contact with your own database.

**Six frames of the same screen beats one perfect frame.** The value of paper is volume. A team that draws one layout has chosen nothing; they have simply accepted the first idea anybody had.

**Do not let AI skip levels on your behalf.** Ask for a wireframe and you will often get a styled component with gradients and an icon set. That is not helpfulness, it is a lost decision. Say "grey boxes only, no colour, no icons, no font choices" and mean it.

**Annotate behaviour, because a still image cannot show it.** "Header sticks on scroll", "this list is capped at ten items", "tapping a sold-out dish does nothing". Unannotated frames get interpreted, and the interpretation will not be yours.

### Verify it

When AI returns a wireframe or a screen, run these checks before you accept it.

1. **Squint at it.** Blur your eyes or shrink the image to thumbnail size. The first thing you can still make out should be your primary action. If it is a decorative image, hierarchy failed.
2. **Count the primary actions.** There must be exactly one per screen. Three equally large buttons means AI could not tell what mattered — which usually means your brief did not say.
3. **Ask for the other three states explicitly.** "Show me this screen with no data, while loading, and after a failed request." If any of the three comes back as the full screen with a spinner slapped on, it was not designed.
4. **Paste in your worst real content.** Your longest dish name, a ₹1,250 price, a 30-character student name. If the layout shifts, overflows or truncates something important, it was built for lorem ipsum.
5. **Check it was asked to stay lo-fi and stayed lo-fi.** Any colour, shadow or icon in a wireframe means the instruction was ignored, and the layout decision you were trying to isolate is now buried.

Red flags: every element the same size, placeholder text still present in a mid-fi frame, no empty state, a screen with no scrollable region when the data is unbounded, and a design that only works at exactly one screen width.

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
