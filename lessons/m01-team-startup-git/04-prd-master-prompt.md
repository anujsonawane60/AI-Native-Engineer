---
id: m01-04-prd-master-prompt
module: m01-team-startup-git
slug: prd-master-prompt
title: "The PRD Is the Master Prompt"
order: 4
duration_min: 25
is_free: false
mission: true
volatility: low
last_reviewed: 2026-11-16
---

## Key learnings
- Write a one-page PRD that an AI can build from without inventing context
- Turn interview facts into constraints rather than adjectives
- State non-goals, and understand why they are the most valuable section
- Write acceptance criteria for a product rather than a feature
- Keep the PRD alive as the product changes

## Lesson

### The pain

A team finishes their interviews with good evidence and a clear persona, then starts building — and the evidence stays in their heads.

Every AI conversation begins from nothing. One member briefs an agent assuming a ₹60 price ceiling; another assumes payments are out of scope. The agent is told three different things about one product in a week and builds three incompatible pieces. By week three they are arguing about what they agreed, and nobody can prove anything.

### The idea

**A PRD is the Director's Brief from M00, written once for the whole product instead of once per task.**

Same fields, larger scope. Its job is to stop you re-explaining your product twenty times, and to stop AI inventing the parts you did not mention.

### The analogy

A canteen stall has a board behind the counter that no customer sees: today's menu, the prices, the limited items, what the stall does not do. With the board, a new helper is useful in five minutes. Without it they guess at prices and confidently tell a student the stall does home delivery, because that seemed reasonable. An AI agent is that helper, arriving fresh every conversation. The PRD is the board.

**Where the analogy breaks:** a human helper eventually learns the stall. An agent never does, so the board gets pasted in every time — which is why it fits on one page, and why in M10 you turn it into a rules file the agent reads automatically.

### Picture it

```
      interviews                 PRD                   every brief
  ┌────────────────┐      ┌──────────────┐      ┌──────────────────┐
  │ 5 conversations│      │ PROBLEM      │      │ CONTEXT:         │
  │ exact quotes   │─────▶│ USER         │─────▶│  paste PRD here  │
  │ workarounds    │      │ GOAL         │      │ GOAL: this task  │
  │ money, timing  │      │ MUST HAVE    │      │ ACCEPTANCE: ...  │
  └────────────────┘      │ CONSTRAINTS  │      └──────────────────┘
                          │ NON-GOALS    │                │
                          │ SUCCESS      │                ▼
                          └──────┬───────┘        consistent output
                                 │                 across the team
                     revised when evidence changes
```

Facts flow in from the interviews, get written once, and flow out into every brief anyone writes. The PRD is a cache for your product decisions, and like any cache it must be invalidated deliberately.

Each section blocks a known failure: USER and CONSTRAINTS block missing context, MUST HAVE blocks three people building different products, NON-GOALS blocks an agent rewriting what you never asked about, SUCCESS blocks shipping something that works and changes nothing.

```
PROBLEM      Two sentences, no solution in it. From PROBLEM.md.
USER         Your persona: money, time window, current substitute.
GOAL         What it does, in one sentence a stranger gets.
MUST HAVE    Numbered. More than six means you are not scoping.
CONSTRAINTS  Stack, free tier, device, network, money, legal, language.
NON-GOALS    What it does NOT do, and what nobody may touch.
SUCCESS      How you know it worked: a number and a date.
```

### Where it lives

The PRD is the CONTEXT section of every brief from now on. You paste it in; by M10 your agent reads it from a rules file. It feeds forward:

- **M03** — the persona's device, network and money become design constraints
- **M04** — MUST HAVE decides the data model; a feature not in the PRD is not in the schema
- **M05** — SUCCESS numbers set the load you plan for
- **M11** — Checkpoint 1 is the PRD plus an architecture diagram
- **M16/M17** — SUCCESS is what you measure

### Words to own

| Term | Plain meaning | Example |
|---|---|---|
| PRD | Product Requirements Document; the one-page spec | Your master prompt |
| Must-have | Without this, the product is pointless | "A student can place an order" |
| Nice-to-have | Real value, deliberately deferred | "Order history" |
| Non-goal | Explicitly excluded, on purpose | "No delivery" |
| Constraint | Something true about your situation | "Free tier only" |
| Acceptance criteria | Testable statements of done | "Completes an order in under 60s" |
| North star metric | The one number that means it is working | Weekly repeat orders |
| MVP | The smallest version that delivers the value | Not the smallest version that compiles |
| Living document | Revised as evidence arrives, with history | Committed to the repo |

### Tradeoffs and gotchas

**Facts, not adjectives.** The left column is agreeable and gives an AI nothing; the right makes decisions on your behalf for six months, which is what a specification is for.

