---
id: m17-05-unit-economics
module: m17-distribution-revenue
slug: unit-economics
title: "Unit Economics: CAC, LTV, Margin and Payback"
order: 5
duration_min: 35
is_free: false
mission: true
volatility: low
last_reviewed: 2026-11-16
---

## Key learnings
- Compute gross profit per subscriber per month, with every cost line visible
- Put the founders' time into CAC and watch the number change by four times
- Replace infinite-horizon LTV with bounded contribution from a real retention curve
- Use payback period, not LTV to CAC, as the number that constrains you
- Catch the four economics lies: revenue as margin, free labour, two weeks of retention, and a missing inference line

## Lesson

### The pain

A team presents at the month-four review. Their slide says: CAC ₹75, LTV ₹3,980, a 53-times return. The room nods. It is the best-looking slide of the afternoon. Then someone asks four questions.

Where did CAC come from? Printing, samosas and one boosted post — ₹3,000 for forty customers, with the sixty hours they spent at a canteen stall left out because "our time is free". Where did LTV come from? Monthly revenue of ₹199 divided by a 5% churn rate. Where did 5% come from? Two weeks of data from one cohort of forty. And the gross margin? Nobody has computed one, because the AI feature's inference cost has never been subtracted from the ₹199.

The honest version of the same slide, from the same team's real numbers: CAC ₹300, six-month contribution ₹430, a ratio of 1.4, and a payback period of four and a half months. That is not a disaster — it is a business that might work — but it is a different business from the one on the slide, and every decision made from the slide was made about a company that does not exist.

### The idea

**Unit economics asks one question: does one customer pay back more than one customer costs, and how fast?**

Four numbers answer it, computed in this order, because each depends on the last.

**Gross profit per customer per month** — revenue minus every cost of serving them. **CAC** — what it cost to get one paying customer, including your time. **Contribution over a bounded horizon** — gross profit times how long they actually stay, from a real retention curve. **Payback period** — months of gross profit needed to recover CAC.

The boundary with M16 is worth stating plainly. **M16 measures behaviour: who activated, who came back, what the retention curve looks like, how many churned. M17 turns that behaviour into money.** If your retention curve is not instrumented, nothing here can be computed — go back and instrument it.

### The analogy

The canteen owner knows two numbers by heart, and they are the only two he needs.

The first is what a plate costs him: ₹51 against a ₹60 price, so ₹9 a plate. The second is what a regular is worth. He printed ₹2,000 of flyers and eighty people became regulars, so ₹25 to get one. A regular eats twelve plates a month, which is ₹108 of margin, so he recovers ₹25 in the first week and can print more flyers tomorrow without thinking.

That is why he sleeps well and the team with the 53-times slide does not. His ₹9 is measured, his ₹25 includes the nephew he paid to hand out flyers, and he has watched the same regulars for two years.

**Where the analogy breaks:** his customers leave and come back with no record, so he can only guess at retention. You can measure it per user, to the day, which removes your excuse for guessing — but you have only a few weeks of history, which is exactly why LTV is the most abused number in a student pitch. Measurable and short is a different problem from unmeasurable and long, and it needs a different kind of honesty.

### Number one: gross profit per subscriber

Start here, because everything else is built on it. Revenue is not margin.

```
  GROSS MARGIN ON ONE SUBSCRIBER, PER MONTH

  revenue, ₹199 plan                       ₹199
  gateway fee, about 2%                     -₹4
  inference, cost per active user          -₹59
  hosting, database, email, per user        -₹6
  ---------------------------------------------
  gross profit per subscriber per month    ₹130
  gross margin                              65%
```

Every cost of serving one subscriber for a month, subtracted from the price, leaving ₹130 of gross profit at 65%.

The line missing from most student calculations is the inference line, and it is usually the largest. Chapter 6 is entirely about where that ₹59 comes from and why it is often worse. If your table has no inference line and your product has an AI feature, you do not have a gross margin — you have a price.

Note what is *not* here: rent, salaries, the laptop, the hours spent building. Those are fixed costs, and mixing them in produces a number that changes every time you hire someone, which makes it useless for deciding whether a customer is worth acquiring.

### Number two: CAC, including your time

**CAC** is acquisition spend divided by new paying customers in the same period. Two things go wrong: people divide by signups instead of paying customers, and people leave out labour.

