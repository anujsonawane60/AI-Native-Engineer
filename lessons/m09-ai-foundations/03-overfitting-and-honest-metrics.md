---
id: m09-03-honest-metrics
module: m09-ai-foundations
slug: overfitting-and-honest-metrics
title: "Overfitting, the Train/Val/Test Split and Honest Metrics"
order: 3
duration_min: 25
is_free: false
mission: false
volatility: low
last_reviewed: 2026-11-16
---

## Key learnings
- Explain overfitting as memorising instead of generalising
- Split data into train, validation and test, and say what each one is for
- Spot the accuracy trap on imbalanced data and ask for precision and recall instead
- Name three kinds of data leakage that make a model look better than it is
- Catch an AI reporting a score measured on the data the model trained on

## Lesson

### The pain

Your group demos the sell-out predictor at the Friday review. 99.2% accurate. The room is impressed. The kitchen agrees to use it from Monday.

On Monday it is wrong about biryani. On Tuesday it is wrong about pongal. By Thursday the head cook has gone back to guessing, and he is doing better than your model.

Nothing broke between Friday and Monday. The 99.2% was measured on the same four hundred days the model had already been shown. The model had not learned which dishes sell out; it had memorised four hundred specific days, the way a student memorises last year's question paper. Then the exam changed and there was nothing underneath.

This chapter is the one that separates people who can judge a model from people who can only repeat a number. It is also, by a wide margin, the most common place where AI-generated machine-learning code lies to you — not maliciously, just by default.

### The idea

**A model is only worth the score it gets on data it has never seen. Anything else is a measurement of its memory.**

So you divide your examples into three piles before you do anything. The **training** pile is what the model learns from. The **validation** pile is what you check against while you are still making choices. The **test** pile you lock away and look at once, at the end, as your honest estimate of how it will behave on Monday.

**Overfitting** is the failure where the model fits the training pile beautifully and the unseen piles badly. **Underfitting** is the opposite and much easier to notice: it is bad at everything, including the data it trained on.

### The analogy

The canteen hires a cashier and tests him two ways.

In the first test, he is handed the same twenty bills he practised on all week. He gets twenty out of twenty. This proves he can remember twenty bills. It proves nothing about Monday's queue, and this is exactly the 99.2%.

In the second test, the manager keeps twenty bills in a locked drawer that the trainee has never seen, and tests him on those. Fifteen out of twenty. Lower, less flattering, and it is the only number that predicts anything.

Now the subtle part. Suppose the manager tests him on the locked twenty, coaches him on his mistakes, tests him again, coaches again, and repeats this fifteen times. His score on those twenty climbs to nineteen. Those twenty bills are no longer unseen — the coaching has leaked them in through the back door. That is why there is a third pile: one drawer for coaching rounds, which is validation, and a second drawer opened exactly once, which is test.

**Where the analogy breaks:** the twenty bills in the drawer stay the same forever, so one locked drawer would eventually be enough. Real data moves. Prices change, a new hostel opens, a food stall opens across the road. Even an honestly measured model decays, which is why chapter 1's point about drift sits next to this one: an honest score has a date on it.

### Picture it

```
  all your examples (400 canteen days)
  ┌──────────────────────────────┬─────────┬─────────┐
  │          TRAIN  70%          │  VAL 15%│ TEST 15%│
  └──────────────────────────────┴─────────┴─────────┘
         learns from this          tune on    open
                                   this      ONCE

  what the two scores tell you
  ┌─────────────┬──────────────┬─────────────────────┐
  │ train score │ val score    │ diagnosis           │
  ├─────────────┼──────────────┼─────────────────────┤
  │   low       │   low        │ underfit: too simple│
  │   high      │   high       │ good: it generalises│
  │   high      │   low        │ OVERFIT: memorised  │
  └─────────────┴──────────────┴─────────────────────┘
```

Data is split three ways before training begins, and the gap between the training score and the validation score is what tells you whether the model learned a pattern or a list.

