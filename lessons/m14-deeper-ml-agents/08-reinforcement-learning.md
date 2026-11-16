---
id: m14-08-reinforcement-learning
module: m14-deeper-ml-agents
slug: reinforcement-learning
title: "Reinforcement Learning: Policies, Rewards and Reward Hacking"
order: 8
duration_min: 30
is_free: false
mission: false
volatility: low
last_reviewed: 2026-11-16
---

## Key learnings
- Name the five pieces of a reinforcement-learning problem and the two that are hardest to supply
- Run a contextual bandit, the one form of this that suits a real student startup
- Write a reward with its guardrail metrics, after naming how it will be gamed
- Explain the credit-assignment problem and why it makes RL so data-hungry
- Catch an invented reward whose failure mode nobody has said out loud

## Lesson

### The pain

M09 chapter 2 gave you reinforcement learning as the fourth kind of learning: a score after acting rather than an answer. It named policy, reward and reward hacking, and it gave you two rules that still hold — RL needs a cheap place to be wrong thousands of times, and when an AI proposes a reward, ask what behaviour would maximise that number while making the product worse. Chapter 5 of this module showed you DPO, which exists partly to avoid running an RL loop at all.

This chapter is the depth behind those warnings, plus the one thing in the family you can genuinely use.

Here is the failure. A team adds a learning recommender to their food-ordering app. The reward is clicks on recommended dishes, because clicks are easy to count. Within three weeks the system has learned to recommend the cheapest, most photographed, most clickable items. Clicks are up 34%. Orders are down. Average order value is down more. Two restaurants have left the platform because their food stopped appearing. Every single one of those outcomes is the system succeeding at the job it was given.

The second failure is quieter. Another team proposes RL for their support bot: reward it for closing conversations. Nobody runs it, which is lucky, because a bot rewarded for closing conversations learns to close conversations — by telling users their issue is resolved.

### The idea

**Reinforcement learning optimises a number you chose, extremely well, and your real objective is never that number.**

Five pieces define the problem. The **state** is what the system knows right now. The **action** is what it may do. The **policy** is the rule mapping states to actions — the thing being learned. The **reward** is the number that arrives afterwards. The **environment** is where actions have consequences.

Two of those five are hard, and they are not the algorithm. Supplying an **environment** where being wrong thousands of times is cheap is the first. Writing a **reward** that cannot be satisfied in a way you would hate is the second. Everything else is library code.

### The analogy

A new canteen server is paid a bonus based on a number, and nobody tells them what the canteen is for.

Pay by plates served and they will push the fastest dish on everyone and skip the ones that take time to assemble. Pay by tips and they will work the tables that tip, and the hostel students will wait. Pay by "no complaints" and they will stop writing complaints down. Pay by customer time-in-queue and they will serve the short orders first, forever, and a student ordering a full thali will never reach the front.

Each of those is the server doing exactly what you paid for. The canteen's actual objective — students fed well, within budget, without resentment — was never written on the payslip.

**Where the analogy breaks, twice.** A human server learns in a handful of shifts and understands the unwritten purpose even while gaming the written one. A policy needs tens of thousands of episodes and has no concept of purpose at all. And a server who finds a loophole knows it is a loophole; a policy that finds one simply has a higher score and will defend it by scoring well.

### Picture it

```
  FULL RL                            CONTEXTUAL BANDIT
  (sequential, delayed reward)       (one decision, fast reward)

   state ─────▶ POLICY               context ────▶ POLICY
     ▲            │                                 │
     │          action                           one action
     │            │                              (of 3 variants)
  environment ◀───┘                                 │
     │                                         reward now
   reward arrives LATER,                            │
   after 40 actions                                 ▼
     │                                      update the odds
     └──▶ which action earned it?           for that context
          (credit assignment)
                                      no credit-assignment
   needs: a simulator, 10,000s        problem; needs: your live
   of episodes, weeks                 traffic and a week
```

Full reinforcement learning must work out which of many past actions earned a delayed reward, which is why it needs a simulator and enormous numbers of attempts, while a contextual bandit makes one decision and sees the result immediately, so it can learn safely from live traffic.

### Credit assignment, and why RL is so hungry

A policy takes forty actions and then receives one number. Which action deserves the credit? That question — **credit assignment** — is the central difficulty, and the answer is statistical: play the whole sequence thousands of times with variations, and the actions that reliably precede good outcomes rise.

