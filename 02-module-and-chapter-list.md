# 02 · Module and Chapter List (v2)

20 `CourseModule` rows · 231 `CourseChapter` rows · 69 missions
`Course.slug = "ai-native-engineer"`

Each chapter is one 20–30 minute read. `R` = `min_tier_rank`.
🚀 = the chapter carries a **mission** (`requires_activity = true`).

> **Counts in this file are authored by hand and WILL drift.** Generate them
> from the seed script at build time — see `01-platform-models.md` §3.6.

| Track | R | Reaches | Modules | Chapters | Missions | Price |
|---|---|---|---|---|---|---|
| Sprint · 30–45 days | 1 | M00–M11 | 12 | 142 | 32 | ₹5,000 |
| Build · 3 months | 2 | + M12–M15 | 16 | 189 | 49 | ₹8,000 |
| Company · 6 months | 3 | + M16–M19 | 20 | 231 | 69 | ₹10,000 |

### The time model (Sprint)

142 chapters × 25 min ≈ **59 h reading** · 32 missions × ~1.5 h ≈ **48 h building**
≈ **107 h self-directed**, plus **60–90 h live** (2 h/day × 30–45 days) — inside
the 120–180 h the page advertises, because missions are largely built *in* the
live room rather than on top of it.

**Missions are the product build, not homework on top of it.** Every mission
applies the chapter's concept to the team's own startup, so mission hours and
MVP hours are the same hours. The canteen ("Tiffin") stays only as the worked
example inside lesson text.

---

# RANK 1 · SPRINT

The full 200+ concept map, your team, your shipped MVP. 12 modules · 142 chapters.

## M00 · Orientation: How Software Works and How to Direct AI
`R1` · 10 ch · 2 missions

1. The Four Boxes: Client, Server, Network, Data
2. Follow One Tap: An Order from Phone to Database and Back
3. Frontend, Backend, Infrastructure: Who Builds What
4. Inside the Machine: CPU, RAM, Disk, OS and Processes
5. The Terminal, the File System and How Data Is Encoded
6. What an LLM Is, and What Chat AI, App Builders and Coding Agents Each Do
7. Why AI Gets Things Wrong, and How You Catch It
8. The Director's Loop and the Director's Brief 🚀
9. Agent Permissions, Sandboxes and Safe Execution
10. Your Toolkit: Accounts, API Keys, Free Tiers and Billing Alerts 🚀

## M01 · Your Team, Your Startup and Version Control
`R1` · 10 ch · 3 missions

1. A Group of Three: Splitting Product, Tech and Growth
2. Finding a Problem Worth Solving, and Who Has It
3. Users, Personas and Validating an Idea in One Week 🚀
4. The PRD Is the Master Prompt 🚀
5. Scoping an MVP You Can Ship, and Your North Star Metric
6. Git: Why Version Control Matters More When AI Writes the Code
7. Branches, Commits and Merge Conflicts When Agents Touch Many Files
8. Pull Requests and Reviewing a Diff You Did Not Write 🚀
9. The Daily Live Session: Standups, Blockers and Reviews
10. Your Daily Portfolio and Decision Log

## M02 · The Web from the Ground Up
`R1` · 16 ch · 2 missions

1. HTML: Tags, Elements, Attributes and Semantic Structure
2. The DOM as a Tree; Forms, Inputs, Links and Images
3. Reading a Page You Did Not Write: View Source and DevTools 🚀
4. CSS: Selectors, the Cascade and the Box Model
5. Layout: Flexbox vs Grid, and When to Use Each
6. Responsive Design: Viewports, Breakpoints and Units
7. Design Tokens, CSS Variables, Dark Mode and Utility-First CSS
8. What JavaScript Does: Events, State and Changing the Page
9. Functions, Objects, Arrays and JSON as Ideas
10. Async, Promises and fetch: Why the Page Does Not Freeze
11. Types: Why Teams Add TypeScript
12. IP Addresses, DNS, Domains and Ports
13. HTTP: Methods, Status Codes and Headers
14. HTTPS, TLS and Certificates; Cookies, Sessions and CORS
15. Why Frameworks Exist: Components, Props, State and the Landscape
16. Rendering Strategies, Bundlers, CDNs and Core Web Vitals 🚀

## M03 · Design: UX, UI and Accessibility
`R1` · 10 ch · 2 missions

