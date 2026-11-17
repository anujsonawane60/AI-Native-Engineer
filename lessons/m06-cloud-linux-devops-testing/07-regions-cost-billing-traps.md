---
id: m06-07-regions-cost-billing
module: m06-cloud-linux-devops-testing
slug: regions-cost-billing-traps
title: "Regions, Zones, Cost Models and Billing Traps"
order: 7
duration_min: 15
is_free: false
mission: false
volatility: medium
last_reviewed: 2026-11-16
---

## Key learnings
- Choose a region by where your users are, and know why that choice is hard to undo
- Name the four things clouds charge for and which one surprises people
- Set a billing alert before you create anything that can cost money
- Recognise the five ways a student team generates an unexpected bill
- Refuse a cost estimate that contains no numbers

## Lesson

### The pain

A team in Nashik accepts the default region, Virginia. One page load makes six database calls, each crossing the Indian Ocean and back at roughly 250 ms each way — three seconds of waiting caused by a dropdown nobody read.

A second team's VM is compromised exactly as chapter 3 described and mines cryptocurrency for nine days: ₹41,000, because nothing looked wrong from outside. Both are routine, and the same four minutes of work prevents both.

### The idea

**Cloud cost is a meter, not a price, and the meter runs whether or not anyone is using your product — so you set the alarm before you turn anything on.**

You are metered on four things at once, three of which you cannot see from inside your app.

| Meter | What it counts | Student-scale trap |
|---|---|---|
| Compute time | Seconds your instance or function runs | An idle VM left running all month |
| Storage | Gigabytes held, per month | Old backups and build artefacts forever |
| Egress | Data leaving the provider | Load tests, large images, video |
| Operations | Requests, reads, writes, invocations | A loop calling an API, a retry storm |

**Egress is the one nobody expects:** data *in* is usually free, data *out* is charged per gigabyte.

### The analogy

The canteen pays rent in one lump each month and the manager knows that number by heart. Gas, electricity and water are meters, and nobody knows those by heart. The freezer runs all night whether or not anyone bought ice cream; a tap left running in the back costs money for nine days before anyone notices. The manager's defence is not vigilance — the electricity board sends an SMS when the bill passes ₹5,000.

**Where the analogy breaks:** a water meter has a physical ceiling. Cloud meters are elastic by design — a loop calling an API a million times can spend in a weekend what the canteen spends on electricity in a year. The alarm is not optional.

### Picture it

```
  WHAT YOU ARE METERED ON

  COMPUTE TIME    ████████████  runs while idle too
  STORAGE         ████          cheap, grows forever
  EGRESS (out)    ████████      the surprise line
  OPERATIONS      ██            per request / per read

         │
         ▼
  ┌──────────────────────────────┐
  │  BUDGET ALERT at a number    │  ◀── set this FIRST
  │  you would be upset to see   │
  └──────────────┬───────────────┘
                 │ email + SMS
                 ▼
          you, on day 2
       instead of on day 30
```

Four separate meters add up to your bill, and a budget alert is the only mechanism that tells you about a problem on day two rather than at the end of the month.

### Where it lives

A **region** is a geographic place: Mumbai, Singapore, Virginia. A **zone** is one independent data centre inside a region. Two rules cover almost everything: pick the region closest to your users on day one — Nashik to Mumbai is single-digit milliseconds, Nashik to Virginia about 250 each way — and keep your app and database in the same region, because a fast app in Mumbai with a database in Virginia pays the ocean crossing on every query.

This is the money chapter: chapter 3's firewall is a cost control as much as a security one, and M11 asks for your billing alert screenshots.

### Words to own

| Term | Plain meaning | Example |
|---|---|---|
| Region | A geographic location for your resources | Mumbai, Singapore |
| Availability zone | One independent data centre in a region | Survives a power failure |
| Latency floor | The delay physics imposes, regardless of code | ~250 ms to Virginia |
| Egress | Data leaving the provider, charged per GB | The surprise line |
| Ingress | Data coming in, usually free | Uploads to a bucket |
| Pay-as-you-go | Metered, no commitment | The default |
| Free tier | A monthly allowance before charges begin | Check the date |
| Budget alert | An email or SMS at a spend threshold | Set it first |
| Spend cap / hard limit | Stops service rather than charging more | Rare; use if offered |
| Reserved / committed use | Cheaper for a 1–3 year commitment | Not for students |
| Orphaned resource | Something running that nothing uses | A week-two VM in week nine |
| Retry storm | Failing calls retried fast, multiplying load | Needs backoff |

