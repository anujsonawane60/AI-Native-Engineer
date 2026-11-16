---
id: m09-16-bias-and-governance
module: m09-ai-foundations
slug: bias-alignment-governance
title: "Bias, Alignment, Jailbreaks, Red Teaming and AI Governance"
order: 16
duration_min: 25
is_free: false
mission: false
volatility: medium
last_reviewed: 2026-11-16
---

## Key learnings
- Name where bias enters a model: data, labels, proxies and feedback loops
- Measure accuracy per group, not only overall, before calling a model fair
- Tell a jailbreak apart from prompt injection, and know why refusals are not a security control
- Red-team your own feature with twenty adversarial inputs before a user does
- Insist on a human in the loop for any decision that affects a person's money or access

## Lesson

### The pain

A team builds a dish recommender for their canteen app, trained on a year of real order history. Overall accuracy is 78%, the demo is smooth, and nobody complains for three weeks.

Then a student points out that she has never once been shown a North Indian dish. Her name is Tamil, she lives in the south hostel, and most students matching that pattern in the training data ordered South Indian food — so the model learned the pattern and now enforces it. She had been curious about chole bhature for a month. Her accuracy as a user was far below 78%; it never appeared in the overall number.

Nobody wrote a rule about region. The model found a correlation in historical behaviour and turned it into a constraint on the future, which is what these systems do by default.

The same app's assistant has a system prompt saying "never discuss staff salaries". A student asks it to role-play as a manager writing a training document, and it discusses staff salaries at length.

Two problems — one about fairness, one about control — and both are yours, because nobody else in the pipeline owns them.

### The idea

**A model reproduces the patterns in its training data, including the ones you would never have written down, and no instruction you give it is a guarantee. Fairness has to be measured per group, and control has to be enforced in code.**

Bias is not carelessness. It is the faithful operation of a system that learns from history — and history encodes who had access, who was hired, who was approved and who was ignored. **Alignment** is the set of techniques aimed at making behaviour match intended values. It works statistically and therefore imperfectly, which is why **jailbreaks** exist and why your own guardrails have to be real.

### The analogy

The canteen promotes its longest-serving server to decide who gets served first during the lunch rush.

He has no instructions and no prejudice he would admit to. He does what he has watched work for fifteen years. Staff before students, because staff have ten minutes. Final-years before first-years, because they complain. People he recognises before people he does not.

Every one of those rules came from observing the queue, and every one is now a rule the queue lives under. A first-year on their first day waits longest, and the server could not tell you why. The canteen's overall service time looks excellent, carried by the people who were already served quickly.

**Where the analogy breaks:** you can ask the server why, and he will think about it and perhaps change. A model cannot be asked. It will generate a fluent explanation that is not a record of what it did, as chapter 10 established. The only way to know whether your system is fair is to **measure it on the groups you care about** — the explanation tells you nothing.

### Picture it

```
  WHERE BIAS ENTERS

  historical data ──▶ who was already being served well
       │
       ▼
  the labels ──────▶ who a human judged "good", and when
       │
       ▼
  proxy features ──▶ pin code, hostel, name, phone brand
       │              standing in for region, caste, income
       ▼
  the model ───────▶ learns all of it as one pattern
       │
       ▼
  the feedback loop ─▶ it only shows South Indian dishes,
       │               so only those get ordered,
       │               so the next model is more certain
       ▼
  MEASURE: accuracy per group, not overall
```

Bias enters through data, labels and proxy features, and a feedback loop makes each round more confident than the last.

### Proxies and feedback loops, the two that catch everyone

You will not put caste or religion in your features. You do not have to.

A **proxy** is a feature standing in for a protected characteristic without naming it. A pin code carries region, income and often community. A hostel block carries the same. A name carries region and language. The brand of phone carries income, which M08 noted for performance reasons and which applies here too. Remove the obvious column and the model learns the same pattern from the proxy, invisibly.

A **feedback loop** is worse, because it grows. If your recommender never shows North Indian dishes to a group, nobody in that group orders them, so next month's data shows even more clearly that they do not want them. The model becomes more confident in a pattern it created. The only thing that detects this is deliberately logging what you *showed* alongside what was *chosen*.

