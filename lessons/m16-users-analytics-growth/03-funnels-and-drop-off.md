---
id: m16-03-funnels-and-drop-off
module: m16-users-analytics-growth
slug: funnels-and-drop-off
title: "Funnels, Drop-Off and Finding the Real Leak"
order: 3
duration_min: 35
is_free: false
mission: true
volatility: low
last_reviewed: 2026-11-16
---

## Key learnings
- Segment a funnel until the average drop stops hiding the real one
- Tell a healthy filter apart from a leak that is losing people who meant to continue
- Overlay errors and latency on a funnel step before blaming the design
- Instrument the step you forgot, which is where leaks usually hide
- Fix one leak, re-measure the identical funnel, and accept the answer

## Lesson

### The pain

A team has a funnel at last. It shows 48% reaching the cart and 23% placing an order, and that 25-point gap is obviously the problem. They spend two weeks redesigning the checkout: fewer fields, bigger buttons, a progress bar, a reassuring message about refunds. It is genuinely better work.

They re-measure. The number is 24%. Two weeks for one point, which is four people.

Then somebody splits the same funnel by device and network. On wifi the drop is small. On a budget Android phone on a 3G connection, 91% of people who reach the payment screen never place the order, because a 600 KB payment script times out before it renders and the screen simply sits blank. Nothing was wrong with the design. The leak was a timeout, and the average had hidden it perfectly, because the average contained both groups and told them about neither.

### What M07 covered, and what is new here

M07's chapter eight taught what a funnel is and the three honest requirements: the steps must be genuinely ordered, it must count people rather than events, and it needs a time window. It also taught how to read drops in percentage points, and why a small late drop is usually more alarming than a big early one. All of that still holds and is not repeated here.

What is new is the hunt. An honest funnel over real users gives you a location, never a cause. This chapter is about turning a location into a cause: segmenting, separating filters from leaks, overlaying errors, instrumenting the step you forgot, and then re-measuring honestly enough that you would believe a disappointing answer.

### The idea

**A funnel tells you where people stop. It never tells you why. The work is to segment until the drop stops being an average, then go and watch one of those people.**

Averages are where leaks hide. A 25-point drop made of two groups — one at 5% and one at 91% — looks exactly like a uniform 25-point drop, and the fix for the first group does nothing for the second. Splitting is not optional polish; it is the method.

### The analogy

At 1:15 p.m. the canteen manager counts 180 students entering and 142 plates sold. Thirty-eight people left without buying. The register gives her that number and nothing else.

So she stands by the counter for twenty minutes. She sees that one till is open during the rush, that the queue for it reaches the door, and that the students who leave are the ones whose next lecture starts at 1:30 — they arrive, see the length, and walk out. The number told her where; standing there told her why, and the answer was staffing rather than food.

**Where the analogy breaks:** she can see the whole queue at once, and the people leaving are still in the room. Your funnel is a summary of people who already left, whom you cannot see and cannot ask. Segmentation is your substitute for standing at the counter — it is how you work out which kind of person left, so you can go and find a live one who looks like them.

### Picture it

```
  SAME FUNNEL, SPLIT BY CONNECTION          n = 412

                  all users   wifi/4G      slow 3G
                              n = 268      n = 144
  viewed menu       100%        100%         100%
  added to cart      48%         51%          43%
  reached payment    41%         46%          32%
  order placed       23%         37%           3%  ◀ leak
                     ───         ───          ───
  drop at payment    −18         −9           −29
```

The combined column shows an 18-point drop that looks like a design problem; split by connection, the same data shows a 9-point drop on fast connections and a 29-point collapse on slow ones, which is a loading problem.

### The segments that actually pay in India

Split by one dimension at a time and stop when a split stops surprising you.

