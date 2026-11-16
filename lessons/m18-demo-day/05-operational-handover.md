---
id: m18-05-handover
module: m18-demo-day
slug: operational-handover
title: "Operational Handover: Runbook, Costs, Access, Known Issues"
order: 5
duration_min: 35
is_free: false
mission: true
volatility: low
last_reviewed: 2026-11-16
---

## Key learnings
- Write a runbook someone else can follow with you silent in the room
- Reconcile every cost line against the real bill, not the pricing page
- Inventory every account and make sure none has exactly one holder
- Publish known issues with workarounds instead of hoping nobody asks
- Name the three ceilings that break first, each with the number it hits

## Lesson

### The pain

It is three weeks after demo day, and the product still has users.

On a Tuesday evening the site stops saving orders. The only person who holds the database password is on a train to Pune with no signal. On the Thursday the domain lapses, because it was bought on somebody's father's card and the renewal notice went to an email address nobody checks. On the Friday a user writes asking for her data to be deleted, and three people spend two hours working out which tables hold rows belonging to one person. The following week a new member joins and loses an afternoon rediscovering the double-order bug that all three original members already knew about and nobody had written down.

Nothing in that product was technically wrong. It had no owner who could act — which is a different failure, and it is the one that actually kills student projects after the course ends.

### The idea

**An operational handover is the five documents that let somebody who is not you keep the product alive: a runbook that has been followed, costs reconciled against the real bill, an access inventory with no single holder, known issues with workarounds, and an ordered list of what breaks first as you grow.**

All five are judged by the same test: **has anyone other than the author actually executed it?** A runbook nobody has followed is fiction. A cost figure nobody has checked against a bill is a guess.

### The analogy

You hand the stall to your cousin for a month. What they need is not your recipes. It is the gas supplier's number, which burner not to use, when the rent is due and to whom, where the spare shutter key is, and the fact that the mixer trips the fuse if the fridge is running. That last one is the known-issues list, and it is always the thing an owner forgets to say — not out of secrecy, but because they stopped noticing it years ago.

**Where the analogy breaks:** your cousin can phone you. Your handover gets used at 11 pm by someone who cannot reach you, and the thing you did not write down is precisely the thing that fails. So the handover is tested the way you tested the product in M11.2: hand it over, stay silent, and watch where they get stuck.

### Picture it

```
  THE FIVE HANDOVER ARTEFACTS, AND WHAT PROVES EACH
  ┌──────────────────────────────────────────────────────┐
  │ 1 RUNBOOK        a teammate deploys and rolls back   │
  │                  from this document alone, timed     │
  │ 2 COSTS          every line matched to last month's  │
  │                  real bill, not the pricing page     │
  │ 3 ACCESS         no account with exactly one holder, │
  │                  and a recovery path tried once      │
  │ 4 KNOWN ISSUES   each with a workaround a non-author │
  │                  could actually follow               │
  │ 5 CEILINGS       what breaks first, and at exactly   │
  │                  what number it breaks               │
  └──────────────────────────────────────────────────────┘
```

Five artefacts, each with the specific proof that distinguishes it from a document written to look complete.

### 1. The runbook

Procedures, written as numbered steps with the exact thing to click, not the concept. Six of them, minimum: deploy; roll back; rotate a leaked or expiring key; restore a backup; and first response for your two or three most likely incidents — the site is down, the model API is failing, payments are not confirming.

"Roll back the deployment" is not a procedure. This is: *open the host dashboard, Deployments tab, find the last deploy before the current one, Promote to Production, confirm, then load `/health` and check it returns ok.* Written that way because it is going to be followed by someone tired, on a phone, who did not build it.

Two more things belong here. **Who is on call, by name, with a date range** — M11.6 said one alert, one person, one condition, and "the group chat" is still not a person. And **where each secret lives**, which is the location, never the value. A runbook with a key in it is a key in a document that gets shared.

### 2. The monthly cost breakdown

One line per service: what it is, which plan, what you **actually paid** last month, what the free tier includes, and what happens when you cross it — does it stop, queue, or start charging.

