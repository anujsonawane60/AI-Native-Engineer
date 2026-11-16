---
id: m07-07-error-tracking-and-alerting
module: m07-security-observability-analytics
slug: error-tracking-and-alerting
title: "Error Tracking, Uptime Monitors and Alerting Without Fatigue"
order: 7
duration_min: 35
is_free: false
mission: true
volatility: medium
last_reviewed: 2026-11-16
---

## Key learnings
- Apply the test that separates a real alert from a notification nobody reads
- Explain how an error tracker turns thousands of events into a few issues
- Alert on symptoms users feel rather than on causes you guessed
- Write a runbook line that makes a 2am alert actionable by a teammate
- Set up an outside-in uptime check that cannot lie to you

## Lesson

### The pain

A team wires their app to a chat channel so they will know when something breaks. By week two the channel has 900 unread messages, most of them the same timeout that happens eleven times an hour and recovers by itself. Somebody muted the channel on day four. Everybody muted it by day six.

In week five the payment flow breaks completely. The alert fires. It arrives in a muted channel, between two identical timeout notices, and sits there for nine hours. They find out from a student in the canteen queue.

A different team has no alerts at all and checks the app each evening. Their database free tier was exhausted on a Saturday morning; they found out on Sunday night, thirty-one hours in.

Both teams failed at the same thing: deciding, in advance, what deserves a human.

### The idea

**An alert is a promise that a human will do something, now. If there is no action, it is not an alert — it is a log line with ambition.**

That single test removes most of what teams set up. Interesting? Dashboard. Needs attention this week? Ticket. Needs a person *right now* or users keep suffering? Alert. The mistake is sending everything to the third destination.

### The analogy

The canteen has a gas-leak alarm. It has sounded twice in six years, and both times the kitchen emptied in fifteen seconds, because that alarm means exactly one thing and everybody knows what to do.

Now wire a second alarm to the toaster, sounding every time bread browns. Within a week nobody hears either. The toaster alarm has not merely added noise; it has destroyed the gas alarm, which was the one that mattered.

Alert fatigue is not tiredness. It is the measurable fact that the fortieth notification of the day makes the forty-first invisible, including the one that was real.

**Where the analogy breaks:** a gas alarm is single-purpose and cannot be tuned. Your alerts are yours to design, which is why "delete an alert nobody acts on" is a healthy weekly act rather than an admission of failure. The channel is a working tool, not a record, and it needs pruning.

### Error tracking: from four thousand events to three issues

Your logs record every failure as a separate line. An error tracker adds the one thing that makes errors usable: it **groups** them.

It reads the shape of each failure — where it happened, what kind of error, the chain of calls — and computes a fingerprint. Four thousand failures with the same fingerprint become one **issue**, with a count of events and a count of distinct users affected. That second number decides priority: four thousand events from one bot is a shrug, forty events from forty students is an emergency.

What a tracker gives you that raw logs do not:

| Feature | What it means | Why it matters |
|---|---|---|
| Grouping | Identical failures become one issue | Three problems, not 4,000 lines |
| User impact | How many distinct people hit it | Priority, not volume |
| First and last seen | When it started; whether it is ongoing | "14 minutes after our deploy" |
| Release tagging | Which version introduced it | The change to revert |
| Breadcrumbs | The user's last actions before it broke | Reproducing without asking |
| Regression detection | A resolved issue has returned | The fix did not hold |

Release tagging deserves a note. An issue that first appeared fourteen minutes after a deploy, in that release only, has told you where the bug is without anybody reading code — which is worth more to you than to someone who wrote it themselves.

### Uptime monitors: checking from outside

Everything so far runs *inside* your app. If the app is completely down it reports nothing, and silence looks identical to health.

An **uptime monitor** is a service elsewhere that requests a URL of yours every minute or two and tells you when it stops answering. It is the only check that survives your app being dead, and the cheapest useful thing in this chapter. Two details decide whether it is real or theatre.

