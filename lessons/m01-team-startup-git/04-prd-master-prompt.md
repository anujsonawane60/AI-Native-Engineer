---
id: m01-04-prd-master-prompt
module: m01-team-startup-git
slug: prd-master-prompt
title: "The PRD Is the Master Prompt"
order: 4
duration_min: 35
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

A team finishes their user interviews with good evidence and a clear persona. Then they start building, and the evidence stays in their heads.

Every AI conversation begins from nothing. One member briefs an agent assuming a ₹60 price ceiling; another assumes payments are out of scope entirely. The agent is told three different things about the same product in one week and produces three incompatible pieces, each internally sensible. By week three they are arguing about what they agreed, and nobody can prove anything because it was never written down.

The fix is one page. Not a document nobody reads — a page you paste into every brief for the next six months.

### The idea

**A PRD is the Director's Brief from M00, written once for the whole product instead of once per task.**

Same fields, same purpose, larger scope. Its job is to stop you re-explaining your product from scratch twenty times, and to stop the AI inventing the parts you did not mention.

### The analogy

A canteen stall has a board behind the counter that nobody shows to customers: today's menu, the prices, which items are limited, what the stall does not do.

A new helper arrives for lunch rush. If that board exists, they are useful in five minutes. If it does not, they must ask about every order, they will guess at prices, and they will confidently tell a student the stall does home delivery because it seemed like a reasonable thing to offer.

An AI agent is that new helper, arriving fresh every single conversation. The PRD is the board.

**Where the analogy breaks:** a human helper eventually learns the stall and stops needing the board. An agent never does. The board has to be pasted in every time — which is exactly why it must fit on one page, and why in M10 you will turn it into a rules file the agent reads automatically.

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

Facts flow in from the interviews, get written once, and flow out into every brief anyone writes. The PRD is a cache for your product decisions — and like any cache, it goes stale and must be invalidated deliberately.

### The one-page PRD

Seven sections. Each one blocks a specific failure you already know about.

```
PROBLEM      One or two sentences, no solution in it. From PROBLEM.md.

USER         Your evidence-based persona. Money, time window, current
             substitute. Every line traceable to a quote.

GOAL          What the product does, in one sentence a stranger gets.

MUST HAVE     Numbered. The smallest set without which the product is
              pointless. If you have more than six, you are not scoping.

CONSTRAINTS   Stack, free tier, device, network, money, legal, language.
              Everything true about your situation that AI would
              otherwise invent.

NON-GOALS     What this product does NOT do, and what nobody may touch.
              The longest section in a good PRD.

SUCCESS       How you will know it worked, with a number and a date.
```

| Section | Blocks |
|---|---|
| USER, CONSTRAINTS | missing context — the most common AI failure |
| MUST HAVE | scope creep, and three people building different products |
| NON-GOALS | an agent helpfully rewriting things you did not ask about |
| SUCCESS | building something that works and changes nothing |

### Facts, not adjectives

The single biggest difference between a PRD that works and one that reads nicely.

| Adjective version | Fact version |
|---|---|
| Users are budget-conscious | Users have ₹40–60 per day; delivery apps are unusable below ₹150 minimum |
| Must be fast | Must be usable on 4G in under 3 seconds on a ₹8,000 Android phone |
| Should feel simple | A first-time user completes an order in under 60 seconds without help |
| Must be secure | Only users with role=staff can change availability; prices are recalculated server-side |
| Mobile-friendly | Usable one-handed at 320px wide, no horizontal scroll |

The left column is agreeable and gives an AI nothing. The right column makes decisions on your behalf for the next six months, which is what a specification is for.

### Non-goals are the most valuable section

Beginners write one line of non-goals and six lines of must-haves. Experienced teams do the opposite.

Three kinds belong here:

**Not now.** Features you have deliberately deferred. "No group ordering in v1." Writing them down stops the weekly argument and reassures whoever wanted them that they were heard rather than ignored.

**Not ever.** Things that are off-strategy. "We do not do delivery." This prevents the slow drift into becoming a different, worse product.

**Do not touch.** The scope limit for agents, which becomes critical the moment you have a real codebase. "Do not modify the auth flow", "do not change the schema without a migration." You learned this in M00 as the NON-GOALS line in a brief; at product level it is what keeps a one-hour task from becoming a three-hour recovery.

### Where it lives

The PRD is the CONTEXT section of every brief you write from now on. Practically, you paste it in — or by M10, your agent reads it from a rules file automatically, which is the same idea with less typing.

It also feeds forward concretely:

- **M03** — the persona's device, network and money decide your design constraints
- **M04** — MUST HAVE decides your data model; a feature not in the PRD should not be in the schema
- **M05** — SUCCESS numbers tell you what load to plan for
- **M11** — Checkpoint 1 is the PRD plus an architecture diagram
- **M16/M17** — the SUCCESS metric is what you actually measure against

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

**One page, hard limit.** A ten-page PRD does not get pasted into briefs, does not get read by your teammates, and will be out of date in a fortnight. If it does not fit, your MUST HAVE list is a wish list.

**MUST HAVE is not "everything we will eventually build."** The test: remove any one item. If the product still delivers the core value, that item is a nice-to-have. Apply this ruthlessly and most teams go from eleven items to four.

**Write SUCCESS before MUST HAVE.** The same inversion as acceptance-before-requirements in M00. Starting from "twenty students place a repeat order within two weeks of launch" tells you which features are actually required, and usually shortens the list.

**A number without a date is not a target.** "Twenty repeat orders" is a hope. "Twenty repeat orders by 15 February" is checkable, and in M11 you will be checked against it.

**Revise it, with history.** Your PRD will be wrong — you will learn something in week five that invalidates a must-have. Change it and commit the change. The diff is evidence of learning, and in M18's defence "we changed this because of what we measured" is a far stronger answer than a PRD that never moved.

**Do not let AI write your PRD from nothing.** Ask it to draft from your interview notes and it will do a reasonable job of structuring what you supply. Ask it to invent a PRD for "a canteen app" and you get the generic product you specifically avoided in M00 — a plausible average of every similar product, built for nobody you interviewed. Supply the facts; let it structure and critique them.

### Verify it

Before you accept your own PRD, six checks.

1. **Does it fit on one page?** If not, cut MUST HAVE.
2. **Is every USER line traceable to a quote?** Anything else is your imagination hardening into a requirement.
3. **Are there any adjectives doing a fact's job?** Search for fast, simple, easy, secure, intuitive, seamless. Replace each with a number or a testable statement.
4. **Could you remove any must-have and still have a product?** Then move it.
5. **Does SUCCESS have both a number and a date?**
6. **Does NON-GOALS include at least one "do not touch"?** Without it, your first agent task on a real codebase will be unbounded.

Then run the real test: **hand your PRD to someone outside the team and ask them to describe what you are building and who for.** If they get it wrong, your AI will get it wrong in the same way — and unlike the AI, the human will tell you which sentence confused them.

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
