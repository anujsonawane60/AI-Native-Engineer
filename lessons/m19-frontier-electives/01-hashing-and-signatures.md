---
id: m19-01-hashing-signatures
module: m19-frontier-electives
slug: hashing-and-signatures
title: "Hashing, Digital Signatures and Key Pairs"
order: 1
duration_min: 25
is_free: false
mission: false
volatility: low
last_reviewed: 2026-11-16
---

## Key learnings
- Explain what a hash proves and what it cannot prove
- Separate signing from encryption, which almost everyone confuses
- Describe a key pair in terms of who holds which half
- Name the three red flags in an AI answer about hashing or signing
- Place hashing and signatures on the map you already know from M02 and M04

## Lesson

### The pain

The canteen keeps a loyalty register in a hardbound notebook. One point per ₹10 spent, fifty points for a free thali. By March the register has 900 students in it and a problem: somebody has been adding points. Not forging whole pages — just turning a 4 into a 9 here and there, in the same blue pen, in handwriting close enough to pass.

The manager cannot prove anything. He cannot tell which lines changed, because he never recorded what the page looked like before. He cannot tell who changed them, because the register has no signatures. And he cannot undo it, because there is no earlier copy to compare against. He has 900 students, a thousand rupees a week leaking out, and no evidence.

This chapter is about the two tools that fix exactly those two gaps: a way to detect that something changed, and a way to prove who approved it. They are the floor that the next three chapters stand on. Nothing in blockchain, in UPI, in your HTTPS connection, in an Aadhaar e-sign or in an Android app update works without them.

This module aims at literacy, not mastery. You are not going to implement a hash function. You are going to be able to read a design, a vendor pitch or an AI answer and say "that is hashing, that is signing, and that claim is wrong."

### The idea

**A hash is a fingerprint of data. A signature is a fingerprint of data plus proof of who stood behind it.**

A hash function takes any input — three characters or a three-gigabyte video — and produces a short fixed-length output, called a digest. Same input, same digest, every time, on any machine. Change one character of the input and the digest changes completely, not slightly. And you cannot work backwards from the digest to the input.

A signature adds identity. You hold two mathematically linked keys: one you never show anyone (the private key) and one you hand out freely (the public key). Signing runs the private key over a hash of the message. Anyone with your public key can check that the signature matches both that exact message and that exact key. If either the message or the key is different, verification fails.

### The analogy

Imagine the canteen manager starts stamping each page of the register with a tamper-evident foil seal, and the pattern of the foil is derived from the ink on the page. Any change to any digit cracks the pattern. That is hashing: cheap, automatic, and it detects alteration.

Then imagine he also initials each page with a signature that nobody on campus can reproduce, because it depends on a stencil locked in his drawer. Anyone can compare the initials against a specimen pinned to the noticeboard. That is signing: it proves this page passed through someone who holds the stencil.

**Where the analogy breaks**, in three places worth naming. A foil seal tells you a page changed but not what it should have said, and a hash is the same — you must have stored the original digest somewhere the attacker cannot reach, or it proves nothing. Second, a human signature can be copied well enough to fool a human; a digital signature either verifies or does not, with no middle ground. Third, and most importantly: the stencil can be re-cut if the manager loses it. A private key cannot. There is no forgot-password link for a key pair, which is the single fact that makes the rest of this block behave so strangely.

### Picture it

```
  input                 hash function          digest (fixed length)
┌───────────────┐      ┌─────────────┐        ┌────────────────────┐
│ "Asha: 42 pts"│─────▶│   SHA-256   │───────▶│ 7f3a9c... (64 hex) │
└───────────────┘      └─────────────┘        └────────────────────┘
┌───────────────┐      ┌─────────────┐        ┌────────────────────┐
│ "Asha: 43 pts"│─────▶│   SHA-256   │───────▶│ b1e04d... (64 hex) │
└───────────────┘      └─────────────┘        └────────────────────┘
```

One digit of difference in the input produces a completely unrelated digest of the same length, which is what makes a hash useful for detecting change.

```
  Manager                                   Any student
┌──────────────┐   message + signature    ┌────────────────┐
│ private key  │─────────────────────────▶│  public key    │
│ signs hash   │                          │  verifies      │
└──────────────┘                          └────────────────┘
      kept secret, never sent                 published freely
```

Signing happens with the half nobody else has; verifying happens with the half everybody has, which is why anyone can check a signature but only one person can make one.

### Where it lives

You have met all of this already without the vocabulary.

