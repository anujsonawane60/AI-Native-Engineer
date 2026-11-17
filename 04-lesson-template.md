# 04 · Lesson Authoring Template

Replaces `Syllabus/v2/lesson-template.md`, which assumed Mermaid diagrams,
5–8 MCQs and Tiffin as the running project. All three changed.

**Locked decisions this template implements:**

| Decision | Value |
|---|---|
| Diagrams | ASCII inside fenced code blocks |
| Lesson check | 3–5 single-answer MCQs, explanation on every one |
| Stage tests | none in v1 |
| Running project | the team's own startup; the canteen is only a worked example |
| Certificates | admin-issued per batch, plus the tier-aware denominator fix |

---

## 1. Authoring format

One markdown file per chapter, at
`lessons/<module-slug>/<nn>-<chapter-slug>.md`. The H2 headings are a **fixed
contract** — `build/build_course_json.py` maps each one to a platform field and
will fail the build on an unknown heading.

````markdown
---
id: m00-01-four-boxes
module: m00-orientation
slug: four-boxes
title: "The Four Boxes: Client, Server, Network, Data"
order: 1
duration_min: 25
is_free: true
mission: false
volatility: low
last_reviewed: 2026-11-16
---

## Key learnings
- Name the four parts every app is built from
- Trace a single tap from a phone to a database and back

## Lesson

Body markdown. Headings inside the body use `###`.

## Walkthrough
1. First step of the "follow one request" trace
2. Second step

## Talk to AI
```prompt
A standalone, copy-pasteable brief.
```

## Mission

Only when `mission: true` in the frontmatter.

## Stuck? Ask AI
```prompt
Explain X to me as if I have never seen a computer.
```

## Quiz

### Q1 · L2
What does TTL control in a cache?
- (a) How large the cache can grow
- (b) How long a cached copy stays valid ✅
- (c) How many users can read the cache
> TTL is a clock, not a size limit. When it expires the entry is refetched.

