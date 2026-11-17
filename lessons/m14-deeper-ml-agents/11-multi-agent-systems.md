---
id: m14-11-multi-agent
module: m14-deeper-ml-agents
slug: multi-agent-systems
title: "Multi-Agent Systems: Orchestrators, Sub-Agents and Handoffs"
order: 11
duration_min: 25
is_free: false
mission: true
volatility: medium
last_reviewed: 2026-11-16
---

## Key learnings
- Run an orchestrator against real traffic with a per-run budget ceiling that can cut it off
- Tell delegation from a handoff, and write the payload contract for each
- Recover from partial failure with idempotent workers and one declared policy
- Keep durable state so a crash costs one step instead of the whole run
- Catch a multi-agent system doing work one agent would have done

## Lesson
### The pain

M10 chapter 15 designed multi-agent systems properly: orchestrator and workers, the sub-agent contract, the single-writer rule, the duty to beat a single-agent baseline. New here is what happens when real users, real money and a 3 am crash are involved.

An orchestrator fans out to six workers. One tool times out, there is no policy for a partial result, so it retries the whole fan-out and the five that succeeded run again, each re-sending a message. At 3 am the same system hits a loop the step budget should have caught, but the budget was per worker, and nine workers spent ₹2,100.

### The idea

**In production, a multi-agent run is one distributed transaction with a shared budget, durable state and an idempotency rule — and the orchestrator's job is to enforce all three.**

Design questions were M10's. Production adds four: where state lives when the process dies, what the single budget is and what cuts the run off, what happens when some workers fail, and what travels on a handoff.

### The analogy

Lunch rush. The head cook is the orchestrator: she assigns stations and she alone plates up. Three things matter now that did not at design time. One gas budget for the service, not one per station. A slip board, so if she is called away the next person sees what is made. And re-reading a slip makes no second plate.

**Where it breaks.** Workers fail silently, so the orchestrator must *check*, not trust — hence the schema it validates.

### Picture it

```
            ┌──────────────────────────────────────┐
  request ─▶│  ORCHESTRATOR                        │
            │  holds: ONE budget ledger (₹ + steps)│
            │         the only write capability    │
            │         the approval gate            │
            └───┬──────────────────────┬───────────┘
                │ via a QUEUE          │ reads/writes
     ┌──────────┼──────────┐           ▼
     ▼          ▼          ▼     ┌──────────────┐
  worker A   worker B   worker C │ DURABLE STATE│
  read-only  read-only  read-only│ run id, each │
  idempotent idempotent idempotent│ step's result│
     │          │          │     └──────────────┘
     └──────────┴──────────┘       crash ──▶ resume
                │                          from last step
       results validated against
       the schema, then ONE write

  concurrency limit: 3 in flight · budget spent: ₹4.10 of ₹8
```

The orchestrator holds one budget for the whole run, the only write capability and the approval gate, dispatches read-only idempotent workers through a queue under a concurrency limit, and records every step in durable state so a crash resumes instead of restarting.

### Partial failure, and why idempotency is not optional

Five workers succeed, two fail. That is the normal case, and one policy must be chosen in advance.

- **All-or-nothing with compensation** — undo the five. Impossible once a message has been sent.
- **Best effort with gaps declared** — use the five, name the two missing.
- **Retry the failed subset only** — usually right, and safe only if every action carries an **idempotency key**, since a retry is indistinguishable from a duplicate request.

Classify failures too — transient, permanent, or the agent's mistake — and retry only the first.

### Budgets that can actually stop a run

A per-agent budget multiplies: nine workers with ten steps each is a ninety-step run, and a growing transcript makes cost grow faster than steps.

So the ceiling is **one ledger for the run**, in steps and rupees, decremented by every agent including the orchestrator, checked before each dispatch, and enforced by refusing to dispatch rather than warning. Beside it: a wall-clock deadline, a concurrency limit, and an alert that pages a human mid-run.

### Where it lives

- M10 chapters 13 to 15 — whether to do this at all, the loop, the contracts
- M04 chapter 16, M05 chapters 4, 8 — queues, rate limits, timeouts
- M07 chapters 5 to 7 — traces, span tree, alert
- Chapter 10 — proof multi-agent still beats the baseline
- Chapters 13, 14 — versioned prompts; the gateway enforcing the budget

### Words to own

