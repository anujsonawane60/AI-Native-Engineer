---
id: m17-02-pricing-models
module: m17-distribution-revenue
slug: pricing-models
title: "Pricing Models and Choosing Yours"
order: 2
duration_min: 35
is_free: false
mission: true
volatility: low
last_reviewed: 2026-11-16
---

## Key learnings
- Set a price from three anchors — the workaround's cost, your cost floor, and the alternatives
- Choose the value metric you charge against, and defend why it is that one
- Show that losing part of the queue to a price rise can still leave you better off
- Test a price with real people before you build the billing for it
- Catch the price that was picked from a feeling and dressed up afterwards

## Lesson

### The pain

A team prices their lunch pre-ordering app at ₹99 a month. Nobody subscribes. After two weeks they drop it to ₹49. Still nobody. They conclude that students will not pay for anything, which is the wrong conclusion and an expensive one, because the same students spend ₹210 a month working around the exact problem the app solves — they just have never been shown the comparison.

A second team goes the other way. They price at ₹19 a month because it is "cheap enough that anyone will say yes", and three hundred people say yes. Their AI feature costs them about ₹40 per user per month in inference. They are now paying ₹21 a month, per user, for the privilege of having users. Growth makes it worse. Every signup is a small, cheerful loss.

Both teams treated price as a number you choose by feeling, and then argued about the feeling. Neither wrote down what the user already spends, what one user costs to serve, or what the user would be giving up. All three of those are knowable in an afternoon.

### The idea

**A price is a decision about who you want as a customer, what you are replacing, and where your cost floor is — and it is arrived at from three anchors, not from a feeling.**

The three anchors, in order of importance: the **value anchor** (what the problem already costs this user, in rupees or in something they would trade rupees for), the **cost anchor** (what one user costs you to serve, which is your floor), and the **alternative anchor** (what the nearest comparable thing charges, which sets what feels normal).

A price between the value anchor and the cost anchor can exist. A price below the cost anchor cannot, however many people say yes. This chapter is about arriving at the number; chapter 3 is about the *structure* you charge it through — subscription, usage, credits, freemium — and chapter 4 is about collecting it.

### The analogy

The canteen thali is ₹60 and the owner is tired of thin margins. His real cost is ₹51 a plate, which chapter 6's method will make painfully familiar. He sells two hundred plates a day, so he makes ₹1,800.

He raises the price to ₹70 and half the queue vanishes overnight. Students complain loudly. His nephew tells him he has destroyed the business.

```
  THE ₹10 RISE, AS ARITHMETIC

  price  plates/day  margin/plate   daily profit
  ₹60          200           ₹9         ₹1,800
  ₹70          110          ₹19         ₹2,090
  ₹70          100          ₹19         ₹1,900
  ₹70           95          ₹19         ₹1,805
  ₹70           90          ₹19         ₹1,710
```

At ₹70 he can lose more than half the queue and still earn more than he did at ₹60, because the margin per plate more than doubled.

The break-even volume is ₹1,800 ÷ ₹19, which is about 95 plates. Below that he is worse off; above it he is better off, and he is also serving half as many people with the same gas and the same staff. Revenue fell from ₹12,000 to ₹7,700 and profit went up. Loud complaints are not the same as a bad decision — but notice that he could only know this because he knew his cost per plate.

**Where the analogy breaks:** the canteen's price change is reversible by rubbing out a number on a board, and tomorrow is a clean slate. Your price change hits people who already subscribed, and what you do about them — grandfather them at the old price forever, or move them and risk the churn — is a decision you cannot un-make. The canteen also sees the queue shrink by lunchtime; you will wait weeks for the same signal.

### The value anchor, done properly

This is the anchor teams skip, and it is the one that produces a defensible price. You are not asking "what would they pay?", which invites a polite lie. You are counting what the problem costs them today.

