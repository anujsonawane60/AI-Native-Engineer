---
id: m03-05-hierarchy-type-colour
module: m03-design
slug: hierarchy-typography-colour
title: "Visual Hierarchy, Typography and Colour"
order: 5
duration_min: 25
is_free: false
mission: false
volatility: low
last_reviewed: 2026-11-16
---

## Key learnings
- Control what the eye sees first using size, weight, colour and space
- Set a type scale and a line length that stay readable on a 5-inch screen
- Build a colour system from roles rather than from favourite colours
- Check contrast as a number, and know why 4.5:1 exists
- Never let colour be the only thing carrying a meaning

## Lesson

### The pain

A team's canteen menu looks, at first glance, quite good. Then you take the phone outside at 2 pm in Nashik in April and you cannot read it. The dish names are light grey on white, 13 pixels, in a thin weight. The price is smaller than the description. The "Sold out" label is red text on a dark red tint, which two students in the batch cannot distinguish at all. The heading, the dish name, the price and the footer are all within two pixels of the same size, so the screen has no shape — your eye lands nowhere and has to read everything to find anything.

None of this is a taste problem. Every single item is a measurable failure with a number attached, and every one of them is fixable in a single sentence of direction once you know what to measure.

### The idea

**Hierarchy is the deliberate ordering of attention, built from four tools: size, weight, colour and space.**

Typography and colour are not decoration layered on at the end. They are the mechanism by which hierarchy exists at all. A screen where everything is the same size has no hierarchy no matter how nice the font is, and a screen with perfect hierarchy works even in one colour.

### The analogy

Look at the canteen's price board. The dish names are large and dark. The prices are large too, right-aligned so your eye runs down the column. "Today's special" is bigger than everything. The small print about timings is small, at the bottom, and nobody reads it, which is correct — it is the least important thing on the board.

Nobody designed that board with a theory. They designed it under pressure: sixty students, two minutes each, read from four metres away, in daylight that comes in sideways through the door. Those constraints produced the hierarchy.

**Where the analogy breaks:** the board is one fixed size, viewed at one distance, lit by one room. Your interface is read at arm's length on a 5-inch screen, on a 6.7-inch screen, in a hostel at midnight, and in direct sun on a bus. The board can be tuned to its conditions; your design must survive a range of them. That is why you work in a *scale* and a *system* rather than picking sizes one at a time.

### Picture it

```
  NO HIERARCHY                  HIERARCHY
┌──────────────────┐        ┌──────────────────┐
│ Canteen menu     │        │ TODAY            │  small, grey
│ Vada pav         │        │ ███████████      │  large, bold
│ Rs 15            │        │ Vada pav         │
│ Spicy potato...  │        │ Rs 15            │  large, bold
│ Add to order     │        │ spicy potato...  │  small, grey
│ Open till 4:30   │        │ [  ADD  ]        │  solid, wide
└──────────────────┘        │ open till 4:30   │  smallest
   eye lands nowhere        └──────────────────┘
                               eye: dish, price, button
```

On the left every element has the same weight, so the eye has to read all of it to find anything; on the right size, weight and colour put the dish name first, the price second and the action third, and the timing last where it belongs.

### A type scale, not a pile of font sizes

Pick sizes from a short fixed list and never improvise one. A workable mobile-first scale:

| Role | Size | Weight | Use |
|---|---|---|---|
| Display | 32 px | bold | One per screen, at most |
| Heading | 24 px | bold | Section titles |
| Subheading | 18 px | semibold | Card titles, dish names |
| Body | 16 px | regular | Paragraphs, descriptions |
| Small | 14 px | regular | Metadata, timings |
| Caption | 12 px | regular | Legal, hints. Rarely. |

Three rules that matter more than the exact numbers.

**16 px is the floor for anything a user must read.** Not 14, not 13. On a budget Android phone held at arm's length by someone who has been looking at screens since 7 am, 14 px body text is a tax. Mobile Safari and Chrome also zoom the page when an input's font is under 16 px, which breaks your layout as a bonus.

**Use three levels per screen, not six.** Display, subheading, body is a complete hierarchy. Every extra level dilutes the distinction between the others.

