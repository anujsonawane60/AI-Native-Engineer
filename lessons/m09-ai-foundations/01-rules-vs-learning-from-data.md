---
id: m09-01-rules-vs-learning
module: m09-ai-foundations
slug: rules-vs-learning-from-data
title: "Rules vs Learning From Data: Features, Labels, Training, Inference"
order: 1
duration_min: 25
is_free: false
mission: false
volatility: low
last_reviewed: 2026-11-16
---

## Key learnings
- Tell the difference between a rule you wrote and a rule a program found
- Name the four words that describe every machine-learning system: features, labels, training, inference
- Decide when a plain rule is the better engineering choice
- Say where the training happens and where the prediction happens in your own architecture
- Catch an AI that proposes machine learning for a problem three if-statements would solve

## Lesson

### The pain

Your group adds a feature to the canteen app: tell the kitchen which dishes will sell out before 1 pm, so they cook more of those. Someone writes the rule. If it is Monday and the dish is biryani, it will sell out. Then: unless it rains, because half the campus stays in the hostel. Then: unless exams are on, because people eat at odd hours. Then: unless there is a cricket match, because everyone leaves campus by noon.

Four weeks later there are forty-one rules in one file, two of them contradict each other, and nobody can say which rule produced any particular prediction. The kitchen has stopped trusting it. Every new rule fixes last Tuesday and quietly breaks a Thursday nobody checked.

The team did not write bad rules. The team attempted something that cannot be done: writing down, by hand, a rule with thirty interacting inputs that also changes every semester. The forty-one rules are not a failure of effort. They are the symptom that tells you which kind of problem you have.

### The idea

**Instead of writing the rule, you show a program thousands of past examples and let it find the rule itself.**

You hand it the things it is allowed to look at — day of the week, dish, weather, how many portions were cooked, whether exams are on — and the answer you already know for each of those past days: sold out, or not sold out. A procedure called **training** searches for a setting of numbers that reproduces those known answers as closely as it can. What comes out is a **model**: a file of numbers, plus a small amount of code that knows how to use them. Running that file on a day you have never seen is **inference**.

Everything else in this module — deep learning, transformers, the model behind your chat assistant — is this same shape, at a scale that makes it feel like something else. It is not something else.

### The analogy

There are two ways to train a new cashier at the canteen counter.

The first is a laminated sheet. Thali is ₹60. Extra curd adds ₹10. Green-card students get 10% off; staff do not. The sheet is short, anyone can read it, and when a price looks wrong you can point at the exact line that produced it. That is a rule-based system, and for pricing it is clearly correct.

The second is two weeks of standing beside the old cashier. Nobody writes anything down. The new cashier watches six thousand transactions and absorbs the pattern — including the parts nobody ever wrote on a sheet, like the fact that the chemistry department settles monthly, or that the Friday crowd wants change counted out before they ask. After a fortnight they are faster and better than the sheet.

Which approach is right depends on one question: is the rule short enough for a human to write down? Pricing is. "Will biryani sell out today" is not.

**Where the analogy breaks:** the shadowing cashier can say "I am not sure about the green card, let me ask". A trained model cannot. It has no idea which situations were absent from those two weeks, and when a customer type it never saw walks up, it produces a confident price anyway. This is the same property you met in M00 as the first AI failure mode, and it starts here, at the very bottom of the stack.

### Picture it

```
RULES: a human writes the logic

  today's data ──▶ ┌───────────────┐ ──▶ answer
                   │ rules I wrote │
                   └───────────────┘

LEARNING FROM DATA: a program finds the logic

  TRAINING  (once, offline, slow, needs a GPU)

  features + labels ──▶ ┌──────────┐ ──▶ model file
  from past examples    │ training │     (just numbers)
                        └──────────┘

  INFERENCE  (every request, fast, costs money per call)

  today's features ──▶ ┌────────────┐ ──▶ "sells out"
                       │ model file │
                       └────────────┘
```

In the rules path a person writes the logic and the computer applies it; in the learning path the computer finds the logic once during training and then applies it on every request.

### The four words, on a real table

Here are six rows of the canteen's own history. This is a **dataset**.

| Day | Dish | Rain | Exams | Portions cooked | Sold out by 1 pm? |
|---|---|---|---|---|---|
| Mon | Biryani | No | No | 120 | Yes |
| Tue | Pongal | No | No | 90 | No |
| Wed | Biryani | Yes | No | 120 | No |
| Thu | Thali | No | Yes | 150 | No |
| Fri | Biryani | No | No | 160 | Yes |
| Sat | Pongal | No | No | 60 | Yes |

The first five columns are **features**: what the model is allowed to see. The last column is the **label**: the answer you already know, because it already happened. One row is one **example**. Training is the search for numbers that turn those five features into that last column. Inference is handing it next Monday's five features and reading what comes out.

