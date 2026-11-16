---
id: m03-03-heuristics
module: m03-design
slug: usability-heuristics
title: "Usability Heuristics, Cognitive Load and the Cost of Choices"
order: 3
duration_min: 25
is_free: false
mission: false
volatility: low
last_reviewed: 2026-11-16
---

## Key learnings
- Review any screen against ten named heuristics instead of saying "it feels off"
- Name the three kinds of mental effort a screen demands and cut the wasteful one
- Explain why adding a choice is never free, even when it is useful
- Turn a vague complaint into a specific, fixable brief
- Catch the heuristic violations AI makes most often, by name

## Lesson

### The pain

A team demos their canteen app. A mentor taps "Place order" and the screen does nothing for four seconds, then jumps to a new page. "Did that work?" they ask. Nobody is sure. They tap again. Two orders.

The same demo has a cancel button next to confirm, both the same size and colour, four millimetres apart. It has a dropdown with forty-two dishes in no particular order. It says "Error 422" when a dish is sold out. And it hides the total until the last screen, so students discover the price after committing.

Each of these has a name. They are not matters of taste, they are not "the design feels a bit off", and they were all findable in ten minutes by one person with a checklist, before any user was involved. The team's actual problem was not skill — it was that they had no vocabulary for what was wrong, so their feedback to the AI was "make it better", and they got a different shade of the same screen.

### The idea

**Most usability problems are a small set of known violations with known names, and naming one turns a feeling into a fixable instruction.**

Jakob Nielsen's ten heuristics have survived since 1994 because they describe how people work, not how software was built that decade. They are a checklist you run yourself, in minutes, with no users present — which makes them the cheapest quality tool you have and the right thing to do before the user testing in chapter 4.

### The analogy

The canteen queue is full of heuristics working or failing. The price board above the counter is *visibility of system status* — you know what things cost without asking. When the server repeats your order back before cooking it, that is *error prevention*. When you can say "actually, no onion" halfway through, that is *user control*. The dishes being called by the names students use, not the supplier's catalogue names, is *match between system and the real world*. And the fact that you do not have to remember yesterday's price because it is on the board is *recognition rather than recall*.

Now imagine a canteen with no board, where you pay first and find out the price afterwards, where the server says "Error 422" when the vada pav is finished, and where you cannot change your mind. Same food. Unusable queue.

**Where the analogy breaks:** the canteen has a human who absorbs every violation. If the board is missing, you just ask. Software has no absorber, so a violation that would be a minor annoyance at a counter becomes an abandoned order. This is why the checklist matters more in software than in a room.

### Picture it

```
            THREE KINDS OF MENTAL LOAD
┌──────────────────────────────────────────────────────────┐
│ INTRINSIC   the task itself          "choose lunch"      │
│             -> cannot remove, only support               │
├──────────────────────────────────────────────────────────┤
│ EXTRANEOUS  effort your design adds  "which of these 6   │
│             -> REMOVE ALL OF THIS     buttons is pay?"   │
├──────────────────────────────────────────────────────────┤
│ GERMANE     effort that teaches      "ah, swipe = next"  │
│             -> keep, but spend once                      │
└──────────────────────────────────────────────────────────┘
```

Three kinds of effort compete for the same limited attention: the task's own difficulty, the difficulty your design adds on top, and the useful one-time effort of learning how the app works. Design work is almost entirely the removal of the middle row.

### The ten heuristics, in canteen terms

| # | Heuristic | Fails like this |
|---|---|---|
| 1 | Visibility of system status | Tap pay, nothing happens for 4 seconds |
| 2 | Match the real world | "Entity unavailable" instead of "sold out" |
| 3 | User control and freedom | No way to remove an item from the cart |
| 4 | Consistency and standards | Back arrow on three screens, not the fourth |
| 5 | Error prevention | Confirm and cancel identical and adjacent |
| 6 | Recognition over recall | Order code you must remember across screens |
| 7 | Flexibility and shortcuts | No reorder for the student who eats the same thing daily |
| 8 | Aesthetic and minimalist design | Six ratings widgets on a menu nobody rates |
| 9 | Recover from errors | "Error 422" with no next step |
| 10 | Help and documentation | Nothing explains what "token number" means |

