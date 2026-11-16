---
id: m00-08-directors-loop
module: m00-orientation
slug: directors-loop
title: "The Director's Loop and the Director's Brief"
order: 8
duration_min: 35
is_free: false
mission: true
volatility: low
last_reviewed: 2026-11-16
---

## Key learnings
- Run the seven steps of the Director's Loop on a real task
- Write a Director's Brief that produces usable output first time
- Ask for architecture and options before asking for anything built
- Write acceptance criteria that are actually testable
- Keep a decision log that survives the conversation

## Lesson

### The pain

Two students get the same thirty minutes with the same AI and the same goal: a page where canteen staff can mark dishes as sold out.

The first types "make a page for canteen staff to manage the menu". They get a page. It is generic, it lets anyone edit, the sold-out state is not stored anywhere, and when they ask for changes the AI rewrites the whole thing and breaks what worked. Thirty minutes later they have something that demos and cannot be used.

The second spends eight minutes writing down what exists, what they want, what must not happen, and how they will know it works. They ask for two approaches before any code. They pick one, build it in three steps, and check each one. Thirty minutes later they have a working page, they can explain every decision, and the next change takes four minutes instead of another thirty.

Same tool. Same model. The difference is entirely in the method, and the method is learnable in one chapter.

### The idea

**Directing AI is a loop with seven steps, and the expensive mistake is starting at step four.**

```
1. FRAME      What problem, for whom, what does "done" look like?
2. SPECIFY    The brief: context, requirements, constraints, non-goals
3. ARCHITECT  Ask for options and tradeoffs BEFORE asking for code
4. DIRECT     Build in small, verifiable steps
5. VERIFY     Test it, inspect it, break it, ask it to explain itself
6. ITERATE    Precise feedback: symptom, expected vs actual, one change
7. DOCUMENT   Record what you chose and why
```

Most people begin at DIRECT, skip VERIFY, and have nothing at DOCUMENT. Steps 1–3 feel like delay and are where the time is actually saved.

### The analogy

A film director does not operate the camera, hold the boom, or act. They decide what the scene must achieve, who does what, which of two staging options is better, and — crucially — they watch the playback and say precisely what to change.

A director who says "make it more cinematic" gets nothing useful. One who says "the actor's face is in shadow on the left; move the key light two feet right and keep everything else" gets the shot.

You are the director. The AI is a crew that is fast, tireless, technically skilled, and has not read the script.

**Where the analogy breaks:** a film crew pushes back when a direction is physically impossible. The AI will attempt almost anything you ask, including the unwise, and report success. The playback review in step 5 is therefore not optional.

### Picture it

```
  FRAME ──▶ SPECIFY ──▶ ARCHITECT ──▶ DIRECT ──▶ VERIFY
    │                        │                      │
    │                        │          fails?      │
    │                        │          ◀───────────┘
    │                        ▼               │
    │                  pick an approach      ▼
    │                                     ITERATE
    │                                   (one change)
    │                                        │
    └──────── goal changed? ◀────────────────┤
                                             ▼
                                         DOCUMENT
```

Most loops run between DIRECT, VERIFY and ITERATE. Returning all the way to FRAME means the goal itself was wrong — which is useful information, not failure.

### The Director's Brief

The template. Every field earns its place by preventing a specific failure from the previous chapter.

```
CONTEXT:      What exists today, who the users are, what stack
GOAL:         What I want built or changed, in one sentence
REQUIREMENTS: Must-haves, numbered
CONSTRAINTS:  Stack, budget, performance, security, free-tier limits
NON-GOALS:    What NOT to do, and what NOT to touch
ACCEPTANCE:   How I will know it works — testable checks
ASK:          "2 approaches with tradeoffs first. No code until I pick."
```

Which failure each field blocks:

| Field | Blocks |
|---|---|
| CONTEXT | missing context — the most common failure of all |
| CONSTRAINTS | answers that assume a paid tier or a different database |
| NON-GOALS | the agent helpfully rewriting forty files |
| ACCEPTANCE | answering the question asked instead of the question meant |
| ASK | plausible-but-wrong reasoning, by forcing comparison |

### Vague versus precise

**Vague:**

> Make a page for canteen staff to manage the menu.

