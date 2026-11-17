---
id: m01-03-validating-an-idea
module: m01-team-startup-git
slug: validating-an-idea
title: "Users, Personas and Validating an Idea in One Week"
order: 3
duration_min: 25
is_free: false
mission: true
volatility: low
last_reviewed: 2026-11-16
---

## Key learnings
- Ask about a person's past behaviour instead of their future intentions
- Avoid the four questions that produce useless answers
- Run five real conversations in a week and extract signal from them
- Write a persona from evidence rather than imagination
- Decide to continue, change or drop the idea, on the record

## Lesson

### The pain

A team describes their idea to twelve classmates. Eleven say it sounds useful, two say they would definitely use it. They record this as validation and build for two months. At launch nobody signs up — including the two who said "definitely."

Nobody lied. They were asked a question humans cannot answer accurately: *would you use this?* Twelve conversations produced zero information and left the team more confident, which is worse than nothing.

### The idea

**Ask what people did last week, not what they would do next month.**

Past behaviour is a checkable fact; future intention is a guess delivered under social pressure. Five honest conversations about the past beat fifty people agreeing with your pitch.

### The analogy

Would a pre-order window work at the canteen? The useless question is "if there was a way to skip the queue, would you use it?" — everyone says yes, because you described a world with no cost and they are being kind. The useful one is "tell me about the last time you skipped lunch. What did you do instead?"

Now you collect facts: three of five skipped lunch last week, one pays ₹20 to a senior who brings food up, and the real constraint is not the queue — the mess shuts at 1:30 and their lab ends at 1:25. That last fact changes the product, and no question about your idea would have found it.

| Do not ask | Because | Ask instead |
|---|---|---|
| "Would you use this?" | Predicts the future; invites politeness | "When did you last have this problem?" |
| "Do you think this is a good idea?" | Asks for a compliment | "What do you do about it today?" |
| "How much would you pay?" | Hypothetical money is imaginary | "What do you currently spend on this?" |
| "What features would you want?" | Makes them design for you | "Walk me through the last time it happened." |

Every good question points at a past event. One hard rule: **do not describe your solution until the end.** The moment they know what you built, they start being supportive instead of informative.

**Where the analogy breaks:** at a canteen you can just stand and watch the queue, which beats any interview. Where you can observe instead of asking, observe.

### Picture it

```
  WEEK ONE                                      signal
  ┌──────────────────────────────────────┐
  │ Mon  write 6 questions, list 5 names │
  │ Tue  conversation 1 ──────────────▶  │  "mess shuts at 1:30"
  │ Wed  conversation 2, 3 ───────────▶  │  "pays a senior ₹20"
  │      revise questions ◀──────────────┤  (new thing to probe)
  │ Thu  conversation 4 ──────────────▶  │  "no, never happens to me"
  │ Fri  conversation 5 ──────────────▶  │  "WhatsApp group of 9"
  │ Sat  count the facts                 │
  │ Sun  decide: continue / change / drop│
  └──────────────────────────────────────┘
                     │
         ┌───────────┼───────────┐
         ▼           ▼           ▼
     CONTINUE     CHANGE       DROP
   3+ of 5 had   problem is   nobody had it
   it last week  different     in a month
```

The questions change mid-week: conversation two exists to improve conversation four. You get no statistics, but three kinds of signal. **Strong signal** is a cost they already pay — "I pay a senior ₹20" proves the pain exceeds the price of a workaround. **Weak signal** is agreement and feature suggestions. **The best signal is that you were wrong**, which feels like failure and is the entire return on the week.

### Where it lives

Write one persona from the five conversations, where **every line traces to something somebody said.** Not "Priya is tech-savvy and values convenience", which is horoscope writing, but "Priya's lab ends at 1:25, the mess shuts at 1:30, she skipped lunch twice last week, has ₹40–60 a day and does not use delivery apps because the minimum order is ₹150." Each fact constrains the product: a price ceiling, a time window, a competitor.

That persona becomes the CONTEXT section of your PRD in chapter 4, the master brief you hand AI from here on. It shapes your design in M03, your data model in M04 and whether you need offline support in M08. It is also your baseline: in M16 you measure whether the product changed it.

### Words to own