Notice two things about this table, because they are the two things that actually decide whether your feature works. First, somebody had to record that last column, honestly, for hundreds of days. That is unglamorous human work and it is the expensive part of every machine-learning project. Second, "portions cooked" matters enormously — 60 portions of pongal sold out and 90 did not — and if you had forgotten to record it, no amount of clever modelling would have recovered it. The features you chose are a bigger lever than the algorithm you chose.

### Where it lives

Put this on the architecture map from M00 chapter 1, because it sits in two different places and people confuse them.

**Training lives outside your app.** It runs on a free Colab or Kaggle notebook, on their GPU, for minutes or hours, once. It reads a file of past examples and writes out a model file. No user ever waits for it. Nothing about it is in your deployed product.

**Inference lives inside your app**, as a hop. Either the model file is loaded by your own server and the prediction is an ordinary function call, or you call somebody else's model over the network — which, exactly as M00 chapter 2 showed, means a second or two of latency and a cost per call. From your server's point of view a model is just another dependency that can be slow or down.

**And the practice of training is M14.** This module is the concepts half. Actual training loops you run and debug, fine-tuning a model on your own data, and shrinking a model to fit a small GPU are the Rank-2 practicum in M14. Here you learn what the words mean and where the costs are, so that when you direct an AI you can tell a sound plan from a confident one.

### Words to own

| Term | Plain meaning | Example |
|---|---|---|
| Feature | An input the model is allowed to see | Day of the week, rain |
| Label | The known answer for a past example | "Sold out: Yes" |
| Example / row | One past case, features plus label | Last Monday |
| Dataset | All the examples together | 400 days of canteen history |
| Training | The search for numbers that fit the examples | A Colab notebook, 6 minutes |
| Model | The resulting file of numbers plus how to use it | 2 MB on disk |
| Parameter / weight | One of the numbers training adjusted | Millions of them in a big model |
| Inference | Using the model on new input | Monday's prediction |
| Prediction | What comes out, often with a confidence | "Yes, 0.81" |
| Ground truth | What actually happened, recorded honestly | The kitchen's 1 pm tally |

### Tradeoffs and gotchas

**Rules are better than you think, and they are free.** If the logic fits on an index card, write it on an index card. A rule is readable, testable, instant, costs nothing per request, and when it is wrong you can see why in four seconds. Reach for learning from data when the rule has many interacting inputs, when it changes over time, or when nobody can state it out loud. "We could use AI for this" is not a reason; "nobody can write this rule down" is.

**Labels are the expensive part, and nobody budgets for them.** A model that predicts sell-outs needs somebody to have written down, for three hundred days, whether each dish sold out. If that record does not exist, your project is not a modelling project this month — it is a data-collection project. Teams discover this in week three, every time.

**Garbage in, confident garbage out.** If the kitchen recorded "sold out" loosely — sometimes meaning finished, sometimes meaning nearly finished — the model learns that inconsistency faithfully and reports it with the same confidence as everything else.

**A model is a photograph of the past.** The canteen raises prices, a new hostel opens, a food stall opens across the road, and a model trained on last year's pattern keeps answering as if none of that happened. This is **drift**, and the only defence is measuring the model against reality on a schedule rather than trusting the number you got on launch day.

**You are now operating two systems.** Code and data. The data pipeline, the retraining, the monitoring and the question "which version of the model produced this answer" are all real work that did not exist when you had forty-one rules. Sometimes forty-one rules were the right answer.

### Verify it

When you ask an AI to help with a prediction feature, four red flags tell you to stop and push back.

**It proposes machine learning for a three-rule problem.** Ask it to write the rule-based version first and tell you specifically why that version is inadequate. If it cannot name a case the rule gets wrong, you do not have a modelling problem.

**It never mentions where the labels come from.** A plan that goes straight to the choice of algorithm has skipped the only genuinely hard part. Ask directly: which column is the label, who recorded it, how many rows exist, and over what period.

**It reports how well the model did on the examples it trained on.** That number is meaningless and always flattering. Chapter 3 is entirely about this, and it is the single most common dishonest metric in generated machine-learning code.

**It proposes "fine-tune a model" for a problem that is really about fresh facts.** Needing today's menu in an answer is a data-retrieval problem, not a training problem. This distinction returns in chapter 11 and again in M10, and it is the most expensive confusion in the whole subject.

And one check for yourself rather than for the AI: say out loud which columns are features and which is the label. If you cannot, nobody can build this yet, including you.

