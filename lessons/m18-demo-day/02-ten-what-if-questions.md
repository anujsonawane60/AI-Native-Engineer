---
id: m18-02-what-ifs
module: m18-demo-day
slug: ten-what-if-questions
title: "The Ten What-If Questions"
order: 2
duration_min: 25
is_free: false
mission: true
volatility: low
last_reviewed: 2026-11-16
---

## Key learnings
- Answer any what-if question in the same five moves instead of from a script
- Name the signal that would tell you first, before you name the fix
- Quote a number from your own system in every answer you give
- Say what you would deliberately not do, which is the stronger half
- Give an honest unknown rather than a bluff, and know why that scores better

## Lesson

### The pain

Three students are asked the same question: *what happens if ten times as many people use this tomorrow?*

The first says they would scale horizontally and add a load balancer. Asked which part refuses work first, they say the server. Asked how many database connections their free tier allows, they do not know. Fluent, and it would have fitted any product in the room.

The second has rehearsed a tidy three-part answer about caching, queues and read replicas. One follow-up — *which would you do first, and what does it cost?* — and the recitation has no next line.

The third says: *the connection pool goes first — my free tier gives sixty and my M13.7 load test refused work at about forty concurrent users. I would know within five minutes, because the uptime check on `/health` touches the database. First hour: turn on the pooler, a setting rather than a deploy. Real fix: cache the menu for sixty seconds, half a day. What I would not do is add a queue I have never run under load.*

All three know the same theory. Only one knows their own system.

### The idea

**A what-if question is not asking for a plan. It is asking whether you know the limits of the thing you built — so answer in five fixed moves, and put a number from your own system in every one.**

The five moves work for all ten questions here and for ones nobody has thought of yet. A script breaks at the first follow-up, and the follow-up is where the question lives.

### The analogy

Ask a canteen owner what happens if the gas cylinder runs out at 1:15 pm. A weak owner says it will not happen. A good one says: the flame drops before it dies, so I get a minute's warning; the spare behind the door takes four minutes to swap; the real fix is a ₹3,200 two-cylinder manifold; and what I will not do is pre-cook sixty plates at noon, because cold poha costs me more than the waiting.

**Where the analogy breaks:** the owner can see and smell the failure, and maybe eight familiar things go wrong in a canteen. Your system has dozens of failure modes, every one invisible, so your answers cannot come from feel. They come from written-down numbers: your bill, your free tier's limits, your load test. A feel-based answer is indistinguishable from a bluff.

### Picture it

```
  A GOOD WHAT-IF ANSWER, IN FIVE MOVES          (90 seconds)
  ┌────────────────────────────────────────────────────┐
  │ 1 BREAKS FIRST  which box, which limit, what number│
  │ 2 I WOULD KNOW  the alert, dashboard or user report│
  │ 3 FIRST HOUR    the cheapest thing I can do today  │
  │ 4 REAL FIX      what it costs, in rupees and days  │
  │ 5 WOULD NOT DO  the tempting move I would refuse   │
  └────────────────────────────────────────────────────┘
```

Five moves in a fixed order, each about fifteen seconds, with a number of your own in moves one, three and four. Move five distinguishes an engineer from a student: naming the tempting wrong move shows you considered more than one option — the REJECTED line from your M01.10 decision log, spoken out loud.

One clause belongs in your vocabulary before the room, not after: *I do not know, and here is how I would find out.* Finish it properly — the dashboard you would open, the number you would look for, how long it would take. A panel marks that as a strong answer. A bluff that collapses on one follow-up is fatal, because after it every other answer becomes suspect.

The ten questions are not a secret, and together they cover every box on your M18.1 map.

| What if… | What it really tests | A good answer names |
|---|---|---|
| Ten times the traffic arrives tomorrow | which limit you hit first | your connection cap, your peak hour, the query that slows |
| The payment webhook fires twice | idempotency | your idempotency key, the unique constraint, what a duplicate costs a user |
| Your AI provider goes down or doubles its price | dependency and margin | your cost per call, your monthly inference bill, the fallback the user sees |
| A user asks you to delete their data | DPDP, and whether you know your own data | every table holding their rows, your deletion path, the days you promise |
| Your database is lost | a backup you have actually restored | when the last backup ran, how much data you would lose, how long a restore takes |
| A key leaks | rotation rather than panic | where the key lives, what it can do, how long rotation takes, what you check in the logs |
| A dependency has a critical CVE | supply chain | how you would hear, whether you use the vulnerable path, how fast you can ship |
| Your best channel stops working | channel concentration | the share of signups it carries, what your second channel costs per user |
| Your co-founder leaves | bus factor and access | what only they hold, what your handover covers, what you would do in week one |
| A competitor ships your main feature | defensibility | what you have that is not the feature: data, users, distribution, a workflow |

