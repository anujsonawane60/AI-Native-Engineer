---
id: m16-06-activation-and-onboarding
module: m16-users-analytics-growth
slug: activation-and-onboarding
title: "Activation and Onboarding: The First Five Minutes"
order: 6
duration_min: 25
is_free: false
mission: true
volatility: low
last_reviewed: 2026-11-16
---

## Key learnings
- Find your activation event from data instead of choosing the one you like
- Put a time box on activation, so the number can move and be compared
- Cut steps and fields before adding any tour, tooltip or explanation
- Measure time to value and activation rate per cohort, every week
- Separate setup work the user does from value the user receives

## Lesson

### The pain

Chapter four handed this team two uncomfortable numbers: retention from signup flattens near 4%, retention from first order at 31%. The product works. Almost nobody reaches it.

So they counted their onboarding: six screens, eleven fields, an email link, a phone OTP and a full address, all before the user sees a dish. Median time to a first order was nine minutes twenty, and 71% never finished. Then a hand-recruited user said "I thought I had already ordered" — she had finished four screens, the fifth was an OTP that never arrived on a weak signal, and she assumed the job was done. She was never shown the product.

### What M07 covered, and what is new here

M07 gave activation one line — the first moment a user got real value — and warned against counting a signup as activation. New here is the work that line hides: finding the event in your data, time-boxing it, and deleting the steps in front of it.

### The idea

**Activation is the earliest action that predicts whether someone will still be here next month, and your onboarding has exactly one job: get a stranger to that action as fast as possible.** Everything else in a first run is at best neutral. Ask of each element: does this stand between a stranger and the action?

Do not pick the event; compute it. For five or six candidate early actions, compare week-four retention between users who did it within 24 hours and users who did not. The candidate with the largest gap, and enough users on both sides to read, wins.

**This is correlation, not causation.** Users who order on day one were already the kind of person who would retain, so forcing a reluctant user through the action does not convert them. It is still the right target: it tells you within a day how a cohort will turn out, and the obstacles in front of it are real obstacles.

### The analogy

A first-year student walks into the canteen on day one. No menu board, no signs, and the cashier asks for a token they have never heard of. They leave and eat biscuits. The canteen's activation event: did this student eat a plate here on their first visit?

**Where the analogy breaks:** the canteen gets a second chance free, because that student still has to eat tomorrow and it is still the nearest option. Your product has no such gravity: a stranger who fails in five minutes returns only if someone asks them to by hand.

### Picture it

```
  TIME TO VALUE, BEFORE AND AFTER

  before   signup ─ verify email ─ OTP ─ address ─ menu ─ order
           0:00     1:40           4:10   7:05      8:30   9:20
                                    ▲                        71% never
                                    OTP never arrives        get here

  after    menu ─ order ─ phone+OTP ─ (address at delivery)
           0:00   0:50    1:30
                                      activation rate 29% → 61%
```

The same product, with verification moved after the first value and the address deferred entirely, takes a stranger from arrival to a placed order in ninety seconds instead of nine minutes.

Deleting steps is the highest-return work in this module:

- **Remove or pre-fill a field.** Needed now, later, or never? Most names and preferences are "later".
- **Defer signup and verification until after first value.** An OTP before anything useful is an activation killer in India, where SMS on a weak signal may never arrive; chapter two's anonymous id joins the before to the after.
- **Write a real empty state before you write a tour,** which only adds steps to the moment you are shortening.

### Where it lives

Chapter four wrote the brief: the gap between the two retention curves is the size of the problem. Chapter three locates which step leaks and chapter five shows why. M03 is the design half, and M17 is where activation becomes money, because acquisition cost is wasted on everyone who never activates.

### Words to own

| Term | Plain meaning | Example |
|---|---|---|
| Activation event | Earliest action that predicts retention | First order placed |
| Activation rate | Share of a cohort reaching it in the window | 29% within 24 hours |
| Time box | The window inside which activation counts | Within 24 hours of signup |
| Time to value | How long from arrival to that action | Median 9m 20s |
| Setup moment | Work the user does for you | Typing an address |
| Aha moment | The point the user sees the benefit | Food arrives in ten minutes |
| Deferred signup | Asking who they are only when needed | After the order is built |
| Progressive disclosure | Revealing complexity only when needed | Address at delivery time |
| Empty state | What a screen shows with no data yet | "Order today's thali" |
| First-run | A user's very first session | Six screens, eleven fields |