```
  WHAT THE WORKAROUND COSTS ONE STUDENT, PER MONTH

  item                                        cash    time
  20 min in the queue x 20 days                 --   400 min
  2 lunches skipped, replaced by snacks       ₹120
  friend bribed with a chai, 6 times            ₹90
  1 missed lab start, re-did the experiment     --    90 min
  -------------------------------------------------------
  already spent, in cash                     ₹210
  already spent, in time                             490 min
```

One user's existing monthly spend on the workaround, separated into cash they can see and time they feel.

That ₹210 is the ceiling that matters, far more than any competitor's price. A ₹99 subscription now has a sentence attached to it: "you already spend about ₹210 a month on this, and roughly eight hours." That sentence is what the two failing teams never had. Price without it and you are negotiating against yourself.

Two cautions. Time savings convert to cash very poorly for students, so do not value 490 minutes at a professional hourly rate and claim your product is worth ₹4,000. And the cash figure must come from asking, not from assuming — the ₹90 of chai is the kind of line nobody guesses and everybody confirms when asked.

### The value metric: what you charge *against*

Separate from how much, there is the question of per *what*. Per user per month, per order, per document, per message, per seat, per collection window. This is the **value metric**, and choosing it badly causes more damage than mispricing by ₹50.

A good value metric does three things: it grows as the customer gets more value, the customer can predict it before the bill arrives, and it is cheap for you to measure honestly. "Per order placed" passes all three for the lunch app. "Per API call" fails the second for a non-technical user. "Per team member" fails the first if value does not actually scale with headcount.

A crucial Indian-market note: the hardest step in your whole price ladder is not ₹99 to ₹199. It is ₹0 to ₹1. UPI has made the mechanics of a small payment almost frictionless, but the *decision* to pay anything at all is a large one, and it is where almost all of your drop-off will sit. Price so that the first paid step buys something the user can feel immediately.

### Where it lives

Backwards: M01's willingness-to-pay conversations are the raw material for the value anchor, and chapter 1's position tells you which workaround you are displacing and therefore which cost you are counting.

Forwards and sideways: chapter 6 gives you your cost floor, and for any product with an AI feature that floor is much higher than teams expect — read it before you commit to a number. Chapter 3 turns the number into a structure. Chapter 4 collects it, including the UPI autopay mandate, which has a maximum amount in it and therefore quietly constrains future price rises. Chapter 5 is where this price becomes ARPU and meets CAC. M03's work on trust and hierarchy is how the price lands on a page without looking like a trap. M12 matters if you sell inside a mobile app, because the store takes a cut of in-app purchases and that cut is part of your cost floor.

### Words to own

| Term | Plain meaning | Example |
|---|---|---|
| List price | The number you publish | ₹99 per month |
| Value metric | The unit you charge against | Per order placed |
| Value anchor | What the workaround already costs them | ₹210 a month |
| Cost floor | What one user costs you to serve | ₹59 a month |
| Elasticity | How much demand moves when price moves | Half the queue left |
| Break-even volume | Units needed to match the old profit | 95 plates |
| Price fence | The rule that separates two plans | Order volume |
| Grandfathering | Keeping old customers at the old price | A promise you keep |
| Annual plan | A year paid upfront, usually discounted | 10 months for 12 |
| Anchor price | A higher option that makes yours look fair | The ₹499 tier |

### Tradeoffs and gotchas

**Cheap is not a strategy, it is a cap.** A ₹19 price makes every other number in chapter 5 impossible: you cannot spend on acquisition, you cannot absorb a refund, and you cannot afford support. Low prices also attract the users least likely to stay, which is the opposite of what you need in a six-month program.

**Discounting is a price change you did not announce.** A 50% launch discount for three months is a ₹49 product with a ₹99 label, and your early cohort's retention and ARPU in chapter 5 will be measured on the discounted number whether you like it or not. If you discount, write down the date it ends and honour it.

**Free is a price, and it is the most expensive one.** It converts cost into a liability with no revenue attached, which chapter 6 treats as a first-class danger for anything with inference in it. Free has uses — chapter 3 names the three conditions under which freemium works — but "free for now" is not one of them.

