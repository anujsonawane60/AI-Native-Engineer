---
id: m01-09-daily-live-session
module: m01-team-startup-git
slug: daily-live-session
title: "The Daily Live Session: Standups, Blockers and Reviews"
order: 9
duration_min: 15
is_free: false
mission: false
volatility: low
last_reviewed: 2026-11-16
---

## Key learnings
- Use the daily two hours for the work that is hard to do alone
- Give a standup update in three sentences that are actually useful
- Raise a blocker before it has cost you a day
- Come to a review with a specific question rather than "it doesn't work"
- Keep the session useful when you have nothing visible to show

## Lesson

### The pain

A group joins the live session every day for two weeks. Each day, each person says "yeah, still working on the frontend" and the session moves on.

In week three it emerges that one of them has been stuck on the same deployment error for nine days. They did not raise it because it felt like something they should be able to solve, and each day the admission got more embarrassing. The error took four minutes to fix once said out loud. This is the commonest way a self-paced learner loses a month.

### The idea

**The daily session is for the things that are slow or impossible alone: being unblocked, having your reasoning checked, and showing work to someone who will be honest about it.**

Reading lessons and briefing an agent are things you do alone. Discovering that your entire approach is wrong is much faster with another person in the room.

### The analogy

A canteen has a five-minute huddle before lunch rush. Nobody reports their whole plan; they say three things — cylinder is low, the paneer did not arrive, we are short one at the counter — decide who handles what, and start. The huddle is for **exceptions**: nobody says "I will be cooking today." A huddle where everyone describes their normal day is one people stop attending.

**Where the analogy breaks:** the canteen's huddle is purely operational. Your session has a second half a canteen does not need — reviewing each other's reasoning and architecture, where most of the learning happens.

### Picture it

```
  2 HOURS, EVERY DAY
  ┌────────────────────────────────────────────────────┐
  │ 0:00  STANDUP            10 min                    │
  │       3 sentences each · blockers named out loud   │
  ├────────────────────────────────────────────────────┤
  │ 0:10  UNBLOCK            20 min                    │
  │       whoever is stuck, in the room, screen shared │
  ├────────────────────────────────────────────────────┤
  │ 0:30  BUILD + REVIEW     70 min                    │
  │       briefs written · PRs reviewed · pairing      │
  │       mentors circulating                          │
  ├────────────────────────────────────────────────────┤
  │ 1:40  DEMO + DECIDE      20 min                    │
  │       show something working · record decisions    │
  │       tomorrow's one thing each                    │
  └────────────────────────────────────────────────────┘
```

The shape matters less than the two bookends: blockers surfaced at the start, something shown at the end. A useful update has exactly three parts, and longer is not better.

```
1. Yesterday I <finished / got to>  — checkable, not "worked on"
2. Today I will <one thing>          — one, not a list
3. I am blocked on <X> / not blocked — say it plainly
```

| Weak | Strong |
|---|---|
| "Still working on the frontend" | "Got the menu rendering from real data. Today: the order form. Not blocked." |
| "Doing the backend stuff" | "Orders now save to Supabase. Today: making staff-only actually staff-only. Blocked on how to check the role server-side." |
| "Nothing much, had classes" | "Nothing shipped — two labs. Today: one hour on the order form. Not blocked." |

The third weak example and the third strong one describe the same day; one is checkable and one invites nothing.

### Where it lives

A blocker raised well gets solved in the session. Bring four things: **what I am trying to do** (intent, not a description of your code), **what I expected**, **what actually happened** with the exact error text rather than a paraphrase, and **what I already tried**. That is the same shape as good feedback to an agent in M00 — precise problem description works on people and on models. **The two-hour rule:** two hours stuck with no progress makes it the group's problem, not a day.

Several threads in this program become real here. The chapter 1 role split shows up as who answers which question. Chapter 8's PR reviews happen with the author in the room, far faster than comment threads. The next chapter's decision log is written at the end, while the reason is fresh. And it answers the weakness of self-paced learning: text cannot notice that you have misunderstood something.

### Words to own

| Term | Plain meaning | Example |
|---|---|---|
| Standup | A short daily sync on progress and blockers | The first ten minutes |
| Blocker | Something stopping you progressing | "Can't check the role server-side" |
| Unblock | Getting someone moving again | Twenty minutes, screen shared |
| Pairing | Two people on one problem together | Debugging a deploy |
| Demo | Showing something working | Twenty minutes at the end |
| Timebox | A fixed duration you do not extend | Ten minutes for standup |
| Rubber ducking | Explaining a problem out loud and solving it mid-sentence | Happens constantly |
| Sunk cost | Reluctance to abandon effort already spent | Why day nine is harder than day one |

### Tradeoffs and gotchas

**Standup is for exceptions, not status.** Three people describing a whole day takes forty minutes and nobody listens to the third.

**Do not debug in the standup.** The moment someone says "oh, that's because…", the other person stops existing. Note it, take it to the unblock slot; that is what keeps ten minutes at ten minutes.

**On a day with nothing to show, come anyway and say the true thing** — that is usually the day you are stuck and have not said so. "Still working on it" for the fourth day is a blocker in disguise; "I read the caching chapter and wrote my brief, nothing is built yet" is legitimate. Learning is work, pretending is not.

**The quiet one needs asking directly.** In a group of three the person who talks least is often the most stuck, and "what are you blocked on?" beats "anything to add?", which is answerable with "no".

**Explaining it out loud solves a startling number of problems**, often halfway through the sentence. That is one reason the session works.

**Mentors are for judgement, not answers.** "What is the error on line 12" is for your agent; "is this the right architecture" is for a human, because the agent will say yes regardless.

