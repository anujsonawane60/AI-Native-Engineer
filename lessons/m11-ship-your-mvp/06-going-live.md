---
id: m11-06-going-live
module: m11-ship-your-mvp
slug: going-live
title: "Going Live: Domain, TLS, Monitoring and Analytics"
order: 6
duration_min: 30
is_free: false
mission: false
volatility: low
last_reviewed: 2026-11-16
---

## Key learnings
- Put your product on a name people can type, with TLS working everywhere
- Find out it is broken from a tool rather than from a user
- Instrument exactly one event for your north star before you have users
- Separate the environment you break from the one strangers use
- Know how to roll back before the first time you need to

## Lesson

### Where this sits in the calendar

**Read this at the start of the final week of the Sprint**, after Checkpoint 3 and before Checkpoint 4. Checkpoint 4 asks for ten real users and a measured north star. Neither is possible on a URL nobody can remember, with no way of knowing the product broke at 9 pm, and no event recording the action you claim to care about. This chapter is the two days of work that makes the last gate achievable.

The underlying material is elsewhere: DNS and domains in M02.12, HTTPS and TLS in M02.14, certificates again at M06.13, monitoring in M07.4–M07.7, events and funnels in M07.8, consent and privacy in M07.9. What this chapter adds is the **order you do it in** and the smallest version that is enough.

### The pain

A group finishes Checkpoint 3 on a host-generated URL that looks like `canteen-mvp-a7f3-git-main.example-host.app`. On Monday they start recruiting users. They put the URL in a WhatsApp message; it wraps onto three lines and two people mistype it. One person's browser warns them about the site before it loads anything.

On Wednesday evening the free-tier database hits a connection limit and every save fails for two hours. The group finds out on Thursday afternoon, from a friend who says "I tried it and it said something went wrong, so I gave up."

On Friday they are asked how many people completed an order. They count WhatsApp replies. The number is four, or six, depending on who is counting, and two of those were group members testing. All three failures are a few hours of work to prevent and much harder to fix afterwards — you cannot measure last Wednesday after the fact.

### The idea

**Going live is not deploying. Deploying makes the code run; going live makes it findable, trustworthy, observable and measured.**

Four pieces of work, not optional extras. They are what turns a deployed URL into something a stranger will use and you can learn from.

### The analogy

The stall moves out of the corridor and into an actual shop. Four things change on the first day, and none of them is about the food.

There is a **signboard** with a name people can say to a friend. There is a **licence displayed** where the customer can see it, so a stranger believes the place is real. There is a **shutter with a lock**, and the owner's number painted beside it so someone can report a problem. And there is a **counter book** recording how many plates went out today, because "it felt busy" is not a number.

**Where the analogy breaks:** a shopkeeper walking up at 7 am sees immediately that the shutter is bent. Software fails silently, and nobody walks past your server. Unless you install the thing that notices, the failure is discovered by a user who has already given up — and users who give up do not complain, they just never come back.

### Picture it

```
  what a person types          who answers           what you see
  ┌──────────────────┐   DNS   ┌───────────────┐
  │ myproduct.in     │────────▶│ your host     │
  │   (A / CNAME)    │         │  TLS cert     │
  └──────────────────┘         │  prod env     │
                               │  /health      │
                               └───┬───────┬───┘
                                   │       │
                    uptime ping ◀──┘       └──▶ one event per
                    every 5 min                  core action
                         │                            │
                         ▼                            ▼
                  ┌──────────────┐            ┌──────────────┐
                  │ alert to one │            │ north star   │
                  │ human, once  │            │ number       │
                  └──────────────┘            └──────────────┘
```

A typed name resolves through DNS to your host, which serves your production environment over TLS, answers a health ping, and emits one event per core action — one path tells a human it is broken, the other tells you whether it worked.

### The four pieces, in order

**1. A name, and TLS.** Buy a domain. A `.in` costs a few hundred rupees a year and is the one real expense in this program, so decide as a group whether to spend it; a free subdomain from your host is acceptable if it is short enough to say out loud. Point it at your host with the record type your host asks for — usually an `A` record for a bare domain and a `CNAME` for `www`. Changes take minutes to hours to spread, so do this on Monday, not the night before you recruit.

