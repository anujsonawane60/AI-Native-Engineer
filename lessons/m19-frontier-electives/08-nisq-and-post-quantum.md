---
id: m19-08-nisq-post-quantum
module: m19-frontier-electives
slug: nisq-and-post-quantum
title: "NISQ, Error Correction and Post-Quantum Cryptography"
order: 8
duration_min: 25
is_free: false
mission: true
volatility: medium
last_reviewed: 2026-11-16
---

## Key learnings
- Explain harvest-now-decrypt-later and why it makes a future machine today's problem
- Separate physical qubits from logical qubits and say roughly what the ratio is
- Name the standardised post-quantum algorithms and what each one replaces
- Decide, with reasons, what your own product should do about this now
- Spot an AI that claims quantum breaks AES or that migration is urgent for everything

## Lesson

### The pain

A health startup launches in 2026. Patient records move between app and server over TLS, with the session key agreed using an elliptic-curve exchange — what every browser does. Nothing is wrong with the implementation.

Someone with a network tap records the encrypted traffic. Not to break it today — they cannot — but to store it. In 2040, if a cryptographically relevant quantum computer exists, that recording can be opened, and a 2026 patient record is still a 2026 patient record. The attacker's clock runs forwards; the data's sensitivity does not expire.

That is harvest-now-decrypt-later, the one reason a technology that does not yet work is already on a real engineering agenda. It is not a reason to believe quantum computers are nearly here, nor to change your product this sprint. It is a reason to know which secrets have a long shelf life.

### The idea

**Today's quantum machines are noisy and small, useful cryptanalysis is many years away, and post-quantum cryptography still matters now — because recorded traffic and long-lived signatures outlive the machine's arrival.**

Three facts in tension, and holding all three at once is the literacy this chapter is for.

Fact one: we are in the NISQ era — noisy, intermediate-scale quantum. Machines have hundreds to low thousands of physical qubits, error rates around one in a hundred to one in a thousand per two-qubit gate, and circuits that must finish before coherence runs out.

Fact two: a *logical* qubit is a reliable qubit assembled from many physical ones by error correction, which spreads one qubit's information across many and measures the disagreements rather than the data. Estimates run from hundreds to a few thousand physical qubits per logical one. Breaking a 2048-bit RSA key is commonly put at a few thousand logical qubits — millions of physical qubits, three to four orders of magnitude beyond today's largest machines.

Fact three: migration takes a decade, across protocols, libraries, certificates, hardware security modules and every vendor in your chain. The deadline is the machine's arrival minus your migration time minus how long your data must stay secret — Mosca's inequality, which needs no formula to use.

### The analogy

The canteen's cold store has a good lock a thief cannot pick. But he comes by weekly with a camera and photographs the lock, the keyway, the hinges, the brand plate. In nine years a machine exists that cuts a key from those photographs. The milk is long gone by then — the safe holding fifteen years of staff records is a different matter.

**Where the analogy breaks**, usefully: the canteen can change its lock in an afternoon. A protocol change rolls across every client, server, certificate and device over years, with a long tail that is never updated at all. Chapter 12's fridge sensors are that tail.

### Picture it

```
  physical qubits ──▶ error correction ──▶ logical qubits
┌────────────────┐                      ┌────────────────┐
│ ~1000 noisy    │   hundreds to a few  │  1 reliable    │
│ qubits today   │   thousand physical  │  logical qubit │
│ error ~1/100   │   per logical one    │  error tiny    │
└────────────────┘                      └────────────────┘

  breaking RSA-2048: a few thousand LOGICAL qubits
                   = millions of physical qubits
                   = three to four orders of magnitude away
```

Error correction trades enormous numbers of unreliable qubits for a small number of reliable ones, and the ratio is why announced qubit counts and cryptographic risk are so far apart.

```
  the three durations that decide your deadline

  how long my data must stay secret   ████████████████ 15 yr
  how long my migration takes         ██████ 5 yr
  when a capable machine may exist              ███████ ?

  if (secrecy + migration) reaches past the machine's
  arrival, you are already late. Compare, then decide.
```

Your deadline is set by adding your data's secrecy lifetime to your migration time and comparing that against the earliest credible arrival, which is why long-lived secrets create urgency that short-lived ones do not.

