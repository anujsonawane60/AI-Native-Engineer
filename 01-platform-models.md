# 01 · Platform Models — how the internship maps onto CTOschool

Source of truth for the platform side: `ctoschool-website/backend/models.py`,
`routes/admin_courses.py` (`validate_content_data`), `helpers/quiz_validation.py`,
`new-frontend/components/chapter/ChapterBody.tsx`,
`new-frontend/app/internship/*`.

**The internship surface is already built and live.** `/internship` (marketing +
pricing + enrol), `/internship/learn` (learner dashboard) and
`/internship/learn/[moduleSlug]/[chapterSlug]` (chapter reader) all ship today
and degrade to static copy because the course row does not exist yet. The slug
is already fixed in code:

```ts
// new-frontend/app/internship/tier-model.ts
export const INTERNSHIP_COURSE_SLUG = "ai-native-engineer";
```

So this is not a "build a platform" job. It is a "author the content into the
shape the live platform already expects" job.

---

## 1. Entity mapping

The platform has exactly three content levels — `Course → CourseModule →
CourseChapter`. The syllabus has four (course → phase → module → lesson), so one
level must collapse. It collapses at the **phase/module** level: a platform
module is one marketing-legible stage, and a platform chapter is one 20–30 minute
lesson.

| Syllabus concept | Platform object | Notes |
|---|---|---|
| The course | `Course` | `slug = "ai-native-engineer"` (already hardcoded in the frontend) |
| Track: Sprint / Build / Company | `CourseTier` rank 1 / 2 / 3 | `price_inr` in **paise**: 500000 / 800000 / 1000000. `duration_label` is display-only |
| Stage | `CourseModule` + `min_tier_rank` | **Public copy** — see §3.3 |
| Lesson | `CourseChapter`, `chapter_type='lesson'` | one 20–30 min read |
| Lesson body | `content_data.languages.en.overview[]` | array of markdown strings, rendered under "Overview" |
| "Picture It" walkthrough | `content_data.languages.en.instructions[]` | numbered cards (01, 02, …) |
| "Talk to AI" / Director's Brief | `content_data.languages.en.prompts[{que}]` | copy-button cards |
| Mission | `content_data.languages.en.activity_text[]` + `requires_activity=true`, `min_activities_required=N` | accent cards; gates chapter completion |
| "Stuck? Ask AI" | `content_data.languages.en.activity_prompts[{que}]` | copy cards under the mission |
| Lesson outcomes | `content_data.languages.en.key_learnings[]` | renders at the **top** as "What you'll learn"; first 4 shown, rest collapsed |
| "Go deeper" | chapter `resources[]` | renders as "Reference reading" |
| Lesson MCQ check | `quiz_data` (v1) | **3–5 single-answer MCQs, hard limit** — see §3.2 |
| Mission submission | `Post` with `chapter_id`, `post_type='chapter_activity'` | `caption`, `ai_chat_url`, `repo_url`, `demo_url` + `Media` uploads |
| Mission "grading" | none — submission counts | matches the "no LLM grading" decision |
| Daily portfolio | `Post` + profile pinning | already built; no new work |
| Certificate | `Course.provides_certificate`, `certificate_completion_threshold` | promised on every track in the FAQ |
| Stage badge | `BadgeTemplate` / `UserBadge` | already built |
| Hinglish / Marathi layer | `content_data.languages.hi` / `.mr` | schema already multi-language |

### The mission upload kit maps cleanly

`Post` was built for exactly this and needs no change:

| Upload kit item | Field |
|---|---|
| 1. The Director's Brief they wrote | `caption` (or a `Media` PDF) |
| 2. AI chat link / export | `ai_chat_url` |
| 3. Proof: deployed URL | `demo_url` |
| 4. Repo | `repo_url` |
| 5. Verification screenshots | `Media` attachments |
| 6. Reflection (≤150 words) | `caption` |

Submitting a Post increments `ChapterProgress.activities_completed`
(`routes/course_complete.py:257`). With `requires_activity=true`, the chapter
cannot be marked complete until `min_activities_required` is met — so missions
genuinely gate progress and the certificate, with no grader.

---

## 2. Tier architecture — this is the part the drafted JSON gets wrong

The live pricing page makes a specific promise:

> "The 200+ concepts are **not cut short** on the shorter tracks — every student
> walks the whole map. What changes is how far you carry it: 30–45 days gets you
> a working product, six months gets you a company with users and revenue."

And Sprint's own feature list says "**The full concept map, end to end**".

