# 03 · Response to the two external syllabus reviews

Both reviewers only saw `02-module-and-chapter-list.md` v1. They could not see
the live `/internship` page, the platform models, or the v2 design docs. That
shows: their *diagnoses* are mostly right and their *prescriptions* are often
wrong, because the binding constraints are invisible from the chapter list.

Verdict: **13 accepted, 6 rejected, 4 partly accepted.** All accepted items are
implemented in `02-module-and-chapter-list.md` v2.

---

## A · Accepted — they caught real problems

### A1. The counts were wrong (ChatGPT)

Verified by script against the file. ChatGPT was right on every number:

| Metric | v1 claimed | v1 actual |
|---|---|---|
| Chapters | 252 | **256** |
| Missions | 54 | **95** |
| Sprint chapters | 194 | **198** |
| Sprint excl. elective | — | **184** |

Per-module headers were correct; only the roll-up was wrong. Fixed, and the
deeper lesson is recorded as a rule: counts must be generated from the chapter
list, never hand-written (`01-platform-models.md` §3.5). v2's counts are
script-verified.

### A2. Git and GitHub were completely absent (both)

Verified: zero matches for `git|github|pull request|branch|commit|merge` across
all 256 chapter titles. This was a real regression I introduced — the v2
syllabus had `1.6 Git and GitHub: The Time Machine` at target level **L5**, the
highest in the course, and I lost it compressing M02 from 27 chapters to 24.

Both reviewers are also right that it belongs in week one, not in a DevOps
module: three people and several agents editing one repo *is* the collaboration
substrate.

→ **M01.6–8**: version control with AI, branches/commits/merge conflicts when
agents touch many files, pull requests and reviewing a diff you did not write.
Placed in the team module, before any building starts.

### A3. Testing and debugging were far too thin (both)

Verified: one testing chapter in Sprint (`The Testing Pyramid and Release
Strategies`) and **zero** debugging chapters. Indefensible for a program whose
own design principle #8 is "Verify, don't trust" and whose central claim is that
students must judge AI output.

→ **M06.10** unit/integration/e2e tests, **M06.11** debugging (stack traces,
reproduction, bisecting), **M11.4** reviewing AI's work, **M13.6** regression
testing and test-data management, **M18.3** live debugging under questioning.

### A4. Sprint was badly overloaded (both)

Correct and the most important finding. 198 chapters and **59 missions** in
30–45 days is roughly two missions per day on top of ~80 h of reading.

Fixed — but not the way either reviewer proposed. See **B1**.

→ Sprint: 198 → **142 chapters**, 59 → **32 missions**.

### A5. Missions should be the product, not homework (implied by both)

Neither said it in these words, but both circled it: the project should run
continuously and the mission load looked impossible. Resolved by making it
explicit in the doc — every mission applies its concept to the team's own
startup, so mission hours *are* MVP hours. That also matches the live page
("each group runs its own company **from the first week**").

### A6. The MVP module should be checkpoints across the sprint (both)

Right, and it contradicted our own sales copy: the page promises a startup from
week one and a portfolio updated daily, while v1 parked all six MVP chapters at
the end.

→ **M11 · Ship Your MVP: Build Checkpoints**, 7 chapters, four of them explicit
week-boundary gates, with a scheduling note that the module is read *across* the
Sprint even though `order_index` places it last.

### A7. Frontier Tech broke the flow before the capstone (Gemini)

Right, and cheap to fix — `order_index` is free.

→ Moved to the end as **M19 · Frontier and Hardware Electives**. Also moved to
rank 3, for a platform reason neither reviewer could know: see **C4**.

### A8. Agent safety deserves more than a passing mention (ChatGPT)

Agreed, and it maps onto OWASP's excessive-agency category.

→ **M00.9** agent permissions, sandboxes and safe execution (early, before any
agent is pointed at a repo); **M07.3** extended to name excessive agency;
**M10.17** the harness's permission model.

### A9. Agent-ready repositories (Gemini)

The best new idea in either review, and it ties A3 to the AI thesis: tests and
linters are what let an agent verify its own diff.

→ **M10.19 · Making a Repo Agent-Ready: Linters, Test Harnesses, Self-Verifying
Diffs.**

### A10. Ingestion and data quality for RAG (both)

Both converged here from different directions — ChatGPT defending data
engineering, Gemini asking for multi-modal/PDF parsing in RAG. They are right
that document ingestion is where real RAG actually fails.

