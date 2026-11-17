---
id: m10-09-ingestion
module: m10-building-with-ai
slug: ingestion-and-data-quality
title: "Ingestion and Data Quality: PDFs, Scans, Tables and Multi-Modal Input"
order: 9
duration_min: 25
is_free: false
mission: true
volatility: low
last_reviewed: 2026-11-16
---

## Key learnings
- Sort documents onto the ladder from born-digital text to handwriting
- Measure extraction quality per page instead of assuming it worked
- Capture the metadata that citation, freshness and permissions depend on
- Handle tables and regional-language scans without silently losing them
- Put a human review gate on ingestion, because bad text looks like good text

## Lesson

### The pain

A team pushes two hundred college circulars through a text extractor, embeds everything, and asks a question. The answers are nonsense.

Sixty of the PDFs are scans, so the extractor returned empty strings — which embed to a point that sits near plenty of questions. The fee table came out as ninety-seven digits with no columns, the Marathi circular came out as boxes, and every chunk ends with "Page 3 of 7 · Confidential", now the corpus's most common phrase. None of this is a retrieval bug; it all happened before the first embedding was computed.

### The idea

**Retrieval quality is capped by ingestion quality. A model cannot find what the extractor never produced, and bad text in your store is indistinguishable from a fact.**

This chapter comes before the RAG pipeline deliberately. Teams spend weeks tuning chunk sizes on a corpus where a third of the documents are blank, and no tuning can help.

### The analogy

Before anything reaches the kitchen, somebody receives the delivery: weighs the sack, looks for rot, rejects the wet crate, writes the date on the side. It is nobody's favourite job and it decides what the food tastes like. A kitchen that cooks whatever arrives serves bad food, plated beautifully.

**Where the analogy breaks:** rotten onions smell. A badly scanned page produces text that looks perfectly ordinary — odd spellings, missing lines, numbers run together — and nothing downstream can tell. Your pipeline has no nose, so every check has to be a number you deliberately compute.

### Picture it

```
  raw files
     │
     ▼
  ┌──────────┐  classify by rung: text / scan / photo / hand
  │  TRIAGE  │
  └────┬─────┘
       ▼
  ┌──────────┐  text extract · OCR · vision model
  │ EXTRACT  │
  └────┬─────┘
       ▼
  ┌──────────┐  chars per page? language? table found?
  │  SCORE   │──── below threshold ──▶ needs-review queue
  └────┬─────┘
       ▼
  ┌──────────┐  strip headers, footers, page numbers,
  │  CLEAN   │  rejoin hyphenated words, drop duplicates
  └────┬─────┘
       ▼
  ┌──────────┐  id, title, source, date, effective, lang,
  │ METADATA │  page, access level, superseded_by
  └────┬─────┘
       ▼
   ready for chunking (chapter 10)
```

Six stages. SCORE is the one teams omit, and the one that catches sixty blank scans before they poison everything downstream.


Sort your corpus onto four rungs first; twenty minutes, and it tells you what the work is.

- **Born-digital text** — the characters are in the file; an extractor gets almost everything.
- **Scanned image PDF** — a picture of a page; the extractor returns nothing, usually silently. Needs OCR.
- **Phone photo** — a vision model usually beats flatbed OCR, because it can use layout and context.
- **Handwriting** — real error rates; design for review, and never a money field.

### Where it lives

Ingestion is a batch job, not a request-time path: M04's queues-and-workers territory, running on upload and whenever a document changes. Originals stay in object storage untouched, so better tools later do not mean re-OCRing everything, and chapter ten chunks from the extracted table rather than raw PDFs.

Metadata is half the deliverable: a stable id, title, source, created and effective dates, language, page number, access level, and a `superseded_by` pointer. Id and page make citations checkable, dates detect staleness, access level makes retrieval permission-aware, language stops a Marathi question retrieving only English.

### Words to own

