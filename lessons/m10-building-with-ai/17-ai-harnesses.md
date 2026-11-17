---
id: m10-17-ai-harnesses
module: m10-building-with-ai
slug: ai-harnesses
title: "AI Harnesses: The Loop, Tools, Permissions and the Sandbox"
order: 17
duration_min: 15
is_free: false
mission: false
volatility: high
last_reviewed: 2026-11-16
---

## Key learnings
- Take any harness apart into four parts: loop, tools, permissions, sandbox
- Stop treating a coding agent as a chat box and point it at the repository
- Set an allowlist deliberately so an approval prompt still means something
- Build the cheapest real sandbox: clean git, no secrets, no production reach
- Read the harness's own trace and diff instead of trusting its summary

## Lesson

### The pain

Two students, same agent, same bug — a price showing as ₹0.40 instead of ₹40. The first uses it as a chat box: pastes a function in, reads the reply, pastes code back into the editor, pastes more context when it does not fit, and forty minutes later has a fix they do not understand.

The second opens the agent *in the repository*. It reads the file, greps for where prices are formatted, runs the test suite, fixes the double division, runs it again, and shows a nine-line diff. Eight minutes — because the second student understood they were holding chapter fourteen's loop with file and shell tools pointed at a working directory. A third student, who understood that much, told it to "clean up the repo" and lost a folder of uncommitted drafts.

### The idea

**A harness is the agent loop, packaged: a model, a tool list, a permission layer, and a working directory that bounds what it can reach. Every harness is those same four parts.**

Learn the four and you can pick up any harness in an hour.

### The analogy

The harness is the kitchen, not the cook. A good kitchen has the knives within reach, the gas behind a valve, the storeroom through a door and the cash box in the office. A cook is exactly as safe as the kitchen is arranged, and arranging it is a one-time job.

**Where the analogy breaks:** a kitchen's boundaries are physical, so a cook at the stove cannot reach into the office. A harness's boundaries are *configuration*: invisible, and wider by default than you assume. The wall between kitchen and office is a setting you have not looked at.

### Picture it

```
  ┌──────────────── THE HARNESS ──────────────────────┐
  │                                                   │
  │   LOOP        observe · decide · act · compact    │
  │                           │                       │
  │   TOOLS       read · edit · shell · search ·      │
  │               fetch · sub-agent · MCP servers     │
  │                           │                       │
  │   PERMISSION  silent  │  prompt  │  never         │
  │   LAYER       tests      delete     prod keys     │
  │               read       push       live DB       │
  │                           │                       │
  └───────────────────────────┼───────────────────────┘
                              ▼
  ┌──────────── SANDBOX: the working directory ───────┐
  │  clean git tree · no .env · no prod access        │
  │  optionally: a container, with the network off    │
  └───────────────────────────────────────────────────┘
```

Four layers. The tools are nearly identical everywhere, so safety comes from the bottom two, both yours to set.

| Part | What it is | What to know |
|---|---|---|
| The loop | Chapter fourteen's loop, inside someone else's code | It compacts old turns automatically, dropping a constraint from twenty turns ago |
| The tools | Read, edit, run shell command, search, fetch URL, spawn sub-agent, plus your MCP servers | `run a shell command` is the escape hatch you were told never to build; every harness has one |
| The permissions | Three buckets — silent, prompt, refused — matched on patterns | Auto-approving the safe set is what makes a prompt informative |
| The sandbox | Whatever bounds the blast radius: working directory, git tree, container | Plan mode, which plans without acting, is any harness's most underused button |

### Where it lives

A harness sits between you and the model, holding your credentials — so M00's permissions chapter is its user manual and M01's git discipline is what makes it safe. The sandbox you can have for free, cheapest first:

- **A clean git working tree.** Everything committed means every edit is reversible — one command, and most of the risk is gone.
- **An empty folder.** The working directory bounds the file tools; an agent opened in your home directory can read your whole life.
- **No secrets in the folder.** A real `.env` is readable regardless of permissions; placeholders locally, real values in your deploy platform.
- **No production reach from that terminal.** If you can deploy or reach the live database, so can the agent.
- **A container when you need more** — a filesystem that is not yours and, optionally, no network at all.

### Tools today

