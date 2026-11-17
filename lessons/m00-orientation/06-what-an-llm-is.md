---
id: m00-06-what-an-llm-is
module: m00-orientation
slug: what-an-llm-is
title: "What an LLM Is, and What Chat AI, App Builders and Coding Agents Each Do"
order: 6
duration_min: 15
is_free: false
mission: false
volatility: high
last_reviewed: 2026-11-16
---

## Key learnings
- Explain what a large language model does, in one sentence, without jargon
- Tell the difference between chat AI, app builders and coding agents
- Choose the right one of the three for a given task
- Say what a harness is and why it matters more than the model sometimes
- Avoid the two expensive tool-choice mistakes beginners make

## Lesson

### The pain

Three students start building. One copies chat answers into files by hand and loses track of which version is current. Another gets a landing page from an app builder in nine minutes, then cannot add the login the product needs. The third points a coding agent at the repository with no instructions and gets a broken build. All three used "AI"; all three used the wrong one.

### The idea

**A large language model predicts what text comes next. Everything else — the chat window, the app builder, the coding agent — is scaffolding built around that one ability.**

The model is the engine; what differs between products is the body around it.

### The analogy

A cook who has read every cookbook ever written and can write a plausible recipe for anything — but has never tasted food or seen your kitchen. That is the model.

**Chat AI** is the cook on the phone: they suggest, they touch nothing. **An app builder** is a franchise kitchen: say "dosa counter" and a working counter appears from a template — fast, but it is their layout. **A coding agent** is the cook in your kitchen with hands on your equipment, so the mistakes now happen to your kitchen.

**Where the analogy breaks:** a human cook who had never tasted food would say so. The model will not — it is equally confident whether it knows or is guessing.

### Picture it

```
                      ┌─────────────────────┐
                      │       MODEL         │
                      │ predicts next text  │
                      └──────────┬──────────┘
                                 │ wrapped in...
       ┌─────────────────────────┼─────────────────────────┐
       ▼                         ▼                         ▼
┌─────────────┐          ┌──────────────┐          ┌──────────────┐
│   CHAT AI   │          │ APP BUILDER  │          │ CODING AGENT │
├─────────────┤          ├──────────────┤          ├──────────────┤
│ sees: your  │          │ sees: your   │          │ sees: your   │
│   message   │          │   prompt     │          │   whole repo │
│             │          │              │          │              │
│ touches:    │          │ touches: its │          │ touches:     │
│   nothing   │          │   own project│          │   your files │
│             │          │              │          │   + terminal │
│ steps: 1    │          │ steps: many, │          │ steps: many, │
│             │          │  inside its  │          │  across your │
│             │          │  sandbox     │          │  real project│
└─────────────┘          └──────────────┘          └──────────────┘
    thinking                 starting                  building
    partner                  from zero                 on what exists
```

The same engine in all three; what changes is how much it can see, how much it can touch, and how many steps it takes alone — and risk rises along that axis.

### Where it lives

The model appears nowhere in your architecture diagram from chapter 1. Using AI to build leaves no trace in the product; putting AI *in* your product — M10's "Ask our app" feature — makes it another server your server calls, a hop with a cost per call.

Around the model sits the **harness**: the loop that lets it take another step, the tools it can call, the files it can read, the permissions it needs. Two products on the same model behave differently because their harnesses differ, so when something breaks ask which one was wrong: "it did not know about my other file" is a harness limit.

### Words to own

| Term | Plain meaning | Example |
|---|---|---|
| LLM | Large language model; predicts the next piece of text | The engine behind all of these |
| Prompt | What you send to the model | Your question or brief |
| Context | Everything the model can see at once | Your message plus files it has read |
| Token | A chunk of text, roughly a short word | Billing and limits are counted in these |
| Hallucination | Confident output that is simply wrong | An invented function name |
| Harness | The system wrapped around the model | Claude Code, Cursor |
| Agent | AI that takes multiple steps and uses tools | Reads a file, edits it, runs tests |
| Tool call | The model asking the harness to do something | "Read this file" |
| Sandbox | A limited space where it can act safely | The builder's own project |
| Vibe coding | Accepting AI output without understanding it | How most people start, and why they stall |

### Tradeoffs and gotchas

**Do not build your real product in an app builder.** Superb at reaching something visible fast; bad at week six, when you need a design you decided on rather than a template they did. Validate, demo, expect to rebuild.

**Do not point an agent at a repository with a vague instruction.** "Make the app better" produces forty changed files and no way to tell what broke. Small scope, one step, review the diff.

**Capability is not understanding.** A model that writes a working payment integration does not know your refund policy, or that your users share one phone. Fluent, not informed. And "vibe coding" — accepting output you do not understand — is fine for a throwaway demo but fails at the first bug you cannot describe.

**This chapter will age fastest.** The categories are stable; the product names are not. Judge any new tool by the diagram's three questions.

### Verify it

Three questions before picking:

1. **Does something already exist that this must fit into?** If yes, a builder is wrong.
2. **Am I deciding, or producing?** Deciding is chat; producing is a builder or an agent.
3. **How bad is it if this is wrong?** The more it can touch, the tighter you scope it.

