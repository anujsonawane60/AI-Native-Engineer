---
id: m03-09-accessibility-wcag
module: m03-design
slug: accessibility-wcag
title: "Accessibility: WCAG, Alt Text, Keyboard and Screen Readers"
order: 9
duration_min: 15
is_free: false
mission: false
volatility: low
last_reviewed: 2026-11-16
---

## Key learnings
- Operate your own product using only a keyboard, and fix what blocks you
- Write alt text that depends on what the image is doing, not on what it shows
- Apply the four WCAG principles as checks rather than as a document
- Label every form field so a screen reader announces something useful
- Catch the five accessibility failures AI produces most often, by name

## Lesson

### The pain

A student who uses a screen reader opens a classmate's canteen app and hears the whole menu screen as "Button. Button. Button. Image. Image. Image. Edit text." Nine controls, none named: no alt text on the photos, icon-only add buttons, a placeholder that vanishes once anything is typed. Nor can it be used by keyboard, because AI removed the focus outline for looking cleaner.

### The idea

**Accessibility is a set of checks you can run yourself in twenty minutes, not a specialist discipline bolted on at the end.**

WCAG organises everything under four principles: content must be **perceivable**, **operable**, **understandable** and **robust**. AA is the level most legislation references, and you need a few habits and four checks, not the specification.

### The analogy

The canteen has a six-inch step at the entrance, there since 1987. Everybody steps over it without noticing, which is why nobody removed it — but a student on crutches cannot get in, nor can the vendor with a trolley. A ramp costs almost nothing while the floor is being laid, and once built everybody uses it.

**Where the analogy breaks:** the step is visible to anybody at the door. Your barriers are invisible to you — you will never notice a missing label, because you can see the icon. Hence *deliberate tests*, not attention.

### Picture it

```
            THE FOUR WCAG PRINCIPLES, AS CHECKS
┌─────────────────────────────────────────────────────────┐
│ PERCEIVABLE    Can it be sensed?                        │
│                -> contrast 4.5:1, alt text, captions    │
├─────────────────────────────────────────────────────────┤
│ OPERABLE       Can it be driven?                        │
│                -> keyboard reaches all, focus visible,  │
│                   48dp targets, no time traps           │
├─────────────────────────────────────────────────────────┤
│ UNDERSTANDABLE Is it predictable and clear?             │
│                -> labels, plain errors, no surprises    │
├─────────────────────────────────────────────────────────┤
│ ROBUST         Does assistive tech understand it?       │
│                -> real buttons, real headings, states   │
└─────────────────────────────────────────────────────────┘
```

Each principle turns into a small number of concrete checks you can run on any screen in minutes, and together those checks catch the overwhelming majority of real problems.

The commonest mistake is alt text that describes the image. The right question is what the image is *doing*.

| Image's job | Alt text |
|---|---|
| Decorative — adds nothing | Empty alt, so it is skipped entirely |
| Informative — shows something | What it shows: "Vada pav on a steel plate" |
| Functional — it is a button or link | What it *does*: "Add vada pav to order" |
| Text in an image — a price board | The text itself, word for word |
| Complex — a chart or map | Short alt plus a longer description nearby |

The functional row is where people go wrong: a tappable dish photo needs "Add vada pav to order", not "Photo of vada pav". Never begin with "image of".

Then run the keyboard, the highest-return twenty minutes in this module. Mouse aside, Tab, Shift+Tab, Enter, Space and arrows only, watching four things: **can you see where you are**, since a vanishing indicator is `outline: none`; **can you reach everything**, because a control you cannot Tab to does not exist and is usually a `div` with a click handler; **is the order the reading order**; **can you escape** a dialog.

Three structural rules fix most of the rest. **Every input needs a real label**, not a placeholder that disappears when the user types. **Headings must be real and in order**, because a screen-reader user skims by jumping between them. **A button must be a real button** — a clickable `div` is announced as nothing. And set the language per element where you mix scripts.

### Where it lives

Accessibility constrains everything here rather than following it.

- **Chapter 5** — contrast is WCAG 1.4.3, colour-alone is 1.4.1
- **Chapter 6** — focus states 2.4.7, tap targets 2.5.8
- **Chapter 7** — these become token obligations, so they cannot drift
- **M06** — an automated check runs in CI on every push
- **M12** — mobile and assistive technology, properly

### Words to own

