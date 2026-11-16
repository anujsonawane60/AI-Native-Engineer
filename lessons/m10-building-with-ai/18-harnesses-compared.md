---
id: m10-18-harnesses-compared
module: m10-building-with-ai
slug: harnesses-compared
title: "Harnesses Compared, and Rules Files, Skills, Sub-Agents and Hooks"
order: 18
duration_min: 25
is_free: false
mission: false
volatility: high
last_reviewed: 2026-11-16
---

## Key learnings
- Move everything you repeat to an agent out of chat and into a file
- Choose between a rules file, a skill, a sub-agent and a hook correctly
- Use hooks for anything that must happen whether the agent cooperates or not
- Compare harnesses on axes that matter, not on feature lists
- Prove your configuration works by starting a session that says nothing

## Lesson

### The pain

A three-person team, three different harnesses. Every session, all three of them re-type the same three sentences: prices are stored in paise and never rupees, do not touch the student-facing menu page, run the test suite before claiming anything is done.

Sometimes they forget. The agent then does the reasonable thing with the information it has, and a price displays as ₹0.40.

The linter is worse. They ask for it every single time. It gets run maybe half the time, because an instruction in a conversation is a request and the agent has forty other things in its context by step twelve.

Then one of them goes home for Diwali, and the other two discover that everything about how this project is worked on lived in that person's head and that person's chat history. Nothing was written down anywhere a new teammate — or a new session — could find it.

### The idea

**Everything you repeat to an agent belongs in a file. Four kinds of file do four different jobs: rules are always loaded, skills load when relevant, sub-agents are scoped workers, and hooks are deterministic code that runs whether the agent cooperates or not.**

The fourth one is the one that changes how you build. The first three are instructions, and instructions are requests. A hook is code, and code is a guarantee.

### The analogy

The canteen manages its knowledge four ways.

**Rules on the wall.** Short, always visible, everyone has read them. Wash hands. Prices as marked. No credit.

**Procedure cards in a drawer.** Three pages on how to clean the deep fryer, pulled out on the day it needs cleaning. Nobody reads them daily, and nobody wants them on the wall.

**A specialist called in for the sweets.** Comes for one job, with their own equipment, does not touch the gravies.

**A gas timer that cuts off at ten.** Not a rule, not a request. It happens at ten whether anybody remembers, agrees or is even present.

**Where the analogy understates the point:** staff can ignore the wall and skip the procedure card, and sometimes do. A gas timer cannot be argued with. That asymmetry is the whole reason hooks matter more than the other three combined for anything that must not be skipped.

### Rules file: the persistent system prompt

A file in your repository that the harness loads into every session. Chapter one's system prompt, made permanent and shared with your team.

What belongs in it: the stack and versions; the exact commands for install, test, lint and run; conventions that are not guessable, like paise rather than rupees; the off-limits directories; and the definition of done — "run the test suite and the linter before reporting completion".

What does not belong: everything. A four-thousand-word rules file has chapter one's contradiction problem and chapter five's cost problem at once, on every call. Keep it short enough that every line is tested and true. Most good ones are under a page.

Two tiers are usually available: a **project** file committed to the repository, which is the team's shared agreement, and a **personal** file that is not, for your own preferences. Your taste for verbose explanations is not your team's convention.

This is where M01's PRD lands. M00 promised the brief's CONTEXT and CONSTRAINTS would become a rules file so you stop retyping them; this is that file.

### Skills: loaded when relevant

A named bundle of instructions with a short description. The description is always visible; the body loads only when the task matches it. Progressive disclosure: the agent always sees "migrations — how this project changes the schema", and reads the three hundred words only when a migration is happening.

This is what keeps your rules file small. Anything that is a *procedure used sometimes* — how you write a pull request description, how you run a migration, how you ingest a new document type from chapter nine — is a skill rather than a rule.

The failure mode is a vague description. If it does not say clearly when to use this, it never loads, and you will conclude that skills do not work.

### Sub-agents: chapter fifteen as configuration

A named worker defined in a file: its own prompt, its own tool list, sometimes its own model. You are buying the same two things as in chapter fifteen — context isolation and tool scoping.

