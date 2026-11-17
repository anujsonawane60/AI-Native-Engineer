---
id: m05-05-scaling-lb
module: m05-scale-reliability
slug: scaling-and-load-balancers
title: "Vertical vs Horizontal Scaling; Stateless Servers and Load Balancers"
order: 5
duration_min: 15
is_free: false
mission: false
volatility: low
last_reviewed: 2026-11-16
---

## Key learnings
- Choose between a bigger server and more servers for a given bottleneck
- Explain what "stateless" means and list the four kinds of state that break it
- Say what a load balancer does, including health checks and draining
- Spot the four things in a codebase that stop it running on three instances
- Catch an AI that proposes sticky sessions instead of moving the state out

## Lesson

### The pain

Three hundred students open your app at once and your single free-tier instance stops answering after ninety seconds. You upgrade to the largest instance; it costs money and helps a little. Then you run three copies instead, which is free, and it gets strange: students are logged out every third page load, morning uploads 404 in the afternoon, the rate limiter triples, and a midnight job sends every student three notifications. None of that is a bug in the new copies — your app was remembering things it had no right to remember.

### The idea

**Scale out, not up — but only a stateless server can scale out, so making your server forget everything is the actual work.**

Vertical scaling is a bigger box; horizontal is more boxes behind something that spreads requests. Vertical is simpler and runs out, because there is a largest machine and the top end is expensive. Horizontal is cheaper per unit of capacity and demands that any request can be handled by any copy. Stateless does not mean your app has no data: the *process* holds nothing that matters between requests. M00 said RAM forgets on restart; with three instances it is also three RAMs that never agree.

### The analogy

One counter, forty students, 1:10 pm. Vertical scaling is hiring one superhumanly fast cashier: it helps, and there is a limit to how fast a human gets. Horizontal scaling is opening three more counters, which needs somebody at the door saying "counter two is free" — the **load balancer**, who watches the queues and stops sending people to a counter whose card machine has died, which is a **health check**. State ruins it: if one cashier keeps in her head who has already paid, that student can only be served by her, so the canteen keeps a shared receipt register all four cashiers read and write.

**Where the analogy breaks:** the canteen has one kitchen, and four counters do not cook faster, so past a point the queue moves from the counter to the kitchen — the next chapter.

### Picture it

```
                        ┌───────────────┐
          request ─────▶│ load balancer │
                        └──┬─────┬───┬──┘
         health checks     │     │   │  least connections
                    ┌──────┘     │   └──────┐
                    ▼            ▼          ▼
               ┌────────┐  ┌────────┐  ┌────────┐
               │ app 1  │  │ app 2  │  │ app 3  │   no state
               └───┬────┘  └───┬────┘  └───┬────┘   in here
                   └───────────┼───────────┘
                               ▼
              shared cache  ·  database  ·  object storage
```

Requests arrive at one load balancer that picks a healthy app instance, and because no instance stores anything, all three reach the same shared cache, database and file store for every piece of state.

Run this checklist against your own project and you will find at least two.

| What the code does | Why it breaks on three instances | Where it belongs |
|---|---|---|
| Stores sessions in memory | Only one instance recognises the login | Shared cache or signed cookie |
| Writes uploads to local disk | The file exists on one instance only | Object storage (M04) |
| Keeps counters or caches in memory | Each instance counts separately | Shared cache |
| Runs a scheduled job in the web process | The job runs three times | A separate worker or scheduler |

Each has a tempting shortcut. For sessions it is **sticky sessions**, pinning you to the instance that served you: it works, and quietly costs most of what you were buying, because traffic is pinned rather than spread and a restart still logs those users out.

### Where it lives

The load balancer sits between network and server in M00's four boxes, and on a zero-config host you already have one — that is what happens when you set the instance count to three. Scaling out multiplies your database connections, which is how teams take down their own free-tier database: three instances at twenty connections each against a plan allowing thirty. That is the next chapter. It also buys survival, because if one instance dies the other two still answer.

### Words to own

