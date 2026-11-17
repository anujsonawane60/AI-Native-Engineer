---
id: m06-08-containers-dockerfiles
module: m06-cloud-linux-devops-testing
slug: containers-and-dockerfiles
title: "Containers: Images, Dockerfiles, Compose and Registries"
order: 8
duration_min: 15
is_free: false
mission: false
volatility: low
last_reviewed: 2026-11-16
---

## Key learnings
- Explain the difference between an image and a container in one sentence each
- Read a Dockerfile line by line and say what each instruction contributes
- Say why "works on my machine" stops being a sentence anyone needs
- Recognise a Dockerfile that bakes a secret into the image
- Describe what Compose and a registry each add, and when you need neither

## Lesson

### The pain

A team's canteen app works on two laptops and not the third: that teammate has a different version of the language runtime. Then the deploy fails: the build server has yet another version and needs a system package nobody there can install. Then the demo breaks with identical code: the difference is in a machine that is written down nowhere.

All three are one problem: the code is in git, but the environment it needs — runtime version, system packages, paths, the user it runs as — lives in three people's memories.

### The idea

**A container image is your application plus the entire environment it needs, written down as a file and built into one sealed artefact that runs identically everywhere.**

An **image** is the sealed artefact: a read-only stack of file-system layers plus a command to run. A **container** is one running copy of it, with its own file system, process list and network view. The recipe is a **Dockerfile**, plain text in your repository next to the code — which finally puts the environment under version control.

### The analogy

The canteen's samosa recipe card is specific: this flour, this oil, this temperature, this long. The card is the Dockerfile — hand it to any kitchen and you get the same samosa. The sealed tiffin box is the image: everything needed, portioned and closed. Three boxes from one batch are three identical meals that do not affect each other — three containers. The market where boxes are handed out is the registry.

**Where the analogy breaks:** a tiffin box is sealed; a container is not nearly as isolated as it looks. It shares the host's kernel, can be given host folders, and anything baked into the image is readable by anyone who pulls it. A secret written into a Dockerfile is not hidden — it is published.

### Picture it

```
  Dockerfile  ──build──▶   IMAGE   ──push──▶   REGISTRY
  (text, in                (sealed              (shared
   your repo)               layers)              store)
                              │                     │
                              │                   pull
                              ▼                     ▼
                         CONTAINER            CONTAINER
                         on your laptop       on the server
                         ┌──────────────┐   ┌──────────────┐
                         │ your app     │   │ your app     │
                         │ runtime      │   │ runtime      │
                         │ system libs  │   │ system libs  │
                         └──────────────┘   └──────────────┘
                              identical, both places
```

A text recipe builds into a sealed image, the image is pushed to a shared registry, and anything that pulls it gets byte-identical containers wherever they run.

```
FROM node:20-alpine            start from a known base image
WORKDIR /app                   all later paths are relative to here
COPY package*.json ./          copy ONLY the dependency list first
RUN npm ci --omit=dev          install deps - cached if list unchanged
COPY . .                       now copy the rest of the source
RUN npm run build              produce the production build
EXPOSE 3000                    document the port the app listens on
USER node                      stop running as root
CMD ["node", "server.js"]      what to run when a container starts
```

Each line in a Dockerfile builds one layer of the image, and the order of the lines is what makes rebuilds fast or slow.

### Where it lives

Containers sit between your code and every place it runs. Chapter 5's PaaS puts your app in one whether or not you asked; chapter 9's Kubernetes orchestrates containers across machines and does nothing without them; chapter 12's CI builds the image and runs your tests inside it, which is how CI results become trustworthy; and M10's agent-ready repository is largely one where the environment is in a file.

**Compose** describes several containers that run together in one file and one command; its value here is local development, not production. A **registry** stores built images; your platform probably uses one without telling you. On a zero-config platform you may not need a Dockerfile at all; writing one anyway makes the environment explicit and keeps your exit open.

### Words to own

| Term | Plain meaning | Example |
|---|---|---|
| Image | A sealed, read-only app plus environment | Built from a Dockerfile |
| Container | One running copy of an image | Three from one image |
| Dockerfile | The text recipe that builds an image | Lives in your repo |
| Base image | What you start from | `node:20-alpine` |
| Layer | The result of one instruction, cached | Why order matters |
| Tag | A version label on an image | `:20-alpine`, not `:latest` |
| Registry | Where images are stored and shared | CI pushes, servers pull |
| `COPY` | Put files from your repo into the image | `COPY . .` |
| `CMD` | What runs when the container starts | `node server.js` |
| Volume | Host storage mounted into a container | Keeps data past restart |
| Compose | Several containers described together | App plus local database |
| Ephemeral | Lost when the container stops | Anything not in a volume |

