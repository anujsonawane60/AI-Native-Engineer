---
id: m16-02-event-taxonomy
module: m16-users-analytics-growth
slug: event-taxonomy
title: "Instrumenting Behaviour Properly: Event Taxonomy"
order: 2
duration_min: 25
is_free: false
mission: true
volatility: low
last_reviewed: 2026-11-16
---

## Key learnings
- Write a taxonomy where every event has one meaning, one firing condition and typed properties
- Apply a versioning rule, because event history cannot be re-recorded
- Stitch anonymous and signed-in identity, and know what shared phones do to it
- Decide which events must fire from the server rather than the phone
- Name the events you will deliberately not track, and why

## Lesson

### The pain

In week six a team checks the dashboard against the database: 1,900 orders versus 614. Four faults: Android fires `orderPlaced` and web fires `order_placed`, so each chart counts half the traffic; the event fires on the Pay tap, so abandoned payments count; a catch-all `button_clicked` holds 1,400 free-text labels, nine with a phone number; and `price` arrived in week four, so averages across the full history exclude three weeks.

None of it is fixable: behaviour cannot be re-recorded, and six weeks of history are gone.

### What M07 covered, and what is new here

M07 gave you the naming convention: object then action in past tense, fire on success, variation in properties, never personal data. New here is what only matters once real history exists — a maintained file, versioning, identity, client versus server, and not tracking things.

### The idea

**A taxonomy is a contract with your future self: a written list in which every event has exactly one meaning, one firing condition and typed properties — and a meaning is never changed, only versioned.**

Code can be rewritten; history cannot. If `order_placed` meant "tapped" in March and "paid" in April, every chart crossing March is wrong and nothing warns you.

The contract must also say whose behaviour each event is. An **anonymous id** sits on the device from the first visit, before there is an account; a **user id** is your internal id after signup; **stitching** tells your analytics tool the two are one person. Without it your signup funnel breaks at step one, because the browser and the account are different rows.

Then India breaks it: one hostel phone used by three students puts three people under one anonymous id, and one user id if they share a login. Treat per-user numbers as approximate and write that limitation into the file.

### The analogy

The canteen register, with headings at the top: date, dish, plates, student id, payment mode. Every line for the next year fits them, so page 200 reads like page 1. In month three the manager adds a "discount" column, and a human reading the old pages knows blank means none.

**Where the analogy breaks:** a dashboard reads your event history, not a human, and cannot infer that blank means none. A property added in week six is absent for weeks one to five, and the average reports a wrong number with full confidence.

### Picture it

```
  ONE ENTRY IN THE TAXONOMY FILE

  event      order_placed                        v2
  fires      server-side, after payment is
             confirmed and the row is committed
  once per   order
  properties item_count   int     1..20
             total_paise  int     >= 0
             payment_mode enum    upi|cash|card
             is_prepaid   bool
  never      phone, name, delivery_note, upi_id
  wrong if   fired on Pay tap, or fired again on
             the confirmation screen reload
  changed    2026-10-04  v1 fired on tap; v1 data
             before this date is not comparable
```

Every event needs those eight lines: name, firing condition, what one event represents, typed properties with allowed values, the forbidden fields, the known ways it could fire wrongly, and the date of any change.

### Where it lives

An event fired from the phone can be blocked, dropped on a weak signal or lost when the app is killed; a server event cannot. **Anything that must be approximately complete fires from the phone; anything that must be nearly exact fires from the server.** Menu views are client events; payment confirmed and order created are server.

A good SDK queues events while offline, which helps only if it keeps the time the action happened rather than the time it was received. Otherwise Tuesday's basement-lab order lands in Thursday's numbers. Check which yours does.

On the map: M04's schema discipline applied to behaviour; M06's CI, where a naming check stops `orderPlaced` before it merges; chapter three's funnels and chapter four's activation event; chapter eleven's legal position on forbidden fields.

### Words to own

