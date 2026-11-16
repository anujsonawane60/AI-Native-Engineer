---
id: m03-07-design-systems-tokens
module: m03-design
slug: design-systems-and-tokens
title: "Design Systems, Tokens and Component Libraries"
order: 7
duration_min: 30
is_free: false
mission: true
volatility: medium
last_reviewed: 2026-11-16
---

## Key learnings
- Write design decisions once as named tokens instead of restating them every session
- Tell the difference between a token, a component and a pattern
- Keep a short component inventory so AI stops inventing a fourth button
- Name tokens by role so a rebrand is one edit, not forty
- Verify that generated screens actually used your system instead of hardcoding around it

## Lesson

### The pain

A team is four weeks into their canteen app. Six screens exist. They were built in six separate AI sessions, and nobody wrote anything down between them.

There are now four button styles. The primary blue is `#2563EB` on three screens, `#1D4ED8` on two, and `#3B82F6` on the checkout screen because that session happened on a Thursday. Card padding is 16 px, 20 px, 12 px and 24 px. There are two different "sold out" badge designs. The heading size on the orders page is 22 px and on the menu page 24 px, and nobody can remember deciding either.

Then the team changes their brand colour. It is in forty-one places. They ask AI to update it, which it does — along with changing two layouts and the sold-out badge, because it had no idea which blues were the brand and which were incidental.

Nothing here was a mistake at the time. Each session was fine. The problem is that no decision survived its own session.

### The idea

**A design system is your decisions written down in a form AI can be handed at the start of every session.**

It has three layers. Tokens are named values. Components are the reusable pieces built from those values. Patterns are the agreed way of solving a recurring problem. You are not building a Material Design clone; you are writing roughly two pages of named decisions, and then refusing to accept any output that ignores them.

### The analogy

Every canteen has a system, whether or not anybody calls it one. One portion of rice is one specific ladle. Chai comes in one cup size. The ₹20 thali always contains the same four items. The sold-out sign is the same piece of card, reused.

That is why a new worker can start on Monday and serve correctly by Tuesday. The decisions are embodied in the ladle and the cup — nobody has to remember them, and nobody has to re-decide them at 1 pm with sixty people waiting.

**Where the analogy breaks:** a ladle enforces itself physically. You cannot accidentally serve a different portion. Your tokens enforce nothing at all — they are a document, and AI can ignore them silently while telling you it followed them. In the canteen the system is the equipment; in your product the system is only as real as your verification. That is most of this chapter.

### Picture it

```
       TOKENS              COMPONENTS            PATTERNS
┌──────────────────┐   ┌────────────────┐   ┌─────────────────┐
│ colour.primary   │   │ Button         │   │ "Confirm a      │
│   #1D4ED8        │──▶│  uses          │──▶│  destructive    │
│ space.md  16px   │   │  colour.primary│   │  action"        │
│ text.body 16px   │   │  space.sm      │   │   = dialog +    │
│ radius.card 8px  │   │  text.body     │   │     danger btn  │
└──────────────────┘   │  radius.btn    │   │     + cancel    │
   named values        └────────────────┘   └─────────────────┘
   one place           reusable pieces      agreed solutions
```

Tokens are named values in one place, components are the reusable pieces assembled from those tokens, and patterns are the agreed way of handling a recurring situation — each layer built only from the one before it.

### Tokens: naming by role, not by appearance

A token is a name bound to a value. The naming is the entire skill, and there is one rule: **name it for what it does, never for what it looks like.**

| Bad name | Good name | Why |
|---|---|---|
| `blue-600` | `color.action.primary` | Survives a rebrand to green |
| `red` | `color.status.danger` | Says the meaning, not the hue |
| `gap-16` | `space.md` | Survives a scale change |
| `font-22` | `text.heading.size` | Ties to a role in the hierarchy |
| `light-grey` | `color.text.secondary` | Carries its contrast obligation |

`blue-600` is a trap because the day your brand turns green, every name in your system is a lie. `color.action.primary` is still correct.

A minimum viable token set for a student product is about twenty-five values: eight colour roles, six spacing steps, six type sizes with their weights, two border radii, and one or two shadows. That is small enough to fit in one brief and large enough that every screen stops being improvised.

Two tokens worth calling out for an Indian product. A `color.status.*` set that is never used alone — chapter 5's rule, encoded. And a `text.body.size` of 16 px minimum, written into the system so it cannot quietly drift to 14 px in session nine.

### Components: an inventory, not a library

You do not need to build a component library. You need to maintain a list of what already exists, so that AI extends it rather than inventing alongside it.

An inventory row is four short things: the name, what it is for, its states, and the one place its canonical version lives. Twelve components is a complete product:

Button (primary, secondary, danger), Input, Select, Card, Badge, List row, Dialog, Toast, Empty state, Loading skeleton, Error block, Header. That is it. If a thirteenth appears in session nine, something is wrong — either you genuinely needed it, or AI invented a fourth button because nobody told it a button already existed.

The practical mechanism: a short `DESIGN.md` in your repository, under version control alongside `PRD.md` from M01. It holds your tokens and your inventory. Every design session starts by pasting it in, and in M10 this becomes part of the rules file that gets loaded automatically. This is the M00 brief's CONTEXT and CONSTRAINTS sections made permanent — exactly the mechanism that chapter described.

### Patterns: the decisions above components

A pattern is how you always solve a recurring problem, so you stop solving it differently each time.

"How we confirm a destructive action" is a pattern: a dialog, a danger-coloured confirm, a cancel that is visually quieter, and the item's name repeated in the question so the user knows what they are deleting. "How we show that something is unavailable" is a pattern. "How we handle a failed payment" is a pattern, and in India it includes saying explicitly whether money moved — which M04's payment work depends on.

Three or four written patterns prevent more inconsistency than twenty components.

### Where it lives

`DESIGN.md` lives in your repository from this chapter onwards. The tokens become CSS variables, which you met in M02. Chapter 8 is where you decide what the values actually *are* — this chapter is only the structure that holds them. Chapter 9's contrast and focus requirements get encoded here as token obligations, which is how accessibility stops being a final-week panic. M10 turns the file into an agent rules file. M11's checkpoint reviews it directly. M15 extends the same tokens to your brand surfaces outside the product.

### Words to own

| Term | Plain meaning | Example |
|---|---|---|
| Design token | A named value used everywhere | `color.action.primary` |
| Semantic naming | Naming by role, not appearance | `danger`, not `red` |
| Primitive | A raw value a semantic token points at | `#1D4ED8` |
| Component | A reusable interface piece | Button, Card, Badge |
| Component inventory | The written list of what exists | Twelve rows |
| Pattern | The agreed solution to a recurring problem | How we confirm a deletion |
| Variant | A named version of one component | Button: primary / danger |
| Hardcoded value | A raw value used instead of a token | `#1D4ED8` typed into a screen |
| Single source of truth | The one place a decision lives | `DESIGN.md` |
| Drift | The system and the product diverging | 22 px headings appearing |

### Tradeoffs and gotchas

**A system built too early is guesswork, and too late is a migration.** Three or four real screens is the right moment: enough to see what repeats, little enough that extracting it is an afternoon.

**Over-tokenising is real.** Forty-one colour tokens for an eight-screen app is worse than twelve, because nobody can remember which is which and AI will pick wrong. If two tokens always have the same value, they are one token.

**A system you do not paste in does not exist.** This is the failure mode that actually happens. The file is written in week four and never opened again, and by week eight the product has drifted completely. Pasting it must be the first move of every design session.

**AI will claim compliance it did not deliver.** You will be told "I used your design tokens throughout" and find three hardcoded hex values. This is not deceit, it is the model summarising its intent. It is also trivially checkable.

**A third-party component library is a real option with a real cost.** It gives you accessible, tested components immediately and saves genuine time. The cost is that it comes with someone else's visual defaults, which is chapter 8's entire problem — and it is why so many student projects look identical. If you use one, override the tokens deliberately rather than accepting the defaults.

### Verify it

The system is only as real as these checks.

1. **Grep for raw values.** Ask AI: "list every colour, spacing and font-size value in these files that is not a token reference." Any raw hex or pixel value outside the token definitions is drift. This is the highest-value check in the chapter and it takes thirty seconds.
2. **Count components of each type.** Ask for a list of every button variant that exists in the code. If there are five and your inventory says three, two were invented.
3. **Change one token and see what moves.** Set `color.action.primary` to bright pink and load every screen. Everything that should be pink and is not, is hardcoded. Everything that turns pink and should not have, is using the wrong token. Then set it back. This is the single most reliable test that a system exists.
4. **Check the token names, not the values.** If you see `blue`, `red`, `gap16` or `font22`, the naming failed and the next rebrand will be forty edits again.
5. **Verify the obligations.** Does `text.body.size` equal 16 px or more? Does every `color.status.*` pairing still pass 4.5:1? Does the focus ring token exist at all? These are the ones that silently degrade.

Red flags: a hex value inside a screen file; two tokens with the same value and different names; a component that exists in the product but not the inventory; token names describing colours or pixel counts; and any claim of compliance without a list of files you can check.