```
  CAC FOR ONE MONTH OF ACQUISITION

  item                                    spend
  200 flyers printed                       ₹600
  samosas for the canteen stall            ₹900
  one boosted Instagram post             ₹1,500
  3 founders x 20 h x ₹150/h             ₹9,000
  ---------------------------------------------
  total spend                           ₹12,000
  new PAYING customers                        40

  CAC, founders' time excluded     ₹3,000/40 = ₹75
  CAC, founders' time included    ₹12,000/40 = ₹300
```

The same month of acquisition produces a CAC of ₹75 or ₹300 depending entirely on whether the founders' sixty hours are counted.

Your time is not free. It is the scarcest thing you have, you are spending it instead of building, and a channel that only works when three people stand in a canteen stops the day you stop standing. Pick a notional hourly rate — whatever a part-time job in your city pays is defensible — write it down, and use it consistently.

One more distinction. **Blended CAC** divides all spend by all new customers, including those who arrived free by word of mouth. **Paid CAC** divides only paid spend by the customers that spend produced. Blended flatters you and hides a bad channel inside a good average; chapter 7 is where the channel-by-channel version lives.

### Number three: contribution, not LTV

`LTV = margin ÷ churn` is a real formula and poison in your hands, because it assumes an infinite horizon and your churn estimate comes from a few weeks. Divide by a slightly wrong small number and the answer moves enormously. Use a **bounded horizon** instead: take a real cohort, follow it for as many months as you actually have, and multiply.

```
  SIX-MONTH CONTRIBUTION AND PAYBACK, 40-USER COHORT
  CAC to recover: ₹300 per original user

  month  paying  of orig  profit/orig  cumulative
      1      40     100%         ₹130        ₹130
      2      26      65%          ₹85        ₹215
      3      20      50%          ₹65        ₹280
      4      17      43%          ₹55        ₹335
      5      15      38%          ₹49        ₹384
      6      14      35%          ₹46        ₹430
```

A real cohort's retention curve multiplied by gross profit, accumulating to ₹430 of contribution per original user over six months, with CAC of ₹300 recovered during month four.

Say "six-month contribution per customer: ₹430" instead of "LTV". It is defensible, computed from your own data, and in M18 nobody can take it apart. With three months of data, say three-month contribution.

### Number four: payback, the one that constrains you

```
  THE SAME COHORT, TOLD TWO WAYS

                        the pitch     the ledger
  CAC                         ₹75           ₹300
  monthly value per user     ₹199           ₹130
  churn used                   5%     real curve
  LTV or contribution      ₹3,980           ₹430
  ratio to CAC                 53x           1.4x
  payback               0.4 months     4.5 months
```

Four separate errors — free labour, revenue as margin, an invented churn rate and an infinite horizon — compound into a number thirty-eight times too good.

The ratio of 1.4 is survivable and needs work. But the number that will actually stop you is the payback period of four and a half months, because it is about **cash**, not returns. Spending ₹12,000 this month means ₹12,000 you do not have for four and a half months. At a student's cash level that is the binding constraint, and a beautiful ratio will not pay a bill. Shorter payback beats higher ratio almost every time at your scale.

### Where it lives

Chapter 2 set the price and therefore the revenue line. Chapter 3's structure decides whether contribution even has a curve — a one-time product has none. Chapter 6 supplies the inference line, which most often turns a good-looking margin into a bad one. M16 supplies the retention curve and the churn measurement; M05 and M13 supply infrastructure costs.

Forwards: chapter 7 computes CAC per channel, chapter 11 puts gross margin and payback on the weekly scorecard, and M18 is where these four numbers get questioned in front of people. The only defence is that you computed them yourself and can show every line.

### Words to own

| Term | Plain meaning | Example |
|---|---|---|
| Gross profit | Revenue minus the cost of serving | ₹130 per month |
| Gross margin | Gross profit as a percentage | 65% |
| Variable cost | Cost that grows per customer | Inference, gateway fee |
| Fixed cost | Cost that does not | Your domain, your time |
| Contribution | Gross profit over a bounded horizon | ₹430 over six months |
| CAC | Spend divided by new paying customers | ₹300 |
| Blended CAC | All spend over all new customers | Flatters you |
| Paid CAC | Paid spend over customers it produced | The honest number |
| Payback period | Months of gross profit to recover CAC | 4.5 months |
| Cohort | Users grouped by when they joined | The 40 from March |
| Runway | Months of cash left at current burn | The real constraint |

