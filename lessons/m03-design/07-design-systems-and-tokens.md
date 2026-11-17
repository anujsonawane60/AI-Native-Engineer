---
id: m03-07-design-systems-tokens
module: m03-design
slug: design-systems-and-tokens
title: "Design Systems, Tokens and Component Libraries"
order: 7
duration_min: 25
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

A team is four weeks into their canteen app: six screens, six AI sessions, nothing written down between them. The primary blue is `#2563EB` on three screens, `#1D4ED8` on two and `#3B82F6` at checkout. Then they change their brand colour, which lives in forty-one places, and AI updates it along with two layouts and the sold-out badge, because nothing told it which blues were the brand.

### The idea

**A design system is your decisions written down in a form AI can be handed at the start of every session.**

Three layers: tokens are named values, components are built from them, patterns are the agreed way of solving a recurring problem. Not a Material clone — two pages of named decisions, then refusing output that ignores them.

### The analogy

Every canteen has a system whether anybody calls it one: one portion of rice is one specific ladle, and the ₹20 thali holds the same four items. A new worker serves correctly by Tuesday because the decisions live in the ladle, not in anybody's memory.

**Where the analogy breaks:** a ladle enforces itself — you cannot accidentally serve a different portion. Tokens enforce nothing. They are a document AI can ignore silently while telling you it followed them, so the system is only as real as your verification.

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

A token is a name bound to a value, and naming is the skill: **name it for what it does, never what it looks like.**

| Bad name | Good name | Why |
|---|---|---|
| `blue-600` | `color.action.primary` | Survives a rebrand to green |
| `red` | `color.status.danger` | Says the meaning, not the hue |
| `gap-16` | `space.md` | Survives a scale change |
| `font-22` | `text.heading.size` | Ties to a role in the hierarchy |
| `light-grey` | `color.text.secondary` | Carries its contrast obligation |

A minimum viable set is about twenty-five values: eight colour roles, six spacing steps, six type sizes, two radii, a shadow. Encode two obligations with them — `color.status.*` never used alone, `text.body.size` at 16 px minimum.

You do not need a component library, only a written inventory — name, purpose, states, where the canonical version lives — so AI extends it instead of inventing alongside it. Twelve is a complete product: Button (primary, secondary, danger), Input, Select, Card, Badge, List row, Dialog, Toast, Empty state, Loading skeleton, Error block, Header. A thirteenth means AI invented a fourth button.

Above components sit patterns: how you *always* solve a recurring problem. "How we confirm a destructive action" is a dialog, a danger confirm, a quieter cancel, the item's name in the question. Three or four written patterns prevent more inconsistency than twenty components.

### Where it lives

All of it lives in a short `DESIGN.md` beside M01's `PRD.md`, pasted in at the start of every session.

- **M02** — the tokens become CSS variables
- **Chapter 8** — decides what the values *are*; this chapter is the structure
- **Chapter 9** — contrast and focus become token obligations
- **M10** — the file becomes the agent rules file
- **M11** — the checkpoint reviews it directly

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

**Too early is guesswork, too late is a migration.** Three or four real screens is the moment: enough to see what repeats, little enough to extract in an afternoon.

**Over-tokenising is real.** Forty-one colour tokens for eight screens is worse than twelve. Two tokens sharing a value are one token.

**A system you do not paste in does not exist.** Written in week four, never reopened, fully drifted by week eight.

**AI will claim compliance it did not deliver.** "I used your design tokens throughout", plus three hardcoded hex values — the model summarising its intent, and trivially checkable.

**A third-party library is a real option with a real cost.** Tested accessible components immediately, plus someone else's visual defaults — chapter 8's problem. Override the tokens deliberately.

### Verify it

The system is only as real as these five checks.

1. **Hunt raw values.** Ask: "list every colour, spacing and font-size value here that is not a token reference." Anything outside the definitions is drift — thirty seconds, highest value in the chapter.
2. **Count variants.** Five button variants against an inventory of three means two are invented.
3. **The swap test.** Set `color.action.primary` to bright pink and load every screen: what stays unpink is hardcoded, what turns pink wrongly uses the wrong token. The most reliable proof a system exists.
4. **Check the names, not the values.** `blue`, `gap16`, `font22` mean the next rebrand is forty edits again.
5. **Verify the obligations.** Is `text.body.size` 16 px or more? Does every `color.status.*` pairing pass 4.5:1? Does a focus-ring token exist?

Red flags: a hex value inside a screen file; two tokens with one value and two names; a component in the product but not the inventory; token names describing colours or pixels; a compliance claim with no file list.

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