## Walkthrough
1. Open three to four existing screens of your own product side by side on one page, as screenshots.
2. Write down every distinct colour you can see, every distinct gap and every distinct text size. Expect an uncomfortable number.
3. Group them. Where you have three near-identical blues or four near-identical gaps, pick one survivor each and note what you are discarding.
4. Write your colour tokens as roles first, with no values: surface, text primary, text secondary, action primary, status success, status warning, status danger, border. Then assign one value to each.
5. Write your spacing scale as six named steps and your type scale as six named roles with size and weight. Set `text.body.size` to 16 px minimum.
6. Create `DESIGN.md` in your repository and put the tokens in it, with one line per token naming its role. Commit it with a message that says why.
7. Add your component inventory: name, purpose, states, and where the canonical version lives. Aim for twelve rows or fewer.
8. Write three patterns in plain sentences: how you confirm a destructive action, how you show something is unavailable, and how you report a failed payment.
9. Paste `DESIGN.md` into a fresh AI session and ask it to refactor ONE existing screen to use only tokens, changing no layout and no copy. Demand a list of every value it replaced.
10. Run the pink test: change `color.action.primary` to bright pink, load every screen, photograph what did and did not change, then set it back. Everything that did not change is a hardcoded value to fix.

## Talk to AI
```prompt
CONTEXT:      I am a student founder in a team of three. I direct AI and
              verify its output; I do not write code. My product has 3-4
              screens built in separate sessions, so values are
              inconsistent. Users are on budget Android phones in India,
              often in sunlight, often on slow 4G. Some screens handle
              money.

              Every distinct colour currently in my product: [list]
              Every distinct gap value: [list]
              Every distinct text size and weight: [list]
              My screens and what each one does: [list]
              Statuses my product shows: [e.g. available, sold out,
              payment pending, payment failed]

GOAL:         A design system document I can paste at the start of every
              future session: tokens, a component inventory, and three
              written patterns.

REQUIREMENTS: 1. Colour tokens named by ROLE, never by hue - exactly
                 eight roles, each with one value
              2. A spacing scale of six named steps, all multiples of 4px
              3. A type scale of six named roles with size and weight,
                 where body size is 16px or more
              4. For every pair of my values that is near-identical, tell
                 me which one survives and which I am discarding
              5. A component inventory of 12 rows or fewer: name,
                 purpose, states, canonical location
              6. Three patterns written as plain sentences: confirming a
                 destructive action, showing something unavailable,
                 reporting a failed payment
              7. The computed contrast ratio, as a number, for every
                 text-on-surface and text-on-status pairing

CONSTRAINTS:  Roughly 25 tokens total - if two tokens would share a
              value, make them one. No token may be named for a colour
              or a pixel count. Body text 16px minimum. Status colours
              must never be the only cue. Free tiers only, no paid font
              or icon licences. Do not change my layouts or my copy.

NON-GOALS:    Do not design new screens. Do not add components I do not
              already have. Do not recommend a third-party component
              library. Do not write code - I want the decisions as a
              document.

ACCEPTANCE:   - Every token name says what it DOES, so a rebrand to a
                different colour leaves every name still true
              - No two tokens share a value
              - Every contrast figure is an actual ratio I can verify,
                not a word like "sufficient"
              - Each pattern names which components it uses, from my
                inventory only
              - A final section titled "Decisions I made for you",
                listing every value you chose where my lists were
                ambiguous, with the alternative you rejected

ASK:          First show me TWO different ways to structure the colour
              tokens for this product - one flat set of roles, one with
              primitives underneath semantic names - with the tradeoff of
              each for a three-person student team. Say which you would
              pick and why. Only after I choose, write the full document.
```

## Mission

**Objective.** Produce a committed `DESIGN.md` for your own product containing roughly twenty-five role-named tokens, a component inventory of twelve rows or fewer, and three written patterns — then prove one real screen actually uses it.

**Steps.**
1. **Frame** — screenshot three to four existing screens and list every distinct colour, gap and text size on them. Write down the total count; it is your starting evidence.
2. **Brief** — use the Director's Brief above to generate your token set, inventory and patterns. Choose between the two token structures it offers and record why.
3. **Consolidate** — for every group of near-identical values, decide the survivor yourself. Write down what you discarded.
4. **Commit** — create `DESIGN.md` in your repository with tokens, inventory and patterns. Commit it with a message saying why, alongside your M01 documents.
5. **Direct** — in a fresh session, paste `DESIGN.md` and have one existing screen refactored to use tokens only, with no layout or copy changes. Demand a list of every value replaced.
6. **Verify** — run the pink test: change `color.action.primary` to a bright pink, load every screen, photograph the result, then set it back. Ask for every non-token colour, spacing and font-size value still present in the files.
7. **Iterate** — fix the single worst hardcoded value with a precise brief and a NON-GOAL. Re-run the pink test on that screen only.
8. **Document** — one decision-log entry: your token structure choice, the values you discarded, and what the pink test revealed.