TLS is then usually automatic: your host issues a certificate once DNS resolves. Your job is to check three things. The padlock appears, with no warning. `http://` redirects to `https://`. And nothing on the page loads over plain `http` — a single image or script on `http` is "mixed content", and some phone browsers will block it or warn, which is exactly the moment a stranger decides your product is not safe.

**2. Separate environments.** You need one place you can break and one place strangers use. Both exist on free tiers: a preview or staging deployment and a production one, with their own environment variables and, if you possibly can, their own database. The failure this prevents is specific and common — a migration tried in production because it was the only environment, deleting the data of users you spent two days recruiting.

Payments stay in test mode for this entire program. If you have a payment step, label it in the interface so a real person does not believe they paid: "test mode — no money moves" next to the UPI button.

**3. Monitoring: error tracking, one uptime check, one alert.** Error tracking catches exceptions your users hit, with the request that caused them. Install it and then cause an error on purpose so you know it works. From M07.7.

An uptime monitor pings a URL every few minutes and tells you when it stops answering. Give it a dedicated `/health` route that actually touches your database and returns a simple ok, rather than your homepage — a homepage can render perfectly while every save is failing, which is precisely the Wednesday-evening failure above.

Then **one** alert, to **one** person, for **one** condition: the site is down, or errors have crossed a threshold. Five alerts in week four becomes a muted channel in week five, and a muted channel is worse than no alert because you believe you are covered. M07.7 calls this alert fatigue.

**4. Analytics: one event, then a funnel.** Before you recruit anyone, send one event when a user completes the core action from your north star in `SCOPE.md`. One. Then add the two or three events before it that make a funnel — opened, started, completed — so that when nine of ten users drop out you can say where.

Do not build a dashboard of forty numbers. M01.5's warning applies: three metrics is a dashboard, and a dashboard lets everyone pick the number that makes them look good.

**Privacy is part of this piece, not a separate chore.** You are about to collect real people's data under India's DPDP framework. The minimum: a plain-language page saying what you collect, why, and how to get it deleted; no personal data in your logs, which M07.5 covers; consent before analytics if your tool sets identifiers; and a working way for a user to ask you to delete their account. Write it in language a second-year student can read, because that is who your users are.

### Where it lives

This is the handover point between building and operating. M07's monitoring and analytics chapters are the depth behind every paragraph above. M11.7 consumes all four pieces directly: ten users arrive through a name they can type, their failures show up in your error tracker, and your north star is a number rather than a count of WhatsApp replies. At the Company tier, M16 turns this one event into a retention cohort and M17 puts a price on it.

### Words to own

| Term | Plain meaning | Example |
|---|---|---|
| Domain | The name people type | `myproduct.in` |
| DNS record | The entry that points a name at a server | `A`, `CNAME`, `TXT` |
| TLS certificate | What makes the padlock appear | Issued by your host |
| Mixed content | An `https` page loading `http` assets | A blocked image |
| Environment | One running copy with its own settings | staging, production |
| Health route | A URL that proves the app really works | `/health`, touches the DB |
| Uptime monitor | A robot that pings you when it stops answering | Every 5 minutes |
| Alert fatigue | So many alerts that all are ignored | Five channels, all muted |
| Funnel | The events before the one that matters | opened → started → completed |
| Rollback | Returning to the last deploy that worked | One click, rehearsed |

### Tradeoffs and gotchas

**Do DNS first, because it is the only step you cannot hurry.** Everything else in this chapter takes minutes once decided. A name change can take hours to spread, and a certificate cannot be issued until it has.

**Cause one real error and one real outage on purpose.** An error tracker you have never seen fire is an assumption. Throw an exception deliberately, confirm it arrives with the request attached, then take the health route offline for two minutes and confirm the alert reaches a human's phone. Both tests take ten minutes and are the only proof that the tooling works.

**One alert, one person, one condition.** Decide now, in `TEAM.md`, who that person is this week, and what they do when it fires. "The group chat" is not a person and will not act.

**Never put personal data in a log line.** A phone number or an email in your logs is a copy of personal data in a place you will forget about, exported to whichever third party stores your logs. Log an id and look the rest up. M07.5 has the detail.

