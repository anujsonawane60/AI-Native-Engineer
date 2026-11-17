---
id: m19-04-need-a-blockchain
module: m19-frontier-electives
slug: do-you-need-a-blockchain
title: "Do You Even Need a Blockchain?"
order: 4
duration_min: 25
is_free: false
mission: true
volatility: low
last_reviewed: 2026-11-16
---

## Key learnings
- Run a five-gate decision tree and reach an honest verdict for your own product
- Name the one condition that actually requires a ledger
- Design the cheaper answer: an append-only log, hash-chained and anchored
- Defend the verdict "no" in a pitch without sounding unambitious
- Spot an AI that builds what you asked for instead of asking who distrusts whom

## Lesson

### The pain

A team presents at a mentor review. Slide four says "blockchain-based verified skill credentials". The mentor asks one question: "Who are the parties that do not trust each other?"

It is supposed to be colleges and employers. But the team issues the credentials itself, from its own database, after its own tests. There is one writer, and the employers will never run nodes. The second question: "If you simply signed a PDF, what would an employer lose?" Nothing.

This is not stupidity — the word blockchain genuinely unlocks rooms. The failure is never separating the pitch word from the design word, so the team pays the full cost of a ledger for a property it already had free. The most likely conclusion of this chapter is that you do not need a blockchain. That is a real engineering result, and saying it with evidence beats any amount of enthusiasm.

### The idea

**A distributed ledger earns its cost only when several parties who do not trust each other must share one write path, and no one of them is allowed to be the referee.**

Everything else people say they want — tamper evidence, an audit trail, proof a document existed on a date, cryptographic receipts, transparency — comes from a database, a hash chain and a signature, at roughly one-thousandth of the cost and with the ability to fix a mistake. So the question is never "should we use blockchain". It is "is there a referee, and if there is, why are we paying to remove him?"

### The analogy

The canteen register one last time. Chapter 2 had five outlets arguing over five copies with no referee — but the canteen chain owns all five outlets, employs all five managers, and will be sued if the points are stolen. Accountability sits in one organisation, so write authority can too.

Now change the facts. Twelve independent colleges pool a mess-coupon scheme, each collecting money, honouring the others' coupons and settling monthly. None will accept another's database as the source of truth, because whoever holds it can quietly favour its own students. Now there is no referee, and the argument for a shared ledger is real.

**Where the analogy breaks**, and this is the honest part: even here the usual winning answer is not a blockchain. It is a shared database run by a neutral body — a trust, a consortium, a clearing house — with every entry signed by its author and a daily digest published where all twelve can see it. That gives proof of tampering without the fees, without permanence you cannot undo, and without a lost key destroying a term's coupons.

### Picture it

```
Does more than ONE organisation need to WRITE?
   │ no ──▶ database. Stop here.
   ▼ yes
Is there a party all of them would accept as referee?
   │ yes ──▶ database run by that party. Stop here.
   ▼ no
Do the writers actively distrust each other's records?
   │ no ──▶ shared database + signed entries. Stop here.
   ▼ yes
Must the record be permanent even against a court order?
   │ no ──▶ hash-chained audit log + daily published digest.
   ▼ yes
Can you accept tens of writes per second, public data,
real fees per write, and no way to delete anything ever?
   │ no ──▶ you cannot use a ledger even though you qualify.
   ▼ yes ──▶ a ledger is defensible. Write down why.
```

Five gates, and almost every student product stops at the first or second, which is the expected and correct outcome rather than a sign the analysis went wrong.

```
  The cheaper answer that covers 90% of stated requirements
┌──────────────────────────────────────────────────────┐
│ append-only table   each row signed by its author    │
│        │            each row stores prev row's digest│
│        ▼                                             │
│ daily digest ──▶ published where you cannot edit it: │
│                  a newspaper notice, a public page   │
│                  held by an auditor, a notary, or    │
│                  one transaction on a public chain   │
└──────────────────────────────────────────────────────┘
```