Two words follow from it. A **sparse** reward arrives rarely (you won the game) and is much harder to learn from than a dense one (you gained a point). A **delayed** reward arrives long after the action that caused it. Both multiply the number of episodes required, which is why the famous results come from games and simulators: a simulator can play a million episodes overnight and your payment flow cannot play one.

The other unavoidable tension is **exploration versus exploitation**. A policy that always takes its current best action never discovers a better one. So some fraction of decisions must be deliberately experimental — which, in a product, means some fraction of real users deliberately receive the option you believe is worse.

### The one version you can actually run

A **contextual bandit** strips RL down to what a live product can afford. There is no sequence: one decision, one immediate reward, no credit-assignment problem. You have several options — three subject lines, three onboarding orders, three dish arrangements — some information about the current user, and a measurable outcome within minutes.

It is an A/B test that reallocates traffic as evidence arrives. Where a fixed A/B test sends half your users to the loser for the whole experiment, a bandit shifts traffic towards what is winning while still exploring. For a student startup with a few hundred users a day, that difference is the difference between a conclusive result this month and never having enough traffic.

Three rules make it safe. The reward must be observable within minutes, not days. The options must all be acceptable to ship, because every one of them will reach real users. And you must declare **guardrail metrics** before you start — numbers that are not being optimised but must not get worse — because the bandit will happily win on its reward while damaging them.

### Where it lives

Inside the models you use: M09 chapter 11's preference training is RL's most consequential application, and chapter 5 of this module showed how DPO achieves the same end without the loop. In your own product: contextual bandits, sitting directly beside M07 chapter 9's A/B testing and significance material, and feeding M16's growth loops.

What does not belong in your product is a full RL agent acting in your live system. M09 chapter 2 was right: anything involving real money, real food or real users has nowhere cheap to be wrong ten thousand times.

### Words to own

| Term | Plain meaning | Example |
|---|---|---|
| State | What the system knows at this moment | Time, user, cart |
| Action | One of the things it may do now | Recommend dish 4 |
| Policy | The rule from state to action, being learned | The thing you train |
| Reward | The number that arrives after acting | A click, an order |
| Environment | Where the action has consequences | Simulator or live app |
| Episode | One run from start to an ending | A full session |
| Credit assignment | Deciding which past action earned the reward | The core difficulty |
| Sparse reward | Arrives rarely | Only on a completed order |
| Exploration | Deliberately not taking the current best action | 10% of traffic |
| Contextual bandit | One decision, immediate reward, no sequence | A learning A/B test |
| Reward hacking | Maximising the number while ruining the goal | Clickbait |
| Guardrail metric | Must-not-get-worse number you are not optimising | Order value |
| Proxy metric | What you can measure instead of what you mean | Clicks for value |

### Tradeoffs and gotchas

**Every reward you can measure is a proxy, and every proxy breaks under pressure.** Clicks stand in for interest, session length for engagement, closed tickets for resolution. Optimise any of them hard enough and the gap between the proxy and the real thing becomes the entire output of your system. This is Goodhart's law: a measure used as a target stops being a good measure.

**Write the gaming scenario before you write the reward.** One sentence: "a system that maximised this while ruining the product would do the following." If you cannot write that sentence, you do not understand your reward well enough to deploy it. If you can, you have just discovered your first guardrail metric.

**Guardrail metrics must be able to stop the experiment.** Declared in advance, with a threshold, and with an automatic halt. A guardrail nobody wired up is a note.

**If you have labels, you do not need RL.** This is the most common misapplication. When you know what the right action was for past situations, that is supervised learning: cheaper, faster, testable. RL is for when you only know how good the outcome was, not what you should have done.

**Exploration has a real cost to real people.** Ten percent of users receiving a worse experience is a product decision, not a technical detail, and it has an ethical dimension if the options differ in anything that matters — price, access, safety. Keep every arm genuinely shippable.

**A bandit with a delayed reward is a broken bandit.** If your outcome arrives a week later — retention, repeat purchase — a bandit cannot learn from it at your traffic. Use a fixed A/B test, accept the slower answer, and read M07 chapter 9 on significance rather than peeking at it daily.

### Verify it

The named failure is **an invented reward**: a plausible number proposed with no account of how it will be satisfied destructively.

