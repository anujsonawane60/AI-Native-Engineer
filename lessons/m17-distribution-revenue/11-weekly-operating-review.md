---
id: m17-11-weekly-operating-review
module: m17-distribution-revenue
slug: weekly-operating-review
title: "Metrics That Matter and the Weekly Operating Review"
order: 11
duration_min: 30
is_free: false
mission: false
volatility: low
last_reviewed: 2026-11-16
---

## Key learnings
- Run a one-page scorecard: one north star, seven supporting numbers, one owner each
- Hold a 45-minute weekly review that produces exactly one decision with a date
- Put a money line and a cash line on the scorecard, not only usage
- Tell a signal from noise when your denominator is forty users
- Catch the review with thirty charts, no owners, no money and no decision

## Lesson

### The pain

A team has four dashboards, two spreadsheets and a Slack channel that posts a chart every morning. In the month-four review they are asked a simple question: what changed last week, and what are you doing about it? The three of them look at each other.

Everything was measured and nothing was reviewed. That month their activation rate had fallen from 61% to 52% and their inference cost per active user had risen from ₹37 to ₹44. Both numbers were on a dashboard the whole time. Nobody owned either, so nobody noticed either, and together they had quietly taken six points off gross margin.

Another team has the opposite problem. Their WhatsApp group is a stream of good news: "200 signups!!", "we crossed 1000 users!". Signups only goes up, so it is the number they talk about. Meanwhile week-four retention is 11% and has been for two months, and cash in hand is under five weeks of burn. Nobody is lying; they have chosen the metrics that feel like progress over the ones that describe their situation.

### The idea

**One north star, five to seven supporting numbers with one named owner each, reviewed on the same day every week by the same people, producing exactly one decision.**

Every part of that sentence is doing work. **One** north star, because two means neither wins an argument. **One owner per metric**, because a number everybody watches is a number nobody watches. **The same day every week**, because a review that happens when someone remembers does not happen. And **exactly one decision**, because a review that ends with a list of concerns ends with nothing.

The boundary with M16, stated once more: **M16 measures behaviour and builds the instrumentation; M17 turns it into money and into a weekly decision.** Your funnel, your activation definition, your retention curves and your experiment program live there. The scorecard reads from them and adds the three lines they do not contain — revenue, margin and cash.

### The analogy

The canteen owner's Sunday hour with a notebook.

Plates sold each day of the week. What he threw away. What ran out early and what nobody touched. Then one page of arithmetic, and then exactly one change for the coming week: stop making the ₹80 paneer on Tuesdays, because for six Tuesdays running he has thrown half of it away.

He does not change five things. He changes one, so that next Sunday he can tell whether it worked. And he does it every Sunday, which is why he knows things his competitor across the road is still guessing about.

**Where the analogy breaks:** his numbers are complete by nature. Every plate passed through his counter, so his data cannot have a hole in it. Your numbers are only whatever you instrumented — and a metric you never implemented reads as zero, and **zero looks exactly like a fact**. The team whose payment events were never tracked saw "0 payment failures" on a dashboard for three weeks. The canteen owner can never make that mistake; you can make it every week until someone checks.

### Picture it

```
  WEEK 41 SCORECARD                          owner

  NORTH STAR
  weekly active orderers      184   ▲ 171     Anand
  --------------------------------------------------
  new signups                  46   ▲  39     Priya
  activation rate, 7-day      52%   ▼ 61%     Priya
  week-4 retention            38%   ▬ 38%     Anand
  --------------------------------------------------
  MRR                     ₹10,900   ▲ ₹9,900  Ravi
  gross margin                61%   ▼ 67%     Ravi
  inference ₹ per active user  ₹44   ▲ ₹37    Ravi
  --------------------------------------------------
  support tickets              17   ▲   9     Priya
  top ticket tag          payment   was: confusion
  cash in hand            ₹14,200   ▼ ₹16,800 Ravi
  runway at this burn      5 weeks             Ravi
```

One page, eleven lines, last week's figure beside this week's, and a named person against every number.