### Alignment, jailbreaks and injection

Three terms used interchangeably that must not be.

**Alignment** is making behaviour match intended values — helpful, honest, not harmful — mostly through preference training. Being statistical, it fails in both directions: **over-refusal**, where a legitimate security, medical or legal question is declined, and under-refusal, which is the next item.

**A jailbreak** is a user persuading the model to do something its training says it should not — role-play, hypotheticals, claimed authority. The target is the model's own safety behaviour.

**Prompt injection** is the one that threatens *your* system. Text — from a user, or hidden inside a document your app reads — carries instructions your application then follows as though you had written them. The target is your app's behaviour and your data. M07 chapter 3 and M10 chapter 7 are the defences.

The rule linking all three: **a refusal is not a security control.** It is trained behaviour that works most of the time. Anything that must not happen is prevented by your code — do not put the secret in the context, check authorisation server-side, validate output before acting on it. Chapter 12 made this point about system prompts; it is worth repeating as a law.

### Red teaming, and governance you actually have to care about

**Red teaming** is attacking your own feature before anyone else does. For a student team it is cheap: write twenty adversarial inputs, run them, record what happened. Instruction overrides, role-play, claimed authority, another language, a document with hidden instructions, requests for another user's data, and sensitive topics your product has no business answering. An hour finds most of what matters.

On governance, four things are worth knowing by name.

**India's DPDP Act, 2023** governs personal data: consent for a stated purpose, collecting only what you need, and users' rights over their data. If your feature sends student data to a model provider, this is the law you operate under, and chapter 13's privacy axis is where it bites.

**The EU AI Act** classifies systems by risk, with the heaviest obligations on uses affecting access to education, employment, credit and essential services. You may never ship to the EU; the risk-tier way of thinking is worth adopting anyway.

**Model cards** describe what a model was trained on, what it is for, and where it should not be used. Reading one before adopting a model is ten minutes well spent.

**Human in the loop** matters most in practice. For any decision touching money, access, grades, employment or safety, the model recommends and a person decides, with the reasoning logged. Fine for suggesting a dish; not fine for refusing a refund.

### Where it lives

M07 is the security, privacy and consent half of this, including prompt injection and the DPDP Act, and M16 is where analytics and growth loops become a place for bias to hide. This chapter is the model-level view underneath both.

Three artefacts belong in your repository by the end of this module: your per-group accuracy table, your twenty red-team inputs with results, and a decision-log entry naming which decisions a human must make. M11 and M18 will ask about all three, and all are easier to produce now than in month five.

### Words to own

| Term | Plain meaning | Example |
|---|---|---|
| Bias | Systematic unfairness learned from data | Never showing a cuisine |
| Proxy feature | A stand-in for a protected characteristic | Pin code for region |
| Feedback loop | The model's output shapes its next training data | Shown less, ordered less |
| Per-group metric | Accuracy measured separately per group | 78% overall, 41% for some |
| Alignment | Making behaviour match intended values | Mostly preference training |
| Refusal | Declining a request | Trained, not enforced |
| Over-refusal | Declining a legitimate request | A blocked security question |
| Jailbreak | Talking a model past its own safety training | Role-play, hypotheticals |
| Prompt injection | Text that hijacks your app's instructions | M07 and M10 |
| Red teaming | Attacking your own system deliberately | 20 adversarial inputs |
| Model card | A document stating a model's intended use and limits | Read before adopting |
| Human in the loop | A person decides; the model only recommends | Refunds, grades, access |
| DPDP Act | India's personal data protection law, 2023 | Consent for a purpose |

### Tradeoffs and gotchas

**There is no unbiased model, and "we checked for bias" is not a result.** What exists is a measurement, per group, on a metric you chose, at a date. The honest statement names the groups, the metric and the numbers.