In M02, your HTTPS connection used the server's certificate — a public key signed by a certificate authority — to prove the server is who the domain says it is. In M04, your password column stored hashes rather than passwords, and your session tokens were signed so a user could not edit their own role. In M06, Git named every commit by the hash of its contents, which is why you cannot quietly rewrite history without every later commit id changing. In M08 and M12, the Play Store refused any app update not signed with the same key as the original.

UPI runs on this too. Every payment instruction moving between your app, the payment service provider and the bank is signed, which is how a bank can accept an instruction from a third-party app without trusting that app's database.

### Words to own

| Term | Plain meaning | Example |
|---|---|---|
| Hash function | Turns any data into a short fixed-length fingerprint | SHA-256 |
| Digest | The output of a hash function | A 64-character hex string |
| Collision | Two different inputs with the same digest | Why MD5 is retired |
| Salt | Random text added before hashing a password | Stops two identical passwords matching |
| Key pair | Two linked keys, one secret and one public | Your wallet, your SSH key |
| Private key | The half that signs; never leaves your control | The stencil in the drawer |
| Public key | The half that verifies; safe to publish | The specimen on the noticeboard |
| Signature | Proof a specific message passed a specific private key | A signed UPI instruction |
| Verification | Checking a signature against message and public key | What your browser does on every TLS handshake |
| Seed phrase | Twelve or twenty-four words that regenerate a private key | A crypto wallet backup |

### Tradeoffs and gotchas

**Hashing is not encryption.** Encryption is reversible by design — that is the point of it. Hashing is one-way on purpose. "Hash the credit card number so we can decrypt it later" is a sentence that cannot be true, and you will hear it.

**A general-purpose hash is the wrong tool for passwords.** SHA-256 is fast, and fast is exactly what you do not want when an attacker is guessing billions of candidates. M04 covered this: passwords need a deliberately slow function such as bcrypt or Argon2, with a salt. An AI that suggests SHA-256 for passwords has given you a 1998 answer.

**Old hashes are broken and still in use.** MD5 and SHA-1 both have practical collisions. They survive in legacy checksums and in a depressing amount of IoT firmware, which you will meet again in chapter 12.

**A signature proves key possession, not human identity.** It says "whoever holds this private key approved this message". Whether that is the person you think depends entirely on key custody, which is a people problem, not a mathematics problem.

**Losing a private key is final.** This is where students get hurt in the next two chapters. A forgotten database password is a support ticket. A lost seed phrase is a permanent loss, because the whole design assumes nobody — including the people who built the system — can recover your half.

### Verify it

Ask an AI about hashing and you will get fluent, confident text that is wrong in specific, repeatable ways. Four checks.

**Red flag: the words hashing and encryption used interchangeably.** Watch for "we encrypt the password" when the answer goes on to describe a one-way store, or "decrypt the hash to compare". If the explanation implies reversal, the answer does not understand the primitive.

**Red flag: SHA-256 or MD5 recommended for passwords.** Correct answers name a slow key-derivation function and mention a per-user salt. If salting is not mentioned at all, the answer is incomplete regardless of which function it picked.

**Red flag: a private key living anywhere a user can reach it.** If a proposed design puts a signing key in frontend code, in an app bundle, in a committed configuration file, or in a prompt, the design is finished before it starts. Ask where the key is generated, where it is stored, and who can read it. M07 gave you the secrets discipline; it applies here without exception.

**Red flag: "hashing makes the data tamper-proof".** It makes it tamper-*evident*, and only if the digest is stored somewhere the attacker cannot also edit. Hashing a register and writing the digest on the last page of the same register protects nothing. Ask the question directly: where does the digest live, and who can change it?

## Walkthrough
1. Open any free in-browser SHA-256 calculator. Type your own name and hash it. Note the length of the output.
2. Type your name again with one letter changed in case. Hash it. Compare the two digests character by character and note how much of the output moved.
3. Paste a whole paragraph of text and hash it. Confirm the digest is exactly as long as the one for your three-letter name. Write down why that matters for storage.
4. Try to find any tool on that page that converts the digest back to the text. There is none. Write one sentence explaining why that is a feature rather than a missing button.
5. Open a free in-browser key pair generator, generate a key pair, and look at both halves. Note which one the tool warns you to protect.
6. Sign a short message — "Asha has 42 points" — with the private key. Save the message and the signature.
7. Verify the signature using only the public key and the message. Confirm it passes.
8. Change one character of the message, keep the same signature, and verify again. Record exactly what the tool says when verification fails.
9. Now verify the original message with a different public key. Record the result and write one sentence about what the failure actually proves.
10. Write a four-line note in your own words: what a hash detects, what it cannot tell you, what a signature adds, and one place in your own project where a signature is already being checked without you having written it.

