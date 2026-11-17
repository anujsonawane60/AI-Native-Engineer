---
id: m16-07-growth-loops-vs-funnels
module: m16-users-analytics-growth
slug: growth-loops-vs-funnels
title: "Growth Loops vs Funnels"
order: 7
duration_min: 15
is_free: false
mission: false
volatility: low
last_reviewed: 2026-11-16
---

## Key learnings
- Tell a loop from a funnel by asking whether the output becomes the next input
- State a loop's cycle time and branching factor before claiming it works
- Place your funnels inside the loop, as the diagnostic for each step
- Accept that a loop below one still pays, by lowering the cost of each new user
- Refuse to build a loop before the product retains anybody

## Lesson

### The pain

A team finishes chapter six in good shape: activation 61%, retention flat at 30%, no leaks left. Then someone asks how many users they will have in two months. Honest answer: however many we personally ask. Every user so far arrived because a team member asked, so on exam week growth is zero. Five chapters improved a conveyor belt, and a better belt does not produce more things to move.

### What M07 covered, and what is new here

M07's chapter eight was pure funnel thinking — the right frame for one user's journey, the wrong frame for how the next user arrives. This chapter adds the other shape.

### The idea

**A funnel consumes users; a loop produces them: a loop's output becomes its own next input, so each turn makes the next turn bigger.**

The test: does running this once bring in a new person without you doing anything? If no, it is a funnel with an arrow drawn back.

Four parts make a loop. **Input:** a new user arrives. **Action:** they do something in the product. **Output:** that action produces something a non-user can see, like a shared link. **Return:** that output brings in a new user.

Two numbers describe it. **Cycle time** is how long one turn takes: a user arriving, then the new user they caused. **Branching factor** is new users brought per existing user per cycle. Both matter: 0.8 at three days is a different business from 0.8 at two months, so always state the pair.

Never quote a branching factor alone: every arrow is a conversion with a drop-off, so the factor is those rates multiplied, which is why it is usually small.

Almost no student product exceeds one, and that is fine. At 0.35, a hundred hand-recruited users produce about fifty more before decaying out — half your manual effort, given back.

### The analogy

The canteen's lunch queue is a funnel: a hundred and eighty students walk in, a hundred and forty-two buy, and tomorrow it needs a fresh queue. The loop is different. A student posts the Thursday thali in the hostel group, two friends come on Friday, and one of them posts a photo too. Friday's customers were produced by Thursday's.

**Where the analogy breaks:** the campus has eighteen hundred students and no more. Even an excellent loop runs out of people — it **saturates** — and the factor then falls towards zero however good the thali is.

### Picture it

```
  THE LOOP, WITH ITS TWO NUMBERS

        ┌──────────── new user arrives ◀──────────┐
        │                                         │
        ▼                                         │
   places an order                          a friend taps
        │                                    the link
        ▼                                         ▲
   shares the order link in                       │
   a WhatsApp group ──────── seen by 9 people ────┘

   cycle time       ~4 days   (order ▶ friend's first order)
   branching factor 0.35      (each user brings 0.35 users)
   so               100 users ▶ 35 ▶ 12 ▶ 4 ▶ decays out
```

A loop with a branching factor below one decays: a hundred users produce thirty-five, then twelve, then four, and the loop stops unless new users keep being added from outside.

### Where it lives

Every arrow above is a rate you can already measure — share rate, people reached, tap rate, activation — and chapters two to six are the instruments.

Chapter one is the outside input every sub-one loop amplifies; chapter four is the gate, and not negotiable; chapter eight compares the loop families; chapter nine tests one loop step; M15's attribution chapter is the same honesty problem; M17 turns the factor into money, since a loop returning half your effort halves acquisition cost.

### Words to own

| Term | Plain meaning | Example |
|---|---|---|
| Funnel | Ordered steps one user passes through | Signup to first order |
| Loop | A cycle whose output becomes its own input | Order ▶ share ▶ new order |
| Loop input | What enters one turn of the cycle | A new activated user |
| Loop output | What the action produces for a non-user | A shared order link |
| Cycle time | How long one turn takes | About four days |
| Branching factor | New users produced per existing user | 0.35 |
| Compounding | Each turn larger than the last | Factor above one |
| Decay | Each turn smaller than the last | Factor below one |
| Saturation | The loop has reached everyone reachable | One hostel, finished |
| Engagement loop | A cycle that brings existing users back | Reminder ▶ return |

### Tradeoffs and gotchas

**Acquisition and engagement loops differ.** A notification that brings an existing user back is valuable but grows nothing. Only output reaching a non-user acquires.

