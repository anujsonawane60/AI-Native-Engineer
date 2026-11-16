---
id: m09-04-classic-ml
module: m09-ai-foundations
slug: classic-ml-landscape
title: "Classic ML: Regression, Classification, Clustering and Trees"
order: 4
duration_min: 25
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

Your group needs to forecast tomorrow's thali demand so the kitchen stops throwing away food. One member wires it to a large language model: send the last thirty days as text, ask for tomorrow's number. It works. It also costs money on every call, takes two seconds, gives a slightly different answer each time you ask, cannot tell you why it said 140, and when you check it against the obvious baseline of "same as last Thursday", the baseline wins.

The data was three hundred rows in a spreadsheet. A method from the 1970s, running in half a second on a laptop with no GPU and no API key, would have beaten it, cost nothing per prediction, and printed out which factors mattered most.

This is the most common waste of money in student AI projects right now. The newest tool is not the right tool, and the gap between them is measured in rupees per request.

### The idea

**Most prediction problems a small startup has are rows and columns, and rows and columns have had excellent, cheap, explainable solutions for decades.**

Four tasks cover almost everything. Predict a number: **regression**. Pick a category: **classification**. Find groups nobody defined: **clustering**. Reduce many columns to a few: **dimensionality reduction**. Classic machine learning is a small toolbox of methods aimed at these four, and the whole toolbox runs free on a laptop.

You will not implement any of them. You need to recognise them by name so that when an AI proposes one you can tell whether it fits, and when an AI proposes something enormous instead you can ask why.

### The analogy

Walk into the canteen kitchen and look at the tools on the wall.

There is a knife, a peeler, a pressure cooker, a tawa and a blender. Nobody argues about which is best, because the question is meaningless. You peel with the peeler. The pressure cooker is magnificent and completely wrong for an omelette.

Classic machine learning is that wall. A linear model is the knife: simple, sharp, you can see exactly what it did. A decision tree is the peeler: crude, fast, and it tells you in plain language what it is doing. Gradient boosting is the pressure cooker: the thing professionals actually reach for on tabular data, powerful and slightly fussy. A neural network is an industrial food processor that arrived last year — astonishing on photos, audio and language, and absurd for peeling one potato.

**Where the analogy breaks:** kitchen tools do not mislead you. A model that fits badly still returns a confident number with no indication that it was the wrong tool. There is no equivalent of a knife feeling wrong in your hand, so the judgement has to happen before you start, which is what this chapter is for.

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

### The four families, in one pass each

**Linear and logistic models.** Fit a straight-line relationship between the inputs and the answer. Linear regression produces a number, logistic regression produces a probability of a category. Their enormous virtue is that you can read the result: "every extra 10 rupees of price reduces expected sales by 4 thalis" is a sentence a kitchen manager can argue with. Their limit is that real relationships are rarely straight.

**Decision trees.** A tree of yes/no questions. "Is it raining? Is the dish biryani? Were more than 130 portions cooked?" Follow the branches and you reach a prediction. A single tree is wonderfully readable — you can print it and show the head cook — and prone to memorising its training data if you let it grow deep.

**Ensembles: random forests and gradient boosting.** Build hundreds of mediocre trees and combine them. A **random forest** grows many trees on different random slices of the data and takes a vote. **Gradient boosting** grows trees one after another, each correcting the previous one's mistakes. Boosting is, as of now, the default serious answer for tabular prediction, and it is what you should expect a competent plan to propose for the thali forecast.

**k-means clustering.** Pick a number of groups, place that many centres in the data, and repeatedly nudge each centre to the middle of the points nearest it. Fast, easy, and remember chapter 2's warning: it will cheerfully hand you however many groups you asked for.

### Feature engineering: the part that actually matters

Switching from a tree to boosting might buy you two percentage points. Adding the right column buys you twenty.

For the thali forecast, the raw data has a date. A date is nearly useless. What matters is what you can build from it: day of the week, whether it is a holiday, whether exams are on, the rolling average of the last seven days, the same weekday last week, whether it rained. Every one of those is a human deciding what the model is allowed to notice.

