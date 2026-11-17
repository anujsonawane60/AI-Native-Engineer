---
id: m13-04-infrastructure-as-code
module: m13-production-engineering
slug: infrastructure-as-code
title: "Infrastructure as Code: Terraform, State, Drift and GitOps"
order: 4
duration_min: 25
is_free: false
mission: true
volatility: medium
last_reviewed: 2026-11-16
---

## Key learnings
- Read a plan output and say exactly which resources will be created, changed and destroyed
- Explain what the state file is, why it is dangerous, and where it must live for a team
- Detect drift and decide whether to fix the code or import the reality
- Protect a stateful resource so no plan can casually destroy it
- Catch an apply that was run without anybody reading the plan

## Lesson

### The pain

M06 chapter 13 taught you what infrastructure as code is and then told you not to use it. For three people with four resources that was right: a committed `STACK.md` gave most of the value for a tenth of the work. It promised M13 would do this properly, with state files, plans and modules. You are here because the text file has started lying — a teammate's 11 p.m. console bucket was never in it, so a cleanup of unidentified resources broke every dish photo.

The tool's own failure mode is worse. State lives on one laptop, a second teammate runs the same code, the tool finds no state and creates a second database. Later the first teammate applies, the tool offers to destroy things, and somebody types yes.

### The idea

**Infrastructure as code gives you one thing no console can — a reviewable diff before a change — and the state file is the price you pay for it.**

Your resources live in text files in git. The tool compares your code, the state file's memory and the cloud's reality, then prints the difference as a plan — your review surface, exactly like M01's `git diff`. State links a line of code to a real resource id, which is why losing, duplicating or committing it are the three ways this goes wrong.

### The analogy

M06 gave you the canteen's recipe binder: layout, suppliers and counters written down so the next manager does not work from rumour.

The binder has a page you did not need before — the inventory page, recording which specific physical things exist. Not "three warmers" but "warmer serial 4471 at counter 2". The recipes say what should exist, the inventory says what does, and before any change the manager reads a sheet comparing them.

**Where the analogy breaks:** a wrong inventory page is harmless, because the warmers are right there. A wrong state file makes the tool act on a false belief at full speed, creating a second database or modifying a ghost.

### Picture it

```
  YOUR CODE          STATE FILE            REAL CLOUD
  (git, reviewed)    (remote, locked)      (what exists)
 ┌──────────────┐   ┌───────────────┐   ┌─────────────────┐
 │ db: size s   │   │ db  → id-8821 │   │ db  id-8821 (m) │
 │ bucket: on   │   │ bkt → id-4417 │   │ bkt id-4417     │
 │ dns: canteen │   │  (no dns)     │   │ (no dns record) │
 └──────┬───────┘   └───────┬───────┘   └────────┬────────┘
        └───────────────────┴────────────────────┘
                            │
                            ▼
                    ┌───────────────┐
                    │  PLAN         │
                    │  ~ db  s ◀ m  │  drift: someone resized
                    │  + dns        │  new: not created yet
                    │  0 destroy    │  ◀── read this line first
                    └───────────────┘
```

The plan is produced by comparing your code, the state file's memory and the cloud's reality, and the counts of create, change and destroy are the three numbers you must read before you type anything.

### Where it lives

This sits under almost everything else in M13:

- **Chapter 3's** cluster should be created this way, not clicked; **chapter 5's** pipeline is where an apply runs.
- **Chapter 9's** second region is only tolerable as a module you instantiate twice, and **chapter 11's** permissions are resources too.
- It closes M06 chapter 7's loop: a resource created in code has an owner, a region and a tag, so surprise bills stop being mysterious.
- **GitOps** goes further: the repo is the desired state and a job reconciles it, undoing clicks.

### Words to own

| Term | Plain meaning | Example |
|---|---|---|
| Provider | The plugin that talks to one cloud or service | A DNS provider |
| Resource | One thing you declare | A bucket, a DNS record |
| Plan | The diff: create, change, destroy counts | Your review surface |
| Apply | Make reality match the code | The point of no return |
| State file | Maps your code to real resource ids | Must be remote |
| Remote backend | Where shared state lives, with locking | Not a laptop |
| State lock | Stops two people applying at once | Prevents duplicates |
| Drift | Reality no longer matches the code | Someone clicked |
| Import | Teach state about a resource you made by hand | Adopting the bucket |
| Module | A reusable group of resources with inputs | One per environment |
| Variable | An input you change per environment | Region, size |
| `prevent_destroy` | A guard that makes a destroy plan fail | On the database |
| Taint / replace | Force a resource to be destroyed and recreated | Rarely what you want |
| GitOps | The repo is the desired state; a job reconciles it | Auto-undo of clicks |