A signed, hash-chained, append-only log whose daily digest is published outside your control gives tamper evidence and third-party verifiability at ordinary database cost, and the last option shows you can borrow a chain for one write a day without building on one.

### Where it lives

This is M01's decision log doing its actual job. "We considered a distributed ledger for credential issuance and chose a signed, hash-chained audit table, because there is one writer and employers accept our signature; we would revisit this if two or more independent issuers joined" is a model entry. At M11's checkpoints and M18's demo day somebody will ask why you did not use blockchain, and that sentence is the whole answer.

Technically it sits on M04 — an append-only table, a signature column, a digest column over the row plus the previous digest — with M07's discipline protecting the signing key. The regulatory cost lands in M17: India treats virtual digital assets as taxable property rather than currency, and the DPDP Act expects erasure on request, which on-chain is impossible. A "no" here removes a whole compliance workstream.

### Words to own

| Term | Plain meaning | Example |
|---|---|---|
| Trust boundary | The line across which parties stop believing each other | Between two rival companies |
| Write authority | Who is permitted to add to the record | Your backend, today |
| Referee | A party everyone accepts as the tie-breaker | A bank, a clearing house, you |
| Append-only log | A table rows are added to and never edited | An audit table |
| Hash-chained log | Each row stores the digest of the row before | Tamper-evident history |
| Tamper-evident | You can tell it was changed | A broken digest chain |
| Tamper-proof | It cannot be changed at all | Rare, expensive, and rarely needed |
| Anchoring | Publishing a digest somewhere you do not control | One daily on-chain write |
| Notarisation | A third party attesting something existed by a date | A timestamping service |
| Disintermediation | Removing the middleman | The only thing a ledger uniquely does |

### Tradeoffs and gotchas

**A private chain is usually a slow database with extra vocabulary.** If you run every node and ship the software, you have the trust model of a database you own plus operational complexity.

**Permanence cuts both ways.** You wanted "nobody can change the record". You also got "nobody can remove the record", including the row holding a minor's phone number.

**Cost per write compounds.** Multiply chapter 3's rupee figure by a year of writes, then compare a free-tier database. The gap is usually three or four orders of magnitude.

**The pitch word and the design word are different words.** Verifiable credentials, cryptographic receipts and public auditability are all true of the cheap design. Claiming a chain you do not have is what ends badly in diligence.

**Sometimes the honest answer is "yes, and we still won't".** You may pass all five gates and decline anyway, because the throughput, the public data or the irreversibility are unacceptable for your users. Write that down; it beats a vague no.

### Verify it

**Red flag: the AI builds it.** Ask "design a blockchain system for X" and a model produces an architecture, a token and a roadmap. A good answer asks who the distrusting writers are *before* drawing anything.

**Red flag: a proposed blockchain where a database with an audit log would do.** Ask for the database alternative designed with equal effort, then compare cost per write, throughput, deletability, and what breaks if a key is lost.

**Red flag: immutability presented as the benefit.** Immutability comes from a hash chain. The only property unique to a ledger is removing the referee; a justification that never mentions the referee is decoration.

**Red flag: a private chain called decentralised.** Ask who runs nodes, who can add one, and who ships the software. Three answers of "us" means you own a database.

**Red flag: no deletion story.** Ask what happens when a user exercises a data-deletion right, or a court orders a record removed.

## Walkthrough
1. Write your product's core record in one sentence: what is being recorded, by whom, and who reads it. Do not use the word blockchain anywhere in that sentence.
2. List every organisation that would need to add to that record — not read it, add to it. Count them. If the count is one, note that gate one has already decided your answer.
3. If the count is more than one, name a party all of them would accept as the tie-breaker. An existing regulator, a bank, a university, an industry body, or you.
4. For each writer, write one sentence on what they could gain by falsifying a record, and whether the others could detect it today.
5. Check your domain for deletion obligations: personal data, minors' data, payment data, anything a user could ask you to erase. Write down what permanence would prevent you doing.
6. Take the rupee-per-transaction figure you recorded in chapter 3 and multiply it by your realistic write volume for one year. Write that annual number next to zero, which is what a free-tier database charges per write.
7. Walk the five-gate tree with these answers and write down the gate at which you stopped and the verdict it produced.
8. Design the cheaper alternative in five lines: the append-only table, what each row holds, who signs it, how the digest chains to the previous row, and where the daily digest gets published outside your control.
9. Write the single condition that would change your verdict — a new independent writer, a regulator mandating shared custody, a counterparty refusing your signature — and state how you would notice it had happened.
10. Turn all of the above into one decision-log entry of under 150 words, in the format from M01: what you chose, what you rejected, why, and the trigger to revisit.