**What it requests.** A monitor hitting your home page — a static file on a CDN — reports "up" through a total database outage. Give it a **health endpoint** that does one small piece of real work, such as a trivial database query, and fails when that work fails. An endpoint that returns success unconditionally is worse than no monitor, because it manufactures confidence.

**Where it checks from.** Pick a location near your users. A monitor in Virginia tells you little about a student in Nashik on 4G — the same argument as measuring p95 where the user is. Add a free **status page** if your platform has one; it turns "is it down for everyone?" into a link you send people.

### Picture it

```
       thousands of raw failures
                 │
                 ▼
     ┌────────────────────────┐
     │  error tracker groups  │  4,000 events ─▶ 3 issues
     │  by fingerprint        │  + users affected
     └───────────┬────────────┘
                 │
                 ▼
      is a human needed NOW?
          │            │
         yes           no
          │            │
          ▼            ▼
    ┌──────────┐  ┌──────────────────┐
    │  ALERT   │  │ ticket, or just  │
    │  a phone │  │ the dashboard    │
    └────┬─────┘  └──────────────────┘
         │
         ▼
   runbook: what to check first, who to tell
```

Failures are grouped before anyone is told, then routed by one question — is a person needed now — and only the "yes" branch reaches a human, carrying a runbook with it.

### Alerting without fatigue

Six rules. They are not opinions; each one is a specific failure mode that student teams hit within a month.

**1. Alert on symptoms, not causes.** "Error rate above 5% for five minutes" is a symptom; users are feeling it. "CPU above 80%" is a guess about a cause, and it fires during every healthy traffic spike. Causes cannot be enumerated in advance; symptoms are a short, stable list — users getting errors, users waiting too long, the app not answering at all.

**2. Every alert needs a duration.** "Error rate above 5%" fires on one bad second; "above 5% for five minutes" fires when something is wrong. Without a duration you get flapping — fire, resolve, fire, resolve — and a muted channel.

**3. Two severities, not five.** *Page* means wake someone. *Notice* means look at it next session. A student team needs to know which messages may be read tomorrow, not a taxonomy.

**4. Every alert carries a runbook line.** One sentence, written when you create the alert: what to check first, and who to tell. "DB connections exhausted → open the database dashboard, restart the app if above 18, tell the group." At 2am nobody invents this.

**5. Start with two alerts.** Usually the uptime check failing for three minutes, and the error rate above a threshold for five. Add a third only after one of those has fired and been acted on. Teams that start with fifteen have fifteen muted alerts by month end.

**6. Delete the ones nobody acts on.** Review the channel weekly. The honest question is not "is it interesting?" but "did anybody do anything?"

### Where it lives

This is where chapter four's three pillars become somebody's phone buzzing. The error-rate and latency signals come from chapter six; the detail you open afterwards is chapter five's logs, which is why a request id on every line matters at 2am.

M05's error budgets are the mature version of rule one: you alert when you are burning your allowance of failures faster than the month can absorb. M06's CI is the other half — a test that fails before deploy is cheaper than an alert after it. M11's going-live checkpoint asks for working monitoring and alerting before you call the product live, and this mission is what you will show there. M12 revisits all of it as mobile crash reporting.

### Words to own

| Term | Plain meaning | Example |
|---|---|---|
| Error tracker | A service that groups failures into issues | 4,000 events, 3 issues |
| Fingerprint | The signature used to group identical failures | Same error, same place |
| Issue | One grouped problem with a count | "DB timeout on /orders" |
| Uptime monitor | An outside service requesting your URL on a schedule | Every 60 seconds |
| Health endpoint | A URL that does real work and reports honestly | Runs one query |
| Alert | A notification that demands action now | Page the on-call |
| Alert fatigue | So many notifications that none are read | A muted channel |
| Runbook | The written first steps for a specific alert | One sentence per alert |
| On-call | Whose turn it is to answer | One of three, one week each |
| Flapping | An alert firing and resolving repeatedly | No duration set |

### Tradeoffs and gotchas