**Annual plans are a cash decision, not a pricing decision.** A year upfront is wonderful for your runway and it makes your retention numbers look better than they are, because nobody can churn until month twelve. Be honest about which effect you are seeing.

**Round numbers that match existing habits beat clever ones.** Indian consumers read ₹49, ₹99, ₹149, ₹199, ₹299 and ₹499 as familiar steps. ₹127 invites the question "why 127?", and every question you invite at the price is a user you lose.

**Raising a price later is harder than starting higher.** You can always discount from a high price. Moving up means a conversation with everyone who already said yes, and a decision about grandfathering that will cost you either money or goodwill.

### Verify it

**Red flag, the headline one: a price with no reference to what users already pay for the workaround.** If you or an AI produce a number and the justification is "comparable products charge this", the value anchor is missing and the price is a guess with a citation. Ask: what does this cost them today, in rupees, and who told us?

**Red flag: a price below the cost floor.** Any price proposed before chapter 6's cost-per-active-user figure exists is unverified. Ask for the floor explicitly, and if the answer is "we will optimise later", the price is not yet a price.

**Red flag: a price quoted in dollars, or converted from one.** AI will cheerfully recommend $9 per month for Indian college students. Any pricing advice that does not visibly know who your buyer is and what they earn is advice about a different market.

**Red flag: no value metric named.** "₹99 per month" with no statement of what happens to a user who places five hundred orders is an unbounded promise. Chapter 6 shows what that costs.

**Red flag: willingness to pay collected by asking "would you pay ₹99 for this?"** Everyone says yes to be kind and no with their wallet. Ask what they spend today, and ask for the money in a real moment.

**Red flag: a price with no break-even arithmetic behind the change.** When you move a price, the only honest question is how much volume you can lose and stay level. That is one division. If nobody has done it, the argument is about feelings.

The check that settles it: put the price in front of five real users in a moment where paying is possible, and count who pays. Five yeses in a survey and one payment is a clear result, and the result is the payment.

## Walkthrough
1. Write down, for one real user you have spoken to, every rupee they currently spend working around your problem in a month. Then write the time separately. Do not convert time into money.
2. Repeat this for four more users. Take the median of the cash figure, not the mean, so one unusual answer does not set your ceiling.
3. Write down your cost floor per active user per month. If you have an AI feature, read chapter 6 first and use the figure from it. If you genuinely do not know yet, write "unknown" and treat every price below as provisional.
4. List the nearest three alternatives your users already pay for anything comparable, with their actual current prices, read from their own pages today.
5. Choose your value metric. Write one sentence on why value grows with it, one on how the user predicts their bill, and one on how you measure it honestly.
6. Propose three price points: one that feels slightly too cheap, one you believe in, and one that feels slightly too expensive. Write each as a number plus what it includes.
7. For your middle price, do the break-even arithmetic in both directions: how many subscribers at that price cover your monthly costs, and how many you could lose if you moved up one step.
8. Run the four willingness-to-pay questions on at least five real users, one at a time: at what price is this so cheap you would doubt it, cheap, expensive, and so expensive you would not consider it. Record all four numbers per person.
9. Ask those same five people for the money, today, for one month, through a real UPI test flow or a stated promise with a date. Count payments, not nods.
10. Pick your price, write the three anchors next to it, and commit it to your repository as `PRICING.md` with today's date and the trigger that would make you revisit it.

