---
id: m10-17-ai-harnesses
module: m10-building-with-ai
slug: ai-harnesses
title: "AI Harnesses: The Loop, Tools, Permissions and the Sandbox"
order: 17
duration_min: 25
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

Two students, same agent, same bug — a price showing as ₹0.40 instead of ₹40.

The first uses it as a chat box. They paste a function in, read the reply, paste the suggested code back into their editor, find it does not fit, paste more context, and forty minutes later they have a fix they do not understand and two new errors.

The second opens the agent *in the repository*. It reads the file. It greps for where prices are formatted and finds three places. It runs the test suite, sees one failure, fixes the double division, runs the suite again, and shows a nine-line diff across two files. Eight minutes.

Same model. The difference is that the second student understood what they were holding: not a chat window, but the agent loop from chapter fourteen with file and shell tools pointed at a working directory.

And then the third student, who did understand that, told it to "clean up the repo" and lost a folder of uncommitted draft designs — which is M00's opening story, and the reason the second half of this chapter exists.

### The idea

**A harness is the agent loop, packaged: a model, a tool list, a permission layer, and a working directory that bounds what it can reach. Every harness is those same four parts. Learn the four and you can pick up any of them in an hour; learn one product's menus and you have learnt a menu.**

### The analogy

The harness is the kitchen, not the cook.

A good kitchen has the knives in a block within reach, the gas behind a valve, the storeroom through a door, and the cash box in the office. A cook is exactly as safe as the kitchen is arranged, and arranging it is a one-time job that pays out every day afterwards.

**Where the analogy breaks:** a kitchen's boundaries are physical. A cook standing at the stove cannot accidentally reach into the office. A harness's boundaries are *configuration*, they are invisible, and the defaults are almost always wider than you assume. The wall between the kitchen and the office is a setting you have not looked at yet.

### Part one: the loop

Chapter fourteen's loop, running inside someone else's code. The harness decides what to observe, asks the model for the next action, executes it, feeds the result back, and repeats.

Things the harness does for you, which you should know are happening: it manages context, usually by compacting older turns automatically when the window fills — which is chapter five's summarisation, and it can lose the constraint you stated twenty turns ago. It keeps a session you can resume. It may offer a plan mode that produces a plan without acting, which is chapter fifteen's plan-then-execute built in and the most underused button in any harness.

### Part two: the tools

Almost identical across harnesses, because the job is the same. Read a file. Write or edit a file. Run a shell command. Search file contents. List files by pattern. Fetch a URL. Spawn a sub-agent. Plus whatever MCP servers you connected in chapter sixteen.

Look at that list as chapter four taught you to: `run a shell command` is the generic escape hatch you were told never to build, and every harness has it, because that is what makes a coding agent useful. It is not a design flaw; it is a design *decision* that moves the entire safety burden onto parts three and four.

### Part three: the permissions

Three buckets, and the shape is the same everywhere: actions that run silently, actions that prompt you, and actions that are refused outright. Good harnesses let you match on patterns, so `run tests` is allowed while `delete anything` or `git push` is not.

M00 gave you the default split and it still holds. Silent: read, search, run tests, run a linter, start a dev server, edit files that are committed. Prompt: delete, push, install a dependency, anything with elevated privileges, anything touching a database, any deploy. Never: production credentials, live payment keys, the production database.

The part students skip is the first list. If everything prompts, you stop reading prompts, and the one that mattered gets approved along with the other forty. Auto-approving the safe set is not laziness — it is what makes a prompt informative.

### Part four: the sandbox

The sandbox is whatever bounds the blast radius, and you can have a real one for free.

**The cheapest sandbox is a clean git working tree.** Everything committed means every edit is reversible, which is why M01 put version control before any building. This single step removes most of the risk in this chapter and costs one command.

**The second cheapest is an empty folder.** The working directory is the boundary for file tools. An agent opened in your home directory can read your whole life; opened in one project folder it cannot. Where you launch it is a security decision.

**No secrets in the folder.** `.env` with a real key is readable regardless of permissions, as M00 established and chapter seven sharpened. Placeholders locally, real values in your deploy platform.

**No production reach from that terminal.** If you can deploy or connect to the live database from there, so can the agent.

**A container when you need more.** A development container or a throwaway virtual machine gives you a filesystem that is not yours and, optionally, no network — which removes chapter seven's outward channel entirely. This is also the only context in which bypassing all permission prompts is a defensible choice.

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

Four layers. The tools are nearly identical everywhere, so your safety comes entirely from the bottom two, and both are yours to set.

### Tools today

The harnesses in common use as this is written are Claude Code, OpenAI's Codex CLI, Cursor, Gemini CLI and the agent modes built into editors. They differ in interface, in pricing and in extras. They do not differ in the four parts. Expect the names and the free tiers to have changed by the time you read this, and expect the anatomy not to have.

### Where it lives

A harness sits on your machine, between you and the model, holding your credentials. Everything from M00's permissions chapter is the user manual for it, and M01's git discipline is what makes it safe.

M06's work on Linux, processes and containers is what a sandbox actually is. M01's review habit is how you check the result: read the file list first, then the diff, and never approve what you cannot explain. Chapter eighteen compares the harnesses and their configuration surfaces, and chapter nineteen prepares the repository so that the agent can check its own work before you see it.

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

**Defaults are tuned for a good first impression.** Not for your repository, your credentials or your risk. Nothing in a harness's default configuration knows that this folder can deploy to production. Spend ten minutes on the permission settings per project; it is a one-time cost.

**Bypassing all prompts is only defensible inside a container.** The mode that approves everything exists and it is genuinely useful — in a disposable environment with no secrets and nothing you would miss. Outside one, it is the setting behind most of the stories in this module.

**Auto-compaction eats your constraints.** You said "prices are in paise, never rupees" at turn three. At turn forty the harness has summarised that away and the agent helpfully divides by a hundred. Put durable constraints in a rules file, which is chapter eighteen, not in a conversation.

**A dirty working tree removes your undo.** Running an agent with uncommitted work is the story in M00. Commit first. Every time.

**Where you launch it is a security decision.** `cd` into the project before starting the agent, not into your home directory. This is the single most common mistake and the easiest to fix.

**The summary is not the trace.** "I updated the price formatting and ran the tests" is generated text. The trace shows the commands and their real exit codes. Read the trace when something is odd, and read the diff always.

**Sub-agents inside a harness inherit a tool list.** Chapter fifteen's contracts apply: check what a spawned sub-agent is allowed to do, because it is easy to assume it is more restricted than it is.

### Verify it

Five checks before you let a harness work, once per project.

**Is the working tree clean and committed?** If not, do that first and nothing else matters as much.

**What folder am I in, and what is in it?** List everything including dotfiles. Assume everything present will be read.

**What is auto-approved, what prompts, and what is denied?** Open the settings and read them rather than discovering them. Move "run tests" to silent and confirm "delete" and "push" are not.

**Can this terminal reach production?** Cloud logins, database URLs, deploy commands. If yes, either move the work or remove the access for now.

**What is the worst case if this goes completely wrong?** If the honest answer involves real money, real user data or unrecoverable work, change the setup rather than relying on care.

Then during the run: **read the file list before the diff, and the diff before you accept.** An agent asked to fix one calculation that touched eleven files has misunderstood the scope, and the file list tells you that in three seconds. That is M01's review discipline, and with a harness it is the whole of your quality control.

Red flags: an agent launched in your home directory; a real `.env` in the working folder; uncommitted work at the start of a run; all prompts bypassed outside a container; a permission list nobody has read; a diff accepted without being opened; a trace never looked at; and a production-capable terminal running an unattended agent.

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
