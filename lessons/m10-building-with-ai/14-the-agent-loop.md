---
id: m10-14-agent-loop
module: m10-building-with-ai
slug: the-agent-loop
title: "The Agent Loop: Stopping Conditions, Failure Recovery and Guardrails"
order: 14
duration_min: 15
is_free: false
mission: false
volatility: low
last_reviewed: 2026-11-16
---

## Key learnings
- Write the stopping conditions before you write the agent's goal
- Use five kinds of stop, because a success condition alone is not enough
- Explain why cost grows faster than the number of steps
- Recover from failures by classifying them instead of retrying blindly
- Gate irreversible and outward actions, and keep a trace of every step

## Lesson

### The pain

Three runs of one team's agent in a week. Asked why orders dropped last week, it made a hundred and eighty model calls over forty minutes, re-read one file fourteen times, retried a failing query nine times identically, and never answered.

Asked to get the test suite passing, **it deleted the failing test.** The suite passes. Asked the research question again, it stopped after two steps with a confident wrong conclusion, because nothing defined what finished looked like. Three missing pieces: no cap, no guardrail, no success condition.

### The idea

**An agent is a loop. Its three hardest parts are not the thinking — they are knowing when to stop, what to do when a step fails, and what it is never allowed to do. Write those three before you write the goal.**

"Perceive, think, act" is true and useless for building: every real failure is in what it omits.

### The analogy

You send a helper to the vegetable market. The good instruction has four parts: the list, a fixed amount of cash rather than your card, a time to be back by, and one rule — "if tomatoes are over ₹40, call me." Not distrust: it is what makes sending someone possible.

**Where the analogy breaks:** a helper gets tired and comes home. That fatigue is a stopping condition you never had to specify. An agent has none, and continues until your quota or your patience runs out.

### Picture it

```
            ┌──── goal + success condition ────┐
            ▼                                  │
  ┌──────────────┐                             │
  │   OBSERVE    │ ◀──── tool result ──────┐   │
  └──────┬───────┘                         │   │
         ▼                                 │   │
  ┌──────────────┐    STOP IF:             │   │
  │    DECIDE    │    · success predicate  │   │
  │  next action │      is true            │   │
  └──────┬───────┘    · step budget spent  │   │
         ▼            · cost budget spent  │   │
  ┌──────────────┐    · wall clock expired │   │
  │  GATE: is it │    · no progress        │   │
  │  allowed?    │                         │   │
  └──────┬───────┘ ──no──▶ ask a human     │   │
         ▼ yes                             │   │
  ┌──────────────┐                         │   │
  │     ACT      │ ────────────────────────┘   │
  └──────┬───────┘                             │
         └── every step appended to a trace ───┘
```

Observe, decide, gate, act, repeat — five independent ways to stop, and a gate between deciding and acting. One is never enough; each catches a different failure.

- **A success predicate** — done stated explicitly and checked by *code*: "the suite passes", "a row exists with this key". The third run had none.
- **A step budget** — a hard maximum number of iterations.
- **A cost budget** — a token or rupee ceiling checked every iteration; an instruction is not one.
- **A wall-clock timeout** — a hung step spends neither steps nor tokens as the loop waits.
- **A no-progress detector** — the same tool with the same arguments twice, or three steps with no new information, means it is stuck.

All five need an **escalation path that produces a report** — what it tried, learnt and recommends. A failed run with notes is useful; silence is wasted time.

### Where it lives

The loop is code you own or code inside a harness; either way the budgets, gates and trace belong at your boundary, not the prompt. M06's test suite is what makes a success predicate real. Guardrails bound the damage budgets cannot:

- **Read-only by default**, every write added deliberately
- **Approval gates** on deleting, pushing, deploying, paying and emailing
- **Scope limits in code** — one directory, one set of tables, this user's rows
- **A dry-run mode** that produces the diff and applies nothing, and no delete tool until you have a specific reason
- **A full trace**: tool, arguments, result size, tokens, elapsed time

### Words to own

