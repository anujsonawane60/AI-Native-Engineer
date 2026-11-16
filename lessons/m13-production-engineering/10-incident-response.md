---
id: m13-10-incident-response
module: m13-production-engineering
slug: incident-response
title: "Incident Response, On-Call and Blameless Postmortems"
order: 10
duration_min: 30
is_free: false
mission: false
volatility: low
last_reviewed: 2026-11-16
---

## Key learnings
- Declare an incident out loud and name one commander before anyone debugs
- Mitigate first and diagnose second, and know why that order is not negotiable
- Write a blameless postmortem whose action items have owners, dates and tests
- Design an on-call arrangement three students with exams can actually sustain
- Catch a postmortem whose root cause is "human error"

## Lesson

### The pain

M05 chapter 8 taught you reliability as a set of decisions made in advance: SLIs, SLOs, error budgets, timeouts, retries with backoff, and circuit breakers. It taught the availability arithmetic — 99.9% is forty-three minutes a month — and it handed runbooks forward. M07 taught you to see: logs, metrics, traces, p95, and alerts that fire on the things that matter. You can now detect a problem and you have defences against the predictable ones. This chapter is about the twenty minutes after the alert fires, which is a human process and the one nobody practises.

9:40 on a Friday evening. Payments start failing. Three teammates all notice within two minutes, and all three start debugging — in three separate WhatsApp threads, two of them with the same log file open. Nobody has told any user anything, so students keep trying to pay and keep failing. At 10:05 one teammate deploys a fix they are fairly confident about. It does not work, and now the system is in a state nobody can describe, because two people have changed things and neither wrote down what. At 11:20 somebody finds the cause — a third-party key that expired — and at 11:35 it is fixed. The next morning nobody can reconstruct the timeline, so the postmortem says "key expired, we renewed it, we will be more careful", and the same class of failure happens again in March with a different key.

Total damage: one hour and fifty-five minutes of broken payments, about forty students who tried and gave up, and zero learning. Almost all of it is process, not engineering. A declared incident with one commander and a rollback at 9:45 would have made this a fifteen-minute event.

### The idea

**An incident is a temporary change in how your team makes decisions: one person decides, one person talks to users, one person writes down what happened, and the first action is always to stop the bleeding rather than to understand it.**

Three roles, even with three people. The **commander** decides and explicitly does not debug — their whole job is to keep one plan running and stop three people changing things at once. The **comms** person tells users, on a schedule, even when there is nothing new. The **scribe** writes a timestamped line for every observation and every action, as it happens, because that log is the only honest source for the postmortem.

### The analogy

There is a gas smell at the canteen counter at 1 p.m.

What a good head cook does, in order: says the words "we are closing this counter" out loud so everyone hears it; sends one person to stand in front of the queue and tell students what is happening; shuts the main valve, which stops the danger without yet knowing where the leak is; and only then starts looking for the leak. Somebody is writing the times on a pad, because the gas company will ask.

What a bad response looks like: three people sniffing different pipes, nobody telling the queue anything so students keep crowding in, and one person tightening a joint on a hunch while another has already turned a different valve.

**Where the analogy breaks:** a gas leak announces itself to everyone in the room instantly and unambiguously. A software incident is invisible, and the first person to notice is often a user on WhatsApp twenty minutes in. That is why M07's alerting exists — it is the smell — and why "how did we find out?" is the first question of every postmortem. If the answer is "a student messaged us", your detection is the first thing to fix, not the bug.

### Picture it

```
  10:04  DETECT      an alert fires, or a user messages you
  10:06  DECLARE     "This is a Sev-2." One commander, named.
            ├─ COMMANDER  decides; does NOT debug
            ├─ COMMS      updates users every 20 min
            └─ SCRIBE     one timestamped line per event
  10:09  MITIGATE    roll back, or flip the kill switch
                     ◀── stop the bleeding BEFORE diagnosing
  10:14  RECOVERED   confirmed by a check you ran
  10:40  CLOSED      the commander says it is over
  +2 d   POSTMORTEM  blameless; actions with owners and dates
```

An incident runs as a fixed sequence with named roles, mitigation happens before diagnosis, and recovery is declared from a check rather than from a feeling.

### Where it lives

This is the operational half of M05 chapter 8. Your SLO decides what counts as an incident at all, and your error budget decides what you do afterwards. M07's alerts are the detection step, and M07's traces are what make diagnosis possible once you have mitigated. Chapter 5 supplies the mitigation: a tested rollback with a known stopwatch time, and a kill switch for the risky dependency. Chapter 9's runbook is what the commander reads when the answer is a failover. Chapter 6 is where an action item goes to die properly — a postmortem item that is only a sentence will be forgotten, and the same item written as a regression test will not.

