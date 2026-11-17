---
id: m06-05-what-the-cloud-is
module: m06-cloud-linux-devops-testing
slug: what-the-cloud-is
title: 'What "the Cloud" Is: IaaS, PaaS, SaaS, BaaS and Serverless'
order: 5
duration_min: 15
is_free: false
mission: false
volatility: medium
last_reviewed: 2026-11-16
---

## Key learnings
- Place any hosting choice on the ladder from renting a machine to renting a function
- Say what you stop being responsible for at each rung, and what it costs you
- Explain why a student team should start as high up the ladder as possible
- Describe what a cold start is and when it matters to your users
- Name the lock-in you accept when you choose convenience

## Lesson

### The pain

Three teams build the same canteen pre-ordering app in one week. The first rents a bare virtual machine and spends nine days on Linux, a web server, a database, certificates and a firewall. The app itself is a weekend's work, and is not started. The second connects their repository to a platform that deploys it: live on day two. The third writes functions that run only when called — also live, costing nothing, and a student loading the menu after a quiet hour waits three seconds. Three trades; one deliberate choice.

### The idea

**"The cloud" is a ladder of rentals, and each rung up hands more of the boring responsibility to someone else in exchange for less control and more lock-in.**

At the bottom you rent a machine and everything on it is yours; at the top you rent one function for the 200 milliseconds it runs. The question is never "which is best" but "what do I want to stop being responsible for this month".

### The analogy

The canteen ladder, bottom to top. **Rent an empty shop** and you bring the stove, gas, cook and licence — total control, and you fix the plumbing at midnight: IaaS, a virtual machine. **Rent a fitted kitchen**, bring only the recipes: PaaS. **Order cooked food by the plate**: SaaS. **Use a shared central kitchen** doing rice, dal and billing while you cook your one dish: BaaS — hosted auth, database and storage. **Hire a cook paid per dish who goes home between orders**: serverless.

**Where the analogy breaks:** a kitchen you rent stays yours for the month. A provider can change free-tier limits, deprecate a feature or raise a price with thirty days' notice while your product sits on it.

### Picture it

```
  MORE CONTROL                                MORE CONVENIENCE
  you manage                                   they manage
  ─────────────────────────────────────────────────────────▶

  IaaS          PaaS          BaaS         Serverless     SaaS
  ┌──────┐     ┌──────┐     ┌──────┐      ┌──────┐     ┌──────┐
  │ your │     │ your │     │ your │      │ your │     │ you  │
  │ app  │     │ app  │     │ app  │      │ fn   │     │ just │
  ├──────┤     ├──────┤     ├──────┤      ├──────┤     │ use  │
  │ run- │     │══════│     │ auth │      │══════│     │ it   │
  │ time │     │ they │     │ db   │      │ they │     │      │
  │ OS   │     │ run  │     │ files│      │ run  │     │      │
  │ VM   │     │ it   │     │══════│      │ it   │     │      │
  └──────┘     └──────┘     └──────┘      └──────┘     └──────┘
   you          you          you           you          nobody
   patch        deploy       write         write        on your
   Linux        code         features      functions    team
```

The double line in each column is the handover point: above it is your responsibility, below it is the provider's, and the line climbs as you move right.

### Where it lives

| Rung | You stop managing | You give up | Typical student use |
|---|---|---|---|
| IaaS | Hardware, power, network | Nothing much; you own the rest | Learning; one odd dependency |
| PaaS | OS, runtime, restarts, TLS | Choice of OS, some tuning | Your product's web app |
| BaaS | Auth, database ops, storage | Schema freedom, portability | Login, data, file uploads |
| Serverless | Capacity planning entirely | Long-running work, warm state | Webhooks, cron, light APIs |
| SaaS | Everything | All of it; you are a user | Email, analytics, payments |

Read column two downward: every row is work you are not doing, and for a team of three that is the only resource you have. Your product will sit on several rungs at once: a PaaS web app, BaaS login and database from M04, serverless payment webhooks, SaaS email. Chapter 6 maps the services; chapter 7 what they cost.

### Words to own