### Tradeoffs and gotchas

**Never commit the state file.** It holds resource ids and often generated passwords in plain text — a secret leak under M07 chapter 2's rules. Put state in a remote backend with locking: a vendor free tier, or an object store with a lock table. Choose on day one.

**Read the destroy count before anything else.** The plan is long and the dangerous line is one word: `destroy`, or `must be replaced` — which on a database means an empty database with a new address.

**Put a destroy guard on everything stateful:** a lifecycle rule that makes a destroy plan *fail* turns your worst accident into an error message.

**Apply from a pipeline, not a laptop.** Chapter 5 runs plan on every pull request and apply only on merge, so no change depends on whose credentials were loaded.

**Drift has two correct responses, chosen deliberately:** apply over reality, or update the code to match it — never both at random. A resource the state never heard of needs `import`, and free-tier resources are still real: set the budget alert before your first apply.

### Verify it

**Nobody applies anything whose plan they have not read.**

1. **Demand the plan as output, not a description.** The plan text with its three counts is the evidence.
2. **Count the destroys yourself,** then say what data each destroyed or replaced resource holds.
3. **Confirm where state lives:** which backend, is it locked, is it in `.gitignore`.
4. **Look for real credentials in the `.tf` files.** They come from variables backed by a secret store, never a literal.

Red flags: applying without reading the plan; a state file in the repository; `must be replaced` on a database, accepted anyway; credentials as literals; one monolithic state for the whole account.

## Walkthrough
1. Inventory honestly. List every cloud resource your team owns, including the ones created by clicking at 11 p.m. Mark each one: in `STACK.md`, or not. The unmarked ones are why you are here. Then pick the smallest safe starting point — a DNS record, or a free-tier object store bucket. Not the database.
2. Set up a free remote backend with state locking before you write a single resource. Confirm `.gitignore` excludes any local state file, and verify with the M01 check that no state file has ever been committed.
3. Have AI write the provider block and that one resource, with the region and the name as variables. Read every line and say what each one will create.
4. Run plan. Read all of it. Find the `Plan: N to add, N to change, N to destroy` line and say the three numbers out loud before you do anything else.
5. Apply, then confirm in the console that exactly the thing you expected exists and nothing else does.
6. Now create drift on purpose. Change that resource by hand in the console — rename a tag, flip a setting. Run plan again and read how the tool describes the difference between your code and reality.
7. Resolve the drift twice, deliberately. First decide the code is right and apply to overwrite the console change. Then make the change again, and this time decide reality is right and update the code so plan reports no changes. Write down which situation each response fits.
8. Import something. Take one resource that was created by clicking and bring it under management with an import, then run plan and confirm it reports no changes. That "no changes" is the proof you adopted it rather than duplicated it.
9. Add a destroy guard to your most important stateful resource, then deliberately write a change that would replace it and run plan. Confirm the plan *fails* instead of offering to destroy it. Screenshot that failure; it is the safety net you are buying.
10. Turn your resources into a module with variables, and instantiate it twice — staging and production — with separate state. Diff the two plans and confirm the only differences are the variables you intended.
11. Finally, wire plan into your pull-request pipeline from M06 chapter 12, so every infrastructure change arrives as a reviewable diff and no apply ever runs from a laptop.

