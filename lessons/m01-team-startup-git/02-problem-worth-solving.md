---
id: m01-02-problem-worth-solving
module: m01-team-startup-git
slug: problem-worth-solving
title: "Finding a Problem Worth Solving, and Who Has It"
order: 2
duration_min: 25
is_free: false
mission: false
volatility: low
last_reviewed: 2026-11-16
---

## Key learnings
- Tell the difference between a problem and a solution looking for one
- Judge a problem by frequency, pain and reachability before building
- Write a problem statement that does not contain its own solution
- Recognise the four problem types that always disappoint students
- Choose a first product you can actually reach users for

## Lesson

### The pain

A group picks "an app for farmers to get better crop prices." It sounds important. It is important. They build for five weeks.

Then they try to find a farmer. None of them knows one. The nearest village is ninety minutes away. The farmers they eventually reach use WhatsApp voice notes and do not install apps from people they have not met. The group has a working product, a good demo, and zero users — and they will not get any, because the problem was real but the *users were unreachable*.

A second group picks "a better note-taking app." They can reach users instantly: themselves and everyone around them. But nobody switches, because the problem is mild and the existing options are free and excellent.

Both groups lost five weeks to a decision they made in forty minutes.

### The idea

**Pick a problem that happens often, hurts enough that people already work around it, and belongs to people you can physically reach this week.**

Three filters. Most student project ideas fail at least one, and the one they fail most is the third — which is also the one nobody checks, because it is not about the idea at all.

### The analogy

Think about why a canteen exists, and why a *particular* canteen succeeds.

The problem is not "students need food." That is a need, not a problem. The problem is that there are forty minutes between classes, the mess is closed, the nearest shop has a twenty-minute queue, and a student has ₹40. That is specific, it recurs daily, it hurts at a predictable time, and the people who have it are standing right there.

Notice that the problem statement contains no solution. It does not say "students need a canteen." Once you write "students need a canteen," you have stopped thinking — a vending machine, a pre-order window, a tiffin delivery to the classroom and a food truck are all answers to the same problem, and you have just excluded three of them.

**Where the analogy breaks:** a canteen's problem is visible from the queue. Most software problems are invisible until you ask, which is why the next chapter is about asking rather than guessing.

### Picture it

```
                    is it worth building?

  FREQUENCY  ──▶  how often does this happen to one person?
                  daily ████████  weekly ████  once a year ▏
                                                    │
  PAIN       ──▶  what do they do about it today?   │
                  pay someone ████████              │
                  tedious workaround ████           ├──▶ build
                  nothing, they shrug ▏             │
                                                    │
  REACH      ──▶  can I get 5 of them in a room     │
                  this week?                        │
                  yes, I know them ████████         │
                  yes, with effort ████             │
                  no ▏ ◀── stop here, whatever else is true
```

All three bars need length. A long REACH bar cannot rescue a short PAIN bar, but a short REACH bar kills the idea outright no matter how good the other two look.

### The reachability test nobody runs

Here is the question that would have saved the farming group five weeks:

**Can you get five of these people in a room, or on a call, within seven days — by name?**

Not "can I find farmers". Names. If you cannot list five actual humans who have this problem and whose number you could get this week, you cannot validate, you cannot test, and you cannot get your first users. You will build in a vacuum and find out at demo day.

This is why student products about students work, products about your parents' business work, and products about an industry none of you has met do not. Your unfair advantage at this stage is *access*, not insight. Spend it.

### Four problem types that always disappoint

Worth naming, because they are the four that get picked most often.

**1. The solution with a problem bolted on.** "We want to use blockchain — what for?" The technology came first. You will recognise this when the problem statement changes every time somebody questions the technology.

**2. The problem nobody has twice.** Wedding planning, choosing a college, buying a house. Enormous pain, almost no frequency. Hard to get feedback on, hard to retain anyone, and your users leave permanently the moment it works.

**3. The mild problem with free competition.** Note-taking, to-do lists, habit trackers. Real but small pain, and the incumbents are free and polished. People will say they like your version and keep using theirs.

**4. The problem that is actually a policy.** "Students should be able to get their marksheet online." The blocker is not software; it is an office that does not want to change. You will build the thing and discover you cannot get the data.