→ **M10.9 · Ingestion and Data Quality: PDFs, Scans, Tables and Multi-Modal
Input.** The warehouse/ETL/BI material they wanted restored went to **M13.12**
(rank 2), where it belongs — see **B4**.

### A11. Three chapter-level reframes (ChatGPT)

All three are correct and now applied:

| Was | Now | Why |
|---|---|---|
| Chain-of-Thought and Extended Thinking | **M10.2** Reasoning Models: Decomposition, Verification, Cost and Latency | Do not teach students to rely on a model's private reasoning — it is not reliably exposed |
| Why RAG: LLMs Don't Know Your Data | **M10.8** Grounding AI in Private, Current and Domain-Specific Data | "Doesn't know your data" is the symptom; grounding is the concept |
| The Agent Loop | **M10.14** The Agent Loop: Stopping Conditions, Failure Recovery and Guardrails | The loop is easy; terminating it safely is the hard part |

### A12. Hands-on model selection (Gemini)

→ **M09.13 · Choosing a Model: Quality, Latency, Context, Privacy and Cost**
(mission), paired with **M09.14 · Token Economics**.

### A13. SSL/TLS inconsistency, and counts should be generated (Gemini)

Both correct. Terminology standardised on "TLS certificates" throughout, with
SSL named once as the legacy synonym. Count generation recorded as a rule
(§3.5).

---

## B · Rejected — right diagnosis, wrong prescription

### B1. "Move M05 Scale, deep M09, and K8s/IaC into Rank 2" (both, emphatically)

**Rejected.** This would break a published promise and expose it automatically.

The live pricing page says:

> "Everyone sees the map. Not everyone ships the company."
> "The 200+ concepts are **not cut short** on the shorter tracks — every student
> walks the whole map."

and Sprint's own card lists "**The full concept map, end to end**".

Worse, the coverage table and the sentence "a Sprint student does not reach X, Y,
Z" are **generated from `min_tier_rank`** (`page.tsx:1056–1148`,
`tier-model.ts:85`). Demoting Scale and AI Foundations to rank 2 would render a
table stating Sprint omits them, directly beneath the headline saying it does
not. There is no copy change that fixes that while keeping the sale honest.

Gemini also argued this on pricing-perception grounds (Rank 1 "gives away" 13 of
19 modules). Same answer: the page deliberately sells it that way — the
differentiator is outcome depth, not chapter count.

**What was done instead:** the real problem was never topic coverage, it was my
chapter granularity. v1 split 61 syllabus modules into 198 Sprint chapters, far
finer than the v2 plan's own ~180-for-the-whole-course target. Consolidating got
Sprint to **142 chapters — inside Gemini's own 135–145 target — without removing
a single concept.** Every deferred topic keeps a named Sprint slot and gains a
rank-2 practice chapter; the full mapping is the "nothing was cut from Sprint"
table in `02`.

And Gemini's value-perception point is answered constructively: Rank 2 gained a
whole new module (**M13 · Production Engineering**, 12 chapters) absorbing the
deferred depth. The ladder went from 194/+31/+27 to **142/+47/+42**.

### B2. "Add Python, JavaScript/TypeScript and SQL coding exercises" (ChatGPT, marked Critical)

**Rejected.** This contradicts the product thesis and the public copy.

- Design principle 7: "Talk, don't type… Students write specs, prompts, and
  reviews. **They never write code.**"
- Live FAQ: "Do I need to know how to code? **No. Nothing is written by hand**;
  AI agents write the code."
- Hero stat: "**0** lines typed by hand."

A syntax-drill track would make the landing page false.