### The accuracy trap

Accuracy is the fraction of predictions that were right, and on imbalanced data it is actively misleading.

Suppose you build a fraud detector for your startup's payments. Out of 10,000 transactions, 50 are fraudulent. A model that outputs "not fraud" for every single transaction is 99.5% accurate and completely worthless. It has never caught anything.

So you need two numbers instead of one, and they pull against each other.

**Precision** asks: of the cases it flagged, how many were really fraud? Low precision means you are blocking honest customers.

**Recall** asks: of the real frauds, how many did it catch? Low recall means fraud is getting through.

You can always buy one with the other. Flag everything and recall is perfect while precision collapses. Flag nothing and precision is undefined while recall is zero. **F1** is a single number that combines the two, useful for comparing models but useless for deciding a threshold, because it hides which direction you were willing to be wrong in.

That decision is yours, not the model's, and it is a business decision. For canteen fraud, blocking an honest student at the counter is worse than missing a small fraud, so you favour precision. For detecting spoiled food, missing one is far worse than an extra inspection, so you favour recall. An AI cannot make this call for you because it depends on which mistake costs your product more.

The clearest way to see all of this is a confusion matrix: four counts, laid out so you can read exactly how it was wrong.

```
                    predicted
                 fraud    not fraud
  actual fraud  │  32   │    18     │  recall: 32 of 50
      not fraud │  70   │  9,880    │
                └───────┴───────────┘
                  precision: 32 of 102
```

Of 50 real frauds it caught 32, and of the 102 transactions it flagged only 32 were genuine, so 70 honest customers were inconvenienced.

### Leakage, the silent version

Leakage is when information that would not be available at prediction time sneaks into training. It produces spectacular scores and a model that fails instantly in production. Three kinds, all of which you will meet.

**Target leakage.** A feature that is a consequence of the label. "Refund issued" is a brilliant predictor of fraud and only exists after the fraud was found, so at prediction time you will never have it.

**Time leakage.** You split 400 days randomly, so the model trains on December and is tested on November. It has effectively seen the future. Any data with a time order must be split by time: train on the earlier months, test on the later ones.

**Duplicate leakage.** The same order appears in both train and test because of a copy in your export. The model recognises it rather than predicts it.

The tell for all three is identical: a score that is suspiciously, delightfully high. Treat 99% on a hard problem as a bug report, not a result.

### Where it lives

This is the chapter that quietly governs M10. When you build a RAG pipeline there, the way you tell whether a change improved it is a locked set of test questions with known good answers — a test split, under a different name. The evals in M10 chapter 12 are this chapter applied to language models, and the reason they work is the discipline you learn here: measure on inputs the system has not been tuned against.

It also governs chapter 13 of this module. Choosing between three models means scoring them on your own held-out examples, not reading a published benchmark. And in M07, dashboards that look great because they count the wrong thing are this same failure one layer out.

The practice of building serious evaluation harnesses is M14. Here you learn the honest-measurement habit that makes those harnesses worth building.

### Words to own

| Term | Plain meaning | Example |
|---|---|---|
| Generalise | Work on data it never saw | 15 of 20 locked bills |
| Overfit | Fits training data, fails on new data | The 99.2% demo |
| Underfit | Bad even on training data | Too simple a model |
| Train split | What the model learns from | 70% of your days |
| Validation split | What you tune against | The coaching drawer |
| Test split | Opened once, at the end | The honest number |
| Baseline | The dumbest reasonable answer | "Always predict no" |
| Accuracy | Fraction of predictions correct | Misleading when imbalanced |
| Precision | Of those flagged, how many were real | 32 of 102 |
| Recall | Of the real ones, how many were caught | 32 of 50 |
| Confusion matrix | The four counts, laid out | The table above |
| Leakage | Training saw what prediction will not have | "Refund issued" |
| Drift | The world moved away from the training data | A new food stall |

### Tradeoffs and gotchas