| Adjective version | Fact version |
|---|---|
| Users are budget-conscious | Users have ₹40–60 per day; delivery apps are unusable below ₹150 minimum |
| Must be fast | Must be usable on 4G in under 3 seconds on a ₹8,000 Android phone |
| Should feel simple | A first-time user completes an order in under 60 seconds without help |
| Must be secure | Only users with role=staff can change availability; prices are recalculated server-side |
| Mobile-friendly | Usable one-handed at 320px wide, no horizontal scroll |

**NON-GOALS holds three kinds of line, and beginners write one where experts write six.** *Not now* ends the weekly argument about deferred features. *Not ever* stops the drift into a worse product. *Do not touch* is the agent's scope limit — "do not change the schema without a migration" — and keeps a one-hour task from becoming a three-hour recovery.

**One page, hard limit.** A ten-page PRD never gets pasted into a brief and is stale in a fortnight. If it does not fit, MUST HAVE is a wish list.

**Write SUCCESS before MUST HAVE, with a number and a date.** The same inversion as acceptance-before-requirements in M00: working backwards from "twenty repeat orders by 15 February" turns eleven must-haves into four.

**Revise it, with history.** Week five will invalidate a must-have. Change it and commit — "we changed this because of what we measured" beats a PRD that never moved.

**Do not let AI write it from nothing.** Ask for a PRD for "a canteen app" and you get the generic product you avoided in M00. Supply the facts; let it structure and critique them.

### Verify it

Six checks before you accept your own PRD.

1. **Does it fit on one page?** If not, cut MUST HAVE.
2. **Is every USER line traceable to a quote?** Anything else is imagination hardening into a requirement.
3. **Are any adjectives doing a fact's job?** Search for fast, simple, easy, secure, intuitive, seamless.
4. **Could you remove any must-have and still have a product?** Then move it.
5. **Does SUCCESS have a number and a date?**
6. **Does NON-GOALS include a "do not touch"?** Without it your first agent task is unbounded.

Red flag: an outsider reads it and describes a different product. Your AI will get it wrong the same way, but only the human will tell you which sentence misled them.

## Walkthrough
1. Open your `PROBLEM.md` and your persona from the last chapter. Everything in the PRD comes from these two, plus your constraints. Nothing new gets invented here.
2. Write SUCCESS first. One number, one date. "Fifteen students place a second order within two weeks of launch." Resist writing a vanity number like signups.
3. Write GOAL in one sentence that a stranger would understand, with no technology in it.
4. Now write MUST HAVE, working backwards from SUCCESS. For each candidate, ask whether SUCCESS is still achievable without it. If yes, it is a nice-to-have.
5. Count your must-haves. More than six means go back to step four and be harder.
6. Write CONSTRAINTS. Go through your interview notes and extract every fact about money, device, network, timing and language. Add your technical constraints: free tier, which services, which region.
7. Write NON-GOALS in three groups: not now, not ever, do not touch. Aim for more lines here than in MUST HAVE.
8. Scan the whole page for adjectives. Replace each with a number or a testable statement. This single pass usually improves the document more than anything else.
9. Ask AI to critique it — not to rewrite it. Use the prompt below. Fix what it finds that you agree with.
10. Hand it to one person outside your team. Ask them to tell you what you are building and who for. Fix whatever they got wrong.
11. Commit it as `PRD.md`. From now on, this is the CONTEXT of every brief any of the three of you writes.

## Talk to AI
```prompt
I have written a one-page PRD and I want you to review it as a hostile critic, not rewrite it.

Here is my PRD:
[paste the whole thing]

Go through these in order and be specific, quoting my own words back to me:
1. Adjectives doing a fact's job. Quote every instance of fast, simple, easy, secure, intuitive, mobile-friendly or similar, and tell me what number or testable statement should replace each.
2. Must-haves that are not. For each one, tell me whether my SUCCESS criterion is still achievable without it. Name the ones I should move to nice-to-have.
3. Missing constraints. Based on my USER section, what is almost certainly true about my users' situation that I have not written down - and that you would otherwise have invented?
4. Missing non-goals. What would an AI agent plausibly build or modify that I have not forbidden?
5. The SUCCESS criterion. Is it a vanity number? Does it have a date? Would it still be true if my product were useless?
6. Contradictions. Anything in MUST HAVE that conflicts with CONSTRAINTS or NON-GOALS.

Finally: based only on this PRD, describe the product back to me in three sentences. If your description is not what I meant, tell me which specific sentence of mine misled you.

Do not rewrite my PRD and do not write code.
```

## Mission

**Objective.** A one-page PRD, built from your interview evidence, that any of the three of you can paste into a brief and get consistent output from — reviewed by AI, tested on a human, and committed.