**Precise:**

```
CONTEXT:      Next.js on Vercel free tier, Supabase free tier for data.
              Existing tables: users (id, email, role), menu_items
              (id, name, price_paise, is_available). Users are canteen
              staff on shared Android phones, often on slow 4G.

GOAL:         Let canteen staff mark a dish sold out, and have students
              see that within a minute.

REQUIREMENTS: 1. Staff see a list of today's dishes with a toggle each
              2. Toggling updates is_available in the database
              3. The student menu reflects the change within 60 seconds
              4. The toggle shows a pending state while saving
              5. If the save fails, the toggle reverts and says so

CONSTRAINTS:  Free tier only. Must work on a 5-inch screen, one-handed.
              Must work on slow 4G. Prices are stored in paise, never
              rupees. Do not introduce any new paid service.

NON-GOALS:    No editing of names or prices. No image upload. No new
              tables. Do not modify the student-facing menu page.

ACCEPTANCE:   - Staff toggles a dish; is_available flips in the DB
              - A student reloading within 60s sees the change
              - A logged-in STUDENT who opens this page is refused
              - With the network throttled, the toggle shows pending
              - With the network cut, the toggle reverts and warns

ASK:          Propose 2 approaches with tradeoffs, especially how
              students get the update. No code until I pick one.
```

Twelve lines of thinking. The difference between them is the difference between the two students.

Notice acceptance criterion three. Nobody asked for it in the goal. It is the Google login bug from the last chapter, caught in advance by writing down what must be *true* rather than what to build.

### Where it lives

The loop runs at every scale. In M01 you will write a PRD, which is this brief for the whole product. In M10 you will write a rules file, which is the CONTEXT and CONSTRAINTS sections made permanent so you stop retyping them. In M11's checkpoints, your decision log is the DOCUMENT step collected up.

Two fields deserve extra attention as your projects grow:

**NON-GOALS** becomes the most valuable line in the brief once a real codebase exists. "Do not modify the student-facing menu page" is what stops a one-hour task becoming a three-hour recovery.

**ASK** is the step most people skip and the one with the best return. Forcing two options with tradeoffs does three things at once: it surfaces an approach you had not considered, it exposes shaky reasoning through comparison, and it puts the decision — which is your job — back in your hands.

### Words to own

| Term | Plain meaning | Example |
|---|---|---|
| Brief | The structured instruction you give AI | The template above |
| Requirement | Something that must be present | "Toggling updates the database" |
| Constraint | A limit on how it may be done | "Free tier only" |
| Non-goal | Something deliberately excluded | "Do not touch the menu page" |
| Acceptance criteria | Testable statements of done | "A student is refused" |
| Scope | How much a task is allowed to change | One component, not the app |
| Decision log | A record of choices and reasons | Why polling, not realtime |
| ADR | Architecture Decision Record; a formal decision log entry | One page per decision |

### Tradeoffs and gotchas

**Do not brief a trivial task.** "Centre this heading" needs no CONTEXT block. The brief earns its cost when the task touches data, money, permissions, or anything that already exists. Over-formalising small changes is its own waste.

**Acceptance criteria must be testable by someone else.** "Works well on mobile" is an opinion. "Usable one-handed on a 5-inch screen without horizontal scrolling" is a test. If you cannot hand it to a teammate and have them reach the same verdict, rewrite it.

**Write the acceptance criteria before the requirements.** Counter-intuitive and effective. Starting from "how will I know this worked?" exposes requirements you would have forgotten — which is precisely how criterion three above appeared.

**One change at a time, especially with agents.** An agent asked to fix three things will often touch thirty files, and then you cannot tell which change broke what. Sequential small steps feel slower and are reliably faster.

**The decision log is for you in three weeks.** Not for marks, not for your mentor. In M11 you will be asked why you chose polling over realtime updates, and either you wrote down "polling: 60s delay is acceptable for a menu, realtime needs a paid tier" or you will invent something and sound like you do not understand your own product.

### Verify it

Before you send a brief, check four things:

1. **Is every constraint that matters written down?** Free tier, existing tables, the device, the network. If it is only in your head, it is not in the brief.
2. **Could someone else test my acceptance criteria and get the same answer?** If not, they are opinions.
3. **Have I said what not to touch?** On anything beyond a greenfield page, this line prevents the most expensive failure.
4. **Am I asking for options, or for code?** If the task has more than one reasonable approach, ask for options.

After output arrives, run step 5 properly: test each acceptance criterion including the one nobody asked for, try it as the wrong kind of user, and ask it to explain its choice against the alternative.

## Walkthrough
1. Pick the task: a one-page canteen menu showing six dishes with name, price and a photo, plus a "sold out" label on unavailable dishes.
2. Open an AI app builder and type one vague line: "make a canteen menu page". Submit it. Save a screenshot and note the time taken.
3. Look hard at what came back. Write down three specific things wrong or missing for a real canteen — prices without a currency, no sold-out state, unreadable on a phone, whatever you actually find.
4. Now write a full Director's Brief for the same page, using the template. Spend eight real minutes on it. Write the ACCEPTANCE section first, then work backwards to REQUIREMENTS.
5. Include at least one NON-GOAL and at least one acceptance criterion about something failing — an empty menu, a missing photo, a very long dish name.
6. Start a fresh conversation — not a continuation — and submit the brief. In the ASK, demand two approaches with tradeoffs before any code.
7. Read both approaches. Pick one and write one sentence on why. That sentence is your first decision-log entry.
8. Have it build only your chosen approach. Then test every acceptance criterion yourself, one at a time, and record pass or fail.
9. For anything that failed, write one precise iteration: symptom, expected versus actual, and an explicit scope limit. Send it. Confirm it fixed that and changed nothing else.
10. Compare the two results side by side and write 150 words on what the eight minutes of briefing actually bought you.

## Talk to AI
```prompt
CONTEXT:      I am learning to direct AI rather than write code. I have no
              existing project. I am building a single web page.
              Audience: students at an Indian college canteen, on budget
              Android phones, often on slow 4G.

GOAL:         A one-page canteen menu that makes it obvious what is
              available today and what it costs.

REQUIREMENTS: 1. Six dishes, each with name, price in rupees, and a photo
              2. Dishes that are sold out are clearly marked and visually
                 de-emphasised, not hidden
              3. Readable one-handed on a 5-inch screen
              4. Works when a photo fails to load
              5. Works when every dish is sold out

CONSTRAINTS:  Free tier only. No paid fonts or images. Must be legible in
              bright sunlight, so strong contrast. Total page weight under
              500 KB. No login.

NON-GOALS:    No cart. No payment. No admin page. No animation.

ACCEPTANCE:   - A stranger can name the cheapest available dish in under
                5 seconds
            - Sold-out dishes are distinguishable without relying on colour alone
            - With images blocked, every dish is still identifiable
            - With all six sold out, the page explains itself rather than
                looking broken
            - No horizontal scrolling at 320px wide

ASK:          First propose 2 different approaches to the layout and the
              sold-out treatment, with the tradeoffs of each, and say which
              you would choose and why. Do not write any code until I pick one.
```

## Mission

**Objective.** Produce the same canteen menu page twice — once from a vague prompt, once from a full Director's Brief — and document exactly what the brief bought you.

This is the only module where missions use the canteen. From M01 onward, every mission applies to your own product.

**Steps.**
1. **Frame** — write down, in one sentence, who this page is for and what "done" means.
2. **Vague attempt** — give an AI app builder a single-line prompt. Screenshot the result. Note the elapsed time.
3. **Critique** — list three specific failures of the vague result for a real canteen.
4. **Brief** — write a full Director's Brief using the template. Write ACCEPTANCE first. Include at least one non-goal and one failure-state criterion.
5. **Direct** — in a fresh conversation, submit the brief. Demand two approaches with tradeoffs before any code. Pick one and record why.
6. **Verify** — test every acceptance criterion yourself. Record pass or fail for each. Block images and check again. Set every dish sold out and check again.
7. **Iterate** — for one failure, write a precise correction with a scope limit. Confirm it fixed that and only that.
8. **Document** — one decision-log entry: what you chose, what you rejected, and why.