## Talk to AI
```prompt
CONTEXT:      I am a student founder. Someone has suggested my product
              should use a blockchain. My product is: [describe it in three
              sentences, including who writes records and who reads them].
              My realistic write volume is [X] records per day. My users are
              in India.

GOAL:         An honest verdict on whether a distributed ledger is justified,
              and a designed alternative if it is not.

REQUIREMENTS: 1. Before designing anything, ask me the questions you need
                 answered - especially who the writing parties are and
                 whether any of them distrusts the others
              2. Walk me through a decision tree with explicit gates and
                 tell me which gate my product stops at
              3. Design the non-blockchain alternative in equal detail: an
                 append-only, signed, hash-chained log, including where the
                 digest must be published for a third party to trust it
              4. A comparison table of the two designs: cost per write in
                 rupees, writes per second, who can delete a record, what
                 happens when a key is lost, and the compliance work each
                 one creates in India
              5. The one condition that would flip your verdict

CONSTRAINTS:  No code. Plain English. Assume free tiers only. Do not assume
              my users own crypto wallets or understand gas. Treat tax and
              regulatory specifics as things I must verify against the
              current rules.

NON-GOALS:    Do not design a token. Do not recommend a chain. Do not try to
              make the blockchain option sound viable if it is not.

ACCEPTANCE:   - You asked me who the distrusting writers are before
                proposing any architecture
              - Your verdict names the gate my product stopped at
              - Your alternative design says where the daily digest is
                published and why that location matters
              - Your table has a number in every cost cell
              - You state plainly whether my product needs a ledger, in one
                unhedged sentence

ASK:          Ask your questions first. Do not design anything until I have
              answered them.
```

## Mission

**Objective.** Produce a written, evidenced verdict on whether your own product needs a distributed ledger, together with a designed alternative — and a decision-log entry you could defend at demo day.

**Steps.**
1. **Frame** — write your product's core record in one sentence: what is recorded, by whom, for whom. Ban the word blockchain from this sentence.
2. **Count the writers** — list every independent organisation that must *add* to that record, and name a referee all of them would accept. Be honest if that referee is you.
3. **Brief** — use the Talk to AI prompt with your real details. Make the AI ask its questions before it designs anything, and record what it asked.
4. **Walk the tree** — go through all five gates in writing, with your answer and one line of evidence at each gate. Record the gate you stopped at.
5. **Cost it** — multiply chapter 3's rupee-per-transaction figure by your annual write volume. Put that number beside the free-tier database cost.
6. **Design the alternative** — specify the append-only table: columns, who signs each row, how the digest chains, where the daily digest is published outside your control, and who verifies it.
7. **Verify** — give your verdict and your alternative design to a second AI and ask it to attack both. Record the strongest objection it raised and your response to it.
8. **Document** — write the decision-log entry: chosen, rejected, why, and the single trigger that would make you revisit.

**Upload.**
- Your five-gate walkthrough with evidence at each gate → post caption or attached file
- Your annual cost comparison, with the assumptions stated → screenshot or table
- Your alternative design, as a table definition and a one-paragraph description of the anchoring step
- The AI conversation where it asked its questions before designing → share link in `ai_chat_url`
- The second AI's strongest objection and your written response
- Your decision-log entry, 150 words maximum
- Reflection, 150 words max → what you expected the verdict to be before you started, and one thing an AI got wrong

