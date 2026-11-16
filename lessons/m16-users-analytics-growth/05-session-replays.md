---
id: m16-05-session-replays
module: m16-users-analytics-growth
slug: session-replays
title: "Session Replays and Heatmaps: Watching People Struggle"
order: 5
duration_min: 25
is_free: false
mission: false
volatility: medium
last_reviewed: 2026-11-16
---

## Key learnings
- Use a replay to answer why, after a funnel has already told you where
- Spend a small free replay quota deliberately instead of recording everything
- Verify masking by watching your own replay, never by trusting the setting
- Read rage clicks and dead clicks rather than pretty heatmaps
- Watch six to ten sessions and count how many show the same thing

## Lesson

### The pain

A team has done chapter three properly. The leak is at the address step: 29 points, concentrated on budget Android phones. They know where. They have no idea why, and they have argued about it for four days — the field order, the label wording, whether people distrust giving an address.

Then they watch six recordings of real sessions at that step. In five of them the same thing happens. The user taps the pincode field, the Android keyboard opens and covers the Continue button, the user scrolls to find it, the scroll dismisses the keyboard and clears the field, and on the third attempt they leave. Two of the five tap the same dead area six times in a row before giving up.

Thirty seconds of watching answered a question four days of charts could not. Nobody's theory had been close, and the fix — moving one button and keeping the keyboard from covering it — took an afternoon.

### What M07 covered, and what is new here

M07 did not cover this at all. Its chapters eight and nine are entirely about counting: events, funnels, cohorts, significance, consent. Nothing in M07 shows you a single human being.

That is the gap this chapter fills. Every instrument so far aggregates people into a percentage. A session replay is the opposite instrument: one person, one visit, in order, with their taps and scrolls visible. It is the only tool in the module that can tell you *why*, and it is the only one with a quota small enough that you must plan how to spend it.

### The idea

**A number tells you where people stop. A replay shows you one person's hands, which is the only way to find out why.**

Replays are not a dashboard you browse. They are an instrument you point at a question you already have. Teams that open a replay tool without a question watch nine pleasant sessions, learn nothing, and burn a month's quota.

### The analogy

The canteen has a camera above the till. The register tells the manager thirty-eight students left without buying; the camera shows her that they all left while the second till was shut.

**Where the analogy breaks, in two ways that matter.** The camera shows faces, and the manager can walk over and ask the student what happened. A replay shows a cursor and a scroll position; there is nobody to ask, no expression, and no follow-up question — so you must watch enough sessions that the pattern speaks for itself. And the student in the canteen can see the camera on the wall. Your user cannot see the recorder, which is exactly why consent and masking are not optional, and why chapter eleven treats a replay as personal data rather than as a diagnostic.

### Picture it

```
  POINT THE INSTRUMENT, DON'T BROWSE IT

  funnel says        address step, −29 pts,
  WHERE              budget Android, slow 3G
        │
        ▼
  sample rule        record only sessions that
                     reached the address step
        │            (≈ 8% of all sessions)
        ▼
  watch 6-10         one line of notes per
  at 1x speed        session, no skipping
        │
        ▼
  count              5 of 6 showed keyboard
  the pattern        covering Continue  ◀ finding
                     1 of 6 mistyped pincode
```

A replay quota is spent on sessions that reached the step you are investigating, watched in full, with the pattern counted rather than remembered.

### Spending a small quota

Free replay tiers are small — typically a few thousand recordings a month, and sometimes a few hundred. That is the constraint that shapes everything about how you use this instrument.

**Record conditionally, not continuously.** Trigger recording only for sessions that reach the step you are investigating, or for users in a particular cohort. A product with three hundred users and unconditional recording can exhaust a month's quota in four days, and it will exhaust it on the happy path, because most sessions are uneventful.

**Sample when you must.** If conditional recording is not available, record a fixed percentage of sessions. A consistent 10% is far more useful than 100% for three days and nothing for the rest of the month.

**Turn it off between investigations.** Recording is not free for your user either: the recorder adds weight to every page and sends data continuously, which on a metered prepaid connection is your user paying for your diagnostics. Leaving it on permanently is both a quota problem and a courtesy problem.

**Keep the recordings you learned from.** Export or annotate the three sessions that showed you something, because retention on a free tier is short and the evidence you will want in M18 will have expired by then.

