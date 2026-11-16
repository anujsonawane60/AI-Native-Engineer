---
id: m01-07-branches-and-conflicts
module: m01-team-startup-git
slug: branches-and-conflicts
title: "Branches, Commits and Merge Conflicts When Agents Touch Many Files"
order: 7
duration_min: 30
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

Three people in a group of three, all working on the main branch, all using agents.

Monday evening: Product's agent restructures the menu component. Tech's agent, working from a copy pulled that morning, rewrites the same component to add a sold-out state. Growth's agent adds analytics by inserting a line into every page, including that one.

Tuesday morning: nobody can push. Everyone has conflicts. Two of the three cannot explain what their agent changed. The main branch has been broken since Monday afternoon, so the deployed site is down, and resolving the conflicts means choosing between two versions of a file that neither author read closely.

This is not bad luck. Three people with agents on one branch produces exactly this by Tuesday, every time.

### The idea

**A branch is a parallel copy of the whole project where you can work without affecting anyone else. One change, one branch, merged back when it works.**

Branches cost nothing to create and nothing to throw away. That is what makes them the right unit of work for agent-driven development, where a change is often large and sometimes wrong.

### The analogy

The canteen register again. You want to try a new pricing scheme.

Without branches you write the new prices directly into the live register, in front of customers, while your colleague is also writing in it. If the scheme is bad you have to remember what the old prices were.

With branches you photocopy the whole register, try the new scheme in the copy, and run it for a week. If it works you copy the changes back into the real register. If it does not, you throw the photocopy away and the real register never knew.

The important detail: while you are working in the copy, your colleague keeps writing in the original. When you come back, the original has moved on. That gap is where conflicts come from.

**Where the analogy breaks:** photocopying a register is slow and a branch is instant, because git stores only the differences. Creating a branch for a ten-minute task is normal and costs nothing.

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

The main branch is the one that always works and always deploys. Each change happens on its own branch, which is merged back when it is finished and verified. Two branches can exist at once without touching each other.

The rule that follows: **main is never broken.** If main is broken, the deployed site is broken, nobody can safely start new work, and your team is blocked rather than just one person.

### Branch naming, and keeping them small

A branch name is read by other people in a list, so it should say what the change is:

```
feat/sold-out-toggle
fix/cart-ignores-quantity
chore/add-gitignore
docs/update-prd
```

A prefix and a short description. What matters more than the convention is **the branch being small**. A branch open for a week has diverged a long way from main and will conflict badly. A branch open for an afternoon usually merges cleanly.

With agents this matters more than it used to, because an agent can produce a week's worth of changes in an afternoon. The size of the diff is what causes conflicts, not the calendar time.

### What a merge conflict actually is

A conflict happens when two branches changed **the same lines of the same file** and git cannot tell which version you want. That is all it is. Git is not confused about anything else — different files, or different parts of the same file, merge silently.

When it happens, git edits the file and puts both versions in it:

```
<<<<<<< HEAD
      <span className="price">₹{item.price}</span>
=======
      <span className="price">{formatPaise(item.price_paise)}</span>
>>>>>>> feat/sold-out-toggle
```

Between `<<<<<<<` and `=======` is what is on your current branch. Between `=======` and `>>>>>>>` is what is coming in. Your job is to delete the markers and leave the correct code — which might be either version, or a combination, or something new.

Then stage the file and complete the merge. The conflict is a question, and you are answering it.

**Read both sides before choosing.** The temptation is to keep your own. If you did not write either side — and with agents you often did not — keeping "yours" is a coin flip, and the resolution is where real bugs get introduced silently.

### The agent conflict pattern

Humans and agents cause different conflicts, and the agent ones are worse.

A human asked to add a sold-out toggle edits the menu component and maybe one more file. An agent asked the same thing may also reformat the file, rename a variable for consistency, update three call sites, add a type definition, and tidy an import. Most of that is reasonable. All of it is surface area for conflict.

So the conflict you get is not "we both changed this one line". It is "forty lines of this file look different and I cannot tell which differences were the point".

Three habits defend against it, and they all come from M00:

**Scope the brief.** "Change only `MenuItem.tsx`. Do not reformat, do not rename anything, do not touch other files." The NON-GOALS line exists for exactly this.

**Keep branches short.** Merge the same day. An agent branch open for three days is a guaranteed painful merge.

**Pull from main before you start, and before you merge back.** Most conflicts are caused by working from a stale copy. `git pull origin main` costs two seconds.

And one more that is specific to teams: **agree who owns which files for the week.** Three agents editing one component is the problem; three agents editing three components is fine. This is the practical reason the role split in chapter 1 matters.

### Where it lives

Branches are the unit that pull requests are built on, which is the next chapter. They are also what makes CI useful in M06 — tests run on the branch before it reaches main, so main stays working automatically rather than by discipline alone.

In M06's deploy chapter, most platforms give every branch its own preview URL. That is the thing that makes a branch genuinely reviewable: your teammate can click a link and use the change rather than reading the code.

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

**Pull before you start. This prevents most conflicts you will ever have.** Working from a copy that is two days old is the single most common cause, and it is free to avoid.

**Never commit directly to main once you have teammates.** Not as a formality — because main is what deploys and what everyone else branches from. One broken commit on main blocks three people.

**Do not let an agent resolve a conflict for you unsupervised.** It will produce something that compiles. Whether it preserved the intent of both changes is a different question, and compiling is not evidence. Read the resolution.

**A conflict is not an error.** Beginners panic at the markers. Git is telling you it did its job and found a genuine question only you can answer. Nothing is broken and nothing is lost.

**Prefer merge over rebase while you are learning.** Rebase produces cleaner history and rewrites commits, which is confusing when it goes wrong and dangerous on a shared branch. Merge is harder to break.

**`git stash` is for emergencies, not storage.** It is easy to stash something, forget it, and lose a day's work that is technically still there. If the work matters, commit it on a branch.

### Verify it

Before merging any branch, five checks. Three of them are about the agent, which is where the risk is.

1. **Did you pull main first?** If not, you are about to merge a stale branch.
2. **Does the file list match what you asked for?** `git diff main --stat` shows files and line counts. An unexpected file is a scope violation to investigate before merging, not after.
3. **Do you understand every resolution you made?** If you kept one side of a conflict without reading the other, go back and read it.
4. **Does it actually work on the branch?** Run it. In M06 CI will do this for you; until then, you do it.
5. **Is main still working after the merge?** Check immediately. A broken main blocks everyone, and the fastest fix is to revert the merge rather than debug it under pressure.

The recovery you should know before you need it: **a merge you have not pushed can be undone.** `git merge --abort` during a conflict returns you to before the merge started. After a completed local merge, `git reset --hard HEAD~1` discards it — destructive, so be sure your own work is committed first. Knowing these two exist is what lets you merge without anxiety.

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
