---
id: m09-13-choosing-a-model
module: m09-ai-foundations
slug: choosing-a-model
title: "Choosing a Model: Quality, Latency, Context, Privacy and Cost"
order: 13
duration_min: 25
is_free: false
mission: true
volatility: high
last_reviewed: 2026-11-16
---

## Key learnings
- Score three models on your own task with your own examples, not on a benchmark
- Weigh five axes together: quality, latency, context, privacy and cost
- Treat every price, limit and model name as stale until you read the provider's page
- Route easy requests to a cheap model and keep the expensive one for the hard ones
- Catch a benchmark score presented as fitness for your task

## Lesson

### The pain

Three teams choose a model for the same feature and all three choose badly, because nobody tested anything. The first wires in the leaderboard leader to sort tickets into four buckets: it works perfectly, costs forty times what the job needs, and burns the credits in eleven days. The second picks the cheapest model, whose Marathi is poor and which breaks the output format once in six requests. The third asks an AI, which names a model with a price and a context window — two facts out of date, and the model renamed six months ago.

### The idea

**Model choice is a measurement, not an opinion. Build twenty examples of your own task with answers you trust, run three candidate models against them, and score the results yourself.**

It takes ninety minutes, needs no code you cannot ask for, and produces something you can defend at a checkpoint, which "we used the best one" does not.

One rule sits above all of it. **Any price, rate limit, context size or model name quoted by an AI — including by a very good AI — is probably stale. The provider's own pricing and model pages are the only authority.** That is the knowledge-cutoff failure mode landing exactly where it costs money.

### The analogy

The canteen needs a new rice supplier and three are available. Nobody chooses by reading a trade magazine's "supplier of the year" award. The cook orders a sample sack from each, cooks the same dish three times and tastes them side by side — then asks what the sample cannot answer: cost per kilo, delivery by 7 am, a festival week at triple volume. The award-winner might suit a five-star hotel and be wrong for a canteen needing cheap rice at dawn. The magazine measured something real, but not this kitchen.

**Where the analogy breaks:** rice suppliers do not change their product every three months. Model line-ups change constantly, so your comparison has a shelf life of months. Write the date on it and plan to rerun it.

### Picture it

```
  FIVE AXES, weighed together for YOUR task

  QUALITY   ─ scored on 20 of YOUR examples, by you
            ─ NOT a leaderboard rank

  LATENCY   ─ time to first token + total time, measured
            ─ on a 4G phone, not your laptop's wifi

  CONTEXT   ─ does your real prompt fit, with room to answer

  PRIVACY   ─ where the data goes, is it used for training,
            ─ do your users' terms permit it

  COST      ─ per request, from measured tokens and the
            ─ price read TODAY from the provider's own page

           │
           ▼
   a table with 3 rows and a decision you can defend
```

Five axes measured on your own task, producing a comparison table rather than an opinion.

Each axis has one discipline behind it.

- **Quality** is scored on twenty of your own examples with answers you trust — not the examples you tuned the prompt on.
- **Latency** is time to first token and total time, measured on a phone on mobile data. What feels instant on campus wifi can feel broken on 4G.
- **Context** means your actual prompt, not the published maximum: system prompt plus retrieved data plus history plus reserved output.
- **Privacy** is three answerable questions: where the data physically goes, whether it trains their models by default, and whether your users' consent permits sending it. For identifiable student data the DPDP Act is the frame, and a weaker local model can be the only acceptable option.
- **Cost** is per request, from measured token counts, at a price you read today.

### Where it lives

- **Model choice is a decision-log entry**, and the one you will be asked about most: three candidates, scores on your examples, measured latencies, prices with the date you read them, and one sentence of why.
- **Write it to be rerun.** Keep the twenty examples and the scoring sheet in your repository.
- **Deeper evaluation — eval harnesses, model-as-judge, quality over time — is M10 and M14.**

### Words to own

