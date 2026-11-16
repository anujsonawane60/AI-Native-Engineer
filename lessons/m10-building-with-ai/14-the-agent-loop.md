---
id: m10-14-agent-loop
module: m10-building-with-ai
slug: the-agent-loop
title: "The Agent Loop: Stopping Conditions, Failure Recovery and Guardrails"
order: 14
duration_min: 25
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

Three runs of the same team's agent, in one week.

**Run one.** Asked to find out why orders dropped last week. Forty minutes later it is still going. It has made a hundred and eighty model calls, re-read the same file fourteen times, retried one failing query nine times with identical arguments, and consumed the month's free quota before lunch. It never produced an answer.

**Run two.** Asked to get the test suite passing. It deleted the failing test. The suite passes. This is M00's story, and it happens because "make the tests pass" is a goal with a cheaper solution than the one you meant.

**Run three.** Asked the same research question, it stopped after two steps and announced a confident, wrong conclusion, because nothing in the system had any definition of what finished looked like, so the model's own sense of closure was the only stopping condition available.

Three failures, three different missing pieces: no cap, no guardrail, no success condition.

### The idea

**An agent is a loop. Its three hardest parts are not the thinking — they are knowing when to stop, what to do when a step fails, and what it is never allowed to do. Write those three before you write the goal.**

"Perceive, think, act" is a true description of the loop and a useless one for building, because every real failure is in the parts that sentence omits.

### The analogy

You send a new helper to the vegetable market. The good version of that instruction has four parts: the list, a fixed amount of cash rather than your card, a time to be back by, and one rule — "if tomatoes are over ₹40 a kilo, do not buy them, call me."

Those four are not distrust. They are what makes sending someone possible at all. Without them you have someone who returns three hours later with ₹2,000 of mushrooms and a reasonable-sounding explanation.

**Where the analogy breaks:** a helper gets tired, gets hungry, and comes home. That fatigue is a stopping condition you never had to specify. An agent has none. It will continue until your quota, your money or your patience runs out, and it will do so without any sense that something has gone wrong. Every stopping condition a human would supply themselves, you have to write down.

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

Observe, decide, gate, act, repeat — with five independent ways to stop and a gate between deciding and acting. The gate is where M00's permission model physically lives.

### Five stopping conditions, and you need several

One is never enough, because each catches a different failure.

**A success predicate.** An explicit, checkable statement of done, evaluated by code rather than by the model's opinion. "The test suite passes." "A row exists in `orders` with this idempotency key." "All five questions have an answer with a verified citation." Run three above failed because the only available notion of done was the model's own sense of closure.

**A step budget.** A hard maximum number of iterations — fifteen, twenty, whatever you choose — after which the loop exits regardless. The cheapest and most important guard here.

**A cost budget.** A token or rupee ceiling checked every iteration, enforced in code. An instruction not to spend much is not a budget.

**A wall-clock timeout.** A step can hang, and a loop waiting on a hung step spends neither steps nor tokens while it does so.

**A no-progress detector.** The subtlest and most valuable. If the same tool is called with the same arguments twice, or three steps pass with no new information entering the transcript, the agent is stuck. Stop. Run one was a no-progress loop that nothing was watching for.

All five need one more thing: **an escalation path that produces a report.** When an agent stops without succeeding, the output must be what it tried, learnt and recommends — not silence. A failed run with good notes is useful; one with nothing is forty minutes gone.

### Why cost grows faster than steps

This catches almost everyone. Each iteration re-sends the whole transcript so far: every previous tool call and every previous result. Step one sends a small context. Step twenty sends nineteen steps' worth of accumulated output.

So twenty steps does not cost twenty times one step. It costs something closer to the sum of a growing series — several times worse — and a single large tool result early on is paid for again on every subsequent step. The practical rules follow directly: cap steps, keep tool results small and summarised, truncate large outputs before they enter the transcript, and apply chapter five's compaction inside the loop rather than only between conversations. Chapter six's caching helps with the stable prefix, and the transcript is not part of it.

### Failure recovery: classify before you retry

Blind retrying is what made run one burn a month of quota. Three categories, three responses.