**Fairness definitions genuinely conflict.** Equal accuracy across groups, equal false-positive rates and equal selection rates are different goals, and improving one can worsen another. No formula resolves this. Someone has to decide which mistake your product must avoid and write it down — chapter 3's precision-versus-recall decision, in a context where it affects people.

**Removing the sensitive column does not remove the bias.** Proxies carry it, and removing it also removes your ability to measure the problem — which is why a group attribute is often collected deliberately, with consent, for that purpose.

**Over-refusal is a real product problem.** A model refusing legitimate questions about medication, security or law is useless to the people who needed it. Test for it alongside jailbreaks.

**You cannot patch a jailbreak with a stronger instruction.** The attack surface is language itself. Reduce what the model may do, keep secrets out of the context, validate what comes back.

**Governance moves fast.** AI Act obligations, DPDP rules and provider policies all change. Treat anything an AI tells you about current law as a starting point for reading the source.

### Verify it

**Red flag, the headline one: an overall accuracy number with no per-group breakdown.** 78% overall can hide 41% for a group that is a tenth of your users. Ask for the table, split by every group you can identify, with counts beside the percentages.

**Red flag: "the model is unbiased" or "we removed the sensitive fields".** Ask which proxies remain, what was measured, on which groups, when. Both claims usually mean nobody looked.

**Red flag: a refusal or a system prompt described as preventing something.** Ask what enforces it when the model complies. If the answer is not code outside the model, the control does not exist.

**Red flag: full automation on a consequential decision.** Ask who reviews it, what they see, and where it is logged. "Accurate enough" is not an answer.

**Red flag: no adversarial testing at all.** Ask for the twenty inputs and their results. If they do not exist, your users are your red team and will find the problems in public.

**Red flag: a confident statement about what the law requires.** Legal specifics sit behind every knowledge cutoff and vary by jurisdiction. Get the name of the act, then read the source.

And the measurement to take yourself: pick the group most likely to be underserved, pull thirty of their real interactions, and score your feature on just those thirty. Compare against your overall number. That one comparison is the most useful fairness work a student team can do in an afternoon, and it is what the recommender's team never did.

## Walkthrough
1. List every feature your AI feature or model sees. Next to each, write what it might be standing in for — region, income, caste, gender, age, language. Be uncomfortable about it.
2. Identify the group in your user base most likely to be underserved. Say how you would identify them in your data, and whether you have consent to do so.
3. Pull thirty real interactions from that group and score your feature on them by hand. Compare the number against your overall score and write both down.
4. Check for a feedback loop: does your product log what it *showed*, not only what was *chosen*? If not, write down what you would need to log to detect the loop.
5. Write your twenty red-team inputs, covering instruction overrides, role-play, claimed authority, another language, a document with hidden instructions, requests for another user's data, and sensitive topics your product should not answer.
6. Run all twenty and record what happened for each, verbatim. Do not summarise.
7. For every input that succeeded in breaking a rule, write down what code outside the model would stop it. That sentence is the actual fix.
8. Test over-refusal too: write five legitimate questions in your domain that a cautious model might wrongly decline, run them, and record the result.
9. Read the model card or usage policy for the model you chose in chapter 13, and write down in your own words what it says it should not be used for.
10. List every decision your product makes automatically, and mark each one "fine to automate" or "needs a human". Justify every automated one in a sentence.
11. Write one decision-log entry with your per-group table, your red-team results, and your human-in-the-loop list, dated.