M18's handover asks for your incident history and your postmortems, because they are the strongest available evidence that you operated a real product rather than demonstrated one.

### Words to own

| Term | Plain meaning | Example |
|---|---|---|
| Incident | User-visible impact that needs a coordinated response | Payments failing |
| Severity | How bad, on a small fixed scale | Sev-1, Sev-2, Sev-3 |
| Declare | Saying out loud that this is an incident | "This is a Sev-2" |
| Incident commander | The single decision-maker; does not debug | One named person |
| Comms | The person who tells users, on a schedule | Every 20 minutes |
| Scribe | Writes the timestamped timeline live | The postmortem's source |
| Mitigate | Stop the impact, without understanding it yet | Roll back |
| Time to detect | Problem start → somebody knew | 22 minutes |
| Time to mitigate | Somebody knew → users unaffected | 5 minutes |
| Blameless | Asking how the system allowed it, not who erred | The whole stance |
| Contributing factor | One of several things that had to be true | Preferred to "root cause" |
| Action item | A change with an owner and a date | "Add expiry alert, Priya, 20 Nov" |
| Error budget policy | What you stop doing when the budget is spent | Freeze features |
| Status page | Where users read what is happening | Even a pinned post |
| Runbook | Numbered steps for one specific failure | Read at 2 a.m. |

### Tradeoffs and gotchas

**Mitigate before you diagnose, always.** The instinct of every engineer is to understand first. It is wrong during an incident, because users are being affected while you are being curious. Roll back, flip the kill switch, take the feature off the board. You will often lose the evidence by doing this, and that is an acceptable trade — take a copy of the logs and a screenshot of the dashboard first, then mitigate. The diagnosis can happen at leisure with nobody suffering.

**One person changes things.** Two people independently fixing the same incident produce a state nobody can reason about, and it is the single most common way a twenty-minute incident becomes a two-hour one. The commander grants the right to change, one change at a time, and the scribe records each one.

**Tell users early and say less than you know.** "We know payments are failing, we are working on it, next update in twenty minutes" costs you nothing and buys enormous goodwill. Silence makes students retry, which increases load, which makes everything worse — the retry storm from M05 chapter 8, generated by humans. You do not need a status-page product; a pinned post or a banner in your own app is a status page.

**Blameless is a technique, not kindness.** The question is never "who deployed it" but "what allowed a change to reach production without being caught?" Human error is never a root cause; it is the thing a system is supposed to tolerate. If a postmortem concludes "we will be more careful", it has failed, because carefulness is not a mechanism and cannot be deployed. Replace it with an alert, a test, a guard, or a checklist item.

**Prefer "contributing factors" to "root cause".** Real incidents need several things to be true at once: the key expired, *and* there was no expiry alert, *and* the error message was unclear, *and* nobody was watching on a Friday. Fixing one of four is weaker than you think, and naming all four shows you understood it.

**Every action item needs an owner, a date and ideally a test.** "Improve monitoring" is not an action item. "Add an alert for any credential expiring within fourteen days — Priya — 20 November" is. And the item that closes the loop best is a chapter 6 regression test, because it keeps working after everyone forgets.

**An on-call rota for three students with exams is a fiction, and pretending otherwise is the dishonest option.** Say instead: we respond during stated hours; outside them, the system must degrade politely rather than fail hard. That is achievable with M05 chapter 8's graceful degradation plus chapter 5's kill switches, and it is a far stronger answer in M18 than a rota nobody kept. Tell users your hours and keep them.

**Spend the error budget deliberately.** M05 chapter 8 gave you the budget; this is the policy. When a month's budget is gone, you stop shipping features and spend the time on reliability instead. Writing that rule down before you need it is what stops it becoming an argument during the week you need it most.

### Verify it

Incident response is the one area where you verify your own process rather than AI's output, but AI will happily write you a postmortem, and the failure modes are specific.

1. **Check the timeline is timestamped and was written live.** A timeline reconstructed from memory the next day is always tidier and less true than the real one. Look for the awkward gaps; they are the honest parts.
2. **Read the "how did we find out" line.** If it says a user told you, your top action item is detection, not the bug.
3. **Find the two intervals: time to detect and time to mitigate.** A postmortem with no numbers has nothing to improve against next time.
4. **Search the document for the word "careful", and for "human error".** Both are signs the analysis stopped early. Make it name the mechanism that would have caught it instead.
5. **Check every action item has a name and a date.** Count the ones that do not; that count is how much of this postmortem will be forgotten.
6. **Ask what would have happened if it had started at 3 a.m.** If the honest answer is "nothing until morning", then say so publicly and make the system degrade politely — do not pretend a rota exists.
7. **Confirm at least one action item is executable** — a test, an alert, a guard — rather than a resolution.