The practical win is a reviewer sub-agent with read-only tools. It cannot change what it reviews, by construction, which beats asking it not to. Check the tool list you gave it; a sub-agent is easy to assume is more restricted than it is.

### Hooks: the guarantee

Code that fires on an event. The common ones: before a tool runs, after a file is edited, when a session ends, when the agent thinks it has finished.

Three things hooks do that nothing else can.

**They make a step unskippable.** A post-edit hook running the formatter runs every time. The linter problem from the opening story disappears permanently, with no instruction involved.

**They can block.** A pre-tool hook can inspect a proposed command and refuse it — "no command containing `--force`", "no writes outside `src/`". That is chapter seven's structural defence, implemented at the precision you need.

**They produce a record.** A hook appending every tool call to a log gives you the trace M07 wants, independent of the harness.

Hooks are also the mechanism behind chapter nineteen. "The agent runs the test suite and must not report success unless it passes" is a hook, not a hope.

### Picture it

```
                      when it loads      can the agent
                                         skip it?
  ┌──────────────┐
  │ RULES FILE   │   every session       yes - it is text
  │ short, shared│                       it may deprioritise
  ├──────────────┤
  │ SKILL        │   when the task       yes - and it may
  │ a procedure  │   matches its         never load if the
  │              │   description         description is vague
  ├──────────────┤
  │ SUB-AGENT    │   when invoked        partly - its TOOL
  │ scoped worker│                       LIST cannot be
  │              │                       exceeded
  ├──────────────┤
  │ HOOK         │   on an event         NO. it is code.
  │ code         │   always              it can also BLOCK
  └──────────────┘                       the agent's action
```

Four surfaces, ordered by how certainly they take effect. Choose by that column: anything that must happen is a hook, and anything else can be text.

### Comparing harnesses

Compare on axes, not features, because feature lists change monthly and the axes do not.

**Where it runs** — terminal, inside an editor, or in the cloud on a pull request. This decides whether it can run unattended.

**Permission granularity** — three buckets, or pattern matching on specific commands? The difference between a usable allowlist and approval fatigue.

**Configuration surface** — which of the four does it have? A harness with hooks can enforce; one without can only ask.

**MCP support** — can you bring chapter sixteen's servers, or only built-ins?

**Automatability** — is there a non-interactive mode for CI? That is how chapter twelve's eval set and chapter nineteen's verification run without a human.

**Committable configuration** — can the whole setup live in the repository, so three teammates get identical behaviour? If not, you are back to knowledge in heads.

**Cost model** — per token, per seat, or a free tier with limits.

### Tools today

As this is written: Claude Code runs in the terminal and has the fullest configuration surface — rules, skills, sub-agents, hooks, MCP. Codex CLI is the terminal agent from OpenAI. Cursor is editor-first with strong inline editing and rules support. Gemini CLI is a terminal agent with a generous free tier. Several editors ship their own agent modes. All of these will have changed by the time you read this; the axes above will not.

### Where it lives

All four surfaces are files in your repository, reviewed in pull requests exactly as M01 taught. That is the point: your team's working agreement becomes a reviewable artefact rather than three people's habits.

Hooks are the bridge to chapter nineteen, which builds the repository an agent can verify itself in. M06's CI is where the non-interactive mode runs.

### Words to own

| Term | Plain meaning | Example |
|---|---|---|
| Rules file | Instructions loaded every session | Stack, commands, conventions |
| Project rules | Committed, shared by the team | In the repository |
| Personal rules | Yours, not committed | Your preferred verbosity |
| Skill | A procedure loaded when relevant | How to run a migration |
| Progressive disclosure | Description always visible, body on demand | Keeps rules small |
| Sub-agent config | A named worker with scoped tools | Read-only reviewer |
| Hook | Code fired on an event | Format after every edit |
| Pre-tool hook | Runs before a tool, and may refuse it | Block `--force` |
| Non-interactive mode | Running the agent with no human present | In CI |
| Committable config | Setup that lives in the repository | Same behaviour for all three |

### Tradeoffs and gotchas

**A bloated rules file is worse than a short one.** Every line competes for attention and costs tokens on every call. If a line has never changed an outcome, delete it — measure by deleting, as chapter one taught.