| Term | Plain meaning | Example |
|---|---|---|
| Vertical scaling | A bigger machine | 1 GB RAM to 8 GB |
| Horizontal scaling | More machines | 1 instance to 3 |
| Instance | One running copy of your server | `app 2` above |
| Stateless | The process keeps nothing between requests | Any instance can serve you |
| Load balancer | Spreads requests across healthy instances | The person at the door |
| Round robin | Each instance in turn | 1, 2, 3, 1, 2, 3 |
| Least connections | Whichever instance is least busy | The shortest queue |
| Health check | A periodic "are you alive?" request | `GET /healthz` |
| Draining | Finish current requests, take no new ones | During a deploy |
| Sticky session | Pin a user to one instance | A crutch for in-memory state |
| Autoscaling | Add and remove instances automatically | 2 instances to 6 at lunch |
| Single point of failure | One thing whose death stops everything | Your only instance |

### Tradeoffs and gotchas

**Vertical scaling is not shameful and is sometimes correct.** Scale app servers out; scale databases up until you have measured a reason not to.

**Autoscaling cannot fix a cold start.** New instances boot and warm their caches, so if your rush arrives in thirty seconds, autoscaling that reacts in two minutes is a comfort rather than a solution.

**Scaling out multiplies your bugs by the instance count,** and will not fix a slow query: three servers waiting on the same 900 ms query serve the same students per second.

**Health checks must mean something.** One that returns 200 whenever the process is alive keeps feeding an instance that has lost its database; one that tests the database on every probe empties your rotation on the first hiccup.

**The load balancer is itself a single point of failure,** which managed platforms solve for you.

### Verify it

- **Ask the one question:** "if I run three copies of this and kill one at random, what breaks?" A good answer names files and state.
- **Red flag: anything written to a local file path.** It works on one instance and generates 404s on three; uploads belong in object storage.
- **Red flag: in-memory sessions, caches or counters called fine** — the same red flag as Chapters 2 and 4.
- **Red flag: sticky sessions offered as the solution.** Ask for the state to be moved instead; if you still pick stickiness, log the reason.
- **Red flag: a scheduler inside the web process.** Ask how many times "every midnight" runs with three instances.
- **Red flag: "a bigger instance" or "raise the memory limit" first.** More memory answers one problem only: needing more memory.
- Prove it in thirty seconds each: three instances, reload ten times still logged in; upload a file, wait, fetch it; check the rate limit is not tripled; restart one instance mid-use and confirm nothing visible happens.

## Walkthrough
1. Find out how many instances your app currently runs. On most zero-config hosts it is one, and the setting is in the service's scaling or instances panel.
2. Before changing anything, search your codebase for writes to a local file path. List every hit. Uploads, generated PDFs, logs written to a file, temporary images.
3. Search for where sessions are stored. If the answer is "in memory" or a plain object, write it down as finding number two.
4. Search for in-memory caches and counters — plain maps or dictionaries holding values between requests. Add them to the list.
5. Search for anything scheduled: cron expressions, intervals, "every night" comments. Note whether it lives inside the web process.
6. You now have a stateless audit. For each finding, write one line saying where that state should live instead: shared cache, object storage, database, or a separate worker.
7. Fix the cheapest finding first — usually sessions, by moving them to the shared cache or to a signed cookie. Brief it as one change, nothing else in scope.
8. Now set your instance count to two. Deploy. Nothing should look different yet.
9. Log in and reload ten times. If you are logged out at any point, your session state is still per-instance; go back to step 7.
10. Upload something, wait a minute, then fetch it. A 404 means a local-disk write you have not moved yet.
11. Finally, restart one instance from your host's dashboard while a teammate uses the app. Write down what they noticed. On a correctly stateless app, the honest answer is nothing.