Red flags: a postmortem with "human error" as the cause; action items with no owner; no timeline; an alert that fires with no runbook attached; an incident where three people were changing things; users never told anything; no severity declared, so nobody knew whether to wake up; a mitigation attempted before any evidence was saved; and the same contributing factor appearing in two postmortems three months apart.

## Walkthrough
1. Write a severity scale with three levels for your own product. Define each in terms of user impact, not engineering difficulty: Sev-1 is payments or data loss, Sev-2 is a core feature broken, Sev-3 is degraded but usable. One line each.
2. Write down, as a team, who may declare an incident and who may be commander. With three people, agree a rotation and write the order. A rule decided now is a rule you will follow at 9:40 on a Friday.
3. Write your stated response hours into your product and your repository, honestly. Then list what must be true for the system to survive outside those hours: graceful degradation, kill switches, a queue that retries, a limit that stops a runaway cost.
4. Create your status channel — a pinned post, a banner, or a simple page — and write the three template messages in advance: we have a problem, we have mitigated it, it is resolved. Writing them calm is the point.
5. Create your postmortem template in the repository: summary, user impact with numbers, how we found out, timeline, contributing factors, what went well, action items with owner and date, and one line on what would have happened at 3 a.m.
6. Now run a game day. Pick a real failure you can cause safely in staging — revoke a third-party key, stop the cache, point a connection string at nothing. Do not tell the whole team which one.
7. Run the full process on it: declare, name a commander, assign comms and scribe, post the first user message, save evidence, then mitigate. Keep the timeline live as you go.
8. Measure two numbers from your own timeline: time to detect and time to mitigate. Write both down. These are the numbers you will try to reduce next time.
9. Hold the postmortem within two days, using the template, with nobody's name attached to a mistake. Produce at least three contributing factors rather than one cause.
10. Turn one action item into a chapter 6 regression test and one into an M07 alert. Assign an owner and a date to every remaining item.
11. Finally, write your error budget policy in one sentence — what your team stops doing when the budget for the month is gone — and commit it next to the postmortem template.

## Talk to AI
```prompt
CONTEXT:      I am a student on an AI-native engineering program. I direct AI
              and verify its output; I do not write code by hand. My team of
              three runs [product in two sentences] with [number] real users
              in India, handling [yes/no] real payments. We are students with
              exams; nobody can be woken at 3 a.m. and we will not pretend
              otherwise. Our monitoring today: [describe what alerts exist, or
              say none]. Our SLO: [state it, or say none]. We have [a tested
              rollback / no tested rollback] and [kill switches / none]. Our
              last real outage was [describe it, including how we found out
              and how long it lasted].

GOAL:         An incident process a three-person student team will actually
              follow, an honest on-call arrangement, and a postmortem template
              that produces action items we complete rather than resolutions
              we forget.

REQUIREMENTS: 1. A three-level severity scale defined by USER IMPACT for my
                 specific product, with one example of each from my app
              2. The three roles - commander, comms, scribe - and how to run
                 them with only three people. Tell me what the commander must
                 NOT do and why that rule matters most
              3. The order of operations during an incident, and the argument
                 for mitigating before diagnosing. Tell me what evidence to
                 save first so mitigation does not destroy it
              4. Three pre-written user messages: problem acknowledged,
                 mitigated, resolved. Plain language, no jargon, suitable for
                 students. Tell me how often to update when there is no news
              5. An honest on-call design for students with exams: stated
                 response hours, and exactly what must be true about my system
                 for the hours outside that to be survivable
              6. A postmortem template including user impact with numbers, how
                 we found out, a live timeline, CONTRIBUTING FACTORS rather
                 than a single root cause, and action items with owner and date
              7. Explain blameless analysis and tell me why "we will be more
                 careful" and "human error" are failures of analysis. Give me
                 three replacements for each, as mechanisms
              8. An error budget policy in one sentence: what we stop doing
                 when the month's budget is spent
              9. Design me a safe GAME DAY in staging: three failures I can
                 cause without risking real users, and what each one should
                 teach me

CONSTRAINTS:  Free tools only - no paid incident-management or status-page
              product; a pinned post or an in-app banner is acceptable. Plain
              English for a second-language reader. Nothing may require a
              person to be awake at night. Do not touch production during the
              game day. Label every command READ-ONLY or CHANGES-THINGS.

NON-GOALS:    Do not design a 24/7 rota. Do not recommend buying monitoring or
              on-call software. Do not write me a postmortem for a
              hypothetical incident - give me the template and the process. Do
              not add new alerts in this conversation beyond telling me which
              ones matter.

ACCEPTANCE:   - I have a three-level severity scale my team agreed on
              - One named person per incident decides, and that rule is
                written down with a rotation order
              - My three user messages are written in advance
              - My response hours are published and my system degrades
                politely outside them
              - My postmortem template asks for contributing factors, numbers,
                and owner-plus-date action items
              - I ran a game day and measured time to detect and time to
                mitigate
              - At least one action item from it became a test or an alert
              - My error budget policy is one committed sentence

ASK:          Before anything, ask me how we found out about our last outage
              and how long it lasted. If the answer is that a user told us,
              say so plainly and make detection the first thing we fix.
```