Then the line most students have never computed: model inference. Cost per call times calls per month, from M09.14 and M17.6, with the number of calls taken from your own usage page. Then the same table projected at a hundred users and a thousand, which is the arithmetic M11.5 asked for and M17.5 turned into unit economics.

Two things reliably missing. **The annual costs** — the domain renewal, once a year, often on a personal card nobody else can see. And **the metered extras you forgot a free tier counts**: bandwidth, build minutes, function invocations, log retention, image transformations. M06.7's billing traps are exactly these.

The pricing page is not your bill. Open the bill.

### 3. The access inventory

Every account, in a table: the service, who holds it, what that account can do, whether two-factor is on, and how you would recover it.

Three rules. **No account with exactly one holder** — that is the bus factor from M18.2's ninth question, and it is the single most common reason a working student product dies. **No credentials in a chat message**, ever; a shared password manager or the host's own team feature instead. **No account owned by a personal email that only one person can read**, which is how domains lapse.

Add a four-line offboarding checklist: what you revoke, what you rotate, what you transfer, and who confirms it. You will not need it during the course. You will need it within a year.

### 4. Known issues and workarounds

A table: the issue, its severity, who it affects, how often, the workaround, and whether somebody who did not build the product could follow that workaround.

**An empty known-issues list is not a clean product. It is an unexamined one.** Every team of three knows at least three — the double submit that occasionally creates two rows, the page that breaks on a very long name, the suggestion that goes blank in Marathi. Writing them down costs you nothing in a defence and earns you a great deal, because the alternative is a reviewer finding one in M18.3 and discovering it was known and hidden.

### 5. What breaks first under growth

An ordered list, each entry with the number at which it hits. Typically: the free tier's row or storage cap; the database connection limit; the model provider's rate limit or your monthly token budget; the nightly job that reads every row and gets slower every week; the free tier's monthly request budget.

Order them by which arrives first, not by which is scariest. Each needs a number — "at about four hundred users" — and that number comes from your load test in M13.7, your usage pages, and the second diagram you drew as M18.1's stretch.

### Where it lives

M11.6 gave you the rehearsed rollback and the single alert. M06.7 is the billing traps, M07.2 the secrets, M13.5 release and rollback, M13.9 backups and disaster recovery, M13.10 on-call and postmortems, M17.5 and M17.6 the money, M17.9 the trust you lose overnight, and M01.9 the working agreements this formalises.

Forward, three of M18.2's ten questions — the lost database, the leaked key and the departing co-founder — are answered directly out of this document. M18.6 closes with three minutes on it.

### Words to own

| Term | Plain meaning | Example |
|---|---|---|
| Runbook | Procedures written to be followed under stress | "Roll back" as nine exact clicks |
| Procedure | One task, numbered steps, no judgement needed | Rotate the model key |
| On call | One named person, for a stated period | "Priya, 1–7 December" |
| Rotation | Replacing a secret and revoking the old one | New key, old one deleted |
| Reconcile | Check your figures against the actual bill | Table says ₹0; bill says ₹240 |
| Access inventory | Every account, holder, scope and recovery | Nine rows, no single holders |
| Least privilege | An account can do only what it must | Read-only analytics login |
| Offboarding | What you revoke when someone leaves | Four lines, agreed in advance |
| Workaround | What to do until the issue is fixed | "Refund manually from the dashboard" |
| Ceiling | A limit you will hit as you grow | 60 connections, at ~400 users |
| Headroom | How far you are from the nearest ceiling | 4× current peak |

### Tradeoffs and gotchas

**Write the click, not the concept.** Every sentence that assumes the reader knows where something is will be the sentence that fails at 11 pm.

**Never put a secret in the runbook.** Put where the secret lives. A document gets pasted into chat; a location does not leak anything.

**Do not hand over knowledge as a video.** A forty-minute screen recording is not a runbook. Nobody watches it during an incident, and nothing in it is searchable.

