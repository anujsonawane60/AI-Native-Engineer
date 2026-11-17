---
id: m06-04-commands-vocabulary
module: m06-cloud-linux-devops-testing
slug: commands-as-vocabulary
title: "Commands as Vocabulary: ls, grep, top, df, curl"
order: 4
duration_min: 25
is_free: false
mission: true
volatility: low
last_reviewed: 2026-11-16
---

## Key learnings
- Sort any command into read-only, changes-things, or destroys-things before running it
- Use five commands to answer the five questions every outage starts with
- Read command output as evidence rather than as noise
- Refuse a command you cannot describe in one sentence
- Insist that AI label what a command does before you run it

## Lesson

### The pain

A student's deployed app is down. An AI gives seven commands and they paste all seven. The fifth, `docker system prune -a`, deleted the one image that still worked — the app is now down for a different reason and the original cause is still unknown.

A second student, same outage, runs four read-only commands: the process is not running, the disk is 100% full, one directory is consuming it, and the log grew to 4 GB in a loop. The difference is not Linux knowledge — the second asked questions; the first applied remedies to a problem nobody had identified.

### The idea

**You do not memorise commands. You learn a small vocabulary of questions you can ask a machine, and you sort every command into read-only, changes-things or destroys-things before it runs.**

Five questions cover most of what goes wrong: what is here (`ls -la`), where does this text appear (`grep -r`), what is using the machine (`top`, `ps aux`), is there room left (`df -h`, `du -sh *`), and what does the server actually say (`curl -i`). All five are read-only: run them in any order, at any time, and nothing changes. `grep` matters most when you work with AI, because it answers "did it change what it said it changed?" `curl` tells you which half is lying when your app works in one place and not another.

### The analogy

A new cook on their first shift does not start by changing the recipe. They look: what is in the store room, how much gas is in the cylinder, which pot is on the fire, what the order slips say. Five glances, no decisions. The experienced cook's advantage is not more recipes; it is looking before acting.

**Where the analogy breaks:** a cook who adds too much salt can usually dilute it. Many commands have no dilution. `rm` does not move a file to a recycle bin — no undo, no confirmation, no trace. This is the read-versus-destroy distinction from M00, and it is why the vocabulary comes in three categories rather than one list.

### Picture it

```
  Before you run anything, sort it:

  READ-ONLY            CHANGES THINGS         DESTROYS THINGS
  nothing is           reversible, but        no undo; back up
  altered              you must know how      or do not run
  ─────────────        ─────────────────      ───────────────
  ls   pwd   cat       mkdir   touch          rm   rm -rf
  grep head tail       mv      cp             DROP TABLE
  top  df   du         chmod   chown          git reset --hard
  curl ps   whoami     git checkout -b        docker prune -a
  git status/diff/log  npm install            force push
     │                       │                       │
     ▼                       ▼                       ▼
  run freely          read it first           say the sentence
                                              out loud first
```

Every command belongs in exactly one of three columns, and the only column you may run without thinking is the left one.

### Where it lives

These commands run everywhere: a free browser dev environment, your platform's web shell, a container in chapter 8, a CI run in chapter 12. They are also how you verify an agent's work, which is their real role here. M00 introduced the read-versus-destroy table and the permission allowlist; this chapter is the vocabulary that makes it usable, and M10's agent-ready repository assumes you can check what an agent did with `git status`, `git diff` and `grep`.

### Words to own

| Term | Plain meaning | Example |
|---|---|---|
| `ls -la` | List everything here, with owners | Confirm `.env` exists |
| `grep -r` | Search for text across files | Find every use of a variable |
| `top` | Live view of what is using the machine | Is it busy or stuck? |
| `ps aux` | One-shot list of all processes | Is my app running? |
| `df -h` | Free disk space, readable units | The silent outage cause |
| `du -sh *` | Size of each thing here | What filled the disk |
| `curl -i` | Make a request, show status and headers | Ask the server directly |
| Flag | A letter changing a command's behaviour | `-h` = human-readable |
| Pipe (`\|`) | Feed one command's output into another | `ps aux \| grep node` |
| `rm -rf` | Delete a folder and everything in it, no undo | Never run unread |
| Exit code | The pass/fail number a command returns | `0` means success |
| Idempotent | Safe to run twice with the same result | `ls` is; `rm` is not |