### Tradeoffs and gotchas

**A high ratio with a long payback can still kill you.** Returns are about eventual value; payback is about cash this month. Student teams run out of cash, not out of returns.

**Blended CAC hides a dead channel.** If word of mouth brings thirty customers free and a ₹9,000 campaign brings ten, your blended CAC looks fine and the campaign cost ₹900 a customer.

**Discounts and refunds reduce ARPU after the fact.** A three-month half-price launch offer means your first cohort's gross profit is not ₹130, it is about ₹30.

**Annual plans distort both ends.** Cash arrives in month one and nobody can churn until month twelve, so your retention curve looks magnificent for a year. State which effect you are seeing.

**Fixed costs do not belong in gross margin, and they do not disappear either.** Keep them on chapter 11's scorecard, against runway.

### Verify it

AI is good at this arithmetic and bad at refusing to do it without data. It will produce a complete, confident model from assumptions it invented, labelled in a clause you skim past.

**Red flag, the headline one: LTV computed from a retention curve with two weeks of data.** Ask how many months of real retention the number rests on, and refuse any horizon longer than that. Two weeks supports a two-week observation and no LTV at all.

**Red flag: CAC that excludes the founders' time.** Ask directly: how many hours, at what rate, and is it in the numerator? A four-times error hides here routinely.

**Red flag: a margin calculation that forgets inference cost.** If the cost lines are gateway fees and hosting only and your product makes model calls, the biggest line is missing. This is the commonest fatal omission in an AI product's economics.

**Red flag: LTV computed from revenue rather than gross profit.** ₹199 is not what a customer is worth to you; ₹130 is.

**Red flag: `margin ÷ churn` with a churn rate from fewer than forty users.** The formula divides by a small, noisy number, so the output is dominated by the noise.

**Red flag: a model with no payback line, or a benchmark quoted as a fact.** A ratio alone hides the cash constraint, and "a healthy LTV to CAC is 3:1" is repeated everywhere with no source attached to your market. Ask for the source; the absence of one is your answer.

The test that settles it: hand your four numbers to a teammate and ask them to find the line that is missing or invented. If it takes under two minutes, M18 will take under one.

## Walkthrough
1. Build your gross margin table for one subscriber for one month: price, gateway fee, inference cost per active user from chapter 6, and every other per-user cost. Show every line.
2. Compute gross profit and gross margin. If the margin is above 90% and you have an AI feature, you have forgotten a line — go back.
3. List every rupee you spent acquiring customers last month, including printing, food, transport, boosts and tools.
4. Add the labour line: total founder hours spent on acquisition, times a notional hourly rate you write down and justify in one sentence.
5. Count new *paying* customers in that same period, not signups. Divide. Write both CACs — with and without labour — side by side.
6. Pull your retention curve from your M16 instrumentation for one real cohort. Write the number still paying in each month you actually have data for.
7. Build the contribution table: for each month, the share of the original cohort still paying, gross profit per original user, and the running cumulative.
8. Find the month where the cumulative crosses your CAC. That month is your payback period. Write it as a number of months.
9. Build the two-ways table: your honest four numbers in one column, and the flattering version in the other, so you can see what each error is worth.
10. Write down the one number you most want to change and the cheapest lever for it: price, inference cost, retention, or acquisition cost.
11. Record the whole thing in your decision log with today's date, the number of months of retention data it rests on, and the trigger to rebuild it.