Read it as a story rather than a list. Signups are up and activation is down, so the new users are worse or something broke for them. Inference cost per user is up and margin is down by six points. Support tickets have nearly doubled and the top tag has changed from confusion to payment. One sentence explains all of it: a launch brought a wave of users and the payment flow is failing for some of them. That sentence is what a review is for, and no single chart contains it.

### What goes on it, and what does not

Three groups, and most teams build only the first.

**Behaviour**, from M16: new signups, activation rate, retention at a fixed age, and your north star. **Money**, from chapters 4, 5 and 6: MRR, gross margin, and inference cost per active user. **Reality**: support tickets with the top tag, cash in hand, and runway.

The cash line is the one students leave off and the one that ends programs. Runway is cash divided by monthly burn, and it is the only number on the page that cannot be improved by working harder.

What does not go on it: anything that can only go up, anything nobody can act on, and anything whose denominator you cannot explain. Total registered users only rises, impressions cannot be acted on, and "engagement" has no denominator anyone agrees on.

Your **north star** is the single number that rises when a user genuinely gets what they came for. "Weekly active orderers" is a north star; "total signups" is a counter. M01 asked you to choose one; this is where it gets used weekly rather than appearing on a slide once.

**Leading versus lagging** matters as you read the page. Activation and support tickets move first; MRR and retention confirm later what the leading numbers already said. A team watching only lagging numbers hears about problems a month after they could have been fixed.

### The 45-minute review

Same day, same time, every week, all three of you, forty-five minutes, three parts.

**Fifteen minutes: read the numbers out loud.** Each owner reads their own lines, this week against last week. No discussion yet. Reading aloud sounds unnecessary and is the single mechanic that makes a review work, because a number you must say in front of two people is a number you have looked at.

**Fifteen minutes: one thing that surprised us.** Not every anomaly — the one that matters. "Activation fell nine points while signups rose" is worth the whole fifteen minutes.

**Fifteen minutes: one decision.** What we will do, who owns it, by when, written in the M01 decision log with the number that caused it. One, so that next week you can tell whether it worked.

Then stop. A ninety-minute review becomes a meeting people avoid, and that is worse than none, because it destroys the habit too.

### Signal and noise at forty users

With a hundred users, one person cancelling moves retention by a point. With forty, it moves it by two and a half. Most week-to-week movement at your scale is noise, and reacting to noise is how teams spend six months changing things at random.

Three practical rules. **Watch direction over three weeks, not level over one** — one bad week is weather and three is a trend. **Check the denominator before you believe the percentage**; an activation rate that "improved" because you changed what counts as activated has not improved at all, and that specific mistake is the most common way a scorecard lies to the people who built it. And **a number you cannot explain is a research task, not a decision** — write it down, assign it, and do not act on it this week.

### Where it lives

This chapter reads from almost everything. M16 supplies behaviour and instrumentation; chapter 4 supplies MRR and churn; chapter 5 supplies gross margin and payback; chapter 6 supplies inference cost per active user, which stays on the page permanently because it moves on its own; chapter 7 supplies the channel table; chapter 9 supplies the ticket count and top tag. M07's events and M13's incident records are the substrate.

Forwards, this is the document that makes M18 possible. The architecture walkthrough and the ten "what if" questions are answerable because you have twelve weeks of scorecards and a decision log. The team that cannot say what changed last week cannot tell the M18 story either, because it is the same story.

### Words to own

| Term | Plain meaning | Example |
|---|---|---|
| North star | The one number that rises with real value | Weekly active orderers |
| Vanity metric | A number that only goes up | Total signups |
| Leading indicator | Moves first, warns you | Activation, tickets |
| Lagging indicator | Confirms later | MRR, retention |
| Denominator | What a percentage is divided by | Users who signed up |
| Metric owner | The one person who reads it aloud | Priya |
| Scorecard | The one page, same shape every week | Eleven lines |
| Burn | Net cash leaving per month | ₹2,600 |
| Runway | Cash divided by burn | 5 weeks |
| Decision log | What you changed, why, and when | From M01 |
| Operating review | The weekly 45 minutes | Same day, same time |

### Tradeoffs and gotchas

**A dashboard is not a review.** Measuring is cheap and looking is not. The thirty-chart dashboard in the opening story contained both problems that cost the team their margin.