The drafted `Syllabus/v2/ai-native-engineer-course.json` does the opposite — it
tags Sprint as phases 0–3 only, which would hide all of AI, ML, cloud and
DevOps from a Sprint student. Worse, it would do so *visibly*: the pricing page
generates its coverage table and its "a Sprint student does not reach …"
sentence from `min_tier_rank` itself
(`new-frontend/app/internship/page.tsx:1141`, `tier-model.ts:85`). Importing
that tagging would publish a table contradicting the copy directly above it.

**Corrected rule: tiers gate build depth, never concept coverage.**

- **Rank 1 (Sprint)** — every concept module, plus the team/startup module and a
  shipped MVP. This is the whole 200+ concept map.
- **Rank 2 (Build)** — adds the practicum modules: apps shipped for real, deeper
  ML/DL/multi-agent, content engine + social automation.
- **Rank 3 (Company)** — adds users/analytics/growth loops, distribution +
  revenue, and demo day.

This reproduces the live `COVERAGE` table (`page.tsx:277`) exactly. Note that
table has three depth levels (`full` / `overview` / `none`) but `min_tier_rank`
is binary, so every "overview → full" row becomes **two modules**:

| Coverage row | Sprint | Build | Company | Modules |
|---|---|---|---|---|
| The 200+ concept map | full | full | full | M00, M02–M10 (rank 1) |
| Group of three, your own startup | full | full | full | M01 (rank 1) |
| Idea to a working MVP | full | full | full | **M11** (rank 1) |
| Portfolio, updated daily | full | full | full | M01 ch. 10 (rank 1) |
| Certificate on completion | full | full | full | course-level |
| Web + mobile apps shipped | overview | full | full | M08 (rank 1) + **M12 (rank 2)** |
| Deeper ML, DL + agents | overview | full | full | M09/M10 (rank 1) + **M14 (rank 2)** |
| Content engine + social automation | none | full | full | **M15 (rank 2)** |
| Users, behaviour analytics + growth loops | none | none | full | **M16 (rank 3)** |
| Distribution + revenue | none | none | full | **M17 (rank 3)** |

M07 (Observability and Analytics) stays rank 1 even though "behaviour analytics"
is a Company row: M07 teaches the *concepts* (events, funnels, cohorts,
retention) because they are part of the map everyone walks; M16 is the
*practicum* of getting real users and running growth loops on them. That is the
"everyone sees the map, not everyone ships the company" line working as intended.

### Hours

The page advertises Sprint as **120–180 hours** and the drafted JSON gives it 52.
Reconciled: the whole concept map **is** Sprint — 142 chapters ≈ 59 h reading
plus 32 missions ≈ 48 h building ≈ 107 h self-directed, plus 60–90 h of live
sessions (2 h/day × 30–45 days). Missions are largely built *in* the live room,
so the two overlap rather than stack, landing inside 120–180 h. Rank 2 and 3 add
runway and depth on top, not more concept map.

---

## 3. Platform constraints that change how we author

### 3.1 No Mermaid — diagrams must be ASCII or images

`ChapterBody.tsx` renders markdown with `react-markdown` + `remark-gfm` +
`remark-breaks` and nothing else. There is no mermaid, no KaTeX, no syntax
highlighter. Confirmed against `new-frontend/package.json`.

The v2 docs assume Mermaid throughout (`04-visual-system.md`, the lesson
template's "Picture It" and "Where It Lives" sections, and the MCQ guidance
"diagram questions use an image exported from Mermaid").

Options, in order of preference:
1. **ASCII diagrams in fenced code blocks.** Works today, zero platform change,
   diffable in git, and they get a copy button for free.
2. Author in Mermaid, export PNG/SVG, embed via `![]()`. Needs an asset pipeline
   and the images can drift from their source.
3. Add a mermaid renderer to `ChapterBody`. Cleanest long-term; a platform change.

### 3.2 The inline quiz caps at 3–5 MCQs — the assessment plan does not fit

`helpers/quiz_validation.py` hard-validates: `version == 1`, **3–5 questions**,
`type == "mcq"` only (single answer), 2–6 options. There is no module-quiz or
phase-test object anywhere in the schema.

The v2 assessment plan calls for lesson checks (5–8 Q), module quizzes (12–15 Q)
and phase tests (25–30 Q) totalling ~2,000 questions. What actually fits:

- Lesson check → `quiz_data` on the lesson chapter, **max 5 Q** (not 5–8)
- ~250 chapters × 5 = **~1,250 questions maximum**, all inline
- Module quizzes and phase tests have **no home**

Also: the inline quiz **never gates "Mark complete"**, and any logged-in user can
attempt it without enrolling. So it is a learning aid, not an assessment gate —
the pass-mark idea (70% phase tests) is not expressible today.