**Skills with vague descriptions never load.** "Database stuff" does not trigger. "Use when adding or altering a table, or writing a migration" does. The description is the trigger, so write it as a condition.

**Slow hooks hurt, because hooks run often.** A post-edit hook running your whole test suite on every save makes the agent unusable. Format and lint on edit; test on completion.

**Hooks are code, which makes them a liability as well as a control.** A hook in a repository runs on your teammate's machine when they open the project. Review hook changes as seriously as a dependency change.

**Do not port your setup; port your files.** Switching harnesses is cheap if your knowledge is in rules, skills and hooks, and expensive if it is in a settings panel and your memory.

**Configuration is not a substitute for the permission layer.** A rules file saying "never push to main" is text. A pre-tool hook that blocks it, plus branch protection from M01, is the real thing.

### Verify it

**The silent-session test, which is the only one that matters.** Start a brand-new session. Say nothing about your conventions. Ask for one small real change. Then check: did it use paise, did it stay out of the forbidden directory, did it run the tests and the linter before claiming to be done? Whatever it got right, your files taught it. Whatever it got wrong is still living in your head, and you have just found your next rules-file line.

**Delete a line from your rules file and re-run the test.** If nothing changes, that line was costing you tokens on every call for nothing.

**Check every hook actually fires.** Make a trivial edit and confirm the formatter ran. A hook with a typo in its matcher fails silently, and you will trust a guarantee you do not have.

**Try to do the forbidden thing.** Ask the agent to run the command your pre-tool hook should block. If it runs, you have an instruction, not a control.

**Read your sub-agents' tool lists** and confirm the read-only reviewer is genuinely read-only.

**Hand your repository to a teammate on a different harness.** What survives the switch is what was written down.

Red flags: the same three conventions re-typed every session; a rules file nobody has read in two months; a rules file longer than a page; a skill that has never loaded; a hook that has never fired; a sub-agent with wider tools than you assumed; configuration that is not committed; and a critical rule that exists only as text when a hook could enforce it.

## Walkthrough
1. Open a chat history from the last week and list every instruction you typed more than once. That list is your configuration backlog, in priority order.
2. Sort each item into one of four buckets: always true (rules), a sometimes-used procedure (skill), a scoped job (sub-agent), or must-happen-regardless (hook).
3. Write the project rules file. Keep it under a page: stack, the exact install, test, lint and run commands, the non-guessable conventions, the off-limits directories, and the definition of done.
4. Commit it and open it as a pull request so your teammates review the team's working agreement as a change, per M01.
5. Take your longest procedure out of the rules file and make it a skill. Write the description as a condition — "use when…" — rather than a topic.
6. Define one sub-agent: a code reviewer with read-only tools. Confirm by reading its tool list that it genuinely cannot edit.
7. Add a post-edit hook that runs your formatter. Make a trivial edit and confirm from the output that it fired.
8. Add a pre-tool hook that blocks one specific dangerous command pattern — a force push, or any write outside your source directory.
9. Test the block: deliberately ask the agent to run the blocked command. Confirm it is refused rather than merely discouraged.
10. Run the silent-session test. New session, no conventions mentioned, one small real change. Record everything it got right and everything it got wrong.
11. Turn each thing it got wrong into one new line in the rules file, one new skill, or one new hook — choosing by the certainty column — and run the silent-session test again.

