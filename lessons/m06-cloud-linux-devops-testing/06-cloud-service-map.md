---
id: m06-06-cloud-service-map
module: m06-cloud-linux-devops-testing
slug: cloud-service-map
title: "The Cloud Service Map, and Zero-Config Platforms"
order: 6
duration_min: 25
is_free: false
mission: false
volatility: medium
last_reviewed: 2026-11-16
---

## Key learnings
- Reduce hundreds of cloud service names to eight categories you can reason about
- Translate a service name you have never seen into the category it belongs to
- Say what a zero-config platform does for you and what it hides
- Choose between a big provider and a zero-config platform for a student project
- Spot an architecture that names services instead of solving a problem

## Lesson

### The pain

A team asks an AI how to host their canteen pre-ordering app. They get a diagram with nine boxes and eleven acronyms. There is a load balancer, a container service, an object store, a managed cache, a message queue, a secrets manager, a monitoring service and an identity provider. It is a genuinely reasonable production architecture.

They spend a week on it. At the end they have an identity provider configured, nothing deployed, a half-finished network configuration they do not understand, and a quiet suspicion that they are not clever enough for this. Meanwhile the team next to them connected their repository to a platform, clicked once, and has students using the app.

The first team's mistake was not ambition. It was that they received a list of *names* when what they needed was an understanding of *categories*. Nine names look like nine things to learn. Eight categories look like eight questions, most of which their product does not yet ask.

### The idea

**Every cloud provider sells roughly the same eight things under different names, and a student team needs two or three of them.**

Once you can sort any service name into a category, the catalogues stop being frightening. You no longer need to know what a particular product is called; you need to know whether you have a compute problem, a storage problem or a networking problem, and then look up what this provider calls their answer.

### The analogy

The canteen manager buys from a wholesale market. There are two hundred stalls and thousands of product names, and the manager does not know most of them. What they know is the *aisles*: grains, vegetables, oil and dairy, spices, gas, disposables, cleaning, equipment. Eight aisles.

A new stall shouting an unfamiliar brand name is not a new problem. The manager asks one question — which aisle is this? — and instantly knows whether they need it this week, roughly what it should cost, and what it would replace.

A zero-config platform is the vendor who delivers a pre-packed box: everything a small kitchen needs, one price, one phone number. You lose the ability to pick your own brand of oil. You gain not having to go to the market.

**Where the analogy breaks:** aisles at a market stay where they are. Cloud providers rename services, merge them, deprecate them, and launch new categories. The names in any article you read — including anything an AI remembers — drift within months. The categories are stable; the names are not, which is exactly why this chapter teaches the aisles.

### Picture it

```
  ┌────────────┬───────────────────────────────────────┐
  │ COMPUTE    │ run my code: VMs, containers, fns     │
  │ STORAGE    │ keep my files: object store, disks     │
  │ DATABASE   │ keep my rows: SQL, NoSQL, cache        │
  │ NETWORK    │ get traffic to it: DNS, CDN, balancer  │
  │ IDENTITY   │ who is this: users, keys, roles        │
  │ MESSAGING  │ work for later: queues, events, cron   │
  │ OBSERVE    │ what is it doing: logs, metrics, alerts│
  │ BUILD      │ test and ship it: CI/CD, registries    │
  └────────────┴───────────────────────────────────────┘
      ▲
      │  every provider's catalogue is these eight
      │  aisles with different signage
```

Any cloud service you encounter belongs to one of eight categories, and identifying the category tells you what the service is for before you know anything about the product itself.

### The eight aisles, with what your product actually needs

| Aisle | The question it answers | What a student team needs |
|---|---|---|
| Compute | Where does my code run? | One thing: your app |
| Storage | Where do uploaded files live? | Object storage, if you have uploads |
| Database | Where do my rows live? | One managed database |
| Network | How does traffic reach me? | A domain and TLS, usually free |
| Identity | Who is this user, and what may they do? | Hosted auth from M04 |
| Messaging | What happens later, or on a schedule? | Maybe one queue or cron job |
| Observability | What is it doing right now? | The platform's log view |
| Build and ship | How does a push become a deploy? | CI, which is chapter 12 |

