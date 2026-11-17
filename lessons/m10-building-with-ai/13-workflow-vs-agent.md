---
id: m10-13-workflow-vs-agent
module: m10-building-with-ai
slug: workflow-vs-agent
title: "Workflow vs Agent, and When Deterministic Wins"
order: 13
duration_min: 15
is_free: false
mission: false
volatility: low
last_reviewed: 2026-11-16
---

## Key learnings
- Decide between a workflow and an agent with three honest questions
- Climb the ladder from one call to an agent only when a rung fails
- Name what an agent costs you: determinism, testability and predictable cost
- Keep money, permissions and write ordering in deterministic code forever
- Put the agent in one bounded step of a workflow rather than around it

## Lesson

### The pain

A team builds "an agent" for order intake: read the message, work out the items, look up prices, total it, create the order, confirm. Six steps, every time, in that order — and they hand the model all six as tools and let it choose the sequence.

What comes back sometimes skips the price lookup and invents a total, sometimes creates the order twice, and cannot say which step failed, because there were no steps. Two days later they rewrite it as one extraction call plus fifteen lines of ordinary code: under a second, a fraction of the cost. They had used an agent for a problem with no decisions in it.

### The idea

**If you can write the steps down in advance, it is a workflow. An agent is for when you genuinely cannot — when the next action depends on what the last one found, in ways you could not enumerate. An agent costs you determinism, and it should have to earn that.**

The instinct that an agent is the sophisticated choice has it backwards. Choosing the simplest structure that solves the problem is the sophisticated choice.

### The analogy

Thali service is a workflow: five items, five compartments, same order, two hundred times. A new helper learns it in ten minutes, and when plates stop coming you know which station is stuck. Catering a wedding is not: the guest count moved, three families are Jain, and somebody has to decide.

**Where the analogy breaks:** a human doing a repetitive job still notices the rice running low. A deterministic pipeline notices nothing it was not told to check, so a workflow needs its error paths written out — empty price lookup, unparseable language, failed insert. The agent's one real advantage is the case you did not anticipate; your workflow pays for that with explicit handling.

### Picture it

```
  1  ONE CALL      prompt ──▶ model ──▶ result
                   classify, extract, rewrite

  2  CHAIN         call A ──▶ call B ──▶ call C
                   fixed order, each feeds the next

  3  ROUTER        model picks 1 of 4 known branches
                       │
                   ┌───┴───┬───────┬────────┐
                   order  complaint  menu   other
                   (then each branch is fixed)

  4  WORKFLOW      your code drives; it calls the
     + MODEL       model at 2 named points

  5  AGENT         model chooses the next tool, loops
                   until a stopping condition

  6  MULTI-AGENT   several agents, each with a role
```

Six rungs of increasing power and decreasing predictability. Climb one only when the rung below has demonstrably failed on your eval set.

Three questions decide the rung. **Can you write the steps down in advance?** Then write them as code. **Is the number of steps bounded?** "Always six" is a workflow; "two to forty, depending what it finds" is agent territory, and that unbounded number is your cost risk. **Does the next action genuinely depend on the last result in a way you cannot enumerate?** That is the real test: "if the message is a complaint, do the complaint thing" is a *router* — one model choice among known branches. Rung three, not rung five.

### Where it lives

A workflow is ordinary application code with model calls at named points, so every step can be logged, retried and tested on its own. An agent is the loop in chapter fourteen. Most real products converge on a **deterministic workflow with an agent inside one step**, with the money, the permissions and the writes outside it. Four things never belong to a model:

- **Money arithmetic** — prices, totals, discounts, splits, rounding, in integer paise in your code
- **Permission checks** — a model's cooperation is not an authorisation decision
- **The ordering of writes** — what is inside a transaction, what has an idempotency key
- **Anything that must be byte-identical** — an invoice number, a receipt, a compliance log

### Words to own