The harnesses in common use as this is written are Claude Code, OpenAI's Codex CLI, Cursor, Gemini CLI and the agent modes built into editors. They differ in interface, pricing and extras, not in the four parts. Expect the names and free tiers to have changed by the time you read this; the anatomy will not have.

### Words to own

| Term | Plain meaning | Example |
|---|---|---|
| Harness | The packaged agent loop plus tools and permissions | Your coding agent |
| Working directory | The folder bounding file access | One project, not home |
| Allowlist | Actions that run without prompting | Run tests |
| Deny rule | Actions refused outright | Touch production |
| Sandbox | The boundary on damage | Clean git tree; a container |
| Dev container | A disposable environment for the project | Filesystem that is not yours |
| Auto-compaction | The harness summarising old turns | Can drop your constraints |
| Plan mode | Produces a plan without acting | Underused |
| Session | A resumable conversation and its state | Resume tomorrow |
| Trace | The record of tools, arguments and results | What actually happened |

### Tradeoffs and gotchas

**Defaults are tuned for a good first impression**, not for your repository or your risk. Nothing in a default configuration knows this folder can deploy to production; ten minutes on the permission settings fixes that.

**Bypassing all prompts is only defensible inside a container** — disposable, no secrets, nothing you would miss. Outside one it is the setting behind most of this module's stories.

**Auto-compaction eats your constraints.** You said "prices are in paise" at turn three; by turn forty the harness has summarised it away and the agent divides by a hundred. Durable constraints belong in a rules file, not a conversation.

**A dirty working tree removes your undo**, which is the third student above. Commit first, and `cd` into the project before starting the agent — the most common mistake and the easiest to fix.

**The summary is not the trace.** "I updated the price formatting and ran the tests" is generated text; the trace shows the commands and their exit codes. Spawned sub-agents inherit a tool list too, so check it.

### Verify it

Five checks before you let a harness work, once per project.

- **Is the working tree clean?** If not, commit first.
- **What folder am I in, and what is in it?** List everything including dotfiles, and assume all of it will be read.
- **What is auto-approved, what prompts, what is denied?** Read the settings; do not discover them.
- **Can this terminal reach production?** Cloud logins, database URLs, deploy commands — if yes, move the work or remove the access.
- **What is the worst case if this goes wrong?** If the answer involves real money, real user data or unrecoverable work, change the setup rather than relying on care.


Then during the run, **read the file list before the diff, and the diff before you accept.** An agent asked to fix one calculation that touched eleven files has misunderstood the scope, and the file list says so at a glance.

Red flags: an agent launched in your home directory; a real `.env` in the working folder; uncommitted work at the start of a run; prompts bypassed outside a container; a permission list nobody has read; a diff accepted unopened; a trace never looked at; a production-capable terminal running an unattended agent.

## Walkthrough
1. Pick one project folder. Run `git status` and commit everything until the working tree is clean. Do not continue until it is.
2. List every file including dotfiles. Open anything that might hold a credential and replace real values with placeholders, keeping the real ones in your deploy platform.
3. Check whether this terminal can reach anything live: a cloud login, a production database URL, a deploy command. Write down what you find.
4. Launch your harness from inside that project folder, never from your home directory. Confirm it reports the working directory you expected.
5. Open the permission settings and write down, for each of these, which bucket it is in: read a file, edit a file, run tests, delete a file, `git push`, install a dependency, anything with elevated privileges.
6. Move "run tests" and "run the linter" to the silent bucket, and confirm delete and push are not silent. The goal is that a prompt appearing means something.
7. Ask the agent to do something genuinely small and real — fix one formatting bug — and let it read and search by itself rather than pasting code at it.
8. Before accepting anything, run `git diff --stat` yourself and read the file list. Investigate every file you did not expect.
9. Read the harness's trace for that run. Compare the commands it actually ran with its own summary of what it did, and note any gap.
10. Now test the gate honestly: ask it to delete a file you do not care about. Either you get a prompt, which is good, or you discover you had no gate, which is exactly what you wanted to learn now.
11. Finally, try one run in plan mode: ask for a plan and no action. Read the plan, change one thing, then let it execute. Note how much that changed the outcome.