Read the right-hand column. For most teams in this program the honest answer is compute, database, identity and the platform's built-in log view — four of eight, and three of those are a single click on a zero-config platform. Storage joins the list the day you add photo uploads. Messaging joins the day you have work that must not happen during a request.

That is the entire service map you need before M11. Everything else exists and is not yours yet.

### Zero-config platforms

A zero-config platform takes your repository and runs it. It detects the framework, installs dependencies, builds, deploys, gives you HTTPS on a subdomain, restarts your app when it crashes, keeps logs, and gives you a preview URL for every pull request — which is chapter 12's subject and one of the most useful things in this program.

What it hides is deliberate and worth naming, because what is hidden is what you cannot debug. You do not choose the Linux distribution, so an odd system dependency may simply not be installable. You do not see the load balancer, so you cannot tune it. You get the platform's regions, not every region. The build has a time and memory limit that you will discover by hitting it. And the scaling behaviour is the platform's policy, not yours.

For a team of three students with two hours a day, that trade is overwhelmingly correct. The one thing to protect is the exit: if your app runs in a container (chapter 8) and your secrets come from environment variables rather than platform-specific calls, moving providers is a day's work rather than a rewrite.

### Where it lives

This chapter is the index to the rest of the module. Compute is chapters 8 and 9. Network is chapter 13. Build and ship is chapter 12. Observability is M07. Database and identity were M04. Messaging was M04's background work. M05's load balancers and caches are the Network and Database aisles at scale.

When M13 revisits all of this for real — Kubernetes, Terraform, proper release engineering — the aisles are the same and the depth changes. The categories you learn here are the ones you will still be using in your first job, long after every product name in any current article has changed.

### Words to own

| Term | Plain meaning | Example |
|---|---|---|
| Object storage | A bucket for files, addressed by key | User-uploaded dish photos |
| Managed database | A database the provider operates | Backups and patching included |
| CDN | Copies of your files near your users | Faster menu images in Nashik |
| Load balancer | Spreads requests across servers | M05's horizontal scaling |
| Secrets manager | A safe place for keys the app reads at run time | Your payment API key |
| Container registry | Where built images are stored | Chapter 8 |
| Zero-config platform | Connect a repo, get a deployed app | Detects the framework for you |
| Build minutes | Metered time your builds consume | A real free-tier limit |
| Egress | Data leaving the provider, usually charged | The surprise line on a bill |
| Managed vs self-hosted | They run it, or you do | Patching is theirs, or yours |
| Service quota | A cap on how much of a service you can use | Raised by request |

### Tradeoffs and gotchas

**Count the services in any proposed architecture and divide by your team size.** Nine services for three beginners is not an architecture, it is a syllabus. A reasonable ceiling before M11 is four. Each additional service brings its own dashboard, its own permissions model, its own failure mode and its own line on the bill.

**The big providers' free tiers are time-limited in a way that platform free tiers are not.** Typically a generous twelve-month trial plus a smaller always-free set. A student who builds on the trial allowance has a cliff edge in month twelve, which may land in the middle of this program. Check which of the two you are using, and write the expiry date in your notes.

**A credit card on file plus a provider with no spend cap is the single biggest financial risk in this module.** Chapter 7 is entirely about this. Mentioning it here because the choice of provider is where the risk is actually taken.

**Service names in any text you read are perishable.** Anything you read today, from an AI or a blog post, may name a service that has been renamed, merged or retired. When a name does not match what you see in the dashboard, trust the dashboard and search for the current name by *category*.

**Multi-cloud is not a goal.** Spreading three services across three providers to avoid lock-in gives you three bills, three permission models and three places to debug, in exchange for a flexibility you will not use this year. Pick one home and keep your app portable instead.

### Verify it

Four checks on any architecture an AI hands you.