| Term | Plain meaning | Example |
|---|---|---|
| Benchmark | A public test on somebody else's questions | Not your task |
| Eval | Your own test set for your own task | 20 examples you trust |
| Time to first token | Delay before anything appears | Driven by input length |
| Rate limit | How many requests or tokens you may send per minute | Free tiers are tight |
| Context window | Tokens allowed per request | Must fit your real prompt |
| Data retention | Whether the provider keeps your data, and for how long | Read the terms |
| Training opt-out | Whether your data trains their model | Often a setting |
| Routing / cascade | Cheap model first, escalate when needed | The professional answer |
| Local model | Runs on your own machine, no data leaves | Lower quality, full privacy |
| Open weights | Weights you can download and run | You host, you pay compute |
| Deprecation | A model being retired or renamed | Plan for it |

### Tradeoffs and gotchas

- **The small model is usually enough, and routing is the professional answer.** Classification, extraction and tagging are easy for a small model, and **routing** sends everything to the cheap one, escalating only when a check fails.
- **Benchmarks are real measurements of the wrong thing.** They cover average performance on standard tasks, and some questions have leaked into training data. Use leaderboards to pick which three to test, never to decide.
- **Twenty examples is noisy and still far better than none.** A three-out-of-twenty gap may be chance; fifteen is not.
- **Free tiers have rate limits your demo will hit.** Read the per-minute limits and know what your feature does when throttled.
- **Models get deprecated.** Keep the name in configuration and a second candidate already scored.
- **"Cheaper per token" is not "cheaper per request."** A cheap model needing two retries can cost more per completed task, so compare cost per *successful* request.
- **Everything numeric here expires.** The method survives; the numbers do not, so date your table.

### Verify it

- **A benchmark score offered as evidence that a model suits your task** — the headline one. Ask what it measures, and what the model scores on your own twenty examples.
- **A confidently quoted price.** Treat any price from a chat window as a rumour: open the pricing page, read the number, write the date beside it.
- **An invented or renamed model.** If the exact string is not on the provider's model page, do not plan around it.
- **A recommendation with no privacy answer.**
- **A single recommendation with no second option** is an opinion dressed as advice.
- **Latency measured on your laptop and presented as the user's experience.** Measure on a phone, on mobile data.
- **The part nobody can do for you:** score the twenty outputs by hand against the answer you decided before you looked.

## Walkthrough
1. Write down the single AI task in your product that matters most. One task, named precisely, with the shape of its input and output.
2. Collect twenty real inputs for it from your own product or your own users. Include at least four in the language your users actually type, two that are messy or misspelled, and two edge cases you expect to fail.
3. For each of the twenty, write the correct output yourself, before you run anything. This is your answer key and it must exist first.
4. Shortlist three candidate models — typically one frontier, one mid-tier, one small or local. Find each one's exact model string, context window and price on the provider's own page, and write the date you read it.
5. Write one prompt and keep it identical across all three models. Changing the prompt per model means you are comparing prompts, not models.
6. Run all twenty inputs through all three models. Record the output, the input and output token counts, the time to first token and the total time for each run.
7. Score every output against your answer key by hand. Use a simple scale and write it down per row; do not score from memory or from an overall impression.
8. Repeat the latency measurement for your chosen task on a phone on mobile data, not on wifi, and record the numbers separately.
9. Compute cost per request for each model from your measured token counts and today's prices. Then compute cost per *successful* request by dividing by your score rate.
10. Answer the privacy questions for each candidate from its terms: where the data goes, whether it is used for training, whether it can be turned off.
11. Build the final table — three rows, five axes — pick one, write the sentence that justifies it including what you gave up, and design the routing rule: which requests go to the cheap model, what check escalates to the expensive one, and what the fallback is when your first choice is rate-limited or retired.