| Term | Plain meaning | Example |
|---|---|---|
| WCAG | The web accessibility guidelines | AA is the usual target |
| Perceivable | Can be sensed somehow | Alt text, contrast |
| Operable | Can be driven by any input | Keyboard reaches everything |
| Screen reader | Software that speaks the interface | TalkBack on Android |
| Alt text | The text alternative for an image | "Add vada pav to order" |
| Decorative image | Adds no information | Empty alt, so it is skipped |
| Focus order | The sequence Tab moves through | Should follow reading order |
| Semantic HTML | Elements that state their own meaning | A real button, not a div |
| Accessible name | What assistive tech announces | The label, not the placeholder |
| Skip link | A link to jump past the navigation | "Skip to main content" |
| Landmark | A named region of the page | Header, nav, main, footer |

### Tradeoffs and gotchas

**Automated checkers find roughly a third of real problems.** Good at contrast and missing labels, blind to whether your alt text is *correct*.

**Overlay widgets do not fix accessibility.** The script promising compliance is rejected by disabled users and has drawn lawsuits. Fix the markup.

**ARIA is a last resort.** `role="button"` on a div is worse than a button; wrong ARIA is more harmful than none.

**Accessibility and visual style are not in conflict.** A look that cannot survive 4.5:1 was never strong.

**Nobody will report these bugs.** The user leaves, and you see a flat line in M07's analytics. Hence scheduled checks.

### Verify it

Four checks, all run by you.

1. **The keyboard run.** Tab every screen; note each control you cannot reach, each point focus disappears, each jump in order.
2. **The screen-reader pass.** TalkBack or VoiceOver, eyes shut. Note every announcement that is just "button" or "image".
3. **The automated scan, read properly** — every item, not the score, remembering it finds a third.
4. **The alt-text audit.** Decide each image's job and check the alt matches it.

Then ask for a list it cannot fake: "for every interactive element, give the element type, its accessible name, and its focus indicator." Verify three yourself. A table is checkable; "follows WCAG AA" is not.

Red flags: `outline: none` anywhere; a placeholder as a label; alt beginning "image of"; a clickable `div`; ARIA roles on elements that already have the role; a skipped heading level; an icon-only button with no name.

## Walkthrough
1. Open your product on a laptop, put the mouse out of reach, and reach the main action using only Tab and Enter. Note the exact step where you first lose track of where you are.
2. Tab through the entire screen and write down every control you cannot reach at all. Each one is invisible to keyboard and screen-reader users.
3. Open any dialog or menu with the keyboard. Try to close it with Escape, and try to Tab out of it. Record both results.
4. Turn on TalkBack on an Android phone and listen to one screen with your eyes closed. Write down every announcement, verbatim.
5. Count the announcements that were just "button", "image", "link" or "edit text" with no name attached. That count is your headline finding.
6. List every image on the screen and assign each one a job: decorative, informative, functional, text, or complex. Write the correct alt text for each from the table in this chapter.
7. Check every form field for a visible label that stays visible after typing. Replace any placeholder-only labels.
8. Run a browser accessibility audit and read every individual item, ignoring the score. Mark which ones the keyboard run had already found.
9. Brief AI with your findings and ask for a table of every interactive element with its element type, accessible name and focus indicator. Verify three rows yourself with the keyboard.
10. Record your four numbers before and after — unreachable controls, unnamed announcements, missing labels, failing contrast pairs — and put them in your decision log.

