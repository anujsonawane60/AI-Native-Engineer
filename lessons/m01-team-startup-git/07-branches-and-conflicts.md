---
id: m01-07-branches-and-conflicts
module: m01-team-startup-git
slug: branches-and-conflicts
title: "Branches, Commits and Merge Conflicts When Agents Touch Many Files"
order: 7
duration_min: 15
is_free: false
mission: false
volatility: low
last_reviewed: 2026-11-16
---

## Key learnings
- Explain what a branch is and why every change gets its own
- Keep the main branch working at all times
- Read a merge conflict and resolve it deliberately
- Avoid the conflict pattern that agents cause more than humans do
- Recover from a bad merge without losing work

## Lesson

### The pain

Three people, all on the main branch, all using agents. Monday evening: Product's agent restructures the menu component; Tech's agent, working from a morning copy, rewrites the same component to add a sold-out state; Growth's agent inserts an analytics line into every page.

Tuesday morning nobody can push. Main has been broken since Monday, the deployed site is down, and resolving the conflicts means choosing between two versions of a file neither author read. Three people with agents on one branch produces exactly this, every time.

### The idea

**A branch is a parallel copy of the whole project where you can work without affecting anyone else. One change, one branch, merged back when it works.**

Branches cost nothing to create and nothing to throw away. That makes them the right unit of work for agent-driven development, where a change is often large and sometimes wrong.

### The analogy

You want to try new canteen prices. Without branches you write them into the live register, in front of customers, while your colleague writes in it too — and if the scheme is bad you must remember the old prices. With branches you photocopy the register, try the scheme in the copy, then copy the changes back or throw the photocopy away. The detail that matters: while you work in the copy, your colleague keeps writing in the original. That gap is where conflicts come from.

**Where the analogy breaks:** photocopying is slow, a branch is instant, because git stores only the differences. A branch for a ten-minute task is normal.

### Picture it

```
  main     ●───────●───────●───────────────●──────▶  always working
            \               \             /
             \               \           / merge
              \               ●───●───●─┘
               \              sold-out-toggle
                \
                 ●───●  analytics      (still in progress)

  ● = a commit
```

Main always works and always deploys. Each change gets its own branch, merged back when verified; two branches coexist without touching each other. The rule that follows: **main is never broken**, because a broken main stops the deployed site and blocks everyone's new work.

A prefix and a short description is enough for a name — `feat/sold-out-toggle`, `fix/cart-ignores-quantity`, `chore/add-gitignore`. What matters more is that the branch is **small**. A week-old branch has diverged far from main and will conflict badly; one open for an afternoon usually merges cleanly. Agents sharpen this, because they produce a week's changes in an afternoon: diff size causes conflicts, not calendar time.

### Where it lives

A conflict happens when two branches changed **the same lines of the same file** and git cannot tell which version you want. That is all — different files, or different parts of one file, merge silently. When it happens, git puts both versions in the file:

```
<<<<<<< HEAD
      <span className="price">₹{item.price}</span>
=======
      <span className="price">{formatPaise(item.price_paise)}</span>
>>>>>>> feat/sold-out-toggle
```

Between `<<<<<<<` and `=======` is your current branch; between `=======` and `>>>>>>>` is what is coming in. Delete the markers, leave the correct code — either version, a combination, or something new — then `git add` the file and complete the merge.

Branches are what pull requests are built on, next chapter, and what makes CI useful in M06: tests run on the branch before it reaches main. Every branch also gets its own preview URL, which is what makes it genuinely reviewable — a teammate clicks a link and uses the change rather than reading code.

### Words to own