- **Connection quality.** The single most productive split for an Indian student product. Slow 3G and metered prepaid data turn a heavy screen into a dead screen.
- **Device class and Android version.** An old mid-range phone with 2 GB of RAM will drop a tab under memory pressure. One WebView version can break one input field.
- **Acquisition channel.** Chapter one's log becomes a funnel dimension. Users recruited in a classroom convert differently from users who tapped a WhatsApp link at midnight, and mixing them hides both.
- **New versus returning.** A first-timer and someone on their fifth visit meet different obstacles. Blending them averages away onboarding problems entirely.
- **Logged in versus anonymous.** If stitching from chapter two is imperfect, this split will show it.
- **Time of day and day of week.** A canteen product has a 1 p.m. spike; a drop that exists only in the spike is a capacity problem, which is M05's territory.
- **App or build version.** A leak that begins on one date and only on one version is a release, not a behaviour.

### Leak or filter

Not every drop is a problem, and treating filters as leaks is how teams waste the two weeks in the opening story.

A **filter** removes people who were never going to continue. Students browsing a menu out of curiosity, people who opened a shared link to see what their friend was talking about. A large early drop is usually a filter, and narrowing it mostly recruits people who will leave later anyway.

A **leak** loses people who had decided to continue. The evidence for a leak is in what happened just before the exit, and your events can show it: unusually long time on the step, a retried action, a back navigation, a tap that produced nothing, an error event, a form resubmitted twice. Someone who spent ninety seconds on a payment screen and then left did not change their mind; something stopped them.

So before redesigning anything, ask of the drop: did these people show effort? Effort followed by exit is a leak. No effort followed by exit is a filter.

### Overlay the boring data first

Your funnel step is also a page and an API call, and M07's chapters four to six gave you the instruments for those. Before you accept any behavioural explanation, check three numbers for the step where people vanish: its error rate, its p95 load time, and its failure rate on the slowest tenth of connections.

A step with a 4% server error rate does not have a copywriting problem. A step whose p95 is eleven seconds does not have a trust problem. Teams skip this because behavioural explanations are more interesting, and they pay for it in redesigns that move nothing. Check the plumbing, then the psychology.

### Where it lives

Chapter two's taxonomy is what makes this chapter possible, and this chapter is what exposes the taxonomy's gaps — the leak is very often between two steps you never instrumented, which costs you a week of waiting to add one event. Chapter five's replays answer the "why" for a leak you have already located, and chapter six turns the earliest part of this funnel into activation work.

Sideways, M03's usability testing is the qualitative half of this chapter: the funnel finds the step, three friends in a room tell you what the step feels like. M05 explains the slow-connection failures you will find. Chapter nine is where a fix becomes an experiment once you have the traffic, and M18 is where someone will ask you which drop you chose to fix and why.

### Words to own

| Term | Plain meaning | Example |
|---|---|---|
| Step conversion | Share of a step's entrants who reach the next | 41% to 23% |
| Segment | A slice of users sharing an attribute | Slow-3G users |
| Filter | A drop that removes people who never intended to continue | Menu browsers |
| Leak | A drop losing people who did intend to continue | Payment timeouts |
| Effort signal | Evidence someone tried before leaving | Retry, 90s on step |
| Overlay | Reading error or latency data against a funnel step | 4% 5xx at payment |
| Missing step | A real step you never instrumented | Address confirmation |
| Re-measure | Running the identical funnel definition after a fix | Same window, same steps |
| Base size | How many people a percentage is computed from | n = 144 |
| Dimension | The attribute you split a funnel by | Connection, channel |

### Tradeoffs and gotchas

**Segmenting shrinks your base size fast.** Split 412 users three ways and you have segments of forty, where one person is two and a half percentage points. Split by one dimension at a time, state the base size beside every percentage, and stop splitting when a segment drops below about fifty people. Below that you have a lead to investigate, not a finding to act on.

**You will find the leak in the step you did not instrument.** This is normal and it costs a week. Add the one event, wait, and resist the urge to guess in the meantime.

**Fix one thing.** Change three things at once and the number moves, and you have learned nothing about which change did it — so you cannot repeat it. One change, re-measure, record.