### Tradeoffs and gotchas

- **A secret in a Dockerfile is in the image, permanently, for anyone who pulls it.** `ENV DATABASE_PASSWORD=...` or `COPY .env .` writes the value into a layer, and deleting it later does not remove the earlier layer. Secrets are supplied at *run* time, from the platform's secret store, never at build time.
- **A container's file system disappears when the container stops.** Uploads, a SQLite file, logs: gone on the next deploy unless they live in a volume or in object storage and a managed database.
- **Large images are slow everywhere.** A 1.2 GB image pulls slowly on every deploy and burns build minutes and egress. A slim base plus a `.dockerignore` excluding `node_modules` and `.git` removes most of that.
- **Your laptop's architecture may not match the server's.** An ARM-built image can fail on an x86 server with "exec format error".

### Verify it

1. **Search it for secrets.** Any `ENV` or `ARG` holding a password or key, and any `COPY` that could pull in `.env`, is a published secret — a reject, not a revision. Check `.dockerignore` excludes `.env` and `.git`.
2. **Is the base image pinned to a version?** `:latest` means your build is not reproducible, which removes the entire point.
3. **Does it drop root?** A `USER` instruction before `CMD`. Agents omit it very often.
4. **Is the dependency file copied before the source?** Layers are cached, so copying the dependency list first reuses the slow install on code-only builds. Reversed, every push reinstalls everything.
5. **Build it and run it,** then `curl` the port from chapter 4. A Dockerfile never built is a guess.

## Walkthrough
1. Find out whether your project already runs in a container. Open your platform's build log and look for a base image name or a build pack. Write down what you find.
2. Ask your AI assistant for a Dockerfile for your exact stack, and require a one-line comment on every instruction explaining what it contributes.
3. Read it line by line against the five checks in "Verify it" before building anything. Write down which checks passed.
4. Specifically search it for secrets: any `ENV` or `ARG` with a credential, and any `COPY` that could include `.env`. If you find one, have it rewritten and note what would have leaked.
5. Create a `.dockerignore` containing at least `.env`, `.git` and your dependency folder, and have the AI explain what each exclusion saves you.
6. Build the image in a free browser dev environment that provides a container runtime. Note the build time and the resulting image size.
7. Run a container from it, map the port, and `curl -i` the app inside it. You have now proved the recipe produces a working app, not just a valid file.
8. Change one line of your source, rebuild, and watch which layers are reused from cache. Then move the `COPY . .` line above the dependency install, rebuild, and compare the time. Put the line back.
9. Write a file inside the running container, stop it, start a new one from the same image, and look for the file. Its absence is the ephemeral lesson, learned cheaply instead of the night before a demo.
10. Pass a secret correctly: run the container with the value supplied as a run-time environment variable rather than in the image, and confirm the app reads it.
11. Finally, write three lines in your repository notes: what your image starts from, what it runs as, and how your app receives its secrets.

## Talk to AI
```prompt
CONTEXT:      I am a student on an AI-native engineering program. I direct AI
              rather than writing code, and I need to be able to READ a
              Dockerfile and judge it, not write one from memory. My team of
              three is building [describe your product in two sentences].
              Stack: [language, framework, database]. We deploy on a
              zero-config platform's free tier. Our secrets are currently in
              a local .env file and in the platform's environment variables.
              One teammate's machine already behaves differently from the
              other two.

GOAL:         Get a correct, minimal Dockerfile for my project that I fully
              understand line by line, and learn the specific things that make
              a Dockerfile wrong or dangerous.

REQUIREMENTS: 1. Explain the difference between an image and a container, and
                 what a registry is for, in plain words
              2. Produce a Dockerfile for my stack with a comment on EVERY
                 line saying what that line contributes
              3. Explain layer caching and why the dependency file is copied
                 before the source, including roughly what the wrong order
                 costs me per build
              4. Pin the base image to a specific version and explain what
                 would go wrong with a floating tag
              5. Show me how my secrets should reach the running app, and
                 explain concretely why putting them in the Dockerfile
                 publishes them to anyone who pulls the image
              6. Give me a .dockerignore for my stack and say what each entry
                 saves me in size or risk

CONSTRAINTS:  Free tier only. Plain English, second-language reader. The
              container must not run as root. The image should be as small as
              is reasonable - tell me its expected size. Do not use any
              paid base image or private registry. Label every command I
              should run as READ-ONLY or CHANGES-THINGS.

NON-GOALS:    Do not introduce Kubernetes, Helm, or a multi-service Compose
              production setup. Do not add a Dockerfile-based deploy pipeline
              yet - that is a later chapter. Do not write any application code.

ACCEPTANCE:   - I can explain every line of the Dockerfile in my own words
              - There is no secret, token or password anywhere in the file
              - The base image has a specific version tag, not latest
              - A USER instruction appears before CMD
              - I can state what happens to a file my app writes inside the
                container after a redeploy
              - I can describe how to pass a secret at run time instead of
                build time

ASK:          First ask me which language, framework and package manager I
              actually use, and whether my app needs any system package
              beyond the runtime. Then explain the concepts before producing
              the file. After the file, give me a five-point checklist I can
              use to review any future Dockerfile an AI writes for me.
```