## Talk to AI
```prompt
CONTEXT:      I am a student founder directing AI to build my product. I
              do not write code; I verify output. Users are students and
              canteen staff in India on budget Android phones. Some use
              TalkBack. Some navigate by keyboard because of a damaged
              touchscreen. Many read Hindi or Marathi more comfortably
              than English, and the app mixes scripts.

              The screen: every element top to bottom, with its exact
              visible text, whether it is interactive, and whether it is
              an image: [paste]
              My findings from my own keyboard run: [paste]
              My findings from listening with TalkBack: [paste the
              verbatim announcements]

GOAL:         A WCAG 2.2 AA accessibility fix list for this one screen,
              ordered by how badly it blocks a real user.

REQUIREMENTS: 1. A table of EVERY interactive element: element type,
                 accessible name, how it receives focus, what its focus
                 indicator looks like
              2. A table of EVERY image: its job (decorative,
                 informative, functional, text, complex) and the exact
                 alt text it should have
              3. For every finding, the specific WCAG success criterion
                 number and name - not "accessibility best practice"
              4. The computed contrast ratio, as a number, for every
                 text and background pairing on the screen
              5. The correct focus order as a numbered list, and where my
                 current order differs
              6. Language marking for any element that mixes Devanagari
                 and Latin scripts
              7. Findings ranked as BLOCKER, SERIOUS or MINOR, where
                 BLOCKER means a user genuinely cannot complete the task

CONSTRAINTS:  WCAG 2.2 level AA. Semantic HTML elements first - ARIA only
              where no native element exists, and say why each time. Focus
              indicators must never be removed. Visible labels, never
              placeholder-only. Colour must never be the sole carrier of
              meaning. Do not change my layout or my copy except where a
              change is required by a criterion, and flag each of those.

NON-GOALS:    Do not redesign the screen. Do not add features. Do not
              recommend an accessibility overlay widget or any paid
              service. Do not tell me the screen "follows WCAG" without
              naming the criterion and the element.

ACCEPTANCE:   - Every interactive element has a row with a real
                accessible name, or is flagged as missing one
              - Every image's alt text matches its JOB, so a tappable
                photo's alt describes the action rather than the picture
              - Every finding cites a numbered success criterion
              - Every contrast figure is a ratio I can verify in a
                checker
              - A final section titled "What an automated checker would
                miss here", naming the problems only a human can find

ASK:          First, predict the three BLOCKER-level problems you expect
              on a screen like mine and why. Then do the full audit, and
              tell me whether your prediction was right.
```

## Stuck? Ask AI
```prompt
Here is every image on one screen of my app and what each one does when tapped, if anything: [list them]. For each, tell me whether it is decorative, informative, functional, text-in-an-image or complex, then write the alt text it should have. Explain in one line why a functional image's alt text must describe the action and not the picture.
```

```prompt
I tabbed through my screen with the keyboard and here is what happened, step by step: [describe each Tab press and what got focus]. Tell me which of these are real problems and which are fine, name the WCAG criterion for each problem, and rank them by how badly they block a user. Do not redesign the screen.
```

```prompt
Explain what a screen reader actually announces for a button, an image, a heading and a text input, and what it announces when each one is missing its name. Then give me five announcements I might hear and ask me what is wrong in each, without telling me the answers first.
```

## Quiz

### Q1 · L3
A dish photo is tappable and adds the item to the order. What should its alt text be?
- (a) Empty, because the photo is decorative
- (b) "Photo of vada pav on a steel plate"
- (c) "Add vada pav to order" ✅
- (d) "Image of vada pav, tap to add"
> Alt text follows the image's job. A functional image is a control, so its alt must say what will happen — and never begin with "image of", which the screen reader already announces.

### Q2 · L3
Which single test finds the most accessibility blockers with no tools at all?
- (a) Running an automated browser audit
- (b) Tabbing through every screen with the keyboard only ✅
- (c) Checking every colour pair in a contrast checker
- (d) Asking AI whether the screen meets WCAG AA
> The keyboard run exposes unreachable controls, invisible focus and broken focus order in twenty minutes — and catches most of what a screen reader would also surface.

### Q3 · L4
Your automated accessibility scan reports zero issues. What can you conclude?
- (a) The screen meets WCAG AA
- (b) About a third of real problems are covered, and the human checks still have to run ✅
- (c) The scan is broken and should be re-run
- (d) Only contrast remains to be verified
> Automated tools are strong on contrast, missing alt attributes and missing labels. They cannot judge whether alt text is correct, whether focus order makes sense, or whether the task is completable.

### Q4 · L4
Why is a placeholder worse than a visible label?
- (a) Placeholders cannot be translated into other languages
- (b) It disappears when the user types, leaving the field with no name — and it usually fails contrast ✅
- (c) Screen readers always ignore placeholder text
- (d) Placeholders are not supported on Android browsers
> The field loses its accessible name at exactly the moment the user is working in it, which affects everyone who gets interrupted mid-form, not only screen-reader users.

### Q5 · L5
AI reports that your screen "follows WCAG 2.2 AA throughout". What is the correct response?
- (a) Accept it and move on to the next screen
- (b) Ask for a table of every interactive element with its type, accessible name and focus indicator, then verify three rows with the keyboard ✅
- (c) Ask it to add ARIA roles to every element for safety
- (d) Install an accessibility overlay widget as a backstop
> A compliance sentence is unfalsifiable; a table is checkable. Adding ARIA on top of correct elements makes things worse, and overlays are rejected by the users they claim to help.

## Go deeper
- [W3C: WCAG 2.2 quick reference](https://www.w3.org/WAI/WCAG22/quickref/)
- [WebAIM: Keyboard accessibility testing](https://webaim.org/techniques/keyboard/)
