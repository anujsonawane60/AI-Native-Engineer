---
id: m10-15-agent-patterns
module: m10-building-with-ai
slug: agent-patterns
title: "Agent Patterns and Multi-Agent Orchestration"
order: 15
duration_min: 25
is_free: false
mission: true
volatility: low
last_reviewed: 2026-11-16
---

## Key learnings
- Name the single-agent patterns and reach for them before multi-agent
- Say exactly what multi-agent buys: parallelism, context isolation, scoped tools
- Give every sub-agent a written contract: input, output schema, tools, budget
- Make a critic useful by giving it external criteria rather than taste
- Prove a multi-agent design beats the single-agent baseline, or delete it

## Lesson

### The pain

A team builds a weekly competitor report with seven agents: a planner, a researcher, a writer, a critic, a fact-checker, a formatter and a quality-assurance agent over the whole thing.

It costs eleven times a single call and takes forty seconds. The critic's verdict is "looks good" every time, including on the output with a fabricated statistic. The fact-checker and the researcher both fetch the same three pages and bill for both. And when the report carries the wrong number, nobody can say which agent introduced it, because each has its own trace. Two days later one well-briefed agent with four tools and a real checklist produces a better report in nine seconds. They had built an organisation chart, not a system.

### The idea

**Agent patterns are a small set of named shapes, and multi-agent is the most expensive of them. It buys exactly three things — parallelism, context isolation and per-role tool scoping — and charges you a new class of failure: coordination.**

If you cannot name which of the three you are buying, you are buying nothing. **A multi-agent system where one agent would do is a red flag, not a sign of sophistication.**

| What multi-agent buys | What it charges |
|---|---|
| Parallelism — independent subtasks at once | Coordination failures: duplicated, dropped or contradictory work |
| Context isolation — each worker starts small and clean | Multiplied cost: every agent sends its own context |
| Per-role tool scoping — the researcher reads, one writer writes | Lost attribution: seven traces, no correlation id |

### The analogy

A wedding function at the canteen: three hundred plates, six dishes. One cook cannot do it, so there is someone on the tandoor, someone on gravies, someone on rice, and the manager who sets timings and plates the final thali. It works because the stations are genuinely independent and one person owns the plate.

**Where the analogy breaks:** real cooks overhear everything, so somebody notices that the rice is sized for three hundred and the gravy for fifty. Agents never overhear. Every shared fact — the guest count, the deadline, what another worker found — must be passed explicitly in writing, or it does not exist.

### Picture it

```
                  ORCHESTRATOR
        decomposes · assigns · synthesises
        holds the only write capability
                      │
      ┌───────────┬───┴───┬───────────┐
      ▼           ▼       ▼           ▼
  ┌───────┐  ┌───────┐ ┌───────┐  ┌───────┐
  │WORKER │  │WORKER │ │WORKER │  │WORKER │
  │  A    │  │  B    │ │  C    │  │  D    │
  └───┬───┘  └───┬───┘ └───┬───┘  └───┬───┘
      │          │         │          │
   read-only tools · own small context ·
   own step budget · returns a fixed schema
      │          │         │          │
      └──────────┴────┬────┴──────────┘
                      ▼
            orchestrator merges,
            resolves conflicts by a
            written rule, then writes once
```

Workers run in parallel, each isolated and read-only, each returning a schema. Only the orchestrator writes, and conflicts are resolved by a rule you wrote rather than whoever answered last.

Most teams reach for this shape without trying the single-agent patterns, which cost nothing and prevent most of their worries.

- **Plan then execute.** The agent writes a numbered plan and stops; you read it, change it, approve it. A bad plan then costs thirty seconds of reading instead of twenty wasted steps. M00's Director's Loop made mechanical, and the default for anything consequential.
- **Reflection with a critic** — generate, critique, revise, but only when the critique has *external* criteria. "Does the suite pass?" is a real critic; "is this good?" is the broken critic in the opening story.
- **Evaluator-optimiser.** Reflection with a *number*: score against chapter twelve's eval set, keep the better version, repeat to a cap.
- **Router.** Chapter thirteen's rung three, the most underused shape in this module.