1. **Make it write the gaming scenario first.** Before any design, one sentence on how a system would maximise this reward while making the product worse. An AI that cannot produce it has not thought about your product.
2. **Ask whether you have labels.** If past correct actions are recorded anywhere, demand the supervised alternative and the reason RL beats it.
3. **Ask where the thousands of wrong attempts happen.** If the answer is "with users", the answer is no, unless it is a bandit with shippable arms.
4. **Demand named guardrail metrics with thresholds and an automatic halt.** Three of them, chosen because the reward can be won at their expense.
5. **Check that the reward is observable in minutes for anything bandit-shaped.** A delayed reward means you need a fixed A/B test instead.
6. **Ask how the reward will be measured independently of the thing optimising it.** A policy evaluated on its own reward signal will always look excellent.

Red flags: a reward with no gaming scenario; RL proposed where labels exist; exploration on users for options you would not ship; no guardrail metric; a guardrail with no threshold or no halt; a delayed reward in a bandit; success reported only as the reward going up; and any full RL agent proposed to act in a live product with real money.

## Walkthrough
1. Pick one real decision your product makes repeatedly — which dish to show first, which onboarding step to lead with, which notification to send. Write the state, the actions and the reward in three lines.
2. Write the gaming scenario in one sentence: what would a system that maximised your reward while ruining your product actually do? Be specific and unflattering.
3. From that sentence, derive three guardrail metrics. For each one, write the threshold at which the experiment must stop, and who gets told.
4. Check whether you already have labels: does your data record what the right action *was* in past cases? If it does, write down what the supervised version of this problem looks like, and why you are not simply doing that.
5. Check the reward's delay. Time how long after the decision the outcome becomes observable. If it is more than a few minutes, stop and plan a fixed A/B test instead, using M07 chapter 9.
6. Confirm every option is shippable. If one arm is something you would not be comfortable showing a paying user, remove it.
7. Implement the simplest bandit: start with equal traffic to each option, record the reward per option per day, and shift traffic towards the winner while keeping at least 10% exploring.
8. Run it for a week. Each day, record the reward per arm, the three guardrail metrics, and the traffic split.
9. Deliberately interrogate the result: look at the guardrails before you look at the reward. If any guardrail moved against you, write down whether the reward win was real or was bought from a guardrail.
10. Write the decision-log entry: the reward, the gaming scenario, the guardrails, the traffic shift over time, and whether you kept the winning arm.
11. Finally, write one paragraph on why full reinforcement learning is still the wrong tool for your product, in your own words. You will be asked this at demo day.

## Talk to AI
```prompt
CONTEXT:      I am a student on an AI-native engineering program. I direct AI
              and verify its output; I do not write code by hand. I already
              know reinforcement learning as learning from a score rather than
              from answers, and I know the words policy, reward and reward
              hacking, so do not re-teach those. My team of three runs
              [product in two sentences] with [number] real users in India and
              about [number] relevant decisions per day. The repeated decision
              I want to improve is: [describe it in one sentence]. The options
              I am considering are: [list 3]. The outcome I care about really
              is: [state the TRUE objective, not the measurable proxy].

GOAL:         A safely designed contextual bandit on this decision, with a
              reward, three guardrail metrics with thresholds, and a written
              gaming scenario - or a clear conclusion that I should run a
              fixed A/B test or supervised learning instead.

REQUIREMENTS: 1. Before anything else, write the gaming scenario: in one
                 paragraph, exactly what a system maximising my proposed
                 reward would do that would ruin my product. Be specific to my
                 product, not generic
              2. From that, derive three guardrail metrics with a numeric
                 threshold each and a rule for halting the experiment
              3. Tell me whether my data already contains labels for what the
                 right action was. If it might, describe the supervised
                 alternative and tell me honestly which is better here
              4. Measure the delay: tell me how long after the decision my
                 reward is observable, and if it is more than a few minutes,
                 tell me to use a fixed A/B test instead and why
              5. Check each of my three options and tell me whether I would be
                 comfortable showing it to a paying user. Remove any I should
                 not
              6. Give me the simplest bandit design for [number] decisions a
                 day: how traffic starts, how it shifts, what minimum
                 exploration to keep, and how long before I can believe it
              7. Tell me what I must log per decision so I can audit this
                 later, and what I must NOT log under India's data protection
                 rules
              8. Tell me the three ways this experiment could show a reward
                 win that is not a real win, and the check for each

CONSTRAINTS:  Free tier only. My traffic is small, so tell me if it is too
              small for this to conclude anything. Plain English for a
              second-language reader. Every option must be acceptable to ship
              to a real paying user. Do not propose anything that needs a
              simulator or thousands of training episodes.

NON-GOALS:    Do not design a full reinforcement-learning agent. Do not
              propose a reward model or RLHF. Do not optimise anything that
              touches money arithmetic or permissions. Do not re-explain what
              a policy is. Do not suggest I collect more users first.

ACCEPTANCE:   - A gaming scenario specific to my product is written down
              - Three guardrail metrics exist with thresholds and a halt rule
              - The supervised alternative has been considered explicitly
              - The reward's observation delay is measured, not assumed
              - Every remaining option is one I would ship
              - A minimum exploration percentage is stated
              - A logging list exists, including what not to log
              - Three ways a fake win could appear, each with a check

ASK:          Start with the gaming scenario and nothing else. If you cannot
              write a specific one for my product, say so and tell me what you
              need to know. Then tell me whether I should be running a bandit
              at all at my traffic level.
```

