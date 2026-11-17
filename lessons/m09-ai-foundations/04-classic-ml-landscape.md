---
id: m09-04-classic-ml
module: m09-ai-foundations
slug: classic-ml-landscape
title: "Classic ML: Regression, Classification, Clustering and Trees"
order: 4
duration_min: 15
is_free: false
mission: false
volatility: low
last_reviewed: 2026-11-16
---

## Key learnings
- Match the four classic tasks to the four classic families of method
- Say why trees and boosting still beat deep learning on small tabular data
- Explain what feature engineering is and why it matters more than the algorithm
- Judge when an LLM is the wrong tool for a numeric prediction
- Catch an AI reaching for a neural network on four hundred rows

## Lesson

### The pain

Your group needs to forecast tomorrow's thali demand so the kitchen stops throwing away food. One member wires it to a language model: send thirty days as text, ask for tomorrow's number. It works — and costs money per call, takes two seconds, answers differently each time, cannot say why it said 140, and loses to "same as last Thursday". The data was three hundred spreadsheet rows, where a method from the 1970s would have won in half a second on a laptop, free per prediction, and printed which factors mattered.

### The idea

**Most prediction problems a small startup has are rows and columns, and rows and columns have had excellent, cheap, explainable solutions for decades.**

Four tasks cover almost everything. Predict a number: **regression**. Pick a category: **classification**. Find groups nobody defined: **clustering**. Reduce many columns to a few: **dimensionality reduction**. The whole toolbox runs free on a laptop. You will not implement any of it; you need the names, so that when an AI proposes one you can tell whether it fits.

### The analogy

Look at the tools on the kitchen wall: knife, peeler, pressure cooker, tawa, blender. Nobody argues about which is best, because the question is meaningless — the pressure cooker is magnificent and wrong for an omelette. A linear model is the knife: simple, sharp, you can see what it did. A decision tree is the peeler: crude, fast, readable. Gradient boosting is the pressure cooker professionals reach for on tabular data. A neural network is the industrial food processor that arrived last year — astonishing on photos and language, absurd for peeling one potato.

**Where the analogy breaks:** kitchen tools do not mislead you. A model that fits badly still returns a confident number, with no equivalent of a knife feeling wrong in your hand, so the judgement happens before you start.

### Picture it

```
  what are you trying to produce?

  a NUMBER ─────▶ regression
                  linear regression · tree · gradient boosting
                  "140 thalis tomorrow"

  a CATEGORY ───▶ classification
                  logistic regression · tree · random forest
                  "this ticket is about payment"

  a GROUPING ───▶ clustering
                  k-means · hierarchical
                  "three kinds of lunch customer"

  FEWER COLS ───▶ dimensionality reduction
                  PCA
                  "40 survey questions become 3 scores"
```

The four classic tasks, with the handful of method names you will actually see proposed for each.

Four families fill that map. **Linear and logistic models** fit a straight line you can read aloud: "every extra ₹10 of price costs 4 thalis". **Decision trees** are readable yes/no questions — print one for the head cook — and memorise their data if grown too deep. **Ensembles** combine hundreds of mediocre trees: a *random forest* votes across random slices, *gradient boosting* builds them in sequence, each fixing the last, and boosting is today's default for tabular prediction. **k-means** places the group centres you asked for and nudges them towards nearby points.

### Where it lives

- **A classic model is a library inside your own server** — a few hundred kilobytes, loading in milliseconds, predicting in microseconds with no GPU, free per request.
- **A language model is a third party you depend on**, a network hop with a price attached. That difference is this chapter's whole argument.
- **Nothing here is deprecated by chapters 5 onward.** Deep learning won on images, audio and language, where features cannot be written by hand. On your orders table it usually does not win.
- **The deeper practice, including real feature pipelines, is M14.**

### Words to own