None of these are impossible. They are just bad *first* products for a six-month program where you need real users by month three.

### Where it lives

This chapter is the input to everything else. The problem statement you write here becomes the first line of your PRD in chapter 4, which becomes the master brief you give AI for the rest of the program. A vague problem produces a vague PRD, which produces exactly the generic output you learned to avoid in M00.

It also decides your architecture more than you would expect. "Daily, during a forty-minute window, for people on campus Wi-Fi" is already telling you about traffic spikes (M05), about mobile-first design (M03), and about whether you need push notifications (M08). A problem specified in time and place carries technical requirements inside it.

### Words to own

| Term | Plain meaning | Example |
|---|---|---|
| Problem statement | The pain, stated without a solution in it | "40 minutes, mess closed, ₹40" |
| Workaround | What people do today instead | Sending a friend to queue |
| Frequency | How often one person hits this | Daily, weekly, yearly |
| Reachability | Whether you can get to these people now | Five names this week |
| Unfair advantage | What you have that others do not | Access to your own campus |
| Willingness to pay | Whether they already spend on this | Paying extra for delivery |
| Substitute | The thing they use now, including "nothing" | A WhatsApp group |
| Beachhead | The small specific group you start with | One hostel, not all students |

### Tradeoffs and gotchas

**Small and specific beats big and vague, every time.** "Second-year students in one hostel who miss lunch because of lab timings" is a better starting point than "campus food." You can name them, reach them, and watch them use it. You can always widen later; you cannot narrow a product that was built for nobody in particular.

**The existing workaround is your real competition, and it is usually not software.** It is a WhatsApp group, a phone call, a spreadsheet, or a person who does it manually. These are strong competitors because they already work and cost nothing to adopt. If your product is not clearly better than the WhatsApp group, you have not found the problem yet.

**"Everyone has this problem" means you have not found the problem.** Products that serve everyone serve nobody specific enough to be chosen.

**Beware the problem you have already solved for yourself.** If you personally stopped caring about it, you will lose interest in month three. Six months is a long time to work on something you find mildly interesting.

**Do not fall in love with the problem statement either.** You will revise it after talking to five people in the next chapter, and if you do not revise it at all, that is a sign you were not listening rather than a sign you were right.

### Verify it

Before you commit the next six months, write your problem statement and run these five checks honestly.

1. **Does it contain a solution?** If the sentence includes an app, a platform, or a technology, rewrite it. The statement should leave several solutions possible.
2. **Can you name five people who have it?** Actual names. If not, this is not your first product, regardless of how good it is.
3. **What do they do today?** If the honest answer is "nothing, they just accept it", the pain is too mild. If the answer is "something tedious", you have something.
4. **How often?** Map it to daily, weekly, monthly or yearly. Below weekly, retention and feedback both become very hard in a six-month window.
5. **Could you be wrong?** Name the one assumption that, if false, kills the idea. That assumption is exactly what you will test in the next chapter.

A warning about using AI here: it will enthusiastically validate almost any idea you bring it, because agreement is its default posture — you saw this in M00. Asking "is this a good idea?" is nearly useless. Ask it to argue *against* the idea, to name who would not want it, and to tell you what would have to be true for it to fail. Then go and ask real people.

## Walkthrough
1. Each member of the team separately lists ten annoyances they have personally experienced in the last month. Not ideas — annoyances. Things that cost time, money or patience.
2. Add a second list: ten annoyances you have *watched* someone else have, where you know that person by name. Family business, hostel, classmates, a relative's shop.
3. Pool all thirty-plus items. Strike out anything where the real blocker is a policy or an office rather than software.
4. For each survivor, write one line: how often does this happen to one person, and what do they do about it today?
5. Strike anything that happens less than weekly, and anything where the honest answer to "what do they do today" is "nothing."
6. For each remaining item, write the names of five people who have this problem. Strike anything where you cannot fill all five lines.
7. You should now have between one and four candidates. If you have zero, your lists were about ideas rather than annoyances — go back to step one and be more specific.
8. For each candidate, write a problem statement with no solution in it. Then check: can you think of three different ways to solve it? If only one solution comes to mind, the solution is still hiding in your statement.
9. Pick one, and write the single assumption that would kill it if false.
10. Commit the file to your repository as `PROBLEM.md`. You will revise it next chapter after talking to people, and the history of those revisions is evidence that you listened.