**Re-measure the identical definition.** Same steps, same window, same people-not-events rule, same segment. A fix that "worked" against a quietly different funnel is the most common false victory in this chapter, and it is usually accidental rather than dishonest.

**Seasonality will fool you.** Exam week, a long weekend, a festival, a campus event. If your before-and-after windows sit either side of one of those, the thing you measured is the calendar.

**Not every leak is worth plugging.** A 29-point leak on a segment that is 4% of your users is worth less than a 6-point leak on half of them. Multiply the gap by the number of people behind it before you choose.

### Verify it

AI will produce a funnel and a confident diagnosis in one reply. The diagnosis is the part to distrust.

1. **Ask whether the steps are genuinely sequential for one individual.** If somebody can reach step four without step three, it is a bar chart wearing a funnel's name.
2. **Ask whether each step counts people or events, and whether a window is stated.** One enthusiastic tester adding six items can inflate a step on their own.
3. **Demand the base size beside every percentage.** A 29-point drop on eleven people is three people. A percentage with no n is decoration.
4. **Check the first step's firing condition.** A funnel whose top step fires on page load counts bounced loads as participants and understates every rate below it.
5. **Ask for the error rate and p95 of the leaking step before accepting any behavioural cause.** If AI explains your drop with user psychology and has not looked at the plumbing, send it back.
6. **Check that any claimed fix was re-measured against the same definition and a comparable period.** Ask explicitly what the window was, both times.

Red flags: a diagnosis offered before any segmentation; percentages without base sizes; a funnel whose steps were reordered to look smoother; a drop attributed to "users not understanding the value" with no evidence of effort; a fix declared successful after two days; and three changes shipped together.

## Walkthrough
1. Write your funnel definition in one place: the ordered steps, the time window, and the statement that it counts distinct people. Four to six steps, ending at the action your product exists for.
2. Build it for the last four weeks and record the count and percentage at every step, with the base size at the top.
3. Mark the two biggest drops in percentage points, then write one sentence for each arguing whether it is a filter or a leak, using effort signals rather than intuition.
4. Split the funnel by connection quality. Record both columns side by side and note any drop that differs by more than ten points between them.
5. Split it again by device class, then by acquisition channel from chapter one's log, then by new versus returning. One dimension at a time, base size written every time.
6. Stop at the split that surprised you most. Name the segment, its size, and its drop in points.
7. For that segment's leaking step, pull the error rate, the p95 load time and the failure rate on slow connections. Write all three down before forming any opinion.
8. Check whether the step where people vanish is actually one step or two. If there is an uninstrumented moment inside it, add exactly one event and note that you must wait a week.
9. Find one live user who matches the segment and watch them attempt that step on their own phone, saying nothing. Write down what you saw.
10. Choose one change. Write the predicted effect as a number and a date before shipping it — "payment completion on slow connections rises from 3% to at least 20% within one week".
11. Ship it, wait the stated period, re-run the identical funnel on the identical segment, and record the result including the case where nothing moved.