### Tradeoffs and gotchas

- **Set the budget alert before you create the first resource.** Four minutes per account, at a number you would be unhappy to see; ₹500 is a sensible student threshold. An alert does not stop spending — it tells you on day two instead of day thirty, the difference between ₹400 and ₹41,000.
- **A free tier is not a spend cap.** What happens at the first unit past the allowance is either a charge or a stop, depending on the provider and the service. Find out per service, and use a hard spend limit if one is offered.
- **Five ways a student team gets a bill:** the thing left running, the compromised machine, the runaway loop with no backoff, the load test, the free trial ending. Only one of those has anything to do with success.
- **Deleting the resource is not always deleting the cost.** A deleted VM may leave its disk, snapshot, address and load balancer behind, each metered. Check the billing page next day.
- **Changing region later is a migration, not a setting,** and the card belongs on the account with the tightest alert and fewest people with access.

### Verify it

1. **Reject any estimate without numbers.** Which meter, how many units, at what rate, totalling what per month, from which page.
2. **Check every quoted limit and price against the provider's page today,** and write the date beside it.
3. **Ask what happens at the free-tier limit — stop, or charge?** Per service; the answers differ, and that difference is the risk.
4. **Ask what egress this design produces.** Anything serving large files straight from a bucket has an egress line; make it say the number.
5. **Ask what is left behind when I delete this.** Disks, snapshots, addresses, logs.

Red flags: a cost claim with no date or source page; "the free tier is generous, you will be fine"; a retry loop with no backoff; a load test with no egress estimate; infrastructure created in a chat that nobody wrote down.

## Walkthrough
1. Open the billing page of every cloud account your team has. All of them, including the one someone created in week one and forgot.
2. For each account, find the current month's spend. Write the number down even if it is zero, and note whether a card is attached.
3. Create a budget alert on each account at a number you would be upset to see — ₹500 is a reasonable student threshold — and add a second at double that. Send it to all three teammates, not just one.
4. Trigger or test the alert if the provider allows it, and confirm an email actually arrives at an address someone reads daily.
5. Find the region of every resource you own: app, database, storage bucket. Write them down. If the app and the database are in different regions, you have found your biggest latency problem and your first action item.
6. Find your own latency floor. Ask your AI assistant for a read-only command that measures round-trip time to your app's host, run it, and write the number down.
7. Walk the resource list in every account and flag anything nobody can explain. One sentence per resource: what it is for, who created it, and whether it should exist tomorrow.
8. Delete one orphaned resource. The next day, check the billing page and confirm the line is gone — and look for the disk or address it left behind.
9. Find today's free-tier limits for your three main services. Write each number with today's date beside it in `STACK.md`, and note for each whether exceeding it stops service or charges you.
10. Write your team's one-line cost rule and put it in the repository. Something you will actually follow, like: nothing is created without a teammate knowing, and nothing is left running past the day we finish with it.
11. Finally, ask an AI for the monthly cost of your current architecture at a hundred users and insist on a number per meter. Then check its two biggest claims against the provider's pricing pages yourself.

## Talk to AI
```prompt
CONTEXT:      I am a student in Nashik, India, on an AI-native engineering
              program. My team of three is building [describe your product in
              two sentences]. Our users are students on campus, on budget
              Android phones and slow 4G. We are on free tiers. We are on
              student budgets - an unexpected bill of a few thousand rupees
              would be a genuine problem, not an inconvenience. Our current
              resources are [list every cloud resource and provider you have,
              including anything created for an experiment].

GOAL:         Make it structurally difficult for us to receive an unexpected
              cloud bill, and choose regions correctly before it is expensive
              to change them.

REQUIREMENTS: 1. Explain the difference between a region and an availability
                 zone, and tell me which region my resources should be in
                 given where my users are
              2. Explain why my app and my database must be in the same
                 region, with a realistic latency number for the wrong choice
                 and what it does to a page that makes six queries
              3. Explain the four things clouds meter - compute time, storage,
                 egress, operations - and which one most often causes a
                 surprise bill for a small team
              4. Walk me through setting a budget alert on each of my
                 providers, and tell me clearly whether each provider offers a
                 hard spend cap or only an alert
              5. Give me the five most likely ways MY specific setup could
                 generate an unexpected bill, ranked, with the one preventive
                 action for each

CONSTRAINTS:  Free tier only. Plain English, second-language reader. Amounts in
              rupees. Do NOT state any price or free-tier limit as a fact -
              for every number, tell me the exact page to open and which value
              to read, and remind me to write today's date beside it. Label
              every console action as reversible or not.

NON-GOALS:    Do not discuss reserved instances, committed-use discounts,
              savings plans, or cost allocation tags. Do not suggest moving
              providers. Do not optimise cost at scale - I have no users yet.

ACCEPTANCE:   - Every account we own has a budget alert at a stated rupee
                figure, going to all three of us
              - I can name the region of every resource we have, and app and
                database are in the same one
              - For each of my main services I know whether hitting the free
                tier limit stops service or starts charging
              - I can name the resource in our account most likely to be
                costing money for no reason
              - I have a written one-line team rule about creating and
                deleting resources

ASK:          Start by asking me which providers we use and what each resource
              is for, including anything we may have forgotten. Then, before
              giving me steps, tell me which ONE action gives the largest
              reduction in risk for four minutes of work, and why.
```