## Talk to AI
```prompt
I need an honest unit-economics model for my startup. You must not invent any number. If I have not given you an input, tell me to go and measure it rather than assuming a value.

My price and billing period: [from chapter 2]
What a user pays in practice, after any launch discount: [be honest]
My payment gateway's fee, as a percentage: [read from their page, with the date]
My inference cost per active user per month, in rupees: [from chapter 6, or "not computed"]
My other per-user costs per month: [hosting, database, email, SMS - list them]
Acquisition spend last month, itemised: [every rupee]
Founder hours spent on acquisition last month: [number]
The hourly rate I am valuing founder time at, and why: [number and reason]
New PAYING customers last month: [number - not signups]
My retention curve, month by month, for one real cohort: [paste what I have]
How many months of real retention data I have: [number]
Cash in hand: [rupees]

Do all of this, in order:

1. Build my gross margin table with one line per cost, and tell me which line I have most likely forgotten. If my inference cost is "not computed", stop and tell me the model cannot be built yet.

2. Compute gross profit and gross margin per subscriber per month. If my margin looks implausibly high for a product with an AI feature, say so.

3. Compute CAC twice - with and without founder time - and show both. Tell me what the difference means for whether this acquisition channel can survive without us standing in a canteen.

4. Tell me whether my CAC is blended or paid, and what it would hide if it is blended.

5. Using ONLY the months of retention data I actually gave you, build a contribution table: month, share of the original cohort still paying, gross profit per original user, and the cumulative. Do not extrapolate beyond my data. If I asked you to, refuse and explain why.

6. Find my payback period from that cumulative column and state it in months. Then tell me what that payback means for how much I can afford to spend on acquisition next month given my cash.

7. Show me the flattering version of the same numbers - revenue instead of gross profit, free founder time, an infinite horizon from a churn rate - and tell me the exact multiple by which it overstates reality. I want to recognise this slide when I see it.

8. Name the four errors in that flattering version and tell me which one I am personally most at risk of making.

9. Give me the cheapest lever to improve payback for MY numbers specifically, and what it would cost me to pull it.

10. Finally, tell me what benchmark figures you are tempted to quote me, and then tell me why I should not use them without a source for my own market.
```

## Mission

**Objective.** The four unit-economics numbers for your own startup — gross profit per subscriber, CAC including your time, bounded contribution from your real retention curve, and payback period — every line visible, every figure traceable to data you measured.

**Steps.**
1. **Frame** — write the one sentence this mission answers: does one customer pay back more than one customer costs, and in how many months.
2. **Gross margin** — build the table with every per-user cost line, including inference from chapter 6. If chapter 6 is not done, do it first.
3. **Acquisition spend** — as a group of three, each person lists every rupee and every hour they personally spent on acquisition last month. Pool them. Nobody's hours are free.
4. **Choose a labour rate** — pick a notional hourly rate, write one sentence justifying it, and use it consistently.
5. **CAC, twice** — compute it with and without labour, divide by new *paying* customers, and put both numbers side by side.
6. **Split the channels** — compute paid CAC separately from blended CAC. If one channel produced customers for free, say so and do not let it flatter the others.
7. **Retention curve** — pull one real cohort from your M16 instrumentation and write the number still paying in each month you have.
8. **Contribution and payback** — build the table with the running cumulative and mark the month where it crosses CAC. Do not extrapolate past your data.
9. **Build the flattering version too** — the same cohort told with revenue as margin, free labour, and an infinite horizon. Write down the multiple between the two.
10. **Pick one lever** — name the number you most want to change, the cheapest way to change it, and what it costs.
11. **Defend it** — hand all four numbers to another team with the instruction to find the missing or invented line. Record what they found.
12. **Document** — commit the whole model with today's date and the number of months of retention data it rests on.

**Upload.**
- Your gross margin table, every line visible → spreadsheet or screenshot
- Your inference cost per active user, with a pointer to your chapter 6 work → text
- Your itemised acquisition spend including every founder's hours → file
- Your chosen labour rate and the sentence justifying it → text
- Both CAC figures, with and without labour, side by side → screenshot
- Paid CAC and blended CAC, separately → text
- Your retention curve for one real cohort, with the number of months stated → screenshot from your analytics
- Your contribution and payback table with the cumulative column and the crossing month marked → screenshot
- The flattering version of the same numbers, with the multiple between the two → file
- What the other team found when they attacked your model → text
- Your decision-log entry with the date → file in your repo
- The AI conversation → share link in `ai_chat_url`
- Reflection, 150 words max → which of the four errors you were about to make, and one number the AI tried to invent for you