**A cost table built from pricing pages will be wrong.** Free tiers meter things you have forgotten about, and the first real bill is always a surprise in the same two or three lines.

**The known-issues list is the most honest and the most tempting to shorten.** Resist. A reviewer who finds an issue that was known and unlisted stops believing the rest of the document.

**This is the artefact that most rewards being written early.** Everything in it is a fact you already have — the bill exists, the accounts exist, the issues are known. The work is only in writing them down and then having someone else try them.

### Verify it

Five checks, four of which need another person.

1. **Hand the runbook to a teammate and stay silent while they deploy and roll back.** Time it. Write down every point where they hesitated; each one is a missing step.
2. **Open last month's bill beside your cost table.** Every line matches, or the table is wrong. Fix the table, not the bill.
3. **For every account, have a second person actually log in today.** Not "they have the password" — watch them get in. Any account where this fails has exactly one holder.
4. **Run one procedure you have never run** — restore the backup into a scratch database, or rotate the model key — and correct the steps as you go.
5. **Ask both teammates to name three known issues from memory** and compare with your list. Anything they name that you did not list was known and unwritten, which is the exact failure this artefact exists to prevent.

The red flags: a runbook nobody has followed; a cost figure nobody has checked against the bill; an access row with one holder; an empty known-issues list; a ceiling with no number; credentials pasted in a chat; an annual renewal on a personal card; and a handover written entirely by the one person who built everything, which tells a reviewer about the team rather than the document.

## Walkthrough
1. Create `HANDOVER.md` with the five section headings and nothing under them, and commit it so the file has a start date.
2. Write the deploy and rollback procedures first, as numbered steps naming the exact buttons. Then write key rotation, backup restore, and first response for your two most likely incidents.
3. Hand the document to a teammate and ask them to deploy and then roll back, using only what is written, while you say absolutely nothing. Time it and note every hesitation.
4. Rewrite every step they hesitated at, then have the other teammate do the same test from the corrected version.
5. Open last month's actual bill or usage pages for every service and write the cost table from those numbers, one line per service, including what the free tier covers and what happens at the limit.
6. Add the inference line: cost per call times the number of calls on your usage page. Then project the whole table at a hundred and at a thousand users.
7. Add every annual cost, including the domain, and name whose card it is on and which email receives the renewal notice.
8. Build the access inventory: service, holder, scope, two-factor, recovery path. Then have a second person log in to each account while you watch, and fix every row where that fails.
9. Write the known-issues table. Ask both teammates for three issues each from memory, add anything you missed, and give every row a workaround somebody else could follow.
10. List the growth ceilings in the order they will arrive, each with the number it hits, taken from your load test and your usage pages rather than from guesswork.
11. Run one procedure you have never run before on a copy, correct the steps as you go, and add a decision-log entry recording what you found.

