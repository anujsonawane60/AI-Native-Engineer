---
id: m14-13-mlops-llmops
module: m14-deeper-ml-agents
slug: mlops-llmops
title: "MLOps and LLMOps: Registry, Versioning, Drift and Tracing"
order: 13
duration_min: 30
is_free: false
mission: false
volatility: medium
last_reviewed: 2026-11-16
---

## Key learnings
- Record your AI system's version as a tuple of six pinned artefacts, not a number
- Put a prompt id and a model version on every trace so any answer can be reconstructed
- Detect the three drifts: changing inputs, a changed provider model, and a changed judge
- Roll back the whole tuple, because restoring only the code restores nothing
- Catch "it worked last Tuesday" when nobody recorded what last Tuesday consisted of

## Lesson

### The pain

By this point in the module you have produced a labelled dataset, a LoRA adapter, a quantized build, a serving configuration, a regression suite, a calibrated judge and possibly an orchestrator with several agents. Every one of those is an artefact that can change independently, and none of them is in the same place.

So here is the Tuesday problem. A user complains on Thursday that the assistant's answers have become worse. On Tuesday they were good. The team goes looking and finds: the system prompt was edited on Wednesday by a teammate improving tone; the retrieval index was rebuilt on Tuesday night when new documents were ingested; the provider's model name still says what it said, but the provider shipped an update; and the eval judge was pointed at a newer model last week so the historical scores are not comparable either.

Four changes, no record of which was in effect when. The honest answer to "what did the system look like on Tuesday?" is that nobody knows, and so the question "what made it worse?" is unanswerable. This is not a tooling problem; it is a bookkeeping problem, and the bookkeeping is this chapter.

### The idea

**An AI system's version is a tuple, not a number, and a release pins every element of it.**

Six things: the **base model and its exact version**, the **adapter or fine-tune**, the **prompt set with ids**, the **retrieval index build**, the **regression suite version**, and the **judge model and rubric**. Change any one and you have a new version of the system, whether or not any code changed. Record the tuple with every release, log it on every request, and "what did Tuesday look like?" becomes a lookup instead of an argument.

### The analogy

The canteen's dal was excellent on Tuesday and ordinary on Thursday. The cook is the same person, the recipe card has not changed, and nobody can explain it.

A kitchen that keeps a service log can. It records, every day: which rice supplier the sack came from, which gas cylinder batch, which cook was on the pot, which version of the recipe card was pinned up, and what the tasting panel scored. With that log, Thursday is a two-minute comparison — the toor dal came from a different supplier — instead of a week of guessing.

And the second rule the log makes possible: when Thursday is bad, you can go back to Tuesday's *whole* configuration. Not just the recipe card. The supplier, the cylinder, the card and the cook, together.

**Where the analogy breaks.** A supplier who changes the dal tells you, eventually, because there is a delivery note. Your model provider can change the thing behind a familiar name with no delivery note at all. The kitchen's ingredient can be inspected; yours has to be *tested* against a fixed set, which is why chapter 10's suite is the only ingredient inspection you have.

### Picture it

```
  THE RELEASE TUPLE  (all six, recorded together)
  ┌───────────────────────────────────────────────────┐
  │ base model      provider/model-name@2026-09-15    │
  │ adapter         support-style-v4 (sha, data v3)   │
  │ prompts         prompt-set-v12 (ids 1–7)          │
  │ index           kb-build-2026-11-14               │
  │ eval suite      suite-v9 (142 cases)              │
  │ judge           judge-model@pinned + rubric-v3    │
  └───────────────────────────────────────────────────┘
            │                           │
            ▼                           ▼
  REGISTRY stages              EVERY REQUEST logs
  ┌─────────────────┐          release id · prompt id
  │ staging  v13    │          model version · tokens
  │ production v12  │◀── roll  cost · cache hit · tools
  │ archived v11,10 │    back  · latency · run id
  └─────────────────┘

  rollback = restore the TUPLE, not the code
```

A release pins six artefacts together, the registry keeps previous whole tuples servable so a rollback restores all of them at once, and every request logs which release and which prompt produced it.

### Three things to keep, and the difference between them

**Experiment tracking** is chapter 1's results table, grown up. One row per run: the settings, the data version, the scores, who ran it, when. Its purpose is comparison, it holds hundreds of rows, and most of them will never be served.

**A registry** holds only what is servable, each entry with a **stage** — staging, production, archived — and a pointer to the experiment that produced it. Its purpose is deployment and rollback. A student team's registry can be a table in your own database plus files in object storage; the concept matters, the product does not.

**Lineage** is the links between them: this adapter came from that data version under that configuration, evaluated by that suite. Lineage is what makes a complaint traceable back to a cause, and it is three extra columns rather than a system.