1. User Research, User Flows and Information Architecture
2. Wireframes: Lo-Fi to Hi-Fi
3. Usability Heuristics, Cognitive Load and the Cost of Choices
4. Usability Testing With Three Friends 🚀
5. Visual Hierarchy, Typography and Colour
6. Spacing, Grids and Component States
7. Design Systems, Tokens and Component Libraries 🚀
8. Avoiding the Generic "AI Look"
9. Accessibility: WCAG, Alt Text, Keyboard and Screen Readers
10. Motion, Micro-Interactions and 3D on the Web

## M04 · Backend, Data, Auth and Payments
`R1` · 17 ch · 4 missions

1. What a Server Does; APIs, Endpoints and Request/Response Contracts
2. REST, GraphQL and gRPC: Three Ways to Shape an API
3. Webhooks, WebSockets and SSE: When the Server Talks Back
4. API Docs, OpenAPI and Versioning 🚀
5. Serverless Functions, Monoliths and Microservices
6. Tables, Rows, Keys and Relationships
7. Designing Your Schema 🚀
8. SQL vs NoSQL, and When to Use Each
9. Reading a Query: Joins, Indexes and Query Plans
10. Search and Ranking: Full-Text, Fuzzy and Relevance
11. Transactions, ACID, Migrations and Backups
12. Authentication vs Authorization; Passwords, Hashing, Sessions and JWTs
13. OAuth, SSO, Passkeys and 2FA 🚀
14. Roles, Permissions and Row-Level Security
15. File Uploads: Object Storage, Buckets and Signed URLs
16. Background Work: Queues, Workers, Cron and Event-Driven Patterns
17. Payments: Gateway, UPI, Checkout, Webhooks and Idempotency 🚀

## M05 · Scale and Reliability
`R1` · 9 ch · 2 missions

1. Why Things Get Slow: Latency Numbers Worth Knowing
2. The Cache Layers: Browser → CDN → App → Redis → Database
3. Hits, Misses, TTL, Eviction and Invalidation 🚀
4. Rate Limiting: Token Bucket, Sliding Window and HTTP 429 🚀
5. Vertical vs Horizontal Scaling; Stateless Servers and Load Balancers
6. Database Scaling: Replicas, Pooling, Sharding and Partitioning
7. Consistency, the CAP Theorem and What You Trade Away
8. Reliability: SLOs, Retries, Backoff, Timeouts and Circuit Breakers
9. System Design Case Studies: Shortener, Chat, Delivery, UPI

## M06 · Cloud, Linux, DevOps and Testing
`R1` · 13 ch · 3 missions

1. Why Servers Run Linux; the File System, Users and Permissions
2. Processes, systemd, Logs and Package Management
3. SSH, Key Pairs and Firewalls
4. Commands as Vocabulary: ls, grep, top, df, curl 🚀
5. What "the Cloud" Is: IaaS, PaaS, SaaS, BaaS and Serverless
6. The Cloud Service Map, and Zero-Config Platforms
7. Regions, Zones, Cost Models and Billing Traps
8. Containers: Images, Dockerfiles, Compose and Registries
9. Orchestration: What Kubernetes Does, and When You Do Not Need It
10. Unit, Integration and End-to-End Tests: What Each Catches 🚀
11. Debugging: Reading a Stack Trace, Reproducing, Bisecting a Failure 🚀
12. CI and CD: Checks on Every Push, Environments and Preview Deploys
13. Domains, DNS Records, TLS Certificates and Infrastructure as Code

## M07 · Security, Observability and Analytics
`R1` · 9 ch · 2 missions

1. OWASP Top Ten in Plain English
2. Secrets, Environment Variables and Leaked API Keys
3. AI-Specific Risks: Insecure Defaults, Prompt Injection, Excessive Agency 🚀
4. Monitoring vs Observability: Logs, Metrics and Traces
5. Structured Logs, Levels and What Is Worth Logging
6. Metrics, Percentiles and p95; Traces and Spans
7. Error Tracking, Uptime Monitors and Alerting Without Fatigue 🚀
8. Events, Funnels, Cohorts, Retention and Churn
9. A/B Tests, Significance, Privacy and Consent

## M08 · Mobile and Multi-Platform
`R1` · 5 ch · 1 mission

1. Native vs Cross-Platform vs PWA vs Hybrid, and How to Choose
2. App Lifecycle, Permissions and Push Notifications
3. Offline-First, Sync, Local Storage and Deep Links
4. Performance and Data on Low-End Android
5. Turn Your Product into an Installable PWA 🚀