### Where it lives

Patterns are shapes you impose around chapter fourteen's loop, or configure inside a harness — chapter eighteen's sub-agents are this idea as a product feature. M05's queues are how fan-out runs; M07's tracing is what makes attribution possible, given one correlation id per run.

A sub-agent is not a personality. It is a function, and it needs a contract written before it exists:

- **Input** — exactly what it receives; anything extra is a distractor and a token cost
- **Output** — a schema from chapter three, never prose
- **Tools** — the minimum list, read-only unless there is a reason
- **Budget** — its own step cap and timeout, or one stuck worker spends the whole run
- **Failure** — what it returns when it cannot do the job, since a silent failure drops a subtask

Two rules on top. **Single writer:** one component commits results, because two agents writing one row is a race condition with a language model in it. And **independence:** if worker B needs worker A's output, they are two steps of a workflow.

### Words to own

| Term | Plain meaning | Example |
|---|---|---|
| Orchestrator | The agent that decomposes and merges | Holds the only write tool |
| Worker / sub-agent | A scoped agent doing one subtask | Summarise one competitor |
| Fan-out / fan-in | Splitting work, then merging results | 5 pages at once |
| Contract | Input, output schema, tools, budget | Written before building |
| Context isolation | Each worker starting clean and small | No shared transcript |
| Reflection | Generate, critique, revise | Needs external criteria |
| Critic | The component that judges output | Best when it runs tests |
| Single writer | Only one component commits results | No write races |
| Coordination failure | Duplicated, dropped or conflicting work | Rice for 200, gravy for 50 |
| Correlation id | One id threaded through every trace | Makes attribution possible |

### Tradeoffs and gotchas

**A critic that always approves is worse than none.** It costs a call and manufactures the appearance of review. Give it a checklist, a schema or a test suite; if it cannot fail your output, it is theatre.

**Named agents create an illusion of competence.** "QAAgent" sounds like quality assurance; it is one prompt. Name components by what they receive and return instead.

**Sequential multi-agent is a workflow.** If the shape is A then B then C with fixed handoffs, write it as code.

**The orchestrator's context grows worst of all**, because it reads every worker's output. Tight schemas rather than essays.

**Measure against the single-agent baseline or do not ship it.** Debate patterns rarely pay: two agents arguing produce longer, more confident output and little more accuracy, at double the cost.

### Verify it

- **Name what you are buying before you build** — parallelism, context isolation, or tool scoping. If the honest answer is "it seems more capable", use plan-then-execute.
- **Run the baseline first and record it:** one agent, good tools, plan-then-execute, with score, cost and latency on your eval set. Every later design must beat that row.
- **Try to attribute one real failure.** Find the agent behind a wrong output; if you cannot in five minutes, add the correlation id first.
- **Check whether your critic can fail you.** Feed it a fabricated number, an invalid schema, a failing test.
- **Look for duplicated and dropped work in the traces** — the same fetch twice, or a subtask missing from the output.
- **Confirm exactly one writer.** Grep the tool lists.

Red flags: a critic that has never rejected anything; agents named for job titles rather than inputs and outputs; workers returning prose instead of a schema; no per-worker step budget; more than one component able to write; no correlation id; a sequential handoff chain called multi-agent; no single-agent baseline.

## Walkthrough
1. Pick one genuinely open-ended task in your product — something with independent parts, such as summarising several sources or checking several rules.
2. Build the single-agent baseline first: one agent, the minimum tools, a fifteen-step cap, and the plan-then-execute pattern with you approving the plan.
3. Run your eval set against it and record three numbers: score, total tokens and wall-clock latency. This row is what any further complexity must beat.
4. Write down which of the three benefits a multi-agent version would buy you — parallelism, context isolation, tool scoping — and how you will measure that specific benefit.
5. Write the contract for each worker before building any: exact input, output schema, tool list, step budget, cost cap, and what it returns on failure.
6. Check independence honestly. If any worker needs another's output, redraw it as a workflow step instead and note that you did.
7. Build the orchestrator with the only write capability, and a written conflict rule for when two workers disagree — newest wins, or highest-confidence wins, or escalate.
8. Thread one correlation id through every agent's trace so all steps from one run can be read as a single sequence.
9. Run the same eval set and record the same three numbers. Compare them with your baseline row and be honest about what got worse.
10. Test the critic by feeding it output you know is wrong. If it approves, replace its criteria with something code can check and test it again.
11. Read the merged trace for duplicated fetches and for any subtask that was assigned and never appeared in the output.