**Do not change the shape of the scorecard every week.** The value is in comparability. Add or remove a line rarely, and never redefine a metric without writing the date on the page.

**Do not let the review become a status update.** "I worked on the payment flow" is not a number. The first fifteen minutes is numbers only, and the temptation to narrate is strong.

**One decision means one.** Three means next week you cannot attribute any change to any of them, which is M16's experiment discipline applied to your own operations.

**Cash deserves its own line even when there is barely any.** ₹14,200 and five weeks is uncomfortable to read aloud, which is exactly why it must be read aloud.

**A missing metric reads as zero.** Before trusting any line, confirm the event behind it fires. Once a month, verify one metric end to end by doing the thing yourself and watching the number move.

### Verify it

**Red flag, the headline one: a scorecard with no money line.** Usage without revenue, margin and cash describes activity, not a company. Ask where MRR, gross margin and runway are.

**Red flag: a metric with no owner.** Put a name against each line. Any line with no name, or with "all of us", will not be read next week.

**Red flag: week-over-week comparison with a changed denominator.** Ask whether the definition of activation, active user or churn changed. An improvement caused by a redefinition is the most flattering and most useless number on any page.

**Red flag: a dashboard with thirty charts and a review with none.** The question is not what you can see; it is what you looked at and what you decided.

**Red flag: a north star that only goes up.** If no action can make it fall, it cannot tell you an action was wrong.

**Red flag: benchmarks quoted without a source.** "Good activation is 40%" is repeated everywhere and attached to nobody's market. Ask for the source and the market; the absence of one is your answer. Your own number from three weeks ago beats anyone's industry average.

**Red flag: a review that ends with concerns rather than a decision.** Check the decision log. If last week's entry is empty, the review did not happen even if the meeting did.

**Red flag: an AI that interprets your scorecard without asking about sample size.** With forty users most movement is noise, and a confident narrative built on a two-point shift is a story, not a finding.

## Walkthrough
1. Choose your north star in one line: the number that rises when a user genuinely gets what they came for. If it can only go up, it is a counter and you need a different one.
2. Pick seven supporting lines: three behaviour from your M16 instrumentation, three money from chapters 4, 5 and 6, and one reality line — tickets, cash or both.
3. Put one name against every line. Three people, eleven lines, no line without an owner and no line owned by "all of us".
4. Build the page in whatever tool you already have — a spreadsheet is correct — with this week and last week side by side, in a shape you will not change.
5. For each percentage on the page, write the denominator next to its definition in a separate note, with today's date. This note is what prevents the changed-denominator lie.
6. Verify three lines end to end: do the thing yourself in the product and watch the number move. Any line that does not move is reporting zero as a fact.
7. Add the cash line and compute runway as cash divided by last month's net burn. Write it on the page even if it is uncomfortable.
8. Book the review: same day, same time, forty-five minutes, every week, in all three calendars. Pick a day that survives exam weeks.
9. Run the first one in three parts: owners read their lines aloud, fifteen minutes on the one surprise, fifteen minutes to produce exactly one decision with an owner and a date.
10. Write the decision in your M01 decision log with the number that caused it, and start next week's review by checking whether it worked.

## Talk to AI
```prompt
I need a weekly operating scorecard and review for my startup, and I need you to be sceptical about my numbers rather than enthusiastic.

My product and north star candidate: [the number, and why it rises with real value]
My instrumented behaviour metrics today, honestly: [what events actually fire - say if some are not implemented]
My current figures: signups [X], activation rate and its exact definition [X], retention at a stated age [X]
My money figures: MRR [X], gross margin [X], inference cost per active user [X from chapter 6]
Support tickets last week and the top tag: [X]
Cash in hand and last month's net burn: [both]
Total paying users: [X - this is my sample size]
My team: three people. Names: [list]

Do all of this, in order:

1. Judge my north star candidate. Tell me whether it can fall when we do something wrong. If it cannot, say it is a counter and propose two alternatives from the metrics I actually have.

2. Build my one-page scorecard with no more than eleven lines, grouped into behaviour, money and reality, with a column for this week and last week. Assign each line to one of my three named people and explain why that pairing.

3. Tell me which of my lines are leading and which are lagging, and what it would mean if I only ever watched the lagging ones.

4. For every percentage on my page, ask me to state its denominator. Then tell me which of my definitions is most likely to drift and what that drift would look like on the page.

5. Given my sample size, tell me how large a week-on-week move has to be before it is worth discussing at all. Be concrete about my numbers.

6. Read my current figures as a story. Tell me the single most likely explanation that accounts for more than one line moving together, and what I would check to confirm or kill it.

7. Tell me which lines I am missing that would most change my understanding of my own situation.

8. Give me the 45-minute agenda in three parts, with the specific instruction for each part, and tell me the two ways this meeting usually degrades.

9. Tell me what benchmark figures you are tempted to quote me for activation, churn or margin - and then tell me why I should not use them, and what I should compare against instead.

10. Finally, write the template for a decision-log entry that records what we changed, the number that caused it, who owns it, and when we will check.
```