| Term | Plain meaning | Example |
|---|---|---|
| IaaS | Rent a virtual machine; manage everything on it | A free-tier VM |
| PaaS | Push code, they run it | Connect repo, it deploys |
| SaaS | Finished software you pay to use | Your email provider |
| BaaS | Hosted auth, database, storage, realtime | Your login and data layer |
| Serverless | Code that runs per request, then stops | A payment webhook handler |
| Managed service | Provider handles upgrades and backups | A managed database |
| Cold start | Delay on the first call after idle | 0.1 s to a few seconds |
| Shared responsibility | Who secures what, written down | They secure the host; you secure your app |
| Vendor lock-in | Cost of moving off a provider | A provider-specific database |
| Elasticity | Capacity follows demand automatically | Scales to zero at night |
| Region | Which physical place it runs in | Mumbai vs Virginia |

### Tradeoffs and gotchas

- **Start as high up the ladder as you can stand.** Every hour patching Linux is an hour not spent on what only your team can build. Climb down later, with a reason.
- **Convenience is paid for in lock-in.** Hosted auth and a hosted database save three weeks and put your users' identities in that provider's shape. Know which parts are portable: your logic must not know who stores the rows.
- **Serverless does not fit long work and keeps nothing in memory.** Functions have a time limit, so a large export is cut off mid-run; that work belongs in a queue with a worker. Scaling to zero means caches, counters and sessions vanish between calls, so state belongs in the database.
- **A cold start is the first request after idle,** typically 0.1 to a few seconds. A payment webhook does not care; a student on 4G tapping your menu at 11 a.m. does.

### Verify it

AI is opinionated about architecture and over-builds.

1. **Did it ask about your team before recommending a rung?** A recommendation that arrives with no question about your constraints is about someone else.
2. **Check every free-tier number against the provider's pricing page today.** Free tiers move, and every limit an AI quotes was true the day it was written. Write the date beside the number.
3. **Ask what it would take to move off.** "Rewrite the data layer" is fine as a choice, not as a surprise.
4. **Ask where the cold start lands for your users.** If it is "opening the app", ask for alternatives.

Red flags: Kubernetes or a self-managed VM for a three-person project; a cost estimate with no numbers; session state in a function's memory; one provider for everything.

## Walkthrough
1. List every piece your product currently needs: the web app, the database, login, file storage, email, payments, any background job. One line each, no technology names yet.
2. Against each line, write which rung it should sit on and one sentence of why. Do this with your team before you look anything up.
3. Open your deploy platform's dashboard and find where your web app actually runs. Identify the rung from the names on the page, not from memory.
4. Find the pricing or billing page of that platform. Write down today's free-tier limits for build minutes, bandwidth and instance hours, with today's date beside each number.
5. Repeat step 4 for your database or BaaS provider: row or storage limit, connection limit, and what happens when you exceed it — pause, throttle or charge.
6. Find one serverless function in your project, or create the simplest possible one. Note how it is triggered.
7. Leave it untouched for at least fifteen minutes, then call it and time the response. Call it again immediately and time that. The difference is your cold start, measured rather than assumed.
8. Decide which single user action in your product would be worst to be slow, and check whether anything in its path can be cold.
9. Write a one-paragraph note in your repository naming, for each component, the provider and the honest answer to "what would it take to move off this".
10. Finally, ask your AI assistant to recommend an architecture for your product *as described by you*, including your team size and hours, and compare its recommendation against your step-2 table. Where you disagree, work out who is right.

## Talk to AI
```prompt
CONTEXT:      I am a student on an AI-native engineering program. My team is
              three people, all beginners, with about two hours a day for six
              months. We are building [describe your product in two
              sentences]. Our users are students in India on budget Android
              phones and often slow 4G. We must stay on free tiers - a
              surprise cloud bill is a real problem on our budget. We already
              use [list what you use today, e.g. a hosting platform and a
              hosted database].

GOAL:         Decide, component by component, how high up the cloud ladder
              each part of my product should sit, and understand exactly what
              I am giving up at each rung.

REQUIREMENTS: 1. Explain IaaS, PaaS, BaaS, serverless and SaaS in plain words,
                 with the handover line made explicit - what I manage and what
                 the provider manages at each rung
              2. For each component of MY product, recommend a rung and give
                 one sentence of justification tied to my team size and hours
              3. Explain cold starts concretely: what causes them, a realistic
                 range for the delay, and which of my user actions would be
                 affected
              4. For each recommendation, state the lock-in: what specifically
                 I would have to rebuild to move to another provider
              5. Name the parts of my product that should NOT be serverless,
                 and why

CONSTRAINTS:  Free tier only. Plain English, second-language reader. Do not
              quote any free-tier limit from memory - instead tell me exactly
              which page to open and which number to read, and remind me to
              write the date next to it. Assume I will never administer a
              Linux server by choice.

NON-GOALS:    Do not recommend Kubernetes, Terraform, or a self-managed
              virtual machine for this project. Do not design for a million
              users. Do not add a provider we do not already have unless it
              removes more work than it adds.

ACCEPTANCE:   - Every component of my product has a rung and a one-line reason
              - I can say for each one what I stopped being responsible for
              - I can name the single user action most at risk from a cold
                start in my product
              - For every provider in my stack I can state what moving off
                would cost in work
              - No free-tier number in your answer is stated as fact; each
                points me at a page to check

ASK:          Before recommending anything, propose two overall architectures
              for my product at different rungs - one maximally convenient,
              one more controlled - with the tradeoffs of each, and say which
              you would choose for a three-person student team and why. No
              implementation steps until I pick one.
```