### Where it lives

M02's TLS is where this lands first, and it has largely landed already: major browsers and providers ship hybrid key exchange, which runs a classical and a post-quantum exchange and mixes the results, so you are protected if either survives. For most products the entire action is "keep your TLS libraries current". Token signing (M04) is less urgent, because a signature need only resist attack while it is still trusted. Crypto agility — changing algorithm without a rewrite — is the M07 and M13 practice this creates.

The standards are settled enough to name, and this is the part that moves, so verify it against current publications. NIST published its first post-quantum standards in 2024: ML-KEM, from the Kyber proposal, for key exchange; ML-DSA from Dilithium and SLH-DSA from SPHINCS+ for signatures. Government guidance points at 2030 to 2035 for retiring classical public-key cryptography; check the current position rather than trusting this paragraph.

### Words to own

| Term | Plain meaning | Example |
|---|---|---|
| NISQ | Noisy, intermediate-scale quantum; today's machines | Hundreds of noisy qubits |
| Physical qubit | One actual device on the chip | A superconducting loop |
| Logical qubit | A reliable qubit made from many physical ones | Hundreds to thousands to one |
| Error correction | Spreading information out and measuring disagreements | Surface code |
| Threshold | The error rate below which correction starts helping | Roughly reached, not exceeded |
| Coherence time | How long a qubit keeps its state | Tens to hundreds of microseconds |
| Harvest-now-decrypt-later | Record encrypted traffic now, open it later | The reason this chapter exists |
| Post-quantum cryptography | Classical algorithms believed safe against quantum | ML-KEM, ML-DSA |
| Hybrid mode | Doing classical and post-quantum together | What browsers already ship |
| Crypto agility | Being able to change algorithm without a rewrite | An operational property |
| Cryptographically relevant | A machine big enough to break real keys | Does not exist today |

### Tradeoffs and gotchas

**Quantum does not break AES, and it does not break hashing.** Grover gives only a square-root improvement on brute-forcing a key, so AES-256 stays comfortably out of reach. Symmetric encryption and modern hash functions are fine; the problem is specifically public-key — RSA, Diffie-Hellman and elliptic curves.

**Short-lived secrets have no urgency.** An hour-long session token, a one-time password, a signature nobody will verify in 2040 — none are harvest-now targets. Sorting secrets by shelf life is the actual work.

**Post-quantum keys and signatures are bigger.** That is the real cost: larger handshakes, more bytes on a radio link, more flash on a microcontroller. On chapter 11's LoRa links, where payloads are tens of bytes, it is a genuine design problem.

**Devices outlive companies.** A sensor shipped in 2026 with no update path still runs 2026 cryptography in 2041.

### Verify it

**Red flag: "quantum computers will break AES".** They will not. An answer that says this does not know which family is at risk, and nothing else it says can be trusted.

**Red flag: "migrate everything immediately".** Ask which secrets must stay secret for more than ten years. For most student products the answer is none, and the action is keeping libraries current.

**Red flag: harvest-now-decrypt-later missing.** An answer arguing nothing needs doing because machines are far away has missed the one argument that makes this present-tense.

**Red flag: NISQ demonstrations presented as cryptographically relevant.** A record qubit count is not a step towards breaking keys. Ask for the logical qubit count implied and the physical-to-logical ratio assumed.

**Red flag: a bespoke "quantum-safe" scheme.** Inventing or combining your own algorithm is worse than doing nothing; these algorithms are young and their implementations carry new bugs. The only acceptable answers are standardised algorithms in maintained libraries, in hybrid mode.

## Walkthrough
1. Open your own product in a browser and inspect the connection's security details. Write down the key exchange and the certificate's signature algorithm.
2. Check whether the key exchange your browser negotiated is a hybrid post-quantum one. Record the exact name shown.
3. List every place your product holds a secret: passwords, session tokens, API keys, payment credentials, stored user data, backups, signed artefacts.
4. Beside each one, write how long it must remain secret or trusted. An hour, a year, ten years, a lifetime. Be specific and be honest.
5. Circle every row with a lifetime over ten years. That circle is your entire post-quantum exposure, and for many products it will be empty or nearly so.
6. Look up the current NIST post-quantum standards and write down which algorithm replaces key exchange and which replaces signatures.
7. Find the published size of a post-quantum key or signature and compare it with the classical equivalent. Note the ratio, and note what that would mean for a radio link carrying fifty bytes.
8. Find a current published estimate of the physical qubits needed to break RSA-2048, and the qubit count of the largest machine announced to date. Write the two numbers together.
9. Ask an AI whether quantum computers break AES-256 and record its answer verbatim. Mark whether it passed or failed the first red flag in this chapter.
10. Write three lines: your longest-lived secret, what you will actually do about it this year, and the one thing you would need to be told for that answer to change.