| Term | Plain meaning | Example |
|---|---|---|
| Taxonomy | The written, versioned list of your events | One file in your repo |
| Firing condition | The exact moment an event is recorded | After payment confirms |
| Property | A typed field carried by an event | `payment_mode: "upi"` |
| Cardinality | How many distinct values a property has | 1,400 button labels |
| Anonymous id | Device identifier used before signup | Set on first visit |
| Identity stitching | Joining an anonymous id to a user id | Once, at signup |
| Client event | Fired from the phone or browser | `menu_viewed` |
| Server event | Fired from your backend | `payment_succeeded` |
| Schema drift | The same event meaning different things over time | v1 fired on tap |
| Deprecation | Retiring an event with a dated note | `order_tapped`, ended Oct 4 |

### Tradeoffs and gotchas

**Twelve events, argued down rather than up.** Thirty means thirty firing conditions to verify. Free tiers meter events rather than users, so one chatty client event exhausts the monthly cap and recording stops in your busiest week. And no feature merges without its taxonomy entry, or the file rots — a team rule, not a tool.

**Never change an event's meaning.** Add a property, add an event, or deprecate the old one with a date, and read any chart crossing it as two charts.

**Not tracked, on purpose.** No catch-alls like `button_clicked` with a `label`. No free text typed by users: search terms and notes carry phone numbers. No high-cardinality properties: they cannot be grouped or charted. Nothing you cannot finish "if this number halved we would…" about. No scroll or hover; chapter five's replays answer those better.

### Verify it

AI will instrument your product in one pass and tell you it is done. Four checks, by hand, on a real device.

1. **Do the action once and confirm one event**, with the right properties, then reload the screen and confirm no second one.
2. **Read every property of one real event, field by field.** A phone number or typed note in a property is a privacy breach, usually because a snippet sent the whole form object.
3. **Test stitching.** Browse logged out, sign up, and check whether the earlier activity attached to the new user.
4. **Check where money events fire from and what timestamp they carry.** A browser payment event undercounts; one stamped at receipt rewrites yesterday.

Red flags: names with no firing conditions; events that fire on page load; near-duplicates like `orderPlaced` and `order_placed`; any free-text property; properties added without a dated note.

## Walkthrough
1. List every decision your team expects to make in the next month, in plain sentences — "whether to remove the address step", "whether the WhatsApp reminder brings people back". Aim for six to ten.
2. For each decision, write the single number that would settle it. Discard any decision whose number you cannot name; it is not a decision yet.
3. From those numbers, derive the smallest event list that produces all of them. Challenge each event once: which decision dies without it?
4. Write the taxonomy file with one entry per event, using all eight lines from the diagram — name, fires, once per, properties with types and allowed values, never, wrong if, changed.
5. Mark each event client or server. Anything involving money, order creation or subscription state goes server-side; justify any exception in the file.
6. Write your identity rule: where the anonymous id comes from, exactly when stitching happens, and one sentence on how shared phones limit per-user accuracy.
7. Write the versioning rule at the top of the file: meanings never change, changes get a dated note, deprecations get an end date, and a chart crossing a dated note is two charts.
8. Add the "not tracked" section listing at least four things you deliberately refuse to record, with the reason beside each.
9. Direct AI to instrument exactly this list and nothing else, naming the file as the specification and forbidding additional events.
10. Verify on a real phone: do each action, confirm one event with correct properties, reload to test duplication, read every property of three events in full, and test stitching across signup.
11. Commit the taxonomy with today's date and add a team rule that no feature merges without its entry in this file.

