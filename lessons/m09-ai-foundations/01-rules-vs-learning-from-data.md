---
id: m09-01-rules-vs-learning
module: m09-ai-foundations
slug: rules-vs-learning-from-data
title: "Rules vs Learning From Data: Features, Labels, Training, Inference"
order: 1
duration_min: 15
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

Your group wants the canteen app to tell the kitchen which dishes will sell out before 1 pm. Someone writes the rule — Monday biryani sells out — then adds "unless it rains", "unless exams are on", "unless there is a match". Four weeks later there are forty-one rules in one file, two of them contradict each other, and the kitchen has stopped trusting it. Nobody wrote bad rules; they attempted what nobody can do by hand — state a rule with thirty interacting inputs that changes every semester.

### The idea

**Instead of writing the rule, you show a program thousands of past examples and let it find the rule itself.**

You hand it the things it may look at — day, dish, weather, portions cooked, exams on or off — and the answer you already know for each past day. **Training** searches for a setting of numbers that reproduces those answers. What comes out is a **model**: a file of numbers plus a little code that uses them. Running it on a day you have never seen is **inference**. Everything later in this module is this same shape at a larger scale.

### The analogy

Two ways to train a new cashier. One is a laminated price sheet: thali ₹60, extra curd ₹10, green card 10% off — short, readable, and when a price looks wrong you can point at the line that produced it. The other is two weeks beside the old cashier, absorbing six thousand transactions including the parts nobody wrote down. The choice turns on one question: is the rule short enough for a human to write down? Pricing is. "Will biryani sell out today" is not.

**Where the analogy breaks:** the shadowing cashier can say "I am not sure, let me ask". A trained model cannot. It has no idea which situations were absent from those two weeks, and produces a confident price for a customer type it never saw.

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

Six rows of the canteen's own history — this is a **dataset**.

| Day | Dish | Rain | Exams | Portions cooked | Sold out by 1 pm? |
|---|---|---|---|---|---|
| Mon | Biryani | No | No | 120 | Yes |
| Tue | Pongal | No | No | 90 | No |
| Wed | Biryani | Yes | No | 120 | No |
| Thu | Thali | No | Yes | 150 | No |
| Fri | Biryani | No | No | 160 | Yes |
| Sat | Pongal | No | No | 60 | Yes |

The first five columns are **features**: what the model may see. The last is the **label**: the answer you already know, because it already happened. One row is one **example**. Somebody had to record that label honestly for hundreds of days — the expensive part of every machine-learning project. And note "portions cooked": 60 portions of pongal sold out, 90 did not. Features are a bigger lever than algorithms.

### Where it lives

- **Training lives outside your app** — a free Colab or Kaggle notebook, on their GPU, once. It reads past examples and writes a model file. No user waits for it.
- **Inference lives inside your app**, as a hop: your server loads the model file and predicts in a function call, or you call somebody else's model over the network, paying the latency and per-call cost M00 chapter 2 showed.
- **Training as a practice is M14.** M09 is concepts; training loops, fine-tuning and shrinking a model to fit a small GPU are the Rank-2 practicum.

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

- **Rules are free and better than you think.** If the logic fits on an index card, write it on an index card. Reach for learning from data only when the rule has many interacting inputs, changes over time, or nobody can state it aloud.
- **Labels are the expensive part, and nobody budgets for them.** With no record of which dishes sold out, this is a data-collection project this month.
- **Garbage in, confident garbage out.** If the kitchen recorded "sold out" loosely, the model learns that inconsistency and reports it just as confidently.
- **A model is a photograph of the past.** A stall opens across the road and the model answers as if last year were still true. That is **drift**; the defence is measuring against reality on a schedule.
- **You now operate two systems**, code and data. Pipelines, retraining and model versioning are work forty-one rules never needed.

### Verify it

- **It proposes machine learning for a three-rule problem.** Ask for the rule-based version first; if it cannot name a case that rule gets wrong, you have no modelling problem.
- **It never mentions where the labels come from.** Ask which column is the label, who records it, how many rows, over what period.
- **It reports accuracy on the examples it trained on.** That number is meaningless and always flattering — chapter 3 is about this.
- **It proposes "fine-tune a model" for a problem about fresh facts.** Needing today's menu is retrieval, not training.
- **Check yourself:** say out loud which columns are features and which is the label. If you cannot, nobody can build this yet.

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