**Alerting on causes is the most common design error.** CPU, memory, disk and queue depth all feel like the right things to watch and all fire when nothing is wrong. Watch what users feel; investigate causes afterwards.

**A free tier can be your outage.** A database row limit, a log quota, an AI spend cap. None looks like a bug, none appears in an error tracker before it bites, and all of them stop your app. Put your nearest quota on the dashboard; it is a strong candidate for your third alert.

**One person on call, in writing, one week at a time.** In a group of three this is a rota in your team agreement, not "we all watch it". Shared responsibility is how an alert goes unanswered with three people awake.

**Error trackers have event quotas too.** A loop reporting an exception per item can consume a month's free events in an afternoon, and a user's wrong password is not an error.

**Do not alert on the p99.** It moves for one person on a train. Alert on the error rate and on a p95 that stays bad for several minutes.

**Resolve means fixed, not hidden.** An issue marked resolved that reappears forty times teaches the team to ignore the tracker.

### Verify it

Five checks, and the first two catch almost everything a generated setup gets wrong.

1. **Read your alert channel for the last week and count the actions taken.** If twenty messages produced zero actions, you have **an alert that fires so often nobody reads it** — the headline red flag of this chapter. Tune or delete it today.
2. **Break your health endpoint and confirm the uptime monitor notices.** Point its database check at a wrong address. If the monitor still reports "up", it is checking a static page or returning success unconditionally — a common generated shape, and pure theatre.
3. **Cause one real error and follow it end to end.** Did the tracker group it, name the release, show the user count and leave breadcrumbs? Could you then find the matching log lines by request id?
4. **Read every alert's runbook line.** If one has none, write it or delete the alert. Then hand it to a teammate to follow without you; anything they cannot do is missing from the line.
5. **Check the quota on every free tier you depend on**, and write down the date each runs out at current usage.

Red flags: every error event forwarded straight to chat with no grouping; a health endpoint that always returns success; alerts on CPU with no user-facing symptom; errors grouped per user so one bug appears as four hundred issues; no duration on any threshold; and "we all watch the channel" instead of a named on-call.

## Walkthrough
1. Write down the three worst things that could happen to your product this month, in user terms: "nobody can place an order", "payments silently fail", "the app does not load at all".
2. For each, write how you would find out today. Be honest. If the answer is "a user tells us", that is the gap this chapter closes.
3. Add an error tracker to your deployed app and confirm it is capturing server-side failures, not only browser ones. Load a page and check that nothing sensitive appears in the captured context.
4. Deliberately cause one error in production — a wrong configuration value is easiest — and watch it arrive. Confirm you can see the release, the user count and the breadcrumbs.
5. Create a health endpoint that performs one small real piece of work, such as a trivial database query, and returns failure if that work fails. Test it by breaking the thing it checks.
6. Set up an outside uptime monitor against that endpoint, checking every minute or two, from a location close to your users. Confirm it reports up.
7. Break the health endpoint's dependency again and wait for the monitor to notice. Time how long it took, and screenshot the notification.
8. Create exactly two alerts: the uptime check failing for three minutes, and the error rate above a threshold for five minutes. Both with a duration. Both routed somewhere a human sees them.
9. Write a runbook line for each: what to check first, and who to tell. Put both in your repository, not in a chat message.
10. Hand the runbook to a teammate and have them follow it while you cause the failure. Note every step they could not complete and fix the wording.
11. Add a line to your team agreement naming who is on call this week and how the rota rotates. Set a weekly reminder to review the alert channel and delete anything nobody acted on.