Three ways out: cap lesson checks at 5 and drop module/phase tests; or add
dedicated `chapter_type='quiz'` chapters at the end of each module (a 25-question
phase test = 5 such chapters, which is clumsy); or raise the cap and add a
gating flag in the platform. **Recommend: cap at 5 for v1, and treat the stage
test as a platform change to schedule separately.** Flagging rather than
deciding — this one is a product call.

### 3.3 Module titles are public marketing copy

`CourseModule.title` renders in three public places: the pricing coverage table,
the "a Sprint student does not reach X, Y, Z" sentence, and the learner
dashboard's locked-module list. Titles must read as things a student would pay
to reach — "Content Engine and Social Automation", not "Phase 15".

This is also why the module list stays at 19 rather than the syllabus's 61: the
coverage table renders one row per module.

### 3.4 Import mechanics

- Format: `export_meta.format` must be `"ai-leela-course-export"`
  (`import_course_cli.py:48`). The drafted JSON has `version`/`source` but **no
  `format` key**, so it would be rejected today.
- The importer **ignores `tiers` and `min_tier_rank`**. After importing, create
  the three tiers in `/admin/courses/<id>/tiers`, then tag each module's
  `min_tier_rank`.
- Import is idempotent on slug: a second run with `slug=ai-native-engineer`
  skips rather than updates.
- `quiz_data` is importable per chapter via `_upsert_chapter`.

### 3.5 Chapter counts must be generated, not hand-written

Hand-authored totals drifted on the very first draft of
`02-module-and-chapter-list.md`: the summary claimed 252 chapters / 54 missions
when the body actually contained 256 / 95. Per-module headers were right; only
the roll-up was wrong, which is exactly the failure mode you cannot catch by
reading.

Any count that appears in a doc, a seed script or marketing copy must be derived
from the chapter list at build time. `InternshipTier.module_count` already does
this correctly server-side — "counted from module tags server-side, so it cannot
drift from what unlocks" (`tier-model.ts:47`). Apply the same rule to chapter
and mission counts.

### 3.6 An "optional chapter" cannot be expressed

There is no per-chapter optional/elective flag, and progress counts every
published chapter:

```python
# routes/course_complete.py:602
total_chapters = sum(1 for module in course.modules
                     for c in module.chapters if c.is_published)
enrollment.progress_percentage = (completed_chapters / total_chapters * 100)
```

So a published chapter is a *required* chapter. "Keep the full syllabus as a
library and require a smaller core pathway" is not representable — the library
chapters would sit in every student's denominator and hold them below the
certificate threshold forever.

Consequences for authoring:
- Reducing workload means genuinely **fewer chapters**, not more chapters marked
  optional. This is why Sprint was consolidated 198 → 142 rather than re-tagged.
- An unadvertised elective block (Frontier Tech) is cleanest at the **highest**
  tier rank, so it stays out of the lower tiers' denominators.

### 3.7 Launch blocker: the progress denominator ignores tiers

`helpers/course_access.py:84` already solves this correctly, and its docstring
names this exact scenario:

> "Progress is completed/total, and the certificate threshold is a percentage of
> it — so a Sprint student measured against all 61 chapters tops out at 38% and
> can never become eligible for the certificate the FAQ promises them."

`reachable_chapter_count(course, tier)` is used when an enrollment is created
(`course_learning.py:393, 621, 641`). But the completion path recomputes the
denominator **untiered** and overwrites the stored value:

```python
# routes/course_complete.py:602 — and again at :773, :1073, :1198
total_chapters = sum(1 for module in course.modules
                     for c in module.chapters if c.is_published)
...
enrollment.total_chapters = total_chapters   # "Keep stored total_chapters in sync"
```

Effect with this syllabus: a Sprint student is enrolled with
`total_chapters = 142` (correct), then the first time they mark any chapter
complete it is overwritten to 231. They cap at 142/231 = **61%** and can never
reach a 100% threshold — breaking the FAQ promise that every student who
completes the program receives a certificate, on the two cheaper tracks.

Note these call sites also fail to filter `module.is_published`, which
`course_access.py` does.

**Fix:** have the completion path call
`reachable_chapter_count(course, enrollment.tier)` instead of recomputing.
Four call sites in `course_complete.py`, plus `course_learning.py:205, 1201,
1296`. This is a platform change, not a content change, and it must land before
the first paid Sprint enrollment.

Until it lands, the fallback is `certificate_completion_threshold = 999`, which
the model documents as admin-only issuance — appropriate anyway for a
cohort-based internship with batches and a demo day, and arguably the right
end state regardless.

