---
id: m17-07-launch-channels
module: m17-distribution-revenue
slug: launch-channels
title: "Launch Channels and Running a Real Launch"
order: 7
duration_min: 35
is_free: false
mission: true
volatility: low
last_reviewed: 2026-11-16
---

## Key learnings
- Run a launch as a dated sequence against named channels, not as one announcement
- Capture an owned channel during the spike, because the spike decays in 48 hours
- Measure CAC and activation per channel, so you know which one to repeat
- Pass the pre-launch checklist that prevents the broken-signup disaster
- Catch a launch plan with no owned channel, no instrumentation and nobody watching

## Lesson

### The pain

Two launches, on the same Thursday, in the same cohort.

The first team posts one message in a hostel WhatsApp group at 11 pm. Forty views, two signups, nothing else. Asked how the launch went, they say "nobody cared". They had no plan, no second channel, no instrumentation and no follow-up, so "nobody cared" is not a finding — it is the absence of one.

The second team does much better and loses more. A post lands well and sends nine hundred people to their site in four hours. The signup flow has a bug on a budget Android browser nobody tested. Somewhere between four and six hundred of those people hit a blank screen and leave. The team finds out on Saturday, because nobody was watching on Thursday evening. They also have no email list and no broadcast list — so of the three hundred who got through, there is no way to reach a single one again.

The second team's product was ready. Their launch was not, and a launch you get one shot at is a strange thing to improvise.

### The idea

**A launch is a dated sequence against named channels, built to convert a 48-hour spike into something you own.**

Three parts, and teams usually build only the middle one. The **sequence**: what happens on which day, who does it, in what order. The **channels**: named places with named owners, not "social media". The **capture**: a channel you own — phone numbers or emails you can reach again next month without anyone's permission.

Attention decays fast. Whatever a launch brings in forty-eight hours is nearly all you will get, and the only durable output is the list.

### The analogy

The canteen's opening day.

The owner does not simply unlock the shutter. A week before, he talks to the hostel warden and the three students who run the cultural committee. Two days before, a hand-painted board goes up at the gate. On the day, the first hundred plates come with a free samosa, and he stands at the counter himself from noon to three, because the first day is when problems surface. He writes every regular's phone number in a notebook so he can tell them when the Sunday special starts. And on day two he is still there, because the people who had a good time on day one bring four friends on day three.

**Where the analogy breaks:** his channel is physical proximity, and it keeps working every day without being refreshed — a student walking to class passes the gate whether or not he posted anything. Your launch is a spike with a steep decay: day three is a fraction of day one, and day ten is nothing. The notebook of phone numbers is the only part of his setup you can copy, and it is the part teams skip.

### Picture it

```
  A LAUNCH AS A SEQUENCE

  T-7  fix the checklist; write the asset once
  T-5  tell 20 people personally, one by one
  T-3  warm the communities you are already in
  T-1  freeze the code; final device test; who is on call
  ----------------------------------------------------------
  T+0  09:00  post to channel A  (owner: Priya)
       12:00  post to channel B  (owner: Anand)
       14:00  personal messages, batch 1
       all day  SOMEONE WATCHING the funnel, live
  T+1  reply to every comment; post to channel C
  T+2  second wave to the 20 from T-5
  T+7  write the report: signups, activation, CAC per channel
  ----------------------------------------------------------
  what you keep:  the list of people who gave you a way
                  to contact them again
```

A launch runs from a week before to a week after, with an owner and a time for every action, and its only durable output is a contact list.

### Owned, rented, earned

Every channel is one of three kinds, and the distinction decides where you invest.

**Owned** — your email list, your WhatsApp broadcast list, your push permission, your users' phone numbers. You can reach these people next month at zero cost, with nobody's permission. The only kind that accumulates.

**Rented** — Instagram, LinkedIn, Reddit, a Telegram group, a college WhatsApp group you do not administer. Someone else's audience, at their discretion and subject to their rules. Useful, free, revocable.

**Earned** — someone else talks about you: a friend's recommendation, a club newsletter, a mention by a popular student account. Highest trust, least controllable.