| Term | Plain meaning | Example |
|---|---|---|
| Delegation | Orchestrator asks, worker returns, orchestrator continues | The default |
| Handoff | Control transfers; the first agent stops | Support to billing |
| Payload contract | What state travels on a handoff, and what must not | Identity, not notes |
| Hop budget | Maximum handoffs per conversation | Two, then a human |
| Budget ledger | One step and rupee allowance for the whole run | ₹8 and 40 steps |
| Concurrency limit | How many workers may be in flight at once | 3 |
| Durable state | Step results stored outside the process | Resume after a crash |
| Idempotency key | Same key, same action performed once | Per worker action |
| Partial failure | Some workers succeeded, some did not | 5 of 7 |
| Compensation | Undoing completed work after a later failure | Often impossible |
| Span tree | The nested trace of a whole run | One span per worker |
| Run id | The id that joins every log line of one run | Correlation id |

### Tradeoffs and gotchas

**The baseline is not a one-time check.** Re-measure the single-agent version quarterly; the honest outcome is sometimes deleting the orchestrator.

**Durable state is written before the action:** record the intent, act, record the result. A crash in between is then a retry on an idempotent action.

**Approval gates belong only at the orchestrator,** never inside a worker. And fan-out makes latency worse before better: three waves plus the synthesis.

**A handoff is not a delegation.** Delegation returns a schema-checked result; a handoff transfers ownership and the first agent stops. Write down the payload contract (what travels, what must not), the hop budget — two or three, since A-to-B-to-A looks like politeness — and the terminal rule: after the last hop, a human.

### Verify it

The red flag is still M10's: a multi-agent system where one agent would do.

1. **Demand the baseline number, dated.**
2. **Ask where the budget lives.** Per-agent means it does not exist, and exhaustion must refuse to dispatch.
3. **Ask what happens if the process dies at step 14.** "It restarts" means no durable state.
4. **Ask what happens when five of seven succeed.** One named policy, with an idempotency key if it retries.
5. **Check the concurrency limit and rate-limit backoff.**
6. **Ask for per-worker cost on one run.**
7. **For a handoff, demand the payload contract and hop budget.**

Red flags: five agents doing one agent's work; a per-agent budget as a ceiling; no durable state; no partial-failure policy; retries without idempotency keys; no hop budget.

## Walkthrough
1. Take your M10 chapter 15 multi-agent design, or build the simplest version of one. Write down, in one sentence each, the three things it buys you — parallelism, context isolation, scoped tools — and which of them you can actually demonstrate.
2. Re-measure the single-agent baseline today, on your chapter 10 regression suite, recording score, tokens, rupees and p95 latency. If multi-agent does not beat it, stop here and write that down as your result.
3. Add a run id to every log line of every agent, and build a span tree view for one run. Confirm you can see each worker's start, end, tokens and cost.
4. Replace per-agent budgets with one ledger for the run: a step count and a rupee amount, decremented by every agent, checked before each dispatch. Make exhaustion refuse to dispatch and return a partial result.
5. Add a wall-clock deadline for the whole run and a concurrency limit. Set the limit low — three — and measure what it does to latency.
6. Move fan-out onto a queue from M04 chapter 16, so workers run as jobs rather than inside one request. Confirm a worker crash does not take down the orchestrator.
7. Add durable state: before each worker dispatch write the intent, after each result write the result, keyed by run id and step. Then kill the process mid-run and confirm it resumes from the last recorded step.
8. Give every worker action an idempotency key and prove it: replay the same step twice and confirm exactly one effect.
9. Choose and implement one partial-failure policy. Then force it — make one worker's tool fail deliberately — and confirm the output either declares the gap or retries only the failed worker.
10. If you have a handoff rather than only delegation, write the payload contract and the hop budget, then test a loop: make agent A hand to B and B hand back, and confirm the budget stops it and a human is notified.
11. Run a cost-explosion drill: deliberately let one worker loop, confirm the ledger cuts the run off and an alert reaches a person, and record the rupees spent first. Then update your decision log with the new baseline comparison, the budget numbers, the partial-failure policy, and the single measurement that would make you delete the orchestrator.

