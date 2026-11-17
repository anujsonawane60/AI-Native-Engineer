---
id: m03-05-hierarchy-type-colour
module: m03-design
slug: hierarchy-typography-colour
title: "Visual Hierarchy, Typography and Colour"
order: 5
duration_min: 15
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

A team's canteen menu looks fine until you take the phone outside at 2 pm in April and cannot read it: dish names light grey on white at 13 px, thin weight, the price smaller than the description, "Sold out" in red on a dark red tint two students cannot distinguish. None of this is taste — each item is a measurable failure with a number.

### The idea

**Hierarchy is the deliberate ordering of attention, built from four tools: size, weight, colour and space.**

Typography and colour are not decoration added at the end; they are the mechanism by which hierarchy exists. Everything at one size has no hierarchy, however nice the font.

### The analogy

The canteen price board has large dark dish names, prices right-aligned so the eye runs down the column, and timings small at the bottom where nobody reads them — correctly, since they matter least. It was designed for sixty students at four metres, not from a theory.

**Where the analogy breaks:** the board is one size, one distance, one room. Your screen is read at midnight in a hostel and in sun on a bus. Hence a *scale*, not sizes picked one at a time.

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

Pick sizes from a fixed list, never improvised:

| Role | Size | Weight | Use |
|---|---|---|---|
| Display | 32 px | bold | One per screen, at most |
| Heading | 24 px | bold | Section titles |
| Subheading | 18 px | semibold | Card titles, dish names |
| Body | 16 px | regular | Paragraphs, descriptions |
| Small | 14 px | regular | Metadata, timings |
| Caption | 12 px | regular | Legal, hints. Rarely. |

Four rules matter more than the exact numbers:

- **16 px is the floor** for anything a user must read
- **Three levels per screen, not six**
- **45–75 characters per line**, line height 1.5
- **A system font stack** — zero bytes on 4G, correct Devanagari, extra line height for matras

Colour the same way: name roles, then pick a colour per role.

| Role | What it is for | Rule |
|---|---|---|
| Surface | Page and card backgrounds | Mostly neutral |
| Text | Primary and secondary text | Must pass contrast |
| Primary | The one action you most want tapped | Used sparingly |
| Success | Confirmed, available, paid | Never alone |
| Warning | Needs attention, nearly out | Never alone |
| Danger | Destructive or failed | Never alone |
| Border | Separation between things | Low contrast on purpose |

Two WCAG numbers to know: normal text needs **4.5:1** contrast, large text (24 px, or 19 px bold) needs **3:1**. And **colour must never carry meaning alone** — one man in twelve has a colour vision deficiency, so pair it with a word or an icon.

### Where it lives

This chapter is the first half of hi-fi.

- **Chapter 6** — spacing, grids and states, the other half
- **Chapter 7** — these decisions become tokens
- **Chapter 8** — direction that avoids the default purple gradient
- **Chapter 9** — contrast and non-colour cues as hard requirements
- **M02** — CSS variables hold these values

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

**More levels means less hierarchy.** Six text sizes is the same as one; a 2 px difference reads as a mistake.

**Thin fonts look premium and read badly.** 300-weight at 14 px in sunlight is invisible.

**Near-black is not an excuse for grey.** `#111` on `#fff` passes; `#999` is 2.8:1 and fails, and most AI output lands in that gap.

**Dark mode is a second colour system, not an inversion.** Flipped values smear on OLED and brand colours fail on dark surfaces. Give it its own role values or skip it.

**Semantic colour is partly cultural.** Green and red travel well enough, but in anything about money, say the words.

### Verify it

Six checks, each under a minute.

1. **The squint test.** At thumbnail size, the first three things you can make out must be the three you meant.
2. **Count distinct text sizes** — over four means the scale was improvised.
3. **Measure contrast on the worst three pairs**: secondary text, placeholders, text on your primary.
4. **The greyscale test.** What you can no longer tell apart was carried by colour alone.
5. **The sunlight test.** Take the phone outside and read it.
6. **Check the smallest text you must read** — under 16 px, raise it.

Red flags: light grey body text; thin body weights; five or more text sizes; status shown only by colour; placeholder text as a label; a contrast claim with no number.

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
