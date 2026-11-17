---
id: m11-03-whole-feature
module: m11-ship-your-mvp
slug: directing-a-whole-feature
title: "Directing an Agent Across a Whole Feature"
order: 3
duration_min: 15
is_free: false
mission: false
volatility: low
last_reviewed: 2026-11-16
---

## Key learnings
- Split a whole feature into steps that can each be verified and reverted alone
- Get a plan approved before any file is written, and hold the agent to it
- Set a stopping condition so the agent reports back instead of running on
- Recover from a bad state by reverting rather than patching forward
- Carry a handoff note into a fresh conversation when context decays

## Lesson

### Where this sits in the calendar

**Read this at the start of week three**, between Checkpoint 2 and Checkpoint 3. It is the only chapter in this module with no gate attached, and it is here because of what changes in week three: Checkpoint 2 asked for one path and one small change at a time, and Checkpoint 3 asks for authentication with roles, a schema with migrations, and an AI feature with tests. Those are not one brief each. They are features, and a feature is where single-brief directing stops working.

M00.8 taught the Director's Loop for one task; this is that loop at feature scale.

### The pain

A student needs staff accounts: a login, a role on each user, pages only staff can open. Eleven minutes after a decent brief, the agent reports success across nineteen files, having also swapped the session library and reformatted four unrelated components. Something is broken and they cannot tell what, because no commit holds only part of this.

### The idea

**A feature is a sequence of small changes, each verifiable and revertable on its own. You decide the sequence, approve it before any file is written, and make the agent stop at every step.**

Capability without checkpoints produces work you cannot review, revert or explain.

### The analogy

The canteen is adding paneer tikka. The chef does not announce it and leave for the day: find a supplier, cook it once and taste it, set a price, put it on the board, add a button on the till — each result checked before the next.

**Where the analogy breaks:** the chef's mouth is the test, instant and free. You have no such sense for code, so your "taste" must be written down *before* the step runs.

### Picture it

```
  FEATURE: staff accounts
  ┌──────────────────────────────────────────────────────┐
  │ 0  PLAN      agent proposes steps, I edit and approve│
  └───────────────────────┬──────────────────────────────┘
      step 1  role column on users    ──▶ check ──▶ commit
      step 2  login page, no roles    ──▶ check ──▶ commit
      step 3  server-side role check  ──▶ check ──▶ commit
      step 4  staff-only page         ──▶ check ──▶ commit
      step 5  make the first staff    ──▶ check ──▶ commit
                                            │
                        check fails? ◀──────┘
                             │
                   revert to last commit,
                   re-brief THAT step only
```

Each step ends in a check you wrote beforehand and a commit, so a failure sends you back one step rather than back to the start.

### Writing the plan

Ask for a plan before any file is written: a sequence with one property, **after every step the product still works.**

A good step is:

- **Independently verifiable** — "`/staff` while logged out sends me to the login page" fails before it, passes after
- **Independently revertable** — undoing step three does not undo steps one and two
- **Small** — under roughly four hundred changed lines, where review stops being real (M01.8)
- **Still working** — a step that leaves main broken is two steps merged

Then edit it: move the permission check earlier, and add the failure-states step no agent plan includes.

### Stopping conditions

An agent left to itself runs to the end of the plan. Say what happens at each boundary:

> Do step 1 only. Stop and report which files you changed, what the plan listed that you did not do, and how I can verify this step. Do not start step 2 until I reply.

"What you did not do" surfaces the quiet shortcut — the skipped validation, the test marked pending, the swallowed error. When a step breaks, **revert to the last commit where the check passed and re-brief that step** rather than patching forward.

### Context decay and the handoff note

Long conversations get worse, not better. Forty messages in, the agent contradicts constraints from message two: the oldest instructions compete with the newest noise (M09.12). Start a fresh conversation at a step boundary with a handoff note:

```
  HANDOFF
  DONE      steps 1-3: role column, login page, server-side check
  VERIFIED  logged-out /staff redirects; student role gets 403
  NEXT      step 4: the staff-only availability page
  CONSTRAINTS  free tier; prices in paise; do not touch the
               student menu page or the session library
  OPEN      the first-staff seed script is still manual
```