| Term | Plain meaning | Example |
|---|---|---|
| Branch | A parallel line of work | `feat/sold-out-toggle` |
| main | The branch that always works and deploys | Protected |
| Checkout / switch | Move to a different branch | `git switch main` |
| Merge | Bring one branch's changes into another | Branch into main |
| Conflict | Both branches changed the same lines | You must choose |
| `git pull` | Fetch and merge the remote's changes | Before you start |
| Diverged | Two branches have both moved on | The cause of conflicts |
| Rebase | Replay your commits on top of newer main | Tidier history, riskier |
| Stash | Park uncommitted changes temporarily | Switching branches in a hurry |
| Atomic commit | One logical change per commit | Easy to revert |

### Tradeoffs and gotchas

**Agent conflicts are worse than human ones.** A human adding a sold-out toggle edits the menu component and maybe one more file. An agent may also reformat it, rename a variable, update three call sites and tidy an import — mostly reasonable, all of it conflict surface. So the conflict becomes "forty lines look different and I cannot tell which differences were the point." Three defences, all from M00: **scope the brief** ("change only `MenuItem.tsx`, do not reformat, do not rename, do not touch other files"), **keep branches short** — merge the same day — and **`git pull origin main` before you start and before you merge back**.

**Agree who owns which files for the week.** Three agents editing one component is the problem; three agents editing three components is fine. This is the practical reason the chapter 1 role split matters.

**Never commit directly to main once you have teammates.** Main deploys and everyone branches from it, so one broken commit blocks three people.

**Read both sides before resolving, and never let an agent resolve for you.** It will produce something that compiles; whether both intents survived is a different question. Keeping "yours" when you wrote neither side is a coin flip, and resolution is where real bugs get introduced silently.

**A conflict is not an error.** Git found a question only you can answer. Nothing is broken, nothing is lost.

**Prefer merge over rebase while learning; treat `git stash` as an emergency, not storage.** Rebase rewrites commits, which is dangerous on a shared branch. Stashed work is easy to forget — if it matters, commit it on a branch.

### Verify it

Five checks before merging any branch.

1. **Did you pull main first?** If not, you are about to merge a stale branch.
2. **Does the file list match what you asked for?** `git diff main --stat` shows files and line counts; an unexpected file is a scope violation to investigate before merging.
3. **Do you understand every resolution you made?** If you kept one side unread, go back and read it.
4. **Does it work on the branch?** Run it. In M06 CI does this; until then, you do.
5. **Is main still working after the merge?** Check immediately — revert the merge rather than debug under pressure.

Know the recovery before you need it: **`git merge --abort`** during a conflict returns you to before the merge started. After a completed local merge, **`git reset --hard HEAD~1`** discards it — destructive, so be sure your own work is committed first.

## Walkthrough
1. Make sure you are on main with a clean tree: `git switch main`, then `git status`, then `git pull origin main`. This three-command sequence starts every piece of work from now on.
2. Create a branch for one small change: `git switch -c docs/add-scope-notes`. Confirm with `git status` that you are on the new branch.
3. Make a small edit to `SCOPE.md`, commit it with a real message, and push the branch: `git push -u origin docs/add-scope-notes`.
4. Now deliberately create a conflict, so you meet one on your own terms rather than under pressure. Switch back to main and edit **the same line** of `SCOPE.md` differently. Commit it on main.
5. Switch to your branch and run `git merge main`. Git will report a conflict. Read the message — it names the file.
6. Open the file. Find the `<<<<<<<`, `=======` and `>>>>>>>` markers. Identify which side is yours and which is coming from main.
7. Resolve it by deleting the markers and leaving the text you actually want. Combine both if that is the right answer. Save.
8. Stage the resolved file with `git add SCOPE.md` and finish with `git commit`. Run `git log --oneline --graph` and look at the shape — you can see the two lines of work joining.
9. Now practise the abort. Create another conflict the same way, and this time run `git merge --abort` instead of resolving it. Confirm with `git status` that you are back to before the merge, with nothing lost.
10. Finally, do it the way you will do it for real: on a clean branch with one small change, run `git diff main --stat` and read the file list. Confirm it contains only what you intended before you merge anything.
11. Agree with your team, in writing in `TEAM.md`: branch for every change, pull before starting, merge the same day, and who owns which files this week.

