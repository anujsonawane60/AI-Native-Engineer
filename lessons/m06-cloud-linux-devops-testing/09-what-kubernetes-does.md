---
id: m06-09-orchestration
module: m06-cloud-linux-devops-testing
slug: what-kubernetes-does
title: "Orchestration: What Kubernetes Does, and When You Do Not Need It"
order: 9
duration_min: 15
is_free: false
mission: false
volatility: low
last_reviewed: 2026-11-16
---

## Key learnings
- State the one problem orchestration solves, in a single sentence
- Explain desired state and the control loop without using jargon
- Name four signals that you have genuinely outgrown a simple platform
- Say honestly why a three-person student team should not run a cluster
- Recognise an architecture chosen for a CV rather than for a product

## Lesson

### The pain

A team watches a conference talk and decides their canteen app needs Kubernetes. Three weeks later they have a cluster, four YAML files nobody fully understands, an ingress returning 502s, and a monthly bill, because a managed control plane is rarely free. The app has eleven users.

A different team has forty containers across twelve machines, deployed by hand. Nobody can say which machine runs what, and when one dies at 2 a.m. nobody notices. The first team has a solution with no problem; the second has a problem with no solution. The only skill in this chapter is telling which one you are in.

### The idea

**Orchestration is what you need when you have more containers than you can keep in your head, running across more machines than you want to visit by hand.**

You stop issuing instructions and start declaring an outcome: "three copies of this image, on this address, restarted if they die, replaced one at a time when the image changes." Something then works continuously to make reality match it. Kubernetes is the dominant implementation, which is worth understanding — a different question from whether you should run one.

### The analogy

A single counter needs no management system; the cook knows what is cooking. A caterer running twelve counters for a festival cannot hold it in their head, so they write a plan: two servers per counter, a replacement pulled from the pool if someone misses a shift, counters switched to the new menu one at a time so the queue never stops, a runner reporting anyone missing. The plan is the desired state; the runner checking and fixing is the control loop. Writing that plan for one counter would be absurd: the plan costs a planner.

**Where the analogy breaks:** the runner uses judgement; a control loop compares a number to a number. Declare three copies, and if one is wedged but still answering its health check, the loop sees three and is satisfied. Orchestration keeps the *count* right, not the *product* working.

### Picture it

```
  YOU                          THE CONTROL LOOP
  ┌───────────────────┐        every few seconds:
  │ desired state:    │        ┌─────────────────────┐
  │  3 copies         │───────▶│ how many are        │
  │  image v7         │        │ running?            │
  │  port 443         │        └──────────┬──────────┘
  └───────────────────┘                   │
                                    2 running
                                          │
                                          ▼
                              ┌─────────────────────┐
                              │ start 1 more,       │
                              │ somewhere with room │
                              └─────────────────────┘

  node A        node B        node C
  [copy 1]      [copy 2]      [copy 3 ← new]
```

You declare the outcome you want, and a loop repeatedly compares reality to that declaration and takes the smallest action that closes the gap.

### Where it lives

Orchestration sits above chapter 8's containers and below chapter 6's platforms: chapter 2's service manager keeps one process alive on one machine, a container runtime keeps one container alive, orchestration keeps many alive across machines.

| The six jobs it does | What that means |
|---|---|
| Scheduling | Which machine has room for a new container |
| Self-healing | Restart a dead container; replace one on a dead machine |
| Scaling | Add or remove copies |
| Service discovery | A stable name for copies whose addresses change |
| Rolling updates | Replace v7 with v8 one copy at a time, with rollback |
| Config and secrets | Values injected at run time, not baked in |

Your zero-config platform already gives you all six: orchestration is not optional in a serious system, but a managed platform is orchestration run by someone else.

### Words to own

| Term | Plain meaning | Example |
|---|---|---|
| Orchestration | Running many containers across many machines | The whole subject |
| Desired state | What you declare should be true | "3 copies of v7" |
| Control loop | Repeatedly compares reality to desired state | Starts a replacement |
| Node | One machine in the cluster | A VM |
| Pod | The smallest unit scheduled; usually one container | One copy of your app |
| Replica | One of several identical copies | 3 replicas |
| Service | A stable internal name for a set of pods | How the app finds the DB |
| Ingress | How outside traffic reaches the cluster | Your public URL |
| Rolling update | Replace copies gradually, with rollback | v7 to v8 |
| Health check | A request that proves a copy is alive | `/health` returns 200 |
| Control plane | The brains; often billed separately | Usually not free |
| YAML | The text format cluster state is declared in | Indentation matters |