### 3.8 The CLI importer silently drops every quiz

`routes/admin_json_upload.py` contains **no reference to `quiz_data` at all**.
`upload_course_chapters()` builds `CourseChapter(...)` with `content_data`,
`resources`, `attachments`, `requires_activity` and the rest — but not
`quiz_data`. So the path used by `import_course_cli.py`:

```
import_course_cli.py → upload_course_modules() → upload_course_chapters()
```

imports every chapter with `quiz_data = NULL`, with no error and no warning.
The quizzes are in the JSON, they validate, and they vanish on import.

The other import path is fine: `routes/admin_courses.py` validates and persists
`quiz_data` on chapter POST, chapter PUT and its own module-import route
(`:1279`, `:1324`, `:1432`). Only the JSON-upload module missed it.

**Fix:** one line in `upload_course_chapters()`, next to `content_data`:

```python
quiz_data=safe_get(chapter_data, 'quiz_data'),
```

Ideally with the same `validate_quiz_data()` guard `admin_courses.py` already
uses, so an invalid quiz is rejected rather than stored.

**Until it lands,** import the course first and then PUT each chapter's quiz
through the admin chapter endpoint, which does persist it. At 10 chapters that
is tolerable; at 231 it is not, so this should land before M01 is authored.

Related, and benign: `upload_course_chapters()` calls
`generate_unique_slug(session, CourseChapter, ...)`, which enforces *global*
chapter-slug uniqueness even though the database only requires uniqueness per
module. A colliding slug would be silently rewritten to `overview-2`, changing
the chapter's URL. Our slugs are unique across the whole course, so nothing is
rewritten — but that is a reason to keep them that way rather than reusing
`overview` per module.

### 3.9 Other fixed points

- `min_tier_rank` is tagged **per module**, deliberately not derived from
  `order_index`, so reordering in the Studio never silently revokes paid access.
- One enrollment per `(user, course)`. An upgrade moves `CourseEnrollment.tier_id`
  and writes a second `CoursePayment`; it does not create a second enrollment.
- `CourseChapter.slug` is unique per module, not globally — so `overview` can
  repeat across modules.
- Backend `/api/programs/<program_slug>/<module_slug>/<chapter_slug>` already
  serves any course by slug triple. No backend route work needed.

---

## 4. Reconciling the v2 syllabus with what is being sold

Five places where `Syllabus/v2` and the live page disagree. Resolutions below are
what the chapter list in `02-module-and-chapter-list.md` implements.

| # | v2 syllabus says | Live page sells | Resolution |
|---|---|---|---|
| 1 | Fully self-paced, text-first, no video | 2 h live session every day, online or offline (Nashik) | Platform content is the **curriculum spine and reference**; live sessions do unblocking, review and pairing. Text-first still correct — it just is not the only channel. |
| 2 | One shared running project, "Tiffin" (campus canteen) | Groups of three, **each group builds its own startup** from week one | Keep the canteen as the **teaching analogy and worked example** inside lessons — it is genuinely good pedagogy and the analogy table is already built. Every **mission** applies the concept to the team's own startup. |
| 3 | Sprint = phases 0–3 (52 h) | Sprint = "the full concept map, end to end", 120–180 h | All concept modules rank 1. See §2. |
| 4 | 12 phases, 61 modules, ~180 lessons | 200+ concepts, 4 domains | 19 platform modules, ~250 chapters. Domains map to module groups. |
| 5 | Frontier Tech is a core-adjacent phase (14 h) | **Not mentioned anywhere** — not in the concept marquee, not in the coverage table, not in the FAQ | Kept as M11, rank 1, explicitly **elective and not required for the certificate**. Open question — see below. |

### Topics the live page sells that v2 does not cover

These are new chapters, not re-tagged ones: SEO, search and ranking, the content
engine, social automation and automated posting, growth loops, distribution,
revenue and pricing, multi-agent systems, prompt caching, Codex, how a group of
three divides work, and the daily-portfolio habit. They live in M01, M15, M16
and M17.

### Open questions for you

1. **Frontier Tech (M11)** — keep as an unadvertised elective, or cut? It is 14 h
   of authoring for something no one is being promised. Default in the list:
   keep, marked elective.
2. **Stage tests** — accept 5-question lesson checks only for v1, or schedule the
   platform change for real stage tests with a pass mark? (§3.2)
3. **Diagrams** — ASCII in code fences for v1, or add a mermaid renderer? (§3.1)
4. **Tiffin** — confirm it stays as the analogy/worked example while missions run
   on the student's own startup. (#2 above)