**Partly accepted** though, because there is a real kernel: reading, debugging
and verifying are not the same as writing, and v1 was genuinely weak on them.
Covered by A3 plus the existing reading-oriented chapters (M02.3 reading a page
you did not write, M04.9 reading a query plan, M01.8 reading a diff, M11.4
reviewing AI's work).

### B3. "Standardise the duration to one month" (ChatGPT)

**Rejected in that direction.** ChatGPT spotted a genuine inconsistency but
resolved it the wrong way. The live page says 30–45 days in all four places it
appears (plan card, FAQ fee answer, internship-credit answer, coverage note).
The outlier is the drafted JSON's `duration_label: "1 month"`.

→ Standardise on **"30–45 days"**, and fix the tier label in the import JSON.

### B4. "Restore Data Engineering Basics" as OLTP/OLAP/warehouse theory (ChatGPT)

**Rejected as framed, accepted in substance.** The valuable half is ingestion and
data quality, which now sits in Sprint at M10.9 (A10). Warehouses, ETL/ELT and BI
are genuinely low value per hour for a 30-day MVP, so they went to **M13.12** at
rank 2 — available to everyone who needs them, not occupying Sprint.

### B5. "Name shadcn/ui, Sentry, PostHog, Vercel, Railway in the curriculum" (Gemini)

**Rejected in chapter titles, accepted in lesson bodies.** Production plan §8:
"Concepts are evergreen; tools are swappable. Tool names and free-tier details
live **only** in each lesson's 'Tools today' box," and high-volatility lessons
get a 6-month review cycle. Tool names in titles would rot the module list —
which, per §3.3, is also public marketing copy.

The substance is right and those tools are the intended defaults. They go in
`Tools today`, where they can be swapped without touching a title or a URL slug.

### B6. "Add a weighted scoring rubric (35/25/15/15/10)" (ChatGPT)

**Rejected as platform scoring, accepted as a review framework.**

There is no grading in this product by design: "Missions are upload-based;
submission = completion. No LLM grading." And §3.2: the inline quiz is capped at
3–5 MCQs, never gates completion, and is attemptable without enrolling. A
weighted rubric is not expressible.

ChatGPT's underlying worry — one strong developer carries the team and all three
get the same certificate — is real, but **structurally already handled**:
`Post.user_id` and `ChapterProgress` are both per-user, so each student must
submit their own mission artefacts to complete a mission chapter. The group
shares a product; it cannot share a completion record.

→ The rubric is adopted as the **review framework for daily live sessions and
Demo Day**, and as the shape of each mission's self-check list. Added **M18.3**
(live debugging under questioning) and **M18.5** (operational handover) so the
defence tests individual understanding, which was ChatGPT's actual point.

---

## C · Things neither reviewer could see

Found while acting on the reviews. These matter more than most of the review
items, and none are visible from the chapter list.

### C1. A launch-blocking platform bug (`01-platform-models.md` §3.7)

`helpers/course_access.py:84` has a correct tier-aware `reachable_chapter_count()`
whose docstring names this exact scenario. But `course_complete.py:602` (and
:773, :1073, :1198) recomputes the denominator **untiered** and overwrites
`enrollment.total_chapters`. A Sprint student enrolls at 142/142, then on their
first "Mark complete" silently drops to 142/231 = **61%** and can never become
certificate-eligible — against the FAQ's promise of a certificate on every track.
Must be fixed before the first paid Sprint enrollment.

### C2. "Optional chapter" is not expressible (§3.6)

Every published chapter counts in the progress denominator. There is no
elective flag. So ChatGPT's central structural proposal — "keep the complete
syllabus as a learning library, select a smaller mandatory Sprint pathway" —
cannot be implemented as stated: library chapters would sit in every student's
denominator and hold them below the threshold permanently.

This is why consolidation (B1) was the only real fix available, and why
reducing workload had to mean fewer chapters rather than more chapters marked
optional.

### C3. The assessment plan does not fit the platform (§3.2)

The inline quiz hard-validates 3–5 single-answer MCQs per chapter, and no
module-quiz or phase-test object exists. The v2 plan's lesson checks (5–8 Q),
module quizzes (12–15 Q), phase tests (25–30 Q) and 70% pass marks are not
expressible. Still an open product decision.

### C4. Why the elective module sits at rank 3

Following from C2: an unadvertised 14-chapter elective block at rank 1 would be
*required* for everyone. Rank 3 keeps it out of the Sprint and Build
denominators while still shipping the content, and makes it a top-tier bonus.
Alternative is to leave it unpublished.

### C5. No Mermaid (§3.1)

The renderer is `react-markdown` + `remark-gfm` + `remark-breaks` only. Every v2
design doc assumes Mermaid. Diagrams must be ASCII in fenced code blocks, or
exported images, or the renderer needs extending.

---

## D · Still open for you

1. **Diagrams** — ASCII in code fences, exported images, or add a mermaid
   renderer? (C5)
2. **Assessment** — accept 5-question lesson checks only, or build real stage
   tests with a pass mark? (C3)
3. **Certificates** — fix the tier-aware denominator (C1), or issue per batch
   from admin (`threshold = 999`)? For a cohort program with a demo day, admin
   issuance is defensible on its own merits.
4. **Electives** — M19 at rank 3 as a Company bonus, unpublished, or cut? (C4)
5. **Solo learners** — Gemini asked what happens to someone who enrols without a
   team. Operational, not curricular: batches assign teams. Worth a line in the
   FAQ either way.