## Talk to AI
```prompt
I have to produce an operational handover for a product my team built, good enough that somebody who is not me can keep it running after I leave. Interrogate it. Do not write the document for me, do not write code, and do not tell me it looks thorough.

MY SYSTEM
As-built architecture and every external service: [paste your map and dependency list]
Tables and what each holds: [paste them]

MY RUNBOOK PROCEDURES, as currently written: [paste them in full - deploy, rollback, key rotation, backup restore, incident first response]
Who is on call, by name and date range: [paste it]
Where each secret lives - the location, not the value: [paste it]

MY COST TABLE, taken from last month's actual bills and usage pages: [paste every line: service, plan, what I paid, what the free tier includes, what happens at the limit]
My model usage last month: [number of calls, and cost per call]
Annual costs: [domain and anything else, with whose card it is on]

MY ACCESS INVENTORY: [service, who holds it, what it can do, two-factor on or off, recovery path]
MY KNOWN ISSUES: [issue, severity, who it affects, how often, workaround]
MY GROWTH CEILINGS: [what breaks first, and the number at which it hits]
My load-test result: [the concurrency at which it started failing]

Work through all of these:

1. FOLLOWABLE. Read each procedure as a tired person on a phone who did not build this. Quote every step that assumes knowledge the reader does not have, and every step that names a concept rather than a thing to click.
2. MISSING PROCEDURES. Given my architecture and dependencies, which procedures are absent that I will certainly need? Rank them by how likely I am to need each in the next three months.
3. SECRETS. Does anything in my runbook leak a value rather than a location? Quote it.
4. COSTS. Which costs are missing entirely for the services I named - metered extras such as bandwidth, build minutes, function invocations, log retention, storage? Then check my arithmetic at 100 and 1,000 users and tell me which line grows fastest.
5. SINGLE POINTS. Go through my access inventory and name every account with exactly one holder, every account on a personal email, and every account that can do more than it needs to.
6. OFFBOARDING. Write me the questions I must be able to answer about what happens the day a member leaves, given my inventory.
7. KNOWN ISSUES. Given my architecture and my tables, name five issues a product like mine almost certainly has that are missing from my list. For each, tell me what a workaround would look like and who would have to perform it.
8. CEILINGS. Based on my stated limits and my load test, put my ceilings in the order they will actually arrive and tell me which of my numbers looks like a guess rather than a measurement.
9. DELETION. Using my table list, write the steps a deletion request would require, and tell me which table I have probably forgotten - including logs and analytics.

Then tell me which of my five artefacts is weakest, and the single test that would prove it.

Finally: name the one thing that will go wrong in the three months after I hand this over that nothing in my document prepares anyone for.
```

## Mission

**Objective.** A `HANDOVER.md` with five sections, where the runbook has been executed by a teammate while you stayed silent, every cost line matches a real bill, and no account has exactly one holder.

**Steps.**
1. **Write six procedures** — deploy, rollback, key rotation, backup restore, and first response for your two most likely incidents, as exact clicks.
2. **Silent test** — a teammate deploys and rolls back from the document alone, timed, while you say nothing.
3. **Rewrite and retest** — fix every hesitation, then have the second teammate repeat the test.
4. **Reconcile costs** — build the table from last month's bills and usage pages, including metered extras and annual costs.
5. **Project** — the same table at a hundred and a thousand users, with the inference line computed from your own usage.
6. **Inventory access** — service, holder, scope, two-factor, recovery; then watch a second person log in to each.
7. **Collect known issues** — three from each teammate from memory, each with a workaround a non-author could follow.
8. **Order the ceilings** — by arrival, each with a measured number.
9. **Run one untried procedure** — restore a backup or rotate a key, on a copy, and correct the steps.

**Upload.**
- `HANDOVER.md` → attached file, plus the repository in `repo_url`
- A recording or timed log of your teammate deploying and rolling back from the document alone
- The list of every point they hesitated at, and what you rewrote
- A screenshot of last month's actual bill or usage pages beside your cost table
- Your cost table at today, at a hundred users and at a thousand, with the inference line shown
- Your access inventory, with evidence that a second person logged in to each account
- Your known-issues table, including the issues your teammates named that you had missed
- Your ordered ceilings, each with its number and where the number came from
- Evidence of the untried procedure you ran, and what you corrected
- Reflection, 150 words max → the step your teammate could not follow, and what that told you

**Self-check.** Tick every box before you submit.
- [ ] Every procedure is numbered steps naming exact things to click
- [ ] A teammate deployed and rolled back using only my document, with me silent
- [ ] I recorded every hesitation and rewrote those steps
- [ ] A second teammate repeated the test on the corrected version
- [ ] No secret value appears anywhere in the runbook, only locations
- [ ] One person is named on call, with a date range
- [ ] Every cost line matches last month's actual bill or usage page
- [ ] My cost table includes metered extras and every annual cost
- [ ] I have named whose card each recurring payment is on and which email gets the renewal
- [ ] The inference line is computed from my own usage, not estimated
- [ ] I have projected the table at a hundred and a thousand users
- [ ] No account in my inventory has exactly one holder, and I watched a second person log in
- [ ] No credentials appear in any chat message
- [ ] My offboarding checklist says what is revoked, rotated, transferred and confirmed
- [ ] My known-issues list includes everything both teammates named from memory
- [ ] Every known issue has a workaround somebody who did not build it could follow
- [ ] Every growth ceiling has a number, and I can say where the number came from
- [ ] I ran at least one procedure I had never run before
- [ ] This submission is my own work; it names which procedures I personally wrote and executed and which parts of the system I personally directed