### Heatmaps, rage clicks and dead clicks

A **heatmap** aggregates where people clicked, how far they scrolled, or where they lingered. Click maps and scroll maps are genuinely useful for one narrow question: is the important thing below the fold on a small screen? Beyond that they are decorative, because a warm patch tells you people clicked there and nothing about whether it helped them.

The derived signals are worth far more than the picture.

A **rage click** is several rapid clicks on the same spot — a person telling you, as clearly as data can, that they expected something to happen. A **dead click** is a click on something that is not interactive, which usually means your design implied a button where there is none. An **error click** is a click immediately followed by a thrown error. These three are searchable, countable, and they point you straight at sessions worth watching.

So the order is: funnel locates the step, rage and dead clicks locate the element, replays explain the behaviour.

### Privacy, which is not optional here

A replay captures the screen. By default that can include whatever the user typed, which on your product may be a phone number, an address, an OTP or a password. **Masking** replaces the contents of sensitive fields with blocks before the recording leaves the device, and every replay tool has it.

Three rules. Mask every input by default and unmask deliberately, rather than the reverse. Never record password, OTP or payment fields at all, not even masked. And verify masking by recording yourself typing a fake phone number and then watching your own replay — a configuration you have not watched is a configuration you do not have.

Under the DPDP Act a replay is personal data about an identifiable person, so it needs notice and consent like any other processing, it needs a retention period, and it must be deletable when someone asks. Chapter eleven makes that concrete. A replay also catches other people's data: a user typing a friend's phone number into a notes field has handed you a third party's information that nobody consented to, which is a reason to mask free-text fields even when they feel harmless.

### Where it lives

This chapter sits between chapter three and chapter six: the funnel says where, the replay says why, and activation work fixes it. It is the qualitative instrument in a quantitative module, and the module needs exactly one.

M03's chapter four taught usability testing with three friends in a room. A replay is unmoderated usability testing at scale, with one crucial loss: you cannot ask a follow-up question. So the two are complementary — replays find the pattern across many strangers, a moderated session explains the pattern in one person's words. Chapter one's concierge onboarding is the same observation done in person, and it is better when you can get it. M07's chapter five's never-log list is the same principle applied to logs.

**Tools today.** Free replay quotas, masking defaults and whether conditional recording is available on a free tier all change often. Check the current limits before you plan your sampling, and treat anything written here about quota sizes as an order of magnitude rather than a number.

### Words to own

| Term | Plain meaning | Example |
|---|---|---|
| Session replay | A reconstruction of one visit, in order | Taps, scrolls, 90 seconds |
| Quota | How many sessions your tier will record | A few thousand a month |
| Conditional recording | Record only sessions meeting a rule | Reached the address step |
| Sampling | Recording a fixed share of sessions | A steady 10% |
| Masking | Replacing field contents before sending | Phone shown as blocks |
| Rage click | Rapid repeated clicks on one spot | Six taps, nothing happened |
| Dead click | A click on something not interactive | A label that looks tappable |
| Heatmap | Aggregated clicks, scrolls or attention | Scroll depth on a phone |
| Fold | What fits on screen before scrolling | Continue button below it |
| Annotation | A saved note attached to a session | "Keyboard covers Continue" |

### Tradeoffs and gotchas

**One replay is a story.** Watching a single session and acting on it is the commonest misuse of this tool, because a single session is vivid and memorable. Watch six to ten, write one line each, and count. Five of six is a finding; one of six is a lead.

**Do not watch at 4x and do not skip the pauses.** The pauses are the data. A nine-second gap where nothing happens is a person reading, or confused, or on a screen that has not finished loading.

**Replays of your own team are worthless and will dominate your sample.** You use the product more than anyone. Exclude your team's user ids from recording before you start.

**A heatmap is not a cause.** "The button is cold" does not mean the button is bad; it may be that nobody got that far. Heatmaps answer layout questions and should not be used to settle behavioural ones.

**Shared devices make a replay ambiguous.** One recording may contain two different people using one phone. If a session's behaviour switches character halfway through, consider that explanation before inventing a psychological one.

**Watching people struggle is uncomfortable and that is the point.** Teams stop using replays because the sessions are unflattering. The discomfort is the signal that the instrument is working.

### Verify it