## Talk to AI
```prompt
Help me decide whether my task needs multiple agents, and design the contracts if it does. Push back hard if one agent would do.

MY TASK: [one sentence]
WHY I THINK IT NEEDS SEVERAL AGENTS: [say honestly]
THE PARTS I THINK ARE INDEPENDENT: [list them]
TOOLS AVAILABLE: [list them, marked read or write]
MY BASELINE, if I have run it: [single-agent score, tokens, latency from my eval set]
MY CONSTRAINTS: free tiers, and users waiting on slow 4G.

Deliver all of this:

1. Tell me which single-agent pattern I should try first - plan-then-execute, reflection with external criteria, evaluator-optimiser, or a router - and what it would have to fail at before multi-agent is justified.
2. State which of the three benefits multi-agent would actually buy me here: parallelism, context isolation, per-role tool scoping. If the answer is none, say so plainly and stop.
3. Check my independence claim: for each part I listed, say whether it truly needs no output from another part. Redraw anything that does as a workflow step.
4. Write the full contract for each worker: exact input, output schema in JSON, tool list with read or write marked, step budget, cost cap, timeout, and the failure return value.
5. Design the orchestrator: how it decomposes, how it merges, the WRITTEN rule for resolving two workers that disagree, and the confirmation that it holds the only write capability.
6. Design my critic so that it CAN fail me: give me criteria that are code or a checklist a stranger could apply, not a judgement. Then give me one piece of deliberately wrong output I should feed it as a test.
7. Estimate tokens and wall-clock latency for the multi-agent version versus my single-agent baseline. Show the arithmetic and state your assumptions.
8. Tell me what to log, including the correlation id, so I can attribute one wrong output to one agent in under five minutes.
9. Name the three coordination failures most likely in MY design - duplicated work, dropped subtask, conflicting results - and the specific check that would catch each.
10. Finally: give me the honest recommendation. Single agent or multi-agent, and what evidence would change your answer.
```

## Mission

**Objective.** One open-ended task in your own product built twice — a single-agent baseline and a multi-agent version — measured on the same eval set, with a written decision about which one you keep and why.

**Steps.**
1. **Frame** — pick a genuinely open-ended task with parts that look independent. Write what done means as something code can check.
2. **Baseline** — build the single-agent version with plan-then-execute, minimum tools and a step cap. Approve its plan yourself before it runs.
3. **Measure** — run your chapter twelve eval set and record score, total tokens and wall-clock latency.
4. **Justify** — write down which of the three benefits multi-agent would buy, and how you will measure that benefit specifically.
5. **Contract** — write every worker's contract before building: input, output schema, tools, budgets, failure return.
6. **Direct** — have AI build the orchestrator and workers. Only the orchestrator may write. Include a written conflict-resolution rule.
7. **Trace** — thread one correlation id through every agent so the whole run reads as one sequence.
8. **Measure again** — the same eval set, the same three numbers.
9. **Test the critic** — feed it output you know is wrong and record whether it caught it.
10. **Decide** — keep whichever version the numbers support, and write the decision down even if it means deleting a week's work.

**Upload.**
- Your single-agent baseline row: score, tokens, latency → screenshot
- Your multi-agent row: the same three numbers, same eval set → screenshot
- Every worker contract, in full: input, output schema, tools, budgets, failure return
- Your written conflict-resolution rule
- Proof that only one component can write → the tool lists, with writes marked
- One merged trace with the correlation id visible across all agents → screenshot
- Your critic test: the wrong output you fed it, and whether it caught it
- Your decision, in one paragraph: which version you kept and the number that decided it
- The AI conversation → share link in `ai_chat_url`
- Reflection, 150 words max → what multi-agent actually bought you, and one thing the AI got wrong