1. **Make it name the category for every box.** "What aisle is this in, and what breaks if I remove it?" Any box whose removal breaks nothing you currently have is a box for later. This single question typically cuts nine boxes to four.
2. **Check every service name against the provider's own console today.** The out-of-date name is this chapter's version of the module's recurring red flag — a confidently stated fact about a provider that was true eighteen months ago. If the name is not in the console, the answer is stale and so may be the rest of it.
3. **Ask for the free-tier status of each service, with a page to open.** Not a number from memory. For a big provider, specifically ask whether each service is always-free or trial-only, and when the trial ends.
4. **Ask what it would take to move each component to a different provider.** A component you could move in a day is fine. A component whose move requires rewriting your data model is a decision, and it should be a conscious one.

Red flags: an architecture with a service you cannot explain in one sentence; a recommendation to use a big provider's full stack for a project with no users; a plan whose first week produces configuration rather than a deployed page; and any free-tier figure stated as fact rather than as "check this page".

## Walkthrough
1. Write your product's needs as plain questions, one per line: where does my code run, where do my rows live, who is this user, where do photos go, what happens on a schedule.
2. Beside each, write the aisle from the diagram. Cross out every question your product does not actually ask yet. Count what is left.
3. Open your current platform's dashboard. List every service you are already using, and sort each into an aisle. Most teams find they have fewer than they thought, or one they forgot.
4. Find the platform's build settings page. Write down the build time limit, the memory limit and the build-minute allowance, with today's date.
5. Find the logs view and the environment-variables view. These two pages are where you will spend most of chapters 11 and 12 — know where they are before you need them.
6. Now open the console of one large cloud provider and search for "storage". Count how many distinct products come back. Sort the first five into aisles without reading their marketing copy.
7. Pick one service name from that list and find its pricing page. Identify whether it is always-free, trial-only, or paid from the first byte. Write that down.
8. Ask your AI assistant for a production architecture for your product. Count the boxes. Then ask it, for each box, "what aisle, and what breaks if I remove it?" and record the shortened list.
9. Compare its shortened list against your step-2 list. Discuss as a team where you disagree and why.
10. Write a short `STACK.md` in your repository: one line per service, its aisle, its provider, its free-tier status with today's date, and one sentence on what moving off it would cost.

## Talk to AI
```prompt
CONTEXT:      I am a student on an AI-native engineering program. My team is
              three beginners with about two hours a day. We are building
              [describe your product in two sentences]. Our users are students
              in India on budget Android phones and slow 4G. We are on free
              tiers only and a surprise bill would be a serious problem. We
              currently use [list every service you actually use today].

GOAL:         Understand the cloud service landscape well enough to sort any
              service name into a category, and decide the smallest set of
              services my product genuinely needs before it has users.

REQUIREMENTS: 1. Explain the main categories of cloud service - compute,
                 storage, database, network, identity, messaging,
                 observability, build and ship - and the one question each
                 answers
              2. For MY product as described, tell me which categories I
                 actually need now, which I will need by the time I have a
                 hundred users, and which I do not need this year
              3. Explain what a zero-config platform does for me, and give me
                 an explicit list of what it hides - including what I will not
                 be able to debug or tune because of it
              4. Compare using a zero-config platform versus assembling the
                 equivalent on a large cloud provider, for a three-person
                 student team, with the tradeoffs stated honestly
              5. For every service you mention, tell me the category it is in
                 and whether it is a managed service or something I operate

CONSTRAINTS:  Free tier only. Plain English, second-language reader. Maximum
              four services in your recommendation. Do not state any pricing
              or free-tier limit as fact - tell me which page to open and
              which number to read, and remind me to date it. If a service
              name may have changed recently, say so.

NON-GOALS:    Do not recommend multi-cloud. Do not include a queue, a cache,
              a container orchestrator or a secrets manager unless you can
              name the specific thing in my product that breaks without it.
              Do not give me a diagram with more boxes than my team has
              people.

ACCEPTANCE:   - I can take a service name I have never seen and name its
                category
              - My recommended stack has four or fewer services and I can
                explain what each one is for in one sentence
              - For each service I can say what breaks if I remove it
              - I know which pages to open to confirm each free-tier limit
              - I can state what the zero-config platform is hiding from me

ASK:          First ask me what my product actually does and what my team has
              already built, so you are not designing for a generic app. Then
              propose two stacks - the smallest thing that works, and the one
              you would choose if we had six months and no other commitments -
              with the tradeoffs of each. No setup steps until I pick one.
```