## Talk to AI
```prompt
I am choosing a model for one specific task and I want a comparison I run myself, not a recommendation. Follow my rules about prices exactly.

My task: [name it precisely - what goes in, what comes out, who uses the result]
Language my users actually type in: [e.g. Marathi, Hindi, English mixed]
Where the result is used: [e.g. parsed as JSON by my code / shown to a student]
Must the same input give the same output? [yes/no]
My real prompt structure: [system prompt + retrieved data + history + message - say which apply and roughly how long each is]
Privacy situation: [whose data this is, and whether I have consent to send it to a third party]
My budget: [e.g. free tiers only, or ₹X per month]
Where my users are: [e.g. Indian students on 4G, budget Android]

Do all of this:

1. Help me design my twenty test examples. Tell me what categories the twenty should cover for MY task specifically, including the edge cases most likely to break it, and how many of each category.

2. Tell me how to write the answer key and why it must exist before I run anything. Give me a simple scoring scale I can apply consistently by hand.

3. Suggest three candidate models to test - roughly one frontier, one mid-tier, one small or local - and explain what each one is in the comparison FOR.

4. CRITICAL: do not give me any price, rate limit or context window number. For each candidate, tell me the exact page I must open to read its price and limits, and tell me plainly that anything you remember about pricing is probably out of date and must not go in my budget.

5. Give me the exact measurements to record per run: which token counts, which timings, and why time to first token and total time are different things.

6. Tell me how to compute cost per SUCCESSFUL request rather than cost per request, and why the difference matters.

7. For privacy, give me the three questions I must answer per provider from their own terms, and tell me which answers would rule a provider out given what I told you about my users' data.

8. Design a routing rule for me: what goes to the cheap model, what check escalates to the expensive one, and what happens when my first choice is rate-limited.

9. Finally, tell me what would make my comparison invalid - the mistakes that would make my own measurements untrustworthy.
```

## Mission

**Objective.** Produce a dated, defensible comparison of three models on your own startup's most important AI task, measured on twenty of your own examples across all five axes, and a routing rule that follows from it.

**Steps.**
1. **Frame** — name the one task. Write what goes in, what comes out, and who uses the result. One task only; comparing three models on three tasks produces nine unusable half-measurements.
2. **Build the answer key** — collect twenty real inputs, including at least four in your users' own language, two messy ones and two edge cases. Write the correct output for each yourself, before running anything. As a group of three, each member writes keys for a different seven so you discover where you disagree about "correct".
3. **Read the real numbers** — for each of three candidates, open the provider's own pricing and model pages. Record the exact model string, the price per million input and output tokens, the context window and the rate limits, each with the date you read it. Do not take any of these from an AI.
4. **One prompt** — write a single prompt and freeze it. The same prompt goes to all three models.
5. **Run** — twenty inputs, three models, sixty runs. Record output, input and output token counts, time to first token, and total time for each.
6. **Score by hand** — grade all sixty outputs against your answer key, row by row, with a scale you fixed in advance. Split the scoring across the three of you and have each person re-score five of someone else's rows to check you agree.
7. **Measure latency where your users are** — rerun your chosen task on a phone on mobile data, not campus wifi. Record the numbers separately and note the difference.
8. **Do the arithmetic** — cost per request from your measured tokens and today's prices, then cost per successful request using your score rate. Show the working.
9. **Answer privacy** — for each candidate, where the data goes, whether it is used for training, whether that can be disabled, and whether your users consented. Rule out anything that fails.
10. **Decide** — build the five-axis table, choose one, and write the sentence that names what you gave up by choosing it.
11. **Design the routing rule** — what the cheap model handles, what check escalates, and the fallback when your first choice is throttled or retired.
12. **Document** — one decision-log entry in your repo with the table, the date, and the rerun trigger.

**Upload.**
- Your comparison table: three models × five axes → screenshot or file
- Your twenty test inputs with your answer key → file
- Your scored sheet, all sixty rows with per-row scores → file or screenshot
- The provider pricing pages you read, with the date visible → screenshots
- Your cost-per-successful-request arithmetic, working shown → text or screenshot
- Latency measured on a phone on mobile data, alongside the wifi number → screenshot
- Your privacy answers per provider, with a link to the terms you read → text
- Your routing rule → text or diagram
- The AI conversation where you designed the test → share link in `ai_chat_url`
- Your decision-log entry → file in your repo
- Reflection, 150 words max → one thing the comparison changed your mind about, and one thing an AI told you that turned out to be stale