| You want to… | Use | Why |
|---|---|---|
| Understand a concept, or be argued with | Chat AI | No risk, fast, and it is the best tutor you have |
| Compare two approaches before committing | Chat AI | You want options, not output |
| Get a landing page or demo live today | App builder | Fastest path from zero to something real |
| Validate an idea this week | App builder | Throwaway by design, and that is fine |
| Add a feature to a project that already exists | Coding agent | It can read what is already there |
| Change something across many files | Coding agent | The only one that can see the whole repository |
| Understand why your own code behaves oddly | Coding agent | It can read the actual code, not your description of it |

Afterwards, **ask it to explain its own output back to you.** An evasive or inconsistent explanation is a reliable signal it was guessing — more reliable than reading the output.

## Walkthrough
1. Pick one small, concrete task — "a page showing today's canteen menu with prices and a photo per item".
2. Give it to a **chat AI** with no further instruction. Notice what you receive: text you must now place somewhere yourself. Nothing has been built. Count the steps still left for you.
3. Give the same task to an **AI app builder**. Notice how long until something is visible in a browser. Then try to change one specific thing the template did not anticipate — move the price above the name, or add a dietary filter. Note how that felt compared to the first nine minutes.
4. Now ask the builder for something structural: "add login so canteen staff see a different page than students." Note whether you get it, and whether you can tell what it did.
5. Compare the three on four axes and write a line for each: time to something visible, how much you understood afterwards, how much you could change, and how much you could explain to someone else.
6. Ask the chat AI to explain the builder's output back to you. This is the move that converts a demo into learning, and it is the habit this program is built on.
7. Write one sentence: for which part of your own product would you use each of the three? Keep it — you will test this judgement in M01 when your group picks what to build.

## Talk to AI
```prompt
I am choosing between three kinds of AI tool for a task, and I want help thinking rather than an answer handed to me.

My task is: [describe it in two or three sentences, including whether anything already exists that this must fit into]

For each of the three options — a chat assistant, an AI app builder, and an AI coding agent — tell me:
1. What I would get in the first 15 minutes
2. What would be easy and what would be painful by day three
3. What I would have to throw away if the product succeeded
4. How much I would understand about my own system afterwards

Then recommend one and name the strongest argument against your own recommendation. Finally, tell me what I would need to know to make this decision myself next time.
```

## Stuck? Ask AI
```prompt
Explain what a large language model does in three sentences, without using the words "neural", "parameter", "transformer" or "token". Then explain it again in one sentence to a ten-year-old.
```

```prompt
I do not understand the difference between a model and a harness. Explain it with an analogy that is not about kitchens or cars, then give me three examples of problems and ask me whether each one is the model's fault or the harness's fault.
```

```prompt
Here is my understanding of the difference between chat AI, app builders and coding agents: [paste your version]. Tell me what I got wrong, what I oversimplified, and what important distinction I missed entirely. Be blunt.
```

## Quiz

### Q1 · L2
What does a large language model fundamentally do?
- (a) Looks up answers in a database of facts
- (b) Predicts what text comes next ✅
- (c) Executes code to work out the correct answer
- (d) Searches the internet and summarises results
> Everything else — chat windows, app builders, agents — is scaffolding around next-text prediction. Some harnesses do add search or code execution, but that is the harness, not the model.

### Q2 · L4
Your group has a working project in a repository and wants to add a dietary-filter feature. Which tool is the right choice?
- (a) An AI app builder, because it is fastest
- (b) A chat assistant, pasting code in by hand
- (c) A coding agent, because it can read the project that already exists ✅
- (d) Any of them; the choice makes no practical difference
> Builders start from zero and cannot fit into an existing codebase. An agent can read what is already there, which is exactly what "add to an existing project" requires.

### Q3 · L3
Two products use the same underlying model but behave noticeably differently. What explains this?
- (a) One is a newer version of the model
- (b) Their harnesses differ — what they can see, touch, and how many steps they take ✅
- (c) One has been fine-tuned on better data
- (d) Randomness in the model's output
> The harness is the loop, tools, file access and permissions around the model. Same engine, different body — which is why "it did not know about my other file" is usually a harness limit.

### Q4 · L4
A team validates an idea in an app builder in two days, users like it, and they plan to run it as a real business for years. What should they expect?
- (a) To continue in the builder indefinitely at no cost
- (b) To hit limits where the template resists what the product needs, and to rebuild ✅
- (c) That the builder will automatically migrate their code out
- (d) That adding authentication will be the easy part
> Builders are excellent for validation and genuinely limited for a product with its own opinions. Planning for the rebuild is sensible; discovering it in week six is not.

### Q5 · L4
Which signal most reliably suggests an AI was guessing rather than drawing on something sound?
- (a) The output is longer than you expected
- (b) It used a term you did not recognise
- (c) Its explanation of its own output is inconsistent or evasive ✅
- (d) It produced the answer very quickly
> Asking it to explain its reasoning back to you exposes guessing more reliably than reading the output does — particularly while you are still learning to judge the output itself.

## Go deeper
- [Anthropic: Building effective agents](https://www.anthropic.com/engineering/building-effective-agents)
- [Introduction to large language models (Google)](https://developers.google.com/machine-learning/resources/intro-llms)