**Upload.**
- Your committed `DESIGN.md` → attach the file or paste it in the caption
- Your repository → `repo_url`
- The AI conversation showing the two token structures and your choice → `ai_chat_url`
- Before and after screenshots of the refactored screen, plus the list of replaced values → screenshots
- Pink-test evidence: every screen photographed with the pink token applied → screenshots
- Your count of distinct values before and after consolidation → two numbers
- Reflection, 150 words max → one token you named badly at first and why the new name is better, plus one thing the pink test caught

**Self-check.** Tick every box before you submit.
- [ ] `DESIGN.md` is committed to the repository, not sitting in a chat window
- [ ] No token name contains a colour word or a pixel count
- [ ] No two tokens share the same value
- [ ] `text.body.size` is 16 px or more
- [ ] Every status has a non-colour cue recorded in the system
- [ ] I recorded an actual contrast ratio for text primary, text secondary and text on each status colour
- [ ] My component inventory is twelve rows or fewer and matches what exists
- [ ] All three patterns are written and name only components from my inventory
- [ ] I ran the pink test on every screen and photographed the result
- [ ] I found at least one hardcoded value the AI claimed it had tokenised
- [ ] My decision log names a value I deliberately discarded during consolidation

**Stretch.** Open one screen of a product your team admires and reverse-engineer its token set from screenshots alone: how many text sizes, how many spacing steps, how many colour roles. Compare the count with yours and write 100 words on what the difference tells you.

## Stuck? Ask AI
```prompt
Here are my token names: [paste them]. For each one, tell me whether the name would still be true if my brand colour changed to green and my spacing scale doubled. Rename the ones that would become lies, and explain the principle behind each rename. Do not add new tokens.
```

```prompt
Here are all the colour values used across my screens: [paste the list]. Group them into the smallest set of distinct roles that still covers everything, tell me which near-duplicates I should collapse, and for each collapse tell me what visual difference I will lose. Do not pick my brand colour for me.
```

```prompt
I pasted my design system into a session and the AI said it used my tokens throughout. Give me a step-by-step verification procedure I can run myself, in order, with the exact questions to ask, to find out whether that is actually true. Assume I cannot read code fluently.
```

## Quiz

### Q1 · L3
Why is `color.action.primary` a better token name than `blue-600`?
- (a) It is shorter and easier to type
- (b) It names the role, so it stays true when the brand colour changes ✅
- (c) It is required by CSS variable syntax
- (d) It renders faster in the browser
> Appearance-based names become lies the moment the value changes. A role-based name survives a rebrand, which turns forty edits into one.

### Q2 · L4
You change `color.action.primary` to bright pink and load every screen. Two screens do not change. What have you learned?
- (a) Those screens are cached and need reloading
- (b) Those screens hardcode the colour instead of referencing the token ✅
- (c) The token was defined with the wrong value
- (d) Those screens do not use the primary action colour at all
> The pink test is the most reliable proof that a system exists. Anything that should change and does not is a raw value that escaped the system.

### Q3 · L2
What is the right scope for a student team's first design system?
- (a) A full component library with documentation for every state
- (b) Roughly twenty-five role-named tokens, a twelve-row inventory and three patterns ✅
- (c) A single file listing every hex value in the product
- (d) Adopting a third-party library and using its defaults
> Small enough to paste into every brief, large enough to stop improvisation. Adopting someone else's defaults is also how products end up looking identical.

### Q4 · L4
AI tells you it "used your design tokens throughout" after refactoring a screen. What should you do?
- (a) Accept it and move on
- (b) Ask it to list every colour, spacing and font-size value in those files that is not a token reference ✅
- (c) Ask it to refactor the screen a second time for safety
- (d) Add more tokens so there is less room for error
> It is summarising intent, not reporting fact. The claim is cheap to make and takes thirty seconds to check — which is the M00 verify step applied to design.

### Q5 · L5
A team writes `DESIGN.md` in week four and their product has drifted badly by week eight. The file is accurate and unchanged. What went wrong?
- (a) The tokens were named incorrectly from the start
- (b) The file was never pasted into subsequent sessions, so it governed nothing ✅
- (c) Four weeks is too early to write a design system
- (d) They needed a third-party component library instead
> Unlike the canteen's ladle, a token document enforces nothing physically. It is only real if it opens every session and if its use is verified afterwards.

## Go deeper
- [Design Tokens Community Group: format and terminology](https://www.w3.org/community/design-tokens/)
- [Nielsen Norman Group: Design systems 101](https://www.nngroup.com/articles/design-systems-101/)
