---
id: m13-03-kubernetes-for-real
module: m13-production-engineering
slug: kubernetes-for-real
title: "Kubernetes for Real: Deployments, Services, Autoscaling, Helm"
order: 3
duration_min: 25
is_free: false
mission: true
volatility: medium
last_reviewed: 2026-11-16
---

## Key learnings
- Read a Deployment, Service and Ingress manifest and say what each one guarantees
- Set resource requests and limits from measured numbers, not from guesses
- Configure a liveness and a readiness probe that fail for the right reasons
- Run a rollout and a rollback on a free local cluster and watch both happen
- Catch the five defects that appear in almost every AI-generated manifest

## Lesson

### The pain

M06 chapter 9 taught you what Kubernetes does and four honest signals for when you need it, and said a three-person student team should not run one. This chapter is the other half: reading, judging and operating a cluster — because your startup grew a third service, an employer runs one, or M18 asks you to defend why you did not.

A manifest has no memory limit: one pod grows to 3 GB, the node runs out, and Kubernetes evicts pods to save itself, then restarts each one, so the control loop reports all is well. Same week, the readiness probe queries the database, a four-second hiccup fails every probe at once, and the blip becomes a ninety-second outage. Both failures are two lines of YAML nobody read.

### The idea

**A cluster does exactly what your manifests say, so the manifest is the product and reading it is the job.**

Four objects carry almost everything. A **Deployment** says how many copies of which image. A **Service** gives those copies one stable internal name. An **Ingress** lets outside traffic in. A **HorizontalPodAutoscaler** changes the copy count from a measured number. Helm is a template engine on top: the same four objects, one values file per environment.

### The analogy

M06 gave you the festival caterer with twelve counters and a written plan; you were reading it, now you write it. Each counter is *promised* table space — the **resource request**, which the scheduler uses to pick a hall with room. Each has a ceiling it may never exceed, the **limit**, because one stall spreading across the hall starves the others. A runner asks two different questions: "are you alive?" and "are you ready for students?" A cook who steps out leaves the counter alive but not ready, so the queue goes elsewhere rather than the stall being shut.

**Where the analogy breaks:** a stall that spills over gets a warning. A container over its memory limit is killed instantly, with only a two-word status: `OOMKilled`.

### Picture it

```
  INGRESS           SERVICE              DEPLOYMENT (3 replicas)
┌───────────┐   ┌──────────────┐   ┌───────────────────────────┐
│ canteen   │──▶│ stable name  │──▶│ pod  image: app:v7        │
│ .in/api   │   │ menu-svc     │   │  requests 128Mi / 100m    │
│ TLS here  │   │ picks a pod  │   │  limits   256Mi / 500m    │
└───────────┘   │ that is READY│   │  liveness  /healthz       │
                └──────────────┘   │  readiness /ready         │
                       ▲           └───────────────────────────┘
                       │                        ▲
                ┌──────┴───────┐                │
                │ HPA: scale   │────────────────┘
                │ 3→8 at 70%   │   replica count
                │ CPU          │
                └──────────────┘
```

Outside traffic enters through the Ingress, the Service forwards it only to pods marked ready, the Deployment keeps the declared number of pods running with declared resources and probes, and the autoscaler changes that number from a measured signal.

### Where it lives

This sits directly on M06 chapter 8's container image — a manifest is instructions for running it.

- The **Service** is M05's load balancer; the **HPA** is M05's horizontal scaling with a trigger; the **probes** are M05's health checks, split in two.
- **Chapter 4** creates the cluster itself, because clicking one into existence is the fastest route to a resource nobody can identify.
- **Chapter 5** uses Deployments for canary releases; **chapter 7**'s load test judges your HPA thresholds.
- **Secrets** stay where M07 put them: a Kubernetes Secret is base64, not encryption.

### Words to own

| Term | Plain meaning | Example |
|---|---|---|
| Deployment | Declares how many pods of which image | 3 replicas of `app:v7` |
| ReplicaSet | The object a Deployment creates per version | One per rollout |
| Service | One stable internal name for a set of pods | `menu-svc` |
| Ingress | How outside traffic and TLS reach a Service | `canteen.in/api` |
| Request | Resources promised to a pod; used to schedule | 128Mi memory |
| Limit | Hard ceiling; exceeding memory means killed | 256Mi memory |
| millicore | 1/1000 of a CPU core | `500m` = half a core |
| Liveness probe | "Is this process wedged?" Failure restarts it | `/healthz` |
| Readiness probe | "Send traffic here?" Failure removes it | `/ready` |
| OOMKilled | Killed for exceeding the memory limit | Pod status |
| HPA | Changes replica count from a metric | 3→8 at 70% CPU |
| Rollout / rollback | Replace versions gradually; return to the last | `v7`→`v8`→`v7` |
| Helm chart | Templated manifests plus a values file | One chart, two envs |
| Namespace | A named partition inside one cluster | `staging`, `prod` |

