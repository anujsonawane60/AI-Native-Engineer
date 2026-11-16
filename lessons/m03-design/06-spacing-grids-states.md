---
id: m03-06-spacing-grids-states
module: m03-design
slug: spacing-grids-states
title: "Spacing, Grids and Component States"
order: 6
duration_min: 25
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

A team's canteen app has a cart screen. The gap between a dish name and its price is 14 px. The gap between one dish and the next is 16 px. So each price visually belongs to the dish below it, and a student reads the vada pav as costing ₹40.

Lower down, "Remove" sits 6 px from "Pay now". Both are 28 px tall, and a thumb is wider than that. In testing, two students paid when they meant to remove an item.

And the pay button looks identical whether it is ready, disabled because the cart is empty, submitting, or already failed once — so the student cannot tell which of those four situations they are in.

Three problems, one cause: nobody decided the spacing, nobody measured the targets, and nobody designed the states. All three are invisible on a laptop and painful on a phone.

### The idea

**Space is how you say what belongs together, and a component is not designed until all its states are.**

Proximity is the strongest grouping signal there is — stronger than a border, a background colour or a shadow. And an interactive element is not one thing; it is eight things that share a shape.

### The analogy

Watch how the canteen counter is laid out. Plates, spoons and water glasses sit in one cluster with a clear gap before the chai urn. Nobody drew a line between them. The gap does all the work — you understand instantly that plates and spoons are one group and chai is another.

Now look at the chai urn. It has a tap, and the tap has states. Empty, so nothing comes out. Full and ready. Mid-pour. "Just boiled, wait two minutes", which the person behind the counter tells you because the urn cannot. That missing signal is exactly the gap your software must close by itself.

**Where the analogy breaks:** the counter is arranged once for one room. Your layout has to hold at 320 px, at 412 px and on a tablet, which is why you work from a *scale* of permitted gaps rather than placing things by eye.

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

### One spacing scale

Pick a base unit — 4 px is standard — and allow only multiples of it. A workable scale:

| Token | Value | Typical use |
|---|---|---|
| xs | 4 px | Label to its value, icon to its text |
| sm | 8 px | Inside a button, between tight items |
| md | 16 px | Padding inside a card, between fields |
| lg | 24 px | Between cards, between groups |
| xl | 32 px | Between sections |
| 2xl | 48 px | Above and below a page heading |

Two laws make this scale useful rather than decorative.

**The proximity law.** The gap *inside* a group must be clearly smaller than the gap *between* groups. Not slightly — clearly. 4 px inside and 24 px between is unambiguous. 14 px and 16 px is the bug at the top of this chapter.

**No improvised values.** If a gap is 13 px, somebody nudged it. Every gap should be nameable from the scale. This sounds pedantic until you have six screens built by AI in six sessions and none of them line up.

Padding and margin are different, and AI confuses them constantly. Padding is space inside an element; margin is space outside it, pushing neighbours away. "The button text is touching the edge" is padding; "the buttons are too close together" is margin. Saying which you mean is the difference between one correction and four.

### Grids, briefly

A grid is an agreement about columns and gutters so that things line up down the page. On mobile you mostly need one thing: a consistent left and right page margin — 16 px is a good default — so every element on every screen starts at the same x position. Its absence is why AI-generated screens often feel subtly untidy even when each component is fine. Beyond that, 4 columns on phones, 8 on tablets and 12 on desktop is the common arrangement, with the gutter taken from your spacing scale. M02 covered how flexbox and grid achieve this; here the point is that the *numbers* get decided once and reused.

### The eight states

Every interactive component has states. Here they are, with what each one must communicate.

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

Four of these deserve a specific warning.

**Focus is not optional and must not be removed.** The focus ring is how a keyboard or switch-access user knows where they are. AI strips it frequently because it looks cleaner. Chapter 9 treats this as a hard failure, and so should you.

**Disabled must say why.** A greyed-out "Pay now" with no explanation is a dead end. "Pay now — add an item first" is a disabled state that teaches. A disabled button still needs 3:1 contrast or it is invisible on a sunlit phone.

**Loading is the state that prevents double orders.** Chapter 3 named this as heuristic 1. On tap, the button changes text, shows progress, and becomes untappable. All three, not one.

**Error states need the user's words.** "Payment failed — your money was not deducted. Try again or pay at the counter." That second sentence is the one that matters in an Indian payment flow, because the student's real question is never "what was the error code", it is "did my money go".

### Tap targets

The guideline everyone quotes is 44×44 px, from Apple; Google says 48×48 dp. Either way the number is the size of a fingertip, not the size of the visible icon. A 20 px icon can still be a 48 px target by padding around it.

Two additions for budget Android phones. Minimum 8 px between adjacent targets, and far more if one is destructive — "Remove" next to "Pay" should not be adjacent at all. And the bottom of a tall phone screen is where the thumb rests one-handed, which is why primary actions belong there rather than at the top, where a 6.7-inch phone forces a grip change. M08 and M12 go further.

### Where it lives

Spacing and states are the second half of hi-fi, after chapter 5's type and colour. Chapter 7 freezes these numbers into tokens so they stop being re-decided per screen. Chapter 8 leans on spacing more than anything else — generous, consistent space is one of the few things that reliably makes a generated interface look intentional. Chapter 9 depends on focus and disabled states being real. In M02 these values were CSS variables; at M11's checkpoint, a consistent scale across screens is visible evidence that your product was directed rather than accumulated.

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

**Tight spacing is not information density.** Cramming fits more on screen and makes all of it slower to read. If you genuinely need density — a staff screen showing forty dishes — get it by removing elements, not by shrinking gaps.

**Borders are not substitutes for space.** A screen where every group sits in its own bordered, shadowed card is a screen where space failed and decoration was applied over the top. Remove the borders first; if the grouping still reads, you never needed them.

**Hover states are nearly irrelevant to your users.** Most of your traffic is touch. Get *pressed* and *focus* right before you think about hover at all.

**Disabled buttons can be a trap.** If the user cannot work out why a button is disabled, an enabled button that explains the problem on tap is often kinder.

**Optical alignment beats measured alignment occasionally.** A circular icon measured flush to a square looks slightly inset. A real exception, and the most over-used excuse for improvised values. Twice per product, not twice per screen.

### Verify it

Six checks. Do all of them on a real phone.

1. **Measure two gaps per group** — one inside, one between. If the inside gap is more than about half the between gap, proximity is broken and the grouping is lying.
2. **List every distinct gap value on the screen.** Anything off your scale was improvised. Ask for the scale to be stated and reapplied, rather than fixing gaps one at a time.
3. **Check the left edge.** Lay a straight edge down the left of a screenshot. Every element should start at the page margin unless deliberately indented.
4. **Tap everything with your thumb, standing up, one-handed.** Not a mouse. Anything you mis-hit twice is too small or too close to its neighbour.
5. **Force every state and photograph it.** Network off, then submit. Submit an empty form. Tab through with a keyboard and confirm a visible focus ring on every control. Empty the cart and look at the pay button. A state you cannot produce does not exist.
6. **Measure the destructive action's distance from the nearest safe one.** Adjacent and the same size is a confirmed real-world failure, not a theoretical one.

Red flags: gaps like 13 px or 22 px; every group wrapped in a bordered card; the focus ring deleted; a disabled state identical to default; no pending state on a submit button; icon-only buttons smaller than a thumb; and AI calling spacing "balanced" or "clean" rather than naming the values.

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