| Term | Plain meaning | Example |
|---|---|---|
| Tabular data | Rows and columns, like a spreadsheet | Your orders table |
| Linear regression | Fits a straight-line relationship to a number | Price versus sales |
| Logistic regression | Same idea, outputs a probability of a class | Chance of cancellation |
| Decision tree | A readable tree of yes/no questions | "Raining? Biryani?" |
| Random forest | Many random trees voting | Steadier than one tree |
| Gradient boosting | Trees built in sequence, each fixing the last | The tabular default |
| Ensemble | Many weak models combined | Forests and boosting |
| k-means | Splits data into k groups you chose | Three customer types |
| PCA | Squashes many columns into a few | 40 questions into 3 |
| Feature engineering | Humans building the columns the model sees | "Same weekday last week" |
| Interpretability | Being able to say why it answered that | A printed tree |

### Tradeoffs and gotchas

- **Feature engineering beats method choice.** Boosting instead of a tree might buy two points; the right column buys twenty. A date is useless until a human turns it into day of week, holiday, exams on, rolling average — and an AI cannot do that for you, because it does not know the chemistry department has a monthly account.
- **On small tabular data, trees beat neural networks.** A few hundred to a few hundred thousand mixed rows is boosting's home ground; a network proposed for your 400-row table is pattern-matching on the word "AI".
- **Interpretability is a feature, not a consolation prize.** "Because boosting said so" ends the kitchen conversation badly.
- **A language model is a bad calculator and an expensive one**, returning a plausible number that varies between calls with no arithmetic underneath. The exception is unstructured input, where classic methods struggle.
- **Beware k-means on unscaled data.** Rupees and minutes are not comparable, and the bigger numbers dominate the grouping.

### Verify it

- **A neural network proposed for a few hundred rows of tabular data.** Ask for the gradient-boosting version and its score, side by side.
- **An LLM call inside a numeric prediction.** Ask its cost per request, and what happens on two identical calls. "A different number" means unfit for a forecast.
- **No baseline in the comparison.** Demand "same as last week" or "always the average" in the results table.
- **A feature list that is just the raw columns.** Ask for fifteen engineered features and cross out the ones that do not fit your canteen.
- **A cluster count presented as a discovery.** "We found four segments" after asking for four is not a finding.
- **Check yourself:** ask which three features mattered most and see whether someone who runs the canteen agrees. If not, you have found insight or leakage.

## Walkthrough
1. Pick one numeric prediction your own startup would benefit from. Write it as a sentence that ends in a number with a unit — "thalis tomorrow", "minutes to ready", "orders this week".
2. Compute the dumbest baseline by hand on real data: last week's value, or the overall average. Write down how wrong it is on average, in the unit, not as a percentage.
3. List the raw columns you actually have. Then list twelve features you could build from them — day of week, holiday flag, rolling seven-day average, same weekday last week, weather, exam period, anything your team knows.
4. Show that list of twelve to someone who knows the business — a canteen worker, a shopkeeper, your own first customer — and ask which three matter most and what is missing. Write down what they say.
5. Brief an AI to train gradient boosting on this, reporting its score against your baseline in the same table, plus the top five features by importance.
6. Separately, ask an LLM the same forecasting question as free text. Record its answer, the latency, and the cost. Then ask it the identical question again and record whether the number changed.
7. Put the three results side by side: baseline, boosting, LLM. Add two more columns — cost per prediction and whether you can explain the answer.
8. Read the boosting feature importances and compare them against what your business person told you in step 4. Investigate any big disagreement; it is leakage or insight.
9. Write one decision-log entry in your repo naming the method you chose, its score against the baseline, its cost per prediction, and what you rejected.