The rule: **spend rented and earned attention to build owned attention.** Every launch post should lead somewhere that asks for a way to reach the person again. Ten thousand impressions and no contact details has converted a scarce resource into nothing.

### The channels that actually work for an Indian student startup

Named specifically, because "social media" is not a channel.

**WhatsApp groups** are the highest-conversion channel most student teams have, and the most easily wasted. A message from a known member of a group of two hundred converts far better than a post to two thousand strangers. One message per group, from a member, at a time people read — not 11 pm.

**Talking to people one at a time** is not a channel, it is a superpower at your scale. Twenty personal messages beat one broadcast, and M16's unscalable-first-hundred chapter is the whole argument.

**Campus institutions**: societies, clubs, noticeboards, a two-minute announcement before a lecture if a professor allows it, fests. These are earned channels with a gatekeeper, which chapter 8 covers.

**Instagram** reaches students; **LinkedIn** reaches recruiters and founders. Different audiences, and the same post will not serve both.

**Reddit and Telegram** have India-focused communities with strict self-promotion rules. Read the rules, participate first, and expect to be removed if you treat a community as an audience.

**Product Hunt and Hacker News** are real channels for developer tools and almost useless for an Indian campus consumer product. Launching there because it is what startups do spends a day on the wrong audience.

**App stores** are their own discipline, covered in M12 — the listing, the screenshots and the first-run experience are the conversion surface.

**Paid** is testable at small budgets and is the only channel that gives you a clean CAC number. Spend ₹1,500, count paying customers, and you have chapter 5's arithmetic for that channel. Treat it as a measurement instrument first and a growth engine much later. **SEO and content** work and are slow; they belong in month four, not in your launch.

### Measuring a launch without lying to yourself

A launch produces several numbers per channel, and only the last two matter.

```
  ONE LAUNCH, BY CHANNEL

  channel          visits  signups  activated  paying   CAC
  WhatsApp, 6 grps    310       74         41      11   ₹27
  personal msgs, 20    20       14         12       6   ₹50
  Instagram post      540       38          9       1  ₹120
  college society     130       29         18       4   ₹45
  --------------------------------------------------------
  total             1,000      155         80      22
```

One launch by channel: Instagram delivered the most visits and the fewest paying customers, and twenty personal messages delivered six.

Impressions and visits feel good and decide nothing. **Activated** — M16's definition, the user who actually got value — and **paying** are the numbers that tell you which channel to repeat. The CAC column uses chapter 5's rule, so the personal messages carry the founders' time and are not free.

To get this table you need the instrumentation before the launch, not after: a distinct link per channel, and M07's event tracking with consent, so a signup can be attributed. M16's attribution chapter is where the honest version lives, including why the numbers never quite add up.

### Where it lives

Chapter 1's position is what you are distributing — a vague sentence launched loudly is still vague. Chapters 2 and 3 decide what the visitor is asked to buy. Chapter 6 decides whether a successful launch is good news or an expensive one, because nine hundred visitors arriving at an uncapped free AI feature is a bill.

M16 owns activation, funnels and attribution; M07 owns the event tracking and consent that make attribution possible; M12 owns the store listing; M13 and chapter 9 own what happens when the spike breaks something. Chapter 8 is the slower, relationship-based distribution a launch cannot replace, and chapter 11 reviews the channel table weekly, which is how a one-off launch becomes a repeatable channel.

### Words to own

| Term | Plain meaning | Example |
|---|---|---|
| Owned channel | Contacts you can reach free, anytime | Your WhatsApp list |
| Rented channel | Someone else's audience, revocably | An Instagram feed |
| Earned channel | Someone else chooses to talk about you | A society newsletter |
| Capture | Getting a way to contact a visitor again | A phone number |
| Spike | The 48 hours of attention a launch buys | Day one traffic |
| Channel owner | The person responsible for that post | Priya, 09:00 |
| Activation | The user who actually got value | M16's definition |
| Attribution | Which channel produced a user | One link per channel |
| Code freeze | No deploys around launch | T-1 to T+1 |
| Soft launch | A deliberately small first release | 20 people, no posts |

### Tradeoffs and gotchas

