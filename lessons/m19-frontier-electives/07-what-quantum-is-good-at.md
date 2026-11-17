---
id: m19-07-quantum-good-at
module: m19-frontier-electives
slug: what-quantum-is-good-at
title: "What Quantum Is Actually Good At"
order: 7
duration_min: 15
is_free: false
mission: false
volatility: low
last_reviewed: 2026-11-16
---

## Key learnings
- Name the four problem families where a quantum advantage is credible
- Explain why a square-root speedup is usually not worth having
- State honestly that quantum machine learning has no demonstrated practical advantage
- Dissect a vendor advantage claim into algorithm, structure, baseline and hardware
- Spot a routing, scheduling or AI claim that quantum cannot support

## Lesson

### The pain

A team puts "quantum-optimised delivery routing" on slide six. The mentor asks which algorithm, on which hardware, against which classical baseline. There is no answer, because the phrase came from a vendor's landing page.

The embarrassing part is not the slide. Routing a fleet of ten vans is a problem the best classical solvers handle in seconds, nobody has demonstrated a quantum machine beating them on a real instance, and even the theoretical gain for that family is modest. The slide claimed a benefit that does not exist, on hardware that cannot run it, against a baseline nobody measured.

So this chapter is a filter. The real list of things quantum computers are expected to be good at is short, specific and mostly not your product, and knowing it protects you from the hype and the reflexive dismissal alike.

### The idea

**Quantum computing helps on problems with a particular hidden structure that interference can exploit. There are four credible families, and your startup is almost certainly in none of them.**

First, breaking certain public-key cryptography — factoring large numbers and discrete logarithms — where Shor's algorithm is an exponential improvement, which is why the world cares and why chapter 8 exists. Second, simulating quantum systems themselves: molecules, catalysts, battery chemistry, materials. Most experts consider this the genuine commercial prize, because a quantum system is the natural thing to simulate on a quantum machine. Third, unstructured search, where Grover's algorithm offers a square-root improvement that is almost always disappointing in practice. Fourth, sampling and linear-algebra niches of mostly academic interest, where "advantage" has been claimed on tasks chosen because they suit the hardware.

Everything else — routing, scheduling, portfolio optimisation, general machine learning, databases, web applications — is either not on the list or in a category where nobody has shown a real-world win.

### The analogy

The canteen buys a dedicated dosa griddle. On dosas it is unbeatable: faster, more even, better than anything else in the kitchen. On dal, rice, chai or washing up it is worse than useless, because it occupies the counter and demands a dedicated gas line. Nobody calls it a general-purpose cooker, and nobody prices the whole menu on its dosa throughput.

**Where the analogy breaks**, and the break is the honest news: the griddle works today. The quantum griddle is a prototype in a cold room that burns most of the dosas and can only manage very small ones. The specialisation is real; the capability is not here yet for anything commercially decisive, with the partial exception of narrow chemistry research.

### Picture it

```
  problem family          speedup            usable when?
┌──────────────────────┬────────────────────┬──────────────┐
│ factoring, discrete  │ exponential        │ not for      │
│ logs (Shor)          │                    │ years - but  │
│                      │                    │ act now, ch8 │
├──────────────────────┼────────────────────┼──────────────┤
│ simulating molecules │ exponential, for   │ early, narrow│
│ and materials        │ the right systems  │ research use │
├──────────────────────┼────────────────────┼──────────────┤
│ unstructured search  │ square root        │ overheads    │
│ (Grover)             │ (Grover)           │ usually eat  │
│                      │                    │ the gain     │
├──────────────────────┼────────────────────┼──────────────┤
│ sampling / linear    │ task-specific,     │ research     │
│ algebra niches       │ contested          │ interest     │
├──────────────────────┼────────────────────┼──────────────┤
│ routing, scheduling, │ none demonstrated  │ use a good   │
│ general ML, your app │                    │ classical    │
│                      │                    │ solver       │
└──────────────────────┴────────────────────┴──────────────┘
```

Four families carry credible claims and the last row carries none, which is the row almost every student pitch accidentally lands in.

```
  why a square-root gain disappoints

  classical brute force : 1,000,000 tries
  Grover, ideal         :     1,000 tries
  but each "try" needs a circuit deep enough that today's
  hardware loses coherence, and error correction multiplies
  every qubit by hundreds - so the 1,000 costs more than
  the 1,000,000 did on a laptop
```

A square-root improvement in step count is real, but each quantum step is vastly more expensive to perform reliably, so the crossover point sits far beyond today's machines.

### Where it lives