## Talk to AI
```prompt
CONTEXT:      I am a student directing AI on my team's web app. Stack:
              [your stack]. Hosting: [your host], free tier, currently
              running ONE instance. Database: [your database], free tier,
              with a connection limit of about [X]. Shared cache: [yes/no,
              which]. Object storage: [yes/no, which].

              I want to run three instances behind my host's load
              balancer. Here is what my app does that might be a problem:
              - sessions are currently stored: [where]
              - uploaded files currently go: [where]
              - caches or counters held between requests: [list them]
              - scheduled or background jobs: [list them, and say whether
                they run inside the web process]

GOAL:         Make my app safe to run on three instances, where any
              request may be served by any instance and any instance may
              be restarted at any moment.

REQUIREMENTS: 1. Audit the list above and tell me, item by item, exactly
                 what breaks with three instances and what breaks on a
                 restart
              2. Add anything I missed that you can infer from my stack
              3. For each item, say where the state should live instead,
                 and why that specific place
              4. Rank the fixes by risk of breaking my app during the
                 change, cheapest and safest first
              5. Tell me how many database connections three instances
                 will open in total, and whether that exceeds my limit
              6. Tell me what my health check endpoint should actually
                 check, and what it should deliberately NOT check

CONSTRAINTS:  Free tier only. No new paid services. I cannot change my
              database plan this week. Any fix must be deployable as one
              small change at a time, not a rewrite.

NON-GOALS:    Do not propose sticky sessions as the solution - if you
              think they are genuinely the right call here, argue for it
              explicitly and tell me what I give up. Do not propose
              Kubernetes. Do not refactor files unrelated to the state
              problem.

ACCEPTANCE:   - Every item on my list has a verdict and a destination
              - You have told me the total connection count for three
                instances and compared it to my limit
              - Your ranking starts with a change I could ship today
              - You have named at least one thing I did not list
              - For each fix, there is a test I can run myself in under a
                minute to prove it worked

ASK:          First give me the audit table and the ranking. Tell me which
              single fix I should do first and which test proves it. No
              code until I pick one.
```

## Stuck? Ask AI
```prompt
Explain what "stateless server" means to someone who has never programmed, using an analogy that is not about restaurants or waiters. Then give me six things a web app might store and ask me, one at a time, whether each one is safe to keep in the server's own memory. Do not tell me the answers first.
```

```prompt
I am told sticky sessions would fix my logout problem. Explain exactly how they work, what they fix, and the four things they quietly cost me. Then explain the alternative, and tell me honestly when a small team should pick each one.
```

```prompt
Explain the difference between vertical and horizontal scaling with real numbers and real limits, then tell me three situations where vertical scaling is genuinely the better engineering choice. Finish by telling me which one applies to a typical student project database.
```

## Quiz

### Q1 · L2
What does it mean for a server to be stateless?
- (a) It stores no data anywhere
- (b) The process keeps nothing important between requests, so any instance can serve any request ✅
- (c) It has no database connection
- (d) It never caches anything
> Statelessness is about the process, not the system. State still exists — it lives in the database, the shared cache or object storage, where every instance can reach it.

### Q2 · L3
After moving from one instance to three, students are logged out at random every few page loads. What is the cause?
- (a) The load balancer is misconfigured
- (b) Sessions are stored in each instance's memory, so only one recognises the login ✅
- (c) The database connection limit was exceeded
- (d) The cache TTL is too short
> Each instance has its own memory, and a session created on instance one is invisible to instances two and three. The fix is to move sessions to a shared place, not to pin users to instances.

### Q3 · L4
A nightly job lives inside the web process. You scale to three instances. What happens?
- (a) The job runs once, on whichever instance the load balancer picks
- (b) The job runs three times, once per instance ✅
- (c) The job stops running entirely
- (d) The load balancer coordinates it automatically
> Every instance runs its own copy of the process, including its scheduler. Three notification emails per student is the usual way teams discover this. Scheduled work belongs in a separate worker.

### Q4 · L4
Your bottleneck is a single 900 ms unindexed query. You scale from one instance to five. What is the result?
- (a) Requests become roughly five times faster
- (b) Throughput is unchanged because all five wait on the same slow query ✅
- (c) The query is automatically parallelised
- (d) The load balancer caches the query result
> More counters do not make the kitchen cook faster. Capacity multiplies only the part you added; the shared bottleneck stays exactly where it was, which is why you measure before you multiply.

### Q5 · L5
An AI reviews your single-instance app and recommends upgrading to a larger instance because memory use climbs steadily all day. What should you suspect?
- (a) The instance is correctly sized and the upgrade is right
- (b) A memory leak, which a bigger instance only postpones ✅
- (c) The cache TTL is too long
- (d) The load balancer is holding connections open
> Memory that grows steadily and never falls is a leak, not a capacity shortfall. Scaling up buys hours before the same crash, at a cost, which is why "add more memory" is the most expensive wrong answer on the list.

## Go deeper
- [The Twelve-Factor App: processes and concurrency](https://12factor.net/processes)
- [AWS: what is load balancing](https://aws.amazon.com/what-is/load-balancing/)