## Stuck? Ask AI
```prompt
Explain the difference between a north star metric and a vanity metric using five examples from Indian consumer products. Then give me six candidate metrics for my product - [describe it] - and ask me which could fall if we made a mistake, without telling me the answers first.
```

```prompt
My activation rate improved from 48% to 59% and I changed the definition of activation three weeks ago. Explain exactly why I cannot compare those two numbers, what I should do now, and what rule would have prevented this. Then tell me how to record metric definitions so this never happens again.
```

```prompt
I have 40 paying users and my week-4 retention moved from 38% to 33%. Tell me whether that is a signal or noise, show me the arithmetic of how much one user moves the number, and tell me what I would need to see before acting on it.
```

## Quiz

### Q1 · L3
A team's scorecard shows signups up, activation down nine points, inference cost per user up, margin down six points, and support tickets doubled with the top tag changing to payment. What is the single most likely explanation?
- (a) The product has become harder to use for existing users
- (b) A launch brought a wave of users and the payment flow is failing for some of them ✅
- (c) The model provider raised prices
- (d) Retention is improving, so costs are rising naturally
> The value of a one-page scorecard is that several lines move together and tell one story. No individual chart contains that sentence, which is why reviewing beats measuring.

### Q2 · L4
Which of these disqualifies a metric from being your north star?
- (a) It is measured weekly rather than daily
- (b) It cannot fall, however badly you perform ✅
- (c) It is a percentage rather than a count
- (d) It depends on instrumentation you built yourself
> A north star must be able to punish you. Total signups and cumulative users only rise, so they can never tell you that a decision was wrong — which is the entire purpose.

### Q3 · L4
Your activation rate rose from 48% to 59% in the same week you changed what counts as activated. What have you learned?
- (a) That activation genuinely improved by eleven points
- (b) Nothing about activation — the denominator changed, so the two numbers are not comparable ✅
- (c) That the new definition is a better one
- (d) That the previous definition was understating performance
> A redefinition dressed as an improvement is the most flattering and least useful number on any scorecard. Metric definitions need a written date, and comparisons must be against the same definition.

### Q4 · L3
Why does the weekly review end with exactly one decision rather than a list?
- (a) Student teams lack the capacity for more
- (b) So that next week you can tell whether that change worked ✅
- (c) To keep the meeting under forty-five minutes
- (d) Because decisions require unanimous agreement
> Three simultaneous changes make attribution impossible, which is the experiment discipline from M16 applied to running your own company. One change, one owner, one check next week.

### Q5 · L5
An AI reads your scorecard — forty paying users — and produces a confident narrative explaining a two-point retention move. What is wrong?
- (a) It lacks access to your raw event data
- (b) At forty users a two-point move is roughly one person, so the narrative is a story rather than a finding ✅
- (c) It cannot know your product's category benchmarks
- (d) It should have used a longer time window
> Sample size governs what any number can mean. A model asked to interpret will interpret, so the question to force is how large a move has to be at your scale before it is worth discussing.

## Go deeper
- [Amplitude: choosing a north star metric and its inputs](https://amplitude.com/north-star)
- [Andy Grove's operating cadence, summarised from High Output Management](https://www.goodreads.com/book/show/324750.High_Output_Management)
