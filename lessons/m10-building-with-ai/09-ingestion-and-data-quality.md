---
id: m10-09-ingestion
module: m10-building-with-ai
slug: ingestion-and-data-quality
title: "Ingestion and Data Quality: PDFs, Scans, Tables and Multi-Modal Input"
order: 9
duration_min: 35
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

A team collects two hundred college circulars for their student-services assistant: fee rules, hostel timings, exam policies, mess notices. They push all of them through a text extractor, embed everything, and ask their first question. The answers are nonsense.

Picking it apart takes an afternoon. Around sixty of the PDFs are scans — photographs of paper saved as PDF — so the extractor returned empty strings, and empty strings embed to a point in space that happens to sit near plenty of questions. The fee table came out as a single line of ninety-seven digits with no column structure, so "what is the hostel fee for second years" retrieves it and no human could answer from it either. The Marathi circular came out as a row of boxes. Every chunk from every document ends with "Page 3 of 7 · Confidential · do not circulate", which is now the most common phrase in the entire corpus and pulls unrelated chunks together. And two of the fee circulars are superseded by a third, with nothing in the pipeline that knows or cares.

Not one of these is a retrieval bug. Every one of them happened before the first embedding was computed.

### The idea

**Retrieval quality is capped by ingestion quality. A model cannot find what the extractor never produced, and bad text in your store is indistinguishable from fact.**

This chapter comes before the RAG pipeline deliberately. Teams spend weeks tuning chunk sizes and rerankers on a corpus where a third of the documents are blank, and the tuning cannot possibly help.

### The analogy

Before anything reaches the canteen kitchen, somebody receives the delivery. They weigh the sack, look for rot, reject the crate that arrived wet, and write the delivery date on the side with a marker. It is nobody's favourite job and it decides what the food tastes like.

A kitchen that cooks whatever arrives, without looking, serves bad food that is plated beautifully.

**Where the analogy breaks:** rotten onions smell. A badly scanned page produces text that looks like perfectly ordinary text — slightly odd spellings, a few missing lines, numbers run together — and nothing downstream can tell. Your pipeline has no nose. Every check has to be a number you deliberately compute.

### The document ladder

Four rungs, each needing a different route, and the whole point is to sort before you process.

**Born-digital text.** A PDF made from a word processor. The characters are really in the file. A text extractor gets almost everything, cheaply, and this is the easy case.

**Scanned image PDF.** A photocopier or a flatbed scanner produced a picture of a page. There are no characters in the file at all. A text extractor returns nothing, usually without an error, which is the dangerous part. This needs OCR.

**Phone photo.** A student photographs a notice board at an angle, in bad light, with a thumb in the corner. OCR built for flatbed scans struggles. A vision-capable model usually does better because it can use layout and context to resolve what a character-by-character engine cannot.

**Handwriting.** A handwritten tiffin order, a signed receipt. Expect real error rates and design for review rather than accuracy. Never let handwriting reach a money field unchecked.

Sort your corpus onto these rungs first. It takes twenty minutes and it tells you what the rest of the work is.

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

Six stages. The SCORE stage is the one teams omit, and it is the one that catches the sixty blank scans before they poison everything downstream.

### Tables are the hard part

A table is a two-dimensional fact flattened into one dimension, and flattening destroys the relation that made it useful. "Second year · hostel · 42000" means nothing once it is "Second year hostel 42000 First year hostel 38000 Mess 14000".

Three workable approaches, in order of preference.

**Take the structured data out of documents entirely.** If the fees are also rows in your database, answer fee questions with a tool from chapter four and never retrieve the table at all. This is the best answer and the most often missed.

**Extract the table to markdown or CSV and keep it whole.** A small markdown table inside one chunk preserves the row-column relation and a model reads it well. Do not let a table be split across two chunks; that is worse than not having it.

**Have a vision model describe the table in sentences,** stored alongside the table. "The hostel fee for second-year students is ₹42,000 per year." Verbose, and it retrieves far better than a grid, because the question is a sentence too.

### Metadata is the deliverable

The extracted text is only half the output. The other half decides whether everything after this chapter can work at all.

Capture, per document and per chunk: a stable **id**, the **title**, the **source** path or URL, the **created** date, the **effective** date if the document says when a rule starts, the **language**, the **page number**, an **access level**, and a **superseded_by** pointer when a newer document replaces it.