### Tradeoffs and gotchas

**No limits is the most common defect; no requests is the second.** Without limits one pod starves a node; without requests the scheduler packs pods onto a node that cannot hold them.

**Set the memory limit from a measurement.** Exceeding memory kills the container instantly; exceeding CPU only throttles it, arriving as mysterious slowness.

**A readiness probe must never check a dependency you do not control.** Readiness means "this copy can serve"; liveness means "a restart would help". A liveness probe failing for a reason a restart cannot fix is a restart loop.

**A managed control plane is usually not free, and nodes certainly are not.** Learn on a free single-node cluster inside Docker on your laptop. If you must use a managed one, use trial credits, set a billing alert per M06 chapter 7, and destroy it the same day.

**Helm hides things, which is the point and the risk.** A chart you did not write can create twenty objects from six values, so render it to plain YAML before installing.

### Verify it

1. **Search for `limits` and `requests`.** Both absent is an automatic reject. Then ask where each number came from: a load-test figure is an answer, a sensible default is not.
2. **Read the two probes and say what each tests.** The same path twice is one probe twice; either one touching the database is a no.
3. **Check the image tag.** `latest` or a branch name means your rollback is imaginary.
4. **Ask for the rollback command before you apply anything,** and confirm the previous ReplicaSet survives.
5. **Make it render, not describe.** Generate plain YAML from the chart, read the objects, and find every secret in it.

Red flags: no resource limits; probes that query a database; `latest` as a tag; an HPA with no maximum, so a spike scales into a bill; a Service whose labels match no pod, failing silently with zero endpoints.

## Walkthrough
1. Install a free single-node local cluster that runs inside Docker on your own laptop, plus the command-line client. No cloud account, no card, no bill. Confirm one node appears and is ready.
2. Build your app's container image from M06 chapter 8, tag it with your current commit SHA rather than `latest`, and load it into the local cluster. Record the tag in your notes.
3. Ask AI for a Deployment and a Service for that image, explicitly requiring resource requests, limits, and separate liveness and readiness probes. Read the YAML line by line before applying it.
4. Apply it, then watch the pods come up. Describe one pod and find the requests, limits and both probe definitions in the output. Confirm the YAML you read is the state the cluster holds.
5. Deliberately break readiness: make `/ready` return a failure code while `/healthz` still succeeds. Confirm the pod stays running but leaves the Service's endpoint list and stops receiving traffic. This is the distinction that matters most.
6. Now set the memory limit absurdly low, apply, and send a few requests. Find the `OOMKilled` status. This is the two-word evidence you will otherwise hunt for in a real incident.
7. Restore a sensible limit, then do a rollout: change one visible thing, rebuild with a new SHA tag, update the Deployment, and watch old pods retire as new ones become ready. Note how many were serving at the thinnest moment.
8. Roll back to the previous tag with one command. Time it. Write the number down — it is the figure chapter 5 and chapter 10 both depend on.
9. Add a HorizontalPodAutoscaler with a CPU target, a minimum and a **maximum**. Generate load against the Service and watch the replica count climb and then settle. Note how long scaling up actually took.
10. Convert your two manifests into a minimal Helm chart with one values file for staging and one for production, differing only in replica count and limits. Render both to plain YAML and diff them. The diff must contain nothing you did not intend.
11. Delete the local cluster and recreate it from your chart in under five minutes. If you cannot, something lives only in your shell history, which is chapter 4's problem.