**Stretch.** Do the deletion drill for real. Create a test account, use the product normally, then delete that user using only the steps in your handover — and afterwards go through every table, log and analytics tool and check whether any row belonging to that account survived. Write 100 words on what you found. This is the DPDP answer in M18.2's fourth question, converted from a claim into a fact.

## Stuck? Ask AI
```prompt
My runbook reads like notes to myself. Take one procedure at a time from what I paste and act as a tired person on a phone at 11 pm who did not build this product. Ask me the question you would actually have at each step - where is that, what is it called, how do I know it worked - and do not move to the next step until I have answered. Do not rewrite my steps; make me rewrite them.
```

```prompt
Explain the difference between what a pricing page says and what a bill says, using a non-software example such as a phone plan with a data cap. Then ask me which services my product uses and, for each, name the metered things a free tier counts that I have probably never looked at. Tell me where in that service I would find last month's real usage.
```

```prompt
I think our product has no known issues. Ask me ten specific questions about things that commonly go wrong in a product like mine - double submits, very long inputs, a second language, a slow network mid-action, an expired session, a failed payment that the user believes succeeded. For each, ask me whether I have actually tested it, and treat "I think it is fine" as a no.
```

## Quiz

### Q1 · L4
What makes a runbook real rather than decorative?
- (a) It covers every procedure the product might ever need
- (b) Someone other than its author has followed it successfully while the author stayed silent ✅
- (c) It was reviewed by the whole team in a meeting
- (d) It is committed to the repository with the rest of the documentation
> A runbook is used under stress by someone who did not write it. Until a non-author has executed it, every assumption in it is untested, and the untested step is the one that fails at 11 pm.

### Q2 · L4
Your cost table says ₹0 a month because everything is on a free tier. Last month's bill says ₹240. What is the most likely explanation?
- (a) The host made a billing error
- (b) A metered extra such as bandwidth, build minutes or log retention crossed its included amount ✅
- (c) The table is correct and the bill includes next month
- (d) One teammate upgraded a plan without telling anyone
> Free tiers meter things nobody thinks about. This is M06.7's billing trap, and it is why the rule is to build the table from the bill rather than from the pricing page.

### Q3 · L3
Why is an empty known-issues list treated as a red flag?
- (a) Because every product is required to document at least three defects
- (b) Because it means the product was not examined, not that it is clean ✅
- (c) Because reviewers assume the issues were fixed badly
- (d) Because it makes the handover document too short
> Every team of three knows at least three rough edges. An empty list means they were not written down, and a reviewer who later finds a known issue stops trusting the rest of the document.

### Q4 · L5
Your access inventory has nine accounts, and the database account is held only by the member who set it up. Why is this the most dangerous row in the document?
- (a) Because database credentials are more valuable than other credentials
- (b) Because a single holder means one unreachable person stops all recovery ✅
- (c) Because it violates least privilege
- (d) Because it cannot be protected with two-factor authentication
> This is the bus factor, and it is the most common reason a working student product dies after the course. The fix is cheap today and impossible on the evening you need it.

### Q5 · L4
Which growth ceiling belongs first in your ordered list?
- (a) The scariest one, so the reader takes the list seriously
- (b) The one that arrives at the lowest number of users, with that number stated ✅
- (c) The one that would cost the most to fix
- (d) The one your host warns about in its documentation
> The list is ordered by arrival, because the first ceiling is the only one that matters this quarter. A ceiling without a number is not a ceiling, it is a worry.

## Go deeper
- [Google SRE Book — on-call, playbooks and why procedures are written down](https://sre.google/sre-book/being-on-call/)
- [PagerDuty — incident response documentation and runbook practice](https://response.pagerduty.com/)