## Talk to AI
```prompt
CONTEXT:      I am a student on an AI-native engineering program. I direct AI
              and verify its output rather than writing code by hand. My team
              of three runs [product, two sentences] with [number] real users,
              in India, on student budgets where a few thousand rupees of
              unexpected cloud spend is a serious problem. Our cloud resources
              today are: [list every one, including any created by clicking in
              a console and never written down]. We currently track them in a
              committed STACK.md file, which is out of date. Our state of
              knowledge: nobody has used Terraform before. Nobody is on call.

GOAL:         Bring our existing resources under infrastructure as code
              safely, with remote locked state, a plan reviewed on every pull
              request, and guards that make it impossible to casually destroy
              anything holding user data.

REQUIREMENTS: 1. Tell me, before anything else, the free options for a remote
                 state backend with LOCKING that a three-person team can use,
                 and what each one costs once we outgrow free
              2. Explain the state file in plain words: what it stores, why it
                 must never be committed, and exactly what goes wrong if two
                 people apply from two laptops with two local states
              3. Give me a safe ORDER of adoption: which of my resources to
                 bring under management first and which to leave until last,
                 and why the database is not first
              4. For each resource I listed that already exists, tell me
                 whether I should IMPORT it or recreate it, and give me the
                 read-only check that proves the import worked
              5. Show me how to read a plan: what the summary line means, what
                 `must be replaced` means for a database specifically, and the
                 exact words in the output I should search for before applying
              6. Give me the lifecycle guard that makes a destroy plan FAIL on
                 stateful resources, and tell me how to verify it actually
                 blocks a destroy
              7. Show me how the same module serves staging and production
                 with separate state and only variables differing
              8. Show me how plan runs on every pull request and apply runs
                 only on merge, so no apply ever comes from a laptop

CONSTRAINTS:  Free tier only, and say clearly where free is not possible.
              Plain English for a second-language reader. Label every command
              READ-ONLY or CHANGES-THINGS, and name the point of no return for
              each one. No credential may appear as a literal in any file -
              use variables backed by a secret store. Rupees for any cost.
              Assume we have live users, so no step may cause downtime without
              warning me first.

NON-GOALS:    Do not teach me HCL syntax as an exercise - I am going to READ
              these files, not type them. Do not introduce a service mesh, a
              Kubernetes operator, or a secrets-management product. Do not put
              our database under management in this first pass. Do not suggest
              `terraform destroy` as a cleanup step anywhere.

ACCEPTANCE:   - Our state is remote and locked, and no state file is in git
              - I can read a plan and state the add/change/destroy counts
              - Every stateful resource has a guard that makes destroy fail,
                and I have seen a plan fail because of it
              - Every resource created by clicking is either imported with a
                no-changes plan, or deliberately listed as out of scope
              - Staging and production come from one module and differ only in
                variables
              - A plan appears automatically on every pull request
              - No credential is a literal in any committed file

ASK:          Before any configuration, list the five ways a beginner team
              destroys production with this tool, and for each one the single
              guard that prevents it. Then give me the adoption order with the
              riskiest step named. No apply instructions until I confirm the
              order.
```

## Mission

**Objective.** Bring your own startup's existing cloud resources under infrastructure as code with remote locked state, prove you can detect and resolve drift in both directions, and prove with a failed plan that your database cannot be casually destroyed.

**Steps.**
1. **Frame** — list every resource you own, marking which are in `STACK.md` and which are not. The gap is your actual problem; write it down as a number.
2. **Backend first** — set up free remote state with locking before you declare any resource. Confirm no state file is in git, using the M01 history check.
3. **Brief** — use the Director's Brief above with your real inventory. Demand the five-ways-to-destroy-production list before any configuration.
4. **Direct, smallest first** — bring one low-risk resource under management. Read the plan in full. Record the three counts before applying.
5. **Import** — adopt at least one resource that was created by clicking. Prove it with a plan that reports no changes.
6. **Create drift deliberately** — change something by hand in the console, run plan, and screenshot how the difference is reported.
7. **Resolve drift both ways** — once by applying your code over the console change, once by updating your code to match reality. Write one sentence each on which situation calls for which.
8. **Prove the guard** — add a destroy guard to your most important stateful resource, then write a change that would replace it and run plan. Screenshot the failure.
9. **Two environments, one module** — instantiate staging and production from the same module with separate state, and diff the two plans.
10. **Pipeline** — wire plan into your pull-request checks so an infrastructure change arrives as a reviewable diff.
11. **Retire the lie** — update `STACK.md` to say which resources are now managed in code and which are deliberately still manual, with an owner for each remaining one.
12. **Document** — one decision-log entry: what you brought under code, what you deliberately left out, and the destroy guard you installed.

**Upload.**
- Your Director's Brief, in full → post caption or an attached file
- Your resource inventory with the managed/unmanaged gap counted → text
- Proof your state backend is remote and locked, and that no state file is in git history → screenshots
- A full plan output with the add/change/destroy summary line visible → screenshot
- The import, followed by a no-changes plan → two screenshots
- Drift detected after a console change → screenshot
- Your two drift resolutions, each with one sentence of justification → text
- The plan that **failed** because of your destroy guard → screenshot
- The diff between your staging and production plans → text
- A pull request showing an automatic plan → link or screenshot
- Your updated `STACK.md` and your decision-log entry
- Reflection, 150 words max → one tradeoff you chose, and one thing the AI got wrong or left out

