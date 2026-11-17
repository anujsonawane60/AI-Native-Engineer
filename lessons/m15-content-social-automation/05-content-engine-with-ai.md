---
id: m15-05-content-engine
module: m15-content-social-automation
slug: content-engine-with-ai
title: "A Content Engine With AI, Without Sounding Like AI"
order: 5
duration_min: 15
is_free: false
mission: false
volatility: medium
last_reviewed: 2026-11-16
---

## Key learnings
- Name the specific tells that mark a piece of writing as AI-generated
- Direct content with evidence, a point of view and bans instead of a topic
- Run a human pass that adds a fact the model could not possibly have known
- Keep a human gate before publish, so the engine can never run on its own
- Explain why undisclosed AI content at scale costs more trust than it earns traffic

## Lesson

### The pain

A team sets up a content engine: three posts a day for thirty days, ninety pieces generated in four hours of prompting. By week three, impressions across all ninety are a few hundred and clicks are eleven, and their mentor replies with the sentence that should end every content-engine fantasy: *"I read three of these and I still have no idea what you actually think."* Then post 34 surfaces: "73% of Indian students skip breakfast, according to recent studies." No such study exists — the model produced a plausible number and a plausible attribution. A nutritionist screenshots it and asks publicly where the figure came from. Ninety posts of filler cost them little; one invented statistic cost them their only credibility.

### The idea

**Volume without a point of view is worth zero — at scale, less than zero, because it spends trust you cannot buy back. The engine's job is not to produce words but to turn things only you know into things people want to read.**

This is the content side of M03 chapter 8: a vague brief produces the statistical average of all writing, exactly as it produces the purple gradient. Four levers fix it.

**Evidence only you have.** Order times, cancellation reasons, the price people actually pay, the words users used in your chapter 1 interviews, what your team got wrong in week three. None is in any training set.

**A point of view someone could disagree with.** Apply the disagreement test: could a reasonable person in your field object? "Good service matters" fails. "Menu variety is a distraction; evening-slot capacity decides survival" passes, which is why it got forwarded into three WhatsApp groups of food-business owners.

**Constraints and a written ban list, not a topic.** "Write about tiffin services" gets you the average; a usable brief names the audience, the claim, the evidence and the length ceiling. Its strongest lines are the prohibitions, because they are checkable: no statistic without a named source, no sentence beginning "In today's", no word from a named list.

Then the rule that decides whether a draft ships. **An edit must add something the model could not have known** — one number from your data, one sentence a user said, one thing you tried that failed. Rephrasing average writing produces average writing in a better mood.

### The analogy

A canteen with forty dishes on the board, every one the same base gravy with a different name and price. Nothing is bad; nothing is memorable, and a student asked "what's good there?" cannot answer. Across the corridor one stall sells one misal and people travel to it, because the owner has an opinion about how much tarri belongs on top.

**Where the analogy breaks** in the way that creates the problem: forty dishes a day is impossible for three staff, so real canteens are forced into focus. AI removes the force — ninety posts cost one afternoon — so the discipline scarcity supplied free must now be chosen on purpose.

### Picture it

```
 ┌──────────┐ ┌─────────┐ ┌────────┐ ┌────────┐ ┌─────────┐
 │ EVIDENCE │ │ BRIEF   │ │ DRAFT  │ │ HUMAN  │ │ PUBLISH │
 │ your own │ │ POV +   │ │ AI     │ │ GATE   │ │ one     │
 │ numbers, │▶│ refs +  │▶│ writes │▶│ adds a │▶│ channel │
 │ user     │ │ bans    │ │ to the │ │ fact,  │ │ first   │
 │ words,   │ │         │ │ brief  │ │ cuts   │ │         │
 │ failures │ └─────────┘ └────────┘ │ filler │ └─────────┘
 └──────────┘                        └───┬────┘
      ▲                                  │ rejected
      └────── nothing to say yet ◀────────┘
```

Content starts from evidence you own rather than a topic, and the human gate can send a piece back to "nothing to say yet" — a healthy output of the engine.

### Where it lives

M03 chapter 8 is this chapter's sibling — the same failure on the visual surface, same cure — and M10's prompting work is the machinery of the brief. Chapter 6 turns your bans and voice into a reusable `VOICE.md`, chapter 9 multiplies one good idea into ten formats, chapter 10 measures whether anybody cared. The evidence feeding the pipeline is M04's database and M07's events.

### Words to own

| Term | Plain meaning | Example |
|---|---|---|
| Content engine | A repeatable pipeline from evidence to published | Evidence, brief, draft, gate |
| Point of view | A claim a reader could argue with | "Variety is a distraction" |
| Slop | High-volume, low-value generated content | 90 posts, 11 clicks |
| Tell | A signature that marks writing as generated | "In today's fast-paced…" |
| Throat-clearing | Sentences before the actual point | A whole opening paragraph |
| Hallucinated citation | A source or statistic that does not exist | "73%, studies show" |
| Human gate | A person who must approve before publish | One teammate, every piece |
| Ban list | Written prohibitions for the brief | "No unsourced statistics" |
| Disagreement test | Could a reasonable person object? | If no, there is no claim |
| Topic vs angle | Subject versus the specific thing said | "Tiffin" vs the 8:40 pm claim |
| Disclosure | Saying AI helped make this | "Written with AI, facts ours" |