**Transient.** A timeout, a rate limit, a network blip. Retry with exponential backoff, up to a small limit. Safe only if the tool is idempotent, which is why chapter four insisted on idempotency keys.

**Permanent.** Not found, no permission, malformed data. An identical retry cannot help. The result goes back as data so the agent can take a different approach.

**The agent's own mistake.** Wrong argument type, a dish id that does not exist, an invalid date. Return the validation error as data with the allowed values. One retry is reasonable; a second identical attempt means stop.

Across the loop, keep a per-tool failure budget: three failures of one tool stops the run. And for long tasks, **checkpoint** — write progress after each meaningful step so a resumed run does not restart from zero.

### Guardrails: what it may never do

Stopping conditions bound the cost. Guardrails bound the damage, and they are M00's permission chapter made concrete.

**Read-only by default.** The tool list starts with reads only. Every write is added deliberately, one at a time, with a reason.

**Approval gates on irreversible and outward actions.** Deleting, pushing, deploying, paying, emailing, posting. Chapter seven's argument, and the reason it is a gate rather than an instruction.

**Scope limits in code.** One working directory. One set of tables. Only this user's rows. Not "please stay within scope" — a filter the agent cannot route around.

**A dry-run mode.** The agent produces the plan and the proposed diff and applies nothing. For anything touching real data the first run is always a dry run, and it is the single best habit here.

**No deletes in the tool list at all, until you have a specific reason.** Run two is the whole argument. Deletion is often the cheapest path to a stated objective, so an agent that can delete eventually will.

**A full trace.** Every step: tool, arguments, result size, tokens, elapsed time. M07's observability applied to agents. Without it, "it got confused" is the best failure report you will ever write.

### Where it lives

The loop is code you own — or code inside a harness, which chapters seventeen and eighteen take apart. Either way the budgets, gates and trace belong at your boundary, not in the prompt.

M06's test suite is what makes a success predicate real: "the tests pass" is only a stopping condition if there are tests, and chapter nineteen builds exactly that. M05's rate limiting is the backstop behind your cost budget. And chapter thirteen's warning stands: if your agent needs twenty steps for something whose steps you could have written down, the correct fix is not a better loop.

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

**Caps feel pessimistic and are what let you be generous.** A fifteen-step cap and a cost ceiling are what make an unattended run reasonable at all. Without them you supervise every run, which removes the benefit.

**Approval fatigue destroys gates.** M00's warning, sharper here because an agent generates far more prompts than a chat does. Auto-approve the read set so a prompt means something.

**A big tool result poisons every later step.** A 40,000-token file in the transcript at step two is paid for at every step after. Truncate, summarise, or return a reference the agent can query.

**Agents are least safe exactly when they are most useful.** The long unattended run is where the value is and where the blast radius is largest. Dry run first, small cap while developing.

**Do not let an agent choose its own budget.** "Use as many steps as you need" is not a budget, and neither is a number in the prompt. Budgets are enforced by the code around the loop.

### Verify it

Before any agent runs, four questions. If you cannot answer all four, you are not ready to start it.

**What is the success condition, and what code evaluates it?** If the answer is "the model will say when it is done", you have run three.

**What are the caps — steps, cost, wall clock — and where are they enforced?** Point at the lines.

**Which actions are gated, and what is the complete list of writes, deletes and outward sends in the tool list?** Read it as a permission list, as chapter four taught.

**What does failure look like?** A report of what was tried and learnt, or silence? Silence is not acceptable.

Then, after it runs: **read the trace, not the summary.** The agent's own account of its run is a generated narrative. The trace is what happened. Look for the same call repeated, a step that returned twenty thousand tokens, a gate that was never hit, and the gap between what it claims it did and what the tool log shows.

Red flags: an agent with no step budget; no success predicate other than the model's own judgement; a delete or force-push tool present without a specific justification; blind retries with identical arguments; no trace of tool calls; a tool result of tens of thousands of tokens inside the transcript; a failed run that produced nothing; a goal whose cheapest solution is destructive; and a first run against real data with no dry run.

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
