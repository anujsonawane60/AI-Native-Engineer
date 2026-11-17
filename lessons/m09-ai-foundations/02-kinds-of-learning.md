---
id: m09-02-kinds-of-learning
module: m09-ai-foundations
slug: kinds-of-learning
title: "Supervised, Unsupervised, Self-Supervised and Reinforcement Learning"
order: 2
duration_min: 15
is_free: false
mission: false
volatility: low
last_reviewed: 2026-11-16
---

## Key learnings
- Match a problem to the kind of learning it actually needs
- Say why prediction requires labels and grouping does not
- Explain how self-supervised learning lets text be its own label, which is why LLMs exist
- Describe reinforcement learning as learning from a score rather than from answers
- Catch an AI that promises prediction from unlabelled data

## Lesson

### The pain

A group pitches three AI features in one meeting: predict which dishes sell out, group customers into types, and a chatbot that answers menu questions. The first needs hundreds of days of recorded answers, which nobody kept; the second needs none at all; the third was built from raw text nobody labelled. The team applies the first feature's data requirements to all three and spends two weeks hunting labels it does not need.

### The idea

**There are four kinds of learning, and they differ in exactly one thing: what the program is given as feedback.**

Given the right answer for each example, it is **supervised**. Given no answers and asked to find structure, it is **unsupervised**. Given raw data from which an answer can be manufactured — hide part of it, predict the hidden part — it is **self-supervised**. Given only a score after acting, it is **reinforcement learning**. Each kind demands something different from you first, and that thing is usually the project's real cost.

### The analogy

Four ways people learn at the canteen.

**Supervised.** The new cashier gets a price list with every correct price and is corrected on every mistake — most useful, most expensive, because the list has to exist.

**Unsupervised.** The manager watches the queue for a month and notices three kinds of customer. Nobody told her those groups existed, and she cannot be "wrong" either.

**Self-supervised.** A trainee gets the recipe book with words blanked out: "add two spoons of ______ and simmer." The answer key was the original page, so the exercise manufactures its own labels, infinitely, from text that already existed. **This is how every large language model is built.**

**Reinforcement learning.** A waiter is told nothing about good service, just handed the tips each shift, and works out which behaviours earn more.

**Where the analogy breaks:** the waiter knows a tip means a happy customer. A reinforcement-learning system does not — it optimises the number it is scored on, so if standing near the door correlates with tips, it stands near the door forever. That gap between your number and the thing you wanted is reward hacking, in chapter 16.

### Picture it

```
             what feedback does it get?

  SUPERVISED       │ the right answer, for every example
                   │ needs: a labelled dataset (expensive)
                   │ gives: prediction and classification
  ─────────────────┼──────────────────────────────────────
  UNSUPERVISED     │ nothing at all
                   │ needs: just the data
                   │ gives: groups, structure, outliers
  ─────────────────┼──────────────────────────────────────
  SELF-SUPERVISED  │ answers it manufactures itself
                   │ needs: a mountain of raw data
                   │ gives: general ability, then fine-tuned
  ─────────────────┼──────────────────────────────────────
  REINFORCEMENT    │ a score, after acting
                   │ needs: a safe place to try, many tries
                   │ gives: behaviour, strategy, policies
```

The four kinds differ only in what feedback arrives, and that decides what you must prepare before training can start.

Supervised comes in two shapes: **classification** picks one of a fixed set of buckets (veg, non-veg, egg), **regression** produces a number on a scale (140 thalis on Thursday). The metrics differ, so mixing them up is a common error in generated code.

### Where it lives

- **Supervised** is where your own small models live — a ticket classifier trained in a free Colab notebook on four hundred of your own tickets.
- **Unsupervised** lives in analytics, not the product: the segments in M16 are clustering, the anomaly alerts in M07 are outlier detection.
- **Self-supervised** lives where you will never touch it — the pretraining run that produced the model you call over the network. Chapters 8 to 11 take it apart.
- **Reinforcement learning** reaches you once: the stage that turned a raw text-predictor into an assistant that follows instructions and refuses harmful requests, in chapter 11. Running it yourself is M14.