**Do not launch and build on the same day.** Freeze the code at T-1. A deploy during the spike is how a working signup flow becomes a blank screen for six hundred people.

**Somebody must be watching, live, for the first six hours.** Name them, and give them nothing else to do. The second team's bug was visible in their own funnel within twenty minutes and nobody was looking.

**Test on a real budget Android phone on mobile data, not on your laptop.** This single check prevents the commonest launch disaster in an Indian student product.

**One channel is not a launch.** If a single post underperforms you learn nothing, because you cannot tell whether the message, the channel or the timing failed.

**Do not burn a community you want to stay in.** A promotional post in a group where you have never participated gets you removed and remembered.

**You can launch more than once.** Launch to twenty people, learn, fix, then launch to two hundred. Teams treat the first post as irreversible and spend six weeks polishing instead.

**A successful launch costs money.** Check your chapter 6 caps and spend alerts before you invite a crowd, or your best day becomes your biggest bill.

### Verify it

**Red flag, the headline one: a launch plan with no owned channel.** Ask: when this is over, how do I contact the people who came? If the answer is "post again", the launch has no durable output and you start from zero next month.

**Red flag: no instrumentation, so attribution is a guess.** Ask for the distinct link per channel and the signup event, before launch day. Afterwards it cannot be reconstructed.

**Red flag: nobody named as watching**, or **a single channel, or a list that says "social media".** A plan where "we" will monitor means nobody will, and channels have names, owners and times.

**Red flag: a plan measured in impressions and visits.** Ask for the activated and paying columns. A channel sending five hundred visits and one paying customer is worse than one sending twenty and six.

**Red flag: a launch that assumes signup works.** The checklist is not optional: a real budget Android phone, mobile data, a fresh account, the payment flow in test mode, and the empty state a brand-new user actually sees.

**Red flag: AI-generated launch copy that could be any product.** Run chapter 1's swap test on every post before it goes out. Fluent and interchangeable is the failure mode, and it is worst in the material you send to the most people.

**Red flag: a plan with no uncapped-cost check.** If a successful launch would break your chapter 6 budget, the plan has a hole in it shaped like a bill.

## Walkthrough
1. Write the one-sentence version of your position from chapter 1, as a single message you would send to one person. If it does not survive being sent to one person, it will not survive being posted to two hundred.
2. List every channel you can actually reach this month, by name, with the number of people in it and whether it is owned, rented or earned. Not "Instagram" — "my Instagram, 340 followers, mostly classmates".
3. Pick four channels. Assign each one a named owner from your team and a specific time on launch day.
4. Decide your capture mechanism: exactly what you ask a visitor for, and where it is stored. A phone number or an email, collected with consent, that you can use again.
5. Set up attribution: a distinct link per channel, and the event in your analytics that marks signup and activation. Verify it by clicking your own links and confirming each one is distinguishable.
6. Run the pre-launch checklist on a real budget Android phone on mobile data: signup with a fresh account, the first-run empty state, the core action, and the payment flow in test mode.
7. Check your chapter 6 guardrails: per-user caps, account spend cap, billing alert. Confirm that nine hundred visitors would not produce a bill you cannot pay.
8. Freeze the code. Name the person who is watching the funnel live for the first six hours, with nothing else assigned to them.
9. Soft launch to twenty people personally, one message at a time, two days before. Fix what they hit. This step is where the Android bug gets found.
10. Launch on the sequence. Post, reply to everything, and watch the funnel. Record visits, signups, activation and paying per channel as it happens.
11. At T+7 write the launch report: the channel table with CAC per channel including your own hours, the size of your owned list, and the one channel you will repeat.