**Analytics measured from Friday cannot tell you about Monday.** The one genuinely unrecoverable item here: everything else you can retrofit, but a week of uninstrumented user behaviour is simply gone.

**Rehearse the rollback.** Find the button or the command, use it once on purpose, and confirm the previous version comes back. The first time you roll back must not be at 10 pm with ten users waiting, and an agent cannot do this for you — it is your host's interface, and your credentials.

### Verify it

Nine checks, and all of them from a device that is not yours.

1. **Type the name on someone else's phone, on mobile data.** It loads, with a padlock, with no warning.
2. **Type `http://` explicitly.** It redirects to `https://`.
3. **Open the browser console and look for mixed-content warnings.** Zero, on every page a user reaches.
4. **Open `/health`.** It answers, and it answers by touching the database rather than returning a hard-coded ok.
5. **Throw an error on purpose.** It appears in your error tracker within a minute, with the request that caused it.
6. **Take the health route down for two minutes.** The alert reaches a human's phone, and you can name who that human is.
7. **Complete the core action once** and confirm exactly one event arrives, with no personal data in it.
8. **Roll back and roll forward again.** Confirm the live site changed both times.
9. **Read your own privacy page as a user would**, and then actually delete a test account using the path you described. If the path does not work, the page is a claim rather than a promise.

The red flags: a monitor pointed at a static page; an error tracker nobody has seen fire; analytics added after recruiting users; a payment step in test mode that does not say so; personal data in log lines; an alert that goes to a group chat; and a rollback nobody has ever performed. Each of these looks like coverage and provides none.

## Walkthrough
1. Decide as a group whether to spend on a domain. If yes, buy it today; if no, choose the shortest subdomain your host offers and say it out loud to check it survives being read over the phone.
2. Add the DNS records your host asks for, then wait. Check from a phone on mobile data rather than from the laptop that configured it, because local caches lie.
3. Once the name resolves, confirm the certificate was issued, then force the `http` to `https` redirect and test it by typing `http://` explicitly.
4. Open every page a user can reach and check the browser console for mixed-content warnings. Fix each one by serving the asset over `https` or hosting it yourself.
5. Separate your environments: a production deployment with its own environment variables, and a staging or preview one you are allowed to break. Give them separate databases if your free tier allows two.
6. If you have a payment step, confirm it is in test mode and label it in the interface so no real person believes money moved.
7. Add a `/health` route that performs a real database read and returns ok, then point an uptime monitor at it with a five-minute interval.
8. Install error tracking, then throw an exception on purpose and confirm it arrives with the request attached.
9. Configure exactly one alert, to one named person, for one condition. Write that person's name and what they do into `TEAM.md`. Then take the health route down for two minutes and confirm the alert reaches their phone.
10. Add one analytics event for your north star action, plus the two or three events before it that form a funnel. Complete the action once yourself and confirm one event arrives with no personal data in it.
11. Write the privacy page in plain language, delete a test account using exactly the path you described, then rehearse the rollback — roll back on purpose, confirm the live site changed, roll forward, confirm again — and write the steps in `TEAM.md` so any of the three of you can do it.

## Talk to AI
```prompt
I am taking a student product live this week, before I recruit real users. I want a launch plan and a verification list, not code. Do not write code in this reply.

WHAT EXISTS
Stack and hosting: [framework, host, database, all free tier]
Current URL: [the host-generated one]
Domain I own or plan to buy: [name it, or say I will use a subdomain]
Users: [who they are, their devices, their network - for example budget Android phones on slow 4G in India]
Core action for my north star metric: [one sentence, from SCOPE.md]
Payments: [say whether there is a payment step and that it is in test mode]
Personal data I collect: [list every field]
Who is on call this week: [one name]

Produce the following, in this order:

1. ORDER OF WORK. The sequence I should do this in, with the step that cannot be hurried first, and a realistic time estimate for each. Tell me which steps block which.
2. DNS AND TLS. The exact record types I will need for a bare domain and for www, what could still show a browser warning after the certificate is issued, and how I would detect mixed content on a phone.
3. ENVIRONMENTS. Given my free tier, how to separate a production environment from one I can break, including what must differ between them. Name the specific disaster this prevents.
4. HEALTH ROUTE. What my /health route should actually do so that it fails when my product is really broken, and what it must not do - including why pointing a monitor at my homepage would mislead me.
5. ONE ALERT. Define a single alert condition for one named person, and tell me what that person should do when it fires. Then tell me three alerts I might be tempted to add and why I should not add them this week.
6. ANALYTICS. Given my core action, name the one event I must send and the two or three events before it that would form a funnel. For each, list the properties worth attaching - and explicitly list what must NOT be attached because it is personal data.
7. PRIVACY. Under India's DPDP framework, write me a checklist for a student product collecting the fields I listed: what my privacy page must say in plain language, what consent I need before analytics, what must never appear in a log line, and what a working deletion path looks like.
8. REHEARSALS. The two failures I should cause on purpose this week to prove my tooling works, and exactly what I should see each time.
9. ROLLBACK. The questions I should answer before I need to roll back, and how to rehearse it safely today.

Then give me a numbered pre-launch checklist I can run from a phone that is not mine, ordered so that the checks most likely to fail come first.

Finally: name the one thing on this list that cannot be fixed retrospectively, and explain why.
```

