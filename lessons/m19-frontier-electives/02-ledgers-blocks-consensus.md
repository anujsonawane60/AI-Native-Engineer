---
id: m19-02-ledgers-consensus
module: m19-frontier-electives
slug: ledgers-blocks-consensus
title: "Distributed Ledgers, Blocks and Consensus"
order: 2
duration_min: 15
is_free: false
mission: false
volatility: low
last_reviewed: 2026-11-16
---

## Key learnings
- Explain why each block carries the previous block's digest
- State what consensus is actually solving, in one sentence
- Compare proof of work, proof of stake and permissioned consensus by what each costs
- Quote the real throughput range of a public chain against UPI
- Spot an AI claim that confuses immutability with correctness

## Lesson

### The pain

The canteen chain grows to five outlets and the loyalty register is copied to all five, because a student who earns points at the hostel mess wants to spend them at the main canteen.

Within a month it is a mess. Outlet three's copy says Asha has 60 points; outlet one's says 35. Both were written in good faith — she earned at one and spent at the other, and the copies were updated an hour apart over a Wi-Fi link that dropped. Every evening the managers argue about whose copy is right, and the only tie-breaker is seniority.

The sealed, signed pages from chapter 1 do not help: every page is perfectly sealed and signed. Five honest copies disagreed about the *order* things happened in, and nobody is allowed to be the referee. How machines that do not trust each other agree on one order without a referee is what a distributed ledger answers, and understanding its price is the point of this chapter.

### The idea

**A blockchain is an append-only log, copied to many machines, where each batch of entries carries the digest of the batch before it, and a public rule decides which copy counts as the real one.**

Three pieces, each doing one job. The log is append-only: you add, you never edit. The chaining of digests means altering any old entry changes that block's digest, which breaks the next block's stored copy, and so on — so tampering is detectable from the latest block alone. Consensus is the rule, written into the software, that every node applies to pick one chain when two valid chains exist.

### The analogy

Each page of the rebuilt register holds twenty transactions, and at the top of every page the manager copies the foil pattern from the bottom of the previous one. Rub out an entry on page four and the pattern no longer matches what page five recorded, and so on to the end of the book.

Now remove the manager. Five outlets each keep the whole book and each wants to write the next page. The rule they adopt: whoever solves a pointless, expensive puzzle first writes the page, and if two books differ everyone switches to the one with more puzzle-solving in it. That is proof of work, as absurd and as effective as it sounds.

**Where the analogy breaks**, and this is the most important paragraph here: the canteen *has* a manager, who is trusted, accountable and cheap. Every cost of a blockchain — the energy, the slowness, the growing storage, the impossibility of fixing a mistake — is the price of not having him. Chapter 4 makes that question explicit.

### Picture it

```
   Block 11            Block 12            Block 13
┌─────────────┐     ┌─────────────┐     ┌─────────────┐
│ prev: 4a9f  │     │ prev: c20d  │     │ prev: 8e71  │
│ 20 entries  │     │ 20 entries  │     │ 20 entries  │
│ digest c20d │────▶│ digest 8e71 │────▶│ digest f33b │
└─────────────┘     └─────────────┘     └─────────────┘
       ▲
       └── edit any entry here and c20d changes, so block 12's
           "prev" no longer matches, and the break is visible
```

Each block stores the digest of the one before it, so a change anywhere in history invalidates every block that follows and is detectable by checking only the newest block.

```
   node A ──┐
   node B ──┤                   ┌── longest / heaviest chain wins
   node C ──┼── two valid ──────┤
   node D ──┤     chains        └── the other chain's blocks are
   node E ──┘                       discarded ("reorg")
```

When honest nodes temporarily disagree, the consensus rule picks one chain and the losing chain's transactions go back to being unconfirmed, which is why confirmation is a matter of degree rather than a yes.

### Where it lives

This is M05 wearing a costume: replication, drift, and the fact that making many copies agree costs either latency or consistency. A blockchain chooses extreme durability and auditability, and pays with throughput that would embarrass a single laptop.

The numbers matter. Bitcoin settles roughly seven transactions a second with an hour's wait for practical finality; Ethereum does low tens per second at the base layer; a permissioned consortium ledger does hundreds to a few thousand. UPI processes well over 600 crore transactions a month — roughly 2,500 a second on average, several times that at peak — with settlement you feel as instant and a dispute process you can escalate to a human. So the reason to use a ledger is never performance. It is only ever that no single party is allowed to hold the write authority.