### Verify it

Four checks on your own session habit, at the end of a week.

1. **Did every person name a blocker or say plainly they had none, every day?** Vague answers are the warning sign, not silence.
2. **Did anything get shown working?** Five days of discussion with nothing demonstrated means the session has become a meeting.
3. **Was anyone stuck more than a day on something they did not raise?** Ask directly. This is the failure the session exists to catch, and it hides.
4. **Were decisions written down?** The same question debated twice in one week means the answer was not recorded.

Red flag: you join with nothing you need from another human. That is usually the day you say "still working on it."

## Walkthrough
1. Agree a fixed daily time with your team and write it in `TEAM.md`. A time that moves is a time people miss.
2. Write your three-sentence standup before the session, not during it. Yesterday as something checkable, today as one thing, blocked or not blocked.
3. Check your sentences. If sentence one contains "worked on" rather than a finished or reached state, rewrite it. If sentence two contains a list, pick one item.
4. Say your blocker out loud even if it feels too small or too embarrassing. Apply the two-hour rule: two hours stuck with no progress makes it the group's problem.
5. When someone else raises a blocker, do not solve it in the standup. Say "take it to unblock" and move on. Protect the ten minutes.
6. In the unblock slot, whoever is stuck shares their screen and gives the four parts: intent, expected, actual with the exact error text, and what they already tried.
7. In the build slot, do the work that benefits from company: review each other's PRs with the author present, pair on anything one person has already failed at alone, and write briefs where someone can read them before you send them.
8. In the last twenty minutes, show something. A screen, a URL, a passing test, a diff. If nobody can show anything, say so out loud — that is information about the week.
9. Write the day's decisions down before you leave, while they are fresh. Who decided what, and why.
10. Each person states tomorrow's one thing. Not a plan — one thing.
11. At the end of the week, run the four checks from "Verify it" as a group and change one thing about how the session runs.

## Talk to AI
```prompt
I am about to join my team's daily session and I want to arrive useful rather than vague.

Here is my raw situation, written messily:
[dump everything - what you did, what you half-did, what is confusing you, what you avoided]

Help me turn this into:
1. A three-sentence standup: yesterday as something checkable (not "worked on"), today as exactly one thing, and blocked or not blocked stated plainly
2. If I am blocked, the four parts I should bring: what I am trying to do, what I expected, what actually happened with the exact error text, and what I have already tried
3. The one question in my situation that genuinely needs a human - architecture, tradeoffs, or whether my whole approach is wrong - as opposed to the questions I should just ask you directly
4. Anything in my dump that I appear to be avoiding or minimising, named plainly

Be direct. If my update sounds like "still working on it", tell me so and tell me what the honest version is.
```

## Stuck? Ask AI
```prompt
I have been stuck on the same problem for three days and I feel too embarrassed to raise it. Here is the problem: [describe it]. First, help me write the four-part blocker description I should say out loud tomorrow. Then tell me why waiting made it harder rather than easier.
```

```prompt
Explain the difference between a question I should ask an AI and a question I should save for a human mentor. Give me six example questions and ask me to sort them, without telling me the answers first.
```

```prompt
My standup updates are always vague. Here are my last three: [paste them]. Rewrite each as three useful sentences, then tell me the pattern in what I am leaving out.
```

## Quiz

### Q1 · L3
A student is stuck on the same deployment error for nine days without mentioning it. Why does waiting make it harder to raise rather than easier?
- (a) The error becomes technically more complex over time
- (b) Sunk cost — each day's silence makes the admission more embarrassing ✅
- (c) The team will have moved to different work by then
- (d) The error message changes as the platform updates
> Every individual day it seems like it will be solved in the next hour. The two-hour rule exists precisely to remove the judgement call.

### Q2 · L3
Which is a useful standup update?
- (a) "Still working on the frontend"
- (b) "Doing backend stuff today, lots to get through"
- (c) "Orders now save to the database. Today: staff-only checks. Blocked on verifying the role server-side." ✅
- (d) "Nothing much to report, I had classes yesterday"
> Yesterday as something checkable, today as one thing, blocker stated plainly. The others invite no help and cannot be verified.

### Q3 · L4
Someone raises a blocker in the standup and a teammate immediately starts diagnosing it. What should happen?
- (a) Let the discussion continue, since unblocking is the priority
- (b) Note it and move it to the unblock slot, to protect the ten minutes ✅
- (c) Ask everyone to research it before tomorrow's session
- (d) Move the whole standup to the end of the session
> Once technical discussion starts, the remaining people stop existing and ten minutes becomes forty. The unblock slot exists for exactly this.

### Q4 · L4
Which question is worth a human mentor's time rather than an AI's?
- (a) "What does this error on line 12 mean?"
- (b) "What is the syntax for a database migration?"
- (c) "Is this the right architecture for what we are trying to do?" ✅
- (d) "How do I check whether a string is empty?"
> The first three are faster and well answered by an agent. The fourth is where an agent will tend to agree with you regardless, so human judgement is the scarce resource.

### Q5 · L3
You have nothing built to show today. What is the right move?
- (a) Skip the session and catch up tomorrow
- (b) Say "still working on it" to keep things moving
- (c) Attend and say the true thing, including that nothing shipped and why ✅
- (d) Show work from two days ago so the session has content
> The day you have nothing is usually the day you are stuck and have not said so. "I read the chapter and wrote my brief" is legitimate; inventing progress is not.

## Go deeper
- [The daily standup, and the anti-patterns it falls into](https://martinfowler.com/articles/itsNotJustStandingUp.html)
- [Rubber duck debugging — why explaining out loud works](https://rubberduckdebugging.com/)