M04's public-key cryptography holding up TLS, signed tokens and payment instructions is family one, and the only family with a deadline attached. Against M09: quantum machine learning is an active research area with no demonstrated practical advantage over good classical methods on real datasets, and several proposed quantum advantages in linear algebra were later matched by improved classical algorithms, a pattern called dequantization. On M17's cost question: access is metered by the second on cloud hardware and there is no business case for a student product. Your free access is in-browser simulators, which run Grover on three or four qubits at no cost — enough to see the mechanism and feel the overhead.

Annealing deserves its own sentence because it is sold hardest. An annealer is a different kind of machine aimed at optimisation problems in one particular form, and independent benchmarks have repeatedly found well-tuned classical solvers competitive or better on realistic instances. Treat any routing or scheduling advantage claim as unproven until you see the classical baseline named.

### Words to own

| Term | Plain meaning | Example |
|---|---|---|
| Shor's algorithm | Factoring and discrete logs, exponentially faster | Why cryptography must change |
| Grover's algorithm | Unstructured search, square-root fewer tries | A million becomes a thousand |
| Quantum simulation | Modelling quantum systems on quantum hardware | Catalysts, batteries |
| Structure | The hidden pattern an algorithm exploits | Periodicity, in Shor's case |
| Exponential speedup | The gap widens enormously with problem size | Shor |
| Square-root speedup | The gap widens slowly with problem size | Grover |
| Annealing | Hardware aimed at one form of optimisation | Sold for routing; unproven |
| Variational algorithm | Classical optimiser driving a short quantum circuit | Most NISQ-era experiments |
| Quantum advantage | A real task done better than any classical method | Claimed; always contested |
| Dequantization | A classical algorithm catching up to a quantum one | Several linear-algebra claims |
| Logical qubit | A reliable qubit built from many physical ones | Chapter 8 |

### Tradeoffs and gotchas

**Grover's gain is usually theoretical.** It needs the search expressible as a circuit the machine can run, and it does not parallelise the way classical search does across cheap machines. For most practical search you already have an index, which beats both.

**Advantage demonstrations are chosen to suit the hardware.** The famous ones involve sampling tasks with no application, selected because they are hard classically and natural for the device. Genuine scientific results; not products.

**Classical methods keep improving.** Any comparison against a weak classical baseline is worthless. Insist on knowing what the classical side was allowed to use.

**Chemistry is the credible near-term case, and it is still narrow.** Useful simulation of industrially interesting molecules needs error-corrected machines that do not exist yet.

**"Quantum AI" and "quantum-inspired" are different from quantum.** Quantum-inspired usually means a classical algorithm borrowing an idea — fine, but then it runs on a normal computer and the word quantum is marketing.

### Verify it

Four questions dissect any advantage claim, and asking them is the whole skill.

**Which named algorithm?** Shor, Grover, a variational method, annealing, or something specific. "Quantum computing will optimise it" names nothing, and without an algorithm there is no claim to evaluate.

**What structure does it exploit?** Shor's exploits periodicity; simulation exploits the fact that the target is itself quantum. If a model claims a speedup for routing or for training, ask what hidden structure interference is cancelling against. There is usually no answer.

**What was the classical baseline?** Named solver, tuned, on the same instance, with its runtime reported. A claim with no baseline is a claim about nothing, and this catches most vendor material.

**What hardware, how many qubits, what error rate, what circuit depth?** Chapter 5's decoherence test applied to a specific claim. A model proposing a useful Shor run without mentioning that estimates call for millions of physical qubits is giving you a fantasy.

And one specific to this chapter: **red flag — "quantum machine learning will make our model better".** Ask for a published result on a real dataset beating a good classical model. There is none.

## Walkthrough
1. Open a free in-browser quantum composer or simulator. Load or build a small Grover search over three qubits, marking one of the eight possible answers.
2. Run it with 1024 shots. Record how often the marked answer came up compared with the other seven.
3. Count the gates in the circuit, including the repeated search step. Write the number down as your circuit depth.
4. Work out how many tries a classical search would need in the worst case over eight items, and write both numbers side by side.
5. Extend the search to four qubits — sixteen items — and repeat the marked-answer run. Record the new depth and note how the depth grew.
6. Write one line on what the depth growth implies for a search over a million items on hardware that loses coherence after a few hundred gates.
7. Find a public vendor or press claim about quantum computing applied to optimisation, logistics or finance. Copy the exact sentence.
8. Dissect that sentence against the four questions: named algorithm, exploited structure, classical baseline, hardware and error rate. Mark each as answered or absent.
9. Look up a published estimate of the physical qubit count needed to break a 2048-bit RSA key, and write it next to the qubit count of the largest machine you can find announced today.
10. Finish with three lines: one problem family where quantum is credible, one claim you dissected and what was missing from it, and one sentence you could say in an interview about where quantum fits for your own product.