## M09 · AI Foundations: ML, Deep Learning and LLMs
`R1` · 16 ch · 2 missions

1. Rules vs Learning From Data: Features, Labels, Training, Inference
2. Supervised, Unsupervised, Self-Supervised and Reinforcement Learning
3. Overfitting, the Train/Val/Test Split and Honest Metrics
4. Classic ML: Regression, Classification, Clustering and Trees
5. Neurons, Layers, Weights and the Forward Pass
6. Loss, Gradient Descent and Backpropagation
7. Why GPUs Matter; CNNs, RNNs, Parameters and Scaling Laws
8. Tokens, Tokenization and Next-Token Prediction
9. Embeddings: Meaning as Coordinates
10. Transformers and Attention, Visually
11. Pretraining, Instruction Tuning, RLHF and Reasoning Models
12. Context Windows, Temperature and System Prompts
13. Choosing a Model: Quality, Latency, Context, Privacy and Cost 🚀
14. Token Economics: What Your AI Feature Actually Costs 🚀
15. Diffusion and Generative Media: From Noise to Image
16. Bias, Alignment, Jailbreaks, Red Teaming and AI Governance

## M10 · Building with AI: RAG, Agents and Harnesses
`R1` · 20 ch · 4 missions

1. System Prompts, Roles and Few-Shot Examples
2. Reasoning Models: Decomposition, Verification, Cost and Latency
3. Structured Outputs and JSON Schemas
4. Tool and Function Calling
5. Context Engineering: What Goes In, and In What Order
6. Prompt Caching and Why It Changes Your Costs
7. Prompt Injection and How to Defend Against It
8. Grounding AI in Private, Current and Domain-Specific Data
9. Ingestion and Data Quality: PDFs, Scans, Tables and Multi-Modal Input 🚀
10. The RAG Pipeline: Chunk → Embed → Store → Retrieve
11. Hybrid Search, Reranking and Citations
12. RAG Failure Modes, Test Sets and Evals 🚀
13. Workflow vs Agent, and When Deterministic Wins
14. The Agent Loop: Stopping Conditions, Failure Recovery and Guardrails
15. Agent Patterns and Multi-Agent Orchestration 🚀
16. MCP: One Port for Every AI Tool
17. AI Harnesses: The Loop, Tools, Permissions and the Sandbox
18. Harnesses Compared: Claude Code, Codex, Cursor, Gemini CLI; Rules Files, Skills, Sub-Agents, Hooks
19. Making a Repo Agent-Ready: Linters, Test Harnesses, Self-Verifying Diffs 🚀
20. Memory, Bots and Voice: What the Assistant Remembers, and Where It Lives

## M11 · Ship Your MVP: Build Checkpoints
`R1` · 7 ch · 5 missions

> **Scheduling note:** this module is read *across* the Sprint, not at the end.
> Checkpoint 1 is week 1. The product is built by the missions in M01–M10; these
> chapters are the review gates. `order_index` puts the module last, but the
> live-session calendar sequences it from day one.

1. Checkpoint 1 · Problem, PRD and Architecture Diagram 🚀
2. Checkpoint 2 · Walking Skeleton, Deployed 🚀
3. Directing an Agent Across a Whole Feature
4. Reviewing AI's Work: Diffs, Tests and Red Flags 🚀
5. Checkpoint 3 · Auth, Data and One AI Feature, With Tests 🚀
6. Going Live: Domain, TLS, Monitoring and Analytics
7. Checkpoint 4 · Ten Users, an Eval Set, a Decision Log and a Technical Demo 🚀

---

# RANK 2 · BUILD

+4 modules · +47 chapters · +17 missions

## M12 · Ship Web and Mobile Apps for Real
`R2` · 11 ch · 4 missions

1. Past the MVP: What "Production" Actually Demands
2. React Native and Expo: One Codebase, Two Stores 🚀
3. Push Notifications People Do Not Mute
4. Offline and Sync When the Network Is Bad 🚀
5. App Store and Play Store Submission, Start to Finish 🚀
6. OTA Updates and Shipping Without a Review Queue
7. Native Device APIs: Camera, Location, Files
8. Crash Reporting and Mobile Observability
9. Performance Budgets on Real Devices 🚀
10. Accessibility and Localisation on Mobile
11. Store Listings, Screenshots and the First-Run Experience

## M13 · Production Engineering: Architecture, Testing and Operations
`R2` · 12 ch · 4 missions