### Tradeoffs and gotchas

**Activation needs a time box.** "Placed an order ever" rises forever and compares nothing.

**Do not pick an event because it is easy to move.** "Viewed the menu" is trivially pushed to 95% and predicts nothing.

**Changing the definition invalidates chapter four's curves,** so every earlier cohort needs a dated note.

**Test on a real bad phone, on real bad data.** Your device has autofill, a saved password, a fast connection and working SMS; each one hides an activation killer.

**Shared devices break "sign in with Google".** On a phone with three accounts, the wrong account is a dead end the user cannot diagnose.

### Verify it

Onboarding is where AI is most confidently wrong, because it optimises what it sees.

1. **Check the definition for a time box and for evidence.** An event chosen by taste rather than from a retention comparison is the first red flag here.
2. **Check that activation is not a signup in disguise.** A verified email looks healthy while the product stays unseen.
3. **Count steps and fields before and after.** If the count did not fall, nothing improved, and a tour added where a step should have been removed is the usual culprit.
4. **Check the firing condition.** An event firing when the success screen loads rather than when the order commits counts failures as activations.
5. **Do your own onboarding on a slow old phone**, autofill off and SMS disabled.

Red flags: an activation rate with no window; a definition changed the week the number improved; a before-and-after measured on your own phones.

## Walkthrough
1. Copy the two retention numbers from chapter four — signup-anchored and value-anchored — and write the gap between them as one sentence. That gap is this chapter's budget.
2. List five or six candidate early actions a new user could take in their first day. Include at least one that is setup work and one that is real value.
3. For each candidate, compute week-four retention for users who did it within 24 hours against users who did not, with n stated for both sides.
4. Pick the candidate with the largest gap and enough volume, and write your activation definition with its time box: "did X within Y hours of first arrival".
5. Write one sentence acknowledging that this relationship is correlation, and one sentence on what you will therefore not do with the number.
6. Measure today's activation rate per cohort and your median time to value, in seconds, from your own event data.
7. Count your onboarding honestly: screens, fields, verifications, and taps from arrival to the activation event. Write the four numbers down.
8. Do your own onboarding as a stranger on the oldest, slowest Android phone you can borrow, with autofill off and SMS switched off part way. Time each step and note where you would have stopped.
9. List every step and field, and mark each one now, later or never. Then choose the smallest change that deletes at least two steps — not a change that explains them.
10. Ship that change, then re-measure activation rate and median time to value on the next cohort only, keeping the definition identical.
11. Repeat the stranger test on the bad phone after the change, and record both timings side by side.

