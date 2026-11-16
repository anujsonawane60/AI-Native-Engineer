---
id: m01-08-pull-requests-and-diffs
module: m01-team-startup-git
slug: pull-requests-and-diffs
title: "Pull Requests and Reviewing a Diff You Did Not Write"
order: 8
duration_min: 35
is_free: false
mission: true
volatility: low
last_reviewed: 2026-11-16
---

## Key learnings
- Open a pull request that someone can actually review
- Review a diff at the concept level without reading every line
- Apply a six-question review checklist to agent-written changes
- Write review comments that get the change fixed
- Explain why review is the core skill of an AI-native engineer

## Lesson

### The pain

A student opens a pull request titled "changes". It contains 1,400 changed lines across 23 files. The description is empty.

Their teammate opens it, scrolls for thirty seconds, sees nothing obviously wrong, and clicks Approve. Three days later the sold-out feature works but prices display 100× too large, because somewhere in those 1,400 lines a conversion from paise to rupees was removed. Both of them "reviewed" it. Neither of them looked at that line.

The reviewer is not lazy. A 1,400-line diff is genuinely unreviewable, and clicking Approve was the only available action. The failure happened when the pull request was opened, not when it was approved.

### The idea

**A pull request proposes a change and asks a human to check it before it reaches main. With AI writing the code, this review is the job.**

You are not reviewing syntax — the agent is better at syntax than you are. You are reviewing intent, scope, and the things an agent systematically gets wrong.

### The analogy

The canteen hires the fast helper from M00. They have reorganised the storeroom.

A bad handover: "I reorganised it." You have no idea what moved. You either trust them completely or re-check every shelf yourself. Both are bad options.

A good handover: "I moved the dry goods to the left shelf so they're nearer the prep station, I threw out three expired packets — here they are — and I did not touch the fridge. The only thing I'm unsure about is whether the oil should be by the stove or in the cupboard."

Now your review takes ninety seconds. You check the three thrown-out packets, you confirm the fridge is untouched, and you answer the one open question. The handover told you where to look.

**Where the analogy breaks:** a helper knows what they were uncertain about. An agent does not — it has no sense of which of its changes were judgement calls. So the description has to come from *you*, which is why writing it is part of your work and not the agent's.

### Picture it

```
  feat/sold-out-toggle              main
         │                            │
         │  open pull request         │
         ├───────────────────────────▶│
         │                            │
         │   ┌──────────────────────┐ │
         │   │ WHAT   one sentence  │ │
         │   │ WHY    the brief     │ │
         │   │ SCOPE  files touched │ │
         │   │ TEST   how to check  │ │
         │   │ RISK   what I'm      │ │
         │   │        unsure about  │ │
         │   └──────────────────────┘ │
         │            │               │
         │       reviewer reads       │
         │       description first,   │
         │       then the diff        │
         │            │               │
         │   comment ─┤               │
         │◀───────────┘               │
         │  fix, push again           │
         │                            │
         │       approve + merge      │
         └───────────────────────────▶│
```

The description is read before the diff, and it tells the reviewer where to look. Without it, the reviewer has to reconstruct your intent from the code, which is slow and unreliable.

### How to review without reading every line

You will not read 1,400 lines, and you should not try. Review in four passes, cheapest first, and stop as soon as you find something that needs fixing.

**Pass 1 — the file list, ten seconds.** Does the set of changed files match the stated scope? A payments file in a styling change is a red flag before you read any code. This pass catches the most problems per second of effort.

**Pass 2 — the line count, five seconds.** Is it proportionate? "Fix one calculation" should not be 600 lines. If it is, ask why before reading further.

**Pass 3 — the diff itself, a few minutes.** Read only the changed lines. Look for the specific things agents get wrong (below), not for style.

**Pass 4 — run it.** Use the preview URL and try the thing. Then try to break it: the empty case, the wrong user, submitting twice. This is the pass that catches what reading cannot.

### The six-question review checklist

These are the questions that find real problems in agent-written code. They come from the failure modes in M00 and the four-box model from M00.3.

**1. Does it do what the brief asked, and nothing else?**
Scope creep is the most common defect. Unrequested improvements are not free — they are unreviewed changes.

**2. Is any decision that matters happening on the client?**
Prices, permissions, discounts, "is this user staff". If the server is not re-deciding it, it is not decided. This is the single highest-value question on the list.

