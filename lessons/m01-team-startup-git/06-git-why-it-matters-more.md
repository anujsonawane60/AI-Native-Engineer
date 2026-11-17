---
id: m01-06-git-why-it-matters-more
module: m01-team-startup-git
slug: git-why-it-matters-more
title: "Git: Why Version Control Matters More When AI Writes the Code"
order: 6
duration_min: 15
is_free: false
mission: false
volatility: low
last_reviewed: 2026-11-16
---

## Key learnings
- Explain what a commit is and why history is the real product of git
- Say why version control matters more with AI than without it
- Tell the difference between your machine, your repository and the remote
- Use `.gitignore` to keep secrets and clutter out of history
- Recognise the three states a file can be in before it is committed

## Lesson

### The pain

A student asks an agent to fix a bug in the cart. Twenty minutes later the cart is fixed and the login page is broken. They did not read the diff, it touched eleven files, and the working version from an hour ago exists nowhere. They spend the evening asking the agent to undo things, which produces a third state, broken differently.

With git, one command returns the project to exactly how it was an hour ago. Cost: four seconds.

### The idea

**Git records a complete snapshot of your project every time you commit, so any past state is one command away.**

It is not a backup. A backup is one copy of now; git is every version you saved, with a note explaining why. Four things change once an agent is in your project:

- **The diff is your review surface** — without git there is no "what changed".
- **Undo is one command, so experiments are cheap.**
- **Scope violations become visible:** you asked for one calculation, the commit shows eleven files.
- **Agent edits become safe to auto-approve** — the M00 allowlist draws its line there *because* committed files are recoverable.

### The analogy

The canteen's register. Overwriting today's total each evening is a backup: you know today, last Tuesday is gone. A register where each day gets a dated line with a note — "price of oil up, raised samosa to ₹15" — is version control, and in March the answer is in the register rather than in somebody's memory. Now imagine a fast helper who rewrites pages while you are not watching.

**Where the analogy breaks:** a register is a list, git is a tree. You can branch — copy the whole register, try something risky, then merge it back or throw it away with the original untouched. That is the next chapter.

### Picture it

```
  YOUR MACHINE                                      REMOTE
  ┌──────────────────────────────────────┐      ┌─────────────┐
  │                                      │      │  GitHub     │
  │  working directory                   │      │             │
  │  ┌────────────────┐                  │      │  origin     │
  │  │ files you and  │                  │      │             │
  │  │ the agent edit │                  │      │  ┌───────┐  │
  │  └───────┬────────┘                  │      │  │history│  │
  │          │ git add                   │      │  └───────┘  │
  │          ▼                           │      │      ▲      │
  │  ┌────────────────┐                  │      │      │      │
  │  │ staging area   │  what goes in    │      │      │      │
  │  │                │  the next commit │      │      │      │
  │  └───────┬────────┘                  │      │      │      │
  │          │ git commit                │      │      │      │
  │          ▼                           │      │      │      │
  │  ┌────────────────┐                  │      │      │      │
  │  │ local history  │ ─── git push ────────────┼──────┘      │
  │  │ every snapshot │                  │      │             │
  │  └────────────────┘                  │      │             │
  └──────────────────────────────────────┘      └─────────────┘
```

Files move one way: you edit them, stage the ones you want with `git add`, commit those as a snapshot, then `git push` to the remote. Every file is in one of three states, and most confusing git moments come from not knowing which.

| State | Meaning | How it got there |
|---|---|---|
| **Modified** | Changed, but not marked for the next commit | You or the agent edited it |
| **Staged** | Marked to go into the next commit | `git add` |
| **Committed** | Safely in history | `git commit` |

Read `git status` as "what has the agent done that I have not reviewed?" Staging lets you commit *some* changes: agent edited six files, you reviewed four, commit those four.

### Where it lives

Git sits underneath everything from here on: in M06 a push triggers CI and a push to main deploys; in M10 the agent commits and watches tests pass; in M11 your history is checkpoint evidence. `TEAM.md`, `PRD.md` and `SCOPE.md` live there for the same reason as the code.