**A loop needs several cycles to show itself.** A four-day cycle needs a month, a three-week cycle a term; a factor from five days is noise.

**WhatsApp loops are invisible by default.** A pasted link carries no referrer, so analytics calls the arrival "direct" — dark social, which dominates Indian student products. The fix is a per-user code inside the link; anyone who retypes the URL is still lost.

**Loops run backwards too.** A bad first experience is told to the same nine people; complaints are more shareable than satisfaction.

**Saturation looks like failure.** When a loop finishes a hostel the factor collapses; the response is a new population, not a redesigned loop.

**You cannot loop your way out of bad retention.** A loop on a curve that never flattens pours strangers into a product that loses them, and only churn grows.

### Verify it

Ask AI for a growth loop and you get four boxes and a curved arrow. Most are not loops.

1. **Check the output reaches a non-user.** Seen only by account holders, it is an engagement loop at best.
2. **Demand the cycle time.** A loop without one cannot be planned, measured or defended.
3. **Demand the factor as a product of named rates.** Rates not listed means the number was invented.
4. **Check the data covers several cycles.** One week on a four-day loop is barely two.
5. **Ask how a WhatsApp share would be attributed.** No code in the link, no measurable loop.
6. **Ask what retention the loop assumes.** Refuse any loop on a curve that has not flattened.

Red flags: a factor quoted to two decimals from a fortnight of data; every organic signup credited to the loop; an incentive attached before the loop works; a diagram whose output is a push notification.

## Walkthrough
1. Draw your current acquisition as it actually is today, honestly. If every user arrives because a team member asked, draw that, with the number of asks per user from chapter one.
2. Write the one-sentence test at the top of the page: does running this once produce something that brings in a new person without us doing anything?
3. List every output your product already produces that a non-user could possibly see — a shared link, a screenshot worth sending, a public page, a message your product sends on a user's behalf.
4. For each candidate output, write the full four-part loop: input, action, output, return. Discard any where the output never reaches a non-user.
5. For your strongest candidate, name the four conversion rates along the cycle and write down which ones you can measure today and which you cannot.
6. Estimate the cycle time in days, from a user arriving to the new user they cause arriving. If you cannot estimate it, say so rather than guessing.
7. Compute the branching factor as the product of your four rates, and write it beside the weakest of the four.
8. Describe in words what happens to a hundred users over four cycles at that factor, and say plainly whether the loop decays.
9. Work out how much manual effort the loop gives back: how many fewer asks per hundred users, using chapter one's ask-to-retained-user rate.
10. Decide how a share will be attributed before you build anything, and write the exact link format including the per-user code.
11. Check the gate: write your flattening point from chapter four beside the loop, and state whether retention justifies building this yet.

## Talk to AI
```prompt
CONTEXT:      I am a student in a team of three in India. I direct AI and
              verify its output; I do not write code. Our product is live:
              [describe it in two sentences]. We have [number] active
              users. Every user so far arrived because we asked them
              personally - our ask-to-retained-user rate is about [number]
              asks per user. From chapter four, our retention flattens at
              [number]% at week [number], measured from first value. Our
              activation rate is [number]% within [window]. Our users are
              students in India; sharing happens almost entirely in
              WhatsApp groups, which carry no referrer. The outputs our
              product already produces that a non-user could see are:
              [list them, or say "none that we know of"].

GOAL:         To know whether a real acquisition loop exists in my product,
              and if so to state its cycle time and branching factor as a
              product of measurable rates.

REQUIREMENTS: 1. Apply one test to each of my candidate outputs: does this
                 reach a person who is not already a user? Tell me which
                 of my candidates fail that test and are therefore
                 engagement loops or funnels with a curved arrow.
              2. For the strongest surviving candidate, write the loop as
                 four parts - input, action, output, return - in my
                 product's own language.
              3. Break the cycle into its conversion rates, named
                 individually, and tell me for each one whether my current
                 event taxonomy can measure it. Name any event I would
                 need to add.
              4. Estimate the cycle time in days and explain what you based
                 that on. Then tell me the minimum number of days of data
                 before a branching factor means anything.
              5. Express the branching factor as the product of those
                 rates, using plausible values, and tell me which single
                 rate is most likely to be the bottleneck.
              6. Describe in words, with no formulas, what happens to 100
                 users over four cycles at that factor. Then tell me how
                 many manual asks the loop saves me per 100 users.
              7. Tell me how to attribute a WhatsApp share that carries no
                 referrer, including the exact shape of the link, and what
                 fraction of shares I should expect to lose anyway.
              8. Given my retention number, tell me plainly whether I
                 should build this loop yet or fix retention first.
              9. Name two ways this loop could run backwards and spread a
                 bad experience.

CONSTRAINTS:  Free tier only. No formulas, no statistics notation -
              explain compounding and decay in words and small numbers.
              Never quote a branching factor without listing the rates it
              came from. Do not propose paid acquisition. Plain English,
              every term explained on first use.

NON-GOALS:    Do not compare referral, content and paid loops in detail;
              the next chapter does that. Do not design an experiment. Do
              not propose a referral incentive. Do not write code and do
              not modify any files.

ACCEPTANCE:   - Every candidate was tested against "does it reach a
                non-user"
              - My loop is written as four named parts
              - Each conversion rate is named and marked measurable or not
              - A cycle time in days is stated, with its basis
              - The branching factor is a product of listed rates
              - I know whether my retention justifies building this yet

ASK:          First ask me up to five questions about my product's outputs,
              my sharing behaviour and my retention. Then answer
              requirement 1 only, and stop.
```