**3. What happens when it fails or is empty?**
Agents build the happy path. No items, no network, a failed save, a very long name. Ask for the states, not the feature.

**4. Did the data shape change, and what about existing rows?**
A new required field breaks every row that already exists. This is cheap to fix before merge and expensive afterwards.

**5. Is anything secret in here?**
Keys, tokens, a database URL, a real email address. Search the diff. The commit is permanent the moment it merges.

**6. Can I explain what this does?**
If you cannot, you cannot maintain it, debug it, or defend it in M18. "I don't understand this file, walk me through it" is a legitimate review comment and one of the most useful you can write.

### Writing comments that get it fixed

The same discipline as giving feedback to AI in M00: symptom, expected versus actual, one thing at a time.

| Weak | Strong |
|---|---|
| "This looks wrong" | "Line 42: price is divided by 100 twice, so ₹40 renders as ₹0.40" |
| "Needs error handling" | "If the save fails, the toggle stays on and the user thinks it worked. Can it revert and show a message?" |
| "Too many changes" | "This PR also reformats `Menu.tsx` and renames `price` to `amount`. Can you split those out so I can review the toggle separately?" |
| "Is this secure?" | "A logged-in student can call this endpoint directly. Is the role checked server-side?" |

Two conventions that help a three-person team move fast:

Mark comments as **blocking** or **non-blocking**. "Blocking: the role is not checked server-side" versus "non-blocking: I'd name this `isAvailable`". Without the distinction, every comment reads as a demand and reviews become slow and tense.

**Approve with comments** when nothing is blocking. Holding a PR hostage over a variable name is how a team stops reviewing each other's work at all.

### Where it lives

Pull requests are where several threads in this program meet. In M06, CI runs your tests automatically on every PR, so the machine checks what it can and you check what it cannot. In M06's deploy chapter, each PR gets a preview URL, which is what makes pass 4 possible. In M10, you will review much larger agent-generated PRs, and the agent-ready repository exists so that tests and linters catch the mechanical problems before a human looks.

And in M18 the defence is individual. Your review comments are evidence that you engaged with code you did not write — which is the actual skill this program is training.

### Words to own

| Term | Plain meaning | Example |
|---|---|---|
| Pull request / PR | A proposal to merge a branch, open for review | Your unit of work |
| Diff | The lines that changed | What you review |
| Review | Checking a change before it merges | The job, with AI |
| Approve | Signing off that it can merge | Your name is on it |
| Blocking comment | Must be fixed before merge | "Role not checked" |
| Non-blocking comment | A suggestion, merge anyway | "I'd rename this" |
| Preview deploy | A live URL for the branch | How you test a PR |
| CI check | Automated tests run on the PR | Green before review |
| Squash merge | Combine the branch's commits into one | Tidier main history |
| Protected branch | Main cannot be pushed to directly | Forces PRs |

### Tradeoffs and gotchas

**The reviewer's problem is usually the author's fault.** An unreviewable PR is a PR that was opened wrong. If your teammate cannot review your change in ten minutes, split it — and the fix is in your brief, which should have been scoped more tightly in the first place.

**Approving without understanding is worse than not reviewing.** It puts your name on the change and creates the false impression that it was checked. If you do not understand it, say so. That is the honest and useful move.

**Do not let the agent open and merge its own PRs.** It is capable of the whole cycle. The result is a repository where nothing was ever reviewed and nobody can explain anything, which is precisely the outcome this program exists to avoid.

**Review the PR, not the person.** "This endpoint trusts the client" is about code. "You always forget authorization" is about a person, and in a three-person team that has to work for six months, it costs more than it gains.

**Protect main once the habit is established.** Turning on branch protection means nobody — including you at 2 am, including an agent — can push to main without a PR. It removes the temptation rather than relying on discipline.

**Small PRs get real reviews. Large ones get rubber stamps.** This is the most reliable finding in all of code review, and it is entirely under the author's control.

### Verify it

Before you open a PR, four checks:

1. **Is the diff under roughly 400 lines?** Beyond that, reviews degrade into skimming. Split it.
2. **Does the file list match the stated scope?** Run `git diff main --stat` and read it yourself first. Fix the surprises before a human sees them.
3. **Have you written WHAT, WHY, SCOPE, TEST and RISK?** The RISK line is the most valuable — name the part you are least sure about, because that is where the reviewer should spend their attention.
4. **Have you run it yourself on the preview URL?** Including the empty case and the wrong-user case.