### Words to own

| Term | Plain meaning | Example |
|---|---|---|
| Supervised | Learns from examples with known answers | Ticket → category |
| Classification | Picks one of a fixed set of buckets | Veg / non-veg / egg |
| Regression | Produces a number on a scale | 140 thalis on Thursday |
| Unsupervised | Finds structure with no answers given | Three kinds of customer |
| Cluster | A group the program found by itself | The 12:15 speed crowd |
| Outlier | A point unlike all the others | One order of ₹4,000 |
| Self-supervised | Hides part of the data and predicts it | Guess the missing word |
| Pretraining | The big self-supervised run that makes a base model | How LLMs are made |
| Reinforcement learning | Learns from a score after acting, not from answers | Learning from tips |
| Reward | The number being optimised | Tips per shift |
| Policy | The behaviour the system has settled on | "Stand near the door" |
| Reward hacking | Maximising the number while missing the point | Gaming the metric |

### Tradeoffs and gotchas

- **You cannot predict from unlabelled data** — the commonest confusion in the subject. "Use unsupervised learning to predict churn" is not a plan: churn is a label. Either somebody records it, or you derive it ("no order in 60 days counts as churned") and it is supervised.
- **Clustering cannot be wrong, which makes it dangerous.** Ask for three segments and you get three; ask for seven and you get seven, and both look convincing. With no ground truth, a cluster is a hypothesis.
- **Self-supervised learning is out of reach at the scale that matters.** You will never pretrain a model — but this is where the model's knowledge, its biases and its cutoff came from.
- **Reinforcement learning needs a cheap place to be wrong thousands of times.** Games have one; your payment flow does not, and the failed attempts land on your customers.
- **Derived labels beat collected labels.** Did the user click, cancel, reopen the ticket? Your product already records behaviour, and nobody is paid to write it down.

### Verify it

- **Prediction promised from unlabelled data.** If the plan says "unsupervised" and the output is a specific prediction, it is wrong. Ask what the label is and where it comes from.
- **Clusters presented as facts.** "Your users fall into four segments" is a result of the number you asked for. Make it show three and six, and name the behaviour that separates them.
- **Reinforcement learning proposed for anything you ship this semester.** Ask where the thousands of failed attempts take place; if real users or money are involved, say no.
- **An invented reward.** Ask what behaviour would maximise that number while making the product worse. There is always one.
- **Check the shape before the method.** Bucket, number, group or behaviour — those four words map to classification, regression, clustering and reinforcement learning.

## Walkthrough
1. Write down three features your own startup could plausibly want from AI. Keep them concrete enough that you can name what comes out of each.
2. For each one, say in one word whether the output is a bucket, a number, a group or a behaviour. Do not skip this; it is the whole exercise.
3. Map those four words onto classification, regression, clustering and reinforcement learning. You should now have a kind of learning attached to each feature.
4. For every feature you marked supervised, name the label column and write down who or what records it today. Mark honestly whether it exists.
5. For any label that does not exist, try to derive one from behaviour your product already records — a click, a cancellation, a repeat order. Write the derivation as a sentence with a number in it.
6. For any feature you marked clustering, write down what you would actually do differently for each group. If you cannot, cross the feature off; it is analytics, not a product feature.
7. Now ask an AI to classify your three features into the four kinds, without telling it your own answers. Compare. Where you disagree, make it defend its choice.
8. Ask it specifically: for each of my supervised features, how many labelled examples would you want before the model beats a simple rule? Write the numbers down.
9. Pick the feature with the cheapest path to labels and write one sentence on why that makes it the one to build first.