## Talk to AI
```prompt
CONTEXT:      I am a student founder with a web and mobile product. I do not
              write code. My product stores: [list what you store]. My
              longest-lived sensitive data is [name it and say for how long
              it must stay secret]. I use managed hosting and standard TLS.

GOAL:         Decide what, if anything, my product should do about
              post-quantum cryptography this year, with reasons I could
              defend to a sceptical reviewer.

REQUIREMENTS: 1. Explain harvest-now-decrypt-later in three sentences and
                 say which of my data it applies to
              2. A table of my secrets with columns for how long each must
                 stay secret or trusted, whether it is protected by
                 symmetric or public-key cryptography, and whether it is a
                 harvest-now target
              3. State plainly which parts of my stack are already covered
                 by defaults I do not control, such as hybrid key exchange
                 in browsers and managed TLS
              4. The current NIST post-quantum standards by name and what
                 each replaces, with a warning that I must verify this
                 against current publications
              5. A clear verdict: what I should do now, what I should do
                 within three years, and what I should deliberately not do
              6. What crypto agility would mean concretely for my product

CONSTRAINTS:  No code. No formulas. Free tiers and managed services only.
              Do not assume I can change TLS libraries directly if my host
              manages them. Treat standards and timelines as things that
              move and tell me to verify them.

NON-GOALS:    Do not propose a custom or combined scheme of your own. Do not
              tell me to migrate everything. Do not discuss quantum
              computing applications other than cryptography.

ACCEPTANCE:   - You state explicitly whether quantum computers threaten
                AES-256 and modern hash functions
              - Your table marks each secret as a harvest-now target or not
              - Your verdict includes at least one thing I should NOT do
              - You name hybrid key exchange and say who is responsible for
                enabling it in my stack

ASK:          Ask me for my secrets list and their lifetimes before giving
              any verdict. Then give the table, then the verdict in three
              unhedged sentences.
```

## Mission

**Objective.** Produce a cryptographic inventory of your own product with a secrecy lifetime against every secret, and a written, defensible decision about what you will do about post-quantum cryptography — which for most products will be "very little, deliberately".

**Steps.**
1. **Frame** — in one sentence, name the single piece of data in your product that must stay secret for the longest, and say for how long.
2. **Inventory** — list every secret and every signature: passwords, tokens, API keys, payment credentials, stored personal data, backups, signed app builds, device credentials if you have any.
3. **Lifetime** — give each row a required secrecy or trust lifetime in years, and mark whether it is protected by symmetric or public-key cryptography.
4. **Classify** — mark each row as a harvest-now-decrypt-later target or not. Explain each "yes" in one line.
5. **Check the defaults** — inspect your live site's negotiated key exchange and certificate signature algorithm. Record what you found and who controls it: you, your host, or the browser.
6. **Brief** — use the Talk to AI prompt with your real inventory. Record its verdict.
7. **Verify** — test the AI with the chapter's red flags. Ask it whether quantum breaks AES-256, and whether you should migrate everything now. Record its answers and mark pass or fail on each.
8. **Document** — write the decision-log entry: what you will do this year, what you will revisit and when, what you deliberately will not do, and the one trigger that would change the plan.

**Upload.**
- Your cryptographic inventory table with lifetimes and harvest-now markings → attached file or post caption
- A screenshot of your live product's connection security details, with the key exchange visible
- The AI conversation including its verdict → share link in `ai_chat_url`
- Your red-flag test results, with the AI's exact wording on the AES question
- Your decision-log entry, 150 words maximum
- Reflection, 150 words max → whether your exposure turned out larger or smaller than you expected, and one thing the AI got wrong