## Talk to AI
```prompt
I am planning a real launch for my startup and I need the plan attacked before I run it. Do not write me marketing copy.

My position, one sentence: [from chapter 1]
What a visitor is asked to do: [sign up / pay / join a list - be exact]
My price and plan: [from chapters 2 and 3]
Channels I can actually reach, by name, with sizes: [list each one, with how many people and whether I am a real member of that community]
My owned channel today: [what it is, and how many contacts - say "none" if none]
My team: three students. Who can be free on launch day: [names and hours]
My analytics setup: [what events exist today, honestly]
My AI feature's per-user cost and caps: [from chapter 6]

Do all of this, in order:

1. Tell me what my launch's durable output would be if I ran it as described. If the answer is "nothing", say so plainly and tell me what to add before launch day.

2. Sort my channels into owned, rented and earned. Tell me which ones I am likely to burn by posting promotionally, given what I told you about my membership in each.

3. Build me a dated sequence from T-7 to T+7, with a named owner and a time for every action, using only the people and channels I actually have.

4. Specify my capture mechanism: exactly what I ask for, at what moment, where it is stored, and what consent wording I need.

5. Specify the attribution setup I must have in place BEFORE launch: one link per channel and the events that mark signup and activation. Tell me what becomes unknowable if I skip this.

6. Give me the pre-launch checklist for an Indian student product, including the device and network conditions I must test on. Be specific about what breaks most often.

7. Tell me what happens to my costs if the launch works much better than expected, using my chapter 6 numbers, and what guardrail I need before I invite a crowd.

8. Build me the launch report template: the channel table with the columns that actually decide what to repeat, and the columns that only feel good.

9. Run a swap test on my position: name two other products my launch message would also describe. If it survives the swap, tell me not to send it.

10. Finally, name the single most likely way this launch wastes its spike, and the one thing I should do tomorrow to prevent it.
```

## Mission

**Objective.** Run a real launch for your own startup against at least three named channels, capture an owned contact list during it, and produce a launch report with activation and CAC per channel.

**Steps.**
1. **Frame** — write the one-sentence message, and run chapter 1's swap test on it before anything else.
2. **Inventory the channels** — every channel you can genuinely reach, by name, with its size and whether it is owned, rented or earned. Be honest about which communities you are actually a member of.
3. **Assign owners** — pick at least three channels, each with one named person from your team and a specific time.
4. **Build the capture** — decide what you ask a visitor for, where it is stored, and the consent wording. Build it before the sequence runs.
5. **Instrument it** — a distinct link per channel, and events marking signup and activation. Test each link yourself and confirm they are distinguishable in your analytics.
6. **Pre-launch checklist** — a real budget Android phone on mobile data, a fresh account, first-run empty state, the core action, and the payment flow in test mode. Record what you found and fixed.
7. **Cost guardrails** — confirm your chapter 6 caps, account spend cap and billing alert, and state what a 10x day would cost you.
8. **Soft launch** — twenty people, personally, one at a time, two days early. Fix what they hit.
9. **Freeze and watch** — no deploys from T-1, and one named person watching the funnel live for the first six hours with nothing else to do.
10. **Run the sequence** — post, reply to every comment and message, and log numbers per channel as they arrive.
11. **Report at T+7** — the channel table with visits, signups, activated, paying and CAC per channel including your own hours at chapter 5's rate.
12. **Decide** — name the one channel you will repeat and the one you will drop, with the number that decided it.

**Upload.**
- Your one-sentence message and the swap-test result → text
- Your channel inventory with sizes and owned/rented/earned labels → file
- Your dated sequence with named owners and times → file or screenshot
- Your capture mechanism and consent wording, live in the product → screenshot
- Proof of per-channel attribution working → screenshot showing distinguishable links or sources in your analytics
- Your pre-launch checklist results, including what the budget Android test found → text and screenshots
- Your cost guardrails and your stated 10x-day exposure → screenshots and one line of arithmetic
- Evidence of the soft launch and what it changed → text
- Who was watching, when, and what they caught → text
- Your launch report with the full channel table including CAC per channel → spreadsheet or screenshot
- The size of your owned list at T+7 → screenshot
- The AI conversation → share link in `ai_chat_url`
- Reflection, 150 words max → the channel that surprised you, and one thing the AI got wrong about your audience