## Walkthrough
1. Take the canteen sell-out problem and write the rule-based version by hand. Give yourself exactly five rules, no more, and write them as plain if-then sentences.
2. Now invent three realistic days that break your five rules — a rainy exam Monday, a match day, a day the kitchen cooked double. For each, say which rule fails and why.
3. Write down the rule you would need to add for each. Notice whether your new rules contradict any of the first five. Count the contradictions.
4. Draw the same problem as a table instead. Write the column headers you would need, mark each one feature or label, and be honest about which columns the canteen does not actually record today.
5. Ask an AI to critique your column list: which feature is probably the most predictive, which is probably useless, and which one you have forgotten. Do not let it write any code.
6. Estimate the labelling work. How many days of history would someone have to write down by hand before training is worth attempting? Put a number on it and a rough number of hours.
7. Decide, in one written sentence, whether this problem should be rules or learned from data at your canteen's scale. Either answer is acceptable; an undefended answer is not.
8. Now place it on your architecture diagram. Draw where training would run, where the model file would live, and where in a request the prediction would be made. Mark the hop that costs latency.
9. Finally, write one sentence describing what happens to this model in six months if nobody retrains it, and what measurement would tell you it had gone stale.

## Talk to AI
```prompt
I am learning the difference between rule-based logic and learning from data, and I want to be argued with rather than handed an answer. Do not write any code.

My problem: predicting which canteen dishes will sell out before 1 pm, so the kitchen can cook more of them. The canteen serves about 600 students a day, 8 dishes, and has been running for years but keeps almost no written records.

Please do four things, in this order:

1. Write the best possible RULE-BASED version using at most five if-then rules in plain English. Then name three realistic situations where your own five rules give the wrong answer.

2. Reframe the same problem as a table of features and one label. List the feature columns you would want, and for each one say how a real canteen would actually record it and how likely it is to be missing.

3. Tell me honestly how many labelled days you would want before a learned model beats my five rules, and what you would do in the months before that data exists.

4. Give me the strongest argument AGAINST using machine learning here at all, as if you had to defend that position.

Finally, ask me three questions whose answers would change your recommendation, and wait for me to answer them.
```

## Stuck? Ask AI
```prompt
Explain features, labels, training and inference to me using an example from a vegetable market, not a canteen or a school. Then give me six short descriptions of data columns, one at a time, and ask me whether each is a feature or a label. Do not tell me the answers before I guess.
```

```prompt
I keep mixing up training and inference. Explain the difference in terms of when each one happens, how long it takes, what it costs, who waits for it, and where it runs. Then give me four situations and ask me which one is happening in each.
```

```prompt
Give me five real problems a small student app might have. For each, argue first that it should be solved with hand-written rules, then argue that it should be solved by learning from data. Then tell me which argument you actually find stronger and why. Be blunt about the cases where machine learning would be a waste of time.
```

## Quiz

### Q1 · L2
In a table of past canteen days, which column is the label?
- (a) The day of the week
- (b) The number of portions cooked
- (c) Whether the dish actually sold out ✅
- (d) The price of the dish
> The label is the known answer you want the model to reproduce. Everything else the model is allowed to look at is a feature.

### Q2 · L3
Your five hand-written rules now number forty-one, two of them contradict each other, and each new rule breaks an older case. What is this telling you?
- (a) The rules were written badly and need refactoring
- (b) The rule has too many interacting inputs to write by hand ✅
- (c) The canteen's data is wrong
- (d) You need a faster server
> Rule count exploding with contradictions is the classic signal that the logic is not humanly writable. That is precisely the condition under which learning from data earns its cost.

### Q3 · L4
A teammate says "let's just use AI" for deciding whether a student gets the 10% green-card discount. What is the right response?
- (a) Agree, because a model will be more accurate
- (b) Agree, but only with a large dataset
- (c) Refuse: the rule fits on an index card, so write the rule ✅
- (d) Refuse, because discounts can never be automated
> A short, stable, stateable rule should be code. It is instant, free per request, auditable, and when it is wrong you can see exactly which line did it.

### Q4 · L4
An AI gives you a plan for the sell-out predictor that names an algorithm, a library and a notebook structure, but never mentions where the "sold out" column comes from. What is wrong?
- (a) Nothing; the algorithm choice is the hard part
- (b) It has skipped the labels, which are the actual blocker ✅
- (c) It should have proposed a bigger model
- (d) It should have used a rule-based approach instead
> Without honestly recorded labels there is nothing to train on. Label collection is the expensive, unglamorous work that decides whether the project is possible at all.

### Q5 · L5
Your sell-out model worked well in March and is clearly worse in September. A new food stall opened outside campus in July. What has happened, and what is the fix?
- (a) The model has a bug; retrain with more epochs
- (b) The world changed, so the past it learned no longer describes today — measure and retrain ✅
- (c) Inference latency has increased
- (d) The features were never relevant
> This is drift. A model is a photograph of the period it was trained on, and the only defence is measuring it against reality on a schedule rather than trusting launch-day numbers.

## Go deeper
- [Google: Introduction to Machine Learning](https://developers.google.com/machine-learning/intro-to-ml)
- [Google: Framing a problem as machine learning](https://developers.google.com/machine-learning/problem-framing)