## Stuck? Ask AI
```prompt
Here is a Dockerfile: [paste it]. Do not improve it yet. First explain what every single line does and what it adds to the image. Then review it against these five questions and answer each explicitly: does it contain or risk including any secret, is the base image pinned, does it drop root before running, is the dependency file copied before the source, and will the image be unnecessarily large. Only then suggest changes, in priority order.
```

```prompt
Explain why a secret written into a Dockerfile is not hidden, even if a later instruction deletes the file. Use the way image layers work to explain it. Then tell me the correct way to pass a database password to a container at run time on a zero-config platform, and what I should do today if I have already pushed an image containing a real credential.
```

```prompt
My app worked in a container and now the data is gone after a redeploy. Explain what a container's file system does when the container stops, which of my data is affected, and the three places that data should live instead. Then ask me what my app currently writes to disk so you can tell me which items are at risk.
```

## Quiz

### Q1 · L2
What is the difference between an image and a container?
- (a) An image runs; a container is the file that describes it
- (b) An image is the sealed read-only artefact; a container is one running copy of it ✅
- (c) They are the same thing with different names on different platforms
- (d) An image holds the code; a container holds the data
> The Dockerfile is the recipe, the image is the sealed result, and a container is one running instance. Three containers from one image are three independent copies.

### Q2 · L4
A Dockerfile contains `ENV DB_PASSWORD=canteen123`, and a later instruction unsets it. Is the password safe?
- (a) Yes — the final image no longer has the variable set
- (b) No — the value is in an earlier layer and is readable by anyone who pulls the image ✅
- (c) Yes, provided the registry is private
- (d) Only if the base image is pinned
> Layers are additive history, exactly like git commits. Removing a value later does not remove it from the layer that introduced it. Secrets are supplied at run time from the platform's secret store, never baked in.

### Q3 · L4
Why does a well-written Dockerfile copy the dependency list before copying the source code?
- (a) Because the build fails otherwise
- (b) Because dependencies must be installed as root
- (c) Because layers are cached, so the slow install is reused on every build where only the source changed ✅
- (d) Because it makes the final image smaller
> Each instruction's layer is rebuilt only when its inputs change. Dependencies change rarely and source changes constantly, so the order saves minutes on every push.

### Q4 · L3
Your app writes uploaded photos to a folder inside the container. What happens on the next deploy?
- (a) They are migrated to the new container automatically
- (b) They are gone, because a container's file system is ephemeral ✅
- (c) They remain, because the image is rebuilt from the same source
- (d) They are moved to the registry with the image
> A new container starts from the image, with nothing added since the build. Uploads belong in object storage, rows in a managed database, and neither inside the container.

### Q5 · L5
An agent adds a Dockerfile to a project that already deploys fine on a zero-config platform, unprompted. What is the right response?
- (a) Accept it — containerising is always an improvement
- (b) Reject it outright, since the platform already handles builds
- (c) Ask what it buys you now, weigh the explicit environment and provider portability against the new thing to maintain, then decide ✅
- (d) Accept it only if the base image is pinned
> There are two genuine benefits — the environment becomes explicit and reviewable, and you keep the option to move providers — and a real cost in something new to maintain. The failure is accepting unrequested scope without that comparison.

## Go deeper
- [Docker's official Dockerfile best-practices guide](https://docs.docker.com/build/building/best-practices/)
- [Play with Docker: a free in-browser container playground](https://labs.play-with-docker.com/)