### Words to own

| Term | Plain meaning | Example |
|---|---|---|
| Ledger | An append-only record of transactions | The loyalty register |
| Block | A batch of entries, sealed together | 20 transactions plus a header |
| Previous hash | The digest of the block before | What makes it a chain |
| Node | A machine holding a copy and checking rules | Your laptop running a client |
| Mempool | Transactions waiting to be included | The queue before a block |
| Fork | Two valid chains existing at once | A brief reorg, or a rule change |
| Finality | The point at which reversal is impractical | Six Bitcoin blocks, roughly an hour |
| Proof of work | Write rights earned by spending computation | Bitcoin mining |
| Proof of stake | Write rights earned by locking up value | Ethereum validators |
| 51% attack | One party controlling enough power to rewrite | Fatal to small chains |
| Permissioned ledger | Known members, no open participation | A bank consortium's chain |

### Tradeoffs and gotchas

**Immutability is not correctness.** A blockchain guarantees that what was written stays written, and nothing about whether it was true. A wrong entry, a fraudulent one, or one containing someone's Aadhaar number is now permanent, which under India's DPDP Act is a problem rather than a feature.

**Storage only grows, and everybody stores everything.** A full node keeps the whole history. Fine for a global currency, ridiculous for a canteen's points.

**"Decentralised" is usually a marketing claim.** A permissioned ledger among twelve independent banks is a real thing; one among twelve servers owned by a single startup is a slow database with extra vocabulary.

**Finality is probabilistic on proof-of-work chains.** "Confirmed" means "increasingly unlikely to be undone". Any product treating one confirmation as settlement has a reorg bug waiting.

**Energy and cost are real.** Proof of work converts electricity into write rights, which is why proof of stake exists. Say which one a system you are evaluating uses.

### Verify it

Ask an AI to design "a blockchain-based attendance system" and it will, fluently. Here is how to tell whether it reasoned.

**Red flag: it never asked who the distrusting parties are.** The most diagnostic question of all. A ledger exists because parties who do not trust each other need one shared write path; if the answer does not name them, it is matching a keyword.

**Red flag: immutability offered as a guarantee of truth.** Records cannot be *altered*. A false record entered correctly is immutable too, and now you cannot remove it even when the law says you must.

**Red flag: throughput quoted without naming the chain and the finality window.** "Thousands of transactions per second" is meaningless without the network, the finality, the fee and the validator count.

**Red flag: a private chain described as decentralised.** Ask who runs nodes, who can add one, and who ships the software. Three answers of "we can" means the trust model is a database you own.

**Red flag: no mention of what happens to a mistake.** Ask directly: a wrong entry is written, what now? An honest answer says you can only append a correction.

## Walkthrough
1. Open a free in-browser blockchain demonstration page — the kind with editable blocks that recompute their own digests live. Start on the single-block view.
2. Type any text into the data field and watch the digest change with every keystroke. Note how the digest never gets longer.
3. Move to the chain view, with four or five linked blocks. Confirm each block's header shows the previous block's digest.
4. Edit the data in block two. Note which blocks turn invalid and which stay valid, and write down why the ones after it broke.
5. Re-mine block two so it is valid again. Note that blocks three onward are still broken, and that you must re-mine each one in order.
6. Switch to the distributed view with several peers. Edit a block on peer A only, and compare the three peers' chains side by side.
7. Write one sentence describing how peer B could tell that peer A's copy had been tampered with, using only the last block of each.
8. Open a public block explorer for any major chain. Find the most recent block and note three things: its number, how many transactions it holds, and the timestamp gap to the block before it.
9. Divide the transactions in that block by the gap in seconds. Write down the transactions-per-second figure you get, and compare it with UPI's average of roughly 2,500 per second.
10. Finish with four lines in your notes: what the chaining proves, what consensus decides, what the measured throughput was, and one situation where that throughput would be unacceptable for your own product.