## Talk to AI
```prompt
CONTEXT:      I am a student in a team of three in India. I direct AI and
              verify its output; I do not write code. Our product is live:
              [describe it in two sentences]. From my taxonomy, my events
              are: [paste the list]. My retention anchored at signup
              flattens near [number]% and anchored at first value it
              flattens near [number]%. My current onboarding is: [list
              every screen, field and verification in order]. My median
              time from arrival to first value is [number] seconds and
              [number]% of signups never reach it. My users are students on
              budget Android phones, often on weak signal where an SMS OTP
              can take minutes or never arrive, often on metered prepaid
              data, sometimes sharing a phone with several Google accounts
              signed in.

GOAL:         An activation definition derived from my data with a time box
              on it, and the smallest set of deletions to my onboarding
              that would move it.

REQUIREMENTS: 1. From my event list, propose five or six candidate
                 activation events, including one that is setup work and
                 one that is genuine value, and tell me how to compare
                 them: exactly which two groups to split and which
                 retention number to compare.
              2. Explain in plain English why this comparison is
                 correlation rather than causation, and tell me the two
                 specific wrong conclusions I might draw from it.
              3. Help me write the activation definition with a time box,
                 and explain what breaks in my existing retention curves if
                 I later change this definition.
              4. Audit my onboarding step by step. For each screen, field
                 and verification, tell me whether it is needed now, later
                 or never, and what it costs me in seconds and in drop-off
                 risk.
              5. Tell me which steps can be deferred until after first
                 value, specifically including identity and verification,
                 and what my anonymous id has to do for that to work.
              6. Name the three things about Indian student users that
                 should change my onboarding - OTP delivery, data cost,
                 shared devices - and the concrete change each implies.
              7. Write the empty state for my first screen: what a brand
                 new user with no data should see, in words I could ship.
              8. Give me a stranger test I can run on an old slow phone in
                 ten minutes, including what to disable to simulate a
                 failed OTP, and what to record at each step.
              9. Tell me three ways my activation number could improve
                 while the product gets no better.

CONSTRAINTS:  Free tier only. Do not propose a product tour, tooltips or a
              welcome carousel unless you first name the step you are
              deleting. Every suggestion must reduce the count of screens,
              fields, verifications or taps - state the before and after
              counts. Plain English, explain every term on first use.

NON-GOALS:    Do not redesign our visual design. Do not propose A/B tests;
              chapter nine covers that and I may not have the traffic. Do
              not change my event taxonomy without saying what it breaks.
              Do not write code and do not modify any files.

ACCEPTANCE:   - My activation definition has a time box and came from a
                retention comparison, not from taste
              - I can state the correlation caveat in my own words
              - Every onboarding element is marked now, later or never
              - At least two steps are deleted, not explained
              - I have a stranger test I can run on a borrowed old phone
              - I know three ways the number could lie to me

ASK:          First ask me up to five questions about my events, my
              onboarding steps and my users' devices. Then answer
              requirements 1 and 2 only, and stop before auditing my
              onboarding.
```

## Mission

**Objective.** An activation event derived from your own retention data with a time box on it, at least two onboarding steps deleted, and a before-and-after measurement of activation rate and median time to value.

**Steps.**
1. **Frame** — write the gap between your signup-anchored and value-anchored retention curves as one sentence, with both numbers.
2. **Compare** — test five or six candidate activation events by comparing week-four retention for users who did and did not do each within 24 hours, with n on both sides.
3. **Define** — write the activation definition with its time box, plus one sentence on the correlation caveat and what you will therefore not do.
4. **Baseline** — record today's activation rate per cohort, median time to value in seconds, and the counts of screens, fields, verifications and taps.
5. **Stranger test** — run your own onboarding on the oldest slow Android phone you can borrow, autofill off, SMS disabled part way. Time every step.
6. **Cut** — mark every element now, later or never, then ship the smallest change that deletes at least two steps. No tours.
7. **Empty state** — write and ship a first screen that offers a path rather than reporting absence.
8. **Re-measure** — activation rate and median time to value for the next cohort only, with the definition unchanged, plus the stranger test repeated on the same phone.
9. **Verify** — run the six checks from "Verify it", especially the firing condition of your activation event.

**Upload.**
- Your candidate comparison table: each candidate action, retention with and without, n on both sides → screenshot or table
- Your activation definition, with its time box and the correlation caveat in your own words
- Before-and-after counts of screens, fields, verifications and taps
- Activation rate and median time to value, before and after, cohort-by-cohort → screenshot
- Your stranger-test timings on a borrowed old phone, before and after, including where you would have abandoned
- The empty state you shipped → screenshot
- Your AI conversation, including any tour or tooltip you rejected → share link in `ai_chat_url`
- Reflection, 150 words max → the step you were most reluctant to delete, and one way your activation number could still be flattering you

**Self-check.** Tick every box before you submit.
- [ ] My activation event came from a retention comparison, not from preference
- [ ] My activation definition states a time window
- [ ] My activation event is value received, not a completed form or profile
- [ ] I verified the activation event's firing condition by hand
- [ ] I can state the correlation caveat in my own words
- [ ] I counted screens, fields, verifications and taps before and after
- [ ] At least two steps or fields were deleted, not explained
- [ ] I added no tour, tooltip or carousel in place of a deletion
- [ ] I ran the onboarding myself on a borrowed old phone with autofill off
- [ ] I tested the case where the OTP never arrives
- [ ] My first screen offers a path rather than reporting emptiness
- [ ] My re-measurement used the identical definition and a later cohort