**Self-check.** Tick every box before you submit.
- [ ] My launch has a capture mechanism and my owned list grew because of it
- [ ] At least three named channels, each with one named owner and a time
- [ ] Every channel had a distinct link and I verified attribution before launching
- [ ] I tested signup on a real budget Android phone on mobile data, not only my laptop
- [ ] I tested the first-run empty state a brand-new user actually sees
- [ ] The payment flow was tested in test mode before the launch
- [ ] Code was frozen from T-1, and nobody deployed during the spike
- [ ] One named person watched the funnel live for the first six hours
- [ ] I soft-launched to twenty people personally before the public sequence
- [ ] My chapter 6 caps, spend cap and billing alert were confirmed before launch
- [ ] My report has activated and paying columns, not only visits
- [ ] CAC per channel includes our own hours at a stated rate
- [ ] I named one channel to repeat and one to drop, with the number that decided it
- [ ] My launch copy passed the swap test before it was posted

**Stretch.** Design the second launch. Using only your own channel table, write what you would do differently, what you would stop doing entirely, and what you would need to build first. Then estimate what the same effort would produce the second time and say why.

## Stuck? Ask AI
```prompt
Explain the difference between owned, rented and earned channels using examples an Indian college student would recognise. Then give me eight channels and ask me to sort them into the three kinds, without telling me the answers first. Finish by telling me what it means if all of my channels are rented.
```

```prompt
My launch sent 900 people to my site and I kept almost none of them. Here is what I had in place: [describe]. Do not console me. Tell me exactly which capture and instrumentation pieces were missing, what is now unknowable forever, and what I must build before I spend attention again.
```

```prompt
Give me the pre-launch checklist for a consumer web product whose users are on budget Android phones on mobile data in India. Order it by how often each item actually breaks a launch, and for each one tell me the exact test to run and what a failure looks like.
```

## Quiz

### Q1 · L3
A launch sends nine hundred visitors in four hours and the team has no email or phone list. What is the cost of that gap?
- (a) None, as long as some of the visitors signed up
- (b) The spike's only durable output is lost — they cannot reach any of those people again ✅
- (c) They cannot compute CAC for the launch
- (d) Their analytics will double-count returning visitors
> Attention decays in about forty-eight hours. Rented and earned channels are spent to build an owned one, and a launch with no capture converts a scarce resource into nothing.

### Q2 · L4
Instagram sent 540 visits and one paying customer. Twenty personal messages sent 20 visits and six paying customers. Which channel do you repeat?
- (a) Instagram, because it reaches far more people per unit of effort
- (b) The personal messages, because the activated and paying columns are what decide ✅
- (c) Both equally, since the visit totals are not comparable
- (d) Neither; the sample is too small to decide anything
> Visits feel good and decide nothing. Six paying customers from twenty messages is a working channel, even though it does not scale yet — and CAC including your hours is how you compare them honestly.

### Q3 · L4
Why is the instruction "freeze the code at T-1" more than caution?
- (a) Deploys are slower under load
- (b) A deploy during the spike is how a working signup flow becomes a blank screen for hundreds of people at once ✅
- (c) It prevents attribution links from changing
- (d) Gateways block new deployments during high traffic
> The spike is the one window you cannot repeat. Any change during it risks the whole audience simultaneously, and you will discover it from your funnel hours later, if anyone is watching.

### Q4 · L3
What single pre-launch test prevents the most common launch disaster for an Indian student product?
- (a) Load-testing the server for a thousand concurrent users
- (b) Completing signup with a fresh account on a real budget Android phone on mobile data ✅
- (c) Checking the site in three desktop browsers
- (d) Asking an AI to review the landing page copy
> Your users are not on your laptop. The device-and-network test is cheap, takes ten minutes, and catches the blank-screen failure that silently discards most of a successful launch.

### Q5 · L5
Your launch works far better than expected and your AI feature has no usage cap. What have you just done?
- (a) Validated demand, which is worth any short-term cost
- (b) Converted your best day into your largest bill, because cost scales with usage and most visitors do not pay ✅
- (c) Nothing concerning, since free-tier credits absorb spikes
- (d) Created a caching problem that will resolve itself
> Chapter 6's liability and chapter 7's spike are the same event seen from two sides. Confirming caps, a per-user limit and a spend alert is part of the launch checklist, not an afterthought.

## Go deeper
- [Y Combinator: how to launch, and how to launch repeatedly](https://www.ycombinator.com/library/6s-how-to-launch)
- [First Round Review: building an owned audience before you need it](https://review.firstround.com/)