### Prompts are code

A prompt that lives in a text box in someone's dashboard is unversioned, unreviewed and unattributable. Treat prompts exactly like code: in the repository, changed by pull request, reviewed by a person, gated by chapter 10's suite, and — the part teams miss — **carrying an id that is logged on every request**.

That id is what turns a complaint into an investigation. A user forwards a bad answer; the trace tells you prompt 4 at version 12 with model version X produced it; you can reproduce it exactly. Without the id, you are looking at today's prompt and guessing whether it is the one that ran.

### The three drifts

**Input drift.** Your users start asking things your system was never built for — a new category, longer messages, a different language mix. Detect it cheaply: track the distribution of input length, detected language and a small set of topic buckets, weekly. A shifting distribution is an instruction to add eval cases, not an error.

**Model drift.** The provider changes the model behind a name. This is the one that produces the Tuesday problem, and the defences are: pin a dated version wherever the provider offers one, keep a **canary set** of twenty fixed prompts whose outputs you store, and diff them nightly. A behaviour change on the canary set with no change on your side is model drift, and it is the moment to re-run the full suite.

**Judgement drift.** Your own measuring instrument moved, because the judge model updated or the rubric was edited. Pin the judge, version the rubric, and re-run M10 chapter 12's twenty-label calibration when either changes. Scores either side of an unrecorded judge change are not comparable, and a team that does not know this will chase a phantom regression for a week.

### Where it lives

M06 chapter 12 runs the gate. M13 chapter 5 owns release engineering generally — shadow deployments, canaries, staged rollout — and the only addition here is that an AI rollback must restore the whole tuple, because reverting code while the index and prompts stay new restores nothing. M07 chapters 5 and 6 own logs, traces and percentiles; LLMOps adds the span attributes listed in the diagram, and those attributes are what make M09 chapter 14's cost arithmetic computable from real traffic instead of estimated.

Inside this module: chapter 1's run log is your experiment tracking, chapter 4's adapter and chapter 6's quantized build are registry entries, chapter 10's suite is the gate and one element of the tuple, chapter 11's agent configurations are prompts with ids, and chapter 14's gateway is the convenient place to stamp the release id onto every request.

### Words to own

| Term | Plain meaning | Example |
|---|---|---|
| Release tuple | The six pinned artefacts that define the system | Model, adapter, prompts… |
| Experiment tracking | One row per training or eval run | Hundreds of rows |
| Registry | What is servable, with a stage | staging / production |
| Stage | Which role an entry currently plays | production |
| Lineage | Which data and config produced an artefact | adapter v4 ← data v3 |
| Prompt id | A logged identifier for the exact prompt used | prompt-4@v12 |
| Pinned version | A dated model version rather than a moving name | model@2026-09-15 |
| Canary set | Fixed prompts whose outputs you store and diff | 20 prompts, nightly |
| Input drift | Users' requests change shape over time | Longer, new language |
| Model drift | The provider changed the model behind a name | No notice given |
| Judgement drift | Your judge or rubric changed | Scores incomparable |
| Rollback | Restoring a previous whole tuple | Not just the code |
| Deprecation diary | Dated reminders for each pinned version's retirement | Per pinned model |

### Tradeoffs and gotchas

**A rollback that restores only code is not a rollback.** The most common incident shape here: the deploy is reverted, the problem persists, and nobody realises the retrieval index is still the new build. Make rollback a single action that restores all six elements, and test it before you need it.

**Pin versions, and put the deprecation in a diary.** A pinned dated model stops drifting underneath you and starts expiring instead, which is a far better problem — you choose when to move. The cost is that you must track the retirement date for every pinned thing, including the base model your chapter 4 adapter depends on.

**Do not log the prompt text on every request; log the id.** Full prompts in logs are expensive, and they carry user data into a system with different access rules. The id plus a versioned prompt file reconstructs it exactly.

**Monitoring errors is not monitoring quality.** An AI feature can be 100% available, zero-error, and giving wrong answers. The nightly online eval from chapter 10 is your quality monitor; uptime is not a proxy for it.

**Drift detection needs a stable instrument.** If your judge changed, you cannot tell drift from measurement error. Pin the judge first, then look for drift — in that order, or you will investigate the wrong thing.

**Keep the dataset version in the tuple, not only in the experiment.** Two adapters trained from different data versions are different artefacts even with identical configurations, and the data version is the field most often lost.

**This is bookkeeping, and bookkeeping decays.** The habit survives only if it is automatic: the release id stamped by the gateway, the prompt id read from the file, the canary diff on a schedule. Anything a person must remember weekly will be forgotten by week three.

### Verify it

The red flag is a system where nobody can say what produced a given answer.