**Line length between 45 and 75 characters.** Longer and the eye loses its place returning to the left edge. On a phone this is automatic; on a tablet or desktop a full-width paragraph becomes a 140-character line that nobody finishes. Line height around 1.5 for body text, tighter for headings.

Two context-specific notes for an Indian product. First, Devanagari needs more vertical room than Latin — Hindi and Marathi text at the same font size and line height will look cramped and may clip the matras above the line. Brief for it explicitly if you support those languages. Second, a system font stack is almost always the right call: it is free, it is already on the device, it costs zero bytes on slow 4G, and it renders Devanagari correctly without you shipping a 400 KB font file.

### Colour as roles, not favourites

The mistake is choosing colours. The method is naming roles and then choosing a colour for each role.

| Role | What it is for | Rule |
|---|---|---|
| Surface | Page and card backgrounds | Mostly neutral |
| Text | Primary and secondary text | Must pass contrast |
| Primary | The one action you most want tapped | Used sparingly |
| Success | Confirmed, available, paid | Never alone |
| Warning | Needs attention, nearly out | Never alone |
| Danger | Destructive or failed | Never alone |
| Border | Separation between things | Low contrast on purpose |

Two numbers you should know by heart, both from WCAG, which chapter 9 covers properly. Normal text needs a contrast ratio of at least **4.5:1** against its background. Large text — 24 px, or 19 px bold and above — needs **3:1**. These are not preferences; they are measurable, and a checker will tell you the number in two seconds.

Light grey text on white is the single most common AI design output and it fails 4.5:1 almost every time. In bright Nashik sunlight it fails far worse than the number suggests, because the phone's effective contrast drops with ambient light. Design for sun, not for your hostel room at night.

And the rule that catches the most people: **colour must never be the only carrier of meaning.** Roughly one in twelve men has some form of colour vision deficiency — in a batch of sixty students that is several people. A sold-out dish shown only by turning it red is invisible to them. Pair every colour signal with a second cue: a word ("Sold out"), a shape, an icon, a line through the price, a reduced opacity. The test is simple — turn the screen greyscale and check you can still tell everything apart.

### Where it lives

This chapter is the first half of hi-fi. Chapter 6 adds spacing, grids and states, which is the other half. Chapter 7 turns all these decisions into tokens, which is how they stop being per-screen choices and become a system. Chapter 8 is where you give AI the style direction that stops it defaulting to the same purple gradient everyone else has. Chapter 9 makes contrast and non-colour cues a hard requirement rather than good practice. In M02 you met CSS variables, which is literally where these values will end up. M15 extends this into brand consistency across every surface you publish.

### Words to own

| Term | Plain meaning | Example |
|---|---|---|
| Visual hierarchy | The order in which the eye notices things | Dish, price, button |
| Type scale | A fixed short list of allowed font sizes | 12 / 14 / 16 / 18 / 24 / 32 |
| Weight | How thick the strokes are | Regular 400, bold 700 |
| Line height | Vertical space per line of text | 1.5× the font size |
| Measure | Line length in characters | 45–75 characters |
| Contrast ratio | A number comparing text and background | 4.5:1 minimum for body |
| Colour role | What a colour means, not what it is | "Danger", not "red" |
| Neutral | The greys that carry most of the interface | Surface and border |
| System font | The font already on the device | Costs zero bytes |
| Greyscale test | Viewing the screen with colour removed | Reveals colour-only meaning |

### Tradeoffs and gotchas

**More hierarchy levels means less hierarchy.** Six text sizes on one screen is the same as one. If you cannot say which of two elements is more important, they should be the same size — and if you can, make the difference obvious rather than subtle. A 2 px difference reads as a mistake.

**Thin fonts look premium and read badly.** A 300-weight font at 14 px on an LCD budget phone in sunlight is close to invisible. Keep body text at 400 and above.

**Pure black on pure white is harsh, but near-black is not an excuse.** `#111` on `#fff` is comfortable and passes easily. `#999` on `#fff` is 2.8:1 and fails. The gap between "softened" and "unreadable" is where most AI output lands.

**Dark mode is a second colour system, not an inverted one.** Flipping the values gives you pure white text on pure black, which smears on OLED screens, and your brand colour will often fail contrast on a dark surface. Treat it as its own set of role values or do not ship it.