This is **feature engineering**, and it is where domain knowledge enters a model. You know things about your canteen that no algorithm can discover from a date column — that the hostel mess closes on Sundays, that first-years get paid on the fifth. The single most valuable thing you can do on any tabular problem is sit with someone who understands the business and write down twelve things they think matter.

It is also the part an AI cannot do for you, because it does not know your canteen. It can suggest generic features competently. It cannot tell you that the chemistry department has a monthly account.

### Where it lives

Classic machine learning lives in a notebook and produces a small file. A trained gradient-boosting model for your thali forecast is likely a few hundred kilobytes, loads in milliseconds, and predicts in microseconds on an ordinary server with no GPU — which means it costs nothing per request. Compare that against the LLM call from the top of this chapter, which is a network hop with a price attached, exactly as M00 chapter 2 described.

That difference is the whole argument. On the architecture map, a classic model is a library inside your own server. A language model is a third party you depend on.

Nothing here is deprecated by the rest of this module. Chapters 5 onward are about deep learning, which won decisively on images, audio and language — data where the useful features are impossible to write down by hand. On your orders table, it generally does not win. The deeper practice of classic ML, including real feature pipelines, is M14.

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

**On small tabular data, trees beat neural networks, and this is not a controversial claim.** A few hundred to a few hundred thousand rows of mixed numbers and categories is gradient boosting's home ground. Deep learning needs much more data to find structure that trees find immediately. If an AI proposes a neural network for your 400-row canteen table, it is pattern-matching on the word "AI", not reasoning about your problem.

**Interpretability is a feature, not a consolation prize.** When the kitchen asks why the model says 140, "because boosting said so" ends the conversation badly. Classic models can usually tell you which features mattered and in which direction, and on a decision that affects real food and real money, that is often worth more than two points of accuracy. It also matters when anyone is affected by the decision — M07's fairness concerns and chapter 16's governance section both lean on being able to explain a model's reasoning.

**A language model is a bad calculator and an expensive one.** It predicts plausible text. Asked for a number it produces a plausible-looking number, varying between calls, with no arithmetic underneath. For forecasting, scoring and ranking on your own tabular data, this is the wrong tool at the wrong price. The exception is when your input is genuinely unstructured — free-text reviews, support messages, photos — which is precisely when classic methods struggle.

**Always check the baseline, again.** "Same as last Thursday" is a real forecasting method with a real score. Many commercial forecasting projects fail to beat it. Compute it in five minutes before you build anything.

**Beware k-means on data with wildly different scales.** A column measured in rupees and a column measured in minutes are not comparable, and whichever has bigger numbers will dominate the grouping. The fix is standard and routinely forgotten in generated code; ask explicitly whether the features were scaled.

### Verify it

**Red flag: a neural network proposed for a few hundred rows of tabular data.** Ask for the gradient-boosting version and the score it gets, side by side. Make it justify the neural network on evidence, not on principle.

**Red flag: an LLM call in the middle of a numeric prediction.** Ask what it costs per request and what happens when you call it twice with identical input. If the answer is "a different number", it is unfit for a forecast. Ask for the classic version and compare both score and cost per prediction.

**Red flag: no baseline in the comparison.** The same check as chapter 3, and still the one most often skipped. Demand "same as last week" or "always the average" in the results table.

**Red flag: a feature list that is just the raw columns.** If the plan feeds a date straight in, nobody has done any feature engineering, and the easiest twenty points of improvement are sitting unclaimed. Ask it to propose fifteen engineered features and then cross out the ones that do not apply to your canteen — that editing is your domain knowledge entering the model.

**Red flag: a cluster count presented as a discovery.** "We found four segments" after asking for four is not a finding, as chapter 2 warned.

And the check to run yourself: ask which three features mattered most, and see whether the answer matches what a person who runs the canteen would say. When the model's top feature is something nobody in the kitchen believes matters, you have either learned something genuinely interesting or you have found leakage. Investigating which is the most productive hour you will spend on the model.

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