## Talk to AI
```prompt
I am choosing a first product to build over six months with a team of three. I want you to stress-test my problem statement, not encourage me.

My problem statement is: [one or two sentences, with no solution in it]
Who has this problem: [be specific - which group, where]
Five people I can actually reach this week: [yes/no, and who]
What they do about it today: [the current workaround]
How often it happens to one person: [daily/weekly/monthly/yearly]

Now argue against me:
1. Restate my problem in your own words. If you had to guess a solution from it, what would you guess? If only one solution is possible, tell me the solution is still hidden in my statement.
2. Who in my target group would NOT want this solved, and why?
3. What is the existing workaround really competing on - and is it cost, habit, trust, or speed?
4. Which of the four bad first-product patterns does this most resemble: technology-first, once-in-a-lifetime, mild-problem-with-free-competition, or actually-a-policy-problem?
5. Name the single assumption that, if false, kills this idea.
6. Give me the three questions I should ask a real person this week to test that assumption - questions that do not mention my solution.

Be blunt. Do not tell me it is a good idea; tell me where it is weakest.
```

## Stuck? Ask AI
```prompt
I cannot tell whether my idea is a problem or a solution. Here it is: [paste it]. Rewrite it three times as a problem statement with no solution in it, each one slightly different in what it emphasises. Then ask me which one matches what I actually observed.
```

```prompt
Explain the difference between a need and a problem using three examples that are not about food. Then give me five statements and ask me to sort them into needs and problems, without telling me the answers first.
```

```prompt
My team has four candidate problems and we cannot choose. Here they are: [list them with frequency, workaround and reachability for each]. Do not pick for me. Instead, tell me what each choice would commit us to for six months, and what we would regret about each.
```

## Quiz

### Q1 · L3
Which of these is a problem statement rather than a solution in disguise?
- (a) Students need an app to pre-order canteen food
- (b) The campus needs a food delivery platform
- (c) Students have 40 minutes between classes, the mess is shut, and the nearest queue takes 20 minutes ✅
- (d) We should build a UPI-based payment system for the canteen
> A problem statement leaves several solutions open — a pre-order window, a vending machine, a food truck. The other three have already chosen one and stopped thinking.

### Q2 · L4
A team picks a genuinely painful, frequent problem, but none of them knows anyone who has it and the users are ninety minutes away. What happens?
- (a) It works, because the problem is real and frequent
- (b) They build in a vacuum, cannot validate, and get no users ✅
- (c) It works if they build a better product than competitors
- (d) It works as long as they interview users by phone
> Reachability is a hard filter, not a weighting. A short reach bar kills an idea however strong frequency and pain look, because you cannot test or acquire.

### Q3 · L4
Your users currently solve the problem with a WhatsApp group. Why is this a strong competitor?
- (a) WhatsApp has more engineers than you
- (b) It already works, everyone has it, and adopting it cost nothing ✅
- (c) It is encrypted, so it is more secure than your product
- (d) It is not a competitor, because it is not a dedicated product
> The real competition is usually the existing workaround, not another startup. Zero adoption cost and existing habit are hard to beat with a product that is merely neater.

### Q4 · L3
Which problem type is "a platform for planning weddings" for a six-month student project?
- (a) Technology-first — a solution with a problem bolted on
- (b) A problem nobody has twice — high pain, almost no frequency ✅
- (c) A mild problem with free competition
- (d) A policy problem disguised as a software problem
> The pain is enormous and the frequency is close to once per lifetime, which makes feedback, retention and iteration very hard inside six months.

### Q5 · L4
Why is asking an AI "is this a good idea?" nearly useless?
- (a) It has no information about markets
- (b) It defaults to agreement, so it will validate almost anything you bring it ✅
- (c) It cannot evaluate business ideas at all
- (d) Its training data is too old to judge current markets
> This is the agreement failure from M00. Useful prompts ask it to argue against the idea, name who would not want it, and say what must be true for it to fail.

## Go deeper
- [The Mom Test — why asking about your idea gets useless answers](http://momtestbook.com/)
- [Y Combinator: How to get startup ideas](https://www.ycombinator.com/library/8g-how-to-get-startup-ideas)