Some files must stay out of history: secrets, dependency folders, build output. `.gitignore` lists patterns git refuses to track:

```
.env
.env.local
node_modules/
.next/
dist/
.DS_Store
*.log
```

**`.gitignore` protects the file from git, not from an agent** — `.env` stays an ordinary file in the folder, readable by anything working there.

**A secret committed once is in history permanently.** Deleting the file later does not remove it from the earlier commit — history is the point of git. Rotate the key immediately, then clean history. This is why `.gitignore` goes in on the very first commit.

### Words to own

| Term | Plain meaning | Example |
|---|---|---|
| Repository / repo | A project plus its entire history | Your product |
| Commit | A saved snapshot with a message | "Fix cart total" |
| Working directory | The files as they are right now | What you and the agent edit |
| Staging area | What will go into the next commit | After `git add` |
| `git status` | What has changed and what is staged | Your most-used command |
| `git diff` | Exactly what changed, line by line | Your review surface |
| `git log` | The list of commits | History, newest first |
| Remote / origin | The shared copy, usually on GitHub | Where you push |
| Clone | Copy a repo to your machine | Starting work |
| `.gitignore` | Patterns git refuses to track | `.env`, `node_modules/` |
| HEAD | The commit you are currently on | Where "now" points |

### Tradeoffs and gotchas

**A commit message answers "why", because the code already answers "what".**

| Weak | Strong |
|---|---|
| `fix` | `Fix cart total ignoring quantity` |
| `update stuff` | `Recalculate price server-side, never trust client total` |
| `changes from AI` | `Add sold-out toggle for staff; polling, not realtime` |
| `asdf` | `Revert caching on /menu — stale after chef updates` |

**Commit before you let the agent start, small and often.** A clean tree means `git status` afterwards shows exactly the agent's work, there is always a known-good point to return to, and one commit per logical change narrows any break to a few lines.

**Never commit `node_modules/` or build output.** It regenerates from your dependency file and makes every diff unreadable; thousands of changed files in one commit is almost always this.

**Git protects only what you committed and pushed** — it is not a backup of your laptop.

**Do not let an agent commit unreviewed.** A history of commits called "Update files" destroys the review surface you wanted.

### Verify it

Four checks per project, and one you run constantly.

1. **Is `.gitignore` in place with `.env` in it, from the first commit?** `git log --all -- .env` must return nothing; anything it returns is a key to rotate today.
2. **Is the tree clean before you brief an agent?** `git status` should say nothing to commit.
3. **Do your commit messages say why?** Read your last five; three "fix" or "update" means you write for nobody.
4. **Is anything enormous in your history?** Thousands of files means `node_modules/` or build output got in.

The constant one: **`git status` and `git diff` before every commit.** That is the review, and it depends on you actually looking.

## Walkthrough
1. In your project folder, run `git status`. If it says "not a git repository", run `git init`. You now have history, starting from nothing.
2. Before anything else, create `.gitignore` and put `.env` in it, along with `node_modules/` and any build folder your stack uses. This must be the first thing, not an afterthought.
3. Run `git status` again. You should see your files listed as untracked, and `.env` should be absent from the list. If `.env` appears, your `.gitignore` is not working — fix it before continuing.
4. Stage everything with `git add .`, then run `git status` once more. The same files are now listed as staged. You have just watched a file move from modified to staged.
5. Commit with a real message: `git commit -m "Initial commit: team agreement, problem statement, PRD, scope"`. Run `git log` and read your own first entry.
6. Now change something small in `PRD.md`. Run `git diff` and read the output. Lines starting with `-` were removed, lines with `+` were added. This is the review surface you will use for the rest of the program.
7. Stage and commit that change with a message explaining *why* you changed it, not what you changed.
8. Run `git log --oneline` to see both commits on one line each. This is what your teammates and your future self will read.
9. Create the repository on GitHub — empty, no README — then connect it as your remote and push. Confirm in the browser that your files are there and that `.env` is **not**.
10. Finally, prove the undo. Change a file badly, do not commit, then run `git restore <filename>`. The bad change vanishes. That four-second recovery is the thing that makes working with agents safe.
11. Commit your `.gitignore` check into your notes: run `git log --all -- .env` and confirm it returns nothing.