## Talk to AI
```prompt
I am working in a team of three where all of us use AI coding agents, and we keep getting merge conflicts. I want to understand the cause and change our process, not just resolve the current conflict.

Our situation:
- Stack and project structure: [describe briefly, including the main folders]
- How we currently work: [e.g. all on main, or branches but long-lived]
- What each of us is working on this week: [three things]
- A recent conflict we had: [describe which file and roughly what both sides changed]

Help me with:
1. Why agent-driven changes cause worse conflicts than hand-written ones, specifically in a project shaped like mine
2. Which files or folders in my project are most likely to become conflict hotspots, and why
3. A file-ownership split across our three roles for this week that would minimise overlap
4. The exact wording I should add to my briefs to stop an agent reformatting, renaming or touching files outside the task
5. A branching rule for our team written as four or five lines we could put in TEAM.md
6. For the specific conflict I described: what questions I should ask myself before choosing a side, rather than which side to choose

Give me plain git commands with no aliases, and tell me which of them are destructive.
```

## Stuck? Ask AI
```prompt
I have a merge conflict and I do not understand what I am looking at. Here is the file content with the markers: [paste it]. Explain which side is which, what each side was trying to do, and what questions I should ask myself before choosing. Do not just tell me which one to keep.
```

```prompt
Explain what a branch is using an analogy that is not about photocopies, parallel universes or timelines. Then explain why creating a branch costs almost nothing, and why that changes how boldly I can experiment.
```

```prompt
I merged a branch into main and now main is broken. Walk me through my options in order, from safest to most destructive, and tell me which one I should choose given that my teammates are also working right now.
```

## Quiz

### Q1 · L3
Three teammates all work directly on main, all using agents. What happens by the second day?
- (a) Nothing, as long as they edit different files
- (b) Conflicts, a broken main, and a blocked team ✅
- (c) Git automatically serialises their changes in order
- (d) The agents coordinate through the repository
> Main is what deploys and what everyone branches from, so one broken commit blocks three people. Agents produce large diffs, which makes the collision near-certain.

### Q2 · L3
What exactly causes a merge conflict?
- (a) Two people working on the same branch at the same time
- (b) Two branches changing the same lines of the same file ✅
- (c) Pushing before pulling
- (d) Having more than two branches open
> Different files, or different parts of the same file, merge silently. Git only stops when it genuinely cannot tell which version of specific lines you want.

### Q3 · L4
Why do agent-driven changes cause worse conflicts than hand-written ones?
- (a) Agents commit more frequently than humans
- (b) Agents often also reformat, rename and update call sites, enlarging the surface area ✅
- (c) Agents cannot read the existing code before editing it
- (d) Agents always work on the main branch
> The conflict becomes "forty lines look different and I cannot tell which differences were the point". Scoping the brief with explicit non-goals is the main defence.

### Q4 · L4
You hit a conflict in a file where neither side was written by you. What is the risk in keeping "your" side?
- (a) Git will reject the merge
- (b) It is a coin flip, and the resolution is where bugs get introduced silently ✅
- (c) The other developer's commit will be deleted from history
- (d) None — your branch is always the more recent version
> Conflict resolution is a decision, not a formality. Code that compiles is not evidence that both intents survived, which is why you read both sides.

### Q5 · L3
You are in the middle of a conflicted merge and want to get back to before you started it. What does that?
- (a) `git reset --hard origin/main`
- (b) `git merge --abort` ✅
- (c) `git stash`
- (d) `git revert HEAD`
> `--abort` returns you to the pre-merge state with nothing lost. Knowing this before you need it is what makes merging feel safe rather than risky.

## Go deeper
- [Pro Git chapter 3 — Branching and Merging](https://git-scm.com/book/en/v2/Git-Branching-Basic-Branching-and-Merging)
- [Learn Git Branching — an interactive visual sandbox](https://learngitbranching.js.org/)