**Self-check.** Tick every box before you submit.
- [ ] My core-record sentence does not contain the word blockchain
- [ ] I counted the independent writing organisations and wrote the number down
- [ ] I named a referee, even if the referee is me
- [ ] I recorded which of the five gates my product stopped at
- [ ] My cost comparison has a rupee number and a stated assumption behind it
- [ ] My alternative design says where the digest is published and why that place is outside my control
- [ ] I checked whether my domain has data-deletion obligations
- [ ] My verdict is one unhedged sentence
- [ ] I named the single condition that would change the verdict and how I would notice it
- [ ] My decision-log entry is short enough that someone would actually read it

**Stretch.** Find a real company that advertises a blockchain product in your sector, walk their public description through the same five gates, and write 150 words on which gate they would stop at and what they appear to be buying.

## Stuck? Ask AI
```prompt
I keep hearing that blockchain gives immutability and auditability. Explain to me, in plain English with no code, exactly which of those two properties I can get from an ordinary database with an append-only table and signatures, and which one I genuinely cannot. Then tell me the single property a blockchain provides that nothing else does.
```

```prompt
Give me six short product descriptions - one college credential system, one inter-bank settlement, one farm supply chain with many independent traders, one single-company loyalty scheme, one land registry, one hospital group sharing patient records. For each, ask me which gate of the decision tree it stops at. Do not tell me your answers until I have given all six with reasons.
```

```prompt
My mentor will ask why I did not use blockchain. Help me rehearse. Act as a sceptical investor who believes blockchain is essential for my product: [describe it]. Challenge my answer three times, each time with a different objection, and tell me afterwards which of my three responses was weakest and why.
```

## Quiz

### Q1 · L3
What is the only property a distributed ledger provides that a signed, hash-chained database table does not?
- (a) Tamper evidence
- (b) An auditable history
- (c) Removing the need for any single trusted referee ✅
- (d) Cryptographic receipts for users
> Tamper evidence, audit history and receipts all come from hashing and signing. Disintermediation is the one thing you cannot get any cheaper, and it is the only honest justification.

### Q2 · L3
Your product has exactly one organisation writing records: yours. Where does the decision tree stop?
- (a) At the permanence gate, since records should still be immutable
- (b) At the first gate, because a single writer needs no shared write path ✅
- (c) At the throughput gate, after costing the writes
- (d) It does not stop; a private chain is still worth considering
> One writer means there is nothing to agree on. The later gates only matter once multiple independent organisations must add to the same record.

### Q3 · L4
A team proposes a private chain where their company runs all the nodes. What is the most accurate description of what they have built?
- (a) A decentralised system with controlled membership
- (b) A database they own, with extra operational cost and a worse delete story ✅
- (c) A tamper-proof system no insider can alter
- (d) A system that removes the need for a referee
> If one party runs every node and ships the software, the trust model is unchanged from a database. Insiders can still rewrite by consensus of themselves.

### Q4 · L4
Which of these requirements is the strongest signal that a ledger is the wrong choice regardless of the trust model?
- (a) Records must be auditable by an external party
- (b) Records must be timestamped reliably
- (c) Personal data must be erasable on user request ✅
- (d) Multiple teams must be able to read records
> Append-only permanence and a deletion obligation are in direct conflict. Auditability, timestamping and shared reads are all satisfiable without a ledger.

### Q5 · L5
You ask an AI to "design a blockchain system for verified skill credentials" and it returns a full architecture with a token and a roadmap. What should you conclude?
- (a) The design is sound because it is detailed
- (b) It answered the question you asked instead of testing whether the question was right ✅
- (c) It has identified a genuine need for disintermediation
- (d) It must have assumed a permissioned chain
> A model given a solution-shaped prompt will deliver that solution. The missing step is the one you must supply: name the distrusting writers first, and only then ask for an architecture.

## Go deeper
- [Wüst and Gervais: Do you need a blockchain?](https://eprint.iacr.org/2017/375.pdf)
- [NIST IR 8202: Blockchain Technology Overview, section on when not to use it](https://nvlpubs.nist.gov/nistpubs/ir/2018/NIST.IR.8202.pdf)