Number 1 is the one Indian apps get wrong most often and the one that matters most on slow 4G. If a tap produces no feedback within about a tenth of a second, the user does not believe the tap registered. That is how you get double orders. The fix is never faster code — it is an immediate pending state, which you can brief AI for in one sentence.

Number 6 deserves a note too. Recognition over recall means the screen shows you what you need rather than making you hold it in your head. UPI gets this right: the payee name is displayed before you confirm, so you recognise it rather than recalling whether you typed the right VPA.

### Why every choice costs something

Hick's law is the honest version of a thing designers say too casually: the time to decide grows as the number of options grows. It is not linear and it is not catastrophic, but it is real, and it compounds with a second cost most people miss — every option you add is a thing you must design, name, test, support, and explain.

Three practical consequences.

**One primary action per screen.** If everything is a button, nothing is. The screen should make the next step obvious enough that no decision is required at all.

**Group and default.** Forty-two dishes is not a choice problem if they are grouped into five sections with today's six specials at the top. Defaults are the strongest tool you have: pre-select UPI if 85% of your users pay by UPI, because the best choice is the one the user never had to make.

**Progressive disclosure.** Show the three things needed now; keep the other nine behind "More". Zomato's filter panel is enormous and almost nobody opens it, which is exactly the point — it is available without being in the way.

The counterweight: removing choices can remove control, and heuristic 3 is also real. An app with no way to change your order is simple and insulting. The question is never "fewer options" in the abstract, it is "fewer decisions on the path to the outcome, with every escape hatch still reachable".

### Where it lives

A heuristic review is a thirty-minute activity you run on your own wireframes before you ever show them to a user. In chapter 4 you test with three friends, and the heuristic names are what let you describe what you saw precisely. Chapters 5 and 6 fix the visual causes of heuristics 4, 5 and 8. Chapter 9 overlaps heavily: an interface that fails recognition-over-recall for a sighted user usually fails completely for a screen reader. In M07 your analytics will show a drop-off at one step, and the heuristic list is how you generate hypotheses for why. And in M00's terms, this is the VERIFY step of the Director's Loop applied to design — a named checklist, run by you, not a summary you trusted.

### Words to own

| Term | Plain meaning | Example |
|---|---|---|
| Heuristic | A rule of thumb for spotting usability problems | "Visibility of system status" |
| Heuristic review | Checking a screen against the list yourself | 30 minutes, no users |
| Cognitive load | Mental effort a screen demands | Remembering a token number |
| Extraneous load | Effort your design added needlessly | Six identical buttons |
| Hick's law | More options means slower decisions | A 42-item dropdown |
| Progressive disclosure | Show what is needed now, hide the rest | Filters behind "More" |
| Default | The pre-selected option | UPI selected already |
| Affordance | A visual hint about what something does | A button that looks pressable |
| Feedback | The system's response to an action | Button shows "Placing…" |
| Destructive action | An action that cannot be undone easily | "Cancel order" |
| Recognition over recall | Show it rather than make them remember it | Payee name before confirming |

### Tradeoffs and gotchas

**Heuristics find problems; they do not rank them.** A review will produce twenty-two findings and you cannot fix twenty-two things. Rank by how many users hit it and how badly it hurts. A missing pending state on the pay button beats an inconsistent back arrow, every time.

**Minimalism is not emptiness.** Heuristic 8 says remove what is irrelevant, not remove everything. A menu card with nothing but a dish name is minimal and useless; students need the price, and in India they need the veg or non-veg mark. Stripping information is a very common way to look clean and be worse.

**Consistency with the platform beats internal consistency.** If your app invents its own back gesture, being consistently wrong across ten screens does not help. Android users expect the system back button to work. M08 and M12 go deeper on this.

**You go blind to your own screens in about three days.** You know where everything is, so nothing is confusing. This is why you run a written checklist rather than trusting your impression, and why chapter 4 exists.

