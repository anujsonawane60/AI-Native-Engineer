---
id: m10-08-grounding-ai
module: m10-building-with-ai
slug: grounding-ai
title: "Grounding AI in Private, Current and Domain-Specific Data"
order: 8
duration_min: 15
is_free: false
mission: false
volatility: low
last_reviewed: 2026-11-16
---

## Key learnings
- Name the three gaps between what a model knows and what your product needs
- Choose between prompt, tool, retrieval and fine-tuning for each gap
- Explain why fine-tuning teaches shape and does not add facts
- Require a citation and allow a refusal, or it is not grounding
- Scope retrieval to who is asking, not just to what was asked

## Lesson

### The pain

A mentor asks a team's canteen assistant about the refund policy. It answers beautifully: full refund within fifteen minutes, fifty percent after that, UPI in three to five working days. There is no refund policy. Nobody wrote one — the model produced the most plausible refund policy for an Indian food business, because that was the shape of the thing requested.

Two more questions land. "Aaj ka rate kya hai" returns last semester's prices from stale demo data. "SPDT kitne ka hai" gets a guess, because at this college SPDT is the Monday special, a fact that exists nowhere outside the campus. One failure, three causes, three fixes.

### The idea

**A model knows what was in its training data: public, general and frozen at a cutoff. Your product needs answers that are private, current and domain-specific. Grounding means supplying those facts at request time and requiring the answer to come from them.**

The second half is the part teams skip: putting facts nearby is not grounding. Grounding is when the model must point at the fact it used — "refunds within fifteen minutes [circular-2026-04]" — and must say "I do not know" when the fact is absent. That refusal has to be a stated rule, or the model will invent.

### The analogy

You hire a skilled cook from another city. They know cooking better than you do — that is the training data. They do not know your prices, do not know the batter ran out, and have never heard of SPDT. Three objects fix that: the **menu board**, small and stable and always visible; the **storekeeper**, asked about stock because it changes hourly; and the **shelf of circulars**, consulted on demand.

**Where the analogy breaks:** after three weeks the cook learns the menu and stops glancing at the board. A model never does — every conversation starts from zero, so all three stay permanent fixtures.

### Picture it

```
  question
     │
     ▼
  Is the fact small and stable?        ──yes──▶ PUT IT IN
  (a 40-row menu, your 6 policies)              THE PROMPT
     │ no
     ▼
  Is it structured and live?           ──yes──▶ GIVE IT
  (stock, order status, balance)                A TOOL
     │ no
     ▼
  Is it large unstructured text?       ──yes──▶ RETRIEVE IT
  (circulars, FAQs, 200 PDFs)                   (RAG)
     │ no
     ▼
  Is it about voice, format or style?  ──yes──▶ FINE-TUNE
  (not facts - never facts)                     (rarely)
```

Four routes, chosen by the nature of the fact rather than by what sounds impressive. Most questions stop at the first or second box.

- **In the prompt** when the data is small and stable — a forty-row menu is six hundred tokens, exact and cacheable.
- **A tool** when it is structured and changes: chapter four's `get_stock` is grounding of the freshest kind, read at the moment of asking.
- **Retrieval** when it is large, unstructured, and you cannot know which part you need.
- **Fine-tuning** almost never, and never for facts.

### Where it lives

Grounding fills the data block of chapter five's context-assembly function. The facts come from M04 — your tables, your storage, your search index — and keeping them fresh is M05's cache-invalidation problem. This chapter is the module's hinge: chapter nine cleans messy documents, chapter ten builds the pipeline, chapter eleven makes citations trustworthy, chapter twelve measures it.

### Words to own

| Term | Plain meaning | Example |
|---|---|---|
| Grounding | Answering from supplied facts, with a source | Menu rows in the prompt |
| Parametric knowledge | What the model learnt in training | General cooking technique |
| Knowledge cutoff | The date training data stops | Nothing about this term |
| Retrieval | Fetching relevant text at request time | Pulling 5 circular chunks |
| Fine-tuning | Adjusting weights on your examples | Teaching house tone |
| Citation | A pointer to the source of a claim | `[circular-2026-04]` |
| Abstention | Refusing when the facts are absent | "No policy on record" |
| Provenance | Where a fact came from and when | Doc id plus updated date |
| Freshness | How recent the grounded data is | Stock as of 14:02 |
| Permission-aware retrieval | Only searching what the asker may see | Students cannot see staff docs |

