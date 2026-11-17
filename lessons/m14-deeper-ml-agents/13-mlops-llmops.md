---
id: m14-13-mlops-llmops
module: m14-deeper-ml-agents
slug: mlops-llmops
title: "MLOps and LLMOps: Registry, Versioning, Drift and Tracing"
order: 13
duration_min: 15
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

By now you have a dataset, an adapter, a quantized build, a serving configuration, a regression suite and a judge. Each changes independently, and none lives in the same place.

So: a user complains on Thursday that answers have got worse; on Tuesday they were good. The team finds the prompt was edited on Wednesday, the index rebuilt Tuesday night, the provider shipped an update behind an unchanged model name, and the judge was repointed last week. Four changes, no record of which was in effect when — so "what made it worse?" is unanswerable.

### The idea

**An AI system's version is a tuple, not a number, and a release pins every element of it.**

Six things: the **base model and its exact version**, the **adapter**, the **prompt set with ids**, the **retrieval index build**, the **suite version**, and the **judge model and rubric**. Change any one and the system is a new version, whether or not code changed. Record the tuple with every release and log it on every request.

### The analogy

The canteen's dal was excellent on Tuesday and ordinary on Thursday. Same cook, same recipe card, no explanation. A kitchen with a service log can answer: it records the rice supplier, the cylinder batch, the cook, the card version and the tasting score — and lets you go back to Tuesday's *whole* configuration.

**Where it breaks.** A supplier who changes the dal eventually tells you; your provider can change what sits behind a familiar name with no note. Their ingredient can be inspected, yours must be *tested* against a fixed set.

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

**Experiment tracking** is chapter 1's results table grown up: one row per run with settings, data version, scores and date. **Lineage** — which data and config produced an artefact — is three more columns, not a system.

**A registry** holds only what is servable, each entry with a **stage** — staging, production, archived — and a pointer to its run.

Prompts belong in the repository too, changed by pull request, each **carrying an id logged on every request** — which turns a complaint into a reproducible investigation.

### The three drifts

**Input drift.** Users start asking things the system was never built for. Track input length, language and topic buckets weekly; a shift means adding eval cases, not an error.

**Model drift.** The provider changes the model behind a name — the Tuesday problem. Pin a dated version where offered, keep a **canary set** of twenty fixed prompts whose outputs you store, and diff them nightly. A change there with none on your side means re-running the suite.

**Judgement drift.** Your own instrument moved, because the judge updated or the rubric changed. Scores either side of that are not comparable.

### Where it lives

- M06 chapter 12 — runs the gate
- M13 chapter 5 — release engineering, plus one addition: an AI rollback restores the whole tuple
- M07 chapters 5, 6 — logs, traces and the span attributes above
- This module — chapter 1's run log is your experiment tracking; chapter 14's gateway stamps the release id

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

**A rollback that restores only code is not a rollback.** The deploy is reverted, the problem persists, and the index is still the new build. Make rollback one action over all six elements.

**Pin versions, and diarise the deprecation.** A pinned model stops drifting underneath you and starts expiring instead — a better problem, because you choose when to move.

**Monitoring errors is not monitoring quality:** a feature can be zero-error and wrong.

**Bookkeeping decays:** stamp ids automatically, schedule the canary diff.

### Verify it

The red flag is a system where nobody can say what produced a given answer.

1. **Pick one bad answer and demand its full tuple.** Any missing field is your first job.
2. **Ask whether model versions are pinned.** A bare name is an unversioned dependency.
3. **Ask what the rollback restores:** index, prompts, adapter.
4. **Ask how model drift would be noticed:** a canary set on a schedule, not "users would tell us".
5. **Check the judge is pinned, the rubric versioned.**
6. **Read one trace end to end:** prompt id, model version, tokens, cost.
7. **Ask which deprecation date is nearest.**

Red flags: "it worked last Tuesday" with no tuple; prompts edited in a dashboard; no prompt id in logs; unpinned models; code-only rollback; an unpinned judge; prompt text logged instead of ids.

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