**Self-check.** Tick every box before you submit.
- [ ] I built and measured the single-agent baseline before the multi-agent version
- [ ] Both versions were scored on the same eval set
- [ ] I can name which of the three benefits I was buying, in one sentence
- [ ] Every worker has a written contract including an output schema
- [ ] Every worker has its own step budget, cost cap and timeout
- [ ] No worker depends on another worker's output
- [ ] Exactly one component can write, and I can show the tool lists that prove it
- [ ] I have a written rule for when two workers disagree
- [ ] One correlation id lets me read the whole run as a single trace
- [ ] I fed my critic output I knew was wrong, and recorded what happened
- [ ] My decision is based on the numbers, and I wrote it down even where it was unflattering

**Stretch.** Delete your multi-agent version if the baseline won, and write 100 words on what you learnt about reaching for complexity. If the multi-agent version won, instead find the single worker that contributes least and delete that, then re-measure.

## Stuck? Ask AI
```prompt
Explain the difference between plan-then-execute, reflection with a critic, evaluator-optimiser, and multi-agent orchestration. For each, give one concrete example from a student product and say what it costs and what it can test. Then tell me which one most people skip even though it is cheapest.
```

```prompt
My critic agent says "looks good" to everything, including output with a fabricated number in it. Explain why a model asked to judge its own work approves it. Then show me three critic designs with external criteria that CAN fail, in order of strength, and tell me how to test that each one works.
```

```prompt
I have seven agents and when the output is wrong I cannot tell which one caused it. Explain what I should have logged, including what a correlation id is and how to thread one through. Then tell me which of my seven agents is most likely unnecessary and what question would prove it.
```

## Quiz

### Q1 · L3
What are the three things multi-agent actually buys you?
- (a) Accuracy, creativity and speed
- (b) Parallelism, context isolation and per-role tool scoping ✅
- (c) Lower cost, simpler debugging and fewer prompts
- (d) Determinism, testability and attribution
> Those three are real and measurable. Multi-agent makes cost, debugging and attribution worse, not better, which is why you must name which benefit you are buying.

### Q2 · L4
A critic agent approves every output, including one with a fabricated statistic. What is the fix?
- (a) Use a larger model for the critic
- (b) Give it external criteria it can fail you on — a schema, a checklist, a test ✅
- (c) Run the critic twice and compare
- (d) Raise the critic's temperature for independence
> A model asked whether its own work is good says yes. A critic is only real when the criteria sit outside the model's taste, and best when they are code.

### Q3 · L4
Worker B needs worker A's output. What does that tell you?
- (a) A needs a larger step budget
- (b) They are sequential workflow steps, not parallel workers ✅
- (c) The orchestrator should run them twice
- (d) They should share one context
> Fan-out only helps when subtasks are independent. A fixed handoff chain is a workflow, which is cheaper, testable and debuggable.

### Q4 · L4
Why must exactly one component hold the write capability?
- (a) Providers limit write tools per key
- (b) Two agents writing the same resource is a race condition with a model in it ✅
- (c) It reduces token cost
- (d) Write tools cannot be parallelised
> Concurrent uncoordinated writes corrupt state in ways no prompt prevents. Centralising writes in the orchestrator, or better in deterministic code, removes the whole class.

### Q5 · L5
Your multi-agent version scores the same as the single-agent baseline, costs five times as much and is slower. What is the right action?
- (a) Add a quality-assurance agent to improve the score
- (b) Delete the multi-agent version and keep the baseline ✅
- (c) Keep both and let users choose
- (d) Reduce each worker's step budget and re-measure
> Complexity has to earn its place on the numbers. Equal score at five times the cost and worse latency is a more fragile system for nothing.

## Go deeper
- [Anthropic: Building effective agents — workflows and agent patterns](https://www.anthropic.com/engineering/building-effective-agents)
- [Anthropic: How we built our multi-agent research system](https://www.anthropic.com/engineering/built-multi-agent-research-system)