## Stuck? Ask AI
```prompt
Explain IaaS, PaaS, BaaS, serverless and SaaS using an analogy that is not about pizza, cars, or houses. Then describe six real situations - a payment webhook, a nightly report, a login system, a video converter, a marketing website, a live chat - and ask me which rung each belongs on and why. Do not tell me the answers until I have attempted all six.
```

```prompt
Explain what a cold start is, why it happens, and what the delay actually consists of. Then help me work out whether it matters for my product: ask me which user action happens first in a typical session, how often my app is used, and what my users' network is like, and then give me your assessment with a number attached.
```

```prompt
I want to understand vendor lock-in without being paranoid about it. For a stack of [list your providers], tell me for each one: what exactly would have to be rewritten to move away, roughly how long that would take a small team, and whether that risk is worth accepting now. Be concrete and do not just tell me to avoid lock-in.
```

## Quiz

### Q1 · L2
You connect your repository to a platform and it builds and runs your app, handles restarts and gives you HTTPS. Which rung is that?
- (a) IaaS
- (b) PaaS ✅
- (c) BaaS
- (d) SaaS
> You supply the application; the provider supplies and maintains the operating system, the runtime, the process supervision and the certificate. That handover line is what defines Platform as a Service.

### Q2 · L3
Why is "start as high up the ladder as you can stand" good advice for a three-person student team?
- (a) Higher rungs are always cheaper at every traffic level
- (b) Lower rungs are less reliable
- (c) Your scarcest resource is hours, and each rung up removes work only the provider can do more cheaply ✅
- (d) Higher rungs avoid vendor lock-in
> Higher rungs usually *increase* lock-in and are not always cheaper at scale. The argument is time: nine days spent configuring Linux is nine days not spent on your product.

### Q3 · L4
A student's menu page is served by a serverless function. At 11 a.m., after a quiet night, the first visitor waits almost three seconds. What is happening and who should care?
- (a) The database is cold and needs an index
- (b) A cold start — the function's container had to be created, and it matters because this is a human's first interaction of the day ✅
- (c) The free tier throttles the first request of each day
- (d) The 4G connection is responsible for the entire delay
> Scaling to zero means nothing is in memory between calls. Machines calling webhooks do not mind the delay; a student on slow 4G, already paying a network round trip, does.

### Q4 · L4
An AI tells you "the free tier includes 100 GB of bandwidth and 6,000 build minutes". What should you do with those numbers?
- (a) Use them to plan your monthly budget
- (b) Treat them as approximate but directionally safe
- (c) Open the provider's own pricing page, read today's numbers, and write the date beside them ✅
- (d) Ask the AI to double-check its figures
> Free-tier limits change and models repeat remembered figures. An out-of-date quoted limit is a recurring red flag in this module, and asking the same model again produces the same memory, not a fresh fact.

### Q5 · L5
Your team stores a user's session counter in a serverless function's module-level variable. It works locally and behaves erratically in production. Why?
- (a) Serverless functions cannot hold variables
- (b) Each invocation may run in a different container, and containers are discarded when idle, so in-process state is not shared or kept ✅
- (c) The function's memory limit is too low
- (d) The provider encrypts variables between calls
> Scaling to zero cuts both ways: nothing running means nothing costing and nothing remembered. State that must survive between requests belongs in the database, not in the process.

## Go deeper
- [Cloudflare: IaaS vs PaaS vs SaaS explained](https://www.cloudflare.com/learning/serverless/glossary/platform-as-a-service-paas/)
- [Google Cloud: what serverless computing is and is not](https://cloud.google.com/discover/what-is-serverless-computing)