**Self-check.** Tick every box before you submit.
- [ ] My twenty examples are real inputs, not invented ones
- [ ] At least four are in the language my users actually type in
- [ ] I wrote the answer key before running any model
- [ ] The identical prompt went to all three models
- [ ] Every price in my table came from the provider's own page, with the date I read it
- [ ] No price, limit or model name in my table came from an AI
- [ ] I scored all sixty outputs by hand, not by impression
- [ ] Someone else in my group re-scored some of my rows and we agreed
- [ ] I measured latency on a phone on mobile data, not only on wifi
- [ ] My cost figure is per successful request, and the arithmetic is shown
- [ ] I answered all three privacy questions for every candidate from their own terms
- [ ] My decision sentence names what I gave up, not only what I gained
- [ ] My routing rule says what happens when my first choice is rate-limited
- [ ] My decision log has a date and a trigger for rerunning the comparison

**Stretch.** Add a fourth candidate that runs entirely on your own machine, with no data leaving it. Score it the same way and write 100 words on what its quality gap would have to shrink to for the privacy advantage to win.

## Stuck? Ask AI
```prompt
I have twenty test examples and three models and I do not know how to score the outputs consistently. Give me a scoring scale with at most four levels, with a clear description of what each level means, for a task I will describe. Then give me five example outputs and ask me to score each one before you tell me how you would have scored it.
```

```prompt
I asked an AI for model prices and I suspect the numbers are out of date. Explain to me exactly why a model cannot know current pricing, what structurally causes that, and give me a short checklist I can run on any number an AI gives me to decide whether it needs verifying at the source.
```

```prompt
Explain the difference between a public benchmark score and a measurement on my own task, using a concrete example where the leaderboard winner would be the wrong choice. Then ask me three questions about my task and tell me which benchmark, if any, resembles it at all.
```

## Quiz

### Q1 · L3
Why is a leaderboard score weak evidence that a model suits your task?
- (a) Leaderboards are usually falsified by vendors
- (b) It measures average performance on someone else's questions, not yours ✅
- (c) Benchmarks only test English
- (d) Scores are only published for expensive models
> Benchmarks are real measurements of a different thing, and some of their questions have leaked into training data. Twenty of your own examples decide; leaderboards only shortlist.

### Q2 · L3
An AI tells you a model's price per million tokens, its context window and its exact name. How should you treat these?
- (a) As reliable, since they are technical specifications
- (b) As stale until read on the provider's own page, with the date noted ✅
- (c) As reliable if the AI is a recent model
- (d) As reliable for the name but not the price
> All three sit behind the model's knowledge cutoff and change constantly. Model names get renamed and retired too, which is why the provider's page is the only authority.

### Q3 · L4
Model A costs a quarter as much per token as model B, but needs two retries and a longer prompt to get a usable answer. What should you compare?
- (a) Price per million input tokens
- (b) Benchmark scores for both
- (c) Cost per successful request, including retries and prompt length ✅
- (d) Parameter counts
> The unit that matters is a completed task, not a token. Retries, longer prompts and failed format checks move the real figure a long way from the headline price.

### Q4 · L4
Your task is classifying support tickets into four buckets and the frontier model performs identically to the small one on your twenty examples. What follows?
- (a) Use the frontier model anyway, for future-proofing
- (b) Use the small model, and route only the uncertain cases to the expensive one ✅
- (c) Collect more examples before deciding anything
- (d) The test was invalid, since bigger models are always better
> This is diminishing returns on your own data. A cascade handles the bulk cheaply and still sends the hard remainder to the better model.

### Q5 · L5
Your feature handles identifiable student data and the best-scoring model's terms say inputs may be used for training. What is the right analysis?
- (a) Use it; quality matters most for user experience
- (b) Use it, but mention it in the privacy policy afterwards
- (c) Rule it out unless training can be disabled and your users' consent genuinely covers it ✅
- (d) Use it only for the four Marathi examples
> Privacy is a constraint, not an axis to trade freely against quality. Where consent does not cover it, a lower-quality local model can be the correct engineering outcome.

## Go deeper
- [Anthropic: models overview and pricing](https://docs.anthropic.com/en/docs/about-claude/models)
- [Hugging Face Open LLM Leaderboard — useful for shortlisting, not deciding](https://huggingface.co/spaces/open-llm-leaderboard/open_llm_leaderboard)