## Talk to AI
```prompt
CONTEXT:      I am a student founder. I keep seeing claims that quantum
              computing will optimise logistics, portfolios or machine
              learning. I do not write code and I am not a physicist. I need
              to judge these claims.

GOAL:         A short, honest map of where quantum computing is credible and
              where it is not, plus a reusable way to dissect any claim.

REQUIREMENTS: 1. A table of problem families with a credible quantum
                 advantage, each with the named algorithm, the structure it
                 exploits, whether the speedup is exponential or square
                 root, and how far from usable it is today
              2. A second table of commonly claimed applications with no
                 demonstrated advantage, and what the best classical
                 approach is instead
              3. A plain explanation of why a square-root speedup is often
                 not worth having once overheads are counted
              4. Your honest current assessment of quantum machine learning,
                 stated plainly rather than hedged
              5. A four-question checklist I can apply to any advantage
                 claim, with an example of applying it to a real claim

CONSTRAINTS:  No formulas, no mathematical notation, no complex numbers, no
              matrices. Plain English for a second-year student reading in a
              second language. Where you give a number, say whether it is an
              estimate and from roughly when. Under 1000 words.

NON-GOALS:    Do not recommend a vendor or a cloud service. Do not explain
              how Shor's algorithm works internally. Do not tell me quantum
              is useless; tell me where it is real.

ACCEPTANCE:   - Every row in your first table names an algorithm
              - Your second table includes routing and general machine
                learning
              - You state explicitly whether any published result shows
                quantum machine learning beating good classical methods on
                a real dataset
              - Your checklist includes asking for the classical baseline

ASK:          Give me the two tables first. Then ask me to describe a
              quantum claim I have actually seen, and dissect it with me
              using your checklist rather than doing it for me.
```

## Stuck? Ask AI
```prompt
Explain to me, as if I have never seen a computer, why a machine can be enormously better at one specific job and useless at everything else. Use one everyday example from a kitchen, keep it under 200 words, no mathematics, and end with the one sentence I should remember about quantum computers.
```

```prompt
I do not understand why reducing a million tries to a thousand tries might not be a good deal. Walk me through what one "try" costs on a real quantum machine today versus on my laptop, in plain words with no formulas, and then ask me at what problem size the deal would start to make sense.
```

```prompt
Give me five claims about quantum computing applications - one about drug discovery, one about delivery routing, one about machine learning, one about breaking encryption, one about materials science. Ask me to rank them from most to least credible and say why. Do not give me your ranking until I have given mine with reasons.
```

## Quiz

### Q1 · L2
Which problem family has an exponential quantum speedup and a known algorithm?
- (a) Sorting a large list
- (b) Factoring large numbers and discrete logarithms ✅
- (c) Training a neural network
- (d) Routing a delivery fleet
> Shor's algorithm is the one clear exponential result with real-world consequences, and it is why cryptography must migrate. The other three have no demonstrated advantage.

### Q2 · L3
Why is Grover's square-root speedup often not worth having?
- (a) Grover's algorithm only works on sorted data
- (b) Each quantum step is far more expensive and the circuit depth exceeds what hardware can hold ✅
- (c) It gives the wrong answer most of the time
- (d) Classical computers can also achieve a square-root speedup
> The step count drops but the cost per step rises enormously once coherence limits and error-correction overhead are counted, pushing the crossover far beyond current machines.

### Q3 · L4
A vendor claims a quantum advantage on portfolio optimisation. Which missing item most undermines the claim?
- (a) The number of qubits used
- (b) The name of the cloud provider
- (c) The classical baseline, named and tuned, with its runtime ✅
- (d) The temperature of the processor
> Without a strong classical comparison there is no claim at all. Weak baselines are the most common way an optimisation advantage is manufactured.

### Q4 · L4
What is the most honest current statement about quantum machine learning?
- (a) It already outperforms classical models on image tasks
- (b) It is an active research area with no demonstrated practical advantage on real datasets ✅
- (c) It is impossible in principle
- (d) It works but only on quantum-generated data
> Several proposed advantages were later matched classically. Research is genuine and ongoing; a product claim built on it today is not supportable.

### Q5 · L5
Your team wants to say something about quantum on a pitch slide. Which sentence is defensible?
- (a) Our routing engine is quantum-optimised
- (b) Quantum computing will make our matching algorithm exponentially faster
- (c) Quantum does not apply to our workload; our exposure is cryptographic, and we have a migration plan ✅
- (d) We use quantum-inspired machine learning for better predictions
> The only quantum issue a normal software product genuinely has is cryptographic, and saying so with a plan reads as judgement. The other three claim advantages nobody has shown.

## Go deeper
- [National Academies: Quantum Computing — Progress and Prospects](https://nap.nationalacademies.org/catalog/25196/quantum-computing-progress-and-prospects)
- [Scott Aaronson: Quantum supremacy FAQ](https://scottaaronson.blog/?p=4317)