A worked answer, so the shape is concrete:

```
  Q: the payment webhook fires twice
  1 A second payment row, and a second order confirmed to the
    user. It happened once in test mode in October.
  2 I would know from the duplicate-order check: two orders,
    same user, same amount, under sixty seconds apart.
  3 First hour: read the logs for the gateway reference and
    count how many duplicates actually got through.
  4 Real fix, already shipped in M04.17: a unique index on the
    gateway's payment reference, so the second insert fails
    instead of paying. Cost: nothing, half a day.
  5 I would not add a retry on my side without an idempotency
    key on the request - that is how you manufacture the double
    charge you were trying to prevent.
```

One question, five moves, a real index on a real table, a refusal with a reason.

The last two are not technical, and they are where strong engineers most often fail. *Your co-founder leaves* means: which credentials and knowledge exist in exactly one head. *A competitor ships your main feature* means: what do you have that is not copyable in a weekend.

### Where it lives

Every question draws on a module you have done: traffic from M05 and M13.7, duplicate webhooks from M04.17, model cost from M17.6, deletion from M16.11 under India's DPDP framework, database loss from M13.9, keys from M07.2, the CVE from M13.11, channels from M17.7, the co-founder question from M18.5, the competitor question from M17.1. Forward, these ten answers are the material for M18.6's questioning block.

### Words to own

| Term | Plain meaning | Example |
|---|---|---|
| Mitigation | A cheap action that reduces harm now | Turn on the pooler, cache for 60s |
| Idempotent | Doing it twice has the same effect as once | A unique index on the payment reference |
| Blast radius | What else fails when one thing fails | Model down: suggestions stop, orders do not |
| Recovery point | How much data you would lose | "Up to 24 hours, last nightly backup" |
| Recovery time | How long getting back takes | "About 40 minutes, I have done it once" |
| Rotation | Replacing a secret and invalidating the old one | New model key, old one revoked |
| CVE | A publicly catalogued vulnerability in software you use | A critical one in your image library |
| Channel concentration | How much of your growth one source carries | 70% of signups from one WhatsApp group |
| Bus factor | How many people must vanish before you stop | One, if only she has the database password |
| Honest unknown | Saying you do not know, plus how you would find out | "I would open the usage page and read the per-call cost" |

### Tradeoffs and gotchas

**Do not memorise ten answers.** A memorised answer is detectable in two sentences: too smooth, no number from your own product, no second paragraph. Learn the five moves and the numbers; the sentences get built in the room.

**Every number you quote will be asked for its source.** A number you cannot source is worse than no number, because it reads as invention.

**"That will not happen" is the one fatal answer.** Even for genuinely unlikely scenarios, say how unlikely and why you accepted the risk — an accepted risk, named, is good engineering.

**Do not promise work you have not costed.** "We would move to a paid tier" invites: which one, how much a month, at what revenue does it pay for itself.

### Verify it

Five checks, out loud, with a timer.

1. **Answer all ten cold, ninety seconds each, recorded.** Count how many contained a number of your own.
2. **For every number, open its source in front of someone.** Anything you cannot show in thirty seconds comes out of the answer.
3. **Have someone ask one follow-up on each** — *how do you know, what would it cost, why not the other option.* A script dies here; the five moves do not.
4. **Count your move-five answers.** Fewer than eight naming something you would not do means you are describing plans rather than choices.
5. **Take the two you answered worst and give the honest-unknown version.** Dashboard, number, time. Practise until it sounds like information rather than apology.

The red flags: an answer that would fit any product in the room; a number with no source; a recitation with no second paragraph; "that will not happen"; a fix with no cost; silence on the co-founder and the competitor; and a confident invention that collapses on one follow-up.

## Walkthrough
1. Create `WHAT-IF.md` and paste the ten questions in as headings, with the five move labels under each one, all blank.
2. Before writing any answer, spend thirty minutes collecting numbers: your last bill, your free tier's stated limits, your load-test result from M13.7, your cost per model call, your monthly active users from M16, your signup sources from M17.
3. Write the three answers you are most confident about first, strictly in the five moves. Keep each to roughly ninety seconds of speech, which is about a hundred and eighty words.
4. For each of those three, check that moves one, three and four each contain a number you can source, and write the source in brackets beside it.
5. Now write the two you are least confident about. Where you genuinely do not know, write the honest-unknown version properly: the dashboard, the number, the time it would take.
6. Answer the deletion question by walking your own tables from M18.1 section 3, naming every table that holds a row belonging to one user, including analytics events and logs.
7. Answer the database-loss question only after checking when your last backup actually ran. If you have never restored one, write that down as your recovery time being unknown.
8. Answer the co-founder question by listing what exactly one person holds today — credentials, accounts, knowledge. Keep that list; it becomes M18.5's access inventory.
9. Answer the competitor question without mentioning the feature. Write only what you have that cannot be copied in a weekend, with a number beside each.
10. Record yourself answering all ten cold, ninety seconds each, then listen back and mark every answer that contained no number of your own.
11. Hand the ten questions to a student from another group and have them ask one follow-up on each. Rewrite the three answers that broke.

