---
id: m16-07-growth-loops-vs-funnels
module: m16-users-analytics-growth
slug: growth-loops-vs-funnels
title: "Growth Loops vs Funnels"
order: 7
duration_min: 25
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

A team finishes chapter six in good shape. Activation is 61%, retention flattens at 30%, the funnel has no embarrassing leaks left. They present this and someone asks how many users they will have in two months.

The honest answer is: however many we personally ask. Every single user so far arrived because a team member sent a message or stood in a classroom. The product is good and the machine does not exist. On the week the team has exams, growth is zero, and every number in the module stops moving.

They have spent five chapters improving a conveyor belt. A conveyor belt moves what you put on it. Nothing about a better conveyor belt produces more things to move.

### What M07 covered, and what is new here

M07 did not mention growth loops. Its chapter eight is entirely funnel thinking — ordered steps, drops, windows — which is the right frame for one user's journey through your product and the wrong frame for how the next user arrives.

This chapter introduces the other shape. Chapters one to six of this module were all funnel work: the ask ladder, the signup funnel, activation. They stay useful. What changes is where they sit: inside a loop, as the diagnostic for each of its steps.

### The idea

**A funnel consumes users; a loop produces them. A loop is a cycle whose output becomes its own next input, so each turn makes the next turn bigger.**

That is the entire distinction, and it is a test you can apply in one sentence: does running this once produce something that brings in a new person without you doing anything? If yes, it is a loop. If no, it is a funnel with an arrow drawn back, which is what most "growth loop" diagrams turn out to be.

### Loop anatomy

Every loop has four parts, and naming all four is what separates a loop from a hope.

**Input.** A new user arrives.
**Action.** They do something inside the product.
**Output.** That action produces something visible to a person who is not yet a user — a shared order link, a public page, a WhatsApp message, a referral code.
**Return.** That output brings a new user in, and the cycle starts again.

Two numbers describe the loop's behaviour. **Cycle time** is how long one turn takes: from a user arriving to the new user they caused arriving. **Branching factor** is how many new users each existing user brings in one cycle.

Both are needed. A loop with a branching factor of 0.8 and a cycle time of three days is a different business from a loop with the same factor and a cycle time of two months.

### The analogy

The canteen's lunch queue is a funnel. A hundred and eighty students walk in, a hundred and forty-two buy, thirty-eight leave. Tomorrow the canteen needs a fresh queue, and nothing about today's queue produces tomorrow's.

The canteen's loop is different. A student photographs the Thursday thali and posts it in the hostel group. Two friends come on Friday. One of them posts a photo as well. The canteen did nothing on Friday, and Friday's customers were produced by Thursday's.

**Where the analogy breaks:** the campus has eighteen hundred students and no more. Even an excellent loop runs out of people to reach — it **saturates** — and when it does, the branching factor falls towards zero no matter how good the thali is. Your addressable market is also finite, and far smaller than you think during the first good week. A loop that worked beautifully inside one hostel will appear to break when it has finished that hostel, and the mistake is to conclude that the loop was never real.

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

### Below one still pays

Almost no student product will have a branching factor above one. A loop above one compounds without help and is what people mean by "viral"; it is rare, usually short-lived, and not a reasonable target for a first product.

A loop at 0.35 is still worth building, and the reason is arithmetic you can do in words. Without a loop, every single user costs you one manual ask from chapter one — and at chapter one's rates, roughly fifteen asks per retained user. With a branching factor of 0.35, every hundred users you recruit by hand produce about fifty more over several cycles before decaying out. That is a fifty per cent discount on the most expensive and least pleasant work your team does.

So the honest framing is not "does this loop go viral" but "how much of my manual effort does this loop give back". That is a number you can report and defend, and M17 turns it into a cost.

### Funnels live inside loops

Every arrow in a loop diagram is a conversion with a drop-off, which means every arrow is a small funnel you already know how to measure.

For the loop above: what share of activated users share a link at all; how many people see each shared link; what share of those tap it; what share of tappers activate. The branching factor is those four rates multiplied together, which is why it is usually small — four steps at fifty per cent each leave you at about six per cent.

This is the practical payoff of the chapter. "Our loop is weak" is not actionable. "Only 9% of activated users ever share, and that is the smallest of our four rates" is a brief. Chapters two to six are the instruments you point at each arrow, one at a time.

### Where it lives

Chapter one is the outside input that every sub-one loop depends on: a loop amplifies, so it needs something to amplify, exactly as that chapter said about SEO and referrals. Chapter four is the gate — a loop built on a product with no flattening point simply moves people through faster on their way out, so retention comes first and this is not negotiable. Chapter eight compares the three loop families in detail, and chapter nine is how you test a change to a single loop step once you have the traffic.

M15's content engine is a content loop measured properly, and its attribution chapter is the same honesty problem. M17 is where the branching factor becomes money: a loop that returns half your manual effort halves your acquisition cost, which is what makes the unit economics work.

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

**Acquisition loops and engagement loops are different things.** A notification that brings an existing user back is an engagement loop: valuable, and it does not grow your user count. Only a loop whose output reaches a non-user is an acquisition loop. Teams conflate them and then wonder why a healthy loop produced no growth.

**A loop needs several cycles to show itself.** With a four-day cycle you need a month; with a three-week cycle you need a term. A branching factor computed from five days of data is noise, and this is the commonest way teams declare a loop dead or alive too early.

**WhatsApp loops are invisible by default.** A link pasted into a chat carries no referrer, so your analytics records the arrival as "direct" and you cannot tell the loop from luck. This is dark social, and it is the dominant sharing channel for Indian student products. The only fix is to put the attribution in the link itself — a per-user code or parameter — and to accept that anyone who retypes the URL is lost to you.

**Loops run backwards too.** A user who has a bad first experience tells the same nine people in the same group. Negative loops have the same mechanics and move faster, because complaints are more shareable than satisfaction.

**Saturation looks exactly like failure.** When a loop finishes a hostel, a class or a campus, the factor collapses. The correct response is a new population, not a redesigned loop.

**You cannot loop your way out of bad retention.** If your curve never flattens, a loop pours strangers into a product that loses them, and the only thing that grows is your churn number. Chapter four is the prerequisite and chapter ten is the consequence.

### Verify it

Ask AI for a growth loop and you will get a diagram with four boxes and an arrow curving back to the first. Most of them are not loops.

1. **Check that the output reaches a non-user.** If the last box's product is seen only by people who already have accounts, this is an engagement loop at best and a funnel with a curved arrow at worst.
2. **Demand the cycle time.** **A loop described without a cycle time** cannot be planned, measured or defended, and its absence usually means nobody thought about how long one turn takes.
3. **Demand the branching factor as a product of named rates**, not as a single confident number. If the four rates are not listed, the number was invented.
4. **Check whether any claimed factor came from fewer cycles than the cycle time allows.** One week of data on a four-day loop is one and a half cycles.
5. **Ask how a share through WhatsApp would be attributed.** If the answer does not involve a code or parameter carried in the link, the loop cannot be measured and any later claim about it is a guess.
6. **Ask what retention the loop assumes.** A loop proposed for a product whose curve has not flattened is a proposal to accelerate churn, and it should be refused until chapter four's number exists.

Red flags: a loop with no cycle time; a branching factor quoted to two decimal places from a fortnight of data; every organic signup attributed to the loop; an incentive attached before the loop has been shown to work at all; and a diagram in which the output is a push notification to existing users.

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
