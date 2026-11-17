---
id: m00-07-why-ai-gets-things-wrong
module: m00-orientation
slug: why-ai-gets-things-wrong
title: "Why AI Gets Things Wrong, and How You Catch It"
order: 7
duration_min: 15
is_free: false
mission: false
volatility: medium
last_reviewed: 2026-11-16
---

## Key learnings
- Name the five distinct ways AI output goes wrong
- Explain why confidence carries no information about correctness
- Recognise the specific failure modes that produce insecure or broken software
- Apply a four-step verification habit to any AI output
- Give feedback that fixes a problem instead of restarting it

## Lesson

### The pain

A student asks an AI to add "login with Google" to the canteen app. It produces clean, confidently-explained code, and it works.

Three weeks later anyone can log in as the canteen manager. The code checked that Google said "this is a real person" and never checked *which* person, or whether they were staff. The AI was not sloppy: it answered the question asked ("add login") rather than the question meant ("only staff reach the admin page").

### The idea

**The model produces fluent text whether or not it has grounds for it, because fluency and correctness come out of the same process.**

There is no internal flag saying "I am unsure here" — a guess and a certainty are written in the same voice. Every verification habit in this program follows from that.

### The analogy

A brilliant, extremely well-read intern on their first day. They have read every textbook, they write beautifully, they never say "I don't know" — and they have never seen your product, do not know your users, and cannot tell what they remember from what they are inferring. You would not merge that work unreviewed; you would also be foolish not to use them. Delegate freely, review always.

**Where the analogy breaks:** an intern learns. A model starts fresh every conversation unless you put the context back in front of it — which is why rules files exist.

### Picture it

```
     the model's output
             │
             ▼
  ┌──────────────────────┐
  │  sounds confident    │ ◀── always, regardless of correctness
  └──────────┬───────────┘
             │
   ┌─────────┴─────────┐
   ▼                   ▼
CORRECT            WRONG — but in one of five ways:
                     │
                     ├── invented something        → check it exists
                     ├── didn't know my situation  → put it in the brief
                     ├── knew an older version     → check current docs
                     ├── answered a different Q    → acceptance criteria
                     └── reasoned plausibly, badly → ask for 2 options
```

Confidence is on the branch above the split, not inside either side of it. This is why "it sounded sure" is never evidence.

The five shapes:

1. **Hallucination** — an invented function, option, library or price, always plausible because plausibility is what the model optimises for.
2. **Missing context** — textbook-correct for a project that is not yours: not your free tier, not your existing `users` table. The most common by far.
3. **Outdated knowledge** — confidently describing last year, because versions, APIs and free tiers all move after the training cutoff.
4. **The question asked, not the question meant** — the Google login story.
5. **Plausible-but-wrong reasoning** — sound-looking steps, wrong conclusion. The most dangerous, since it survives a casual read.

### Where it lives

These map onto specific traps later:

| Failure | Where it bites hardest |
|---|---|
| Missing context | M04 — a schema that ignores the data you already have |
| Question asked vs meant | M07 — authentication built without authorization |
| Outdated knowledge | M06 — deploy configuration for a platform version that changed |
| Plausible-but-wrong | M05 — caching advice that is fine at 10 users and wrong at 10,000 |
| Hallucination | M10 — invented model names, parameters and pricing |

### Words to own

| Term | Plain meaning | Example |
|---|---|---|
| Hallucination | Confident output that is simply invented | A function that does not exist |
| Knowledge cutoff | The date its training data ends | Why it quotes old pricing |
| Context | What it can see right now | Your brief plus files it read |
| Grounding | Giving it real data to work from | Pasting your actual schema |
| Acceptance criteria | Testable statements of "done" | "A student cannot open /admin" |
| Happy path | The case where nothing goes wrong | What it builds by default |
| Edge case | The unusual input that breaks things | Zero items; 500 items; emoji in a name |
| Regression | Something that used to work and now does not | Fixing the cart broke checkout |

### Tradeoffs and gotchas

**AI defaults to the shortest path, and secure defaults are rarely the shortest path.** The recurring shapes, all returning in M07:

- An API key in frontend code, where any user can read it
- Authentication without authorization — *who* you are, never *what you may do*
- Inputs trusted because the form validated them, but the form is on the user's device
- Error messages returning the database error verbatim, table names included
- Permissions enforced by hiding a button

All of these pass a casual "does it work?" test, because they do work — for the attacker too.

**It will agree with you when you are wrong.** Push back on a correct answer and it often folds into a worse one — and folding feels like you won the argument. Ask "am I wrong about this?" instead.

**Working is not the same as correct.** The broken Google login worked. "I tested it and it ran" covers the happy path only. Test the empty case, the huge case, the wrong-user case and the repeat case.

**Long conversations drift.** Thirty messages in, the constraints from message two are out of the window. Restating them is how the tool works; if a thread has gone sideways, start fresh.

### Verify it

The habit:

1. **Does it exist?** Every library, function, option and price checked against real docs.
2. **Does it fit my situation?** Re-read your constraints — free tier, existing data, users, region — and confirm each survived.
3. **Does it satisfy what I meant?** Test every acceptance criterion, including the one you did not ask for, as the wrong kind of user.
4. **Can it explain itself consistently?** "Why this approach rather than the alternative, and where would it stop working?" Inconsistency signals guessing more strongly than the output does.

Then give feedback that fixes rather than restarts:

| Weak | Strong |
|---|---|
| "It's broken" | "The cart total ignores quantity: 3 items at ₹40 shows ₹40, expected ₹120" |
| "Make it secure" | "A logged-in student can open /admin. Only users with role=staff should get past it" |
| "That's wrong" | "This assumes Postgres; I'm on the Supabase free tier. Does that change your answer?" |
| "Do it properly" | "Fix only the quantity calculation. Do not change anything else." |

Symptom, expected versus actual, one change at a time. The last matters more with agents: one asked to fix three things will touch thirty files.

## Walkthrough
1. Ask an AI something you already know the answer to and that is specific enough to be checkable — a detail of your college's rules, the fee structure of a scheme you know, a fact about your city. Note how confident the wrong answer sounds.
2. Now ask for something versioned: "what is the current free-tier limit for [a service you use]?" Check it against the actual pricing page. This demonstrates the knowledge-cutoff failure on something that matters commercially.
3. Ask it to design a database table for storing canteen orders. Then tell it you already have a `users` table with particular columns and ask again. Compare the two answers — the gap between them is the cost of missing context.
4. Deliberately provoke the agreement failure. Take a correct answer and insist it is wrong, without giving a reason. Note whether it defends itself or folds. Then ask the same thing as "am I wrong about this?" and compare.
5. Ask for "two approaches to storing uploaded dish photos, with the argument against each, and the point at which each stops working." Notice how much more useful the comparison is than a single recommendation.
6. Take any code it produced and ask it to find the security problems in its own output. It will often find real ones. This is worth internalising: it can critique better than it can originally produce.
7. Write down the one failure mode from this chapter you think you are personally most likely to miss, and why. Keep it where you will see it during missions.

## Talk to AI
```prompt
I want you to review your own previous answer as a hostile critic rather than as its author.

The answer I want reviewed is: [paste it]

My actual situation is: [stack, free tier limits, existing data, who the users are, region]

Go through these in order:
1. Anything you named — a library, function, option, price, limit — that you are not certain exists. List it and say how I would verify it.
2. Anything that assumes a situation different from mine. Be specific about which assumption.
3. Anything that may have changed since your training data ends.
4. Anything I asked for that this does NOT actually achieve, including things I should have asked for and did not.
5. Any security problem: exposed keys, missing authorization, trusted client input, leaked error details.
6. The point at which this approach stops working, and what breaks first.

End with the single most likely way this goes wrong in production. Do not reassure me.
```

## Stuck? Ask AI
```prompt
Give me five short pieces of AI-generated advice about building a web app. Make some of them subtly wrong — plausible reasoning with a flawed conclusion. Ask me to spot which are wrong and why. Do not tell me the answers until I have committed to all five.
```

```prompt
Explain the difference between authentication and authorization using the broken "login with Google" story from my lesson, where anyone could log in as the manager. Then give me three other examples of the same mistake in different kinds of app.
```

```prompt
I wrote this feedback to an AI agent: [paste your feedback]. Rewrite it as a precise bug report with symptom, expected versus actual, and a clearly limited scope. Then explain what was wrong with my version.
```

## Quiz

### Q1 · L3
An AI adds "login with Google" and it works, but any logged-in student can then open the admin page. Which failure mode is this?
- (a) Hallucination — it invented a login method
- (b) Outdated knowledge — the OAuth standard changed
- (c) It answered the question asked rather than the question meant ✅
- (d) Plausible-but-wrong reasoning about cryptography
> "Add login" was satisfied exactly. Authorization was never requested, so it was never built — which is why acceptance criteria state what must be true, not what to add.

### Q2 · L4
Which of the five failure modes is the most common in day-to-day work?
- (a) Hallucination, because invented functions are frequent
- (b) Missing context — a textbook-correct answer that does not fit your project ✅
- (c) Outdated knowledge about library versions
- (d) Plausible-but-wrong reasoning
> Hallucinations usually fail loudly and immediately. Missing context produces answers that run perfectly and are still wrong for your free tier, your existing data or your users.

### Q3 · L4
You tell an AI its correct answer is wrong, without giving a reason. What most often happens?
- (a) It explains why it was right and holds its position
- (b) It agrees and produces a worse answer ✅
- (c) It asks you for evidence before changing anything
- (d) It repeats the identical answer
> Models tend to defer to the user. This is especially dangerous because folding feels like you were right. "Am I wrong about this?" is the better phrasing.

### Q4 · L4
Which feedback is most likely to get a correct fix from a coding agent?
- (a) "The cart is broken, please fix it"
- (b) "Make the cart calculation more robust"
- (c) "Cart total ignores quantity: 3 items at ₹40 shows ₹40, expected ₹120. Fix only this calculation." ✅
- (d) "This is wrong, do it properly this time"
> Symptom, expected versus actual, and a bounded scope. The scope limit matters particularly with agents, which will otherwise touch far more than you intended.

### Q5 · L4
Why is "I tested it and it worked" insufficient evidence that AI output is correct?
- (a) Testing is unreliable on free-tier servers
- (b) It only exercises the happy path, while the failures are in edge cases and wrong-user cases ✅
- (c) AI output cannot be tested in the normal way
- (d) It proves nothing unless you also read every line
> The broken login worked. An exposed API key works. Verification has to include the empty case, the oversized case, the wrong-user case and the repeated case.

## Go deeper
- [OWASP Top 10 for LLM Applications](https://owasp.org/www-project-top-10-for-large-language-model-applications/)
- [Anthropic: Claude's constitution and limitations](https://www.anthropic.com/news/claudes-constitution)