## Talk to AI
```prompt
I have an individual technical defence of my own product and I will be asked ten "what if" questions. I want you to interrogate my answers, not write them for me. Do not write code. Do not tell me my answers are good.

MY SYSTEM
As-built architecture, boxes and labelled arrows: [paste your map from ARCHITECTURE.md]
Tables and what each holds: [paste section 3]
External services I depend on: [paste section 6]
Stated free-tier limits I know of: [connections, rows, requests, model rate limits - paste what you actually looked up]
My load test result, if I have one: [concurrency at which it started failing]
Cost per model call and my last monthly bill: [paste the real numbers]
Users and signup sources: [monthly active users, and the share from each channel]
Backups: [when the last one ran, and whether I have ever restored one]
Who holds which credentials: [one line per account]

MY ANSWERS, each written as five moves - breaks first, I would know, first hour, real fix, would not do:
1. Ten times the traffic tomorrow: [paste]
2. The payment webhook fires twice: [paste]
3. My AI provider goes down or doubles its price: [paste]
4. A user asks me to delete their data: [paste]
5. My database is lost: [paste]
6. A key leaks: [paste]
7. A dependency has a critical CVE: [paste]
8. My best channel stops working: [paste]
9. My co-founder leaves: [paste]
10. A competitor ships my main feature: [paste]

Do all of this, in order:

1. GENERIC TEST. For each answer, say whether it would fit any product or only mine. Quote the sentence that makes it specific, or say there is none.
2. NUMBERS. List every number I used and, for each, name the exact dashboard, bill or test output I would have to open to prove it. Flag any number that looks invented.
3. MISSING FIRST FAILURE. For each answer, tell me whether the thing I named as breaking first really would, given my architecture. If something else would break sooner, name it.
4. SIGNAL. For each answer, would I actually find out the way I claimed? Which of my claimed signals does not exist in the monitoring I described?
5. COST. Every real fix I proposed: is it costed? Rewrite any cost I stated vaguely as the question a reviewer would ask instead.
6. MOVE FIVE. Which answers have no "would not do"? For each, name the tempting wrong move I should be refusing.
7. FOLLOW-UPS. For each of the ten, write the single hardest follow-up question, then tell me which three of my answers would not survive theirs.
8. THE TWO NON-TECHNICAL ONES. Judge my co-founder answer and my competitor answer as a founder would, not an engineer. What is missing from each?

Then rank my ten answers from strongest to weakest and tell me which two to rewrite first.

Finally: name the eleventh question - the one my own architecture invites that is not on this list - and tell me why I would struggle with it.
```

## Mission

**Objective.** A `WHAT-IF.md` with ten answers in the five-move shape, every number sourced from your own bill, limits or dashboards, plus a cold recording of you answering all ten.

**Steps.**
1. **Collect numbers** — bill, free-tier limits, load test, cost per call, users, channel shares, backup date.
2. **Write ten answers** — five moves each, roughly one hundred and eighty words, source in brackets beside every number.
3. **Walk your tables** — answer the deletion question from your own table list, not from memory.
4. **Check the backup** — if you have never restored one, record your recovery time as unknown.
5. **List single-holder access** — for the co-founder question; keep the list for M18.5.
6. **Answer the competitor question without naming your feature.**
7. **Honest unknowns** — for anything you genuinely do not know, write the dashboard, the number and the time.
8. **Record cold** — all ten, ninety seconds each, no notes.
9. **Outside follow-ups** — a student from another group asks one follow-up per answer; rewrite the three that broke.

**Upload.**
- `WHAT-IF.md` → attached file, plus the repository in `repo_url`
- The live URL → `demo_url`
- Screenshots that source your numbers: the bill, the usage page, the free-tier limits page, the load-test output
- Your recording of all ten answers, cold → link or attachment
- The ten follow-up questions you were asked, and which three answers broke
- Your rewritten versions of those three
- Your list of accounts and knowledge held by exactly one person
- Your honest-unknown answers, written out in full
- The AI interrogation conversation → share link in `ai_chat_url`
- Reflection, 150 words max → the question you answered worst and the number you could not source