| Term | Plain meaning | Example |
|---|---|---|
| Validation | Evidence from reality that the problem exists | Three of five had it last week |
| Persona | A short, evidence-based description of your user | Built from the five conversations |
| Leading question | One that suggests its own answer | "Wouldn't it be useful if…" |
| Signal | Information that changes what you do | "I pay a senior ₹20" |
| Noise | Information that feels good and changes nothing | "Sounds great" |
| Workaround | What they built or arranged themselves | A WhatsApp group of nine |
| Baseline | The current behaviour, measured before you build | Skipped lunch twice last week |
| Pivot | Changing the problem or the user based on evidence | Timing, not queues |

### Tradeoffs and gotchas

**Five is enough to decide, not enough to be sure.** You want a clear signal to continue, change or drop. All five saying it never happens is enough; three describing a workaround is enough.

**Interview strangers, not only friends.** Get two of the five from outside your circle, or you gathered encouragement rather than evidence.

**Write notes during, not after, with exact quotes.** "I pay a senior ₹20 on Thursdays" is evidence you can revisit in month four; "seemed keen" is a memory you will bend in your own favour.

**Beware your own excitement.** Every minute explaining your solution is a minute of data lost.

**Do not run these interviews with AI.** It produces a plausible, agreeable, fictional user — precisely the failure you are avoiding. Use it to critique questions, role-play practice and find patterns in your notes; never as the respondent.

### Verify it

Five checks before you accept your validation.

1. **Does every persona line trace to a quote?** Anything else is imagination that will mislead you for six months.
2. **Did at least two of five describe a workaround?** The strongest evidence short of money.
3. **Did you learn something that surprised you?** If not, you were probably pitching.
4. **Can you state the decision in one sentence with a number?** "Three of five skipped lunch last week, so we continue with a timing focus."
5. **Did you talk to anyone who does not care about you?** If not, add one before deciding.

## Walkthrough
1. Write six questions, every one pointing at a past event. Start each with "when was the last time", "tell me about", or "what did you do when". Not one of them may mention your solution.
2. List five people by name, with at least two from outside your immediate friend group. Message them asking for fifteen minutes about a problem, not about your idea.
3. Run conversation one. Take written notes with exact quotes. Resist explaining your solution. When you catch yourself talking for more than twenty seconds, stop and ask another question.
4. Immediately afterwards, write one line: the single most surprising thing you heard.
5. Run conversations two and three. After the third, revise your questions. Anything that produced only agreement gets cut; anything surprising gets a follow-up question.
6. Run conversations four and five with the revised questions. These two will be noticeably better than the first three — that is the point of revising mid-week.
7. Count the facts. How many of five had the problem in the last week? How many had it in the last month? How many described a workaround? How many spend money or favours on it today?
8. Write the persona. One paragraph, every sentence traceable to a quote. Include their money, their time window and their current substitute.
9. Make the decision and write it down with the number that drove it: continue, change the problem, change the user, or drop it.
10. Update `PROBLEM.md` in your repository to reflect what you learned, and commit it. The revision itself is the evidence that you listened.

## Talk to AI
```prompt
I am preparing five user conversations to validate a problem. I need help preparing and practising, not help inventing users.

My problem statement: [paste it]
Who I am talking to: [specific group]
My draft questions: [list them]

First, review my questions:
1. Which of mine are leading, hypothetical, or invite politeness? Quote each one and say what is wrong with it.
2. Rewrite each weak question so it points at a specific past event.
3. Add two questions I have not thought of that would test my core assumption.
4. Tell me which single question is most likely to produce the surprise that changes my product.

Then role-play once, so I can practise. You be a person from my target group who has only a mild version of this problem and is slightly impatient. Answer briefly and realistically, the way a real person would - do not be helpful, do not suggest features, and do not compliment my idea. Ask me your first answer and wait for my next question.

Important: you are practice, not data. I will be talking to five real people regardless of how this goes.
```

## Mission

**Objective.** Five real conversations in seven days, an evidence-based persona where every line traces to a quote, and a written continue / change / drop decision with a number in it.

**Steps.**
1. **Frame** — write the one assumption that would kill your idea if false. That is what the week tests.
2. **Prepare** — six questions, all pointing at past events, none mentioning your solution. Have AI critique them, then rewrite.
3. **Reach** — list five people by name, at least two outside your friend group. Ask for fifteen minutes about a problem, not about an idea.
4. **Listen** — run all five. Written notes with exact quotes. Revise your questions after the third.
5. **Count** — how many had it last week, last month, and how many have built a workaround.
6. **Write** — one persona, every sentence traceable to a quote, including money, time window and current substitute.
7. **Decide** — continue, change the problem, change the user, or drop. State the number that drove it.
8. **Verify** — run the five checks from "Verify it" against your own work before you submit.