| Term | Plain meaning | Example |
|---|---|---|
| Workflow | Steps you wrote down in advance | Extract, price, insert, send |
| Chain | A fixed sequence of model calls | Summarise then translate |
| Router | A model picking one of known branches | Order vs complaint |
| Agent | A model choosing its next tool in a loop | Open-ended research |
| Determinism | Same input, same steps, same output | A priced order |
| Bounded steps | A known maximum number of steps | Exactly six |
| Model-in-the-loop | Code drives, model is consulted | Two named call sites |
| Idempotency key | A token making a repeat safe | One order, not two |
| Escalation | Handing the hard case to a human | Unparseable message |
| Orchestration | Whoever decides what happens next | Your code, or the model |

### Tradeoffs and gotchas

**An agent whose trace is identical every run is a workflow you are overpaying for.** Log ten runs and compare the tool sequences; if they match, write the workflow down and delete the loop.

**Non-determinism is a liability, not a feature.** You cannot reproduce a bug, write a test that passes twice, or tell a user why something happened.

**Routers are the most underused rung.** One cheap classification call, then four fixed paths: the flexibility people want from agents, with every branch testable as ordinary code.

**Do not let the word creep.** If the model is not choosing its next action, it is not an agent — and nobody is impressed by an agent that is slower.

### Verify it

- **Write the steps on paper first.** If you succeed, you have designed a workflow. If you cannot get past step two without "it depends what it finds", you may need an agent.
- **Take ten real runs and compare the tool sequences.** Identical means workflow; wildly varying means a real agent problem, or a lost agent.
- **Trace the money.** Confirm no amount is ever produced by a model.
- **Run the same input twice and diff the result.** A workflow must produce the identical row and total.
- **Ask what a failure report looks like.** If a failure is only describable as "the agent got confused", you cannot support this feature.

Red flags: an agent whose steps are the same every run; a model computing a price or discount; a write with no idempotency key; an agent chosen before a simpler rung was tried; no step-level logging; no error path for unparseable input; "agent" used for a single model call; an agent sitting *around* the money rather than inside one bounded step.

## Walkthrough
1. Pick one AI feature you are planning. Write its steps on paper, numbered, in order, as if a new teammate had to do it by hand.
2. Count the steps. If you reached a complete list, write "workflow" at the top of the page. If you could not get past step two without "it depends", write "candidate agent".
3. For each step, mark whether the work is unstructured — reading, understanding, writing prose — or consequential: money, permissions, writes, identifiers.
4. Draw a line. Every unstructured step is a model call. Every consequential step is your code. Note where the structured output from chapter three carries data across the line.
5. Identify any step that is really a choice among known options and implement it as a router: one cheap classification call, then fixed branches.
6. Build the workflow version first, even if you believe you need an agent. Time it, price it, and run it twice on the same input to confirm the results are identical.
7. Write the explicit error path for every step: empty result, unparseable input, failed lookup, timeout, duplicate submission. A workflow with no error paths is not finished.
8. Run your eval set from chapter twelve against the workflow and record the score. This is the number an agent would have to beat to justify itself.
9. Only if the workflow failed on specific rows, identify what those rows need that fixed steps cannot provide. Write that down in one sentence — it is the brief for the agent step.
10. Put the agent, if you still need one, inside a single step of the workflow, with money, permissions and writes outside it. Then compare both versions on score, latency and cost, and record the comparison in your decision log.