## Stuck? Ask AI
```prompt
Here is a postmortem my team wrote: [paste it]. Review it as if you will be answering for the next identical outage. Tell me: is the stated cause actually a cause or is it a person being blamed? How many contributing factors are missing? Which action items have no owner or no date? Which of them is a resolution rather than a mechanism? And what single thing would have shortened this incident the most? Do not rewrite it - tell me what to ask my team.
```

```prompt
Explain why mitigating before diagnosing is the correct order during an incident, then argue the other side as strongly as you can and tell me when diagnosing first is genuinely right. Give me a list of evidence I should capture in the sixty seconds before I mitigate, so that mitigation does not destroy what I need for the postmortem. Then ask me what my fastest mitigation actually is and whether I have ever timed it.
```

```prompt
I am three students with exams and we cannot have a night-time on-call rota. Do not tell me to get one. Instead, help me design the honest alternative: stated response hours, and the specific properties my system needs so an overnight failure waits politely until morning. Go through each part of a typical food-ordering app - payments, orders, notifications, background jobs, third-party calls - and tell me what "degrade politely" means for each, and which one genuinely cannot wait.
```

## Quiz

### Q1 · L3
An alert fires and three teammates all start debugging in separate chats. What is the first thing that should have happened?
- (a) Each person takes a different layer of the stack to be efficient
- (b) Someone declares an incident and names one commander who decides and does not debug ✅
- (c) The team reverts the most recent deploy immediately
- (d) Someone opens a trace to find the slow span
> Three people changing things independently produces a state nobody can describe, which is the commonest way a twenty-minute incident becomes a two-hour one. Declaring and naming is a cheap step that makes every later step possible.

### Q2 · L4
You know payments are failing and you do not know why. What do you do first?
- (a) Diagnose, because mitigating without understanding may make it worse
- (b) Save evidence quickly, then mitigate — roll back or flip the kill switch — and diagnose afterwards ✅
- (c) Wait ten minutes to see whether it recovers on its own
- (d) Post a detailed explanation to users so they understand the cause
> Users are being affected while you are being curious. Capture the logs and the dashboard, then stop the bleeding; the diagnosis loses nothing by happening calmly an hour later with nobody suffering.

### Q3 · L4
A postmortem concludes: "the key expired because a teammate forgot to renew it; we will be more careful." What is wrong?
- (a) It names the teammate, which is a privacy issue
- (b) Human error is not a cause and carefulness is not a mechanism — the missing expiry alert is the finding ✅
- (c) It should have been written within 24 hours rather than two days
- (d) The severity level is not stated
> Blameless analysis asks what allowed the failure to reach users. A system is supposed to tolerate a person forgetting. Replace the resolution with an alert, a test, a guard or a checklist item — something that keeps working after everyone forgets.

### Q4 · L3
Why does telling users early matter operationally, not just reputationally?
- (a) It satisfies most consumer-protection requirements
- (b) Silence makes users retry, which increases load and worsens the incident ✅
- (c) It gives the team more time before a fix is expected
- (d) It allows users to diagnose the problem for you
> Humans generate the retry storm from M05 chapter 8 all by themselves. "We know, we are working on it, next update in twenty minutes" costs nothing and reduces the load you are already struggling with.

### Q5 · L5
Your three-person team is asked how you handle a 3 a.m. outage. What is the strongest honest answer?
- (a) We have a rotating on-call schedule covering all hours
- (b) We respond during stated hours, and outside them the system degrades politely through kill switches and graceful degradation ✅
- (c) We rely on automated recovery, so no human response is needed
- (d) We monitor continuously and respond as quickly as we can
> A rota three students with exams will not keep is worse than no rota, because it makes the system's real behaviour unplanned. Stating your hours and engineering for them is a design decision, which is exactly what M18 will want you to defend.

## Go deeper
- [Google SRE workbook: incident response and the on-call model](https://sre.google/workbook/incident-response/)
- [Etsy: debriefing facilitation guide for blameless postmortems](https://extfiles.etsy.com/DebriefingFacilitationGuide.pdf)