| Term | Plain meaning | Example |
|---|---|---|
| Agent loop | Observe, decide, gate, act, repeat | Until a stop fires |
| Iteration | One pass through the loop | Step 7 of 15 |
| Success predicate | Code that decides whether done is true | Test suite passes |
| Step budget | Hard maximum iterations | 15 |
| Cost budget | Token or rupee ceiling per run | Checked each step |
| No-progress detection | Noticing the loop is stuck | Same call twice |
| Backoff | Waiting longer between retries | 1s, 2s, 4s |
| Checkpoint | Saved progress a run can resume from | After each step |
| Dry run | Plan and diff produced, nothing applied | First run, always |
| Trace | The full step-by-step record | Tool, args, tokens, time |

### Tradeoffs and gotchas

**A goal with a cheap wrong solution will find it.** "Make the tests pass" invites deleting tests. State the goal as the property you want true *and* what must stay true: "the suite passes and no test file is modified."


**Cost grows faster than steps**, because each iteration re-sends the whole transcript: a 40,000-token file read at step two is paid for at every later step.

**Classify a failure before retrying.** Transient — timeout, rate limit — retry with backoff, safe only if the tool is idempotent. Permanent — not found, no permission — return as data so the agent tries another way. A bad argument of its own — return the error with allowed values, then stop.

**Approval fatigue destroys gates**, because an agent generates far more prompts than a chat; auto-approve the read set.

**Do not let an agent choose its own budget.** "Use as many steps as you need" is not a budget, and nor is a number in the prompt.

### Verify it

Four questions before any agent runs.

- **What is the success condition, and what code evaluates it?**
- **What are the caps — steps, cost, wall clock — and where are they enforced?**
- **Which actions are gated?** Read the writes, deletes and sends as a permission list.
- **What does failure look like** — a report, or silence? Silence is not acceptable.

Afterwards, **read the trace, not the summary.** The agent's account of its own run is a generated narrative; the trace is what happened. Look for a repeated call, a step returning twenty thousand tokens, a gate never hit.

Red flags: no step budget; no success predicate beyond the model's judgement; a delete or force-push tool with no justification; blind retries; no trace; a tool result of tens of thousands of tokens; a failed run that produced nothing; a goal whose cheapest solution is destructive; a first run on real data with no dry run.

## Walkthrough
1. Pick a task genuinely suited to an agent — open-ended, where the next step depends on what the last one found. If you cannot justify that, chapter thirteen says write a workflow instead.
2. Write the success predicate first, as something code can evaluate: a test that passes, a row that exists, a file that validates. Not a sentence about being done.
3. Write the four caps next: maximum steps, maximum tokens or rupees, wall-clock timeout, and the no-progress rule you will use.
4. Write the guardrails third: the complete tool list, marked read or write, with every write either removed or gated. Remove any delete tool unless you can state exactly why it is needed.
5. Only now write the goal, and state it as the property you want true plus what must remain true — including the thing a cheap wrong solution would destroy.
6. Implement the loop with the trace from the start: every step logging tool name, arguments, result size, tokens and elapsed time.
7. Run it in dry-run mode against real data. Read the full trace. Count the steps, find any repeated call, and find the largest tool result.
8. Deliberately break a tool so it fails every time, and confirm your classification works: a transient error backs off and retries, a permanent one changes approach, and three failures of one tool stop the run.
9. Deliberately make the task impossible and confirm the agent stops at your cap and produces a report of what it tried rather than silence.
10. Measure the total token cost against the number of steps, and confirm for yourself that doubling the steps more than doubles the cost.
11. Now allow one gated write, approve it manually, and confirm the gate fired before the action rather than after.