**Always compute the stupid baseline first.** What score does "always predict the most common answer" get? What does "predict today the same as yesterday" get? If your model is not clearly beating that, it is not working, no matter how respectable its number looks. Many teams ship models that are worse than a one-line rule because nobody ever computed the one-line rule.

**A small test split is a noisy test split.** On 400 examples, a 15% test split is 60 rows, and 60 rows means your score has a wobble of several percentage points. Two models that differ by 1% on 60 rows are indistinguishable, and treating that gap as real is how teams pick the worse model confidently.

**Touch the test set once.** Every time you look at it and change something, it becomes a little more of a validation set, and your final number becomes a little more of a fiction. The discipline costs nothing and is broken constantly.

**Leaderboard scores are not your score.** A model at the top of a public benchmark was measured on somebody else's examples. Chapter 13 makes this concrete, and it is the main reason model choice has to be tested rather than read.

**An honest score has a date on it.** Rerun your test monthly against fresh labelled data. A model that scored 82% in March and 71% in September has not broken; the world has moved, and your measurement is the only thing that would have told you.

### Verify it

**Red flag number one, above all others: a score reported on the training data.** Ask the direct question — "was this measured on examples the model trained on?" — and make it show you where the split happens. This is the default behaviour of quickly generated notebooks, and it is the reason the Friday demo lied.

**Red flag: accuracy quoted with no mention of class balance.** On any rare-event problem — fraud, churn, spoilage, cancellation — demand the precision and recall or the confusion matrix. If an AI reports only accuracy on an imbalanced problem, it has either not looked at the balance or is hiding it.

**Red flag: a suspiciously high score.** 99% on a hard problem means leakage until proven otherwise. Ask it to list every feature and say, for each one, whether that value would be known at the moment of prediction. That question finds target leakage in about a minute.

**Red flag: a random split on time-ordered data.** If the rows have dates, the split must respect them. Ask explicitly: is this split random or chronological, and why.

**Red flag: no baseline anywhere in the plan.** Require the dumbest-possible-answer score next to the model's score, in the same table, every time.

And then run the one check nobody can fake: take ten fresh examples from this week, ones that exist in no file the model has ever touched, predict them by hand with the model, and compare against what actually happened. Ten real rows will tell you more than any number in the notebook.

## Walkthrough
1. Take any prediction problem from your own startup and write down the label, then count honestly how many labelled examples you have.
2. Decide your split. Write the three percentages and the actual row counts. If the test split comes out under about fifty rows, write a sentence acknowledging that your score will be noisy.
3. Check whether your data has a time order. If it does, write down the date boundary you will split on, with earlier rows for training and later ones for test.
4. Compute the dumbest baseline by hand, with no model at all: always predict the most common answer. Write down the score it gets. This is the bar.
5. List every feature column and, next to each, write "known before the outcome" or "known only after". Anything in the second group is target leakage and must be deleted.
6. Check for duplicates. Count distinct rows versus total rows. If they differ, find out why before you trust any score.
7. Now brief an AI to train a simple model, insisting it reports the score on train and on validation separately, plus the baseline, in one table. Read the gap between the two scores and diagnose it against the table in this chapter.
8. Ask for the confusion matrix, not just accuracy. Read all four numbers aloud and say in plain words what each wrong box costs your product.
9. Decide, as a business call, whether you would rather have higher precision or higher recall, and write the one sentence that justifies it in terms of which mistake hurts your users more.
10. Open your test split exactly once, record the number with today's date next to it, and write down when you will re-measure.