## Talk to AI
```prompt
CONTEXT:      I am a student in a team of three in India. I direct AI and
              verify its output; I do not write code. Our product is live:
              [describe it in two sentences]. Our event taxonomy is:
              [paste it]. Our users are students on budget Android phones,
              many on slow or metered connections, some sharing a device.
              Our funnel definition is: [paste ordered steps, the time
              window, and confirm it counts distinct people]. Here are the
              counts at each step for the last four weeks: [paste them,
              including the base size]. The dimensions I can split by are:
              [list them, e.g. connection, device, channel, new/returning,
              app version, hour of day].

GOAL:         To locate the real leak in this funnel rather than the
              biggest drop, and to choose exactly one change with a
              predicted number and a date attached.

REQUIREMENTS: 1. Interrogate my funnel before interpreting it. Are the
                 steps genuinely sequential for one person? Does it count
                 people? Is the window stated? Is the first step firing on
                 a deliberate action? Tell me what to fix first if any of
                 these fail.
              2. For each of my two biggest drops, tell me whether the
                 evidence I have supports calling it a filter or a leak,
                 and name the specific effort signal I should look for in
                 my event data to settle it.
              3. Rank the dimensions I listed by how likely each is to
                 reveal a hidden leak in a product like mine, and say what
                 pattern in each split would count as a finding.
              4. Tell me the minimum segment size below which I should
                 treat a result as a lead rather than a finding, and why.
              5. Before any behavioural explanation, list the operational
                 numbers I must check for the leaking step and what value
                 of each would make the design irrelevant.
              6. Tell me whether my funnel has a missing step - a real
                 moment between two of my instrumented steps - and name
                 the single event I should add.
              7. Help me write one change as a falsifiable prediction: the
                 metric, the segment, the expected value, and the date I
                 will check it.
              8. Tell me three ways my re-measurement could fool me into
                 declaring a false win.

CONSTRAINTS:  Free tier only. Quote a base size beside every percentage
                 you produce. Do not propose more than one change. Plain
                 English, explain every term on first use. No code.

NON-GOALS:    Do not design an A/B test; chapter nine covers that and I
              may not have the traffic. Do not redesign my screens. Do not
              propose new events beyond the single missing step. Do not
              modify any files.

ACCEPTANCE:   - You interrogated the funnel before interpreting it
              - Every percentage you quote carries its base size
              - Filter and leak are distinguished by evidence, not by size
              - Operational numbers are checked before any design opinion
              - I leave with exactly one change, one predicted number and
                one date

ASK:          First ask me up to five questions about my funnel, my
              instrumentation and my users. Then answer requirement 1
              only, and stop before interpreting any drop.
```

## Mission

**Objective.** A segmented funnel that locates one real leak with evidence, one shipped change with a prediction written beforehand, and an honest re-measurement including the case where nothing moved.

**Steps.**
1. **Frame** — write the funnel definition: ordered steps, window, distinct people. Confirm the first step fires on a deliberate action, not a page load.
2. **Measure** — build it for the last four weeks with counts, percentages and base sizes.
3. **Classify** — for your two biggest drops, argue filter or leak using effort signals from your event data, not intuition.
4. **Segment** — split by connection, device, acquisition channel and new-versus-returning, one dimension at a time, base size stated every time.
5. **Overlay** — for the leaking step in the worst segment, record error rate, p95 and slow-connection failure rate before forming an opinion.
6. **Watch one person** — find a live user matching that segment and observe them attempt the step on their own phone, in silence.
7. **Predict** — write the change, the metric, the segment, the expected value and the check date. Before shipping.
8. **Ship one thing** — exactly one change.
9. **Re-measure** — the identical funnel on the identical segment after the stated period, and record the result whatever it is.

**Upload.**
- Your funnel definition, including the window and the people-not-events statement
- The overall funnel with counts, percentages and base size → screenshot
- At least four segmented funnels, each with its base size → screenshots
- Your filter-versus-leak argument for two drops, with the effort signal you used
- The three operational numbers for the leaking step → screenshot
- Notes from watching one real user attempt that step
- Your written prediction, dated before the change shipped
- The re-measurement against the identical definition → screenshot
- Your AI conversation where it interrogated the funnel → share link in `ai_chat_url`
- Reflection, 150 words max → the segment that surprised you, and one diagnosis the AI offered that the data did not support

**Self-check.** Tick every box before you submit.
- [ ] My funnel's steps are genuinely sequential for one individual
- [ ] It counts distinct people and states a time window
- [ ] My first step fires on a deliberate action, not a page load
- [ ] Every percentage I quote has its base size beside it
- [ ] I segmented by at least four dimensions, one at a time
- [ ] I can name a segment, its size and its drop in points
- [ ] I checked error rate and p95 for the leaking step before blaming design
- [ ] I argued filter versus leak using an effort signal from my own data
- [ ] I watched at least one real user attempt the leaking step
- [ ] I wrote the predicted number and date before shipping
- [ ] I shipped exactly one change
- [ ] I re-measured the identical funnel definition and reported the real result