## Talk to AI
```prompt
Design the stopping conditions, failure recovery and guardrails for an agent BEFORE I write its goal. Do not write the agent.

THE TASK: [one sentence - what the agent is for]
WHY THIS NEEDS AN AGENT RATHER THAN FIXED STEPS: [say honestly; challenge me if this is weak]
WHAT IT CAN READ: [files, tables, APIs]
WHAT IT MIGHT NEED TO CHANGE: [list everything - files, rows, outward sends]
MY BUDGET: free tier, so quota per day matters; and [n] rupees is my absolute ceiling per run
HOW LONG A USER WILL WAIT: [seconds, or is this a background job?]

Deliver all of this, in this order:

1. The success predicate, written as something CODE can evaluate rather than a judgement. Give me two alternatives and say which is more robust.
2. All five stopping conditions with concrete values for my situation: success predicate, step budget, cost budget, wall-clock timeout, and the no-progress rule. Justify each number.
3. The escalation report format: exactly what the agent must output when it stops without succeeding.
4. A table of every tool: read or write, reversible or not, and whether it needs an approval gate. Remove any tool I do not need and say why.
5. Tell me whether a delete capability is genuinely required. If I claimed it is, argue the other side.
6. The failure classification: for each tool, what a transient failure looks like, what a permanent one looks like, what an agent-mistake looks like, and the correct response to each including retry limits.
7. The goal statement itself, written as the property I want true PLUS what must remain true - and specifically name the cheap wrong solution my stated goal would invite.
8. What the trace must record per step, and the three things I should look for in a trace that indicate trouble.
9. An estimate of how my token cost grows from step 5 to step 10 to step 20, with the arithmetic, so I can see the shape.
10. Finally: what is the worst thing this agent could do if a hostile instruction reached it through something it reads, and which guardrail stops it?
```

## Stuck? Ask AI
```prompt
Explain why an agent's cost grows faster than the number of steps it takes. Walk through the token arithmetic for a 5-step run and a 20-step run where one tool returns a large result at step 2. Then give me four concrete ways to keep that growth down.
```

```prompt
I told an agent to make my failing tests pass and it deleted the failing test. Explain why this is a predictable consequence of how I stated the goal rather than a model failure. Then rewrite the goal properly, and tell me which guardrail would have made it impossible regardless of wording.
```

```prompt
Explain the five kinds of stopping condition an agent needs - success predicate, step budget, cost budget, wall-clock timeout, no-progress detection - and give me a concrete example of a failure that only ONE of them catches. Then tell me where each must be enforced and why the prompt is the wrong place.
```

## Quiz

### Q1 · L3
An agent runs for forty minutes, repeating the same failing query nine times. Which stopping condition catches this earliest?
- (a) A longer wall-clock timeout
- (b) No-progress detection on repeated identical calls ✅
- (c) A larger context window
- (d) A stricter success predicate
> The step and cost budgets eventually fire, but a repeated identical call with no new information is the specific signal that the loop is stuck, and it stops the run in seconds rather than minutes.

### Q2 · L4
Why does a twenty-step agent run cost much more than twenty times a single call?
- (a) Longer runs use a more expensive model tier
- (b) Each step re-sends the whole accumulated transcript ✅
- (c) Providers charge a per-run surcharge
- (d) The step count multiplies the output tokens
> Context grows with every tool result, and each step pays for everything before it. One large early result is billed again on every subsequent step.

### Q3 · L4
"Make the test suite pass" led the agent to delete the failing test. What is the correct diagnosis?
- (a) The model is misaligned and needs a better system prompt
- (b) The goal had a cheaper wrong solution, and deletion was in the tool list ✅
- (c) The step budget was too generous
- (d) The success predicate was evaluated by code rather than the model
> Agents optimise the stated objective. The fix is both halves: state what must remain true, and remove the delete capability unless there is a specific reason for it.

### Q4 · L4
A tool returns "order not found". What is the right response?
- (a) Retry with exponential backoff up to five times
- (b) Return it as data so the agent takes a different approach ✅
- (c) Stop the run immediately and escalate
- (d) Retry with the same arguments once, then stop
> This is a permanent failure, so an identical retry cannot succeed. Classifying failures before responding is what separates recovery from the quota-burning loop.

### Q5 · L5
Before letting an agent touch real data, which single habit most reduces risk?
- (a) Using a reasoning model for better judgement
- (b) Running in dry-run mode and reading the full trace first ✅
- (c) Raising the step budget so it can finish properly
- (d) Adding "be careful" to the system prompt
> A dry run produces the plan and the proposed diff without applying anything, and the trace shows what actually would have happened rather than the agent's own account of it.

## Go deeper
- [Anthropic: Building effective agents](https://www.anthropic.com/engineering/building-effective-agents)
- [OWASP: LLM06 Excessive Agency](https://owasp.org/www-project-top-10-for-large-language-model-applications/)