Before you approve one, two checks:

1. **Did you run the six questions, or just scroll?** Questions 2 and 3 find the most real defects.
2. **Can you explain what this change does, in one sentence, without looking?** If not, comment instead of approving.

## Walkthrough
1. Start clean: `git switch main`, `git pull origin main`, `git switch -c feat/your-change`.
2. Write a tightly scoped brief for one small real change in your product. Include a NON-GOALS line naming the files the agent must not touch and forbidding reformatting and renaming.
3. Let the agent make the change. Do not accept it yet.
4. Run `git diff main --stat` and read the file list and line counts. If there are files you did not expect, that is your first finding — ask the agent why, and have it revert the out-of-scope parts.
5. Read the diff with `git diff main`, or in your editor's diff view. Go through the six questions. Write down anything you find, even if you are not certain.
6. Commit and push the branch, then open a pull request on GitHub.
7. Write a real description: WHAT in one sentence, WHY with the brief, SCOPE as the files touched, TEST as the steps to check it, RISK as the one thing you are least sure about.
8. Now swap. Have a teammate review yours while you review theirs. If you are working alone, review it yourself the next morning — the delay genuinely helps, and reviewing your own agent's work is the common case anyway.
9. As reviewer, work the four passes in order: file list, line count, the diff, then run it on the preview URL. Try the empty case and the wrong-user case.
10. Leave at least three comments, each labelled blocking or non-blocking, each naming a symptom with expected versus actual and a line number.
11. As author, fix the blocking comments, push again, and reply to each comment saying what you changed. Then merge.
12. Add the rule to `TEAM.md`: every change goes through a PR, PRs stay under 400 lines, descriptions include RISK, reviews happen within one day.

## Talk to AI
```prompt
I am reviewing a pull request that an AI agent wrote, and I want to review it properly rather than scroll and approve.

The brief I gave the agent was: [paste your original brief, including non-goals]
The files it changed: [paste the output of git diff main --stat]
The diff itself: [paste it, or the most important parts]

Act as a strict reviewer who did not write this code. Work through these in order:
1. SCOPE - does the file list match my brief? Name every file that should not be here and say what it appears to be doing.
2. CLIENT TRUST - is any decision about price, permission, role or identity being made in client code without the server re-checking it? Quote the lines.
3. FAILURE STATES - what happens when the list is empty, the network fails, the save fails, or the same action is submitted twice? Name each state that is unhandled.
4. DATA - did the shape of stored data change? What happens to rows that already exist?
5. SECRETS - is there any key, token, database URL or real personal data in this diff?
6. UNDERSTANDING - pick the two changes a beginner would be least likely to understand, and explain each in two sentences.

Then write me three review comments in the correct form - symptom, expected versus actual, line reference - and label each one blocking or non-blocking.

Finally: what is the single most likely bug in this change that none of the above would catch?
```

## Mission

**Objective.** One real change to your product, made by an agent on a branch, opened as a reviewable pull request, reviewed against the six questions, fixed, and merged — with your review comments as the evidence.

**Steps.**
1. **Frame** — pick one small real change from your slice. Small enough to be under 400 lines.
2. **Brief** — write a scoped brief with an explicit NON-GOALS line forbidding reformatting, renaming and out-of-scope files.
3. **Direct** — branch from an up-to-date main, let the agent make the change.
4. **Self-review** — `git diff main --stat` first. Investigate every unexpected file before a human sees it.
5. **Open** — push and open a PR with WHAT, WHY, SCOPE, TEST and RISK.
6. **Review** — a teammate reviews yours and you review theirs, using the four passes and the six questions. Alone: review the next morning.
7. **Comment** — at least three comments, labelled blocking or non-blocking, each with a symptom and expected versus actual.
8. **Fix and merge** — address blocking comments, reply to each, merge, and confirm main still works.

