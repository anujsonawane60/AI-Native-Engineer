---
id: m11-02-checkpoint-2
module: m11-ship-your-mvp
slug: checkpoint-2-walking-skeleton
title: "Checkpoint 2 · Walking Skeleton, Deployed"
order: 2
duration_min: 35
is_free: false
mission: true
volatility: low
last_reviewed: 2026-11-16
---

## Key learnings
- Pass the week-two gate: one complete user path, live on a public URL
- Prove the data is really saved rather than held in the page
- Test your product on a device and a network that are not yours
- Tell a deployed skeleton from a demo that only runs on your laptop
- Show that all three of you can ship through git, not just one of you

## Lesson

### Where this sits in the calendar

**Checkpoint 2 is due at the end of week two of the Sprint.** Checkpoint 1 was last week: problem, PRD, scope, diagram, team, decision log. This week the gate is the first thing a human being can use.

The work itself is built by missions elsewhere. M01.6–M01.8 gave you git, branches and pull requests. M02 gave you the page. M04.6–M04.7 gave you tables and a schema. M06.12 gave you CI and preview deploys. This chapter does not re-teach any of that. It states what must be live by a date, and how to check it is genuinely live rather than merely demonstrable.

### The pain

A group demos on day fourteen. One of them turns their laptop around and walks through the product: the menu loads, they tap an item, they type a name, a confirmation appears. It looks finished.

Then someone asks to try it on their own phone. There is no URL. Then someone asks what happens if you reload the page. The order is gone — it was never saved anywhere, it was held in the page's memory. Then the laptop's owner goes home for the weekend and the other two members cannot work on the product at all, because the only copy of the database is on that machine and the environment variables that make it run are in a file that was never shared.

Nothing in that demo was a lie. It was a genuine screen showing genuine code. It was also worth almost nothing, because the product does not exist anywhere except on one machine, in one state, for one person.

### The idea

**A walking skeleton is a thin complete path through every layer, deployed at a public URL, with the data really stored.**

All the bones connected, no muscle yet. Ugly is fine. Narrow is fine — one action, not six. Fake is not fine, and local-only is not fine, because both of those are the exact two things that make a week-two product worthless in week three.

### The analogy

The canteen stall's first real sale. Not a tasting in the kitchen for the two people who cooked it: one actual student walks up to a counter that exists, reads a price written where customers can see it, hands over ₹20, gets a plate of poha, and the ₹20 goes into a cash box that the owner can count at the end of the day.

Almost everything is missing. One dish, no second burner, a price list on paper, no change for a ₹500 note. It does not matter. The loop is closed: a stranger paid, got the thing, and the record survives.

**Where the analogy breaks:** a stall built badly can be rebuilt over a weekend. Software accumulates, so the tape holding your skeleton together has to be *replaceable* tape. That is exactly why the skeleton still goes through git, still deploys properly, and still writes to a real database — those three are the parts you cannot retrofit cheaply.

### Picture it

```
  WHAT A DEMO LOOKS LIKE          WHAT A SKELETON LOOKS LIKE
  ┌──────────────────────┐        ┌────────────────────────────┐
  │ one laptop           │        │ any phone, any network     │
  │   page state in RAM  │        │   public https URL         │
  │   sample data in code│        │   real rows in a database  │
  │   secrets in a local │        │   secrets in the host's    │
  │   file nobody shared │        │   environment settings     │
  │   one person can run │        │   all three can deploy     │
  └──────────────────────┘        └────────────────────────────┘
       reload = gone                  reload = still there
```

The two columns are the same feature; the left one survives only as long as the browser tab and the laptop, and the right one survives a reload, a different phone and a different person.

### What must exist by the end of week two

**1. One complete user path, live.** A public HTTPS URL, reachable from a phone that has never been on your wifi. The single most important action from `SCOPE.md`, start to finish, ending in something the user can see.

**2. Data really saved.** A row in a real database, visible in the host's table view. If reloading the page loses the thing the user just did, you do not have a skeleton.

**3. The path works for a stranger.** Hand your phone to someone outside the group with no explanation and watch them. You are not testing them, you are testing whether the path exists without a narrator.

**4. One handled failure.** Exactly one, chosen deliberately: an empty list, a failed save, or a submit with the network off. Agents build the happy path by default, and one real failure state is what separates a skeleton from a screenshot.