## Stuck? Ask AI
```prompt
Here is a growth diagram I was given: [paste or describe it]. Tell me whether this is genuinely a loop or a funnel with an arrow drawn back to the start, and justify your answer using only the test of whether the output reaches someone who is not already a user. Then tell me what would have to change to make it a real loop.
```

```prompt
Explain compounding and decay in a growth loop using a college canteen and a hostel WhatsApp group, with small whole numbers and no formulas. Show me what happens over four cycles at a branching factor of 0.3, then at 1.0, then at 1.4 - and tell me which of the three is realistic for a student product and why.
```

```prompt
Almost all sharing for my product happens in WhatsApp groups, and my analytics records those arrivals as direct traffic. Explain what dark social is and why it breaks loop measurement. Then give me three ways to attribute those shares, with the tradeoff of each, and tell me what share of shares I will still lose under the best option.
```

## Quiz

### Q1 · L4
Which of these is a genuine acquisition loop?
- (a) A push notification that brings a lapsed user back to order again
- (b) A shared order link posted in a group, which a non-user taps and then orders ✅
- (c) A better onboarding flow that lifts activation from 29% to 61%
- (d) A weekly email to existing users summarising their orders
> Only option (b) produces an output that reaches someone who is not yet a user. The others are an engagement loop, a funnel improvement and another engagement loop — all useful, none of them growth.

### Q2 · L4
Your loop has a branching factor of 0.35. What does that mean for a hundred recruited users?
- (a) The loop will produce thirty-five users a week indefinitely
- (b) The loop decays, producing roughly fifty extra users across several cycles ✅
- (c) The loop compounds slowly and will eventually exceed a hundred
- (d) The loop is broken and should be rebuilt
> Below one, each turn is smaller than the last: 35, then 12, then 4. It still returns about half your manual recruiting effort, which is why a sub-one loop is worth building and should not be called a failure.

### Q3 · L5
Your branching factor was 0.9 for three weeks inside one hostel and has now fallen to 0.1. What is the most likely explanation?
- (a) The loop's mechanics broke and need rebuilding
- (b) The loop saturated the population it could reach ✅
- (c) The cycle time increased
- (d) Retention fell, so fewer users share
> A finite population is exhausted by a working loop, and the collapse looks identical to failure. The response is a new population — another hostel, another college — rather than a redesign of a loop that did its job.

### Q4 · L4
Your retention curve from first value has not flattened and still falls towards zero at week six. AI proposes a referral loop with an incentive. What is the right response?
- (a) Build it; more users will make the retention data clearer
- (b) Refuse until retention flattens, because a loop would accelerate churn ✅
- (c) Build it but without the incentive
- (d) Build it and measure referral quality instead of retention
> A loop amplifies whatever the product does, and a product that loses everyone amplifies loss. Growing a leaking product faster raises the churn number and the acquisition cost and improves nothing.

### Q5 · L3
Why must a loop's branching factor be reported as a product of named rates rather than as a single number?
- (a) Because analytics tools cannot compute it directly
- (b) Because the individual rates show which step is the bottleneck, and a lone number cannot be checked ✅
- (c) Because branching factors are always estimates
- (d) Because the platform requires percentages rather than ratios
> "Our loop is weak" is not actionable. "Only 9% of activated users ever share, which is the smallest of our four rates" names the step to work on and lets anybody verify the arithmetic.

## Go deeper
- [Reforge: growth loops are the new funnels](https://www.reforge.com/blog/growth-loops)
- [Andrew Chen: the law of shitty clickthroughs and loop saturation](https://andrewchen.com/the-law-of-shitty-clickthroughs/)