## Talk to AI
```prompt
CONTEXT:      I am a student in a team of three in India. I direct AI and
              verify its output; I do not write code. Our product is live
              with real users: [describe it in two sentences]. Our users
              are students on budget Android phones, often on prepaid
              data, sometimes sharing a device, with patchy connectivity.
              We have [number] active users. We already have an event list
              from earlier work: [paste it, or say "none"]. The decisions
              we expect to make in the next month are: [list 6-10, each
              with the number that would settle it]. India's DPDP Act
              applies to us.

GOAL:         A finished event taxonomy file of at most twelve events that
              I can hand to an AI as a specification, including firing
              conditions, typed properties, an identity rule, a versioning
              rule and a "not tracked" section.

REQUIREMENTS: 1. For each decision I listed, say which single number
                 settles it and which events are required to compute that
                 number. Then tell me which of my decisions are not
                 actually decisions and why.
              2. Propose at most twelve events. For each, give: name in
                 object_action past tense, exact firing condition, what
                 "once per" means, properties with a type and allowed
                 values, fields that must never appear, and one realistic
                 way it could fire wrongly.
              3. Mark each event client-side or server-side and justify
                 every server-side choice in one sentence.
              4. Write my identity rule: anonymous id, user id, exactly
                 when stitching happens, and how I test that stitching
                 worked in under two minutes on my own phone.
              5. Tell me specifically what shared devices and offline
                 queueing do to these numbers, and what sentence I should
                 put in the file so nobody is surprised later.
              6. Write the versioning rule as three or four lines I can
                 paste at the top of the file.
              7. Give me at least five things I should deliberately NOT
                 track, with the reason for each, including anything that
                 would create a high-cardinality property or risk personal
                 data.
              8. Estimate events per active user per session for this list,
                 and tell me whether a free analytics tier would survive
                 ten times my current users.

CONSTRAINTS:  Free tier only. At most twelve events - argue me down rather
              than up. Every property must have a type and, where it is an
              enum, the full list of allowed values. No personal data in
              any property. Explain every term on first use. No code until
              I approve the file.

NON-GOALS:    Do not build funnels, retention or experiments; later
              chapters do that. Do not recommend an analytics vendor. Do
              not instrument anything yet and do not modify any files.

ACCEPTANCE:   - Every event has a firing condition I could test by hand
              - Every property has a type and bounded allowed values
              - Money and order events fire server-side
              - I can test identity stitching myself in two minutes
              - The file has a versioning rule and a "not tracked" section
              - I can name the decision that dies if I delete each event

ASK:          First ask me up to five questions about my product, my
              decisions and what I already record. Then answer
              requirement 1 only, and stop before proposing events.
```

## Mission

**Objective.** A committed taxonomy file of at most twelve events, each with a firing condition and typed properties, instrumented in your live product and verified by hand on a real phone.

**Steps.**
1. **Frame** — list the decisions your team will make this month and the single number that settles each one. Discard the ones without a number.
2. **Derive** — turn those numbers into the smallest event list that produces them. Write beside each event the decision that dies without it.
3. **Brief** — direct AI using the prompt above. Make it argue you down to twelve, and make it supply firing conditions, types and allowed values rather than names alone.
4. **Write the file** — eight lines per event, plus the identity rule, the versioning rule and the "not tracked" section. This file, not the dashboard, is the deliverable.
5. **Direct** — have AI instrument exactly this file and nothing more. Name the file as the specification and forbid extra events.
6. **Verify by hand** — on a real Android phone, perform every action, confirm one event with correct properties, reload to test duplication, and read every property of three events field by field.
7. **Test identity** — browse logged out, sign up, and confirm your earlier activity attached to the new user.
8. **Check the clock** — turn off data, perform one action, turn data back on, and record whether the event arrives with the original time or the time it was received.
9. **Commit** — the taxonomy file with today's date, and a written team rule that no feature merges without its entry.

**Upload.**
- Your taxonomy file in full, including identity rule, versioning rule and "not tracked" section → file or attached PDF
- The decision-to-number-to-event mapping → table or screenshot
- Screenshots of three real events with every property visible, personal data confirmed absent
- Evidence of your duplication test — the same screen reloaded, one event not two
- Evidence of your stitching test — pre-signup activity attached to the user
- The offline test result: original timestamp or receipt timestamp
- Your AI conversation, including where it tried to add events you rejected → share link in `ai_chat_url`
- Reflection, 150 words max → one event you cut and why, and one thing the AI got wrong about firing conditions