## Talk to AI
```prompt
I am learning to tell the four kinds of machine learning apart, and I want to be tested rather than taught. Do not write any code and do not give me a lecture.

Here are three features my student startup is considering:
1. [feature one, in one sentence, including what comes out of it]
2. [feature two]
3. [feature three]

For each feature, do this:
a) Say whether the output is a bucket, a number, a group, or a behaviour.
b) Name which kind of learning that implies: supervised classification, supervised regression, unsupervised clustering, or reinforcement learning.
c) If it is supervised, name the exact label column, and tell me three ways a small team could obtain that label cheaply - including deriving it from behaviour we already record.
d) If it is clustering, tell me what we would have to do differently per group for it to be worth building, and warn me if the answer is "nothing".
e) Tell me whether a plain hand-written rule would do the job well enough for the first three months.

Then rank the three features by how cheaply we could get the data they need, and tell me which one to build first and why.

Finally, name the one feature in my list that you think we have misunderstood, and say what we have misunderstood about it.
```

## Stuck? Ask AI
```prompt
Explain supervised, unsupervised, self-supervised and reinforcement learning using one single running example from a cricket team, not a canteen. Then give me eight short problem descriptions, one at a time, and ask me which of the four kinds each needs. Do not reveal the answers until I have guessed.
```

```prompt
I do not understand why a large language model does not need labelled data. Explain self-supervised learning step by step, with a concrete sentence being masked and predicted, and tell me where the "label" comes from in that process. Then explain why this approach could use the whole internet when hand-labelling could not.
```

```prompt
Give me three examples of reward hacking, where a system maximised the score it was given and made the real situation worse. Then give me a reward I might choose for a food-delivery app and ask me to guess how it could be gamed, before you tell me.
```

## Quiz

### Q1 · L2
Which kind of learning is used to find customer groups nobody defined in advance?
- (a) Supervised classification
- (b) Unsupervised clustering ✅
- (c) Self-supervised pretraining
- (d) Reinforcement learning
> Clustering gets no answers at all and returns structure it found in the data. There is no ground truth, which is why clusters are a hypothesis rather than a finding.

### Q2 · L3
A teammate says "we have 10,000 orders and no labels, so we will use unsupervised learning to predict which customers will stop ordering". What is wrong?
- (a) Nothing; clustering can predict churn
- (b) 10,000 orders is too few for any method
- (c) Prediction needs a label; churn must be defined and derived ✅
- (d) Reinforcement learning would be the correct choice
> Predicting a specific outcome is supervised by definition. The fix is usually to derive the label from behaviour already recorded, such as "no order in 60 days".

### Q3 · L3
Why could large language models be trained on internet-scale text when hand-labelled datasets stayed small?
- (a) Internet text comes pre-labelled by its authors
- (b) The text is its own label: hide part of it and predict the hidden part ✅
- (c) Unsupervised learning needs no feedback of any kind
- (d) Reinforcement learning scales better than supervision
> That is self-supervised learning. The answer key is the original text, so labels are manufactured automatically and effectively without limit.

### Q4 · L4
An AI proposes reinforcement learning to optimise your delivery-fee pricing on real customers from day one. What is the strongest objection?
- (a) Reinforcement learning cannot handle prices
- (b) The thousands of failed attempts would happen to real paying users ✅
- (c) Pricing is a clustering problem
- (d) It needs a labelled dataset of correct prices
> Reinforcement learning needs a cheap place to be wrong many times. When failed attempts land on customers and revenue, the learning cost is paid by your business.

### Q5 · L5
Your startup wants to predict how many minutes until an order is ready. Which shape is this, and what does that imply about judging it?
- (a) Classification; judge it by how often it picks the right bucket
- (b) Clustering; judge it by how distinct the groups look
- (c) Regression; judge it by how far off the number is on average ✅
- (d) Reinforcement learning; judge it by total reward
> The output is a number on a continuous scale, so it is regression and the honest metric is average error in minutes. Treating it as classification hides how badly a wrong answer missed.

## Go deeper
- [Google: Types of machine learning systems](https://developers.google.com/machine-learning/intro-to-ml/supervised)
- [Spinning Up in Deep RL: what reinforcement learning is](https://spinningup.openai.com/en/latest/spinningup/rl_intro.html)