### Tradeoffs and gotchas

- **You need about two of four signals before a cluster earns its cost:** more than a handful of genuinely distinct services, traffic with a predictable shape, a dedicated person whose job includes the cluster, a constraint the platform cannot meet.
- **Kubernetes solves a coordination problem and costs you one of its own** — networking, storage classes, ingress, permissions, YAML. The complexity moves from "which machine was that on" to "why is the ingress returning 502".
- **A managed control plane is usually not free.** A cluster is several metered resources at once, most running whether or not anyone uses your app.
- **Health checks are the part everyone gets wrong.** A check that only proves the process is alive keeps a wedged copy in rotation forever; a check that queries the database takes your app down when the database hiccups. "Am I alive" versus "am I ready for traffic" is the system's most useful distinction.

### Verify it

AI over-recommends orchestration more reliably than almost anything else.

1. **Make it answer "what breaks today without this?"** Not at a million users — today, with your traffic. If the answer is nothing, it is a solution looking for a problem.
2. **Make it compare against doing nothing.** Ask what you would hit first by staying put; an agent that cannot argue the simple case has not evaluated it.
3. **Make it price the control plane and the nodes,** per chapter 7: numbers, meters, a page.
4. **Ask who operates it, and what happens when they are in an exam.** Name a person or do not build it.

Red flags: Kubernetes for a project with no users; YAML with health checks unexplained; scaling as the main benefit when traffic is flat; a cost claim with no numbers; an architecture justified by future scale or by a CV rather than a present problem.

## Walkthrough
1. Write down, honestly, how many distinct services your product has — not copies, distinct components — and how many machines they run across. Most teams write "one" and "zero, the platform handles it".
2. Score your team against the four signals: multiple services, traffic with a shape, a dedicated owner, a constraint the platform cannot meet. Count how many are true.
3. Open your platform's dashboard and find where it already does each of the six jobs: scheduling, self-healing, scaling, service discovery, rolling updates, secrets. Screenshot the two you did not know it did.
4. Find your platform's deployment history and look at how it replaced the previous version. Note whether it kept the old version serving until the new one was healthy.
5. Find or create a health endpoint in your app that returns 200 and nothing else. `curl -i` it, per chapter 4, and confirm the status code.
6. Ask your AI assistant for the difference between a liveness check and a readiness check, then decide which of the two your health endpoint is, and say why.
7. Ask it for a minimal Kubernetes manifest for your app — purely to read. Identify the desired replica count, the image tag, the service and the health checks. Do not apply it anywhere.
8. Ask it, for that manifest, what breaks today if you do not use it. Record the answer verbatim in your team notes.
9. Ask it to price a small managed cluster in your region with real numbers and the pricing page to check, then check the two largest figures yourself.
10. Write one paragraph in your repository: the architecture you chose, the two alternatives you rejected, and the present-tense reason. This is a decision-log entry and you will reuse it in M11 and M18.

## Talk to AI
```prompt
CONTEXT:      I am a student on an AI-native engineering program. My team is
              three beginners with about two hours a day, in India, on student
              budgets where an unexpected bill of a few thousand rupees is a
              real problem. Our product is [describe it in two sentences] and
              it currently has [number] users. It runs on [your platform] on a
              free tier, as [number] distinct services. Nobody on the team has
              operated a server cluster, and nobody is on call.

GOAL:         Understand what container orchestration actually does, and get
              an honest, evidence-based answer about whether we should run a
              cluster now - including the case for not doing it.

REQUIREMENTS: 1. Explain desired state and the control loop in plain words,
                 with no jargon I have not met
              2. List the jobs orchestration does for me, and for EACH one,
                 tell me whether my current platform already does it and where
                 in its dashboard I can see that
              3. Give me the specific signals that a team has genuinely
                 outgrown a managed platform, and score MY situation against
                 each one honestly
              4. Explain liveness versus readiness checks, why getting them
                 wrong is worse than not having them, and what a correct pair
                 would look like for my app
              5. Price a minimal managed cluster in my region: control plane,
                 nodes, load balancer, storage - as separate meters with units
                 and rates, and the pricing page I should check each against

CONSTRAINTS:  Free tier only where possible, and say clearly where free is not
              possible. Plain English, second-language reader. Rupees. Do not
              state any price as fact - give me the page and the value to read,
              and remind me to date it. Assume we have no dedicated operator.

NON-GOALS:    Do not teach me YAML syntax. Do not set anything up. Do not
              design for a million users. Do not recommend Helm, service
              meshes, operators, or GitOps tooling.

ACCEPTANCE:   - I can state in one sentence the problem orchestration solves
              - For every job it does, I know whether my platform already does
                it for me
              - I have an honest score against the outgrown-the-platform
                signals, and a recommendation that follows from it
              - I can explain the difference between liveness and readiness
              - I have a per-meter cost estimate with pages to verify it
              - I can answer "what breaks today without a cluster?" in one
                sentence

ASK:          Before recommending anything, argue BOTH sides: make the
              strongest honest case for staying on my current platform, and
              the strongest honest case for a cluster, given my actual numbers.
              Then say which you would choose for my team and why. Do not give
              me any manifests or commands until I ask.
```