**Upload.**
- Your six questions, before and after the AI critique → text or screenshot
- Your interview notes with real quotes, five conversations, names initialised for privacy → file or screenshot
- Your persona paragraph, with each sentence's supporting quote beside it
- Your decision in one sentence, with the number that drove it
- The commit history of `PROBLEM.md` showing it changed after the interviews → screenshot of the log
- Your AI conversation critiquing the questions and role-playing practice → share link in `ai_chat_url`
- Reflection, 150 words max → the most surprising thing you heard, and one place you caught yourself pitching instead of listening

**Self-check.** Tick every box before you submit.
- [ ] I spoke to five real people, at least two from outside my friend group
- [ ] Not one of my questions asked whether they would use my solution
- [ ] I have exact quotes written down, not summaries of impressions
- [ ] Every line of my persona traces to a specific quote
- [ ] My persona includes their money, their time window and their current substitute
- [ ] I can say how many of five had the problem in the last week
- [ ] I revised my questions partway through the week
- [ ] I can name one thing I was wrong about
- [ ] My decision has a number in it, not just a direction
- [ ] `PROBLEM.md` has been updated and committed since the interviews

**Stretch.** Observe instead of asking. Spend thirty minutes watching the behaviour in question — the queue, the shop, the office — and count something. Write 100 words on how what you saw differs from what people told you.

## Stuck? Ask AI
```prompt
Here are my interview notes from five conversations: [paste them]. Do not tell me whether my idea is good. Instead: what pattern appears in at least three of the five? What did I fail to follow up on? Which of my notes are facts and which are opinions I recorded as facts?
```

```prompt
Everyone I talked to was encouraging and I learned nothing. Here is one of my conversations: [paste it]. Show me exactly where the conversation went wrong, and rewrite three of my questions so they would have produced facts instead of agreement.
```

```prompt
Explain why "would you use this?" produces unreliable answers, using an example from outside software. Then give me eight questions and ask me to sort them into useful and useless, without telling me the answers first.
```

## Quiz

### Q1 · L3
Eleven of twelve classmates say your idea sounds useful and two say they would definitely use it. What have you learned?
- (a) The problem is validated by a clear majority
- (b) Essentially nothing — they predicted future behaviour under social pressure ✅
- (c) That two users are enough to start building
- (d) That the idea needs only minor changes
> People cannot accurately predict their own future behaviour and tend to be kind to someone visibly excited. The dangerous part is that the team felt more confident afterwards.

### Q2 · L4
Which sentence from an interview is the strongest evidence that a problem is real?
- (a) "That sounds really useful, you should build it"
- (b) "I'd definitely use that every day"
- (c) "I pay a senior ₹20 on Thursdays to bring food up" ✅
- (d) "You should add a feature for group orders"
> An existing workaround that costs money, time or social capital proves the pain exceeds the price of working around it. Agreement and feature suggestions prove nothing.

### Q3 · L4
Your five interviews leave your understanding of the problem completely unchanged. What is the most likely explanation?
- (a) Your original hypothesis was correct
- (b) You were pitching rather than listening ✅
- (c) Five conversations is too small a sample to learn from
- (d) You chose users who were too similar to each other
> Being wrong about something is the main return on the week. No change at all usually means the solution was described early and the conversation became supportive rather than informative.

### Q4 · L3
Which persona line is usable?
- (a) Priya is tech-savvy and values convenience
- (b) Priya is a typical second-year student who likes saving time
- (c) Priya's lab ends at 1:25, the mess shuts at 1:30, and she skipped lunch twice last week ✅
- (d) Priya is frustrated by long queues and wants a better experience
> Every line of a persona must trace to a quote. The usable version supplies a time window and a measured baseline; the others could describe anyone.

### Q5 · L4
What is AI legitimately useful for during this week?
- (a) Acting as one of the five users, to save time
- (b) Critiquing your questions, role-playing practice, and finding patterns in your notes ✅
- (c) Deciding whether to continue or drop the idea
- (d) Generating the persona from your problem statement
> It will produce a plausible, agreeable, fictional user — exactly the failure being avoided. Preparation and analysis are its real value here.

## Go deeper
- [The Mom Test — how to talk to customers](http://momtestbook.com/)
- [Interviewing Users, Steve Portigal — summary of the core method](https://www.portigal.com/interviewing-users/)