## Talk to AI
```prompt
I am evaluating a prediction model and I want an honest measurement rather than a flattering one. Do not optimise the score. Do not write a long notebook.

My situation:
- What I am predicting: [the label, in plain words]
- Number of labelled examples: [X]
- Roughly how often the positive case occurs: [e.g. 50 out of 10,000]
- Does my data have a time order: [yes/no, and the date range]
- My feature columns: [list them all]

Please do these, in order, and stop after each so I can respond:

1. For EVERY feature I listed, tell me whether its value would genuinely be known at the moment of prediction, or only after the outcome is known. Name the ones that are target leakage and must be removed.

2. Tell me how to split this data, and whether the split must be chronological rather than random given my answer above. Give me the actual row counts per split, and tell me plainly whether my test split is too small for the differences I will be trying to measure.

3. Give me the dumbest possible baseline for this problem and the score it would get. I want this number before any model exists.

4. Tell me which metric I should judge this on given my class balance, and explain in one sentence each what precision and recall mean FOR MY problem - what real-world mistake each one represents.

5. Ask me which of the two kinds of mistake is more expensive for my product, and explain why you cannot make that choice for me.

Throughout: if any score you would report could have been measured on data the model trained on, say so explicitly and tell me how to tell the difference.
```

## Stuck? Ask AI
```prompt
Explain overfitting using an analogy that is not about exams, students or memorising question papers. Then describe six situations, one at a time, and ask me whether each one is overfitting, underfitting, leakage or a healthy model. Do not tell me the answers until I have guessed each one.
```

```prompt
I have a model that is 99% accurate at detecting something that happens 1% of the time. Walk me through, step by step, why that number might be worthless. Then show me the confusion matrix for a model that predicts "no" every single time, and ask me to work out its precision and recall before you tell me.
```

```prompt
Explain data leakage with three concrete examples: one where a feature is a consequence of the label, one where the split ignores time, and one where rows are duplicated. For each, tell me the single question I could ask that would have caught it, and what the score would look like before and after the fix.
```

## Quiz

### Q1 · L2
What is the test split for?
- (a) Teaching the model the hardest examples
- (b) One honest estimate of performance on unseen data, looked at once ✅
- (c) Choosing between model settings during tuning
- (d) Replacing the training data when it runs out
> Validation is for tuning; test is the locked drawer opened once. Every extra look at the test set turns it into another validation set and makes the final number a fiction.

### Q2 · L3
A fraud detector reports 99.5% accuracy on data where 50 of 10,000 transactions are fraud. What should you suspect first?
- (a) The model is excellent and ready to ship
- (b) Accuracy is hiding that it may catch almost no fraud ✅
- (c) The test split was too large
- (d) The learning rate was set too high
> Predicting "not fraud" every time scores 99.5% and catches nothing. On rare events, demand precision and recall or the confusion matrix.

### Q3 · L4
Your model scores 0.97 on training data and 0.62 on validation. What is the diagnosis?
- (a) Underfitting; the model is too simple
- (b) Overfitting; it memorised the training rows ✅
- (c) The validation split is mislabelled
- (d) Healthy generalisation
> A high training score with a much lower validation score is the signature of memorisation. Low on both would be underfitting instead.

### Q4 · L4
Your sell-out predictor includes a feature called "extra portions cooked at 2 pm". Why is this a problem?
- (a) It is not numeric enough to train on
- (b) It is a consequence of selling out, so it will not exist at prediction time ✅
- (c) It will make the model underfit
- (d) It needs to be scaled before use
> That is target leakage. The feature is only known after the outcome, so the model scores brilliantly in the notebook and is useless on Monday morning.

### Q5 · L5
An AI hands you a notebook reporting 94% accuracy on your canteen data. What single question most reliably exposes whether that number is honest?
- (a) Which algorithm did you use?
- (b) Was this measured on rows the model trained on, and where is the split? ✅
- (c) How many epochs did you run?
- (d) What is the model's file size?
> Scoring on training data is the default failure of quickly generated notebooks. Locating the split, and seeing the baseline next to the score, is what turns a number into evidence.

## Go deeper
- [Google: Classification metrics, precision and recall](https://developers.google.com/machine-learning/crash-course/classification/accuracy-precision-recall)
- [Google: Overfitting and generalization](https://developers.google.com/machine-learning/crash-course/overfitting/overfitting)