**Self-check.** Tick every box before you submit.
- [ ] Every row in my inventory has a lifetime in years, not an adjective
- [ ] I marked which rows are protected by public-key cryptography rather than symmetric
- [ ] I can name which of my secrets, if any, are harvest-now targets
- [ ] I inspected my real connection rather than assuming what it negotiates
- [ ] I recorded who controls the TLS configuration: me, my host, or the browser
- [ ] I tested the AI on whether quantum breaks AES-256 and wrote down its exact answer
- [ ] My plan includes at least one thing I have decided NOT to do, with a reason
- [ ] My decision log names a revisit date and a trigger
- [ ] I did not propose inventing or combining any cryptography myself
- [ ] My reflection names something specific the AI got wrong

**Stretch.** Take one constrained device scenario from chapter 11 — a sensor on a radio link with a fifty-byte payload budget — and write 150 words on what a post-quantum signature size would do to that design, and what you would do instead.

## Stuck? Ask AI
```prompt
Explain to me, as if I have never seen a computer, why a machine that does not exist yet can already be a problem for data I send today. Use one everyday example, no mathematics, under 200 words, and end with the one question I should ask about any piece of my own data.
```

```prompt
I do not understand the difference between a physical qubit and a logical qubit. Explain it with an everyday comparison, tell me roughly how many of one it takes to make the other according to current estimates, and then ask me why a headline announcing a record qubit count is not a cryptography story. Wait for my answer.
```

```prompt
Give me six secrets a small web product might hold - a session token, a user password, a stored medical note, a payment credential, a signed app build, a database backup. Ask me to assign each one a required secrecy lifetime and say whether it is a harvest-now-decrypt-later target. Do not tell me your own answers until I have done all six.
```

## Quiz

### Q1 · L3
Why does post-quantum cryptography matter now, given that capable machines do not exist?
- (a) Because current machines can already break short RSA keys
- (b) Because encrypted traffic recorded today can be decrypted later, and migration takes years ✅
- (c) Because AES is already weakened by Grover's algorithm
- (d) Because regulators have banned classical cryptography
> Harvest-now-decrypt-later plus a decade-long migration means the deadline is the machine's arrival minus both. Short-lived secrets, by contrast, carry no urgency.

### Q2 · L2
Which of these is genuinely at risk from a large quantum computer?
- (a) AES-256 encryption of data at rest
- (b) SHA-256 hashing of file contents
- (c) RSA and elliptic-curve key exchange ✅
- (d) Password hashing with Argon2
> The threat is specific to public-key cryptography. Symmetric encryption and modern hashing face only a square-root improvement, which AES-256 absorbs comfortably.

### Q3 · L3
Roughly what is the relationship between physical and logical qubits?
- (a) One physical qubit is one logical qubit with better calibration
- (b) Hundreds to a few thousand physical qubits are needed per logical qubit ✅
- (c) Logical qubits are a software abstraction with no hardware cost
- (d) Ten physical qubits make one logical qubit
> Error correction spreads one qubit's information across many and measures the disagreements. That overhead is why announced qubit counts sit far below cryptographic relevance.

### Q4 · L4
An AI tells you to migrate your entire stack to post-quantum algorithms immediately. What is the best response?
- (a) Do it, since the standards are published
- (b) Ask which of your secrets must stay secret for more than ten years ✅
- (c) Ignore the advice entirely, since machines are far away
- (d) Design a custom hybrid scheme to be safe
> The work is proportional to secrecy lifetime. For most products the answer is to keep libraries current and record crypto agility as a requirement, not to rebuild now.

### Q5 · L5
Which product is in the worst position on this issue?
- (a) A web app with one-hour session tokens on managed TLS
- (b) A payments integration using short-lived signed requests
- (c) A fleet of deployed sensors with no update path, shipped with long-lived keys ✅
- (d) A static marketing site behind a managed certificate
> Devices that cannot be updated keep the cryptography they shipped with, and they frequently outlive the company that made them. That is chapter 12's problem, named here.

## Go deeper
- [NIST: Post-Quantum Cryptography project and standards](https://csrc.nist.gov/projects/post-quantum-cryptography)
- [Cloudflare: the state of the post-quantum Internet](https://blog.cloudflare.com/pq-2024/)