## Talk to AI
```prompt
CONTEXT:      I am a student on an AI-native engineering program. I direct AI
              and verify its output; I do not write code by hand. My team of
              three runs a product called [name] that does [two sentences]
              and currently has [number] real users and [number] distinct
              services. Our container image is [image name], built from a
              Dockerfile, tagged with the git commit SHA. We are learning on a
              FREE single-node local Kubernetes cluster running inside Docker
              on a student laptop (8 GB RAM). Nobody on the team has operated
              a cluster and nobody is on call. An unexpected cloud bill of a
              few thousand rupees is a real problem for us.

GOAL:         Manifests I can read, judge and operate - a Deployment, a
              Service, an Ingress and an autoscaler for ONE of our services -
              plus the specific numbers I need to measure rather than guess.

REQUIREMENTS: 1. A Deployment with resource requests AND limits for both CPU
                 and memory. For each of the four numbers, tell me how I
                 MEASURE the right value rather than giving me a default, and
                 what symptom I will see if each one is too low
              2. Separate liveness and readiness probes, with different
                 paths, and one sentence each on exactly what failure means.
                 The readiness probe must not depend on our database - explain
                 why in terms of what happens during a 4-second DB hiccup
              3. A Service, and a note on how its label selector matches the
                 pods, plus the read-only command that proves it has non-zero
                 endpoints
              4. A HorizontalPodAutoscaler with a target, a minimum AND a
                 maximum, and an explanation of why the maximum protects my
                 wallet
              5. The exact read-only commands to inspect a rollout in
                 progress, and the single command to roll back - plus what
                 must be true for rollback to work at all
              6. A minimal Helm chart wrapping the above, with values files
                 for staging and production that differ ONLY in replica count
                 and resource limits, and the command that renders plain YAML
                 so I can read what would be created

CONSTRAINTS:  Free only. Assume a local single-node cluster, not a managed
              cloud cluster - say clearly wherever that changes something.
              Image tags are commit SHAs, never `latest`. Plain English for a
              second-language reader. Label every command READ-ONLY or
              CHANGES-THINGS. Do not put any real secret value in a manifest.
              Rupees for any cost.

NON-GOALS:    Do not add a service mesh, an operator, a CRD, cert-manager, or
              GitOps tooling. Do not create any cloud resource. Do not
              recommend a managed cluster. Do not teach me YAML syntax as an
              exercise - I am going to READ this, not type it.

ACCEPTANCE:   - Every container has both requests and limits, and I can say
                where each number came from
              - My liveness and readiness probes have different paths and I
                can state what each failure does to the pod
              - Breaking readiness removes the pod from the Service while the
                pod keeps running, and I have watched that happen
              - A too-low memory limit produces OOMKilled, and I have seen it
              - I can roll back with one command and I have timed it
              - Rendering my staging and production values produces a diff
                containing only replica count and limits

ASK:          Before any YAML, list the five defects you most often see in
              generated Kubernetes manifests and tell me how I would spot each
              one by reading. Then give me the manifests with a comment on
              every line I am expected to judge rather than accept.
```

## Mission

**Objective.** Run your own startup's service on a free local Kubernetes cluster with measured resource limits, two correctly separated probes and a timed rollback, and prove each of those with your own observation rather than the AI's summary.

**Steps.**
1. **Frame** — pick exactly ONE of your services. Write down its image, its current memory use under normal traffic, and how many real users it serves. One service only.
2. **Measure first** — before writing any manifest, run the service in a container and record its memory and CPU at idle and under chapter 7's load. These two pairs of numbers become your requests and limits. Guessing here is the whole failure this mission exists to prevent.
3. **Brief** — use the Director's Brief above with your real numbers. Demand the five-defects list before any YAML.
4. **Direct** — have it produce the Deployment, Service and autoscaler. Read every line. Mark the lines you had to look up.
5. **Verify probes** — break readiness while leaving liveness healthy. Screenshot the pod still running and the Service endpoint list dropping to zero for it.
6. **Verify limits** — set memory far too low, trigger the work, and capture the `OOMKilled` status. Then restore your measured value and show the pod stable.
7. **Verify rollback** — deploy a visibly different version, then roll back with one command. Time it with a stopwatch and record the seconds.
8. **Verify autoscaling** — generate load, record the replica count over time, and state whether your HPA maximum would have protected you from a bill on a managed cluster.
9. **Package** — wrap it in a Helm chart with staging and production values, render both, and diff them.
10. **Rebuild from nothing** — delete the cluster, recreate it from your chart, and confirm the service answers. Record how long it took.
11. **Document** — one decision-log entry in your repo: why you did or did not choose a cluster for your real deployment, with your measured numbers as the evidence. This entry is what M18 will ask you to defend.

**Upload.**
- Your Director's Brief, in full → post caption or an attached file
- Your measured idle and under-load memory and CPU figures, taken *before* the manifest → screenshot
- Your final Deployment and Service YAML, with your four resource numbers visible → text or screenshot
- The AI conversation including its five-defects list → share link in `ai_chat_url`
- Readiness broken: pod running, endpoints zero → two screenshots
- The `OOMKilled` pod status → screenshot
- Your stopwatch rollback time, in seconds → text
- Replica count over time during your load test → screenshot
- The rendered diff between staging and production values → text
- Proof of rebuild from nothing, with elapsed time → screenshot
- Reflection, 150 words max → one tradeoff you chose, and one thing the AI got wrong or left out