### Tradeoffs and gotchas

**Fine-tuning changes style, not facts.** It teaches shape — a house tone, a rigid format. It does not install knowledge you can rely on: facts come out blurred, you cannot cite a weight or delete one student's data from a trained model, and a price change makes it stale. The test: if the answer should change when the data changes, it is retrieval or a tool.

**Retrieval is a system; a prompt is a string.** RAG means chunking, embeddings, a store, a ranker, an eval set and a refresh job — absurd for forty menu rows.

**Grounding without a refusal rule produces better-dressed hallucinations.** Give a model five chunks that lack the answer and no permission to abstain, and it blends them confidently.

**Retrieval must know who is asking.** If your index holds staff-only circulars beside the student FAQ, a student's question will retrieve them. Filter by permissions *before* search.

**Stale is worse than absent**, because last semester's prices look as confident as correct ones. Carry an "as of" timestamp on every grounded fact, and remember grounding sends that data to a provider.

### Verify it

- **Ask a question whose answer is not in your data and insist on a refusal.** Any answer at all is a bug — the chapter's most diagnostic test.
- **Make it cite, then check the citation.** A real citation to a document that does not contain the claim is the module's most dangerous output, because it looks verified.
- **Change a fact and re-ask.** Edit one price in your source; an unchanged answer means the model is reading training data or a cache.
- **Ask the same question as two different users**, a student and a staff member. If the student's answer quotes a staff-only document, retrieval is not permission-aware.
- **Ask "as of when?"** Every grounded answer should know the age of the fact it used.

Red flags: a confident policy nobody wrote; a factual answer with no citation; a citation pointing at a document that does not contain the claim; "we will fine-tune on our data" offered as the fix for freshness; a vector store built for data that would fit in the prompt; retrieval that ignores who is asking; no timestamp on grounded facts.

## Walkthrough
1. Write down ten real questions your users will ask your product. Use their actual words, including at least two in Hindi or a regional language.
2. For each, write where the true answer lives today: a database row, a document, somebody's head, or nowhere at all. Be honest about "nowhere" — those need writing before they need engineering.
3. Tag each question with the gap it exposes: private, current, domain-specific, or none of the three.
4. Route each one using the decision tree: small and stable goes in the prompt, structured and live gets a tool, large and unstructured gets retrieval. Write the route next to the question.
5. Count how many ended up in each bucket. If almost everything landed in retrieval, re-examine — that usually means structured data was mislabelled as documents.
6. Take your prompt-bucket facts and put them in the system prompt now, in the stable block from chapter six. Test those questions. Several will already be solved.
7. Take your tool-bucket facts and write the tool definitions, using chapter four's rules. Test those questions. Note the freshness: "as of" what time.
8. Add a refusal rule and one few-shot example showing an abstention, then ask three questions whose answers are genuinely not available anywhere. Record whether it refused or invented.
9. Add a citation requirement to the output format, then check one citation by opening the source and finding the sentence. Record whether it was really there.
10. Write down which questions still have no good route. That list is the scope of your retrieval work in the next four chapters, and it should be shorter than you expected.