## Stuck? Ask AI
```prompt
Explain desired state and the control loop to someone who has never used a cluster, with an analogy that is not about shipping containers, orchestras, or restaurants. Then give me five situations - a container crashes, a machine dies, traffic doubles, I push a new image, a pod answers its health check but returns errors - and ask me what the control loop does in each. Tell me which one it cannot fix, and do not give me the answers before I guess.
```

```prompt
Explain the difference between a liveness check and a readiness check, with a concrete example of each for a web app that talks to a database. Then explain the two classic mistakes: a check so shallow it keeps a broken copy in rotation, and a check so deep that one slow dependency takes the whole app out. Tell me which mistake is more common and how I would notice each.
```

```prompt
I am tempted to put Kubernetes on my student project because it would look good when I talk about this work. Give me the honest argument against that, including what I would spend the time on instead and what a reviewer would actually ask me about my architecture. Then tell me what I could learn about orchestration without running a cluster at all.
```

## Quiz

### Q1 · L2
What problem is container orchestration built to solve?
- (a) Making a single application start faster
- (b) Keeping many containers running correctly across many machines without manual intervention ✅
- (c) Reducing the size of container images
- (d) Replacing the need for a database
> It is coordination at scale. One container on one machine needs a container runtime; dozens across many machines need something maintaining the whole arrangement continuously.

### Q2 · L3
You declare three replicas and the control loop sees two running. What does it do?
- (a) Alerts you and waits for a decision
- (b) Starts one more copy on a machine with room ✅
- (c) Restarts all three so they match
- (d) Reduces the desired state to two
> The loop takes the smallest action that closes the gap between reality and your declaration. It does not ask, and it does not restart what is already healthy.

### Q3 · L4
Three pods are running and healthy, and every user gets a 500 error. What does this tell you about orchestration?
- (a) The control loop is broken and should be restarted
- (b) The health checks must be configured as liveness rather than readiness
- (c) Orchestration keeps the count right, not the product working — monitoring is a separate job ✅
- (d) The desired state should be raised to five replicas
> A satisfied control loop means reality matches your declaration. It has no view on whether your application is correct, which is exactly why the observability work in M07 is not a free consequence of running a cluster.

### Q4 · L4
Which of these is a genuine signal that a team has outgrown a managed platform?
- (a) A conference talk made the case for cluster architecture
- (b) The team wants the experience on their CV
- (c) Several distinct services, traffic with real peaks, and someone whose job includes operating it ✅
- (d) A plan to support a million users next year
> The signals are present-tense and concrete. Future scale, enthusiasm and career considerations are the three reasons most often given and none of them changes what breaks today.

### Q5 · L5
Your platform already schedules, self-heals, scales, discovers services, does rolling updates and injects secrets. What follows about whether orchestration is optional?
- (a) Orchestration is optional, and you have chosen not to use it
- (b) Orchestration is not optional in a serious system — you are already using it, operated by someone else ✅
- (c) Your platform is a replacement for orchestration with different capabilities
- (d) You will need to adopt a cluster once you deploy a second service
> This is the core point of the chapter. A managed platform is orchestration with the operations outsourced. The real question is never whether to orchestrate, but who operates it — and for three students with two hours a day, not them.

## Go deeper
- [Kubernetes: what it is and what it is not, from the official docs](https://kubernetes.io/docs/concepts/overview/)
- [Kubernetes: configuring liveness, readiness and startup probes](https://kubernetes.io/docs/tasks/configure-pod-container/configure-liveness-readiness-startup-probes/)