| Term | Plain meaning | Example |
|---|---|---|
| Ingestion | Turning raw files into clean text plus metadata | The six-stage pipeline |
| Born-digital | Real characters already in the file | A PDF exported from Word |
| OCR | Reading characters out of an image | A scanned circular |
| Vision extraction | A model reading a page as an image | A phone photo at an angle |
| Extraction rate | Characters recovered per page | 4 chars means it failed |
| Boilerplate | Repeated furniture on every page | "Page 3 of 7" |
| De-duplication | Removing near-identical documents | Three copies, one kept |
| Effective date | When a rule starts applying | Fees from 1 July |
| Superseded by | Pointer to the document that replaced this | Old fee circular |
| Needs-review queue | Low-confidence items held for a human | Glare over an amount |

### Tradeoffs and gotchas

**A silent empty extraction is the worst outcome here.** No exception, no warning, a blank string stored and embedded — and it *will* retrieve, because a near-empty vector sits in a central place. Score every page; fail loudly.

**Tables are the hard part**, because a table is a two-dimensional fact flattened into one. Best: take the structured data out of documents and answer fee questions with a tool. Otherwise extract the table to markdown and keep it whole in one chunk, or have a vision model describe it in sentences — which retrieves better, because the question is a sentence too.

**Triage before you OCR, and keep every original.** Use the per-page confidence your tool reports: below threshold goes to human review, not your store. Devanagari OCR is weaker than Latin-script, so route low confidence to a person.

**Ingestion is an injection surface.** A document can carry instructions aimed at whatever reads it, so extract in a tool-less call and treat the result as data forever after.

**Nobody owns data quality, so it decays.** Put re-ingestion on a schedule and a count of documents older than six months on a dashboard. Strip the furniture: repeated footers, hyphens across line breaks, near-duplicates.

### Verify it

- **Compute characters per page for every document and sort ascending**, then read the worst twenty by hand. This finds nearly every failed scan.
- **Spot-check a tenth of the corpus with your eyes**, extracted text beside the original page. The percentage with a missing section or mangled table *is* your corpus quality.
- **Ask a question whose answer is only in a table.** Wrong or absent means table handling is broken.
- **Ask a regional-language question against a regional-language document, then the reverse.** They fail differently.
- **Search your store for a phrase you know is in document seventeen.** Not found means it is not indexed, whatever the file count says.
- **Check the newest version wins.** If an obsolete circular answers, your dates or `superseded_by` are unused.

Red flags: documents stored with near-zero extracted text; no characters-per-page measurement; a corpus whose most frequent phrase is a page footer; a table split across two chunks; no language field; no effective date; no access level; originals deleted after extraction; a corpus nobody has sampled by hand.

## Walkthrough
1. Gather the real documents your product needs. Real ones, including the ugly scan and the regional-language one — a clean sample proves nothing.
2. Sort every file onto the ladder: born-digital text, scanned image, phone photo, handwriting. Count each rung and write the counts down.
3. Run a plain text extractor over everything, then compute characters per page for each document and sort ascending.
4. Read the worst twenty results by hand. Confirm which are failed scans rather than genuinely short documents, and move those to the OCR or vision route.
5. Re-extract that group with OCR or a vision-capable model. Recompute characters per page and compare. Record how many documents the second route rescued.
6. Pick the document with the most important table. Try all three table approaches — flat text, markdown table in one chunk, and a sentence description — and ask the same question of each.
7. Identify your boilerplate: find every line that appears on more than half the pages of a document and strip it. Re-check your most frequent phrases afterwards.
8. Build the metadata record for every document: id, title, source, created date, effective date, language, page, access level, superseded_by. Leave nothing blank by default; use an explicit unknown.
9. Set a confidence threshold and route everything below it to a needs-review list. Review ten items yourself and note what fraction needed correction.
10. Store extracted text and metadata in a table, keep every original file untouched in object storage, and confirm you could re-extract everything tomorrow without re-uploading anything.