### Tradeoffs and gotchas

- **The dangerous commands look like the safe ones.** `rm -rf build` and `rm -rf /` differ by six characters, with no confirmation and no recovery. Say it out loud as a sentence first: "delete the build folder and everything in it, permanently" is fine; "delete everything on this machine, permanently" stops your hand.
- **A flag can change the category.** `git checkout branch-name` is reversible; `git checkout -- .` throws away every uncommitted change with no undo. Same verb, different category — ask what an unfamiliar flag does before running it.
- **Copy-pasting a block of commands is the actual risk.** Individually you would have questioned the fifth one. Run them one at a time; stop when something surprises you.
- **`curl` tells you the truth your browser will not.** Browsers cache, follow redirects silently, run service workers and may be logged in. Students have lost an hour to a bug `curl -i` would have shown to be a 500.

### Verify it

Read output as evidence: expect something specific before you look ("if the app is running, `ps aux | grep node` shows one line"), copy the exact text, and treat absence as output — an empty `grep` is a finding. Then four checks.

1. **Demand a label on every command** — READ-ONLY, CHANGES-THINGS or DESTROYS-THINGS, plus the undo for anything in the middle category.
2. **Can you describe it in one sentence?** If not, you may not run it. "It cleans things up" is not a sentence about what it does.
3. **Did it diagnose before prescribing?** Seven fixes and no read-only check is a remedy for an unidentified problem. Ask for the read-only checks first.
4. **Does its claim survive `grep`?** "I updated all references to the old variable name" is checkable in three seconds.

Red flags: `sudo` inside your own project folder; `rm -rf` on a path you have not read character by character; a command pasted without its flags explained; a prune or clean command as a first step; an agent reporting success on a change `grep` disproves.

## Walkthrough
1. Open a free browser-based Linux environment with your team's project in it, or your platform's web shell. Everything here is read-only, so nothing you do can break the project.
2. Run `pwd` then `ls -la`. Identify three files you recognise, any hidden file starting with a dot, and whether `.env` is present. Do not open `.env`.
3. Run `ls -l` on one file and say out loud who owns it and who can write to it, using chapter 1's reading of the nine letters.
4. Run `grep -r "DATABASE" . --include="*.ts" --include="*.js"` — or your stack's extensions — and read the result. Note how many files mention it and whether any of them surprises you.
5. Pick a variable or function you believe is unused and `grep` for it. Zero results is a finding: it means nothing references it. Write down what you concluded.
6. Run `ps aux | head -15`, then `top`, then press `q` to leave. Name the process using the most memory, and say whether you expected it.
7. Run `df -h` and find the line for the file system your project lives on. Write down the percentage used. Then run `du -sh *` in your project folder and name the largest thing in it.
8. Run `curl -i https://example.com` and read the first line: protocol, status code, status text. Then run it against your own deployed app's URL and compare.
9. Now break something safely. Request a path on your app that does not exist, with `curl -i`, and read the status code. That is what your users see when a link is wrong.
10. Write a three-column table in your team notes — read-only, changes-things, destroys-things — and sort every command you ran today into it. Add three commands an AI has previously given you and sort those too.
11. Finally, ask your AI assistant for five commands to diagnose a slow app, and insist it label each one. Reject and re-ask if any label is missing.