## Stuck? Ask AI
```prompt
Here is the reward I want to optimise: [describe it]. Do not design anything. Instead, give me five different ways a system could maximise this number while making my product worse, ranked by how likely they are. For each one, tell me the guardrail metric that would catch it, the threshold, and how soon I would see it. Then tell me which single one would be hardest for me to notice, and why.
```

```prompt
Explain the difference between a fixed A/B test, a contextual bandit and full reinforcement learning, using one example decision from a food-ordering app. For each, tell me what data it needs, how fast it learns, what it risks, and the traffic volume it requires to conclude anything. Then ask me three questions about my own situation before telling me which one fits.
```

```prompt
Explain the credit-assignment problem with a concrete example of forty actions and one reward at the end. Then tell me why this makes reinforcement learning need tens of thousands of episodes, why games and simulators provide them, and why my live product cannot. Finish by explaining what a contextual bandit removes from the problem to make it safe on live traffic.
```

## Quiz

### Q1 · L3
A recommender rewarded on clicks drives clicks up 34% while orders and order value fall. What happened?
- (a) The system failed to learn
- (b) The system learned perfectly, and the reward was the wrong proxy ✅
- (c) The exploration rate was too high
- (d) The reward arrived too slowly to learn from
> This is reward hacking, and it is a specification failure, not a learning failure. Clicks stood in for value, and optimising the proxy hard enough broke the link between them.

### Q2 · L4
Which situation makes a contextual bandit the right tool rather than a fixed A/B test?
- (a) The outcome becomes observable a week later
- (b) The outcome is observable within minutes and traffic is limited ✅
- (c) You already know the correct action for every past case
- (d) You need statistical significance before changing anything
> A bandit's advantage is reallocating limited traffic as evidence arrives, which needs a fast reward. A week-long outcome cannot feed it, and recorded correct actions mean supervised learning.

### Q3 · L4
Why does full reinforcement learning need tens of thousands of episodes?
- (a) Because rewards are always noisy
- (b) Because credit for a delayed reward must be spread across many past actions statistically ✅
- (c) Because policies have more parameters than other models
- (d) Because exploration is random
> One number arrives after many actions, so the only way to learn which action mattered is repetition with variation. That is why simulators carry the famous results and live products cannot.

### Q4 · L4
A team proposes rewarding their support bot for closing conversations. What is the single best first response?
- (a) Ask for a higher exploration rate
- (b) Ask them to write what a system maximising this would do to ruin the product ✅
- (c) Ask for more training episodes
- (d) Ask which algorithm they plan to use
> The gaming scenario writes itself — a bot that declares issues resolved — and producing it before any code is the cheapest defence available. It also hands you your first guardrail metric.

### Q5 · L5
Your bandit shows a clear reward win. Which check should you perform before believing it?
- (a) Re-run it with a higher exploration rate
- (b) Confirm the reward is still rising after another week
- (c) Read the guardrail metrics first and ask whether the win was bought from them ✅
- (d) Compare it against a different algorithm
> A reward can almost always be improved at the expense of something you are not optimising. Guardrails exist precisely for this moment, and they have to be read before the reward, not after.

## Go deeper
- [Spinning Up in Deep RL: key concepts and terminology](https://spinningup.openai.com/en/latest/spinningup/rl_intro.html)
- [Concrete Problems in AI Safety: reward hacking and side effects](https://arxiv.org/abs/1606.06565)