## Talk to AI
```prompt
I am setting up version control for a six-month team project and I want to understand what I am doing rather than copy commands blindly.

My situation:
- Stack: [e.g. Next.js, Supabase, deployed on a free platform]
- Team: three people, all new to git
- Files that exist now: [list them, including any .env or config]
- Already a git repo: [yes/no]

Explain, in plain language and in the order I should do them:
1. What my .gitignore should contain for this specific stack, and why each line is there
2. The exact sequence to get from "a folder of files" to "pushed to a remote", with what each command actually does
3. How I check, afterwards, that no secret made it into history - give me the command and tell me what a bad result looks like
4. What my team's rule should be about when to commit, given that we will all be using AI agents
5. The three mistakes beginners make in their first week with git, and how I would notice each one

For every command you give me, tell me whether it changes anything irreversibly. Do not give me aliases or shortcuts - I want the plain commands so I understand them.
```

## Stuck? Ask AI
```prompt
Explain the difference between the working directory, the staging area and a commit using an analogy that is not about photography or snapshots. Then give me five situations and ask me which state the file is in, without telling me the answers first.
```

```prompt
I ran `git status` and I do not understand the output. Here it is: [paste it]. Explain each section in plain language, and tell me what I should do next and why.
```

```prompt
Explain why a secret committed to git once is still exposed after I delete the file. Then tell me, in order, what I should actually do about it — and what I should do first if I only have two minutes.
```

## Quiz

### Q1 · L3
Why does version control matter *more* when an AI agent is writing the code, rather than merely as much?
- (a) Agents cannot work in folders that are not git repositories
- (b) The diff is how you review work you did not write, and undo makes letting it try cheap ✅
- (c) Git prevents agents from editing files they should not
- (d) Agents automatically write better commit messages than humans
> Without history there is no "what changed", only the current state. Git is also why editing committed files is safe to auto-approve in the M00 permission model.

### Q2 · L3
You asked an agent to fix one calculation. The commit shows eleven changed files. What have you learned, and how quickly?
- (a) Nothing useful until you read all eleven files
- (b) That the brief failed to bound scope — in about three seconds ✅
- (c) That the agent found and fixed ten additional bugs
- (d) That the eleven files all needed to change for the fix to work
> The file count alone reveals a scope violation before you read any code. This is why NON-GOALS exists in a brief and why you look at the diff first.

### Q3 · L4
`.env` is listed in `.gitignore`. What does that protect it from?
- (a) Being read by an agent working in that folder
- (b) Being committed to git — but not from being read ✅
- (c) Both being committed and being read by an agent
- (d) Appearing in the deployed build output
> It keeps the file out of version control. It remains an ordinary file in the directory, readable by anything with access — the same point as in M00.

### Q4 · L3
Which commit message is genuinely useful in month four?
- (a) `fix`
- (b) `changes from AI`
- (c) `Recalculate price server-side, never trust client total` ✅
- (d) `updated several files`
> The code already shows what changed; the message must supply why. In M18's defence, the history is evidence you understood your product rather than accepted it.

### Q5 · L4
What is the most useful habit to adopt before giving an agent a new task?
- (a) Push everything to the remote
- (b) Commit your work so the tree is clean and the agent's changes are isolated ✅
- (c) Create a `.gitignore` entry for the files it will touch
- (d) Delete the previous commit so history stays tidy
> A clean tree means `git status` afterwards shows exactly the agent's work, with none of yours mixed in — and there is always a known-good point to return to.

## Go deeper
- [Pro Git, chapters 1–2 (free online)](https://git-scm.com/book/en/v2)
- [Oh Shit, Git!?! — plain-language recovery for common mistakes](https://ohshitgit.com/)