**Self-check.** Tick every box before you submit.
- [ ] Every container in my manifest has both requests and limits
- [ ] I measured those numbers before the manifest existed, and I can say how
- [ ] My liveness and readiness probes use different paths
- [ ] My readiness probe does not touch the database, and I can say why
- [ ] I watched a pod stay alive while leaving the Service, with my own eyes
- [ ] I produced an `OOMKilled` on purpose and recognise the status
- [ ] My image tag is a commit SHA, not `latest`
- [ ] I timed a rollback with a stopwatch and know the number
- [ ] My HPA has a maximum, and I can explain what it protects
- [ ] I rendered my Helm values and the diff contained only what I intended
- [ ] No real secret value appears in any file I committed
- [ ] My reflection names something specific the AI got wrong

**Stretch.** Price the same workload on a managed cluster in the Mumbai region as separate meters — control plane, nodes, load balancer, storage — in rupees per month, with the pricing page for each. Then write 80 words on whether your product's present-tense problem justifies that number.

## Stuck? Ask AI
```prompt
Here is a Kubernetes Deployment manifest: [paste the complete YAML]. Do not rewrite it. Instead, audit it as if you will be paged at 2 a.m. when it fails. List every missing or dangerous field in order of how much damage it will do, and for each one tell me the exact symptom I would observe in a real incident and the read-only command that would reveal it. Finish by telling me which single line I should change first.
```

```prompt
Explain the difference between a liveness probe and a readiness probe using a concrete web app that talks to a database. Then walk me through what happens, second by second, in two scenarios: a readiness probe that queries the database during a 4-second database hiccup, and a liveness probe that queries the database during the same hiccup. Tell me which produces a worse outage and why, then ask me to design a correct pair before you show me one.
```

```prompt
My pod keeps restarting and I cannot tell why. Explain the difference between CrashLoopBackOff, OOMKilled, ImagePullBackOff, Pending and a failing probe - what each one actually means about where the fault is. For each, give me the read-only command that confirms it and tell me what a healthy result looks like. Then ask me what status I am seeing so you can narrow it down.
```

## Quiz

### Q1 · L3
Your pod's memory limit is set too low. What do you observe?
- (a) The pod runs slowly as memory is throttled
- (b) The pod is killed with status `OOMKilled` and restarted ✅
- (c) The scheduler refuses to place the pod on any node
- (d) The autoscaler adds replicas to compensate
> Memory is not throttleable: exceeding the limit means instant termination. CPU is the opposite — exceeding a CPU limit throttles you, producing mysterious slowness rather than a crash, which is why the two numbers fail in completely different ways.

### Q2 · L4
Your readiness probe queries the database. The database has a four-second hiccup. What happens?
- (a) Nothing; readiness probes do not affect traffic
- (b) Every pod fails readiness at once and leaves the Service, turning a 4-second blip into a full outage ✅
- (c) The liveness probe restarts the pods, which fixes it
- (d) Kubernetes scales up to route around the database
> Readiness controls whether a pod receives traffic. A probe that depends on something you do not control removes all pods simultaneously. Readiness means "this copy can serve"; the database's health is M07's monitoring problem, not the probe's.

### Q3 · L3
Why does `image: latest` break your ability to roll back?
- (a) Kubernetes refuses to pull images tagged `latest`
- (b) Both the old and new Deployment point at the same moving label, so there is no distinct previous version to return to ✅
- (c) `latest` images are always larger and slower to pull
- (d) Rollback requires a Helm chart, which `latest` prevents
> Rollback means switching to a previously known-good immutable artefact. A moving tag has no history, so the rollback command succeeds and changes nothing. This is why M01's commit SHA is the right tag.

### Q4 · L4
An AI gives you a Deployment with no `resources` block at all. What is the strongest reason to reject it?
- (a) Kubernetes will refuse to apply a manifest without resources
- (b) Without requests the scheduler cannot place pods sensibly, and without limits one pod can exhaust a node and cause unrelated pods to be evicted ✅
- (c) The HorizontalPodAutoscaler cannot be created without them
- (d) Helm charts require a resources block to render
> It applies happily, which is the trap. Requests are the scheduler's only information about room; limits are the only thing stopping one misbehaving pod from taking down everything that shares its node.

### Q5 · L5
Three pods are running, all probes pass, the HPA is satisfied, and every user gets a 500 error. What does that tell you about where to spend your next hour?
- (a) Increase the replica count until the errors stop
- (b) Restart the control plane, which has lost track of pod state
- (c) In the application and its dependencies — the cluster reports count, not correctness, so this is an observability question ✅
- (d) Lower the readiness probe's failure threshold so broken pods are removed
> A satisfied control loop means reality matches your declaration and nothing more. M06 made this point at overview level and it is the first thing a real cluster will make you relearn: M07's metrics and logs, not the pod list, are where the answer is.

## Go deeper
- [Kubernetes: managing resources for containers](https://kubernetes.io/docs/concepts/configuration/manage-resources-containers/)
- [Helm: charts, values and templates](https://helm.sh/docs/topics/charts/)
