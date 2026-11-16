---
id: m03-09-accessibility-wcag
module: m03-design
slug: accessibility-wcag
title: "Accessibility: WCAG, Alt Text, Keyboard and Screen Readers"
order: 9
duration_min: 25
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

A student in the batch uses a screen reader. They open a classmate's canteen app and hear this:

"Button. Button. Button. Image. Image. Image. Link. Edit text. Button."

That is the entire menu screen. Nine controls, not one of them named. The dish photos have no alt text, so they are announced as "image". The add buttons contain only an icon, so they are announced as "button". The search field has placeholder text instead of a label, so once anything is typed the field has no name at all. There is no way to know which button adds which dish, so the app cannot be used at all — not slowly, not with effort. At all.

The same app is also unusable for a student whose phone's touchscreen is cracked along the bottom third and who navigates with a keyboard, because the AI removed the focus outline in session two for looking cleaner.

Neither student is an edge case. India's 2011 census counted over 26 million people with a disability, and that is an undercount. And every one of these failures took under a minute to fix at the time.

### The idea

**Accessibility is a set of checks you can run yourself in twenty minutes, not a specialist discipline you bolt on at the end.**

WCAG — the Web Content Accessibility Guidelines — is organised under four principles: content must be **perceivable**, **operable**, **understandable** and **robust**. AA is the level everyone targets and most legislation references. You do not need to read the specification. You need five habits and four checks.

### The analogy

The canteen has a step at the entrance. Six inches, and it has been there since 1987. Everybody steps over it without noticing, which is exactly why nobody has ever removed it.

A student on crutches cannot get in. Neither can the vendor with a trolley, nor the person carrying a full tray, nor anybody in the dark during a power cut. A ramp would cost almost nothing at the time the floor was laid and costs a great deal to add later. And the ramp, once built, is used constantly by people who are not disabled at all.

**Where the analogy breaks:** the step is visible. Anybody standing at the door can see it. Your app's barriers are invisible to you — you will never notice a missing label, because you can see the icon. This is why accessibility needs *deliberate tests* rather than attention. You cannot find these problems by looking harder at your own screen.

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

### Alt text depends on the job, not the picture

The single most common mistake is writing alt text that describes the image. The right question is what the image is *doing*.

| Image's job | Alt text |
|---|---|
| Decorative — adds nothing | Empty alt, so it is skipped entirely |
| Informative — shows something | What it shows: "Vada pav on a steel plate" |
| Functional — it is a button or link | What it *does*: "Add vada pav to order" |
| Text in an image — a price board | The text itself, word for word |
| Complex — a chart or map | Short alt plus a longer description nearby |

The functional row is where most people go wrong. If a dish photo is tappable and adds the item, its alt text must be "Add vada pav to order", not "Photo of vada pav". Describing the picture when the picture is a button leaves the user knowing what they are looking at and not what will happen.

Two things to never do: do not start with "image of" — the screen reader already says that — and do not leave a decorative divider with alt text, because every meaningless announcement is noise the user has to wade through.

### Keyboard first, because it finds everything

Here is the highest-return twenty minutes in this module. Put your mouse aside, open your product on a laptop, and use only Tab, Shift+Tab, Enter, Space and the arrow keys.

Four things to watch for. **Can you see where you are, at every single step?** If the focus indicator vanishes, that is `outline: none` and it is a hard failure. **Can you reach everything?** Any control you cannot Tab to does not exist for a keyboard user — this is almost always a `div` with a click handler instead of a real button. **Is the order sensible?** Focus should move roughly the way the eye reads. Jumping from the header to the footer and back means the structure is wrong. **Can you escape?** Open a dialog. Can you close it with Escape? Can you Tab out of it into the page behind, which you should not be able to do?

This test requires no tools, no extension and no screen reader, and it catches most of what a screen reader would also catch. It is the one you should run every single week.

### Labels, headings and real controls

Three structural rules that between them fix most screen-reader experiences.

**Every input needs a real label, not a placeholder.** Placeholder text disappears the moment the user types, taking the field's name with it, and it usually fails contrast as well. Visible labels above the field, always.

**Headings must be real headings, in order.** A screen reader user navigates by jumping between headings — it is their equivalent of skimming. Text that is merely large and bold is not a heading, and skipping from a level-one to a level-three heading breaks the outline. One level-one heading per page.

**A button must be a real button.** A styled `div` that responds to clicks is invisible to keyboards and announced as nothing. AI produces these constantly, and the fix is one word in a brief: "every interactive element must be a real semantic control". M02's HTML chapters are where this vocabulary came from.

For an Indian product, two extras. Set the page's language correctly, and set it per-element when you mix scripts — a screen reader reading Marathi text with English pronunciation rules is unusable. And never make a colour the only carrier of meaning, which is chapter 5's rule appearing again because it is also WCAG 1.4.1.

### Where it lives

Accessibility is a constraint on everything in this module, not a chapter after it. Chapter 5's contrast minimums are WCAG 1.4.3. Chapter 6's focus states are 2.4.7 and its tap targets are 2.5.8. Chapter 7 is where these become token obligations so they cannot drift. Chapter 10's motion rules are 2.3.3. Chapter 4's usability testing gets a sibling in M12, which covers mobile and assistive technology properly. In M06 an automated accessibility check can run in CI on every push, which catches regressions you would otherwise ship.

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

**Automated checkers find roughly a third of real problems.** They are excellent at contrast, missing alt attributes and missing labels, and blind to whether your alt text is *correct*, whether focus order makes sense, and whether the app is actually usable. A clean automated report is the start of the work.

**Overlay widgets do not fix accessibility.** The one-line script that promises compliance is widely rejected by disabled users and has been the subject of lawsuits. Fix the markup.

**ARIA attributes are a last resort, not a decoration.** Adding `role="button"` to a div is worse than using a button, and AI sprinkles ARIA liberally because it looks diligent. Wrong ARIA is more harmful than none.

**Accessibility and your visual style are not actually in conflict.** Contrast and focus rings are chapter 8's constraints, not its enemies. The look that cannot survive 4.5:1 was never a strong look.

**Nobody will report these bugs to you.** A user who cannot use your app leaves. You will see it as a flat line in M07's analytics and never know why, which is precisely why the checks have to be scheduled rather than triggered by complaints.

### Verify it

Four checks, every one of which you run yourself.

1. **The keyboard run.** Tab through every screen. Write down each control you cannot reach, each point where focus disappears, and each place the order jumps. Any one of these is a blocker, not a nice-to-have.
2. **The screen-reader pass.** Turn on TalkBack on an Android phone, or VoiceOver on an iPhone, and listen to one screen with your eyes shut. Write down every announcement that is just "button", "image", "link" or "edit text" with no name. Ten minutes, and it will change how you brief design permanently.
3. **The automated scan, read properly.** Run a browser accessibility audit and read every item rather than the score. Then remember it found about a third of what is there.
4. **The alt-text audit.** List every image and, for each, decide its job — decorative, informative, functional, text, complex — and check the alt text matches that job. Any functional image whose alt describes the picture rather than the action is wrong.

When briefing AI, ask it for a list it cannot fake: "for every interactive element on this screen, give me the element type, its accessible name, and its focus indicator." Then verify three of them yourself with the keyboard. A table is checkable; "this follows WCAG AA" is not.

Red flags: `outline: none` anywhere; placeholder text used as a label; alt text beginning "image of"; a clickable `div`; ARIA roles on elements that already have the right role; a heading level skipped; a contrast claim with no number; an icon-only button with no accessible name; and any claim of compliance that does not name a specific guideline.

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