**AI optimises for looking finished, not for being usable.** It will reliably give you a dense screen with many equally weighted controls, no pending states and no useful error messages, because that is what most of its training data looks like. None of those are hard to fix once you can name them.

### Verify it

Run this on any screen AI gives you. Ten minutes, no users, highest return of anything in this module.

1. **Tap the primary action and count.** Does something change on screen within a blink? If the first visible change is the next page arriving, heuristic 1 failed — ask for a pending state and a disabled button during submission.
2. **Find the destructive action and look at its neighbour.** Is "cancel order" the same size, colour and shape as "confirm", sitting next to it? Ask for them to be visually different and physically separated.
3. **Read every message aloud as if to a friend.** "Error 422", "Invalid input", "Something went wrong" all fail heuristics 2 and 9. Each one must say what happened and what to do next, in words a first-year student uses.
4. **Count the options on the longest decision.** More than about seven ungrouped options means grouping, defaults or disclosure is missing.
5. **Look for anything you have to remember between screens.** An order code, a total, a dish name. Every one of those is a recall failure that should be displayed instead.
6. **Count the equally-weighted buttons.** More than one primary action means the brief did not name one.

Red flags: a screen where every control is the same visual weight; an error message containing a number; no disabled or pending state anywhere in the output; a dropdown with more than twenty unsorted items; and AI telling you the design "follows best practices" without naming a single heuristic. Ask it to name them and check each claim yourself — a confident summary is not a review.

## Walkthrough
1. Open the screen you most recently had AI build for your own product, on a real phone, not a laptop simulator.
2. Write the ten heuristic names down the left side of a page. You will use this page as your scoring sheet.
3. Tap the primary action and watch the first 300 milliseconds. Write down the first visible change and how long it took. If the answer is "nothing", you have your first finding.
4. Find every destructive or irreversible action on the screen. Note its size, colour and distance from the nearest safe button, in millimetres on the real device.
5. Force every error you can: cut the network, submit the form empty, enter a 60-character name, enter ₹0. Copy down the exact message each one produces.
6. Count the options in the largest single decision on the screen and note whether they are grouped, sorted or defaulted in any way.
7. List everything the user must carry in their head from one screen to the next. Each item is a recognition-over-recall finding.
8. Go through your ten rows and mark each pass or fail with one sentence of evidence. Expect fifteen to twenty-five findings on a first review. That is normal.
9. Rank your findings by how many users will hit it multiplied by how much it hurts. Pick the top three and write one precise correction brief for each, naming the heuristic.
10. Send the three briefs one at a time, not together, and re-run only the relevant check after each. Record what the AI changed that you did not ask for.

## Talk to AI
```prompt
CONTEXT:      I am a student founder reviewing one screen of my own
              product before showing it to real users. My users are on
              budget Android phones in India, often on slow 4G, often in
              bright sunlight, and many read English as a second
              language. I do not write code - I direct AI and verify its
              output.

              The screen: [describe it region by region, top to bottom,
              and name the primary action]
              What happens when the primary action is tapped: [describe
              exactly what the user sees, including any delay]
              The exact text of every message and label on it: [paste]

GOAL:         A written heuristic review of this one screen that I can
              turn into three correction briefs.

REQUIREMENTS: 1. Go through Nielsen's ten heuristics by name, in order,
                 one at a time
              2. For each, say PASS or FAIL and quote the specific
                 element or message that justifies it
              3. Do not invent elements - if something I described is
                 ambiguous, ask instead of assuming
              4. Separately list every place the user must remember
                 something between screens
              5. Rank all findings by (how many users hit it) x (how much
                 it hurts), and explain the ranking

CONSTRAINTS:  Judge only what I described. No redesign. No colour or
              font suggestions. Assume a 5-inch screen at 320px wide,
              a thumb-sized tap target, and a one-second network delay
              on every request. Assume no paid service is available.

NON-GOALS:    Do not rewrite the screen. Do not suggest new features. Do
              not propose a design system or component library. Do not
              tell me the design "follows best practices" without naming
              which heuristic and quoting the evidence.

ACCEPTANCE:   - All ten heuristics appear by name with a PASS or FAIL
              - Every FAIL quotes a specific element or exact message
              - Every error message I pasted is judged on whether it says
                what happened AND what to do next
              - The ranking explains both factors, not just severity
              - A final section lists what you could not judge from my
                description and why

ASK:          Before reviewing, tell me which three heuristics you expect
              to be the weakest on a screen like this and why. Then do
              the full review, and tell me whether your prediction was
              right.
```