## Stuck? Ask AI
```prompt
Give me twelve real cloud service names from any provider, mixed up, without saying what they do. I will sort each into one of these categories: compute, storage, database, network, identity, messaging, observability, build and ship. Then tell me which I got right and explain the ones I got wrong. Do not reveal any answers before I have attempted all twelve.
```

```prompt
Here is an architecture an AI proposed for my student project: [paste it]. For every single box, tell me which category it belongs to, what specific thing in my product breaks if I delete it, and whether a three-person team with two hours a day needs it before they have a hundred users. Then give me the shortest version of this architecture that still works, and be blunt about what I lose.
```

```prompt
Explain what a zero-config deploy platform is actually doing between my git push and my live URL, step by step. Then tell me exactly what it is hiding from me: which decisions it made on my behalf, which of those I cannot change, and which of them would become a problem if my app needed an unusual system dependency.
```

## Quiz

### Q1 · L2
Object storage, as a category, solves which problem?
- (a) Running your application code
- (b) Keeping uploaded files, addressed by a key ✅
- (c) Spreading requests across several servers
- (d) Storing rows you can query with SQL
> Object storage is a bucket for files — photos, PDFs, exports — fetched by key rather than queried. Rows belong in a database; request spreading is a load balancer.

### Q2 · L3
An AI proposes a nine-service architecture for your three-person team's first deploy. What is the most useful question to ask about each box?
- (a) How much does it cost per month?
- (b) Which provider offers the best version of it?
- (c) Which category is this, and what in my product breaks if I remove it? ✅
- (d) Can it be replaced by something open source?
> Cost and provider choice matter later. The question that actually shrinks the diagram is what breaks without it — most boxes in an over-built architecture protect against problems you do not have yet.

### Q3 · L4
A tutorial names a cloud service that does not appear anywhere in your provider's console. What is the right conclusion?
- (a) Your account lacks permission to see it
- (b) The service has likely been renamed, merged or retired — trust the console and search by category ✅
- (c) The service is only available in certain regions
- (d) The tutorial is wrong about the whole architecture
> Provider catalogues churn constantly. Names are perishable; categories are stable. Searching for what you need by aisle finds the current product, whatever it is called this year.

### Q4 · L4
What is the most important difference between a large provider's free tier and a zero-config platform's free tier for a student on this program?
- (a) The platform's tier is always more generous
- (b) The large provider's generous allowance is often a time-limited trial with a cliff edge, which may land mid-program ✅
- (c) The platform's tier requires no account
- (d) Large providers do not offer free tiers at all
> Big providers typically combine a twelve-month trial with a much smaller always-free set. Building on the trial allowance creates a dated cliff. Check which one you are on and write the expiry date down.

### Q5 · L5
Your team spreads compute on one provider, the database on a second and file storage on a third, specifically to avoid lock-in. What have you actually bought?
- (a) Genuine resilience against one provider failing
- (b) Three bills, three permission models and three debugging surfaces, for flexibility you will not use this year ✅
- (c) Lower cost, because each provider's free tier is used separately
- (d) Easier migration, because nothing is coupled
> Multi-cloud for a small team multiplies operational surface without removing the real coupling, which is in your code. Portability comes from containers and environment variables, not from scattering services.

## Go deeper
- [AWS free tier: which services are always-free versus 12-month trial](https://aws.amazon.com/free/)
- [The Twelve-Factor App: the portability principles behind zero-config platforms](https://12factor.net/)
