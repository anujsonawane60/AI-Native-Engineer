---
id: m13-03-kubernetes-for-real
module: m13-production-engineering
slug: kubernetes-for-real
title: "Kubernetes for Real: Deployments, Services, Autoscaling, Helm"
order: 3
duration_min: 35
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

M06 chapter 9 taught you what Kubernetes does and gave you four honest signals for when you need it. It also told you, correctly, that a three-person student team should not run one. This chapter is the other half: you are now at the point where you have to read, judge and operate a cluster — maybe because your startup grew a second and third service, maybe because an employer runs one, maybe because M18's architecture defence requires you to explain why you did not.

So here is what going wrong actually looks like. A team moves their canteen app onto a cluster. The manifest an agent produced has no memory limit. One pod gets a request that loads six months of orders into memory, grows to 3 GB, and the node runs out. Kubernetes starts evicting pods to save itself — including the database proxy and the pod serving the menu. The control loop reports everything is fine, because it faithfully restarts each evicted pod, which then gets evicted again. The team spends two hours reading logs from containers that no longer exist.

Second failure, same week. The readiness probe queries the database. The database has a four-second hiccup. Every pod fails its probe at the same moment, Kubernetes removes all of them from the Service, and a four-second database blip becomes a ninety-second total outage. The database recovered on its own; the cluster turned a wobble into a disaster.

Neither failure is about Kubernetes being hard. Both are two lines of YAML nobody read.

### The idea

**A cluster does exactly what your manifests say, so the manifest is the product and reading it is the job.**

Four objects carry almost everything you need. A **Deployment** says how many copies of which image. A **Service** gives those copies one stable internal name. An **Ingress** lets outside traffic in. A **HorizontalPodAutoscaler** changes the copy count based on a measured number. Helm is a template engine on top, so the same four objects can be reused across staging and production with one values file per environment.

### The analogy

M06 gave you the festival caterer with twelve counters and a written plan. You were the person reading the plan. Now you write it.

The caterer's plan has lines you did not notice before. Each counter is *promised* a certain amount of table space — that is the **resource request**, and the scheduler uses it to decide which hall has room. Each counter also has a hard ceiling it may never exceed — the **limit** — because one stall spreading across the hall starves the others. A runner asks each counter two different questions: "are you still alive?" and, separately, "are you ready for students?" A counter whose cook has stepped out for two minutes is alive but not ready, and the queue should be sent elsewhere rather than the stall being shut down.

**Where the analogy breaks:** a stall that spills over its allotted space gets a word from the caterer and a chance to tidy up. A container that exceeds its memory limit is killed instantly, with no warning and no negotiation, and the only evidence is a two-word status: `OOMKilled`. There is no gradual degradation to notice. This is why the limit is a number you measure rather than a number you type.

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

This sits directly on top of M06 chapter 8's container image: a manifest is instructions for running the image you already built. The Service is M05's load balancer, internal and automatic. The HPA is M05's horizontal scaling with a trigger attached. The probes are M05's health checks, now split into two kinds. Chapter 4 of this module is how the cluster itself gets created, because clicking a cluster into existence is the fastest way to a resource nobody can identify. Chapter 5 uses Deployments to do canary releases properly, and chapter 7's load test is what tells you whether your HPA thresholds were sensible.

Secrets stay exactly where M07 put them: a Kubernetes Secret is base64, not encryption, and anyone who can read the manifest can read the value.

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

**No limits is the single most common defect, and no requests is the second.** Without limits, one pod can starve a node. Without requests, the scheduler has no idea how much room anything needs and will pack pods onto a node that cannot hold them. A manifest with neither is not a manifest; it is a wish.

**Set the memory limit from a measurement, and set it generously.** Kubernetes kills a container that exceeds memory, instantly. CPU behaves differently: exceeding the CPU limit only throttles you, which shows up as mysterious slowness rather than a crash. So a too-low memory limit crashes your app and a too-low CPU limit makes it slow. Both come from typing a round number instead of watching the pod under chapter 7's load test.

**A readiness probe must never check a dependency you do not control.** If it queries the database, a database hiccup removes every pod from rotation at once. Readiness means "this copy can serve"; liveness means "this process is wedged and a restart would help". If your liveness probe can fail for a reason a restart cannot fix, you have built a restart loop.

**`image: latest` destroys your ability to roll back.** The tag must name an immutable build. Otherwise "roll back to the previous version" has no meaning, because both versions point at the same moving label. Use the commit SHA from M01.

**A managed control plane is usually not free, and the nodes certainly are not.** This is the chapter where a student can produce a surprise bill in rupees. Learn on a free local cluster — a single-node cluster inside Docker on your own laptop costs nothing and teaches ninety per cent of this. If you must use a managed cluster, use trial credits, run it inside one live session, and destroy it before you close your laptop. Set a billing alert first, per M06 chapter 7.

**Helm hides things, which is the point and the risk.** A chart you did not write can create twenty objects from six values. Render the templates to plain YAML and read the output before you install anything. "It is just a chart from the internet" is how a cluster acquires objects nobody can explain.

**Three healthy pods can still serve errors to everyone.** M06 said this and it is worth repeating because the cluster will reassure you. The control loop is satisfied when the count matches. M07's metrics are what tell you the product works.

### Verify it

AI writes plausible Kubernetes YAML faster than anything else in this module, and it is confidently incomplete. Run these six checks on every manifest before it goes anywhere.

1. **Search the manifest for `limits` and `requests`.** Both absent is an automatic reject. Then ask where each number came from. "A sensible default" is not an answer; a figure from a load test is.
2. **Read the two probes and say out loud what each one tests.** If they point at the same path, you have one probe twice. If either touches the database, say no.
3. **Check the image tag.** If it is `latest` or a branch name, your rollback is imaginary.
4. **Ask for the rollback command before you apply anything**, and confirm the previous ReplicaSet will still exist. A rollout with no tested rollback is a one-way door.
5. **Find every secret in the YAML.** If a real credential is sitting in a Secret value in a file you are about to commit, stop. Base64 is not encryption.
6. **Make it render, not describe.** For a Helm chart, generate the plain YAML and read the actual objects. Summaries omit exactly the fields that hurt you.

Red flags: a manifest with no resource limits; probes that query a database; `latest` as an image tag; an HPA with a target but no maximum, so a traffic spike scales into a bill; a Service with no matching pod labels, which fails silently with zero endpoints; and a recommendation to install a service mesh or an operator before you have measured anything.

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