**Self-check.** Tick every box before you submit.
- [ ] All ten answers follow the five moves in order
- [ ] Moves one, three and four each contain a number from my own system
- [ ] Every number has a source I can open in under thirty seconds
- [ ] No answer would fit a product other than mine
- [ ] At least eight answers name something I would deliberately not do
- [ ] My deletion answer names every table that holds a row belonging to one user
- [ ] My deletion answer states the number of days I promise, under DPDP
- [ ] My backup answer states when the last backup ran and whether I have restored one
- [ ] My key-leak answer says where the key lives and how long rotation takes
- [ ] My channel answer states the share of signups that channel carries
- [ ] My co-founder answer lists what exactly one person holds today
- [ ] My competitor answer does not mention my main feature at all
- [ ] I recorded all ten cold, with no notes, at roughly ninety seconds each
- [ ] Someone outside my group asked one follow-up on each answer
- [ ] At least one answer is an honest unknown with a dashboard, a number and a time
- [ ] This submission is my own work; my answers and numbers are mine, and I have named which parts of the system I personally directed

**Stretch.** Pick the scenario you are least prepared for and actually run it on a copy — restore your backup into a scratch database, or rotate your model key and watch what breaks. Write 100 words on the difference between your written answer and what actually happened. This is the only way move four stops being a guess.

## Stuck? Ask AI
```prompt
I keep giving what-if answers that would fit any product. Ask me one question at a time about my own system - which service hosts it, what the free tier allows, what my peak hour looks like, what my last bill said - and after each answer tell me which what-if question that number now lets me answer specifically. Do not write any answers for me. Stop when I have five numbers I can source.
```

```prompt
Teach me how to say "I do not know" in a technical defence so that it sounds like information rather than an admission. Give me the structure, then make me practise: ask me a question about my product, and when I say I do not know, tell me which part of the honest-unknown answer I left out - the dashboard, the number I would look for, or how long it would take.
```

```prompt
Explain what idempotency means to someone who has never heard the word, using a non-software example such as a canteen token being handed over twice. Then ask me, for my own payment flow, what would happen today if the gateway sent me the same webhook twice, and keep asking until I can name the exact column or constraint that stops the second one.
```

## Quiz

### Q1 · L4
Which answer to "what if ten times the traffic arrives tomorrow" is strongest?
- (a) We would scale horizontally and add a load balancer
- (b) Our free tier allows sixty database connections and my load test refused work at about forty concurrent users, so the pool goes first ✅
- (c) The server would be the bottleneck, so we would upgrade it
- (d) We have caching, so traffic is not a concern
> Only one of these says anything about this product. A what-if question asks whether you know your own limits, so the answer has to contain a number you can source.

### Q2 · L3
What does move five of the answer shape — the thing you would deliberately not do — demonstrate?
- (a) That you are cautious about making changes
- (b) That you considered more than one option and chose ✅
- (c) That the system has no remaining weaknesses
- (d) That the fix is too expensive to attempt
> It is the REJECTED line from your decision log, spoken out loud. Naming a tempting wrong move and why you refuse it is what separates a choice from a default.

### Q3 · L5
You are asked what your product costs to run at a thousand users and you genuinely do not know. What is the best thing to say?
- (a) Give a rough estimate so the answer sounds complete
- (b) Say the free tier covers it, since you have not paid anything yet
- (c) Say you do not know, name the usage page you would open, the per-call figure you would look for, and how long it would take ✅
- (d) Say cost is not a concern at this stage
> A completed honest unknown is marked as a strong answer. A confident invention collapses at one follow-up and then retrospectively discredits every other answer you gave.

### Q4 · L4
A student answers the eight technical questions well, then has nothing for "what if a competitor ships your main feature". What is that question actually about?
- (a) Whether the feature is technically difficult to copy
- (b) What you have that is not the feature — users, data, distribution, an existing workflow ✅
- (c) Whether you have filed any intellectual property
- (d) How quickly you could ship the next feature
> "We would build it better" is not an answer. The question is about defensibility, which lives in M17's positioning and distribution work rather than in your codebase.

### Q5 · L4
Why is "that will not happen" the single worst answer available?
- (a) It is too short to score any marks
- (b) It refuses the question, where naming the risk and why you accepted it is good engineering ✅
- (c) It implies the system has no monitoring
- (d) It is only acceptable for the non-technical questions
> An accepted risk, stated with its likelihood and the reason you accepted it, is a real engineering position. Denying the scenario tells a panel you have not thought about it.

## Go deeper
- [Google SRE Workbook — incident response and the questions to answer before one](https://sre.google/workbook/incident-response/)
- [Stripe docs — idempotent requests and why keys exist](https://docs.stripe.com/api/idempotent_requests)
