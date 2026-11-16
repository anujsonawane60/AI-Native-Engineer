---
id: m06-08-containers-dockerfiles
module: m06-cloud-linux-devops-testing
slug: containers-and-dockerfiles
title: "Containers: Images, Dockerfiles, Compose and Registries"
order: 8
duration_min: 25
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

A team's canteen app works on two laptops and not on the third. The third teammate has a different version of the language runtime. Two evenings disappear into this. The fix is "install exactly this version", and it holds until the next person joins.

Then the deploy fails, because the build server has yet another version, and a library the app depends on needs a system package that nobody installed there. Nobody can install it, because they do not control the build server's operating system.

Then the demo breaks. The code is identical to what worked an hour ago. The difference is somewhere in the machine, and the machine is not written down anywhere.

Every one of these is the same problem: the project is only half described. The code is in git. The environment the code needs — the runtime version, the system packages, the paths, the user it runs as — lives in three people's memories and three different states.

### The idea

**A container image is your application plus the entire environment it needs, written down as a file and built into one sealed artefact that runs identically everywhere.**

An **image** is the sealed artefact: a read-only stack of file-system layers plus a command to run. A **container** is one running copy of an image, with its own file system, its own process list and its own network view. The recipe that produces the image is a **Dockerfile**, and it is plain text that lives in your repository next to the code — which means the environment is finally under version control alongside everything else.

### The analogy

The canteen's samosa is famous, and the recipe card is specific: this flour, this brand of oil, fried at this temperature for this long. The recipe card is the Dockerfile. Hand it to any kitchen and you get the same samosa.

The sealed tiffin box is the image. Everything needed, portioned and closed. A courier can carry it anywhere without knowing what is inside, and it arrives as it left.

A container is one opened box being eaten right now. Opening three boxes from the same batch gives you three identical meals that do not affect one another.

The wholesale market where boxes are collected and handed out is the registry: build once, push, and any kitchen in the city pulls the exact same box.

**Where the analogy breaks:** a tiffin box is genuinely sealed, and a container is not nearly as isolated as it looks. It shares the host machine's kernel, it can be given access to host folders, and anything baked into the image is readable by anyone who pulls it. The isolation is a convenience boundary, not a safe. This is precisely why a secret written into a Dockerfile is not hidden — it is published.

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

### Reading a Dockerfile

You will never write one from memory. You will read them constantly, because an agent will generate them and you have to judge whether what it produced is sane. Here is one for a typical web app, with every instruction doing one visible job.

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

Three things in that file are not obvious and are worth understanding.

**Why copy `package.json` before the source.** Each instruction's result is cached, and a layer is rebuilt only if its inputs changed. Dependencies change rarely; your source changes constantly. Copying the dependency list first means the slow install step is reused on every build where you only edited code. Reverse those two lines and every build reinstalls everything — a difference of minutes per push, every push.

**`USER node`.** Without it the container runs as root, which chapter 1 told you not to do. Agents omit this line very often.

**`node:20-alpine` is a *pinned* base.** Not `node:latest`. A floating tag means today's build and next month's build may differ, which destroys the one property you wanted.

### Compose, registries, and what you do not need yet

**Compose** describes several containers that run together — your app, a database, a cache — in one file, started with one command. Its honest value for a student team is local development: one command gives every teammate an identical database on their own machine, with no install and nothing to uninstall. It is not how you run things in production in this program.

A **registry** stores built images. Your platform almost certainly has one and uses it without telling you. You will meet it by name in chapter 12, where CI builds an image and pushes it, and in chapter 9, where a cluster pulls it.

And the honest part: **on a zero-config platform you may not need a Dockerfile at all.** The platform detects your framework and builds a container for you. Writing one anyway buys you two things — the environment becomes explicit and reviewable, and you keep the option to move providers, which is the portability chapter 6 told you to protect. Both are real. Neither is urgent on day three.

### Where it lives

Containers sit between your code and every place it runs. Chapter 5's PaaS puts your app in a container whether or not you asked. Chapter 9's Kubernetes orchestrates containers across machines, and does nothing at all without them. Chapter 12's CI builds the image and runs your tests inside it, which is how CI results become trustworthy. Chapter 10's tests run in a container so that they fail the same way on your laptop and in CI.

In M10, an agent-ready repository is substantially a repository where the environment is described in a file, because an agent cannot ask a teammate which version of the runtime to install.

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

**A secret in a Dockerfile is in the image, permanently, for anyone who pulls it.** `ENV DATABASE_PASSWORD=...` or `COPY .env .` puts the value into a layer. Deleting it in a later instruction does not remove it from the earlier layer — exactly like M01's point about a secret committed to git. Secrets are supplied at *run* time, as environment variables from the platform's secret store, never at build time. This is the single most important line in this chapter.

**A container's file system disappears when the container stops.** Anything your app writes — uploads, a SQLite file, logs — is gone on the next deploy unless it is in a volume or, better, in object storage and a managed database. Students discover this by losing their demo data the night before a checkpoint.

**Containers are not a security boundary you should lean on.** Shared kernel, and a container run with elevated privileges or a mounted host folder has reach. M07 covers this; for now, treat "it is in a container" as no reason to relax.

**Images get large, and large images are slow everywhere.** A 1.2 GB image pulls slowly on every deploy and burns build minutes and egress from chapter 7. Starting from a slim or alpine base and not copying `node_modules` or `.git` into the image — which is what a `.dockerignore` file is for — usually cuts it by most of its size.

**Your laptop's processor architecture may not match the server's.** An image built on an ARM laptop can fail to run on an x86 server with a baffling "exec format error". Worth knowing by name so you recognise it rather than debugging it from scratch.

### Verify it

An agent will happily write you a Dockerfile. Five checks, in order of how much damage they prevent.

1. **Search it for secrets.** Any `ENV` or `ARG` holding a password, token or key, and any `COPY` that could pull in `.env`, is a published secret. This is a reject, not a revision. Then check that `.dockerignore` excludes `.env`, `.git` and `node_modules`.
2. **Is the base image pinned to a specific version?** `:latest` means your build is not reproducible, which removes the entire point.
3. **Does it drop root?** A `USER` instruction should appear before `CMD`. If not, ask for it.
4. **Is the dependency file copied before the source?** If not, every build is a full reinstall. Ask why, and accept no answer other than fixing it.
5. **Ask it to explain each line, then check two claims yourself.** Build it, run it, and `curl` the port from chapter 4. A Dockerfile that has never been built is a guess, however well-formed it looks.

Red flags: `ENV` with a credential in it; `COPY .env`; `FROM node:latest`; no `USER`; `sudo` inside a Dockerfile, which is meaningless because the build already runs as root; and an agent that adds a Dockerfile to a project already deploying fine on a zero-config platform without being asked and without explaining what it buys you.

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