1. **Pick one real bad answer from last week and demand its full tuple.** Model version, adapter, prompt id, index build, suite version. If any field cannot be produced, that is the first thing to fix.
2. **Ask whether model versions are pinned.** A bare provider model name with no date is an unversioned dependency.
3. **Ask what the rollback restores.** If the answer is "the deployment", ask specifically about the index, the prompts and the adapter.
4. **Ask how model drift would be noticed.** The answer must be a fixed canary set diffed on a schedule, not "users would tell us".
5. **Check the judge is pinned and the rubric versioned,** and ask when the calibration was last re-run.
6. **Read one real trace end to end.** It must carry prompt id, model version, token counts, cost, cache hit, tool calls and latency. Any missing attribute is a question you will be unable to answer later.
7. **Ask which artefact's deprecation date is nearest.** A team that cannot answer has an unscheduled outage in its future.

Red flags: "it worked last Tuesday" with no tuple recorded; prompts edited in a dashboard; no prompt id in logs; unpinned model names; a rollback that reverts only code; drift monitoring that watches only errors and latency; an unpinned judge; a dataset version recorded nowhere; and a release process that depends on somebody remembering a step.

## Walkthrough
1. Write down your release tuple as it stands right now, filling in all six fields with real values. Mark every field you cannot determine — those marks are your work list.
2. Move every prompt out of any dashboard or inline string into files in your repository, each with an id and a version. Confirm a prompt change now requires a pull request.
3. Log the prompt id and the model version on every request. Then take one real answer from your logs and reconstruct exactly what produced it.
4. Pin your model to a dated version if the provider offers one. Record the pinned string in the tuple and create a diary entry for its expected retirement.
5. Build the canary set: twenty fixed prompts covering your main categories. Run them, store the outputs, and schedule a nightly diff against the stored copies.
6. Create the smallest possible registry: a table with one row per servable artefact, its stage, its lineage pointers and its storage location. Populate it with your current adapter, quantized build and index.
7. Turn chapter 1's results table into your experiment log, with a row per run and a pointer from any promoted artefact back to its run.
8. Add the LLM span attributes to your tracing from M07 chapter 6: prompt id, model version, input and output tokens, rupees, cache hit, tool calls, latency, release id.
9. Build the input-drift report: weekly distributions of input length, detected language and a handful of topic buckets. Look at last month's and write one sentence on whether anything moved.
10. Make rollback one action that restores all six tuple elements. Then actually roll back to your previous release in a non-production environment and confirm the index and prompts went back too.
11. Finally, run the Tuesday drill: ask a teammate to change one element of the tuple without telling you, then find out which one they changed using only your logs, your canary diff and your registry. Record how long it took.

## Talk to AI
```prompt
CONTEXT:      I am a student on an AI-native engineering program. I direct AI
              and verify its output; I do not write code by hand. I already
              have structured logs, traces with spans, percentile metrics, a
              CI pipeline, and a regression suite wired as a required check.
              Do not re-teach those. My team of three runs [product in two
              sentences] with [number] real users in India. Our AI system
              currently consists of: model [name, and say whether it is
              pinned], adapter [name or none], prompts stored in [where,
              honestly], retrieval index [how it is built and how often],
              regression suite [version or none], judge [model and whether
              pinned]. Last week a user reported worse answers and we could
              not determine what had changed.

GOAL:         A release tuple recorded on every deploy and every request, a
              minimal registry with rollback of the whole tuple, and
              detection for all three kinds of drift.

REQUIREMENTS: 1. Write out my release tuple with all six fields filled from
                 what I told you, and mark every field that is currently
                 unknowable. Tell me which single missing field costs me most
              2. Tell me how to move my prompts into version control with ids,
                 and exactly what to log per request so I can reconstruct any
                 answer
              3. Tell me whether my model can be pinned to a dated version,
                 what to do if it cannot, and what diary entries I need for
                 deprecation
              4. Design my canary set: 20 fixed prompts for MY product's
                 categories, how to store their outputs, how to diff them
                 nightly, and what a real drift signal looks like versus noise
              5. Give me the smallest possible registry as a table definition:
                 columns, stages, and lineage pointers back to the run that
                 produced each artefact
              6. List the span attributes my traces must carry for an AI
                 request, and tell me which of them lets me compute cost per
                 request from real traffic rather than estimating it
              7. Design an input-drift report I can run weekly using only
                 data I already log, and tell me what change in it should make
                 me add eval cases
              8. Tell me why my historical eval scores may not be comparable,
                 and what to pin and version so they become comparable
              9. Make rollback one action that restores all six tuple
                 elements. Tell me how to test it, and what goes wrong if only
                 the deployment is reverted
             10. Tell me what NOT to log, and why logging full prompt text on
                 every request is a mistake

CONSTRAINTS:  Free tier only, using the database and object storage I already
              have. Plain English for a second-language reader. Label every
              command READ-ONLY or CHANGES-THINGS. Do not propose a paid
              MLOps or experiment-tracking platform. Anything that depends on
              a person remembering a weekly step will be forgotten, so prefer
              automatic. No user personal data in logs.

NON-GOALS:    Do not re-teach tracing, percentiles or CI. Do not propose
              Kubernetes or a feature store. Do not design a model-training
              pipeline. Do not recommend logging every prompt in full.

ACCEPTANCE:   - A six-field release tuple is recorded for my current system,
                with unknowable fields marked
              - Prompts live in version control with ids logged per request
              - I reconstructed one real past answer from logs alone
              - My model version is pinned, or I know why it cannot be, and a
                deprecation diary entry exists
              - A 20-prompt canary set is stored and diffed on a schedule
              - A registry table exists with stages and lineage pointers
              - My traces carry prompt id, model version, tokens, rupees,
                cache hit, tool calls and latency
              - Rollback is one action and I have tested it restoring the
                index and prompts, not only the code
              - My judge is pinned and my rubric versioned

ASK:          First, fill in my release tuple from what I told you and tell me
              which fields are currently unknowable. Then tell me, for last
              week's complaint, which of the four likely causes I could have
              distinguished with the logs I already have. Answer both before
              proposing anything new.
```