**Self-check.** Tick every box before you submit.
- [ ] My state is remote and locked, not on anybody's laptop
- [ ] `git log --all` shows no state file has ever been committed
- [ ] I read a full plan and can state its three counts from memory
- [ ] At least one hand-created resource is imported, with a no-changes plan
- [ ] I created drift on purpose and recognised it in the plan output
- [ ] I resolved drift in both directions and can say which fits which case
- [ ] My database has a destroy guard, and I have seen a plan fail because of it
- [ ] No credential appears as a literal in any committed file
- [ ] Staging and production come from one module with separate state
- [ ] A plan runs automatically on every pull request
- [ ] A budget alert was already set before my first apply
- [ ] My reflection names something specific the AI got wrong

**Stretch.** Set up a GitOps reconciliation for one small piece of your infrastructure, then change it by hand in the console and record how long until the change is automatically reverted. Write 80 words on whether that behaviour would help or hurt a three-person team at 11 p.m. before a demo.

## Stuck? Ask AI
```prompt
Here is a Terraform plan output: [paste the complete output, including the summary line]. Do not apply anything and do not rewrite my configuration. Walk me through it resource by resource in plain language: what is being created, what is being changed in place, and what is being destroyed or replaced. For every destroy or replace, tell me what data lives inside that resource and what my users would experience. Finish with the single sentence I should say out loud before deciding whether to apply.
```

```prompt
Explain the Terraform state file to someone who has never used it, using an analogy that is not about maps, blueprints or recipes. Then walk me through exactly what happens, step by step, in three situations: two teammates apply the same code from two laptops with no shared state; someone commits the state file to git; and someone deletes a resource by hand in the console and then runs apply. Tell me which of the three is most likely to happen to a student team and why.
```

```prompt
I found a cloud resource that exists but is not in my Terraform code. Explain the difference between importing it, recreating it, and deleting it, and what each choice costs me. Then give me the read-only steps to identify what that resource is, what depends on it, and whether anything is currently using it - before I decide. Do not give me any command that changes state until I tell you which option I picked.
```

## Quiz

### Q1 · L3
Two teammates run the same Terraform code from two laptops, each with its own local state file. What happens?
- (a) The second run detects the first one's resources and makes no changes
- (b) The second run sees empty state, concludes nothing exists, and creates duplicate resources ✅
- (c) Terraform refuses to run without a remote backend
- (d) The state files merge automatically on the next apply
> State is the only link between your code and real resource ids. A state that has never seen a resource believes it does not exist. This is why remote state with locking is step one, not a later improvement.

### Q2 · L4
A plan says a database "must be replaced". What does that actually mean for your users?
- (a) The database will be upgraded in place with a brief pause
- (b) The database will be destroyed and a new empty one created, with a new address ✅
- (c) Only the resource's metadata changes; the data is untouched
- (d) Terraform will migrate the data automatically before switching over
> Replacement is destroy-then-create. You lose the data and the connection address changes. This is the single line in a plan most worth training yourself to find, and it is why stateful resources get a destroy guard.

### Q3 · L3
Why must the state file never be committed to git?
- (a) It is too large for most repositories
- (b) It contains resource ids and, for many providers, generated passwords in plain text ✅
- (c) Terraform refuses to read state from a git working directory
- (d) It changes on every run, which makes diffs noisy
> Committing it is a secret leak under M07's rules: revoke first, clean history afterwards. The noisy diffs are real but trivial next to the credentials.

### Q4 · L4
You discover a storage bucket that exists in the cloud but is not in your configuration. What is the correct response?
- (a) Delete it, since anything unmanaged is unsafe
- (b) Add it to the configuration and apply, creating a managed copy
- (c) Import it so state learns its id, then confirm the plan reports no changes ✅
- (d) Leave it alone and note it in `STACK.md`
> Import adopts the existing resource. Declaring it without importing creates a second bucket and leaves the first orphaned — which is exactly the mess M06's `STACK.md` was trying to prevent, now automated.

### Q5 · L5
Your team adopts infrastructure as code and the number of production incidents goes **up** in the first month. What is the most likely cause?
- (a) The tool is unsuitable for small teams
- (b) Applies are running from laptops with unread plans, so one tool now makes large changes fast ✅
- (c) Remote state introduces latency on every operation
- (d) Modules add abstraction that hides errors
> The tool does not reduce risk; the review habit does. A tool that creates forty resources from a file destroys forty resources from a file, which is why plan-in-a-pull-request and a destroy guard are the parts that actually buy you safety.

## Go deeper
- [Terraform: state, remote backends and locking](https://developer.hashicorp.com/terraform/language/state)
- [OpenTofu: getting started with a free, open-source Terraform-compatible CLI](https://opentofu.org/docs/intro/)