## Talk to AI
```prompt
Design the ingestion pipeline for my product's real documents, and tell me how I will know it worked.

MY PRODUCT: [one sentence]
WHAT USERS WILL ASK ABOUT THESE DOCUMENTS: [5 real questions, including one in a regional language]

MY DOCUMENTS - be honest about the mess:
- How many files, total: [number]
- Born-digital PDFs or text files: [roughly how many]
- Scanned image PDFs: [roughly how many]
- Phone photos of paper: [roughly how many]
- Handwritten: [roughly how many]
- Languages present: [list them, including mixed-script]
- Documents containing tables: [how many, and what the tables hold]
- Documents that supersede each other: [describe]
- Anything with a confidentiality or access restriction: [describe]

MY CONSTRAINTS: free tiers only, slow uploads, budget Android phones.

Deliver all of this:

1. A triage rule that sorts my files onto the four rungs automatically, and what signal it uses for each.
2. The extraction route for each rung, naming the kind of tool rather than assuming I have a paid one, and what to do when the free-tier quota runs out mid-run.
3. The quality score I should compute per page, the threshold below which a page goes to human review, and why that threshold.
4. My boilerplate-stripping rule, plus how to detect boilerplate automatically rather than me listing it.
5. The exact metadata schema per document AND per chunk, with the reason each field exists and what breaks downstream if it is missing.
6. How to handle my tables specifically, given what they contain - and tell me plainly if any of that data should be in a database table and never ingested at all.
7. How to handle the regional-language and mixed-script documents, including what will go wrong and what I should expect to lose.
8. The five checks I should run after ingestion to prove it worked, each with a number I can write down.
9. What should happen when a document is updated or replaced, so my index does not quietly become last year's.
10. Where in this pipeline a hostile instruction hidden in a document could reach my system, and what makes that harmless.
```

## Mission

**Objective.** A working ingestion pipeline over your own startup's real documents, with a measured quality number per document and a human-reviewed sample, so that nothing enters your store unscored.

**Steps.**
1. **Frame** — write the five questions your product must answer from documents, in your users' real words, including one in a regional language.
2. **Collect** — gather the actual files. Include at least one scan, one photo, one table and one non-English document. If your startup has none, get the real ones from the organisation you are building for.
3. **Triage** — sort every file onto the four rungs and record the counts.
4. **Brief** — write a Director's Brief for the pipeline, with the metadata schema and the quality threshold as explicit acceptance criteria and non-goals that forbid deleting originals.
5. **Direct** — have AI build the six stages: triage, extract, score, clean, metadata, store. One stage at a time, each verified before the next.
6. **Verify** — compute characters per page for every file, sort ascending, and read the worst twenty by hand. Then spot-check a tenth of the corpus against the original pages with your own eyes.
7. **Fix the worst rung** — re-extract the failures with OCR or a vision model and record how many were rescued.
8. **Review gate** — route everything below your confidence threshold to a needs-review list, and review ten items yourself.
9. **Document** — one decision-log entry: your threshold, your table approach, and what you chose to lose.

**Upload.**
- Your Director's Brief for the pipeline → post caption or attached file
- Your triage table: file counts per rung → screenshot
- Your characters-per-page table, sorted ascending, with the failures visible → screenshot
- A side-by-side of one original page and its extracted text, for your worst and your best document
- Your metadata schema, with one real filled-in example record
- Your corpus quality number: what fraction of your hand-checked sample was faithful
- Your needs-review list with at least one item you corrected by hand
- The AI conversation → share link in `ai_chat_url`
- Reflection, 150 words max → the document type that broke your pipeline, and one thing the AI got wrong