**Self-check.** Tick every box before you submit.
- [ ] Twelve events or fewer, and I can name the decision each one serves
- [ ] Every event has an exact firing condition, not just a name
- [ ] Every property has a type, and every enum lists its allowed values
- [ ] No property anywhere contains a phone number, email, name or free user text
- [ ] Money and order events fire from the server, and I verified where from
- [ ] I performed every action myself and saw exactly one event each time
- [ ] I reloaded a confirmation screen and no duplicate event appeared
- [ ] Identity stitching works and I have the screenshot proving it
- [ ] My file has a versioning rule and a dated-change convention
- [ ] My file names at least four things I deliberately do not track
- [ ] My file states what shared devices do to per-user accuracy
- [ ] The file is committed and a team rule ties new features to it

**Stretch.** Estimate your monthly event volume at ten times your current active users, compare it against your analytics free tier's cap, and write 100 words on which event you would drop first and what question you would lose.

## Stuck? Ask AI
```prompt
Here is my event taxonomy: [paste it]. Do not improve it yet. Interrogate it first. For each event, tell me one realistic way it could fire when it should not, and one way it could fail to fire when it should. Then tell me which single event in this list is most likely to be wrong in production and how I would catch it on my own phone in under two minutes.
```

```prompt
Explain the difference between an anonymous id and a user id, and what identity stitching does, using a college canteen where students browse the menu board before anyone knows who they are. Then tell me exactly what breaks in a signup funnel when stitching is missing, and what the funnel would look like in both cases.
```

```prompt
My analytics dashboard says 1,900 orders and my database says 614. List the six most likely causes in order of probability for a product instrumented by AI, and for each give me the specific check that would confirm or rule it out. Do not guess which one it is; give me the diagnostic order.
```

## Quiz

### Q1 · L4
In March your `order_placed` event fired when Pay was tapped. In April you corrected it to fire when payment confirmed. What is the right thing to do about March?
- (a) Reprocess March so the old events match the new meaning
- (b) Record a dated note in the taxonomy and treat any chart crossing it as two charts ✅
- (c) Delete March's events so the history is consistent
- (d) Nothing; the correction fixes the historical data automatically
> Behaviour cannot be re-recorded and a corrected definition does not travel backwards. The only honest option is a dated note that forces anyone reading a chart across the boundary to read two charts.

### Q2 · L4
Your signup funnel shows almost nobody progressing from "viewed menu" to "signed up", although you watched five people do exactly that. What is the most likely cause?
- (a) The menu view event is firing twice
- (b) Anonymous activity is never stitched to the user id, so the pre-signup person is a different row ✅
- (c) The funnel has no time window
- (d) Consent refusals are suppressing the signup event
> Without stitching, the browsing session and the account are two separate identities, so no individual appears to have passed both steps. It is the commonest instrumentation fault once a product has a real funnel.

### Q3 · L3
Which event must fire from your server rather than the user's phone?
- (a) Menu screen opened
- (b) Payment succeeded ✅
- (c) Search box focused
- (d) Onboarding tip dismissed
> Client events are lost to blockers, killed apps and dropped signals, which is tolerable for approximate counts. Anything close to money needs to be nearly exact, so it is fired where you control delivery.

### Q4 · L4
A teammate proposes a `button_clicked` event with a free-text `label` property so that "nothing is lost". Name the strongest objection.
- (a) It will slow the app down measurably
- (b) It creates an unbounded property, burns the event quota and is a route for personal data ✅
- (c) Past-tense naming conventions forbid it
- (d) Analytics tools reject properties with more than 100 values
> Unbounded values cannot be grouped or charted, volume caps are metered in events, and free text typed by users regularly contains phone numbers. Three failures from one convenience.

### Q5 · L5
You add a `total_paise` property in week four and in week eight report the average order value since launch. What is wrong?
- (a) Nothing, as long as the property type is correct
- (b) Weeks one to three have no such property, so the average silently describes only part of the period ✅
- (c) Averages should never be computed from event properties
- (d) The property should have been named `total_rupees`
> A property added later is absent, not zero, and a dashboard cannot infer what the blank means. This is why every change to an event gets a dated note and why cross-boundary charts are read as two.

## Go deeper
- [Segment: a guide to naming and structuring events](https://segment.com/academy/collecting-data/naming-conventions-for-clean-data/)
- [Amplitude: data taxonomy and tracking plans](https://amplitude.com/docs/data/data-planning-playbook)