## Talk to AI
```prompt
Help me decide how to ground each answer my product needs, and talk me out of over-engineering.

MY PRODUCT: [one sentence]
MY USERS AND THEIR PERMISSIONS: [who asks, and what each kind of user may and may not see]

REAL QUESTIONS USERS ASK - use their wording, include regional language ones:
1. [question]
2. [question]
3. [...]

WHERE MY DATA LIVES TODAY:
- Database tables: [names, row counts, how often each changes]
- Documents: [what kind, how many, how long, how often they change]
- Things that exist only in someone's head: [list them]

For each question, do all of this:

1. Name the gap: private, current, domain-specific, or none of the three.
2. Route it: put-it-in-the-prompt, give-it-a-tool, retrieve-it, or fine-tune. Justify in one line and estimate the token cost of your choice.
3. If you route it to retrieval, tell me what would have to be true for the prompt or a tool to be enough instead - I would rather not build a retrieval system I do not need.
4. State what the correct REFUSAL looks like when the fact is unavailable, in the user's words.
5. State what the citation should look like, and what a human would open to check it.
6. State what permission filter must apply before any search runs, given my user types.

Then across the whole product:
A. Which of my questions have no answer anywhere, and therefore need a human to write something before any AI work helps?
B. How much of my data would fit in a cached system prompt, with a token estimate?
C. What would I be fine-tuning for, if anything - and say plainly if the answer is nothing.
D. What private data would leave my building under your plan, and what should I strip before it does?
```

## Stuck? Ask AI
```prompt
Explain why fine-tuning a model on my company documents does not reliably teach it the facts in those documents. Use a concrete example of a price that changes. Then tell me what fine-tuning IS good for, and how I would know if my problem is one of those.
```

```prompt
I have a model, a database and a pile of PDFs. Explain the difference between putting facts in the prompt, giving the model a tool, and retrieving documents - what each costs, what each gets right, and what each gets wrong. Use one canteen example that needs all three.
```

```prompt
My AI feature gives a confident answer to a question my company has never written a policy about. Explain exactly why this happens in terms of next-token prediction, then give me the three changes - a rule, an example and a schema field - that would make it say "I do not know" instead.
```

## Quiz

### Q1 · L2
Your product needs today's sold-out dishes. Which route fits best?
- (a) Fine-tune nightly on the day's menu
- (b) Give the model a tool that reads the live stock row ✅
- (c) Embed the menu and retrieve it
- (d) Rely on the model's training data
> Structured data that changes belongs behind a tool, which reads the current row at the moment of asking. Embeddings add staleness and imprecision for no gain.

### Q2 · L3
A team wants their bot to know their forty-row menu. What is the right first move?
- (a) Build a vector store and retrieve the relevant rows
- (b) Put the menu in the system prompt, in the cacheable block ✅
- (c) Fine-tune on the menu
- (d) Add a search tool over the menu table
> Forty rows is a few hundred tokens: exact, cheap, cacheable and with no retrieval step to get wrong. Retrieval for data this small is usually less accurate, not more.

### Q3 · L4
Why is fine-tuning a poor way to teach a model your prices?
- (a) Fine-tuning requires too much data to be practical
- (b) Facts come out blurred, cannot be cited, and go stale until you retrain ✅
- (c) Providers do not allow numeric data in fine-tuning sets
- (d) Fine-tuned models cannot use tools
> Fine-tuning teaches shape, not retrievable fact. You cannot cite a weight or edit one, whereas changing a price should be one row in a table.

### Q4 · L4
You supply five retrieved chunks and none contains the answer. The model gives a confident answer anyway. What is missing?
- (a) A larger context window
- (b) More chunks
- (c) A refusal rule, an example of abstaining, and a schema field for it ✅
- (d) A reasoning model
> Grounding without permission to abstain produces better-dressed hallucinations. "I do not know" has to be stated as a rule, demonstrated, and representable in the output.

### Q5 · L5
A student asks about the refund policy and gets a correct answer citing a staff-only supplier circular. What has gone wrong?
- (a) Nothing; the answer was accurate
- (b) The citation format is wrong
- (c) Retrieval is not permission-aware, so it searched documents this user may not see ✅
- (d) The chunk size is too large
> Correctness is not the only requirement. The permission filter must be applied before the search runs, exactly as row-level security works for ordinary queries.

## Go deeper
- [Anthropic: Reduce hallucinations](https://docs.anthropic.com/en/docs/test-and-evaluate/strengthen-guardrails/reduce-hallucinations)
- [OpenAI: Fine-tuning versus retrieval, in the fine-tuning guide](https://platform.openai.com/docs/guides/fine-tuning)