**Semantic colour is cultural, partly.** Green for success and red for danger carry over reasonably well, but do not lean on colour alone to communicate anything money-related. In a payment flow, say the words.

### Verify it

Six checks on any screen AI gives you. All of them take under a minute and none require a tool you do not have.

1. **Squint or shrink to thumbnail.** Name the first three things you can still make out. If that order is not the order you intended, hierarchy failed. Fix it by changing size and weight, not by adding colour.
2. **Count distinct text sizes.** More than four on one screen means the scale was improvised. Ask for the scale to be stated explicitly and reapplied.
3. **Measure contrast on the three worst pairs** — secondary text, placeholder text, and any text sitting on your primary colour. Use a contrast checker and record the actual number. Never accept "this has good contrast" as an answer; ask for the computed ratio and verify it yourself.
4. **Screenshot it and make it greyscale** on your phone's photo editor. Can you still tell sold out from available, error from success, selected from unselected? Anything that disappears was carried by colour alone.
5. **Take the phone outside.** Not a metaphor. Read the screen in direct sunlight. This single check catches more real problems for an Indian product than any theory in this chapter.
6. **Check the smallest text you must read.** If any required information is under 16 px, ask for it to be raised and something else lowered instead.

Red flags: light grey body text; a thin font weight anywhere in body copy; five or more text sizes; status shown only by colour; placeholder text used as the only label for an input; a contrast claim with no number attached; and a design that was clearly built and judged on a desktop screen.

## Walkthrough
1. Take one existing screen of your product and screenshot it on a real phone, at the real size. Everything below happens on that screenshot, not on a laptop mockup.
2. Shrink the screenshot to thumbnail size or squint hard. Write down the first three things you can still identify, in order.
3. Write down the order you *intended*. Any mismatch between the two lists is your work for this chapter.
4. List every distinct text size you can find on the screen. If there are more than four, you have improvised sizes.
5. Write your type scale as a six-row table with a role and a size for each, and set your body size to 16 px minimum.
6. Name your colour roles before choosing any colour: surface, text primary, text secondary, primary action, success, warning, danger, border. Eight roles, no favourites yet.
7. Pick values and then check contrast for text-primary, text-secondary and text-on-primary. Record the three actual numbers. Anything under 4.5:1 for body text gets changed now.
8. Convert the screenshot to greyscale and find everything whose meaning vanished. For each, decide a second cue — a word, an icon, a line, an opacity change.
9. Brief AI with your scale, your roles and your contrast minimums, and ask it to reapply them to the screen without changing any layout or copy.
10. Re-run the thumbnail test, the greyscale test and the sunlight test on the result. Record which of the three it still fails, and fix that one next.

## Talk to AI
```prompt
CONTEXT:      I am a student founder directing AI to build my product's
              interface. I do not write code. My users are students and
              canteen staff in Nashik, India, on budget Android phones
              with 5 to 6 inch LCD screens, often outdoors in strong
              sunlight, often on slow 4G. Some read Hindi or Marathi
              more comfortably than English.

              The screen I am working on, region by region top to bottom,
              with the exact text on every element: [paste]
              What I want the eye to notice, in order: [1st, 2nd, 3rd]

GOAL:         A visual hierarchy, type scale and colour role system for
              this screen that works in sunlight on a cheap LCD phone.

REQUIREMENTS: 1. A type scale of at most six roles, stated as a table
                 with role, pixel size, weight and where to use it
              2. Body text no smaller than 16px; nothing a user must
                 read smaller than 16px
              3. Colour defined as ROLES first - surface, text primary,
                 text secondary, primary action, success, warning,
                 danger, border - and only then given values
              4. The computed contrast ratio, as a number, for: text
                 primary on surface, text secondary on surface, and text
                 on the primary action colour
              5. For every status in my screen, a second non-colour cue
                 so the meaning survives greyscale
              6. A system font stack that renders Devanagari correctly
                 and costs zero download

CONSTRAINTS:  Body text 4.5:1 minimum, large text 3:1 minimum, no
              exceptions. No font weight below 400 for body text. At
              most three hierarchy levels visible per screen. No
              downloaded font files. Must stay readable in direct
              sunlight. Do not change my layout or my copy.

NON-GOALS:    No new components. No animation. No gradients, glass
              effects or shadows. No dark mode yet. Do not reorder my
              regions. Do not shorten or rewrite my text.

ACCEPTANCE:   - Every contrast figure is given as an actual ratio I can
                verify in a checker, not described as "good" or
                "sufficient"
              - No text size below 16px for anything a user must read
              - Each status has a cue that is not colour, named
                explicitly
              - At thumbnail size, the three things I listed are the
                three most visible, in that order
              - A final list: every value you chose where you were
                guessing, and what you would need from me to decide it
                properly

ASK:          First give me TWO different hierarchy treatments for this
              screen using only size, weight and space - no colour at
              all - with the tradeoff of each. Say which you would pick
              and why. Only after I choose, add the colour system.
```