## Talk to AI
```prompt
CONTEXT:      I am a student on an AI-native engineering program. I direct AI
              rather than writing code, and I do not memorise command syntax.
              My team runs a web product on a free-tier hosting platform with
              a managed database. I have access to a browser terminal. I have
              previously pasted a block of commands from an AI without reading
              them and made an outage worse.

GOAL:         Build a small, reliable vocabulary of read-only commands that
              lets me diagnose problems and verify AI's claims about my own
              project, and a rule for never again running something I cannot
              describe.

REQUIREMENTS: 1. For each of ls, grep, top, df and curl, tell me the ONE
                 question it answers, the exact form I should use, and what a
                 normal result versus a worrying result looks like
              2. Give me a three-column table - READ-ONLY, CHANGES-THINGS,
                 DESTROYS-THINGS - containing every command you have
                 mentioned, plus the ten commands students most often paste
                 without reading
              3. For every command in the middle column, state the undo. For
                 every command in the right column, state exactly what is
                 lost and whether it is recoverable
              4. Show me three pairs of commands that look nearly identical
                 but sit in different columns, and explain the difference
              5. Give me the exact grep command that would check whether a
                 claim like "I renamed every reference to X" is actually true

CONSTRAINTS:  Plain English, second-language reader. Free tier only, no paid
              tools. Assume a slow connection and a small terminal window, so
              show me how to cut long output down with head and grep.
              Every command you give me must carry its category label.

NON-GOALS:    Do not teach me shell scripting, awk, sed, vim, or regular
              expression syntax beyond plain strings. Do not give me aliases
              or a dotfile to install. Do not optimise for brevity - I want to
              understand, not to type fast.

ACCEPTANCE:   - I can look at any command and name its column
              - I can state in one sentence what `rm -rf build` destroys and
                whether I can get it back
              - I can name the command that proves whether my app is actually
                running, and the one that proves whether the disk is full
              - I can use curl to find out whether a failure is the browser's
                fault or the server's
              - I can verify a rename claim across my whole project in under
                a minute

ASK:          First ask me what my project is built with and where it runs, so
              the examples use my real file types and URL. Then answer. Finish
              by giving me eight commands one at a time and asking ME which
              column each belongs in, without telling me the answers first.
```

## Mission

**Objective.** Produce a one-page command runbook for your own startup — the five read-only questions, answered with your real project's paths and URL — plus a three-column safety table, and prove you used it to catch at least one claim an AI made about your code.

**Steps.**
1. **Frame** — write one sentence naming the five questions you would need answered if your product went down during your demo, in the order you would ask them.
2. **Brief** — write a Director's Brief asking AI for the exact read-only command for each of your five questions, against *your* stack, *your* file extensions and *your* deployed URL. Require a category label on every line.
3. **Direct** — run all five against your own project and capture the real output. Not an example output. Yours.
4. **Classify** — build the three-column table with every command you ran today plus the five riskiest commands an AI has ever given your team. For each destroys-things entry, write what would be lost and whether git or a backup would recover it.
5. **Catch something** — ask an AI to make a small change to your project, then verify its claim with `grep` and `git diff` rather than its summary. Record what it said and what you found. If it was correct, say so and record the commands that proved it.
6. **Verify** — hand your runbook to a teammate and have them follow it on their own machine without asking you anything. Note every place they got stuck.
7. **Document** — add the runbook to your repository as `RUNBOOK.md` and commit it. It will be used again in chapters 11 and 12, and in M11's going-live checkpoint.

**Upload.**
- Your `RUNBOOK.md`, with real paths and your real URL → attached file or post caption
- Your three-column safety table → in the runbook or as a screenshot
- Terminal output from all five read-only commands against your project → screenshots
- The AI conversation where you demanded category labels → share link in `ai_chat_url`
- Your verification of an AI claim: what it said, the `grep` or `git diff` you ran, what you found → screenshot
- Reflection, 150 words max → one command you will never run again without reading, and one thing the AI got wrong