## Talk to AI
```prompt
CONTEXT:      I am a student on an AI-native engineering program. I direct AI
              and verify its output; I do not write code by hand. I already
              have a multi-agent design with an orchestrator, written
              sub-agent contracts with output schemas, a single-writer rule
              and correlation ids, and I know the five stopping conditions for
              an agent loop. Do not re-teach any of that. My team of three
              runs [product in two sentences] with [number] real users in
              India. My orchestrator does [describe it] and dispatches
              [number] workers that [describe them]. My single-agent baseline
              scored [number] on my regression suite at [tokens] tokens and
              ₹[number] per run; the multi-agent version scores [number] at
              [tokens] and ₹[number]. It is triggered by [a user request / a
              webhook / a schedule].

GOAL:         Run this in production safely: one shared budget that can stop
              the run, durable state that survives a crash, an idempotency
              rule, a declared partial-failure policy, and per-worker cost
              visibility.

REQUIREMENTS: 1. Replace my per-agent budgets with one ledger for the whole
                 run in steps and rupees. Tell me where it is checked, what
                 refusing to dispatch looks like, and what the user sees
              2. Give me a wall-clock deadline and a concurrency limit for my
                 worker count, and tell me what the limit does to my latency
              3. Design the durable state: what is written before a dispatch,
                 what after a result, keyed how, and how a resume works. Then
                 tell me how to test it by killing the process mid-run
              4. Give every worker action an idempotency key. Tell me what the
                 key should be derived from, and how I prove a replayed step
                 produces exactly one effect
              5. Recommend one partial-failure policy for my case out of
                 all-or-nothing with compensation, best effort with declared
                 gaps, or retry-the-failed-subset. Justify it, and tell me
                 what my output must be able to say
              6. Tell me how to classify a worker failure as transient,
                 permanent or the agent's own mistake, and which of the three
                 may be retried
              7. If my design has a real handoff rather than only delegation,
                 write the payload contract - what state travels, what must
                 NOT - and set a hop budget with a terminal human rule
              8. Tell me exactly what to log per worker per run so I can
                 attribute cost and failure, and what the alert should be when
                 one run costs more than a multiple of my median
              9. Give me a cost-explosion drill: how to deliberately make a
                 worker loop, and what I should observe

CONSTRAINTS:  Free tier only. My provider has a low rate limit, so tell me
              where backoff belongs. Plain English for a second-language
              reader. Label every command READ-ONLY or CHANGES-THINGS. Only
              the orchestrator may write anything or ask a human for approval.
              No worker may send a message to a user. Keep the whole design
              runnable without Kubernetes.

NON-GOALS:    Do not redesign my agent contracts or re-explain why
              multi-agent buys parallelism. Do not add more agents. Do not
              propose agent-to-agent conversation between workers. Do not
              propose a paid orchestration platform. Do not let a worker hold
              an approval gate.

ACCEPTANCE:   - One budget ledger for the run exists, in steps and rupees, and
                exhaustion refuses to dispatch
              - A wall-clock deadline and a concurrency limit are both set
              - Killing the process mid-run resumes from the last step
              - A replayed worker step produces exactly one effect
              - One partial-failure policy is implemented and was tested by
                forcing a worker to fail
              - Per-worker tokens and rupees are logged against a run id
              - Any handoff has a written payload contract and a hop budget
              - A deliberate worker loop was cut off by the ledger and alerted
              - A dated single-agent baseline comparison exists from this week

ASK:          First, look at my baseline numbers against my multi-agent
              numbers and tell me honestly whether this system should exist at
              all. If the gap does not justify the complexity, say so and stop.
              Only if it does should you design the production controls.
```

## Mission

**Objective.** Run your multi-agent system against your own product's real traffic with a shared budget ceiling, durable state, idempotent workers and a declared partial-failure policy — and a dated comparison against the single-agent baseline.

**Steps.**
1. **Frame** — write which of parallelism, context isolation and scoped tools your design actually delivers, and which you only hoped for.
2. **Re-baseline** — run the single-agent version on your chapter 10 regression suite today. Record score, tokens, rupees and p95. If it wins, your mission result is the decision to delete the orchestrator, and that is a pass.
3. **Brief** — write a Director's Brief for the production controls: one budget ledger, a deadline, a concurrency limit, durable state, idempotency keys, and one partial-failure policy.
4. **Direct** — implement them, with fan-out through a queue and the orchestrator holding the only write and the only approval gate.
5. **Verify crash recovery** — kill the process mid-run and confirm it resumes from the last recorded step.
6. **Verify idempotency** — replay one worker step and confirm exactly one effect.
7. **Verify partial failure** — force one worker to fail and confirm your policy behaves as declared.
8. **Verify the ceiling** — deliberately make a worker loop and confirm the ledger cuts the run off and an alert reaches a person. Record the rupees spent first.
9. **Document** — the span tree of one real run with per-worker cost, the dated baseline comparison, and the measurement that would make you delete this design.

**Upload.**
- Your Director's Brief → post caption or an attached file
- A span tree for one real run, with per-worker tokens and rupees → screenshot
- Your dated single-agent versus multi-agent comparison table
- Proof of crash recovery: logs showing a resume from the last step → screenshot
- Proof of idempotency: one replayed step, one effect → screenshot
- Proof of your partial-failure policy with one worker forced to fail
- Proof of the budget cut-off and the alert, with rupees spent before it fired
- Your handoff payload contract and hop budget, if you have a handoff
- The AI conversation → share link in `ai_chat_url`
- Reflection, 150 words max → one tradeoff you chose, and one thing the AI got wrong