## Talk to AI
```prompt
CONTEXT:      I am a student in a team of three. I direct AI and verify its
              output; I do not write code. Our product: [describe it in two
              sentences]. Stack: [your stack] on [platform], free tier,
              with [your database] and [external providers you call]. Users
              are in India on budget Android phones and slow 4G. We
              currently find out about problems when a user tells us. We
              already have server logs with a request id, and a dashboard
              with latency p95, traffic, error rate and our nearest quota.

GOAL:         Error tracking and an outside-in uptime check in place, plus
              exactly two alerts that a team of three will still be reading
              in a month.

REQUIREMENTS: 1. List the three worst user-facing failures for a product of
                 this shape, and for each one say which signal would detect
                 it: the error tracker, the uptime monitor, or a metric
                 threshold.
              2. Specify a health endpoint for my stack: what real work it
                 should do, what it must return on success and failure, and
                 why checking my home page instead would be misleading.
              3. Specify exactly two alerts. For each give me the signal,
                 the threshold, the duration it must persist, the severity,
                 where it is routed, and a one-sentence runbook naming what
                 to check first and who to tell.
              4. Tell me three alerts I will be tempted to add that I
                 should not, and explain for each what would make it fire
                 when nothing is wrong.
              5. Tell me what personal data an error tracker captures by
                 default — request bodies, headers, cookies, user context —
                 and exactly what I must configure it to strip, given that
                 the DPDP Act applies to us.
              6. Tell me what free-tier quotas could take my app down
                 without producing a single error, and how I would see each
                 one coming.
              7. Propose an on-call rota for three students with classes,
                 in two or three sentences, including what happens at night.

CONSTRAINTS:  Free tiers only. Explain every term on first use. Assume I
              verify by breaking things on purpose and reading the result.
              Exactly two alerts — argue me out of a third rather than
              adding one.

NON-GOALS:    Do not design dashboards; I have those. Do not add user or
              product analytics. Do not recommend paid plans, incident
              management platforms or status-page vendors beyond what my
              platform already includes free.

ACCEPTANCE:   - Each of my three worst failures has exactly one detector
              - My health endpoint fails when the database fails, and I know
                how to prove that by breaking it
              - Both alerts have a duration and a runbook line a teammate
                could follow without me
              - I can name what the error tracker must be told not to
                capture
              - I know the date my nearest free-tier quota runs out

ASK:          First ask me up to five questions about my stack, providers
              and team availability. Then give me the three-failures table
              and the health endpoint specification, and stop before the
              alert definitions.
```

## Mission

**Objective.** Your product tells you when it is broken, through exactly two alerts you have each personally triggered and acted on.

**Steps.**
1. **Frame** — write the three worst user-facing failures for your product, and how you would find out about each one today.
2. **Brief** — write a Director's Brief for error tracking, a health endpoint and exactly two alerts. Demand the specification before any setup, and include the redaction requirement for personal data.
3. **Direct** — set up the error tracker on the server side, the health endpoint that does real work, and the outside uptime monitor against it.
4. **Alert** — create your two alerts, each with a threshold, a duration, a destination and a runbook line in your repository.
5. **Break it on purpose, twice** — cause a real error and watch it group in the tracker; break the health endpoint's dependency and wait for the uptime monitor to fire. Time both.
6. **Verify the runbook** — hand it to a teammate and have them follow it while you cause the failure. Record every step they could not complete, and fix the wording.
7. **Check the redaction** — open one captured error and confirm no password, token, card number, OTP or full request body is in it.
8. **Rota** — add a named on-call rotation to your team agreement, with what happens at night, and a weekly reminder to prune the alert channel.

**Upload.**
- Your Director's Brief, in full → post caption or an attached file
- Your three-failures table with its detector for each → screenshot or text
- Screenshots of the grouped issue in the tracker, showing the release and the number of users affected
- Screenshots of both alerts firing, with the time from breakage to notification
- Your two runbook lines, and what changed after a teammate tried to follow them
- Proof of redaction → a screenshot of one captured error with nothing sensitive in it
- Your on-call rota line from the team agreement
- Proof it works → deployed URL in `demo_url`; the AI conversation → share link in `ai_chat_url`
- Reflection, 150 words max → the alert you decided not to create, and why

