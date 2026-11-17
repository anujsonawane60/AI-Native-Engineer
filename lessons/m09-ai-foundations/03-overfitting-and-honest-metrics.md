---
id: m09-03-honest-metrics
module: m09-ai-foundations
slug: overfitting-and-honest-metrics
title: "Overfitting, the Train/Val/Test Split and Honest Metrics"
order: 3
duration_min: 15
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

Your group demos the sell-out predictor: 99.2% accurate. The kitchen uses it from Monday, and by Thursday the head cook is back to guessing and doing better. Nothing broke. The 99.2% was measured on the same four hundred days the model had already been shown — it had memorised them, the way a student memorises last year's question paper. This is the commonest place where generated machine-learning code lies to you, by default rather than by malice.

### The idea

**A model is only worth the score it gets on data it has never seen. Anything else measures its memory.**

So you divide your examples into three piles first. The **training** pile is what the model learns from, the **validation** pile is what you check against while still making choices, and the **test** pile is locked away and opened once, as your honest estimate of Monday.

**Overfitting** is fitting the training pile beautifully and the unseen piles badly. **Underfitting** is bad at everything, including its own training data.

### The analogy

The canteen tests a trainee cashier twice. First with the twenty bills he practised on all week: twenty out of twenty, which proves he remembers twenty bills and nothing about Monday's queue. That is the 99.2%. Then with twenty from a locked drawer he has never seen: fifteen out of twenty, less flattering, and the only number that predicts anything. And if the manager tests him on the locked twenty, coaches him and repeats fifteen times, his score climbs to nineteen — but those bills are no longer unseen, because the coaching leaked them in. Hence the third pile: one drawer for coaching, which is validation, and one opened exactly once, which is test.

**Where the analogy breaks:** the bills in the drawer stay the same forever, so one locked drawer would eventually do. Real data moves, and even an honestly measured model decays — which is why an honest score has a date on it.

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

Accuracy misleads on imbalanced data: of 10,000 payments 50 are fraud, so "not fraud" every time scores 99.5% and is worthless. Two numbers pull against each other instead. **Precision**: of the cases it flagged, how many were really fraud — low precision blocks honest customers. **Recall**: of the real frauds, how many it caught — low recall lets fraud through. **F1** combines them, useful for comparing models and useless for choosing a threshold, because it hides which direction you were willing to be wrong in. That direction is a business call no AI can make for you.

```
                    predicted
                 fraud    not fraud
  actual fraud  │  32   │    18     │  recall: 32 of 50
      not fraud │  70   │  9,880    │
                └───────┴───────────┘
                  precision: 32 of 102
```

Of 50 real frauds it caught 32, and of the 102 transactions it flagged only 32 were genuine, so 70 honest customers were inconvenienced.

### Where it lives

- **M10's evals are this chapter applied to language models.** A locked set of test questions with known good answers is a test split under another name.
- **Chapter 13 depends on it.** Choosing between three models means scoring them on your own held-out examples, not reading a published benchmark.
- **M07 dashboards fail the same way** one layer out, by counting the wrong thing confidently.
- **Building real evaluation harnesses is M14.** Here you learn the honest-measurement habit that makes them worth building.

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

- **Compute the stupid baseline first.** What does "always predict the most common answer" score? A model not clearly beating that is not working.
- **Leakage produces spectacular scores and instant production failure.** Three kinds: *target leakage*, a feature that only exists after the label ("refund issued"); *time leakage*, a random split on dated rows; *duplicate leakage*, the same row in both piles. The tell is a delightfully high score.
- **A small test split is a noisy test split.** A 15% split of 400 rows is 60 rows, so two models 1% apart are indistinguishable.
- **Touch the test set once.** Every look-and-change turns it into a validation set and your final number into fiction.
- **Leaderboard scores are not your score**, and an honest score has a date on it, so re-measure monthly.

### Verify it

- **A score reported on the training data** — the first red flag. Ask whether it was measured on examples the model trained on, and where the split happens.
- **Accuracy quoted with no mention of class balance.** On a rare-event problem, demand precision and recall or the confusion matrix.
- **A suspiciously high score.** 99% on a hard problem means leakage until proven otherwise. Ask, per feature, whether that value is known at prediction time.
- **A random split on time-ordered data.** If the rows have dates, the split must respect them.
- **No baseline in the plan.** Require the dumbest-possible-answer score beside the model's, in the same table.
- **The check nobody can fake:** predict ten fresh examples from this week that exist in no file the model has touched, and compare with reality.

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