## Talk to AI
```prompt
Audit my coding-agent setup before I let it work, and tell me what my real exposure is.

WHERE I LAUNCH THE AGENT: [the exact folder, and whether it is my home directory]
WHAT IS IN THAT FOLDER: [list it, including dotfiles, config and any .env]
GIT STATUS: [clean and committed, or uncommitted changes - say which]
WHAT THIS TERMINAL IS LOGGED INTO: [git host, cloud CLI, database tools]
CAN THIS FOLDER REACH PRODUCTION: [yes or no, and how]
MY CURRENT PERMISSION SETTINGS: [paste them, or describe which actions prompt and which do not]
CONNECTED MCP SERVERS: [list them and their tools]

Do all of the following:

1. List everything an agent in this folder could read, including things I probably have not thought of.
2. List everything it could change or destroy that I could NOT easily undo, given my git status.
3. Name every credential reachable from here and the worst use of each.
4. Tell me whether my working directory is the right boundary, and if not, what it should be.
5. Review my permission buckets. Which actions should move to silent so that a prompt still means something, and which must never be silent? Give me the specific list for my setup.
6. Tell me whether anything here justifies a container or a throwaway environment, and what I would gain - including whether turning the network off would remove a real risk.
7. Explain what my harness's automatic context compaction could lose from a long session, and which of my project constraints must therefore live in a file rather than in conversation.
8. Give me the three changes that would most reduce my blast radius, in order of impact, and what each one costs me in convenience.
9. Tell me how to read the trace of a run and what three things in it indicate that something went wrong, even when the summary says it succeeded.
10. Finally: name the single worst thing this setup allows today, and the one change that would make it impossible.
```

## Stuck? Ask AI
```prompt
Explain the difference between using an AI as a chat box and using a harness pointed at a repository. Walk me through what the harness actually does on one small bug fix - which tools it calls, in what order, and what it reads. Then tell me what I gain and what I take on.
```

```prompt
Explain the four parts of any AI harness - the loop, the tools, the permissions, the sandbox - and for each one tell me what I control and what the product decides for me. Then tell me which of the four is where almost all of my actual safety comes from, and why.
```

```prompt
I want to let an agent run unattended for twenty minutes on my project. List everything I should set up first, in order of importance, including what makes mistakes recoverable, what keeps secrets out of reach, and when a container is worth the trouble. Then tell me what I should read afterwards and in what order.
```

## Quiz

### Q1 · L3
What are the four parts of any AI harness?
- (a) Model, prompt, memory, interface
- (b) Loop, tools, permissions, sandbox ✅
- (c) Editor, terminal, browser, git
- (d) Planner, executor, critic, reporter
> Harnesses differ in interface and pricing and share this anatomy. Learning the four transfers between products; learning one product's menus does not.

### Q2 · L4
Every harness ships a "run a shell command" tool, which chapter four called the escape hatch you must never build. Why is that not a contradiction?
- (a) Shell commands are sandboxed by the operating system
- (b) It is a deliberate choice that moves the safety burden onto permissions and the sandbox ✅
- (c) The model cannot construct dangerous commands
- (d) Harnesses filter commands before running them
> An agent that cannot run commands cannot run your tests, so the capability is the point. That is exactly why the permission layer and the working directory carry all the weight.

### Q3 · L4
Why auto-approve reads, searches and test runs rather than prompting on everything?
- (a) It reduces token cost
- (b) Otherwise you stop reading prompts and approve the dangerous one ✅
- (c) Reads cannot fail
- (d) The harness runs faster with fewer prompts
> If everything prompts, prompts carry no information. Making routine reversible actions silent is what makes an unexpected prompt worth stopping for.

### Q4 · L4
You stated at turn three that prices are stored in paise. At turn forty the agent divides by a hundred. What most likely happened?
- (a) The model ignored your instruction
- (b) Automatic context compaction summarised the constraint away ✅
- (c) The permission layer blocked the correct edit
- (d) The working directory changed
> Harnesses compact old turns to fit the window, and constraints stated in conversation are exactly what a summary drops. Durable constraints belong in a rules file.

### Q5 · L5
What is the cheapest sandbox that genuinely reduces risk?
- (a) Setting every permission to prompt
- (b) A clean, fully committed git working tree ✅
- (c) A longer system prompt about being careful
- (d) Using a smaller model for edits
> Git does not restrict the agent at all; it changes the consequence of a mistake from permanent to recoverable, which is what makes routine edits safe to auto-approve.

## Go deeper
- [Claude Code documentation](https://docs.anthropic.com/en/docs/claude-code/overview)
- [Development containers specification](https://containers.dev/)