**Self-check.** Tick every box before you submit.
- [ ] My error tracker captures server-side failures, not only browser ones
- [ ] My health endpoint fails when its dependency fails, and I proved it by breaking it
- [ ] My uptime monitor checks from a location near my users, not the default
- [ ] I have exactly two alerts, and I can justify not having a third
- [ ] Both alerts have a duration, so neither flaps
- [ ] Each alert has a runbook line naming the first check and who to tell
- [ ] A teammate followed one runbook without me, and I fixed what they could not do
- [ ] I personally triggered both alerts and timed the delay
- [ ] I opened a captured error and confirmed no secret or personal data in it
- [ ] My team agreement names who is on call and how it rotates
- [ ] My reflection names an alert I chose not to create, in my own words

**Stretch.** Leave the setup running for a week, then review the channel and delete or re-tune anything that fired without anybody acting. Write 100 words on what the week taught you about your own thresholds.

## Stuck? Ask AI
```prompt
Explain alert fatigue using an example from a canteen kitchen. Then give me ten possible alerts for a food-ordering app and ask me, one at a time, whether each one is a real alert, a ticket or just a dashboard line. After each answer, tell me what would happen in month two if I got it wrong.
```

```prompt
Explain what a health endpoint is and why checking my home page is not the same thing. Then describe four different health endpoints for the same app, from useless to genuinely useful, and tell me how I would break each one on purpose to prove the monitor works.
```

```prompt
Here is an alert I set up: [describe the signal, the threshold and where it goes]. Interrogate it. Will it flap? Does it describe a symptom a user feels, or a cause I guessed? What is the runbook line, and could a teammate follow it at 2am without me? Rewrite the alert if it fails any of those.
```

## Quiz

### Q1 · L3
Which test decides whether something should be an alert?
- (a) Whether it indicates a serious technical problem
- (b) Whether a human must do something about it now ✅
- (c) Whether it happens more than once a day
- (d) Whether it affects more than ten users
> Alerts compete for the one scarce resource, a person's attention. Interesting goes on the dashboard, important-this-week becomes a ticket, and only "act now or users keep suffering" earns a notification.

### Q2 · L4
Your uptime monitor reports "up" throughout a complete database outage. What is the most likely cause?
- (a) The monitor checks too infrequently
- (b) It requests a page that does not touch the database ✅
- (c) The monitor is in the wrong region
- (d) The database failure did not produce errors
> A static home page served by a CDN answers happily while everything behind it is dead. The fix is a health endpoint that performs one small real query and fails honestly when that query fails.

### Q3 · L5
Your team adds an alert on CPU above 80%. It fires during every lunchtime rush, when the app is working fine. What is the underlying design error?
- (a) The threshold is too low and should be 95%
- (b) It alerts on a guessed cause rather than a symptom users feel ✅
- (c) It lacks a duration, so it flaps
- (d) CPU cannot be measured accurately on a free tier
> Raising the threshold postpones the problem. Causes are unbounded and often benign; symptoms — users getting errors, users waiting — are a short, stable list, which is why alerts belong there.

### Q4 · L4
An error tracker shows one issue with 4,000 events from 1 user, and another with 40 events from 40 users. Which do you work on first, and why?
- (a) The first, because 4,000 is a hundred times more events
- (b) The second, because user impact is the priority signal, not volume ✅
- (c) The first, because high-volume errors exhaust the event quota
- (d) Neither; wait for release tags before deciding
> Forty affected students is forty people having a bad day; 4,000 events from one client is usually a loop or a bot. Grouping exists precisely so you can read impact instead of counting lines.

### Q5 · L4
Your alert channel has twenty messages from last week and nobody took any action on any of them. What is the correct response?
- (a) Add a daily summary so the messages are easier to scan
- (b) Tune or delete the alerts, because an unread channel hides the real one ✅
- (c) Move the channel to email so it is harder to ignore
- (d) Nothing; the messages prove the monitoring is working
> Volume without action is the definition of fatigue, and it does not merely waste attention — it makes the one real alert invisible. Pruning weekly is maintenance, not failure.

## Go deeper
- [Google SRE Workbook: alerting on SLOs](https://sre.google/workbook/alerting-on-slos/)
- [Sentry docs: issues and grouping](https://docs.sentry.io/product/issues/)
