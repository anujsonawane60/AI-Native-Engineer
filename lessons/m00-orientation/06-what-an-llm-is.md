---
id: m00-06-what-an-llm-is
module: m00-orientation
slug: what-an-llm-is
title: "What an LLM Is, and What Chat AI, App Builders and Coding Agents Each Do"
order: 6
duration_min: 25
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

A group of three starts building. One member uses a chat assistant and copies answers into files by hand, losing track of which version is current. Another uses an app builder, gets a beautiful landing page in nine minutes, then discovers they cannot add a login system the way the product needs. The third points a coding agent at the repository with no instructions and comes back to forty changed files and a broken build.

All three used "AI". All three used the wrong one. The tools are genuinely different, and nobody told them how.

### The idea

**A large language model predicts what text comes next. Everything else — the chat window, the app builder, the coding agent — is scaffolding built around that one ability.**

The model is the engine. What differs between products is the body wrapped around the engine: what it can see, what it can touch, and how many steps it is allowed to take on its own.

### The analogy

Imagine a cook who has read every cookbook ever written, remembers the patterns in all of them, and can write a plausible recipe for anything you name — but has never actually tasted food, cannot see your kitchen, and does not know what is in your fridge.

That cook is the model. Enormously well-read, confidently fluent, and completely disconnected from your reality.

Now consider three ways to employ that cook:

**Chat AI** is talking to the cook on the phone. You describe your kitchen; they suggest. They never touch anything. Nothing happens unless you do it. Fast, cheap, zero risk, but every step of the actual work is yours.

**An AI app builder** is a franchise kitchen. You say "I want a dosa counter" and a complete, working counter appears, built from a standard template, remarkably fast. It genuinely works. But it is their layout, and when you want something the template does not anticipate, you are negotiating with the franchise rather than building.

**A coding agent** is the cook standing in your actual kitchen with their hands on your equipment. They can open your fridge, read your existing recipes, change them, throw things out and run the stove. Vastly more capable — and now their mistakes happen to your kitchen rather than in a conversation.

**Where the analogy breaks:** a human cook who has never tasted food would say so. The model will not. It produces equally confident output whether it knows or is guessing, and that is the single most important fact about working with it.

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

The same engine in all three. What changes is how much it can see, how much it can touch, and how many steps it takes before checking with you — and risk rises along exactly that axis.

### Where it lives

Notice that the model itself appears nowhere in your architecture diagram from chapter 1. That is deliberate and worth sitting with.

When you *use* AI to build, the model is a tool on your desk — it leaves no trace in the product. When you *put* AI in your product — the "Ask our app" feature you will build in M10 — the model becomes another server your server calls across the network. It is a hop, with a cost per call and a latency of a second or two, exactly as you saw in chapter 2.

Two completely different relationships, often confused. For M00 through M09 you are using AI as a tool. In M10 you start shipping it as a feature.

### What a harness is

This is the word that explains why two products using the *same* model feel so different.

A **harness** is everything around the model: the loop that lets it take another step, the tools it can call, the files it can read, the permissions it must ask for, the sandbox it runs in, and the way it manages a context that is too small for your whole project.

Claude Code, Codex and Cursor can all run the same underlying model and still behave differently, because their harnesses differ. One asks before editing; another edits and shows you the diff. One reads three files; another searches the whole repository first.

The practical consequence: **when something goes wrong, ask whether the model was wrong or the harness was.** "It did not know about my other file" is usually a harness limit, not a model failure — and the fix is to tell it where to look, not to switch models. You will take this apart properly in M10.

### Choosing between the three

| You want to… | Use | Why |
|---|---|---|
| Understand a concept, or be argued with | Chat AI | No risk, fast, and it is the best tutor you have |
| Compare two approaches before committing | Chat AI | You want options, not output |
| Get a landing page or demo live today | App builder | Fastest path from zero to something real |
| Validate an idea this week | App builder | Throwaway by design, and that is fine |
| Add a feature to a project that already exists | Coding agent | It can read what is already there |
| Change something across many files | Coding agent | The only one that can see the whole repository |
| Understand why your own code behaves oddly | Coding agent | It can read the actual code, not your description of it |

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

**The two expensive mistakes.**

The first is **building your real product in an app builder.** Builders are superb for getting to something visible fast, and genuinely bad at the thing you need in week six: fitting a design you have decided on, rather than a template they have. Use them to validate, demo, and prove an idea is worth building. Expect to rebuild.

The second is **pointing an agent at a repository with a vague instruction.** "Make the app better" produces forty changed files and no way to tell what broke. Agents are powerful in proportion to how specific you are. Small scope, one verifiable step, review the diff — which is exactly the loop you learn in chapter 8.

**Capability is not understanding.** A model that writes a working payment integration does not know your refund policy, your regulator, or that your users are students who share one phone. It is fluent, not informed. The gap between those two words is your job.

**"Vibe coding" works until it does not.** Accepting output you do not understand is fast and genuinely fine for a throwaway demo. It fails at the first bug you cannot describe, because you cannot debug a system you cannot explain. This whole program is the alternative: understand the concepts, direct the tool, verify the result.

**This chapter will age fastest.** The three categories are stable; the product names are not. Judge any new tool by the three questions in the diagram — what can it see, what can it touch, how many steps does it take alone — and you will place it correctly without needing anyone to tell you.

### Verify it

A quick triage before you pick a tool, and one after.

Before:
1. **Does something already exist that this must fit into?** If yes, a builder is the wrong choice.
2. **Am I trying to decide, or to produce?** Deciding is chat. Producing is a builder or an agent.
3. **How bad is it if this is wrong?** The more it can touch, the more carefully you scope it.

After, regardless of tool: **ask it to explain its own output back to you.** "Walk me through what this does and why you chose this approach over the alternative." Two things come out of that. You learn the thing. And inconsistent or evasive explanations are a reliable signal that it was guessing — often more reliable than reading the output itself, especially while you are still learning.

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