**Self-check.** Tick every box before you submit.
- [ ] Every command in my runbook is labelled read-only, changes-things or destroys-things
- [ ] Every destroys-things entry says what is lost and whether it is recoverable
- [ ] My five commands use my project's real paths and my real deployed URL, not placeholders
- [ ] I captured output from my own project, not an example from the AI
- [ ] I used `curl -i` against my own app and can state the status code it returned
- [ ] I checked an AI claim with `grep` or `git diff` and recorded the result either way
- [ ] A teammate followed my runbook without me explaining anything
- [ ] I can describe every command in my runbook in one sentence, unaided
- [ ] `RUNBOOK.md` is committed to our repository

**Stretch.** Add a sixth question specific to your product — "are the uploads getting through?", "is the payment webhook arriving?" — and find the read-only command or `curl` call that answers it. That one is worth more in your demo than the other five combined.

## Stuck? Ask AI
```prompt
Give me a command and tell me nothing about it. I will guess whether it is read-only, changes things, or destroys things, and what it does. Then tell me if I was right and explain. Do this ten times, increasing in difficulty, and include at least two pairs that look nearly identical but sit in different categories. Do not give me the answers before I guess.
```

```prompt
An AI told me it had renamed every reference to a variable in my project. I do not want to read every file. Give me the exact grep command that would prove or disprove that claim for a project using [your file types], explain what output would mean it succeeded, and what output would mean it missed something. Then tell me the equivalent check using git.
```

```prompt
My app works in my browser but a teammate says it is down. Give me the read-only commands, in order, that separate "the server is broken" from "the browser is lying to me", starting with curl. For each one tell me what result would point at which half. Do not give me any fix - only the diagnosis sequence.
```

## Quiz

### Q1 · L2
Which of these commands is read-only — nothing on the machine is altered by running it?
- (a) `chmod 600 .env`
- (b) `df -h` ✅
- (c) `npm install`
- (d) `git checkout -- .`
> `df -h` reports free disk space and changes nothing. `chmod` alters permissions, `npm install` writes a dependency folder, and `git checkout -- .` destroys every uncommitted change with no undo.

### Q2 · L3
Your app "works for me" but a teammate sees an error. What does `curl -i` against the deployed URL add that the browser cannot?
- (a) It tests from a different country
- (b) It shows the raw status code and headers with no cache, extension, login or service worker in the way ✅
- (c) It bypasses the firewall rules
- (d) It retries automatically until it succeeds
> The browser caches, follows redirects silently, runs a service worker and may be logged in as you. `curl -i` asks the server directly and prints exactly what it said.

### Q3 · L4
An AI gives you seven commands to fix an outage, none labelled. What is the correct next move?
- (a) Paste all seven and read the combined output afterwards
- (b) Run them in reverse order so the risky ones come last
- (c) Ask for the read-only diagnostic steps first, with a label and an expected result for each ✅
- (d) Run only the ones you recognise and skip the rest
> Seven fixes with no diagnosis is applying remedies to an unidentified problem. One of them is likely to destroy evidence or state, and as a pasted block you will never know which one did what.

### Q4 · L4
An agent reports: "I updated every reference to `MENU_API_URL` to the new name." What is the fastest honest check?
- (a) Ask it to confirm
- (b) Redeploy and see whether the app still works
- (c) `grep -r "MENU_API_URL" .` and expect zero results ✅
- (d) Read every file it listed as changed
> A rename claim is directly checkable in seconds. Zero results means it is genuinely gone; any hit names the file it missed. A redeploy only tells you whether the paths you happened to exercise still work.

### Q5 · L5
`git checkout feature-branch` and `git checkout -- .` look almost identical. Why does this pair matter more than any other example in this chapter?
- (a) One is slower than the other on large repositories
- (b) One is a reversible move between branches; the other permanently discards all uncommitted work ✅
- (c) One requires `sudo` and the other does not
- (d) One only works on committed files
> They differ by three characters and sit in different columns. There is nothing in the shape of a command that reveals its category, which is exactly why the labelling habit, not pattern recognition, is what protects you.

## Go deeper
- [Explain Shell: paste any command and see every flag decoded](https://explainshell.com/)
- [The Linux Command Line, free book by William Shotts](https://linuxcommand.org/tlcl.php)