## Go deeper
- [MDN: How the web works](https://developer.mozilla.org/)
````

### Heading → platform field

| Heading | Field | Renders as |
|---|---|---|
| `## Key learnings` | `content_data.languages.en.key_learnings[]` | "What you'll learn", **top of page**, first 4 visible |
| `## Lesson` | `…overview[]` (one entry) | "Overview" section, markdown |
| `## Walkthrough` | `…instructions[]` | numbered cards `01`, `02`, … |
| `## Talk to AI` | `…prompts[{que}]` | copy-button cards |
| `## Mission` | `…activity_text[]` | accent cards under "Activity" |
| `## Stuck? Ask AI` | `…activity_prompts[{que}]` | copy cards beneath the mission |
| `## Quiz` | `chapter.quiz_data` | inline "Check your understanding" |
| `## Go deeper` | `chapter.resources[]` | "Reference reading" link list |

Any section left empty is silently absent from the page. `## Key learnings`,
`## Lesson` and `## Quiz` are required on every chapter.

---

## 2. Renderer gotchas — read these once, they will bite otherwise

All verified against `new-frontend/components/chapter/ChapterBody.tsx`.

**1. Never hard-wrap prose.** `remark-breaks` turns every single newline into a
`<br>`. A sentence wrapped at 80 columns renders as a ragged column of broken
lines. Write each paragraph as one long line and let the editor soft-wrap. (The
generator does not re-join lines — this is on the author.)

**2. Never paste a bare GitHub/GitLab/Replit/StackBlitz/CodeSandbox URL into
`## Lesson` or `## Walkthrough`.** `detectRepoUrl` scans both, grabs the *first*
match, and hoists it into a prominent "Code repository · Open repo" card at the
top of the chapter. A casual reference link becomes a call to action. Put such
links in `## Go deeper` instead, which is never scanned.

**3. Do not number the walkthrough steps in your text.** The renderer stamps
`01`, `02`, `03` onto each card. Write the step content only; the markdown `1.`
is just the parser's delimiter.

**4. Prompts must stand alone.** Every `prompt` block gets a Copy button, and
what copies is exactly the block's text. No "as shown above", no surrounding
prose, no placeholders the student will not notice.

**5. Put the four best `key_learnings` first.** Anything past the fourth is
behind a "Show all" toggle.

**6. No Mermaid, no KaTeX, no syntax highlighting.** The pipeline is
`react-markdown` + `remark-gfm` + `remark-breaks` and nothing else. GFM tables,
task lists and strikethrough do work.

**7. Do not write your own `## Overview` heading inside `## Lesson`.** The
renderer already prints one.

---

## 3. ASCII diagrams

Fenced code blocks, no language tag. They inherit the copy button for free.

Conventions, so 231 chapters look like one course:

- `│ ─ ┌ ┐ └ ┘ ├ ┤ ┬ ┴ ┼` for boxes, `──▶` and `◀──` for flow
- Label every arrow with the thing that travels, not the verb: `order JSON`, not
  `sends data`
- One diagram per concept. If it needs a legend, it is two diagrams.
- Max 72 characters wide — it must not force horizontal scroll on a phone
- Always follow a diagram with one plain sentence describing it, for screen
  readers and for anyone who skipped it

```
  Phone                  Server                   Database
┌─────────┐          ┌─────────────┐          ┌──────────────┐
│  tap    │──order──▶│  checks it  │──query──▶│  saves the   │
│ "Order" │          │  is valid   │          │  order row   │
└─────────┘◀──────── └─────────────┘ ◀────────└──────────────┘
             "order        order id
              placed"
```

A tap travels from the phone to the server, which validates it and writes a row
to the database; the order id comes back the same way.

---

## 4. Lesson body structure

The ten-part concept card from `01-course-philosophy-and-design.md` still
applies, but it now lives inside `## Lesson` as `###` subheadings rather than as
platform sections:

1. **The pain** — 3–5 sentences. Something concrete going wrong without this concept.
2. **The idea** — one bold sentence, then 2–3 plain ones.
3. **The analogy** — the canteen where it fits, *and where the analogy breaks*.
4. **Picture it** — the ASCII diagram plus its sentence. (The step-by-step trace goes in `## Walkthrough`.)
5. **Where it lives** — place the concept on the architecture map.
6. **Words to own** — a GFM table: Term | Plain meaning | Example. 5–10 rows.
7. **Tradeoffs and gotchas** — when not to use it, how it fails, who got burned.
8. **Verify it** — how to check the AI got this right, and the red flags.

"Talk to AI" and the mission are platform sections, not body subheadings.

### Writing rules

- **Length:** 700–1,100 words in `## Lesson`. This is a hard ceiling, not a target to fill. The platform carries the scaffolding — the mental model, the diagram, the vocabulary, the verification discipline. Depth is carried by the student's own AI tutor via the prompts, which is why every chapter ships three of them.
- **Cut noise, keep signal.** Noise is: the same point restated, two paragraphs where one works, a second example after a good first one, throat-clearing transitions, hedging, and `###` sections that duplicate the eight-part skeleton. Signal is: the pain in 2–3 sentences, the one-sentence idea, the analogy and where it breaks, every ASCII diagram, every table, the tradeoffs, and the Verify-it checks with their red flags.
- **Reading level:** plain English for a second-year student reading in their second language. Short sentences. One idea per paragraph.
- **Define before use.** No term appears before its definition.
- **Concrete first.** A story or number before the abstraction.
- **Name real systems.** "UPI does this", "Swiggy does this" — not "some companies".
- **Volatile tool names belong nowhere except `## Go deeper` and the lesson's `Tools today` note.** Concepts are evergreen; free tiers are not. High-volatility chapters get `volatility: high` and a 6-month review.

---

## 5. Quiz rules

`quiz_data` v1 is hard-validated by `helpers/quiz_validation.py`. Violations are
rejected at import, so the build script checks them first.

- **3–5 questions.** Not 2, not 6.
- **Single answer only.** `type: "mcq"` is the only supported value. True/false, "read the output" and "spot the flaw" need no new type — they are MCQs with better stems.
- **2–6 options**, unique ids `(a)`–`(f)`, exactly one marked `✅`.
- **Explanation on every question.** For a self-paced reader the explanation *is* the teaching, and it shows after answering.
- **At least one question per chapter at L3 or above** — placement, sequence, scenario, or spot-the-flaw. Not all definitions.
- **Distractors must be plausible misconceptions**, never jokes. "Fine-tune it" for a fresh-data problem is a good distractor.
- Tag each question `· L1`–`· L5` after the `Q<n>`.

The quiz never gates "Mark complete", and any logged-in user can attempt it
without enrolling. It is a learning aid, not an assessment gate.

---

## 6. Mission format

On chapters with `mission: true`. Every mission applies the chapter's concept to
**the team's own startup** — except in M00, which runs before teams and ideas
exist and therefore uses the canteen.

```markdown
## Mission

**Objective.** What must exist when you are done, in one sentence.

**Steps.**
1. Frame — ...
2. Brief — ...
3. Direct — ...
4. Verify — ...

**Upload.**
- Your Director's Brief → post caption or an attached PDF
- The AI conversation → share link in `ai_chat_url`
- Proof it works → deployed URL in `demo_url`, or screenshots
- Your verification → screenshots of the checks you ran
- Reflection, 150 words max → one tradeoff you chose, one thing the AI got wrong

**Self-check.** Tick every box before you submit.
- [ ] My brief states the problem with a number in it
- [ ] I asked for options before I asked for code
- [ ] I ran the check myself rather than trusting the summary
- [ ] My reflection names a tradeoff in my own words
```

There is no grader. The self-check list is what holds the bar, so it must be
specific enough to fail yourself against. Submitting a `Post` against the
chapter completes the mission; `Post.user_id` is per-student, so in a group of
three each member submits their own evidence.

---

## 7. Build and import

```bash
python build/build_course_json.py            # validate + emit
python build/build_course_json.py --check    # validate only, non-zero on error
```

The script walks `lessons/`, validates frontmatter and quizzes, and writes
`build/ai-native-engineer-course.json` in `ai-leela-course-export` format. It
also prints the chapter/mission counts, which are the only counts anyone should
quote — per `01-platform-models.md` §3.5, hand-written totals drift.

Import, then set up tiers:

```bash
python backend/import_course_cli.py build/ai-native-engineer-course.json
```

The importer ignores `tiers` and `min_tier_rank`. After importing, create Sprint
(rank 1), Build (rank 2) and Company (rank 3) at
`/admin/courses/<id>/tiers`, then tag each module's `min_tier_rank` from the
`min_tier_rank` field the export carries for documentation.