## Talk to AI
```prompt
Help me move my project knowledge out of my chat history and into committed files, choosing the right surface for each item.

MY PROJECT: [stack, what it does, how it is run and tested]
THINGS I RETYPE TO AN AGENT EVERY SESSION: [list them all, even the small ones]
THINGS THAT HAVE GONE WRONG BECAUSE THE AGENT DID NOT KNOW SOMETHING: [list them]
THINGS THE AGENT KEEPS FORGETTING TO DO: [list them]
MY HARNESS: [name it, and say which of rules, skills, sub-agents and hooks it supports]
MY TEAM: [how many people, and whether they use the same harness]

Do all of the following:

1. Sort every item I listed into one of four surfaces - rules file, skill, sub-agent, hook - and justify each choice by how certainly it must take effect. Anything that must happen regardless of cooperation goes to a hook; say so explicitly.
2. Write my project rules file, under one page, with the exact sections: stack, commands, non-guessable conventions, off-limits paths, definition of done. Tell me what you deliberately left out and why.
3. Tell me which of my items belong in a PERSONAL rules file rather than the committed one, and why that distinction matters to my team.
4. Write one skill in full, including the description written as a trigger condition rather than a topic. Explain how I would know it is loading.
5. Define one read-only reviewer sub-agent: prompt, exact tool list, and what it must not be able to do.
6. Write the hooks: a post-edit hook for formatting, and a pre-tool hook that blocks the single most dangerous command pattern in my project. Give me the matcher and explain what it does and does not catch.
7. Tell me which hooks would be too slow to run on every edit, and what should run on completion instead.
8. Design the silent-session test for my project: the exact small change to ask for, and the five things I should check in the result.
9. Tell me what breaks if a teammate on a DIFFERENT harness opens this repository, and what I should write down so that it does not matter.
10. Finally: name the one rule I am relying on as text that should be a hook, and what it would cost me to make it one.
```

## Stuck? Ask AI
```prompt
Explain the difference between a rules file, a skill, a sub-agent and a hook, with one example of each from a student project. For each, tell me when it loads and whether the agent can skip it. Then tell me which one I should use for something that must never be skipped, and why the others cannot guarantee it.
```

```prompt
My rules file is 3,000 words and the agent still gets things wrong. Explain why a longer rules file can make things worse rather than better. Then tell me how to work out which lines are doing nothing, and what should move out of the rules file and into a skill or a hook instead.
```

```prompt
I keep asking my agent to run the linter and it does it about half the time. Explain why an instruction in a conversation is unreliable for this, and show me how a hook makes it certain instead. Then tell me what else in my workflow is secretly relying on the agent remembering something.
```

## Quiz

### Q1 · L3
The agent keeps forgetting to run the linter despite being asked every session. What is the right fix?
- (a) Put it in capitals in the rules file
- (b) A hook that runs the formatter and linter on an event ✅
- (c) A skill describing how to lint
- (d) A sub-agent whose job is linting
> Rules and skills are text, which is a request. A hook is code attached to an event, so it happens whether the agent cooperates, remembers or agrees.

### Q2 · L3
Which belongs in a skill rather than the rules file?
- (a) Prices are stored in integer paise
- (b) The command that runs the test suite
- (c) The three-hundred-word procedure for a database migration ✅
- (d) The directory the agent must never modify
> Rules are always loaded and should stay under a page. A procedure used occasionally has its description always visible and its body loaded only when a migration is happening.

### Q3 · L4
Why is a read-only reviewer sub-agent a stronger guarantee than instructing an agent not to edit while reviewing?
- (a) Sub-agents use a cheaper model
- (b) Its tool list cannot be exceeded, so editing is not available to it ✅
- (c) Sub-agents run in a separate container
- (d) Review output is validated against a schema
> Capability beats instruction. A worker with no write tool cannot write, which needs no cooperation, whereas "please do not edit" competes with everything else in context.

### Q4 · L4
Why review a change to a hook in a pull request with the same care as a dependency change?
- (a) Hooks consume tokens on every call
- (b) A hook in the repository runs as code on every teammate's machine ✅
- (c) Hooks override the harness's permission settings
- (d) Hooks cannot be disabled once committed
> Committable configuration is a strength and a supply-chain surface. Code that executes automatically when someone opens the project deserves real review.

### Q5 · L5
Which test best proves your configuration files are doing their job?
- (a) Reading the rules file aloud to check it is accurate
- (b) A brand-new session, no conventions mentioned, one small real change ✅
- (c) Counting how many lines the rules file has
- (d) Asking the agent to summarise its rules
> Whatever it gets right in a silent session was taught by your files. Whatever it gets wrong is still only in your head, and that gap is your next rule, skill or hook.

## Go deeper
- [Claude Code settings, hooks and sub-agents](https://docs.anthropic.com/en/docs/claude-code/settings)
- [Agent rules files: the AGENTS.md convention](https://agents.md/)