**Self-check.** Tick every box before you submit.
- [ ] My corpus includes at least one scan, one photo, one table and one non-English document
- [ ] Every document has a characters-per-page number recorded
- [ ] I read the worst twenty extractions with my own eyes
- [ ] I hand-checked a tenth of the corpus against the original pages
- [ ] I can quote my corpus quality as a percentage
- [ ] Every document has id, title, source, date, language, page and access level
- [ ] No metadata field is blank by default; unknown is written explicitly
- [ ] My table data is either kept whole in one chunk or answered from a database instead
- [ ] Items below my confidence threshold go to review, not to the store
- [ ] Every original file still exists, untouched, and I could re-extract tomorrow
- [ ] My extraction step has no tools and cannot act on instructions found in a document

**Stretch.** Take your three worst documents and ask a question whose answer is only in them. Record the answer before and after your fixes, and write 100 words on what that difference would have cost you if you had tuned retrieval instead of ingestion.

## Stuck? Ask AI
```prompt
My PDF text extraction returns an empty string for some files and no error at all. Explain why that happens, how to detect it reliably rather than noticing it later, and what the right extraction route is for those files. Then tell me what an empty string does to a vector search if I store it anyway.
```

```prompt
I have a PDF containing a fee table with five columns. Show me three different ways to get that table into a retrieval system - flattened text, a markdown table kept whole, and a sentence description - and tell me what each one gets right and wrong for the question "what is the hostel fee for second years". Then tell me when I should not retrieve the table at all.
```

```prompt
Design the metadata schema for documents in a retrieval system. For each field, tell me what downstream feature breaks if that field is missing - citation, freshness, permissions, language matching, or version handling. Then show me one filled-in example for a college fee circular that was replaced by a newer one.
```

## Quiz

### Q1 · L3
A text extractor returns an empty string for a scanned PDF, with no error. Why is this the most dangerous failure in ingestion?
- (a) It wastes the extraction quota
- (b) The blank is stored and embedded, and it still retrieves ✅
- (c) It corrupts neighbouring documents
- (d) It causes the chunker to crash later
> Nothing reports the failure, and a near-empty vector sits in a central position that matches many questions. Scoring characters per page is what catches it.

### Q2 · L3
Which measurement finds most failed extractions fastest?
- (a) Total file size per document
- (b) Characters recovered per page, sorted ascending ✅
- (c) Number of pages per document
- (d) Average chunk length after chunking
> One cheap number per page separates failed scans from genuinely short documents, and reading the worst twenty by hand takes minutes.

### Q3 · L4
Your fee table flattens into one long line of numbers. Which option is best when the same figures are already rows in your database?
- (a) Keep the flattened text and add more chunks around it
- (b) Describe the table in sentences with a vision model
- (c) Answer fee questions with a database tool and do not retrieve the table ✅
- (d) Increase the chunk size so the whole table fits
> Structured data belongs behind a tool. Ingesting, chunking and embedding a PDF of figures you already hold exactly is three lossy steps to a worse version of what you had.

### Q4 · L4
Why does `superseded_by` belong in your document metadata?
- (a) It reduces storage by removing old files
- (b) It stops obsolete circulars competing with the current one in retrieval ✅
- (c) It is required by most vector stores
- (d) It improves OCR accuracy on later versions
> Three versions of a fee rule will all match a fee question and fill your retrieval slots. The pipeline needs a way to know which one is current.

### Q5 · L5
Where in the ingestion pipeline can a hostile instruction hidden in a document reach your system, and what makes it harmless?
- (a) At embedding; using a different embedding model fixes it
- (b) At extraction; doing extraction in a call with no tools, returning text and metadata only ✅
- (c) At chunking; smaller chunks fix it
- (d) It cannot; ingestion is read-only
> Whatever reads the document can be instructed by it. If the reading call has no tools and can only return text and metadata, the instruction has nothing to act with.

## Go deeper
- [Anthropic: Vision — working with images and documents](https://docs.anthropic.com/en/docs/build-with-claude/vision)
- [Unstructured: document preprocessing concepts](https://docs.unstructured.io/open-source/core-functionality/partitioning)