### Tradeoffs and gotchas

**Pick the cadence your human gate can sustain.** If one teammate can genuinely edit two pieces a week, your engine produces two a week; an engine that outruns its gate has no gate. One good piece a week beats five weak ones a day once you measure forwards and replies, not posts published.

**AI is excellent at the parts around the writing.** Outlines, titles, cutting your draft by 30%, turning one piece into chapter 9's ten formats, finding the hole in your argument. What it cannot do is hold your opinion or know your numbers: the model is average by construction.

**Disclose assistance, and never claim an experience you did not have.** Search engines penalise not AI help but content made at scale with no value to a reader, so the question is never "was AI involved" but "is there anything here". Publishing generated material as personal experience is a lie — a review you did not write, a case study that did not happen — and at scale a systematic one.

**Machine translation is not localisation.** A Marathi version translated and published unread will contain phrasing no Marathi speaker would use. Someone must read it aloud before it ships.

### Verify it

You cannot avoid what you cannot name. These tells shift every few months, so treat the list as a method, not scripture.

| Tell | What it looks like | What to do instead |
|---|---|---|
| Throat-clearing opener | "In today's fast-paced digital world…" | Open on the fact or the number |
| The fake contrast | "This isn't about X. It's about Y." | Say the one thing you mean |
| Register words | delve, leverage, harness, unlock, elevate | Plain verbs |
| Everything in threes | Triple adjectives, triple clauses | Vary; cut two of the three |
| Hedged to nothing | "can often help to potentially" | A claim you will defend |
| The empty listicle | 7 tips, each restating its heading | Three tips with real detail |
| Fake authority | "studies show", "experts agree" | A named source, or nothing |
| The recap close | "In conclusion, as we've seen…" | Stop at the last real point |
| Emoji as structure | 🚀 ✨ between every section | Words |
| Uniform paragraphs | Every block exactly four lines | Length follows the thought |
| No proper nouns | No names, prices, dates or places | Nashik, ₹1,800, 8:40 pm |
| The closing nudge | "So what are you waiting for?" | Nothing, or one real ask |

Several of these are ordinary writing habits; a list is not a crime. The problem is a piece where *every* choice is the default one, and the giveaway is the last row.

**Red flag: no point of view.** Apply the disagreement test to the main claim. If nobody could object, nothing was said.

**Red flag: no proper nouns and no numbers.** Zero names, prices, dates or places means the piece could be about any company anywhere.

**Red flag: an unsourced statistic.** Any percentage, ranking or "studies show" needs a named source you opened yourself. Assume fabrication; delete what you cannot trace.

**Red flag: the swap test.** Put a competitor's name in place of yours. If the piece still works, it contains nothing of yours — M03 chapter 8's test, applied to prose.

**Red flag: an edit that only rephrases.** No new fact, quote or number in the diff means the piece has not earned publication.

**Red flag: you would not send it to one person.** If you would be embarrassed to WhatsApp this to one real user, do not publish it to a thousand.

## Walkthrough
1. Open your own database or analytics and write down three numbers nobody outside your team knows. Real counts, times, prices or rates — not estimates.
2. Open your chapter 1 interview notes and copy out three sentences in the exact words a user used, including the Hinglish.
3. Write down two things your team got wrong in the last month and what you learned. These are content, and they are the hardest kind to fake.
4. From those eight items, write ONE claim in a single sentence that a reasonable person in your field could disagree with. Read it to a teammate and ask them to argue the opposite. If they cannot, rewrite it.
5. Write a ban list of eight prohibitions for your brief. Each must be checkable by reading the draft — "no statistic without a named source" is checkable; "not too generic" is not.
6. Brief AI with the claim, the evidence, the audience, a length ceiling and the bans. Ask for the shortest piece that makes the claim, not the longest.
7. Read the draft once and mark every tell from the table above. Count them. Three or more means the brief lost.
8. Do the human pass and keep a note of exactly what you added. If you added no number, quote or specific, stop and go back to step 1.
9. Run the swap test: put a competitor's name in place of yours and reread. Then delete the first paragraph and reread. Keep whichever version is better.
10. Publish to one channel only, and write down the one number from chapter 10 you will check in seven days: replies, forwards or clicks. Not impressions.