AI will set up a replay tool quickly and get the privacy configuration wrong in predictable ways.

1. **Record yourself typing a fake phone number, a fake address and a fake OTP, then watch your own replay and read every frame.** **A replay that captures a password, OTP or payment field** is the red flag here, and a setting you have not watched is not a guarantee.
2. **Check the consent order in the network tab.** If the recorder starts before the user accepts, the banner is decoration and chapter eleven's problem has already happened.
3. **Ask what the sampling or trigger rule is, in one sentence.** "Record everything" means your quota dies before your investigation starts.
4. **Confirm your own team is excluded.** Then look at the session list and check for sessions that are obviously yours.
5. **Ask what the retention period on recordings is**, and whether you can delete one person's sessions on request. "Until the quota rolls over" is not a policy.
6. **Count the pattern before accepting the conclusion.** Ask how many of how many sessions showed the behaviour. An AI summary of replays that says "users appear confused" without a count is a sentence, not evidence.

Red flags: recording on by default across the whole product; masking left to the tool's defaults; a conclusion drawn from one session; a heatmap used to explain a drop-off; a quota exhausted on uneventful sessions; and a replay library in which half the sessions are your own team.

## Walkthrough
1. Write the question first, in one sentence, naming the step and the segment: "why do budget Android users on slow connections abandon the address step?" Without this, do not open the tool.
2. Estimate your monthly session volume and compare it against your tier's recording quota. Write the ratio down; it decides whether you can record conditionally or must sample.
3. Set the trigger or sampling rule so that recordings come only from sessions relevant to your question, and record the rule beside the question.
4. Mask every input field by default, and exclude password, OTP and payment fields from recording entirely rather than masking them.
5. Exclude your own team's user ids and devices from recording.
6. Record a session yourself in which you type a fake phone number, a fake address and a fake OTP. Watch it back frame by frame and confirm nothing readable survived.
7. Open your app in a fresh browser with the network tab recording, and check that nothing from the replay tool fires before consent is given.
8. Wait until you have at least ten relevant sessions, then watch six to ten of them at normal speed, without skipping pauses, writing exactly one line of notes per session.
9. Count how many sessions showed the same behaviour, and write the finding as a fraction — "5 of 6" — rather than as an impression.
10. Search for rage clicks and dead clicks on that screen, and check whether they point at the same element your watching did.
11. Export or annotate the two most informative sessions, note the retention period after which they vanish, and turn recording off until your next question.

## Talk to AI
```prompt
CONTEXT:      I am a student in a team of three in India. I direct AI and
              verify its output; I do not write code. Our product is live:
              [describe it in two sentences]. Our users are students on
              budget Android phones, often on slow or metered connections,
              sometimes sharing a device. We have [number] active users and
              roughly [number] sessions a week. My funnel has already
              located a drop: [name the step, the segment and the drop in
              percentage points]. The input fields on that screen are:
              [list every field, including anything free-text]. We are on a
              free replay tier with a quota of roughly [number] recordings
              a month. India's DPDP Act applies to us.

GOAL:         A replay setup that answers one specific question within my
              quota, records no personal data, and loads nothing before
              consent.

REQUIREMENTS: 1. Restate my question as something a replay can actually
                 answer, and tell me if it cannot.
              2. Given my session volume and my quota, tell me whether to
                 use conditional recording or percentage sampling, and
                 write the exact rule in one sentence. Show the arithmetic
                 for how long my quota will last under your rule.
              3. For each field I listed, say whether it should be masked,
                 excluded from recording entirely, or safe as-is - and name
                 which fields could contain a THIRD party's personal data.
              4. Give me the step-by-step test I can run on my own phone in
                 five minutes to prove masking actually works, including
                 what I should look for frame by frame.
              5. Tell me exactly what must not load before consent, and how
                 I check that in a browser network tab.
              6. Give me a watching protocol: how many sessions, at what
                 speed, what to write per session, and the rule for when a
                 pattern counts as a finding rather than a story.
              7. Tell me what rage clicks, dead clicks and error clicks are,
                 and which of them is most likely to be useful for my
                 specific drop.
              8. Tell me what a heatmap can and cannot answer for a screen
                 like mine, and one wrong conclusion people draw from them.
              9. Tell me the retention period I should set on recordings and
                 how I would delete one person's sessions if they asked.

CONSTRAINTS:  Free tier only. Assume I will verify masking by watching my
              own recording rather than trusting a setting. Do not name a
              specific vendor's current quota as fact - tell me what to go
              and check. Plain English, explain every term on first use.

NON-GOALS:    Do not design fixes for the screen; the next chapter does
              that. Do not design experiments. Do not write code and do not
              modify any files.

ACCEPTANCE:   - My question is one a replay can answer
              - My sampling rule fits my quota, with the arithmetic shown
              - Every field is classified as masked, excluded or safe
              - I can prove masking works by watching my own session
              - I know what must not load before consent
              - I have a counting rule that stops me acting on one session

ASK:          First ask me up to five questions about my screen, my fields,
              my session volume and my quota. Then answer requirements 1
              and 2 only, and stop.
```