## Talk to AI
```prompt
Help me decide whether my AI feature should be a workflow or an agent, and push back if I am over-engineering it.

MY FEATURE, in one sentence: [what it does for whom]
WHAT I THINK THE STEPS ARE: [number them, in order, as best I can]
WHERE I GET STUCK WRITING THE STEPS: [say honestly where "it depends" appears]
WHAT TOUCHES MONEY: [list every amount, price, total, discount, split]
WHAT TOUCHES PERMISSIONS: [who may do what to whose data]
WHAT WRITES TO MY DATABASE: [list every insert or update]
MY CONSTRAINTS: free tiers, slow 4G users, demo in [n] weeks.

Do all of the following:

1. Place my feature on the ladder: one call, chain, router, workflow with model calls, agent, or multi-agent. Justify the rung and say what would have to be true for the rung below to be enough.
2. Answer the three questions for me explicitly: can the steps be written in advance, is the step count bounded, does the next action genuinely depend on the last result in a way nobody can enumerate.
3. Design the workflow version in full, even if you think I need an agent. Name every step, say whether it is a model call or ordinary code, and show where structured output carries data between them.
4. List everything that must stay in deterministic code no matter what, with the reason: money, permissions, write ordering, identifiers.
5. Write the explicit error path for every step: empty result, unparseable input, failed lookup, timeout, duplicate submission. Include what the user sees in each case.
6. Identify any step that is secretly just a choice among known branches, and rewrite it as a router.
7. Estimate latency and cost per run for the workflow version and for an agent version, and state the assumptions.
8. Tell me plainly whether I need an agent at all. If I do, tell me which single step it belongs inside, and what must stay outside that step.
9. Name the thing most likely to go wrong in the workflow version that an agent would have handled, and what I should write to cover it instead.
```

## Stuck? Ask AI
```prompt
Explain the difference between a chain, a router and an agent, with one concrete example of each from a canteen ordering app. For each, tell me what I can test, what I can predict about cost and latency, and what I lose by moving up to the next one.
```

```prompt
I built an agent and it sometimes skips a step and sometimes repeats one, and a student got charged twice. Explain what went wrong structurally rather than as a prompting problem. Then show me the same feature as a workflow, and tell me exactly where the idempotency key goes and why.
```

```prompt
Here are the steps of my AI feature: [paste your numbered list]. Tell me which steps must be deterministic code and which genuinely need a model, and why. Then tell me whether this is a workflow or an agent, and be blunt if I am reaching for an agent because it sounds more impressive.
```

## Quiz

### Q1 · L3
What is the clearest test that a feature should be a workflow rather than an agent?
- (a) It uses fewer than five tools
- (b) You can write all its steps down in advance ✅
- (c) It needs to run in under a second
- (d) It does not touch money
> If the sequence is knowable before runtime, write it as code. An agent's only real advantage is deciding a next step you could not enumerate, and you pay for that in determinism.

### Q2 · L4
An "agent" produces the same tool sequence on all ten logged runs. What have you learnt?
- (a) The agent is working well and is stable
- (b) It is a workflow you are paying agent prices for ✅
- (c) The stopping condition is too strict
- (d) The tool descriptions need improving
> Identical traces mean there were no decisions to make. Writing the sequence down makes it faster, cheaper, testable and debuggable with nothing lost.

### Q3 · L4
Which of these must stay in deterministic code regardless of model quality?
- (a) Reading a WhatsApp message in Hinglish
- (b) Writing the confirmation sentence
- (c) Computing the order total and applying the discount ✅
- (d) Summarising a complaint for staff
> Money is about accountability, not accuracy. Totals, rounding and splits belong in integer arithmetic in your code, with the model handling only the unstructured parts.

### Q4 · L4
"If the message is a complaint, run the complaint path; if an order, run the order path." What rung is this?
- (a) A single model call
- (b) A router, after which each branch is fixed ✅
- (c) An agent, because the model decides
- (d) Multi-agent orchestration
> One classification call followed by known branches is a router. It gives the flexibility people reach for agents to get, while every branch stays independently testable.

### Q5 · L5
You choose a workflow over an agent. What must you now add that the agent would have given you for free?
- (a) A larger context window
- (b) Explicit error paths for every step ✅
- (c) An eval set
- (d) Prompt caching
> An agent's genuine advantage is coping with the unanticipated. Choosing fixed steps means writing out what happens on empty results, unparseable input, failed lookups and timeouts.

## Go deeper
- [Anthropic: Building effective agents](https://www.anthropic.com/engineering/building-effective-agents)
- [OpenAI: A practical guide to building agents](https://cdn.openai.com/business-guides-and-resources/a-practical-guide-to-building-agents.pdf)