Six lines replace forty messages, and CONSTRAINTS is repeated because the new conversation knows nothing.

### Where it lives

This is the method behind every multi-step mission from here — M04.13's OAuth, M10.10's RAG pipeline, M10.19's agent-ready repository. It also makes M11.4's review tractable, and your edited plan proves the sequencing was yours.

### Words to own

| Term | Plain meaning | Example |
|---|---|---|
| Feature | A unit of value needing several changes | Staff accounts with roles |
| Step | One change that leaves the product working | Add the role column |
| Plan | The approved sequence of steps | Five steps, edited by you |
| Stopping condition | What makes the agent stop and report | "Do step 1 only, then report" |
| Checkpoint commit | A commit at a known-good state | One per verified step |
| Revert | Undo back to a known-good commit | Cheaper than patching forward |
| Context decay | Long conversations drifting from instructions | Constraints ignored by message 40 |
| Handoff note | The six lines carried to a fresh chat | DONE, VERIFIED, NEXT, CONSTRAINTS |
| Scope creep | Changes nobody asked for | A swapped session library |

### Tradeoffs and gotchas

**Do not plan a one-step task.** The pattern earns its cost when a feature touches data, permissions or money.

**Ten steps is usually two features.** Checkpoint 3's auth and AI feature are briefed separately.

**The agent's first plan puts the permission check last.** Permissions and prices belong early, so later steps inherit the check.

**Approving a plan is not approving the code.**

### Verify it

Before you approve a plan:

1. **A check per step**, failing before it and passing after. A step you cannot write one for is too vague.
2. **The permission or money check sits in the first half.**
3. **One step is for the failure states** — the agent did not add it.

After each step: run your own check, not the agent's summary; read the changed-file list before the diff, treating anything outside the step's scope as a finding; ask what it did not do.

Red flags: "all steps complete" when you asked for one; one enormous commit at day's end; a feature where you cannot name the step a decision was made in.

## Walkthrough
1. Pick a real feature from your `SCOPE.md` that genuinely needs several changes — accounts with roles, the AI feature end to end, or payments in test mode.
2. Write the feature-level brief using the M00.8 template, with CONSTRAINTS and NON-GOALS filled in properly, and an ASK that demands a plan and forbids writing any file.
3. Read the plan the agent returns and count the steps. More than eight means it is two features; send it back to split.
4. Edit the plan yourself. Split steps that touch both data and UI, move any permission or price check into the first half, delete nice-to-haves, and add an explicit step for failure states.
5. Write one verification check per step, in your own words, before any code exists. Each must fail before its step and pass after it.
6. Create one branch for the feature. Brief step one alone, with an explicit stopping condition and a request for the three-part report.
7. When it stops, run your own check for step one, read the changed-file list, and ask what it did not do. Commit only when your check passes.
8. Repeat for each step. At the first step where your check fails, revert to the last good commit and re-brief that step rather than patching forward.
9. At the halfway point, start a fresh conversation and carry only a six-line handoff note. Notice whether the quality of the replies changes.
10. When the feature is done, open one pull request per step if your host supports it, or one pull request with the commits kept separate so a reviewer can read them in order.
11. Write two decision-log entries: one plan edit you made and why, and one step you had to revert and what you changed in the re-brief.

## Talk to AI
```prompt
I am about to build a whole feature with your help, and I want a plan before any file is written. Do not write or modify any code in this reply.

CONTEXT: [paste your PRD, or its PROBLEM, USER and CONSTRAINTS sections. Then describe what already exists: the framework, the host, the database, the tables and the pages that are already live.]

FEATURE: [one sentence describing the whole feature, for example "canteen staff can log in and mark dishes sold out, and students cannot reach those pages"]

CONSTRAINTS: [free tier only; the devices and network your users have; money units; anything in the repo that must not be touched]

NON-GOALS: [name the files and areas that must not change, and forbid reformatting, renaming and new dependencies unless you ask me first]

Produce a plan of 5 to 8 steps with these properties, and tell me explicitly if you cannot meet them:
1. After every step, the product still works and can be deployed.
2. Every step is independently revertable - undoing step N does not undo step N-1.
3. Every step touches as few files as possible, and under roughly 400 changed lines.
4. Any check involving permissions, roles, prices or money appears in the first half of the plan.
5. One step is dedicated to failure states: empty data, failed save, slow network, the same action submitted twice.

For each step give me, in this order: a one-line title; the files you expect to change; a single verification check that would FAIL before this step and PASS after it, written as something I can do by hand; and the one thing most likely to go wrong in it.

Then answer three questions about the plan as a whole:
- Which step am I most likely to get wrong, and why?
- What are you assuming about my existing code that I have not told you?
- If I could only ship three of these steps this week, which three, and what would the product do at that point?

Finally, wait. Do not start step 1. I will edit this plan and brief you on one step at a time, and I expect you to stop and report after each one.
```