**Upload.**
- The brief you gave the agent, including its non-goals
- A screenshot of `git diff main --stat` showing the file list and line counts
- The PR URL, or a screenshot showing the full description with all five fields → `repo_url`
- Screenshots of your review comments, with the blocking/non-blocking labels visible
- At least one comment that found a real problem, and the fix that followed
- Proof the change works, including one failure state → `demo_url` or screenshots
- Your AI review conversation → share link in `ai_chat_url`
- Reflection, 150 words max → what the six questions caught that scrolling would have missed, and one thing you approved that you were not fully sure about

**Self-check.** Tick every box before you submit.
- [ ] My branch started from an up-to-date main
- [ ] My brief had an explicit non-goals line about scope, reformatting and renaming
- [ ] The diff is under roughly 400 lines
- [ ] I ran `git diff main --stat` and checked the file list before opening the PR
- [ ] My PR description has WHAT, WHY, SCOPE, TEST and RISK
- [ ] My RISK line names something I was genuinely unsure about
- [ ] I asked all six review questions, not just scrolled the diff
- [ ] I ran the change myself, including an empty or failing case
- [ ] My comments name symptoms with expected versus actual, not opinions
- [ ] Every comment is labelled blocking or non-blocking
- [ ] Main still works after the merge, and I checked

**Stretch.** Turn on branch protection so main cannot be pushed to directly and requires a PR. Then try to push to main and screenshot the rejection. Write 100 words on what the team gains by making the rule structural rather than a shared intention.

## Stuck? Ask AI
```prompt
I have a pull request with 1,400 changed lines across 23 files and I do not know how to review it. The brief was: [paste it]. Help me decide how to split it into reviewable pieces, and tell me which parts of this diff I should look at first given that I cannot read all of it.
```

```prompt
Here are three review comments I wrote: [paste them]. Rewrite each as a symptom with expected versus actual and a clear scope, and label each blocking or non-blocking. Then explain what was wrong with my versions.
```

```prompt
Explain why small pull requests get real reviews and large ones get rubber stamps. Then explain whose responsibility that is, and what I should change about how I brief agents as a result.
```

## Quiz

### Q1 · L4
A PR has 1,400 changed lines across 23 files and an empty description. The reviewer scrolls briefly and approves. Where did the failure actually happen?
- (a) At approval — the reviewer should have read all 1,400 lines
- (b) When the PR was opened unreviewably large with no description ✅
- (c) At merge — CI should have caught the bug
- (d) In the agent's code generation
> A 1,400-line diff is genuinely unreviewable and Approve was the only available action. Reviewability is the author's responsibility, and it traces back to an unscoped brief.

### Q2 · L4
Which review pass catches the most problems per second of effort?
- (a) Reading every changed line carefully
- (b) Checking that the changed-file list matches the stated scope ✅
- (c) Checking code style and naming consistency
- (d) Reading the commit messages on the branch
> A payments file in a styling change is a red flag before any code is read. Ten seconds on the file list beats ten minutes of line-by-line reading.

### Q3 · L4
Of the six review questions, which finds the most serious class of defect in agent-written code?
- (a) Whether variable names are consistent
- (b) Whether a decision about price, permission or role happens only on the client ✅
- (c) Whether the commit messages explain why
- (d) Whether the PR is under 400 lines
> The client cannot be trusted, and agents default to the shortest path. An unenforced permission or a client-calculated price works perfectly in testing and also works for an attacker.

### Q4 · L3
Why label review comments as blocking or non-blocking?
- (a) GitHub requires a label before a comment can be posted
- (b) Otherwise every comment reads as a demand, and reviews become slow and tense ✅
- (c) Non-blocking comments are hidden from the author
- (d) It determines whether CI runs again
> In a three-person team that must work for six months, holding a PR hostage over a variable name is how mutual review stops happening altogether.

### Q5 · L4
You cannot understand part of a diff you are reviewing. What is the right action?
- (a) Approve it, since the agent is more reliable than your judgement
- (b) Comment asking for an explanation instead of approving ✅
- (c) Reject the PR and ask for it to be rewritten by hand
- (d) Merge it and investigate afterwards if something breaks
> Approving puts your name on it and falsely signals it was checked. "Walk me through this file" is a legitimate and valuable review comment.

## Go deeper
- [Google's code review developer guide — what to look for](https://google.github.io/eng-practices/review/reviewer/looking-for.html)
- [GitHub docs — about pull requests](https://docs.github.com/en/pull-requests/collaborating-with-pull-requests/proposing-changes-to-your-work-with-pull-requests/about-pull-requests)