## Talk to AI
```prompt
I have a tabular prediction problem and I want the cheapest method that works, not the most impressive one. Do not use a large language model anywhere in the solution unless you can defend it against the alternatives.

My problem:
- What I am predicting, with its unit: [e.g. number of thalis sold tomorrow]
- Rows of history available: [X]
- Raw columns I have: [list them]
- Who will use the prediction and what decision it changes: [describe]
- Can I explain "why" to this user: [does it matter? yes/no]

Please work through this in order and stop after step 3 for my input:

1. Give me the dumbest reasonable baseline for this problem and tell me how I would compute its error by hand, in my unit.

2. Propose fifteen engineered features from my raw columns. For each, say in one line what real-world effect it is meant to capture. Mark the five you would bet on.

3. Recommend ONE classic method - linear, tree, random forest, or gradient boosting - and defend it against the other three for MY row count specifically. Then tell me, honestly, at what row count your recommendation would change to a neural network.

4. Tell me what this costs to run per prediction once trained, where it would live in a normal web app, and whether it needs a GPU.

5. Tell me what I would lose by using a large language model for this instead: on accuracy, on cost per request, on repeatability, and on being able to explain the answer. Be specific with numbers where you can, and flag any number you are unsure of.

6. Name the three features whose importance, if it came out high, would make you suspect data leakage in my setup.
```

## Stuck? Ask AI
```prompt
Explain the difference between a decision tree, a random forest and gradient boosting using one running example about sorting mangoes, with no mathematics at all. Then give me four situations and ask me which of the three I should reach for, without telling me the answers first.
```

```prompt
I do not understand feature engineering. Take a single date column and show me ten different features that can be built from it, explaining what real-world effect each one captures. Then give me a raw column of my own - I will paste it - and ask me what features I think it hides, before you tell me yours.
```

```prompt
Explain, with concrete numbers, why gradient boosting usually beats a neural network on a few thousand rows of spreadsheet data, but loses badly on photographs. Then ask me three questions about a dataset and tell me which family you would choose for it and why.
```

## Quiz

### Q1 · L2
You need to predict how many minutes until an order is ready. Which classic task is this?
- (a) Classification
- (b) Regression ✅
- (c) Clustering
- (d) Dimensionality reduction
> The output is a number on a continuous scale, so it is regression. Classification would be picking from fixed buckets such as "fast" or "slow".

### Q2 · L3
An AI proposes a deep neural network for a 400-row canteen sales table. What is the right response?
- (a) Accept it; neural networks are state of the art
- (b) Ask for gradient boosting and a baseline alongside it, and make the network justify itself ✅
- (c) Collect ten million rows first
- (d) Switch to clustering instead
> On small tabular data, boosted trees are the strong default. A network proposed without evidence at this row count is pattern-matching on the word "AI".

### Q3 · L4
Your team forecasts demand by sending thirty days of history to a language model as text. Name the two worst problems with this.
- (a) It is too slow to train and needs a GPU
- (b) It costs money per request and gives a different number each call ✅
- (c) It cannot read numbers at all
- (d) It requires labelled data that you do not have
> A classic model runs inside your own server for effectively nothing per prediction and is repeatable. An LLM call is a priced network hop producing plausible rather than computed numbers.

### Q4 · L4
Which change is most likely to improve a tabular model by the largest margin?
- (a) Switching from a random forest to gradient boosting
- (b) Adding engineered features such as "same weekday last week" ✅
- (c) Increasing the number of training epochs
- (d) Using a larger test split
> Algorithm swaps typically buy a couple of points. The right features, which encode what humans know about the business, routinely buy far more.

### Q5 · L5
Your boosting model's most important feature is one the head cook insists is irrelevant. What should you do?
- (a) Ignore it; feature importance is unreliable
- (b) Investigate it: this is either real insight or data leakage ✅
- (c) Remove the feature and retrain immediately
- (d) Trust the model over the cook in all cases
> A strong disagreement between the model and a domain expert is the highest-value hour of the project. It resolves into either a leaked consequence of the label or something the business did not know.

## Go deeper
- [scikit-learn: choosing the right estimator](https://scikit-learn.org/stable/machine_learning_map.html)
- [Google: feature engineering in the ML crash course](https://developers.google.com/machine-learning/crash-course/numerical-data)