**Self-check.** Tick every box before you submit.
- [ ] I re-measured the single-agent baseline this week, not last month
- [ ] My multi-agent version beats it on a number I can show, or I deleted it
- [ ] There is exactly one budget ledger for the whole run, in steps and rupees
- [ ] Exhausting it refuses to dispatch rather than logging a warning
- [ ] A wall-clock deadline and a concurrency limit both exist
- [ ] I killed the process mid-run and watched it resume from the last step
- [ ] Every worker action has an idempotency key and I proved one replay
- [ ] One partial-failure policy is declared and I forced it to happen
- [ ] Only the orchestrator writes anything or asks for human approval
- [ ] Per-worker tokens and rupees are logged against a run id
- [ ] A deliberate loop was cut off and alerted a human
- [ ] Any handoff has a hop budget and ends with a human

**Stretch.** Run the same real traffic through the single-agent version for one day and the multi-agent version for the next, and compare cost per request, p95 latency and your regression-suite categories. Report whether the complexity earned its place in production, not in a notebook.

## Stuck? Ask AI
```prompt
Here is my orchestrator's design: [paste it, including the worker list and what each does]. Do not improve it. Instead, tell me honestly whether a single agent with the same tools would produce the same result, and what specific evidence would settle that. Then tell me, for each worker, whether it is genuinely independent of the others' output — and if any one depends on another, explain why that pair is a sequential workflow rather than parallel workers.
```

```prompt
Explain the difference between delegation and a handoff in a multi-agent system, using a support conversation that turns into a billing dispute. For the handoff, write the payload contract: exactly what state should travel, what must not travel and why, and who verifies the user's identity. Then tell me what a handoff loop looks like, how I would detect it in my traces, and what the hop budget and terminal rule should be.
```

```prompt
My orchestrator fans out to seven workers and two of them failed. Walk me through the three possible policies — all-or-nothing with compensation, best effort with declared gaps, and retry the failed subset — and for each one tell me what my final output must be able to say, what could go wrong, and what idempotency requirement it imposes. Then ask me what my workers actually do before telling me which policy to pick.
```

## Quiz

### Q1 · L3
Nine workers each have a 10-step budget. What is wrong with calling that the run's ceiling?
- (a) Nothing, as long as each worker respects its own budget
- (b) The run can spend up to ninety steps, because the budget is not shared ✅
- (c) Step budgets should be measured in tokens instead
- (d) The orchestrator does not consume steps
> Per-agent budgets multiply. Production needs one ledger for the whole run, in steps and rupees, decremented by every agent and checked before each dispatch, with exhaustion refusing to dispatch.

### Q2 · L4
Your orchestrator retries a fan-out after one of seven workers times out, and users receive duplicate messages. What was missing?
- (a) A higher concurrency limit
- (b) A partial-failure policy plus idempotency keys on worker actions ✅
- (c) A longer timeout on the failing worker
- (d) More workers, so one failure matters less
> Retrying the whole fan-out re-ran six successful workers. Retrying only the failed subset is the usual answer, and it is only safe when the same key means the same action happens once.

### Q3 · L4
A support agent determines an issue is a billing dispute and transfers the live conversation to a billing agent. What must be written down?
- (a) The billing agent's step budget only
- (b) The payload contract, a hop budget, and what the receiving agent must not be told ✅
- (c) A critic to review the transfer
- (d) Nothing; a handoff is the same as delegation
> A handoff transfers ownership, so state crosses a permission boundary. Without a payload contract and a hop budget, A-to-B-to-A loops happen and information leaks between agents with different permissions.

### Q4 · L4
Your process dies at step 14 of a 20-step run and starts again from step 1. What is missing?
- (a) A concurrency limit
- (b) Durable state written before and after each step, keyed by run id ✅
- (c) A larger wall-clock deadline
- (d) A retry policy on the orchestrator
> Step results held only in process memory are lost with the process. Write the intent before the action and the result after, so a crash becomes a resume on an idempotent step.

### Q5 · L5
Your multi-agent system was 11 points better than a single agent when you designed it six months ago. What should you do now?
- (a) Nothing; the comparison was made properly at the time
- (b) Add a critic agent to improve it further
- (c) Re-measure the single-agent baseline today, and be willing to delete the orchestrator ✅
- (d) Increase the concurrency limit to reduce latency
> Models improve and the cheap version often catches up, while your complexity cost stays. A dated baseline is evidence; a six-month-old one is history, and deleting an orchestrator is a legitimate result.

## Go deeper
- [Anthropic: how we built a multi-agent research system](https://www.anthropic.com/engineering/built-multi-agent-research-system)
- [OpenTelemetry: traces, spans and context propagation](https://opentelemetry.io/docs/concepts/signals/traces/)