**Steps.**
1. **Frame** — write SUCCESS first: one number, one date.
2. **Draft** — fill the seven sections from `PROBLEM.md` and your persona. Nothing invented.
3. **Cut** — remove every must-have whose absence still allows SUCCESS. Get to six or fewer.
4. **De-adjective** — replace every fast, simple, secure and mobile-friendly with a number or a test.
5. **Forbid** — write non-goals in three groups, including at least one "do not touch".
6. **Critique** — run the AI review prompt. Fix what you agree with; note what you rejected and why.
7. **Test on a human** — one person outside the team describes your product back. Fix what they got wrong.
8. **Prove it works as a prompt** — paste the PRD as CONTEXT into one real brief and get something built. This is the actual test of a PRD.
9. **Commit** — `PRD.md` in the repository.

**Upload.**
- Your final `PRD.md`, all seven sections
- Your first draft alongside it, so the cuts are visible → both files or a diff screenshot
- The AI critique conversation → share link in `ai_chat_url`
- One thing the AI flagged that you disagreed with, and why, in two sentences
- What your outside reader said the product was, before you corrected anything
- One real brief that used the PRD as CONTEXT, plus what got built → `demo_url` or screenshots
- Reflection, 150 words max → which must-have you cut and why it hurt, and one constraint you had only in your head

**Self-check.** Tick every box before you submit.
- [ ] It fits on one page
- [ ] Every USER line traces to a quote from a real conversation
- [ ] No adjective is doing a fact's job anywhere on the page
- [ ] Six or fewer must-haves, and removing any one breaks the product
- [ ] NON-GOALS has more lines than MUST HAVE
- [ ] NON-GOALS includes at least one "do not touch"
- [ ] SUCCESS has a number and a date, and is not a signup count
- [ ] An outside reader described the product correctly
- [ ] I used it as CONTEXT in a real brief and something got built
- [ ] `PRD.md` is committed, and all three of us know to paste it

**Stretch.** Give the same PRD to two different AI tools with an identical GOAL, and compare what each builds. Differences are the ambiguity still left in your PRD. Write 100 words naming the sentence that caused the biggest divergence.

## Stuck? Ask AI
```prompt
My MUST HAVE list has eleven items and I cannot cut it. Here it is, with my SUCCESS criterion: [paste both]. For each item, tell me whether SUCCESS is still achievable without it. Do not be diplomatic — I need this list to be four items.
```

```prompt
Explain the difference between a nice-to-have and a must-have using one product example, then show me the test I should apply. Then give me eight features for a canteen app and ask me to sort them, without telling me the answers first.
```

```prompt
I wrote this SUCCESS criterion: [paste it]. Tell me whether it is a vanity metric, whether it would still be true if my product were useless, and what I should measure instead. Explain why signups are usually the wrong number.
```

## Quiz

### Q1 · L3
What is a PRD, in terms of what you learned in M00?
- (a) A technical design document describing the architecture
- (b) The Director's Brief written once for the whole product instead of per task ✅
- (c) A marketing document describing the product's benefits
- (d) A list of every feature the product will ever have
> Same fields and same purpose, at product scope. Its job is to stop you re-explaining the product and to stop the AI inventing what you did not mention.

### Q2 · L4
Which line belongs in a PRD?
- (a) Users are budget-conscious and value convenience
- (b) The app must be fast and feel simple to use
- (c) Users have ₹40–60 per day; delivery apps are unusable below a ₹150 minimum ✅
- (d) The product should provide a seamless mobile experience
> Facts make decisions on your behalf for six months. Adjectives are agreeable and give an AI nothing to work from, so it invents the specifics.

### Q3 · L4
Why is NON-GOALS usually the most valuable section once a real codebase exists?
- (a) It shortens the PRD so it fits on one page
- (b) It bounds what an agent may build or modify, preventing unscoped changes ✅
- (c) It is required by the platform's import format
- (d) It documents features for the next version's marketing
> It is the product-scale version of the NON-GOALS line in a brief — the thing that stops a one-hour task becoming a three-hour recovery across many files.

### Q4 · L4
Why write SUCCESS before MUST HAVE?
- (a) It is easier to write and builds momentum
- (b) Starting from the outcome reveals which features are actually required, and usually shortens the list ✅
- (c) The AI reads the sections in that order
- (d) Because must-haves cannot be written until the architecture is known
> The same inversion as acceptance-criteria-before-requirements in M00. Working backwards from a measurable outcome exposes wish-list items as nice-to-haves.

### Q5 · L4
What happens if you ask AI to write your PRD from scratch for "a canteen app"?
- (a) A strong first draft you can refine, since it has seen many similar products
- (b) A plausible average of every similar product, built for nobody you interviewed ✅
- (c) It will refuse, because it lacks user research
- (d) A PRD that is technically correct but badly formatted
> This is the generic-output failure from M00. Supply your own facts and let AI structure and critique them; it cannot supply evidence it does not have.

## Go deeper
- [Shape Up — pitches and appetite](https://basecamp.com/shapeup/1.5-chapter-06)
- [Amazon's working-backwards method and the PR/FAQ](https://www.productplan.com/glossary/working-backward-amazon-method/)