**Stretch.** Take the drop you decided was a filter and try to prove yourself wrong: find five users who exited there and ask each what they were doing. Write 100 words on whether the filter was really a filter.

## Stuck? Ask AI
```prompt
Here is my funnel with counts at each step: [paste it, with the base size]. Do not interpret it yet. First tell me everything that could be wrong with the funnel itself: whether the steps are really ordered, whether it counts people, whether a window is stated, and whether the top step could be firing on page load. Then tell me which single check I should run first and why.
```

```prompt
Explain the difference between a drop that is a healthy filter and a drop that is a real leak, using a canteen queue as the example. Then give me eight drop-offs described in one sentence each, some filters and some leaks, and ask me to sort them and justify each answer before you tell me anything.
```

```prompt
I spent two weeks redesigning a screen and my funnel moved by one percentage point. Here is the funnel and the change I made: [paste both]. Tell me, in order, the diagnostic steps I should have run before redesigning anything, and for each one say what I would have seen if that was the real cause.
```

## Quiz

### Q1 · L4
Your payment step shows an 18-point drop overall. Split by connection, it is 9 points on fast networks and 29 on slow ones. What does the overall number represent?
- (a) The true drop, which the segments only approximate
- (b) An average of two different problems, describing neither of them ✅
- (c) Evidence that the segments are too small to trust
- (d) A sign the funnel steps are out of order
> Averages hide leaks by blending populations. Neither group experiences 18 points, so a fix aimed at the average helps one group marginally and the other not at all — which is exactly how two weeks buys one point.

### Q2 · L4
Two users exit at your payment screen. One left after four seconds without touching anything. The other spent ninety seconds, retried twice, then left. What do these tell you?
- (a) Both are leaks, since both abandoned at the same step
- (b) Both are filters, since neither completed the purchase
- (c) The first looks like a filter, the second is strong evidence of a leak ✅
- (d) Nothing without a larger sample
> Effort before exit is the signal. Someone who tried twice had decided to buy and was stopped; someone who glanced and left may never have intended to. Time on step and retries are the evidence your events can supply.

### Q3 · L5
You split your funnel four ways and find a segment of 14 users with a 40-point drop. What is the correct response?
- (a) Fix it immediately; 40 points is the largest drop you have
- (b) Treat it as a lead worth investigating, not a finding worth acting on ✅
- (c) Discard it, since small segments are meaningless
- (d) Re-run the split until the segment grows
> Fourteen people make each person seven percentage points, so the number is noise around an anecdote. The honest use is to go and watch one of those fourteen, which is how a lead becomes evidence.

### Q4 · L3
Before accepting that your drop-off is caused by confusing design, which should you check first?
- (a) Whether competitors' screens look different
- (b) The step's error rate, p95 load time and slow-connection failure rate ✅
- (c) Whether the button colour has enough contrast
- (d) How many users refused analytics consent
> A step that errors or takes eleven seconds has no design problem worth discussing. Checking the plumbing takes minutes and routinely prevents a fortnight of redesign.

### Q5 · L4
You ship a fix, and a week later your conversion looks better. Which would most undermine the claim that your fix caused it?
- (a) The sample is larger than before
- (b) The before window was exam week and the after window was not ✅
- (c) The fix was implemented by AI rather than by hand
- (d) The improvement was 6 points rather than 20
> Seasonality is the most common false win in this chapter. If the two windows differ in character, the thing you measured is the calendar, and the identical funnel definition does not protect you from that.

## Go deeper
- [PostHog docs: funnels and drop-off analysis](https://posthog.com/docs/product-analytics/funnels)
- [Mixpanel: segmentation and breakdowns](https://docs.mixpanel.com/docs/reports/insights)
