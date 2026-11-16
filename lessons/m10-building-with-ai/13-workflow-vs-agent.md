---
id: m10-13-workflow-vs-agent
module: m10-building-with-ai
slug: workflow-vs-agent
title: "Workflow vs Agent, and When Deterministic Wins"
order: 13
duration_min: 25
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

A team builds "an agent" for order intake. It handles a WhatsApp message end to end: read the message, work out the items, look up prices, compute the total, create the order, send a confirmation. Six steps, every time, in that order.

They give the model all six as tools and let it decide the sequence. What they get back is a system that sometimes skips the price lookup and invents a total, occasionally calls order creation twice so the student gets charged twice, takes between four and nineteen seconds, costs several times what they budgeted, and — when an order goes wrong — offers no way to tell which step failed, because there were no steps.

Two days later they rewrite it as one model call that extracts items into chapter three's schema, followed by about fifteen lines of perfectly ordinary code. It runs in under a second, costs a fraction, double-creates nothing because the insert has an idempotency key, and every failure names the line it happened on.

Nothing was wrong with the model. They used an agent for a problem that had no decisions in it.

### The idea

**If you can write the steps down in advance, it is a workflow. An agent is for when you genuinely cannot — when the next action depends on what the last one found, in ways you could not enumerate. An agent costs you determinism, and you should make it earn that.**

The instinct that an agent is the sophisticated choice has this exactly backwards. Choosing the simplest structure that solves the problem is the sophisticated choice. Reaching for an agent because it sounds advanced is how a one-second feature becomes a fourteen-second one.

### The analogy

Thali service at the canteen is a workflow. Five items, five compartments, same order, two hundred times. A new helper learns it in ten minutes, every plate looks the same, and when the plates stop coming you know exactly which station is stuck.

Catering a wedding is not a workflow. The guest count moved, three families are Jain, the hall has one gas point, and somebody must decide what to cook less of. That needs a person making judgement calls as information arrives.

A canteen that treats thali service as a judgement call serves two hundred different thalis and runs out of rice by one o'clock.

**Where the analogy breaks:** a human doing a repetitive job still notices the rice is running low and says something. A deterministic pipeline notices nothing it was not told to check. So a workflow needs its error paths written out explicitly — what happens when the price lookup returns nothing, when the message is in a language you cannot parse, when the insert fails. The agent's one genuine advantage is handling the case you did not anticipate, and your workflow has to pay for that with explicit handling instead.

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

Six rungs of increasing power and decreasing predictability. Climb one rung only when the rung below has demonstrably failed on your own eval set.

### The three questions

**Can you write the steps down in advance?** If you can list them on paper, it is a workflow. Write them as code.

**Is the number of steps bounded and known?** "Always exactly six" is a workflow. "Between two and forty, depending on what it finds" is agent territory — and that unbounded number is also your cost risk, which chapter fourteen caps.

**Does the next action genuinely depend on the content of the last result, in a way you cannot enumerate?** This is the real test. "If the message is a complaint, do the complaint thing" is a *router* — a model making one choice among known branches, after which everything is fixed. That is rung three, not rung five, and it covers an enormous number of features teams build agents for.

If all three answers point to workflow, an agent will be slower, dearer, less testable and no more capable.

### What deterministic code must always own

Some things never belong to a model, no matter how good it gets. Not because of accuracy — because of accountability.

**Money arithmetic.** Prices, totals, discounts, splits, rounding, tax. Integers in paise, in your code, every time. Chapter three said never ask for a total; this is the same rule at the system level.

**Permission checks.** Whether this student may cancel that order. A model's cooperation is not an authorisation decision, as chapter four's wrong-user test showed.

**The ordering of writes.** What is inserted before what, what is inside a transaction, what has an idempotency key. The double-charged student in the opening story is this failure exactly.

**Anything that must be byte-identical every time.** An invoice number. A receipt. A compliance log. "Mostly the same" is not a property you can build on.

The clean division is: the model handles the **unstructured** part — reading a message, understanding a document, writing a sentence — and your code handles the **consequential** part. Chapter three's schema is the boundary between them.

### Where it lives

A workflow is ordinary application code in M04's sense, with model calls at named points. It can run in a request, or as a queued job, and every step can be logged, retried and tested individually with M06's test suite.

An agent is the loop in chapter fourteen. The design most real products converge on is a **deterministic workflow with an agent inside one step** — a bounded place where open-ended work happens, with the money, the permissions and the writes safely outside it. That shape is also what makes the permission model from M00 workable: you only need a gate around the one step that can act.

And remember chapter two's build-time trick. An agent exploring a problem once, while you watch, producing a workflow you then run deterministically forever, is often the best of both.

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

**An agent whose trace is identical every run is a workflow you are overpaying for.** Log ten real runs and compare the tool sequences. If they match, you have discovered a workflow. Write it down and delete the loop.

**Non-determinism is a liability, not a feature.** It means you cannot reproduce a bug, cannot write a test that passes twice, and cannot tell a user why something happened. For anything touching money or permissions that is disqualifying, regardless of accuracy.

**Routers are the most underused rung.** One cheap classification call, then four fixed paths. You get the flexibility people want agents for, with full testability, because each branch is ordinary code you can test independently.

**Workflows fail silently if you skip the error paths.** The agent's real advantage is coping with the unexpected. If you choose a workflow, you must write out what happens on every failure — empty result, wrong language, failed insert, timeout — or you have simply moved the unpredictability into production.

**Cost is bounded in a workflow and open-ended in an agent.** Six steps is six calls. "Until it is done" is however many it takes, and a loop that cannot decide when to stop is a bill. Chapter fourteen exists for this.

**Nobody is impressed by an agent that is slower.** Demo day rewards a feature that works in one second. "We used an agent" is not a result; "it handles a Marathi voice note in under two seconds for under a rupee" is.

**Do not let the word creep.** Teams call any model call an agent, and then cannot reason about it. If the model is not choosing its next action, it is not an agent. Use the words from the ladder and your design conversations get sharper immediately.

### Verify it

**Write the steps on paper before you build anything.** If you succeed, you have just designed a workflow and the decision is made. If you cannot get past step two without "it depends what it finds", you may genuinely need an agent.

**Take ten real runs and compare the tool sequences.** Identical sequences mean a workflow. Wildly varying sequences mean either a real agent problem or an agent that is lost — chapter twelve's ladder tells you which.

**Trace the money.** Follow every rupee through your design and confirm that no amount is ever produced by a model. Any arithmetic in a model's output is a bug waiting for a demo.

**Run the same input twice and diff the result.** A workflow must produce the identical order row, the identical total and the identical confirmation. If it does not, something non-deterministic is sitting where it should not be.

**Ask what a failure report looks like.** Can you say which step failed, with a log line? If a failure is only describable as "the agent got confused", you cannot support this feature.

Red flags: an agent whose steps are the same on every run; a model computing a price, total or discount; a write with no idempotency key; an agent chosen before any simpler rung was tried; no step-level logging; no explicit error path for an empty or unparseable input; the word "agent" used for a single model call; and a design where an agent sits *around* the money rather than inside one bounded step.

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