**5. Everything through git.** Each of the three of you has at least one merged pull request. Main is protected if you managed the stretch in M01.8. No secrets in any commit, ever — M07.2 explains why that is permanent.

**6. One automated check.** One test that genuinely fails when you break the thing it tests, running on every push. One is enough this week. From M06.10 and M06.12.

**7. Two more decision-log entries.** Dated this week. Why this slice and not another, or what you cut when you ran out of days.

### Where it lives

The skeleton is the spine that every later module hangs flesh on. M03 restyles the page you deployed this week. M04.12 adds real accounts to the name-in-a-text-box you used instead of login. M05.3 caches the query you wrote. M10 adds the AI feature to the server you already have. M07.7 points an uptime monitor at the URL you already own.

That ordering is not an accident. Each of those is an increment on something live, and an increment is a brief you can verify. "Add login to the deployed app" is checkable in ten minutes; "build the whole product" is not checkable at all.

**Each of you submits your own evidence**, as in Checkpoint 1. One shared URL, three posts. Your reflection names the pull request you personally opened and what the reviewer found in it.

### Words to own

| Term | Plain meaning | Example |
|---|---|---|
| Walking skeleton | Thin complete path through every layer | See menu, order, saved, confirmed |
| Deployed | Running on a machine that is not yours | A public HTTPS URL |
| localhost | Your own machine, reachable by nobody | `http://localhost:3000` |
| Environment variable | A secret or setting supplied outside the code | The database URL |
| Persisted | Written somewhere that survives a reload | A row in a table |
| Preview deploy | A temporary URL for one pull request | How a reviewer tries your change |
| Failure state | What the user sees when it goes wrong | "Could not save — try again" |
| Smoke test | One check that the main path still works | Runs on every push |

### Tradeoffs and gotchas

**Ugly but real beats beautiful but fake.** A styled prototype with sample data teaches you about layout. A plain page writing real rows teaches you about people. You need the second this week; M03 gives you the first next week.

**Deploy on day one of the week, not day seven.** Deployment is where the surprises live: a missing environment variable, a build that works locally and fails on Linux because a filename's capitalisation differs, a free-tier database that sleeps. Hitting those with fifty lines of code is a bad afternoon. Hitting them in week five is a bad week.

**Secrets never go in the repository.** Not in a config file, not "temporarily", not in a comment. A commit is permanent the moment it merges, and a key in git history is a key you must rotate. Use the host's environment settings, as in M07.2.

**One slice, not four.** The temptation at this gate is to show more. Three half-features look better in a demo and are worse at every subsequent checkpoint, because none of them has a verified failure state and none can be built on.

**Check the free tier's limits before you depend on them.** Row caps, cold starts, sleeping databases, monthly request budgets, a region far from India adding latency on 4G. M06.7 covers the billing traps; the ones that bite in week two are usually the sleeping database and the cold start.

**If only one of you can deploy, you have one engineer and two passengers.** All three must have run a deploy at least once. This is the cheapest possible insurance against the member with the credentials falling ill in week four, and it is also what makes three individual defences possible in M18.

### Verify it

Six checks. Run them in this order, because the first two find most of the problems.

1. **Open the URL on a phone that has never been on your wifi, with wifi off, on mobile data.** This single test catches localhost-only builds, missing environment variables, mixed-content errors and anything that depended on your machine. If it fails here, nothing else matters yet.
2. **Do the action, then reload the page.** Then close the browser and open the URL again on a different device. If what you did is gone, your data is not persisted.
3. **Look at the row in the database yourself.** Not the confirmation message — the actual table. Agents will happily show a success toast for a request that errored.
4. **Hand it to a stranger with no explanation.** Say nothing, including when they go the wrong way. Where they hesitate is where next week's work is.
5. **Break your one test on purpose.** Change the thing it checks, push, and confirm the check goes red. A test that cannot fail is proving nothing, and this is the only way to know the difference.
6. **Throttle the network and try the failure state.** Turn on airplane mode mid-submit. The product must say something true rather than hanging forever or claiming success.

The red flags at this gate: **a demo that only works on the author's machine**, a product with no failure state at all, data that disappears on reload, a single member holding every credential, and a test that passes whether or not the feature exists. Each one is cheap to fix in week two and expensive to discover in the final week.