## Stuck? Ask AI
```prompt
Explain the difference between intrinsic, extraneous and germane cognitive load using an example that is not about software. Then give me five things from a mobile app screen and ask me to sort each one into the right category, without telling me the answers first.
```

```prompt
Here are the exact error and empty-state messages from my app: [paste them all]. For each one, tell me whether it says what happened, whether it says what to do next, and whether a first-year student reading English as a second language would understand it. Then rewrite each in under twelve words. Do not change anything else about my app.
```

```prompt
I have a screen with [N] options the user must choose between. Give me three different ways to reduce the number of decisions without removing any of the options, explain the tradeoff of each, and tell me which one risks taking control away from the user. Do not pick for me.
```

## Quiz

### Q1 · L3
A student taps "Place order" and nothing visibly changes for four seconds, then a new screen appears. Which heuristic failed, and what is the real-world consequence?
- (a) Aesthetic and minimalist design — the screen is too busy
- (b) Visibility of system status — users re-tap and place duplicate orders ✅
- (c) Help and documentation — there is no explanation of ordering
- (d) Flexibility and shortcuts — power users need a faster path
> Without feedback inside roughly a tenth of a second, users do not believe the tap registered. The fix is a pending state and a disabled button, not faster code.

### Q2 · L4
Your dish dropdown has forty-two unsorted items. Which response best respects both Hick's law and user control?
- (a) Cut the menu down to eight dishes
- (b) Group into sections, put today's specials on top, keep all forty-two reachable ✅
- (c) Replace the dropdown with a free-text box
- (d) Leave it — users can scroll
> The goal is fewer decisions on the path to the outcome, not fewer options. Grouping, sorting and defaults reduce decision cost while every option stays available.

### Q3 · L2
Which of these is extraneous cognitive load?
- (a) Deciding what you feel like eating today
- (b) Working out which of six identically styled buttons submits the order ✅
- (c) Learning once that swiping left opens the cart
- (d) Remembering your own name to type it in
> Extraneous load is effort your design added on top of the task. Choosing lunch is intrinsic; learning a gesture once is germane; decoding identical buttons is pure waste.

### Q4 · L4
A heuristic review of your screen produces twenty-two findings. What is the correct next move?
- (a) Send all twenty-two to the AI in one brief and ask it to fix them
- (b) Rank by how many users hit it times how much it hurts, then brief the top three one at a time ✅
- (c) Fix only the visual findings, since those are fastest
- (d) Discard the review — twenty-two findings means the method is too strict
> Heuristics find problems but do not rank them. And an agent asked to fix twenty-two things will touch everything, leaving you unable to tell which change broke what.

### Q5 · L5
AI tells you your screen "follows accessibility and usability best practices". What is the correct response?
- (a) Accept it — the model has seen more interfaces than you have
- (b) Ask it to name each heuristic and quote the evidence, then verify every claim yourself ✅
- (c) Ask for a second opinion from a different model
- (d) Ask it to rewrite the screen to be even more compliant
> A confident summary is not a review. This is M00's verify step: the claim is cheap to make and cheap to check, so check the specific elements rather than trusting the sentence.

## Go deeper
- [Nielsen Norman Group: 10 usability heuristics for user interface design](https://www.nngroup.com/articles/ten-usability-heuristics/)
- [Nielsen Norman Group: Response times — the 3 important limits](https://www.nngroup.com/articles/response-times-3-important-limits/)