**Self-check.** Tick every box before you submit.
- [ ] My gross margin table has a separate inference line, or my product genuinely makes no model calls
- [ ] I used gross profit, not revenue, everywhere a customer's value appears
- [ ] My CAC numerator includes every founder's hours at a stated rate
- [ ] My CAC denominator is new paying customers, not signups
- [ ] Paid CAC and blended CAC are reported separately
- [ ] My retention curve comes from instrumentation, not from memory or estimate
- [ ] My contribution horizon is no longer than the months of data I actually have
- [ ] I did not use margin divided by churn anywhere
- [ ] My model reports a payback period in months, not only a ratio
- [ ] I wrote the flattering version too, and know the multiple between them
- [ ] Any launch discount is reflected in what the cohort actually paid
- [ ] No benchmark figure appears in my model without a source and a market
- [ ] Another team attacked the model and I recorded what they found
- [ ] My decision log states how many months of retention data this rests on

**Stretch.** Compute what your numbers would have to become for you to afford a ₹10,000 paid campaign next month without running out of cash before payback. Work backwards from your cash in hand and state which of the four numbers would have to move, and by how much.

## Stuck? Ask AI
```prompt
I do not understand why payback period matters more than the LTV to CAC ratio at my scale. Explain it with two worked examples using the same ratio but different payback periods, in rupees, for a team with ₹15,000 of cash. Stop after each example and ask me which team runs out of money first before you tell me.
```

```prompt
Explain why margin divided by churn is a dangerous way to compute LTV when my churn estimate comes from a few weeks of data. Show me the same formula with churn at 5%, 7% and 9% and let me see what happens to the answer. Then tell me what to use instead.
```

```prompt
My CAC excludes the time my team spent on acquisition because we are not paid. Argue against me. Tell me what decisions I will get wrong because of that omission, and what happens to that acquisition channel the month we get busy with exams.
```

## Quiz

### Q1 · L3
A team reports CAC of ₹75 from ₹3,000 of spend and forty customers. Their three founders also spent sixty hours standing at a canteen stall. What is the defect?
- (a) None; founder time is a fixed cost, not an acquisition cost
- (b) Labour is missing from the numerator, so CAC is understated by roughly four times ✅
- (c) They should have divided by signups rather than paying customers
- (d) CAC cannot be computed from a single month
> Sixty hours at a defensible rate is ₹9,000, which takes CAC from ₹75 to ₹300. The omission also hides the fact that this channel stops working the week the founders get busy.

### Q2 · L4
Which is the honest way to state what a customer is worth when you have four months of retention data?
- (a) LTV from gross profit divided by the monthly churn rate
- (b) Four-month contribution per original user, from the cohort's actual retention curve ✅
- (c) LTV from revenue divided by the monthly churn rate
- (d) Twelve-month contribution, extrapolating the curve's current slope
> A bounded horizon you have measured is defensible; an infinite horizon from a small, noisy churn estimate is not. Say "four-month contribution" and say it rests on four months.

### Q3 · L4
Your gross margin table lists the gateway fee and hosting, and your product makes model calls on every use. What is almost certainly wrong?
- (a) Hosting should be a fixed cost, not a variable one
- (b) The inference line is missing, and it is probably the largest cost of serving a user ✅
- (c) The gateway fee should be included in CAC instead
- (d) Nothing, as long as the margin looks reasonable
> For an AI product the inference line usually dwarfs the others. A margin computed without it is not a margin, and every number built on top of it inherits the error.

### Q4 · L4
Two teams both report an LTV to CAC ratio of 3. One recovers CAC in two months, the other in eleven. Why is the second team in more danger?
- (a) Its churn must be higher
- (b) It must spend cash eleven months before getting it back, and cash is the binding constraint ✅
- (c) Its ratio will fall as it grows
- (d) Its gross margin must be lower
> Returns are eventual; cash is now. A long payback means every acquisition rupee is locked up for most of a year, which is how a team with good economics still stops operating.

### Q5 · L5
An AI produces a complete unit-economics model for your startup in one reply, including churn, LTV and payback, from the three inputs you gave it. What is the correct response?
- (a) Use it, since the arithmetic is standard and correct
- (b) Find every number you did not supply, mark it as an assumption, and go and measure the ones that matter ✅
- (c) Ask it to recompute with more conservative assumptions
- (d) Discard the model and compute everything by hand
> The arithmetic will be right and the inputs will be invented, labelled in a clause you skimmed. The method is worth keeping; every figure that did not come from your own data is not.

## Go deeper
- [a16z: 16 startup metrics, including CAC, LTV and the common errors](https://a16z.com/16-startup-metrics/)
- [David Skok: SaaS unit economics, payback and the cash problem](https://www.forentrepreneurs.com/saas-metrics-2/)
