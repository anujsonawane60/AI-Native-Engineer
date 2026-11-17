---
id: m19-05-bits-vs-qubits
module: m19-frontier-electives
slug: bits-vs-qubits
title: "Bits vs Qubits: Superposition and Measurement"
order: 5
duration_min: 15
is_free: false
mission: false
volatility: low
last_reviewed: 2026-11-16
---

## Key learnings
- Describe a qubit without using the phrase "tries every answer at once"
- Explain why measurement destroys the thing that made the qubit useful
- State why quantum programs are run thousands of times instead of once
- Name decoherence as the reason today's machines are small and short-lived
- Spot a quantum claim that ignores measurement, shots or noise

## Lesson

### The pain

A student reads a headline — "quantum computer solves in minutes what would take a classical computer ten thousand years" — and repeats it in an interview as "quantum computers try every possible answer at the same time, so they will break all encryption soon."

Three sentences in, the interviewer asks what you get when you read a qubit. The honest answer: one bit, zero or one, chosen at random with a probability set by the state you prepared, and the state is destroyed in the process. The student does not know that, and the interview moves on.

These four chapters exist so you can hold a conversation about quantum computing without being either the person repeating headlines or the person dismissing the field. Literacy, not mastery. You will not design an algorithm; you will be able to read a claim and say which part is physics, which part is engineering, and which part is a press release.

### The idea

**A classical bit is either 0 or 1. A qubit carries a weighted blend of both until it is measured, and measuring it gives you exactly one bit and erases the blend.**

Two words to be careful with. The blend is a superposition, and each possibility carries a weight. Those weights are not simply probabilities — they can work against each other and cancel out, which no probability can do, and that cancellation is the only reason quantum computing is interesting. Chapter 6 takes it apart.

Measurement is the second word. You never see the weights. You get one outcome, with probabilities derived from the weights, and the qubit is left in whatever you measured. So a quantum program is prepared and measured a thousand or more times, and what you read is the histogram of outcomes. Each run is a shot.

### The analogy

The canteen flips a coin to decide whether today's special is poha or upma. While the coin is spinning you cannot sensibly call it heads or tails. You slam your hand on it and now it is one of them. That is roughly superposition and measurement, and it will get you through a dinner conversation.

**Where the analogy breaks**, and it breaks hard. A spinning coin is a coin whose value you do not yet know — the answer exists and you are ignorant of it. A qubit in superposition is not an unknown value; the blend is physically real, and two blends can be combined so one possibility's weight cancels another's and that outcome becomes impossible. Second, a qubit cannot be copied, which is a theorem rather than an engineering limitation. Third, the coin keeps spinning until it lands, while a real qubit loses its blend to heat and stray fields in microseconds whether you are ready or not.

### Picture it

```
 classical bit            qubit before measuring
┌──────────┐             ┌────────────────────────┐
│    0     │             │ weight on 0  ████████  │
│  or  1   │             │ weight on 1  ████████  │
└──────────┘             └────────────────────────┘
 one of two               a blend, with weights you
 definite values          can set but never read

        measure
           │
           ▼
┌────────────────────────────────────────────┐
│ you get ONE bit: 0 or 1, at random, with   │
│ probability set by the weights. The blend  │
│ is gone. Measuring again returns the same. │
└────────────────────────────────────────────┘
```

A qubit holds adjustable weights on both outcomes, and measurement converts those weights into a single random bit and discards everything else.

```
  one program, 1024 shots
  prepare ─▶ gates ─▶ measure ─▶ record bit ──┐
      ▲                                       │
      └───────── repeat 1024 times ◀──────────┘

  result:  0 ▏▏▏▏▏▏▏▏▏▏▏▏▏  512
           1 ▏▏▏▏▏▏▏▏▏▏▏▏▏  512
```

Because a single run yields one random bit, the actual output of a quantum program is a histogram built from many repeated runs, and the answer lives in the shape of that histogram.

### Where it lives

A quantum computer replaces none of M00's four boxes. It is a specialised coprocessor, reached over an ordinary HTTPS API of the kind you met in M02, sitting in a lab fridge colder than deep space, which a classical computer feeds a circuit to and reads a histogram back from.