## Stuck? Ask AI
```prompt
I watched one session replay and I am certain I have found the problem. Explain to me, with a worked example using plausible numbers, why acting on one session is dangerous and what the minimum number of sessions is before a pattern counts. Then give me a one-line-per-session note format and the rule for turning my notes into a finding.
```

```prompt
Explain what session replay masking does and the three ways it commonly fails in a product built by AI. Then give me a five-minute test I can run on my own phone that would catch each failure, and tell me exactly what I should see in the recording if masking is working correctly.
```

```prompt
My free replay quota was exhausted in four days and I have nothing useful recorded. Explain why unconditional recording wastes a quota on uneventful sessions, then help me write a conditional recording rule for this question: [paste your question]. Show me how long my quota should last under that rule.
```

## Quiz

### Q1 · L3
Your funnel shows a 29-point drop at the address step. What is a session replay the right instrument for?
- (a) Confirming the size of the drop
- (b) Showing why individual people abandoned that step ✅
- (c) Testing whether a redesign performs better
- (d) Measuring retention among users who completed it
> Counting is the funnel's job and comparing is an experiment's job. Replay is the only instrument in the module that shows one person's behaviour in order, which is what a "why" question needs.

### Q2 · L4
You have three hundred users and a free tier that records a few thousand sessions a month, and you enable recording on every session. What happens?
- (a) Nothing; the quota is generous enough
- (b) The quota is consumed mostly by uneventful sessions and runs out before the investigation ✅
- (c) The tool automatically samples down to fit
- (d) Older recordings are compressed rather than lost
> Most sessions are uneventful, so unconditional recording spends the quota on the happy path. Conditional recording aimed at the step under investigation, or a steady low sampling rate, is what makes a small quota usable.

### Q3 · L4
Which setup would be a privacy breach even if the recordings are only ever watched by your own team?
- (a) Recording a session that includes a masked phone-number field
- (b) A replay in which a user's typed OTP is visible on screen ✅
- (c) Recording only 10% of sessions rather than all of them
- (d) Keeping annotated recordings for thirty days with a stated policy
> An OTP must never be recorded at all, masked or otherwise. Who watches it is irrelevant: the data was captured and transmitted, which is the processing the DPDP Act governs.

### Q4 · L4
A heatmap shows almost no clicks on your Continue button. What may you conclude?
- (a) The button's colour and contrast need improving
- (b) Users do not understand what the button does
- (c) Very little on its own — they may never have reached or seen it ✅
- (d) The button is in the wrong place on the page
> Heatmaps answer layout questions like fold depth and nothing about cause. A cold button is equally consistent with nobody arriving, nobody scrolling far enough, or the keyboard covering it.

### Q5 · L5
You watch six relevant replays. Five show the keyboard covering the Continue button; one shows a mistyped pincode. How should this be written up?
- (a) "Users are confused by the address form"
- (b) "5 of 6 sessions: Android keyboard covers Continue at the pincode field" ✅
- (c) "The pincode field has a validation problem"
- (d) "Replays confirm the funnel drop of 29 points"
> A finding is a counted pattern with the mechanism named. The fraction tells a reader how much weight to give it, and the mechanism is specific enough to fix, which "users are confused" never is.

## Go deeper
- [PostHog docs: session replay, sampling and privacy masking](https://posthog.com/docs/session-replay)
- [Nielsen Norman Group: why you only need to test with a handful of users](https://www.nngroup.com/articles/why-you-only-need-to-test-with-5-users/)