**Upload.**
- Your Director's Brief, in full → post caption or an attached file
- The vague prompt and its result → screenshot
- The briefed result → deployed URL in `demo_url`, or screenshots
- The AI conversation showing the two proposed approaches and your choice → share link in `ai_chat_url`
- Your acceptance-criteria checklist with pass/fail marked per item → screenshot or text
- Your one decision-log entry
- Reflection, 150 words max → what the briefing time bought you, and one thing the AI got wrong

**Self-check.** Tick every box before you submit.
- [ ] My brief fills in all seven fields, not just GOAL
- [ ] My ACCEPTANCE criteria are testable by someone who is not me
- [ ] At least one criterion covers a failure state, not the happy path
- [ ] I have at least one NON-GOAL written down
- [ ] I asked for two approaches and got them *before* any code
- [ ] I can say in one sentence why I picked the approach I picked
- [ ] I tested every criterion myself rather than trusting the AI's summary
- [ ] I tested with images blocked and with everything sold out
- [ ] My iteration names a symptom and an expected-versus-actual
- [ ] My reflection names something specific the AI got wrong

**Stretch.** Give the identical brief to a second AI tool. Compare the two results and write 100 words on where they differed and what that tells you about the tools rather than the brief.

## Stuck? Ask AI
```prompt
I wrote this Director's Brief: [paste it]. Do not act on it. Instead, review it as if you were going to be held responsible for the result. Which field is weakest? Which of my acceptance criteria are opinions rather than tests? What constraint have I probably left in my head instead of writing down? What will you most likely get wrong because of what is missing?
```

```prompt
Give me five acceptance criteria for a feature, where some are genuinely testable and some are disguised opinions. Ask me to sort them and explain my reasoning. Do not tell me the answers first.
```

```prompt
I keep writing briefs that are too long for small tasks and too short for big ones. Give me three example tasks of different sizes and tell me, for each, which of the seven brief fields I should actually bother filling in.
```

## Quiz

### Q1 · L3
Which step of the Director's Loop do people most often skip, at the greatest cost?
- (a) FRAME, because the goal is usually obvious
- (b) ARCHITECT — asking for options and tradeoffs before any code ✅
- (c) DIRECT, because the AI builds anyway
- (d) ITERATE, because the first result is usually fine
> Skipping ARCHITECT means accepting the first approach without knowing an alternative existed. Comparison is also what exposes plausible-but-wrong reasoning.

### Q2 · L4
Which of these is a genuine acceptance criterion rather than an opinion?
- (a) The page should look clean and modern
- (b) The page must work well on mobile
- (c) At 320px wide there is no horizontal scrolling ✅
- (d) Performance should be good on slow networks
> A criterion is testable by someone who is not you, with the same verdict every time. "Clean", "well" and "good" are judgements; a pixel width is a test.

### Q3 · L4
Why does the brief include a NON-GOALS field?
- (a) To make the brief look more professional
- (b) To stop the agent changing things you did not ask it to touch ✅
- (c) To reduce the number of tokens used
- (d) To list features for a later version
> On any project that already exists, scope limits prevent the most expensive failure: a one-hour task becoming a three-hour recovery across forty files.

### Q4 · L4
The acceptance list for a staff page includes "a logged-in STUDENT who opens this page is refused", though the goal never mentioned students. Why does this matter?
- (a) It makes the brief longer and therefore better
- (b) It catches authentication-without-authorization before it ships ✅
- (c) It tells the AI which framework to use
- (d) It is required by OWASP
> This is the broken Google login from the previous chapter, prevented. Writing what must be *true* rather than what to *add* surfaces the requirement nobody stated.

### Q5 · L5
You are given a vague brief and a precise brief for the same task. What is the strongest reason the precise one produces better results?
- (a) Longer prompts always produce better output
- (b) It supplies the context and constraints the AI otherwise invents ✅
- (c) It makes the AI think for longer before answering
- (d) It avoids hallucination entirely
> Missing context is the most common failure mode. Absent real constraints, the model fills the gap with a plausible generic project — which is not yours.

## Go deeper
- [Anthropic: Prompt engineering overview](https://docs.anthropic.com/en/docs/build-with-claude/prompt-engineering/overview)
- [Architecture Decision Records: a lightweight format](https://github.com/joelparkerhenderson/architecture-decision-record)