## Stuck? Ask AI
```prompt
I gave an agent one brief for a whole feature and it changed nineteen files at once. Something is broken and I cannot tell what. Do not try to fix it. Instead, help me get back to a state I understand: ask me what my version control history looks like, what still works, and what my last known-good commit is. Then tell me what to revert and how to split the original brief into steps I could have verified one at a time.
```

```prompt
Here is a plan an agent gave me for a feature: [paste the steps]. Act as a sceptical reviewer of the plan, not the code. Which steps leave the product broken halfway? Which two steps are really one? Where has it put a permission or price check too late? What failure states has it not planned for at all? Give me an edited plan with your reasoning for each change, and do not write any code.
```

```prompt
Explain context decay in a long AI conversation to someone who has never heard of a context window, using an everyday example such as briefing a helper over a very long shift. Then show me the shortest possible handoff note that would let a fresh conversation continue my work, and tell me which parts I must repeat every time even though I already said them.
```

## Quiz

### Q1 · L4
What is the defining property of a good step in a feature plan?
- (a) It takes the agent less than five minutes
- (b) It can be verified and reverted on its own, and leaves the product working ✅
- (c) It touches exactly one file
- (d) It was proposed by the agent rather than by you
> Independently verifiable and independently revertable is what makes a failure cost one step instead of the whole feature. One file is sometimes impossible; still-working is not negotiable.

### Q2 · L4
Your check for step four fails. What is the cheaper move?
- (a) Describe the symptom and let the agent patch it, then continue
- (b) Revert to the last commit where a check passed and re-brief step four ✅
- (c) Skip step four and continue with step five
- (d) Start the whole feature again from a fresh plan
> Patching forward stacks unreviewed changes on a state you already do not understand. Reverting costs ten minutes and keeps a history you can explain — which is why each step ends in a commit.

### Q3 · L4
Why does the brief tell the agent to stop and report after each step?
- (a) To reduce the number of tokens the feature consumes
- (b) Because agents cannot hold more than one step in context
- (c) Because otherwise it completes the whole plan and produces one unreviewable change ✅
- (d) Because version control requires one commit per conversation
> The agent is capable of doing everything at once, and that is the problem. Five reviewable changes of eighty lines beat one change across nineteen files every time.

### Q4 · L3
An agent-written plan almost always has one predictable flaw. Which?
- (a) Too few steps overall
- (b) The permission or role check placed last, and no step for failure states ✅
- (c) Steps listed in reverse order
- (d) Steps that each touch only one file
> Permissions belong early so every later step inherits the check, and no agent plan includes failure states unless asked. Both edits are yours to make, which is where your judgement pays most.

### Q5 · L5
Forty messages into a conversation, the agent starts ignoring a constraint you set at the beginning. What is the right response?
- (a) Repeat the constraint more forcefully and continue in the same conversation
- (b) Start a fresh conversation at a step boundary with a short handoff note ✅
- (c) Switch to a model with a larger context window and continue
- (d) Accept it, since the agent has more context than you do
> Long conversations crowd out the oldest instructions. A fresh conversation that knows the five things that matter beats a stale one that technically knows everything and has started ignoring it.

## Go deeper
- [Anthropic engineering — best practices for agentic coding](https://www.anthropic.com/engineering/claude-code-best-practices)
- [Martin Fowler — on the value of small, safe steps in refactoring](https://martinfowler.com/articles/workflowsOfRefactoring/)