## Stuck? Ask AI
```prompt
I bought a domain and pointed it at my host, and it still does not load for me. Ask me diagnostic questions one at a time, cheapest first, to work out whether the problem is the DNS record, propagation and caching, the certificate, a redirect loop, or my app itself. After each answer tell me what you have ruled out, and tell me how to check from a device that is not the one I configured it on.
```

```prompt
Explain what a health check route should do, using a non-software example such as checking whether a shop is really open rather than just unlocked. Then tell me why pointing an uptime monitor at my homepage can show green while every save in my product is failing, and what my health route should touch instead.
```

```prompt
I am about to collect real users' names, phone numbers and order history for a student product in India. Give me the smallest honest version of a privacy page in plain English that a second-year student could read in a minute, and a list of the four things I must actually be able to do - not just claim - for that page to be true. Then tell me what must never appear in my server logs.
```

## Quiz

### Q1 · L4
Which step of going live should you do first, and why?
- (a) Analytics, because the north star matters most
- (b) DNS, because it is the only step whose delay you cannot shorten ✅
- (c) Error tracking, because failures are the biggest risk
- (d) The privacy page, because it is required by law
> Everything else takes minutes once decided. A name change spreads over minutes to hours and the certificate cannot be issued until it has, so DNS on Monday rather than Thursday night.

### Q2 · L4
Your uptime monitor shows green while every order save has been failing for two hours. What is the most likely cause?
- (a) The monitor's interval is too long
- (b) The monitor is pointed at a page that renders without touching the database ✅
- (c) TLS has expired on the API routes only
- (d) The alert was sent but went to the wrong person
> A static page keeps rendering long after writes stop. A health route that performs a real read or write is what makes the monitor's green mean something.

### Q3 · L3
Why exactly one alert, to one named person, in the final week?
- (a) Free tiers only allow one alert
- (b) More alerts become a muted channel, and a muted channel is worse than none because you believe you are covered ✅
- (c) One alert is cheaper in tokens
- (d) Multiple alerts cause duplicate error reports
> Alert fatigue from M07.7. "The group chat" is not a person and will not act, so `TEAM.md` has to name who is on call and what they do.

### Q4 · L5
Of everything in this chapter, which item genuinely cannot be fixed retrospectively?
- (a) The domain and certificate
- (b) The rollback procedure
- (c) The analytics event for your north star ✅
- (d) The privacy page
> A name, a certificate, a monitor and a rehearsed rollback can all be added later. A week of user behaviour you did not instrument is gone, which is why the event ships before you recruit anyone.

### Q5 · L4
Your product has a UPI step running in test mode. What must the interface do?
- (a) Hide the step until payments are real
- (b) Say plainly that it is test mode and no money moves ✅
- (c) Show a realistic receipt so users are not confused
- (d) Nothing — test mode is self-evident from the payment screen
> A user who believes they paid ₹60 and did not has been harmed. Labelling it is the honest version and also the detail a reviewer checks first on a payment flow.

## Go deeper
- [Let's Encrypt — how certificates are issued and validated](https://letsencrypt.org/how-it-works/)
- [Google SRE Book — monitoring distributed systems](https://sre.google/sre-book/monitoring-distributed-systems/)