## Stuck? Ask AI
```prompt
Explain the difference between a free tier and a spend cap as if I have confused them, because I have. Then, for each of these providers [list yours], tell me what actually happens the moment I cross a free-tier limit - does the service stop, throttle, or start charging - and tell me which page in each console shows me the answer rather than asking me to trust you.
```

```prompt
Here is my current architecture and where each part runs: [list each resource and its region]. Calculate the extra latency my users experience because of these region choices, show your working with a realistic round-trip number, and tell me what it means for a page that makes six database queries. Then tell me what changing region would actually involve for each resource.
```

```prompt
I want to run a load test against my own staging environment for twenty minutes. Before I do, tell me which meters that will move, estimate the egress in gigabytes with your assumptions stated, and give me a rupee figure with the pricing page I should check it against. Then tell me the cheaper way to get the same information.
```

## Quiz

### Q1 · L3
Your app is in Mumbai and your database is in Virginia. A page makes six queries. What is the practical consequence?
- (a) Negligible — the queries run in parallel
- (b) Roughly three seconds of unavoidable waiting, because each query crosses the ocean and back ✅
- (c) Higher storage costs, but the same speed
- (d) The database refuses cross-region connections
> A round trip to Virginia is around 250 ms each way. Six sequential queries pay it six times, before any processing. Keeping app and database in one region is the highest-value choice in this chapter.

### Q2 · L2
Which of the four cloud meters most often produces a bill a student team did not expect?
- (a) Storage
- (b) Compute time
- (c) Egress — data leaving the provider ✅
- (d) Operations
> Data coming in is usually free and data going out is charged per gigabyte. Load tests, database dumps and uncompressed images served straight from a bucket are the usual causes.

### Q3 · L4
A teammate says "we are on the free tier, so we cannot be charged". What is wrong with that reasoning?
- (a) Nothing — free tiers are hard limits by definition
- (b) A free tier is an allowance, not a cap; what happens past the limit is per-service and may be a charge ✅
- (c) Free tiers only apply to compute, never to storage
- (d) Free tiers require a card, so charges are always expected
> Free tier and spend cap are different mechanisms. Some services stop at the limit; others keep serving and bill you. You have to check, per service, and write down what you find.

### Q4 · L4
Which of these five scenarios is the most common cause of an unexpected student cloud bill?
- (a) The product became popular and traffic grew
- (b) A VM created for a week-two experiment still running in week nine ✅
- (c) A pricing change by the provider
- (d) Storing too many database backups
> Orphaned resources top the list, followed by compromised machines and runaway loops. Almost no student bill is caused by success; nearly all are caused by something left on, broken into, or looping.

### Q5 · L5
An AI estimates your monthly cost as "well within the free tier, you should not see a bill". Why must you reject this, and what do you demand instead?
- (a) Reject it because AI cannot do arithmetic; ask a teammate
- (b) Accept it, but set a budget alert as a backup
- (c) Reject it because it has no numbers, no meters, no rates and no date; demand units per meter, a rate, a total, and the page each rate came from ✅
- (d) Reject it and move to a provider with simpler pricing
> An estimate without numbers cannot be checked, and remembered prices carry no date. The module's recurring red flag — a confidently quoted limit that is out of date — is at its most expensive here, and a budget alert is a safety net rather than a substitute.

## Go deeper
- [AWS free tier: per-service limits and what is 12-month versus always-free](https://aws.amazon.com/free/)
- [Cloud latency between regions, measured live](https://www.cloudping.info/)
