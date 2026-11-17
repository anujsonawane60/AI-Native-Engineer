---
id: m10-18-harnesses-compared
module: m10-building-with-ai
slug: harnesses-compared
title: "Harnesses Compared, and Rules Files, Skills, Sub-Agents and Hooks"
order: 18
duration_min: 15
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

A three-person team, three different harnesses. Every session all three re-type the same sentences: prices are in paise, not rupees; do not touch the student-facing menu page; run the tests before claiming to be done. Sometimes they forget, and a price displays as ₹0.40.

The linter is worse: they ask every time and it runs maybe half the time, because an instruction in a conversation is a request. Then one of them goes home for Diwali, and the other two discover that everything about how this project is worked on lived in that person's head.

### The idea

**Everything you repeat to an agent belongs in a file. Four kinds do four jobs: rules are always loaded, skills load when relevant, sub-agents are scoped workers, and hooks are deterministic code that runs whether the agent cooperates or not.**

The fourth changes how you build: the first three are instructions, and instructions are requests. A hook is code, and code is a guarantee.

### The analogy

The canteen manages knowledge four ways: **rules on the wall**, short and always visible; **procedure cards in a drawer**, pulled out the day the fryer needs cleaning; **a specialist called in for the sweets**, who never touches the gravies; and **a gas timer that cuts off at ten**, which is not a rule or a request.

**Where the analogy understates the point:** staff can ignore the wall and skip the card. A gas timer cannot be argued with, and that asymmetry is why hooks matter more than the other three combined.

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

Four surfaces, ordered by how certainly they take effect. Anything that must happen is a hook; the rest can be text.

| Surface | When it loads | What it is for |
|---|---|---|
| Rules file | Always | The stack, the exact commands, unguessable conventions, off-limits directories, the definition of done |
| Skill | When its description matches | A procedure used sometimes: a migration, a pull request description |
| Sub-agent | When invoked, with its own prompt and tools | Context isolation and tool scoping — a read-only reviewer cannot change what it reviews |
| Hook | On an event, as code | Making a step unskippable, blocking a command, producing a record |

A rules file is chapter one's system prompt made permanent and shared, so keep it under a page: four thousand words carries chapter one's contradiction problem and chapter five's cost problem on every call. Hooks do three things nothing else can — a post-edit formatter hook runs *every* time, so the linter problem disappears; a pre-tool hook refuses any command containing `--force`; and a hook logging tool calls gives you M07's trace.

### Where it lives

All four surfaces are files in your repository, reviewed in pull requests as M01 taught, so the team's working agreement becomes a reviewable artefact rather than three people's habits. Hooks bridge to chapter nineteen. Compare harnesses on axes, because feature lists change monthly and these do not.

| Axis | The question |
|---|---|
| Where it runs | Terminal, editor, or cloud on a pull request — decides if it can run unattended |
| Permission granularity | Three buckets, or patterns on specific commands? Allowlist versus approval fatigue |
| Configuration surface | Which of the four? With hooks it can enforce; without, only ask |
| MCP support | Can you bring chapter sixteen's servers, or only built-ins? |
| Automatability | A non-interactive mode, so evals run in CI? |
| Committable configuration | Can the setup live in the repo, so teammates behave identically? |
| Cost model | Per token, per seat, or a limited free tier |

### Tools today

As this is written: Claude Code runs in the terminal with the fullest configuration surface — rules, skills, sub-agents, hooks, MCP. Codex CLI is OpenAI's terminal agent, Cursor is editor-first, Gemini CLI has a generous free tier, and several editors ship their own agent modes. All of this will have changed by the time you read it; the axes will not.

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

**A bloated rules file is worse than a short one.** Every line competes for attention and costs tokens on every call, so a line that never changed an outcome should go.

**Skills with vague descriptions never load.** "Database stuff" does not trigger; "use when adding or altering a table" does. The description *is* the trigger.

**Slow hooks hurt, because hooks run often** — format and lint on edit, test on completion. They are also a liability as well as a control: a hook in a repository runs on your teammate's machine, so review changes as seriously as a dependency bump.

**Do not port your setup; port your files.** And configuration is no substitute for the permission layer: "never push to main" is text, while a pre-tool hook plus branch protection is real.

### Verify it

- **The silent-session test, the only one that matters.** Start a new session, say nothing about your conventions, ask for one small change. Did it use paise, avoid the forbidden directory, and run the tests before claiming to be done? What it got right, your files taught it; what it got wrong still lives in your head.
- **Delete a line from your rules file and re-run the test.** No change means it was costing tokens for nothing.
- **Check every hook fires.** Make a trivial edit and confirm the formatter ran; a typo in a matcher fails silently.
- **Try the forbidden thing.** Ask for the command your pre-tool hook should block; if it runs, you have an instruction, not a control.
- **Read your sub-agents' tool lists** and confirm the reviewer really is read-only.
- **Hand the repository to a teammate on a different harness.** What survives is what was written down.

Red flags: the same conventions re-typed every session; a rules file nobody has read, or longer than a page; a skill that has never loaded; a hook that has never fired; a sub-agent with wider tools than you assumed; uncommitted configuration; a critical rule left as text when a hook could enforce it.

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