## Stuck? Ask AI
```prompt
Explain contrast ratio to me as if I have never heard of it. Tell me where the numbers 4.5:1 and 3:1 come from, what they actually protect against, and why a screen that passes them can still be unreadable in bright sunlight. Then give me three text-and-background pairs and ask me to guess which pass, without telling me first.
```

```prompt
Here are all the text sizes, weights and colours on one screen of my app: [list them]. Tell me how many distinct hierarchy levels I have actually created, as opposed to how many I think I have, and which pairs are so close that they read as a mistake rather than a distinction. Do not redesign anything.
```

```prompt
My app uses colour to show these states: [list each state and its colour]. For each one, give me two possible non-colour cues and tell me which is stronger for a user who is outdoors on a cheap phone. Then tell me which of my states would completely disappear if the screen were greyscale.
```

## Quiz

### Q1 · L3
A screen has eight elements, all within two pixels of the same size and all the same weight. What is the problem?
- (a) The font choice is wrong
- (b) There is no hierarchy, so the eye has to read everything to find anything ✅
- (c) The colour palette needs more variety
- (d) The spacing between elements is too tight
> Hierarchy is built from size, weight, colour and space. Without differences, attention is not directed, and a 2 px gap reads as a mistake rather than a distinction.

### Q2 · L2
What is the minimum contrast ratio for normal body text under WCAG AA?
- (a) 2:1
- (b) 3:1
- (c) 4.5:1 ✅
- (d) 7:1
> 4.5:1 is the AA minimum for normal text; 3:1 covers large text at 24 px, or 19 px bold and above. 7:1 is the stricter AAA level.

### Q3 · L4
Your sold-out dishes are shown by turning the card red. Why is this insufficient, and what is the fix?
- (a) Red is culturally inappropriate — use orange with a lighter tint
- (b) Around one in twelve men cannot distinguish it, so add a non-colour cue such as the word "Sold out" ✅
- (c) Red cards use more battery on OLED screens
- (d) Nothing is wrong, as long as the red passes 4.5:1
> Colour must never be the only carrier of meaning. The greyscale test reveals it instantly: if the state disappears without colour, a second cue is required.

### Q4 · L4
AI returns a screen with 14 px body text in a 300-weight font, light grey on white. Which single check exposes the most problems fastest?
- (a) Reading the generated CSS line by line
- (b) Taking the phone outside and trying to read the screen in direct sunlight ✅
- (c) Counting the number of components on the page
- (d) Asking the AI whether it followed accessibility guidelines
> Sunlight collapses marginal contrast, thin weights and small sizes all at once — and for a product used outdoors in India it is the condition that actually matters.

### Q5 · L5
AI reports that your palette "has good contrast throughout". What should you do, and why?
- (a) Accept it; contrast is one of the things models are reliable at
- (b) Ask for the computed ratio of each pair as a number and verify the worst three yourself ✅
- (c) Ask it to make everything darker as a safety margin
- (d) Switch to pure black on pure white to be certain
> Contrast is measurable, so a qualitative claim is a refusal to be checked. Secondary text, placeholder text and text on the primary colour are where it usually fails.

## Go deeper
- [WebAIM: Contrast and colour accessibility](https://webaim.org/articles/contrast/)
- [web.dev: Learn Typography for the web](https://web.dev/learn/design/typography/)