Your path to touching one is free and immediate: the large vendors expose in-browser circuit builders and simulators at no cost, with limited free queue time on real hardware. A laptop simulator handles twenty to thirty qubits, which covers these four chapters. Nothing here requires money or hardware. What a qubit physically *is* varies — a superconducting loop, a trapped ion, a photon, a neutral atom — but the programming model is the same across all of them.

### Words to own

| Term | Plain meaning | Example |
|---|---|---|
| Bit | A value that is 0 or 1 | Everything in M00 |
| Qubit | A two-outcome system holding a weighted blend | A superconducting loop |
| Superposition | The blend of both outcomes before measuring | A qubit after one gate |
| Weight | How strongly each outcome is represented | What a gate adjusts |
| Measurement | Reading the qubit; yields one bit, destroys the blend | The last step of any circuit |
| Shot | One complete run of prepare, operate, measure | 1024 shots is a common default |
| Gate | An operation that adjusts the weights | The building block of a circuit |
| Circuit | An ordered list of gates on named qubits | A quantum program |
| Decoherence | The blend leaking away into the environment | Why circuits must be short |
| No-cloning | You cannot copy an unknown qubit state | A theorem, not a limitation |
| Simulator | Classical software that imitates qubits exactly | Free, and fine up to ~30 qubits |

### Tradeoffs and gotchas

**"Tries all answers at once" is wrong, and expensively so.** You can prepare a blend over many possibilities but you cannot read them; you read one random outcome. A useful algorithm must make wrong answers cancel before measuring, which is why only a handful of such algorithms exist.

**You cannot inspect a quantum state.** Not on hardware. A simulator shows you the internal weights because it is just classical arithmetic, and that misleads beginners badly.

**You cannot copy a qubit.** So the habits you have from M06 — retry, log, duplicate, snapshot — are unavailable, and error correction has to work entirely differently.

**Qubits die on a schedule.** Coherence times are tens to hundreds of microseconds, and every gate adds error. A circuit is a race, and depth matters more than qubit count for whether a result means anything.

**Randomness is not the product.** A coin-flip circuit produces random bits, and people present that as an achievement. Your laptop has had a good random number source for years.

### Verify it

**Red flag: "parallel universes" or "tries every possibility simultaneously".** The clearest sign the explanation came from popular press rather than physics. Ask directly: what do I get when I measure, and how many times must I run it?

**Red flag: no mention of shots.** A description that does not say the program is repeated many times and read as a distribution has skipped what governs how long an experiment takes and how precise its answer is.

**Red flag: claiming you can read the full state of a qubit.** Ask whether the answer describes a simulator or hardware. The window exists only in simulation.

**Red flag: decoherence absent.** The diagnostic test for a quantum claim. A model that says a thousand-gate circuit will run on a current machine without mentioning error rates or coherence time is describing a machine nobody has built. Ask for circuit depth and the per-gate error rate.

**Red flag: exponential speedup asserted for an ordinary problem.** Sorting, searching an indexed database, training a neural network, routing vans. Ask which named algorithm applies; chapter 7 has the short list.

## Walkthrough
1. Open a free in-browser quantum circuit composer. Start a new circuit with one qubit and one measurement, and nothing else.
2. Run it on the simulator with 1024 shots. Record the histogram. Explain in one line why you got the result you got.
3. Add a single gate that puts the qubit into an even blend of 0 and 1 — the composer will label it H, for Hadamard — before the measurement. Run it again with 1024 shots.
4. Record the histogram. Note that it is close to half and half but not exactly, and write one sentence on why exact halves would be suspicious.
5. Change the shot count to 8 and run. Then to 4096 and run. Write down how the histogram's reliability changed, and what that tells you about the cost of precision.
6. Add a second measurement of the same qubit immediately after the first. Run it, and record whether the two measurements ever disagree.
7. Write one sentence explaining the result of step 6 in terms of the blend being destroyed by the first measurement.
8. Add a gate that flips the qubit before the blend gate, run again, and compare the histogram with step 4. Note whether the output distribution changed at all.
9. Switch the backend from the simulator to a real device if free queue time is available. Run the same circuit, record the histogram, and write down the difference from the simulator's result.
10. Finish with three lines: what you get from one shot, what you get from a thousand, and what the real device's extra imbalance is called.