## Walkthrough
1. Open `SCOPE.md` and read slice one out loud as a sentence a person performs. If it still sounds like a list of components, rewrite it before you brief anything.
2. Deploy an almost-empty page to a public URL today, before the feature exists. Get the URL working first; you want the deployment surprises now, not on day seven.
3. Create the one or two tables slice one needs. Keep it minimal — the schema work proper belongs to M04.7, not to this week.
4. Put every secret in the host's environment settings, never in a file you commit. Confirm by searching your repository for the key's first six characters.
5. Write one tightly scoped brief for the first half of the path — the screen and the submit — with a NON-GOALS line forbidding styling work, extra features and new dependencies.
6. Direct the agent, review the diff against the six questions from M01.8, open a pull request, have a teammate review it, and merge. Repeat for the second half of the path — the save and the confirmation.
7. Add one failure state deliberately. Pick the empty list or the failed save, and brief it as its own small change.
8. Add one automated check that exercises the whole path, then break the feature on purpose and confirm the check goes red before you fix it again.
9. Run all six verification checks, including the stranger test and the mobile-data test. Write down each result as pass or fail.
10. Record a sixty-second screen capture of the path on a phone, from opening the URL to seeing the saved result, and a screenshot of the matching database row.
11. Make sure all three of you have merged at least one pull request this week and have each run a deploy once. Add two dated entries to `DECISIONS.md`.

## Talk to AI
```prompt
I am two weeks into a startup sprint and I have to pass a checkpoint called "walking skeleton, deployed". I want you to interrogate what I have, not build anything. Do not write code in this reply.

My slice-one scope, as a user path: [paste from SCOPE.md]
My public URL: [paste it]
My stack and hosting: [name the framework, the host, the database, all free tier]
The tables slice one writes to: [paste the table and column names]
The one failure state I handled: [describe it]
My one automated check: [describe what it asserts]

Work through these in order and be blunt:

1. COMPLETENESS. Given my path, name every step where the browser, my server, and my database are each involved. If any step of my path never reaches the database, tell me - that means nothing is being saved there.
2. PERSISTENCE. Based on what I described, what exactly would survive a page reload, and what would be lost? Name anything that is being held in the page rather than stored.
3. LOCAL-ONLY RISKS. List the specific things that work on a developer's own machine and fail on a public URL, for my stack - environment variables, file paths, case-sensitive filenames, database connection limits, cold starts. For each, tell me how I would notice.
4. FAILURE STATES. Given my path, list every state I have probably not handled: empty data, slow network, failed save, the same action submitted twice, a very long input. Rank them by how likely a real user is to hit one this week.
5. MY TEST. Based on what I told you it asserts, could that check still pass if the feature were removed entirely? If yes, say so and tell me what it should assert instead.
6. FREE TIER. For the services I named, what limits or behaviours will surprise me first - sleeping databases, cold starts, row caps, a region far from India? Tell me how each would look to a user on slow 4G.

Then give me a numbered list of the five checks I should run myself, in the order that finds problems fastest.

Finally: name the single most likely reason my product works for me and fails for a stranger.
```

## Mission

**Objective.** One complete user path from your `SCOPE.md`, live on a public URL, writing real rows, with one handled failure state, one real automated check, and a merged pull request from each of the three of you.

**Steps.**
1. **Deploy empty first** — get a public URL working before the feature exists.
2. **Schema** — create only the tables slice one needs.
3. **Secrets** — every key in the host's environment settings; confirm nothing is in git.
4. **Build in two halves** — screen and submit, then save and confirm. One scoped brief and one pull request each, reviewed by a teammate.
5. **One failure** — add a single deliberate failure state as its own small change.
6. **One check** — add an automated check, then break the feature and confirm it goes red.
7. **Verify** — run all six checks from the lesson, including mobile data and the stranger test. Record pass or fail for each.
8. **Evidence** — a sixty-second phone capture of the path, plus a screenshot of the database row it created.
9. **Spread the keys** — all three of you deploy once. Add two dated decision-log entries.

**Upload.**
- The live URL → `demo_url`
- The repository → `repo_url`
- A sixty-second screen capture of the full path, recorded on a phone on mobile data
- A screenshot of the actual database row created by that capture
- A screenshot of your automated check going red when you broke the feature, and green after
- A screenshot of your handled failure state as the user sees it
- Your six verification checks with pass or fail recorded per item
- The pull request you personally opened and one comment a teammate left on it
- Reflection, 150 words max → what broke when you deployed that never broke locally, and what the stranger did that you did not expect