## Talk to AI
```prompt
CONTEXT:      I am a student learning engineering judgement, not writing
              code. I already understand hashing and digital signatures. I
              want to understand distributed ledgers well enough to review a
              design and challenge a vendor claim.

GOAL:         Understand what consensus actually solves and what each family
              of consensus costs.

REQUIREMENTS: 1. One paragraph stating the exact problem consensus solves,
                 phrased in terms of who does not trust whom
              2. A table comparing proof of work, proof of stake and
                 permissioned consensus, with columns for how write rights
                 are earned, roughly what throughput is achievable, how
                 finality behaves, and what it costs to attack
              3. A plain explanation of why a chain of digests makes old
                 entries tamper-evident, with no mathematics
              4. Three statements commonly made about blockchain that are
                 technically false, each with a one-line correction
              5. A one-paragraph comparison against a single replicated
                 SQL database with an append-only audit table

CONSTRAINTS:  No code. No formulas. Plain English for a second-year student
              reading in a second language. Name real networks and real
              numbers where you can, and say when a number is approximate.
              Under 900 words.

NON-GOALS:    Do not recommend any chain. Do not discuss tokens, prices or
              investment. Do not explain mining hardware.

ACCEPTANCE:   - Your consensus paragraph names the distrusting parties
              - Every throughput figure names the network it belongs to
              - You state plainly that immutability does not imply
                correctness
              - Your database comparison says what the database is better at,
                not only what it is worse at

ASK:          Give me the table first. Then ask me to describe a situation
              from my own project where no single party could be allowed to
              hold write authority, and tell me honestly whether my example
              actually qualifies.
```

## Stuck? Ask AI
```prompt
Explain to me, as if I have never seen a computer, why five copies of the same notebook can all be honest and still disagree about what happened. Use one everyday example, keep it under 200 words, and then tell me in one sentence what a blockchain adds to fix it.
```

```prompt
I do not understand why a block stores the previous block's fingerprint. Walk me through a four-block chain where somebody edits block two, and tell me exactly which checks fail and in what order. Then ask me what would happen if they also edited block three, and wait for my answer.
```

```prompt
Give me five short descriptions of real systems - a bank's internal ledger, a public cryptocurrency, a shipping consortium's record, a college attendance app, and a group of hospitals sharing records. Ask me to rank them by how much they genuinely need consensus without a referee. Do not reveal your own ranking until I have given mine and explained my reasoning.
```

## Quiz

### Q1 · L2
Why does each block store the digest of the previous block?
- (a) To compress the chain's storage
- (b) So that altering an old block invalidates every block after it ✅
- (c) To prove who created the block
- (d) To let nodes skip verifying old blocks
> The chaining is what makes history tamper-evident. Identity comes from signatures, not from the chain, and the chain makes storage grow rather than shrink.

### Q2 · L3
What problem is consensus solving?
- (a) How to keep transaction data secret from other nodes
- (b) How to compress a growing ledger
- (c) How machines that do not trust each other agree on one order of events ✅
- (d) How to make transactions faster than a database
> Consensus is about agreeing on order without a referee. It makes the system slower, not faster, and it provides no secrecy at all, since public chains publish everything.

### Q3 · L4
A vendor says their private chain is "decentralised and tamper-proof". Every node runs in their own cloud account. What is the correct response?
- (a) Accept it, since the digests still chain correctly
- (b) Ask how many independent organisations run nodes and can change the software ✅
- (c) Ask them to switch to proof of work
- (d) Ask for a throughput benchmark
> If one party controls every node and the software, the trust model is a database they own. The chaining is real; the decentralisation is not, and that is the part the claim rests on.

### Q4 · L4
Someone's phone number is written into a block by mistake and the user asks for it to be deleted. What is the honest answer?
- (a) The node operators can edit that block
- (b) A correction can be appended, but the original entry stays forever ✅
- (c) The chain can be rolled back to before that block
- (d) Deleting it is simple because the data is hashed
> Append-only means exactly that. This is the direct conflict between on-chain permanence and data-deletion rights, and it is a reason not to put personal data on a ledger.

### Q5 · L4
Your product needs to record 400 loyalty-point updates a minute for a campus canteen chain. Which statement is the most defensible?
- (a) A public chain is suitable because the volume is low
- (b) Proof of stake makes this volume achievable at low cost
- (c) A single database with an append-only audit log handles this easily and a ledger adds cost without removing a referee ✅
- (d) Only a permissioned ledger can guarantee the points are correct
> 400 a minute is trivial for one database. There is also a trusted referee here — the canteen chain itself — so the one thing a ledger buys you is not needed.

## Go deeper
- [Anders Brownworth: interactive blockchain demo](https://andersbrownworth.com/blockchain/)
- [NIST IR 8202: Blockchain Technology Overview](https://nvlpubs.nist.gov/nistpubs/ir/2018/NIST.IR.8202.pdf)