## Talk to AI
```prompt
I need to set a price for my startup and I want the reasoning exposed, not a recommendation. Do not give me a number until the end, and do not give me any price in dollars.

My product and position: [the four lines from chapter 1]
Who the buyer is: [who actually pays - note if it is not the user]
What my users spend TODAY working around this problem, from real conversations: [rupees per month, and how many people told me]
What they spend in TIME today: [keep this separate from the cash]
My cost to serve one active user per month, in rupees: [number, or "unknown"]
Does my product include an AI feature with a per-request cost? [yes/no - if yes, say what I know]
The nearest alternatives my users already pay for, with current prices: [list]
Where my users are and what they earn: [be specific - Indian college students, hostel, etc.]

Do all of this, in order:

1. Tell me which of my three anchors - value, cost, alternative - is weakest or missing, and exactly what I would have to go and find out to fix it. Say plainly if my inputs cannot support a defensible price yet.

2. Build me a table of what the workaround costs one user per month, with cash and time in separate columns, using only the figures I gave you. Do not estimate figures I did not supply.

3. Propose three candidate value metrics for my product. For each, tell me whether value grows with it, whether the user can predict their bill, and how hard it is for me to measure honestly. Then recommend one and say what it exposes me to.

4. Give me three candidate price points as a ladder, with what each includes and which user each is for. Explain what each price assumes about my cost floor.

5. For my middle price, show me the break-even arithmetic as a small table: subscribers needed to cover my monthly costs, and how much volume I could lose if I moved one step up the ladder. Show the division, not just the answer.

6. Tell me what my price assumes about users who use the product far more than average, and what happens to me if five per cent of users use it twenty times as much.

7. Write me the exact wording of the four willingness-to-pay questions to ask five real users, plus the one question I should NOT ask and why.

8. Finally, name the single biggest risk in the price you would pick, and what signal in the first month would tell me I got it wrong.
```

## Mission

**Objective.** A written, dated price for your own startup, with all three anchors filled in, a chosen value metric, break-even arithmetic you can defend, and evidence from five real users — at least one of whom actually paid.

**Steps.**
1. **Frame** — write in one sentence who pays, for what, and what they stop doing. If the payer is not the user, say so now; it changes everything that follows.
2. **Value anchor** — interview five real users and count what the workaround costs them per month in cash, and separately in time. Take the median of the cash. As a group of three, split the interviews so no single person's phrasing shapes all five answers.
3. **Cost anchor** — write your cost to serve one active user per month. If you have an AI feature, do chapter 6 first and bring that number here. Label any assumption.
4. **Alternative anchor** — list three things your users already pay for that are even loosely comparable, with prices read from the sellers' own pages today, with the date.
5. **Value metric** — choose what you charge against and write the three defences: value grows with it, the user can predict it, you can measure it.
6. **Ladder** — propose three price points and say which user each is for, with what each includes and what it caps.
7. **Break-even** — for your chosen price, show the arithmetic: subscribers needed to cover monthly costs, and how much volume you could afford to lose at the next price up. Show the division.
8. **Test it** — run the four willingness-to-pay questions on your five users and record all four numbers for each. Then ask for the money in a real moment and count payments, not nods.
9. **Decide** — pick the price. Write the three anchors beside it, the evidence, and the one signal that would make you change it.
10. **Document** — commit `PRICING.md` with the date, the anchors, the metric, the ladder, the break-even table and the test results.

**Upload.**
- Your workaround-cost table for five users, cash and time in separate columns → file or screenshot
- Your cost floor per active user per month, with its source → text
- Your three alternative prices, with the date you read each one → screenshot
- Your chosen value metric and its three defences → text
- Your three-price ladder → screenshot of the page or table
- Your break-even arithmetic, with the division visible → file or screenshot
- All four willingness-to-pay answers from each of the five users → file
- Evidence of at least one real payment attempt, in test mode → screenshot
- Your `PRICING.md` with today's date → file in your repo
- The AI conversation → share link in `ai_chat_url`
- Reflection, 150 words max → the gap between what users said they would pay and what happened when you asked for money, and one thing the AI got wrong

**Self-check.** Tick every box before you submit.
- [ ] My value anchor comes from five real conversations, not from an estimate
- [ ] Cash and time are separate in my table, and I did not convert time into rupees
- [ ] I have a cost floor per active user per month, and my price is above it
- [ ] If my product has an AI feature, my cost floor came from chapter 6's method
- [ ] Every alternative price I quote was read from the seller's own page, with the date
- [ ] My value metric has all three defences written out
- [ ] My break-even arithmetic shows the division, not only the answer
- [ ] I asked all four willingness-to-pay questions, in the right order, to five people
- [ ] I asked at least one real person for money and recorded what happened
- [ ] No price in my submission is in dollars or converted from dollars
- [ ] `PRICING.md` is committed with today's date and a trigger to revisit
- [ ] My reflection names the gap between stated and revealed willingness to pay