## Stuck? Ask AI
```prompt
A user says our assistant was good on Tuesday and bad on Thursday. Here is what we log today: [paste your log fields]. Tell me, field by field, whether I could determine which of these changed: the system prompt, the retrieval index, the provider's model behind an unpinned name, or my eval judge. For each one I cannot determine, tell me the single field I should start logging and where it comes from.
```

```prompt
Explain the difference between experiment tracking, a model registry and lineage, using my own artefacts: a dataset version, a LoRA adapter, a quantized build and a retrieval index. For each of the three concepts, tell me what it holds, what question it answers, how many rows it has after six months, and the simplest way to implement it with a database table and object storage I already have.
```

```prompt
Design a canary set for drift detection on [describe your AI feature]. Give me 20 fixed prompts covering my real categories, tell me what to store for each one, and tell me how to decide whether a nightly difference is real drift or ordinary non-determinism. Then tell me what I should do the moment the canary set changes with no change on my side, in order.
```

## Quiz

### Q1 · L3
Why is an AI system's version a tuple rather than a number?
- (a) Because semantic versioning does not apply to models
- (b) Because the model, adapter, prompts, index, suite and judge all change independently ✅
- (c) Because providers do not publish version numbers
- (d) Because the code changes more often than the model
> Any one of the six can change with no code change at all, and each changes behaviour. Pinning all six together is what makes "what did Tuesday look like?" answerable.

### Q2 · L4
Your deploy is reverted and the bad answers continue. What was most likely missed?
- (a) The cache was not cleared
- (b) The retrieval index and prompts were not rolled back with the code ✅
- (c) The model needs to be restarted
- (d) The regression suite was out of date
> Reverting code restores one element of six. A rollback has to restore the whole tuple in one action, and the index rebuilt last night is the element teams forget most often.

### Q3 · L4
How should model drift be detected?
- (a) By watching error rates and latency
- (b) By a fixed canary set of prompts whose outputs are stored and diffed on a schedule ✅
- (c) By reading the provider's changelog
- (d) By waiting for users to report it
> A provider can change the model behind a name with no notice, and the change is in quality, not availability. Only a fixed set of inputs with stored outputs can show you that behaviour moved.

### Q4 · L4
Your eval scores from two months ago look much higher than today's. What should you check before investigating a regression?
- (a) Whether the suite has more cases now
- (b) Whether the judge model or rubric changed in between ✅
- (c) Whether the users have changed
- (d) Whether the traces are complete
> An unpinned judge means the measuring instrument moved, and scores either side of that are not comparable. Pin the judge and version the rubric first, then look for real drift.

### Q5 · L5
Why log a prompt id on every request rather than the prompt text?
- (a) Text takes longer to write to the log
- (b) The id plus a versioned prompt file reconstructs it exactly, without cost or user data in logs ✅
- (c) Prompt text cannot be stored in a database
- (d) Ids make the traces easier to sort
> Full prompts are expensive and carry user content into a system with different access rules. An id is cheap, exact and attributable, provided the prompt itself is under version control.

## Go deeper
- [Google: MLOps — continuous delivery and automation pipelines in machine learning](https://cloud.google.com/architecture/mlops-continuous-delivery-and-automation-pipelines-in-machine-learning)
- [OpenTelemetry: semantic conventions for generative AI spans](https://opentelemetry.io/docs/specs/semconv/gen-ai/)