## Talk to AI
```prompt
CONTEXT:      I am a student learning engineering judgement, not writing code.
              I want to understand hashing and digital signatures well enough
              to review a design and spot wrong claims.

GOAL:         A clear separation of four things people mix up: hashing,
              encryption, signing, and password storage.

REQUIREMENTS: 1. A table with four rows - hashing, encryption, signing,
                 password storage - and columns for "what it gives you",
                 "is it reversible", "what key is involved", "one real
                 system that uses it"
              2. One worked example of a tamper-evident log, including where
                 the digest must be stored for it to prove anything
              3. Three sentences that sound plausible but are technically
                 wrong, each with a one-line correction
              4. A plain explanation of why a lost private key cannot be
                 recovered, with no mathematics

CONSTRAINTS:  No code. No formulas. No mathematical notation. Plain English
              at the level of a second-year student reading in a second
              language. Under 700 words total.

NON-GOALS:    Do not explain how SHA-256 works internally. Do not recommend
              any library. Do not discuss blockchain yet.

ACCEPTANCE:   - Your table says explicitly which rows are reversible
              - You name a slow password-hashing function and mention salt
              - Each of your three wrong sentences is one I might plausibly
                hear from a classmate, not an obvious joke
              - You state where a digest must live for tamper evidence to
                hold

ASK:          Give me the table first. Then ask me two questions to check
              whether I have actually understood the difference between
              signing and encrypting, and wait for my answers before
              telling me if I am right.
```

## Stuck? Ask AI
```prompt
Explain to me, as if I have never seen a computer, why you can turn a sentence into a fingerprint but cannot turn the fingerprint back into the sentence. Use one everyday analogy, keep it under 200 words, and use no mathematics at all. Then give me the one sentence I should memorise.
```

```prompt
I keep confusing signing and encrypting. Give me six short scenarios - a bank instruction, an app update, a password, a private message, a software download, a login token - and ask me which of signing or encrypting each one needs and why. Do not tell me the answers until I have guessed all six, then mark each one and explain only the ones I got wrong.
```

```prompt
Write a short description of a tamper-evident attendance register for a college, where someone has deliberately made one mistake in where the digest is stored. Ask me to find the mistake. If I cannot find it after two guesses, give me a hint rather than the answer.
```

## Quiz

### Q1 · L2
What does a hash of a file actually let you prove?
- (a) That the file has not changed since the digest was recorded ✅
- (b) That the file was created by a specific person
- (c) That the file cannot be read by anyone else
- (d) That the file can be restored if it is lost
> A hash is a fingerprint, nothing more. Identity needs a signature, secrecy needs encryption, and recovery needs a backup. Hashing provides none of those three.

### Q2 · L3
A design hashes every row of an audit table and stores each digest in a column of that same table. What is wrong?
- (a) Nothing; this is the standard pattern
- (b) An attacker who can edit the row can also edit its digest ✅
- (c) SHA-256 is too slow to run per row
- (d) Digests cannot be stored in a database column
> Tamper evidence only holds if the digest lives somewhere the attacker cannot reach. A digest stored beside the data it protects is edited in the same breath as the data.

### Q3 · L3
Which half of a key pair signs, and which half verifies?
- (a) The public key signs; the private key verifies
- (b) Either half can sign and the other verifies
- (c) The private key signs; the public key verifies ✅
- (d) Both halves are needed to sign
> Only the holder of the secret half can produce a signature, and anyone holding the published half can check it. That asymmetry is the entire reason the scheme is useful.

### Q4 · L4
An AI proposes storing user passwords as SHA-256 digests with no salt. What is the strongest objection?
- (a) SHA-256 is reversible, so the passwords are exposed
- (b) SHA-256 is fast, so offline guessing is cheap, and identical passwords look identical ✅
- (c) SHA-256 produces digests that are too short to be unique
- (d) Hashing passwords is unnecessary if the database is encrypted
> Password storage needs a deliberately slow function with a per-user salt. Speed helps the attacker, and an unsalted scheme reveals which users share a password.

### Q5 · L4
Your app verifies a signature on an incoming payment instruction and it passes. What have you actually established?
- (a) That the message was sent by a trustworthy company
- (b) That the message content is correct
- (c) That the message was approved by whoever holds the matching private key ✅
- (d) That the message was transmitted over an encrypted channel
> Verification binds a message to a key, not to a human or an intention. Whether that key is in the right hands is a custody question, and whether the content is sensible is still your job.

## Go deeper
- [Cloudflare Learning: What is asymmetric encryption?](https://www.cloudflare.com/learning/ssl/what-is-asymmetric-encryption/)
- [OWASP Password Storage Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Password_Storage_Cheat_Sheet.html)