## Talk to AI
```prompt
CONTEXT:      I am a student founder. I direct AI; I do not write code.
              My product: [one sentence]. My audience for this piece:
              [who, where, what language, what they already know].

              EVIDENCE ONLY I HAVE - use only these facts, and add no
              others:
              - Numbers from my own data: [paste 3, with what they count
                and over what period]
              - Things real users said, in their words: [paste 3 quotes]
              - Things my team got wrong and learned: [paste 2]

              MY CLAIM, which a reasonable person could disagree with:
              [one sentence]

GOAL:         The shortest possible piece that makes my claim using only
              my evidence, in a voice that could not be mistaken for
              generic content marketing.

REQUIREMENTS: 1. Open on a number or a fact from my evidence. No scene
                 setting, no context paragraph.
              2. Every factual statement must trace to one of my pasted
                 items. Mark each one with which item it came from.
              3. State my claim plainly, once, and defend it with my
                 evidence
              4. Name the strongest objection to my claim and answer it
                 honestly, including where my evidence is weak
              5. Under 450 words. Shorter is better. Do not pad.

BANS - never produce any of these:
              - Any statistic or "studies show" not in my evidence
              - An opening of the form "In today's..." or "In the world
                of..."
              - The words delve, leverage, harness, unlock, elevate,
                seamless, robust, landscape, realm, testament
              - A sentence of the form "This isn't X, it's Y"
              - Emoji anywhere
              - A concluding recap paragraph
              - A closing call to action of the form "So what are you
                waiting for"
              - Any claim about my product I have not given you evidence
                for

ACCEPTANCE:   - Zero facts that are not traceable to my evidence list
              - At least four proper nouns or numbers in the piece
              - Zero bans violated
              - If my product name were swapped for a competitor's, the
                piece would read as false - and you can say why
              - A final section titled "What I left out and why",
                including anything my evidence did not support

ASK:          First, before writing, tell me what a generic version of
              this piece would say if you had none of my evidence -
              describe it in detail so I can recognise it. Then tell me
              which single item of my evidence does the most work. Only
              then write the piece.
```

## Stuck? Ask AI
```prompt
Here is a draft: [paste it]. Do not rewrite it. Instead, tell me in one sentence what claim it makes, then tell me whether any reasonable person could disagree with that claim. Then list every factual statement in it and mark each one as traceable to a named source, or not. Do not improve anything — just diagnose.
```

```prompt
Here is a piece I wrote with AI: [paste it]. Go through it and mark every sentence that is a known AI tell — throat-clearing, fake contrast, register words, unsourced statistics, recap closings, hedging. Give me the count. Then tell me which single cut would improve it most, and stop there.
```

```prompt
I have no idea what to write about. Do not suggest topics. Instead, ask me questions about my own product's data, my users' actual words and my team's mistakes, one at a time, until I have said something that you could not have known. Then tell me which of my answers is the most interesting and why.
```

## Quiz

### Q1 · L3
Which draft is most likely to be worth publishing?
- (a) 1,200 words covering the industry broadly, with three statistics from "recent studies"
- (b) 400 words built on one number from your own database and one claim a competitor would argue with ✅
- (c) A seven-item listicle where each item restates its own heading
- (d) 900 words of balanced overview with no position taken
> Specific evidence plus a contestable claim is the only combination nobody else can produce. Breadth, unsourced statistics and balance with no position are all signatures of the average.

### Q2 · L4
You edit an AI draft by tightening sentences and cutting a paragraph. What is still missing?
- (a) Nothing — editing for concision is the human pass
- (b) Anything the model could not have known: a number, a quote or a specific ✅
- (c) A stronger call to action at the end
- (d) Keywords for search
> Rephrasing average writing produces average writing. The human pass is defined by addition, not polish, so a diff with no new fact in it means the piece has not earned publication.

### Q3 · L4
A generated post includes "73% of Indian students skip breakfast, according to recent studies", and the study does not exist. Why is this worse than ninety bland posts?
- (a) It will be penalised by the search algorithm immediately
- (b) Bland posts are forgettable, but one fabricated claim permanently costs credibility you cannot rebuy ✅
- (c) It breaks copyright rules
- (d) Statistics are always unnecessary in short posts
> Dullness is cheap; a traceable false claim is not. Treat every statistic as fabricated until you have opened the source yourself, and delete anything you cannot trace.

### Q4 · L3
Your content engine produces fifteen pieces a week, and your one available editor can genuinely edit two. What should you change?
- (a) Publish all fifteen and edit the best performers afterwards
- (b) Reduce output to two a week, because an engine that outruns its gate has no gate ✅
- (c) Add an AI reviewer step instead of the human one
- (d) Keep fifteen but shorten each piece
> The gate is the only thing standing between your engine and slop. Output is capped by the human pass, not by how fast the model can write.

### Q5 · L5
You swap your product name for a competitor's in a finished post and it reads perfectly. What does that prove, and what is the fix?
- (a) The writing is versatile, which is a strength
- (b) Nothing in the post came from your own evidence; put your data, your users' words or your mistakes in it ✅
- (c) The post needs more keyword mentions of your brand
- (d) The competitor is too similar to you
> This is the swap test from M03 chapter 8 applied to prose. A piece that fits anybody was shaped by nothing of yours, and only your own numbers, quotes and failures can fix it.

## Go deeper
- [Google Search Central: creating helpful, reliable, people-first content](https://developers.google.com/search/docs/fundamentals/creating-helpful-content)
- [Julian Shapiro: a guide to writing well](https://www.julian.com/guide/write/intro)