**Self-check.** Tick every box before you submit.
- [ ] The URL opens on a phone that has never been on our wifi, over mobile data
- [ ] The path can be completed start to finish without anyone explaining it
- [ ] What the user did is still there after a reload, on a different device
- [ ] I looked at the actual database row, not just the confirmation message
- [ ] Exactly one failure state is handled, and I triggered it on purpose
- [ ] No secret, key or database URL appears anywhere in git history
- [ ] All three of us have at least one merged pull request this week
- [ ] All three of us have run a deploy at least once
- [ ] One automated check runs on every push
- [ ] I broke the feature and watched that check go red before fixing it
- [ ] Nothing in the path uses sample or hard-coded data
- [ ] `DECISIONS.md` has two new entries, dated this week
- [ ] This submission is mine: my own pull request, my own reflection

**Stretch.** Open your URL on the cheapest Android phone anyone in your group can borrow, on 4G, and time the path from tap to confirmation. Write 100 words on what was slow and what you would cut to halve it. Keep the number — M05 and M16 will both ask you for it again.

## Stuck? Ask AI
```prompt
My app works perfectly when I run it on my own machine and fails after I deploy it. Ask me diagnostic questions one at a time, starting with the cheapest to answer, to narrow down which layer is failing: the build, the environment variables, the database connection, or the browser. After each answer, tell me what you have ruled out. Do not guess at a fix until you have ruled out at least two layers.
```

```prompt
I have an automated check and I am not sure it is testing anything real. Here is what it asserts: [paste or describe it]. Tell me whether this check could still pass if the feature were deleted, and show me how I would prove that to myself by breaking something on purpose. Then rewrite the assertion so it cannot pass without the feature.
```

```prompt
Explain the difference between data held in a web page and data stored in a database, using a non-software example such as a canteen writing orders on their hand versus in a notebook. Then give me four symptoms a user would notice if my app only had the first kind, in the order they would notice them.
```

## Quiz

### Q1 · L4
A group demos a working product from a laptop on day fourteen. There is no public URL and reloading the page loses the order. What is the most accurate description of what they have?
- (a) A walking skeleton that needs deploying
- (b) A demo, because neither the deployment nor the storage exists ✅
- (c) A skeleton with one unhandled failure state
- (d) A complete slice that is not yet styled
> A skeleton is defined by the deployment and the persistence, not by the screen. Those are the two parts you cannot retrofit cheaply, which is why this gate checks them in week two.

### Q2 · L4
Which single verification check finds the most problems at this gate?
- (a) Reading the diff for the whole week's work
- (b) Opening the URL on a phone that has never been on your wifi, on mobile data ✅
- (c) Counting how many pull requests each member merged
- (d) Checking that the page looks right on a laptop screen
> One test catches localhost-only builds, missing environment variables, mixed-content failures and anything that depended on your machine. If it fails there, nothing else is worth checking yet.

### Q3 · L3
Why does this gate ask for exactly one handled failure state rather than several?
- (a) Handling more than one failure is beyond a beginner
- (b) Agents build the happy path by default, so one real failure state proves you went past it ✅
- (c) The platform only accepts one screenshot
- (d) Extra failure states would slow the page down
> One is a verifiable minimum this week, not a target. The point is to prove the path has a truthful behaviour when something goes wrong, instead of a toast that always says success.

### Q4 · L4
You added a test. How do you know it is worth having?
- (a) It passes on every push
- (b) It runs in under a second
- (c) It goes red when you deliberately break the thing it tests ✅
- (d) The agent wrote it alongside the feature
> A test that cannot fail proves nothing. Breaking the feature on purpose and watching the check go red is the only way to tell a real check from a decorative one.

### Q5 · L5
Your group is two days from Checkpoint 2 with four half-built features and no public URL. What is the honest move?
- (a) Extend the checkpoint by a few days to finish all four
- (b) Demo from a laptop and deploy next week
- (c) Cut to one path, deploy it, and handle one failure ✅
- (d) Keep all four and skip the automated check
> Scope is the only lever that does not lie. Three half-features cannot be built on at the next gate, because none has a verified failure state and none is live for anyone outside the room.

## Go deeper
- [The Twelve-Factor App — config, backing services and dev/prod parity](https://12factor.net/)
- [Alistair Cockburn — Walking Skeleton](https://alistair.cockburn.us/walking-skeleton/)