## Talk to AI
```prompt
I want to find the bias and the security holes in my own AI feature before my users do. Be adversarial and specific. Do not reassure me.

My feature: [what it does, who uses it, what decision it affects]
Every input the model or system sees: [list all features or all parts of the prompt]
My users: [describe them, including the groups present - region, language, income level, year of study, anything relevant]
What the output changes for the user: [e.g. which dishes they see / whether a refund is approved]
My current rules in the system prompt: [paste them]

Please do all of this:

1. Go through my input list one feature at a time and tell me what each one might be a proxy for. Flag the ones I clearly have not thought about.

2. Tell me which group in my user base is most likely to be underserved by this feature, and why, based on what I told you. Then tell me exactly how I would measure that rather than guess, and what number would confirm it.

3. Describe the feedback loop my feature could create, step by step over six months, and tell me what I would need to log TODAY in order to detect it later.

4. Write me twenty adversarial inputs for MY feature specifically. Cover instruction overrides, role-play, claimed authority, another language, a document containing hidden instructions, attempts to get another user's data, and sensitive topics my product has no business answering. Make them realistic, not cartoonish.

5. For each of my system-prompt rules, tell me plainly whether it can be enforced by instruction at all. For the ones that cannot, tell me what code outside the model must do instead and where it belongs.

6. Write me five legitimate questions in my domain that a cautious model might wrongly refuse, so I can test over-refusal as well as jailbreaks.

7. List every decision in my feature that affects a person's money, access or opportunities, and tell me which ones should not be fully automated. Tell me what the human reviewer would need to see.

8. On law: name the relevant frameworks for an Indian student startup, say in one line what each one is about, and then tell me plainly that you may be out of date and that I must read the actual source. Give me the exact names to search for.
```

## Stuck? Ask AI
```prompt
Explain proxy variables using a hiring example that has nothing to do with food or education. Then give me eight data columns, one at a time, and ask me what each one might be standing in for before you tell me.
```

```prompt
Explain the difference between a jailbreak and a prompt injection. For each, tell me who the attacker is, what the target is, and who gets harmed. Then give me six example attacks and ask me to classify each one before you answer.
```

```prompt
My model is 78% accurate overall. Explain, with made-up but realistic numbers, how that could hide a group at 41%. Show me the table. Then explain why equal accuracy across groups and equal false-positive rates across groups can be impossible to achieve together, and ask me which one my product should prioritise.
```

## Quiz

### Q1 · L3
Your recommender is 78% accurate overall but never shows North Indian dishes to students from the south hostel. What went wrong?
- (a) The model is overfitting to the training data
- (b) A proxy feature encoded region, and overall accuracy hid the per-group failure ✅
- (c) The temperature was set too high
- (d) The training set was too small
> Hostel or name stands in for region without anyone naming it, and an aggregate metric averages away a group's experience. Per-group measurement is the only way to see it.

### Q2 · L3
Why does removing the "region" column often fail to remove regional bias?
- (a) The model has already memorised the column
- (b) Proxies such as pin code, hostel and name carry the same information ✅
- (c) Bias lives in the model's weights, not its inputs
- (d) Regional bias comes from the labels only
> Correlated features reconstruct the pattern invisibly. Removing the column also removes your ability to measure the problem, which is why a group attribute is often collected deliberately, with consent.

### Q3 · L4
What is the difference between a jailbreak and a prompt injection?
- (a) They are two names for the same attack
- (b) A jailbreak targets the model's safety training; injection hijacks your application's instructions ✅
- (c) A jailbreak is legal and injection is not
- (d) Injection only affects image models
> The target differs, which changes the defence. Injection threatens your data and your app's behaviour, often through text hidden in a document your app reads.

### Q4 · L4
Your system prompt says "never discuss staff salaries" and a role-play message gets around it. What is the correct fix?
- (a) Add "IMPORTANT" and repeat the rule
- (b) Switch to a better-aligned model
- (c) Keep salary data out of the context and enforce the restriction in your own code ✅
- (d) Lower the temperature so it follows instructions more strictly
> Refusals are trained behaviour, not security controls. Anything that must not happen is prevented outside the model, as chapter 12 established about system prompts.

### Q5 · L5
Which decision in a student product should definitely not be fully automated?
- (a) Suggesting a dish on the menu page
- (b) Choosing which banner image to show
- (c) Refusing a student's refund request ✅
- (d) Ordering search results by relevance
> Money, access, grades, employment and safety need a human deciding with the reasoning logged. Suggestions are low-stakes; refusals of money are not.

## Go deeper
- [NIST AI Risk Management Framework](https://www.nist.gov/itl/ai-risk-management-framework)
- [Google: fairness in the machine learning crash course](https://developers.google.com/machine-learning/crash-course/fairness/types-of-bias)