**Stretch.** Write the memo you would send to existing subscribers if you raised this price by 50% in four months: what you say, who you grandfather, for how long, and what you expect to lose. Hand it to another team and have them argue the subscriber's side.

## Stuck? Ask AI
```prompt
I do not understand how losing customers to a price rise can leave me better off. Walk me through it with a worked table, one line at a time, using my numbers: my price is [X], my cost per unit is [Y], and I currently sell [Z] units a month. Stop after each line and ask me what I expect next before you show it.
```

```prompt
Explain the difference between what someone says they would pay and what they actually pay, with three concrete examples from Indian consumer products. Then give me five ways of asking about price - some that produce useful answers and some that produce polite lies - and ask me to sort them before you tell me which is which.
```

```prompt
My team is split between pricing at a low number to get users and a higher number to have a business. Here are our numbers: cost per active user [X], candidate prices [A and B], target users [N]. Do not pick for me. Instead, tell me what each choice commits us to for the next six months - what we could and could not afford to do at each price.
```

## Quiz

### Q1 · L3
Which anchor is most often missing from a student team's price, and makes the price undefendable?
- (a) The alternative anchor — what comparable products charge
- (b) The value anchor — what the workaround already costs this user in rupees ✅
- (c) The psychological anchor — whether the number ends in 9
- (d) The brand anchor — how premium the product looks
> Teams reliably find a competitor's price and stop there. Without what the user already spends, the price has no relationship to the buyer's own life and cannot be argued for.

### Q2 · L4
A canteen sells 200 plates at ₹60 with a ₹51 cost. It raises the price to ₹70 and sells 110. What happened to daily profit?
- (a) It fell, because revenue fell from ₹12,000 to ₹7,700
- (b) It rose, from ₹1,800 to ₹2,090, because margin per plate more than doubled ✅
- (c) It stayed level, because the losses offset the gains
- (d) It cannot be determined without knowing fixed costs
> Margin went from ₹9 to ₹19. Break-even volume was ₹1,800 ÷ ₹19, about 95 plates, so 110 plates is better off. Revenue and profit can move in opposite directions.

### Q3 · L4
Your product has an AI feature and you have not computed what one active user costs you. What does that mean for your price?
- (a) Nothing, as long as the price matches competitors
- (b) The price is provisional, because you do not yet know your floor and could be selling at a loss ✅
- (c) You should price low to acquire users while you find out
- (d) You should price in dollars to hedge the inference cost
> A price below the cost floor loses money faster the better it sells. Without the floor, "₹19 and everyone says yes" is indistinguishable from a cheerful, scaling loss.

### Q4 · L3
Which of these is the strongest value metric for a lunch pre-ordering product used by students?
- (a) Per API call made by the client application
- (b) Per team member added to the account
- (c) Per order placed, capped per month ✅
- (d) A flat fee with unlimited everything
> Value grows with orders, the student can predict their own bill, and you can measure it honestly. API calls are invisible to the buyer and seats do not match how value arrives here.

### Q5 · L5
Five users tell you in a survey they would pay ₹99 a month. You then ask all five to pay today and one does. What is the correct read?
- (a) The price is right; the other four need a better onboarding flow
- (b) The payment is the data and the survey is noise — at ₹99 your conversion is one in five, not five in five ✅
- (c) The sample is too small to mean anything
- (d) The price should be dropped to ₹49 immediately
> Stated willingness to pay is a politeness; revealed willingness is a payment. One in five is a real number to work with, and it is the one that will show up in chapter 5's arithmetic.

## Go deeper
- [Patrick Campbell / ProfitWell on value metrics and pricing research](https://www.paddle.com/resources/pricing-strategy)
- [Van Westendorp's price sensitivity meter, explained](https://en.wikipedia.org/wiki/Van_Westendorp%27s_Price_Sensitivity_Meter)