## Talk to AI
```prompt
CONTEXT:      I am a student learning engineering judgement, not physics and
              not code. I want to understand qubits well enough to read a
              news claim and tell which part of it is true.

GOAL:         A correct mental model of a qubit, superposition and
              measurement, with no mathematics of any kind.

REQUIREMENTS: 1. Explain a qubit without ever using the phrase "tries all
                 answers at once", and explicitly say why that phrase is
                 wrong
              2. State exactly what I receive when I measure one qubit, and
                 what happens to the qubit afterwards
              3. Explain why a quantum program is run many times and read as
                 a histogram, with a worked example at 8 shots and at 4096
              4. A table of five differences between a bit and a qubit
              5. One paragraph on decoherence and why it limits how long a
                 circuit can be
              6. Three sentences about qubits that sound right but are wrong,
                 each with a one-line correction

CONSTRAINTS:  No formulas, no mathematical notation, no Greek letters, no
              matrices, no complex numbers. Plain English for a second-year
              student reading in a second language. Under 800 words.

NON-GOALS:    Do not explain any algorithm yet. Do not discuss whether
              quantum breaks encryption. Do not recommend a vendor.

ACCEPTANCE:   - Your explanation never says a qubit holds many values at
                once in a way I could read
              - You state the number of bits I get from one measurement
              - Your histogram example shows why few shots give an
                unreliable answer
              - Your three wrong sentences are ones I could plausibly hear
                from a classmate or a headline

ASK:          Give me the table first. Then ask me two questions to test
              whether I have understood measurement, and wait for my
              answers before telling me whether I am right.
```

## Stuck? Ask AI
```prompt
Explain to me, as if I have never seen a computer, what is different between a coin you have not looked at yet and a qubit in superposition. Use no mathematics. Keep it under 200 words. End with the one sentence I should remember about why the coin comparison eventually fails.
```

```prompt
I do not understand why a quantum program has to be run a thousand times. Walk me through one circuit, one shot at a time, for the first five shots, telling me what I learn after each one. Then ask me how many shots I would need to be confident about a 60-40 split, and discuss my answer rather than just correcting it.
```

```prompt
Give me five statements about quantum computing taken from the style of news headlines. Ask me to mark each one as true, misleading, or false, and to say which word makes it misleading. Do not reveal your own assessment until I have done all five.
```

## Quiz

### Q1 · L2
What do you get when you measure a single qubit?
- (a) Both of its weights, as two numbers
- (b) One bit, 0 or 1, chosen at random with probabilities set by the weights ✅
- (c) A probability between 0 and 1
- (d) The full state, which you can then copy
> Measurement collapses the blend into one classical bit and discards the rest. The weights are never directly readable on hardware, which is why programs are repeated.

### Q2 · L2
Why is a quantum program typically run with 1024 shots?
- (a) To correct hardware errors by majority vote
- (b) Because each run gives one random outcome and the answer is in the distribution ✅
- (c) Because the first runs warm up the processor
- (d) Because each shot uses a different qubit
> The output of a circuit is a histogram. One shot tells you almost nothing; many shots reveal the shape the circuit was designed to produce.

### Q3 · L3
A circuit measures a qubit, then immediately measures the same qubit again. What do you observe?
- (a) The two results are independent coin flips
- (b) The second measurement always gives the opposite result
- (c) The second measurement agrees with the first every time ✅
- (d) The second measurement fails with an error
> The first measurement destroyed the blend and left the qubit in the state that was read. There is nothing left to be random about.

### Q4 · L4
An AI describes a 2,000-gate circuit running on today's hardware and never mentions error rates or coherence time. What is the problem?
- (a) 2,000 gates is too few to be useful
- (b) It is describing an idealised machine; real qubits lose their state in microseconds and every gate adds error ✅
- (c) Gates cannot be counted before running a circuit
- (d) Coherence time only matters for trapped-ion machines
> Circuit depth against decoherence is the binding constraint on current devices. An answer that omits it is describing a machine nobody has built.

### Q5 · L4
Which claim should make you most suspicious?
- (a) A simulator on a laptop can handle around 30 qubits
- (b) Measuring a qubit gives one bit
- (c) A quantum computer evaluates all possible answers at once and returns the best one ✅
- (d) Qubits cannot be copied
> You cannot read a blend. A useful algorithm must make wrong answers cancel before measurement, which only works for problems with specific structure — a very short list.

## Go deeper
- [Quantum Country: Quantum computing for the very curious](https://quantum.country/qcvc)
- [Quirk: a free in-browser quantum circuit simulator](https://algassert.com/quirk)