Each buys something specific. The id and page make a citation checkable in chapter eleven. The dates are how you detect staleness and prefer the newest fee circular. The access level is how retrieval becomes permission-aware, which chapter eight named as the failure that turns a feature into an incident. The language field stops a Marathi question retrieving only English chunks. And `superseded_by` stops two obsolete circulars competing with the current one.

### Cleaning, and what to throw away

**Repeated boilerplate.** Headers, footers, page numbers, confidentiality notices. A line appearing on more than half a document's pages is furniture, not content. Strip it.

**Hyphenation across line breaks.** "admis-\nsion" must become "admission" or neither search nor embedding will find it.

**Near-duplicates.** Three copies of the same circular with different filenames will all retrieve, filling your five slots with one fact. De-duplicate on content, not on filename.

**Near-empty pages.** A page yielding four characters is a failed extraction, not a short page. Send it to review.

### Multi-modal input, in Indian conditions

Your real inputs are not clean. A photo taken on a budget phone at 11 pm under a tube light. A WhatsApp forward with four layers of quoted text. A fee receipt photographed at an angle with glare across the amount. A Marathi voice note that needs transcribing first. Devanagari OCR is noticeably weaker than Latin-script OCR, and mixed-script text breaks naive pipelines entirely.

Design for this rather than apologising for it later. Compress before upload. Keep the original file forever so you can re-extract when tools improve. And route anything low-confidence to a human rather than guessing — a wrong fee amount read through a glare is worse than no answer.

One more thing, from chapter seven: **ingestion is an injection surface.** A document you are about to read can contain instructions aimed at whatever reads it. Extraction should happen in a call with no tools, returning text and metadata only, and anything that looks like an instruction stays data forever after.

### Where it lives

Ingestion is a batch job, not a request-time path. It belongs in M04's queues-and-workers territory: a worker picks up a file, extracts, scores, cleans, writes metadata, and either stores the result or files it for review. It runs on upload and again whenever a document changes.

The original files live in object storage from M04, untouched. The extracted text and metadata go to a table. Chapter ten chunks and embeds from that table, never from the raw PDFs, so that re-chunking later does not mean re-OCRing everything.

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

**A silent empty extraction is the worst outcome in this chapter.** No exception, no warning, a blank string stored and embedded. It will retrieve, because a near-empty vector sits in an unhelpfully central place. Score every page and fail loudly.

**OCR everything is wasteful; OCR nothing is broken.** Triage first. OCR is slow and, on free tiers, quota-limited. Running it on born-digital PDFs wastes your allowance on the easy rung.

**Keep the originals, always.** Extraction tools improve and your chunking will change. A pipeline that discards source files has made every future improvement impossible.

**Confidence scores are a gift — use them.** Most OCR and vision tools report per-page confidence. A number below your threshold should route to review, not to your store. Without a threshold you are trusting the worst page as much as the best.

**Nobody owns data quality, so it decays.** New circulars arrive, nothing re-ingests them, and the index quietly becomes last year's. Put re-ingestion on a schedule and a count of documents older than six months on a dashboard, per M07.

**Do not ingest what belongs in a table.** Fees, prices, timings and stock are structured data. Ingesting a PDF of them, then embedding it, is three lossy steps to reach something you already had exactly.

### Verify it

**Compute characters per page for every document and sort ascending.** Read the worst twenty by hand. This single measurement finds nearly every failed scan in one pass, and it takes minutes.

**Spot-check a tenth of the corpus, with your eyes, side by side.** Open the extracted text next to the original page. Count how many have a missing section, a mangled table or dropped diacritics. Write the percentage down — that number is your corpus quality and you should be able to quote it.

**Ask a question whose answer is only in a table.** If the answer is wrong or absent, your table handling is not working regardless of how good the rest looks.

**Ask a regional-language question whose answer is in a regional-language document.** Then the reverse. Both directions fail differently and you need to know which.

**Search your store for a phrase you know is in document seventeen.** If it is not found, that document is not really in your index, whatever the file count says.

**Check that the newest version wins.** Ask about something that changed. If an obsolete circular answers, your dates or `superseded_by` are not being used.

Red flags: documents stored with zero or near-zero extracted text; no characters-per-page measurement anywhere; a corpus where the most frequent phrase is a page footer; a table split across two chunks; no language field; no effective date; no access level; original files deleted after extraction; and a corpus nobody has read a sample of by hand.

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
