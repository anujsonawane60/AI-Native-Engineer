---
id: m09-02-kinds-of-learning
module: m09-ai-foundations
slug: kinds-of-learning
title: "Supervised, Unsupervised, Self-Supervised and Reinforcement Learning"
order: 2
duration_min: 25
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

A group pitches three AI features for their canteen app in the same meeting.

"We will predict which dishes sell out." Fine — but somebody has to have recorded, for hundreds of days, whether each dish sold out. Nobody did.

"We will group our customers into types so we can market to each." Also fine, and this one needs no recorded answers at all, because there are no correct answers to record. The team does not notice that these two features have completely different data requirements and spends two weeks looking for labels they do not need.

"We will build a chatbot that answers questions about the menu." Fine again, and here the team assumes somebody must have hand-written thousands of question-and-answer pairs, because that is what the first feature taught them. Nobody did, and nobody needed to. The model behind that chatbot learned from raw text that was never labelled by anyone.

Three features, three different kinds of learning, and the team is applying the data requirements of the first one to all three. That mistake costs weeks.

### The idea

**There are four kinds of learning, and they differ in exactly one thing: what the program is given as feedback.**

Given the right answer for each example, it is **supervised**. Given no answers and asked to find structure, it is **unsupervised**. Given raw data from which an answer can be manufactured automatically — hide part of it and predict the hidden part — it is **self-supervised**. Given only a score after acting, it is **reinforcement learning**.

That is the whole taxonomy. The reason it matters to you is practical: each kind demands a different thing from you before you can start, and the thing it demands is usually the project's real cost.

### The analogy

Four ways people learn at the canteen.

**Supervised.** A new cashier gets a price list with every dish and its correct price, and is corrected each time they charge wrongly. Clear right answers, supplied by someone who already knows. Most useful, most expensive to set up, because the price list has to exist.

**Unsupervised.** The manager watches the lunch queue for a month and notices, without anyone telling her anything, that customers fall into roughly three kinds: the 12:15 crowd who want speed, the 1:30 crowd who want to sit and talk, and the staff who order the same thing daily. Nobody told her those groups existed. She found them. She also cannot be "wrong", which is both the appeal and the catch.

**Self-supervised.** A trainee is handed the canteen's recipe book with random words blanked out, and has to guess each missing word. "Add two spoons of ______ and simmer." Nobody wrote an answer key — the answer key was the original page, so the exercise generates its own labels, infinitely, from text that already existed. Do this for ten million recipes and the trainee ends up knowing a great deal about cooking without a teacher ever having graded anything. **This is how every large language model is built**, which is why they could be trained on the internet rather than on a hand-labelled dataset nobody could afford to make.

**Reinforcement learning.** A new waiter is told nothing about good service. He is simply given the tips at the end of each shift. Over months he works out which behaviours earn more, by trying things and watching the number.

**Where the analogy breaks:** the waiter understands that a tip means a happy customer. A reinforcement-learning system understands nothing of the sort. It optimises the number it is scored on, and if standing near the door happens to correlate with tips, it will stand near the door forever. The gap between the number you chose and the thing you actually wanted is where these systems go wrong, and it has a name you will meet in chapter 16: reward hacking.

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

The four kinds differ only in what feedback arrives, and that difference decides what you must prepare before any training can start.

### Supervised learning, and the two shapes it takes

Supervised is the workhorse and almost everything a student startup needs is one of its two shapes.

**Classification** puts an example into one of a fixed set of buckets. Is this review positive or negative? Is this support ticket about delivery, payment or food quality? Is this dish veg, non-veg or egg? The answer is a category.

**Regression** produces a number on a continuous scale. How many thalis will we sell on Thursday? How many minutes until this order is ready? What should this dish be priced at?

Telling them apart matters because the metrics differ. A classifier is judged on how often it picks the right bucket, with all the subtlety chapter 3 covers. A regressor is judged on how far off its number was, on average. Mixing those up is a common and confusing error in generated code.

### Where it lives

Supervised learning is where your own small models live, if you build any — a ticket classifier trained in a free Colab notebook on four hundred of your own tickets. Unsupervised learning usually lives in analytics rather than in your product: the customer segments you will build in M16 are clustering, and the anomaly alerts in M07 are outlier detection.

Self-supervised learning lives somewhere you will never touch. It is the enormous pretraining run that produced the model you call over the network — months of compute on thousands of GPUs, costing more than your whole college. You consume the result. Chapters 8 to 11 take that process apart.

Reinforcement learning appears in your world in exactly one place, and it is an important one: it is the stage that turned a raw text-predictor into an assistant that follows instructions and declines harmful requests. That is chapter 11. Running actual reinforcement learning yourself is M14.

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

**You cannot predict from unlabelled data, and this is the single most common confusion in the subject.** "We have 10,000 orders, let's use unsupervised learning to predict churn" is not a plan. Churn is a label. Either somebody records who churned, or you derive the label from the data you have — "no order in 60 days counts as churned" — and then it is supervised. Deriving the label is often the right move and takes an afternoon. Pretending you do not need one takes a month.

**Clustering cannot be wrong, which makes it dangerous.** Ask for three customer segments and you get three. Ask for seven and you get seven. Both will look convincing on a chart. There is no ground truth to check against, so the only honest validation is whether the groups mean something to a human who knows the business, and whether acting differently towards each group changes an outcome you can measure. Treat clusters as a hypothesis, never as a finding.

**Self-supervised learning is not available to you at the scale that matters.** The reason it works is the mountain of data and the months of GPU time. You will never pretrain a model, and you do not need to. What you need is to know that this is where the model's knowledge came from, which is also why it carries the internet's biases and why it has a knowledge cutoff — the two M00 failure modes, explained at their source.

**Reinforcement learning needs a cheap place to be wrong thousands of times.** Games have that. Simulations have that. Your payment flow does not. If someone proposes reinforcement learning for a system that touches real money or real users, the first question is where the failed attempts happen, and the honest answer is usually "to your customers".

**The labels you can derive are usually better than the labels you can collect.** Did the user click? Did the order get cancelled? Did the ticket get reopened within a week? Your product already records behaviour, and behaviour is a label that nobody has to be paid to write down. This habit is worth more to a student startup than any modelling technique in this module.

### Verify it

**Red flag: prediction promised from unlabelled data.** If a plan says "unsupervised" and the output is a prediction of something specific, it is wrong. Ask: what is the label, where does it come from, and how many examples of it exist? Either an answer appears or the plan collapses, and both outcomes are useful in the first ten minutes.

**Red flag: clusters presented as facts.** "Your users fall into four segments" is a result of the number you asked for. Make the AI tell you what happens with three and with six, and what observable behaviour distinguishes the groups. If nothing distinguishes them that you could act on, you have a chart rather than a finding.

**Red flag: reinforcement learning proposed for anything you are building this semester.** It is almost always the wrong tool at your scale. Ask where the thousands of failed attempts take place. If the answer involves real users, real money or real food, say no.

**Red flag: an invented reward.** When an AI proposes a reward signal, ask what behaviour would maximise that number while making the product worse. There is always one, and naming it out loud is the only cheap defence.

**Check the shape before the method.** Say out loud whether you want a bucket, a number, a group or a behaviour. Those four words map to classification, regression, clustering and reinforcement learning. Getting the shape right takes one sentence and prevents the two-week detour from the top of this chapter.

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