> This module holds the depth deferred out of Sprint. Sprint covers each of
> these concepts once, at overview level; here they become practice.

1. GraphQL and gRPC in Practice: Schemas, Resolvers, Contracts 🚀
2. Event-Driven Architecture at Scale: Kafka, Streams, Idempotent Consumers
3. Kubernetes for Real: Deployments, Services, Autoscaling, Helm 🚀
4. Infrastructure as Code: Terraform, State, Drift and GitOps 🚀
5. Advanced Release Engineering: Blue-Green, Canary, Feature Flags, Rollback
6. Regression Testing and Test-Data Management
7. Load Testing and Finding Your Breaking Point 🚀
8. Caching Strategy at Scale: Layers, Stampedes, Invalidation Graphs
9. Multi-Region, Failover, Backups and Disaster Recovery
10. Incident Response, On-Call and Blameless Postmortems
11. Security Hardening: Threat Modelling, Pen-Test Basics, Dependency Scanning
12. Data Engineering for Products: Pipelines, Warehouses, ELT and BI

## M14 · Deeper ML, Deep Learning and Multi-Agent Systems
`R2` · 14 ch · 5 missions

1. Running a Real Training Loop 🚀
2. Datasets and Data Quality: The Part That Decides Everything
3. The Customization Ladder: Prompt → RAG → Fine-Tune → Train
4. SFT, LoRA and QLoRA in Practice 🚀
5. DPO, Distillation and When Fine-Tuning Is the Wrong Answer
6. Quantization and Running Models on Cheap Hardware 🚀
7. Inference, GPUs and Serving Providers
8. Reinforcement Learning: Policies, Rewards and Reward Hacking
9. Diffusion Deep Dive: Latents, Guidance, ControlNet, Style LoRAs
10. Writing Evals That Actually Catch Regressions 🚀
11. Multi-Agent Systems: Orchestrators, Sub-Agents and Handoffs 🚀
12. Agent Memory Architectures in Production
13. MLOps and LLMOps: Registry, Versioning, Drift and Tracing
14. AI Gateways, Semantic Caching and Cost Control

## M15 · Content Engine and Social Automation
`R2` · 10 ch · 4 missions

1. Distribution Is the Product's Other Half
2. SEO Foundations: Crawling, Indexing and Search Intent
3. Technical SEO: Sitemaps, Schema, Canonicals and Core Web Vitals 🚀
4. Programmatic and Landing-Page SEO at Scale 🚀
5. A Content Engine With AI, Without Sounding Like AI
6. Brand Voice, Style Guides and Visual Consistency
7. Generating Brand Imagery and Video With AI 🚀
8. Automated Social Posting: Scheduling, Queues and Multi-Channel
9. Repurposing One Idea Into Ten Formats
10. Measuring Content: Attribution Without Lying to Yourself 🚀

---

# RANK 3 · COMPANY

+4 modules · +42 chapters · +20 missions

## M16 · Users, Behaviour Analytics and Growth Loops
`R3` · 11 ch · 5 missions

1. Getting Real Users: The Unscalable First Hundred 🚀
2. Instrumenting Behaviour Properly: Event Taxonomy 🚀
3. Funnels, Drop-Off and Finding the Real Leak 🚀
4. Retention Curves, Cohorts and the Flattening Point
5. Session Replays and Heatmaps: Watching People Struggle
6. Activation and Onboarding: The First Five Minutes 🚀
7. Growth Loops vs Funnels
8. Referral, Content and Paid Loops Compared
9. Running a Real Experiment Program 🚀
10. Churn: Measuring It, Predicting It, Reducing It
11. Privacy, Consent, DPDP and GDPR With Real User Data

## M17 · Distribution, Revenue and Running the Company
`R3` · 11 ch · 5 missions

1. Positioning: Why Anyone Should Care
2. Pricing Models and Choosing Yours 🚀
3. Monetization: Subscriptions, Usage, Freemium, Marketplace
4. Billing for Real: Plans, Upgrades, Dunning and Invoices 🚀
5. Unit Economics: CAC, LTV, Margin and Payback 🚀
6. AI Cost Economics: When Inference Eats Your Margin 🚀
7. Launch Channels and Running a Real Launch 🚀
8. Partnerships, Communities and B2B Outreach
9. Support, Reliability and the Trust You Can Lose Overnight
10. Legal and Compliance Basics for an Indian Startup
11. Metrics That Matter and the Weekly Operating Review

## M18 · Demo Day: Ship and Defend
`R3` · 6 ch · 6 missions