**Stretch.** Build the version where signup happens *after* first value: a stranger can reach the activation action without an account, and identity is collected only when it becomes necessary. Measure activation rate on that path and write 100 words on what it cost you elsewhere.

## Stuck? Ask AI
```prompt
Here are five candidate activation events for my product and the retention numbers for users who did and did not do each one in their first day: [paste the table with n on both sides]. Tell me which candidate to choose and why, which ones I should reject as setup rather than value, and the single strongest argument against treating any of them as a lever I can pull.
```

```prompt
Explain the difference between a setup moment and an aha moment using a college canteen on a first-year student's first day. Then give me eight onboarding steps from a typical app and ask me to sort them into "needed now", "needed later" and "never", without telling me your answers first.
```

```prompt
My onboarding has [number] screens and [number] fields before a user sees anything useful. Here is the list: [paste it]. Do not propose a tour, tooltips or a welcome carousel. For each element, tell me what deleting or deferring it would cost me, and then give me the single change that removes the most steps for the least risk.
```

## Quiz

### Q1 · L4
Which activation definition is usable as a metric you can improve and compare across cohorts?
- (a) Completed the signup form
- (b) Has ever placed an order
- (c) Placed an order within 24 hours of first arrival ✅
- (d) Opened the app more than once
> Activation must be value received, not a form submitted, and it needs a time box. "Has ever" rises forever and compares nothing between cohorts, which makes it useless as a target.

### Q2 · L5
Users who place an order in their first 24 hours retain at 38% at week four; those who do not retain at 3%. A teammate concludes that forcing every new user through an order will lift retention to 38%. What is wrong?
- (a) The sample needs to be larger before acting
- (b) The relationship is correlation — early orderers were already the people who would stay ✅
- (c) Week four is too early to measure retention
- (d) The comparison should use bracketed rather than N-day retention
> The action identifies people likely to retain; it does not manufacture them. The legitimate use is to remove the obstacles in front of users who did want to get there, which helps real people without expecting the gap to transfer.

### Q3 · L4
Your product requires an SMS OTP before a user sees anything. Why is this especially damaging for Indian student users?
- (a) OTPs are not permitted under the DPDP Act
- (b) Delivery on a weak hostel signal can take minutes or fail silently, and the user cannot tell which ✅
- (c) OTP screens are difficult to design accessibly
- (d) SMS costs the user money to receive
> An invisible, un-diagnosable wait placed before any value is the strongest possible reason to leave. The fix is to verify at the moment verification matters, such as before payment, rather than at the door.

### Q4 · L4
AI proposes a four-screen welcome tour to explain your onboarding. What should you ask first?
- (a) Whether the tour can be skipped
- (b) Which onboarding step this change deletes ✅
- (c) Whether the copy matches your brand voice
- (d) How the tour will be A/B tested
> A tour adds steps to the exact moment you are trying to shorten. If five people in a row need it, the screen itself is wrong, and explanation is being used as a substitute for deletion.

### Q5 · L3
Your activation rate rose from 29% to 61% in the same week your team changed the activation event's firing condition from "order committed" to "success screen loaded". What should you conclude?
- (a) The onboarding change worked very well
- (b) The two numbers are not comparable, and the new one counts failures as activations ✅
- (c) The new firing condition is more accurate and should be kept
- (d) Nothing, until the next cohort is complete
> A screen that loads is not an order that saved. Changing a definition in the same period as a change to the product makes the before-and-after meaningless, which is why definitions get dated notes and are not tuned.

## Go deeper
- [Reforge: the activation metric and time to value](https://www.reforge.com/blog/activation-metric)
- [Nielsen Norman Group: onboarding and first-use experience research](https://www.nngroup.com/articles/onboarding-tutorials/)
