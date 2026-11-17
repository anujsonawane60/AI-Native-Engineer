---
id: m19-01-hashing-signatures
module: m19-frontier-electives
slug: hashing-and-signatures
title: "Hashing, Digital Signatures and Key Pairs"
order: 1
duration_min: 15
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

The canteen keeps a loyalty register in a hardbound notebook: one point per ₹10 spent, fifty points for a free thali. By March it holds 900 students and a problem. Somebody has been adding points — turning a 4 into a 9 here and there, in the same blue pen, in handwriting close enough to pass.

The manager cannot prove anything. He never recorded what the page looked like before, and the register has no signatures. A thousand rupees a week is leaking out and there is no evidence.

This chapter fixes those two gaps: a way to detect that something changed, and a way to prove who approved it. Nothing in blockchain, UPI, HTTPS or an Android app update works without them. This module aims at literacy, not mastery — you will not implement a hash function, but you will be able to read a design, a vendor pitch or an AI answer and say "that is hashing, that is signing, and that claim is wrong."

### The idea

**A hash is a fingerprint of data. A signature is a fingerprint of data plus proof of who stood behind it.**

A hash function takes any input — three characters or a three-gigabyte video — and produces a short fixed-length output called a digest. Same input, same digest, every time, on any machine. Change one character and the digest changes completely, not slightly. And you cannot work backwards from the digest to the input.

A signature adds identity. You hold two mathematically linked keys: one you never show anyone (the private key) and one you hand out freely (the public key). Signing runs the private key over a hash of the message. Anyone with your public key can check that the signature matches both that exact message and that exact key.

### The analogy

The manager starts stamping each page with a tamper-evident foil seal whose pattern is derived from the ink on the page. Any change to any digit cracks the pattern. That is hashing: cheap, automatic, and it detects alteration. He also initials each page with a signature nobody can reproduce, because it depends on a stencil locked in his drawer, and anyone can compare the initials against a specimen on the noticeboard. That is signing.

**Where the analogy breaks**, in three places. A foil seal tells you a page changed but not what it should have said, so you must have stored the original digest somewhere the attacker cannot reach or it proves nothing. A human signature can be copied well enough to fool a human; a digital signature either verifies or does not. And the stencil can be re-cut if the manager loses it, while a private key cannot. There is no forgot-password link for a key pair, which is the single fact that makes the rest of this block behave so strangely.

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

You have met all of this already without the vocabulary: M02's TLS certificate is a public key signed by a certificate authority, M04's password column stored hashes and its session tokens were signed, M06's Git named every commit by the hash of its contents, and the Play Store in M08 and M12 refuses any update not signed with the original key. UPI runs on it too — every instruction moving between your app, the payment service provider and the bank is signed, which is how a bank accepts an instruction from a third-party app without trusting that app's database.

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

**Hashing is not encryption.** Encryption is reversible by design; hashing is one-way on purpose. "Hash the credit card number so we can decrypt it later" is a sentence that cannot be true, and you will hear it.

**A general-purpose hash is the wrong tool for passwords.** SHA-256 is fast, and fast is what you do not want when an attacker is guessing billions of candidates. Passwords need a deliberately slow function such as bcrypt or Argon2, with a salt — M04 covered this.

**Old hashes are broken and still in use.** MD5 and SHA-1 both have practical collisions, and they survive in legacy checksums and in a depressing amount of IoT firmware.

**A signature proves key possession, not human identity.** It says "whoever holds this private key approved this message". Whether that is the person you think is a custody problem, not a mathematics problem.

**Losing a private key is final.** A forgotten database password is a support ticket. A lost seed phrase is a permanent loss, because the design assumes nobody — including the people who built the system — can recover your half.

### Verify it

AI answers about hashing are fluent and wrong in specific, repeatable ways. Four checks.

**Red flag: hashing and encryption used interchangeably.** Watch for "decrypt the hash to compare". If the explanation implies reversal, it does not understand the primitive.

**Red flag: SHA-256 or MD5 recommended for passwords.** Correct answers name a slow key-derivation function and a per-user salt. No salt mentioned means the answer is incomplete whichever function it picked.

**Red flag: a private key living anywhere a user can reach it.** A signing key in frontend code, an app bundle, a committed config file or a prompt ends the design before it starts. Ask where the key is generated, where it is stored, and who can read it.

**Red flag: "hashing makes the data tamper-proof".** It makes it tamper-*evident*, and only if the digest is stored where the attacker cannot also edit it. Hashing a register and writing the digest on its last page protects nothing. Ask directly: where does the digest live, and who can change it?

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