1. The Architecture Walkthrough Document 🚀
2. The Ten "What If" Questions 🚀
3. Live Debugging Under Questioning 🚀
4. Telling the Story: Problem, Insight, Product, Traction 🚀
5. Operational Handover: Runbook, Costs, Access, Known Issues 🚀
6. Demo Day: Present and Defend 🚀

## M19 · Frontier and Hardware Electives
`R3` · 14 ch · 4 missions

> Not advertised on `/internship` and not part of any track's promise. Placed at
> rank 3 as a Company-tier bonus **because the platform cannot express an
> optional chapter** — every published chapter counts in the progress
> denominator (`01-platform-models.md` §3.6). Rank 3 keeps it out of Sprint and
> Build denominators while still shipping the content.

1. Hashing, Digital Signatures and Key Pairs
2. Distributed Ledgers, Blocks and Consensus
3. Smart Contracts, Tokens, Wallets and Gas
4. Do You Even Need a Blockchain? 🚀
5. Bits vs Qubits: Superposition and Measurement
6. Entanglement, Interference and the Bloch Sphere
7. What Quantum Is Actually Good At
8. NISQ, Error Correction and Post-Quantum Cryptography 🚀
9. Microcontrollers: Arduino vs ESP32 vs Raspberry Pi
10. GPIO, Sensors, Actuators and Power
11. Wi-Fi, BLE, LoRa and MQTT
12. Firmware, Flashing, Edge AI and IoT Security 🚀
13. Robot Anatomy, Control Loops and PID
14. Kinematics, SLAM, ROS 2 and Embodied AI 🚀

---

## Concept-map coverage: nothing was cut from Sprint

Consolidation reduced Sprint from 198 chapters to 142 **without removing a
concept**. Topics that lost a dedicated chapter kept a named slot:

| Topic | Sprint (overview) | Rank 2 (practice) |
|---|---|---|
| GraphQL, gRPC | M04.2 | M13.1 |
| Kubernetes | M06.9 | M13.3 |
| Infrastructure as Code | M06.13 | M13.4 |
| Event-driven, Kafka | M04.16 | M13.2 |
| Sharding, CAP, failover | M05.6–8 | M13.9 |
| Load testing | M05.9 | M13.7 |
| Release strategies | M06.12 | M13.5 |
| Diffusion / generative media | M09.15 | M14.9 |
| Reinforcement learning | M09.2, M09.11 | M14.8 |
| CNNs, RNNs, scaling laws | M09.7 | M14.1 |
| Fine-tuning ladder | M09.11 | M14.3–5 |
| Agent memory | M10.20 | M14.12 |
| Bots, voice | M10.20 | M15 |
| Multi-agent | M10.15 | M14.11 |
| Data engineering | M10.9 | M13.12 |
| 3D / Three.js | M03.10 | — |
| Native mobile | M08.1 | M12 |

## Mapping from v1 of this file

| v1 | v2 |
|---|---|
| M00 (13 ch) | M00 (10 ch) — consolidated; **+ agent permissions/sandboxing** |
| M01 (9 ch) | M01 (10 ch) — **+ Git, branches, merge conflicts, PR review** |
| M02 (24 ch) | M02 (16 ch) — consolidated |
| M03 (14 ch) | M03 (10 ch) — consolidated |
| M04 (24 ch) | M04 (17 ch) — consolidated; **+ object storage and queues moved in** |
| M05 (15 ch) | M05 (9 ch) — consolidated; stays rank 1 |
| M06 (18 ch) | M06 (13 ch) — consolidated; **+ testing, + debugging** |
| M07 (10 ch) | M07 (9 ch) — consolidated; **+ excessive agency** |
| M08 (8 ch) | M08 (5 ch) — consolidated |
| M09 (20 ch) | M09 (16 ch) — consolidated; **+ model selection, + token economics** |
| M10 (23 ch) | M10 (20 ch) — **+ ingestion/data quality, + agent-ready repos**; reframed ch. 2, 8, 14 |
| M11 Frontier | **→ M19**, moved to the end, rank 3 |
| M12 Ship MVP (6 ch) | **→ M11**, restructured as 7 scheduled checkpoints |
| M13 | → M12 |
| — | **M13 Production Engineering is new** (absorbs deferred depth) |
| M14 | → M14 |
| M15 | → M15 |
| M16, M17, M18 | → M16, M17, M18 (M18 **+ live debugging, + operational handover**) |
