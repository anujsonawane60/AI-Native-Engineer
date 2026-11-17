---
id: m00-04-inside-the-machine
module: m00-orientation
slug: inside-the-machine
title: "Inside the Machine: CPU, RAM, Disk, OS and Processes"
order: 4
duration_min: 15
is_free: true
mission: false
volatility: low
last_reviewed: 2026-11-16
---

## Key learnings
- Say what the CPU, RAM and disk each do, and how they differ in speed
- Explain why RAM forgetting everything on restart matters to your app
- Describe what an operating system actually does for you
- Tell the difference between a program and a process
- Read a server's resource numbers and say which one is the problem

## Lesson

### The pain

Your deployed app was fine for three weeks. This morning it is slow; by lunchtime it stops responding. You restart it and it works again — for four hours.

The explanation always has the same shape: something ran out — memory, CPU, disk or connections. Without those four words you can only restart and pray; with them you can read a dashboard and say "it is memory, growing in a straight line."

### The idea

**A computer has one part that thinks fast and forgets instantly (CPU and RAM), one part that remembers slowly (disk), and a manager that shares them between competing programs (the operating system).**

Every performance problem you will ever see is one of those resources exhausted, or one waiting on another.

### The analogy

The canteen kitchen is a surprisingly exact model. **The CPU is the cook**: fast, but only a few things at a time — four cooks cook four dishes at once, which is what "four cores" means. **RAM is the counter space**: ingredients within arm's reach, instant to grab, wiped at closing time every night. **The disk is the storeroom**: enormous, cheap, survives the night — but a cook who walks there per ingredient makes the kitchen crawl. **The OS is the kitchen manager**: who cooks what, who gets counter space, who waits.

**Where the analogy breaks:** a cook out of counter space puts something back in the storeroom and continues, slightly slower. A computer out of RAM may do that — swapping, which is brutally slow — or it may simply kill your program. Running out of memory is often fatal, not merely slow.

### Picture it

```
                       speed              survives restart?
  ┌──────────────┐
  │     CPU      │   ~1 ns per op          n/a
  │  (the cook)  │   does the work
  └──────┬───────┘
         │  needs data within arm's reach
         ▼
  ┌──────────────┐
  │     RAM      │   ~100 ns               NO — wiped
  │  (counter)   │   8 GB typical          
  └──────┬───────┘
         │  must fetch from storage when not in RAM
         ▼
  ┌──────────────┐
  │   DISK/SSD   │   ~100,000 ns           YES
  │ (storeroom)  │   512 GB typical
  └──────────────┘
```

Reading from RAM is about a thousand times faster than reading from disk, and disk is the only one that remembers — the trade that makes databases, caches and files separate ideas.

### Where it lives

Two numbers to hold on to: **RAM is ~1000× faster than disk, and RAM forgets.** Almost everything later follows from them.

- **Databases and indexes** (M04) put things on disk so they survive, without a full storeroom search per question.
- A **cache** (M05) keeps an answer in RAM, accepting that a restart loses it because it can be rebuilt.
- **Stateless servers** (M05) exist because RAM forgets. A cart held only in RAM dies with the process; moving it to the data box lets you run twenty servers.
- **Containers** (M06) package a program with what it needs, so the OS underneath stops mattering.

One distinction error messages depend on: a **program** is the inert file on disk, a **process** is that program running — in RAM, given CPU time, holding files and connections open, identified by a PID. One program can be many processes, which is why a frozen browser tab does not take the browser down, and why "restart it" works: killing the process discards everything in RAM, leak included.

### Words to own

| Term | Plain meaning | Example |
|---|---|---|
| CPU | The part that executes instructions | "CPU is at 95%" |
| Core | One independent worker inside the CPU | A 4-core server does 4 things at once |
| RAM / memory | Fast temporary working space, wiped on restart | "The process is using 2 GB" |
| Disk / SSD | Slow permanent storage | Where the database file lives |
| OS / kernel | The manager sharing hardware between programs | Linux, Windows, macOS |
| Process | A running instance of a program, with a PID | Your server, running |
| Thread | A strand of work inside one process | Handling one request |
| Swap | Using disk as emergency RAM; very slow | The symptom before a crash |
| OOM | Out Of Memory; the OS kills the process | "OOMKilled" in a deploy log |
| Memory leak | Memory taken and never released | Usage rising in a straight line |

### Tradeoffs and gotchas

**The four classic exhaustions**, each with a signature:

| Symptom | Resource | What it usually means |
|---|---|---|
| CPU pinned at 100%, requests queue | CPU | Doing too much work per request, or an infinite loop |
| Memory rising steadily, then the process dies | RAM | A memory leak; restarting only delays it |
| Everything slow, disk light constant | Disk / swap | Out of RAM and swapping, or slow queries hitting disk |
| "Too many connections" | Connections | Each request opened a database connection and never closed it |

**More memory is not a fix for a leak.** Doubling RAM buys twice the time before the same crash. Legitimate at 2 am to stop the bleeding; not a fix, and AI often proposes it as one.

**CPU at 100% is not automatically bad.** A server doing heavy work should use the CPU you pay for; idle capacity is wasted money. The problem is CPU at 100% *while requests are waiting*.

**"It works on my machine" is usually a resource story.** Your laptop has 16 GB and no other users; the free-tier server has 512 MB and runs your database too. The code is identical; the counter space is not.

### Verify it

Three questions catch most resource problems early:

1. **"How much memory does this hold, and when is it released?"** Anything loading a whole file, table or AI model needs a real number attached. "It loads the CSV into memory" is fine for 200 rows and fatal for 20 million.
2. **"Does this work if the process restarts right now?"** If the answer is "the user loses their session" or "the queue is lost", that state is in RAM and belongs in the data box.
3. **"What is opened here, and where is it closed?"** Files and connections are finite. AI frequently opens them in a loop and closes none — perfect in testing, broken at 100 users.

Red flags: "we'll keep it in memory for speed" (fine for a cache, fatal for anything you cannot lose) and "load all the records then filter".

## Walkthrough
1. Open your machine's resource monitor. On Windows press Ctrl+Shift+Esc for Task Manager; on macOS open Activity Monitor; on Linux run `top` or `htop` in a terminal.
2. Find the total RAM and how much is currently in use. Note the number — this is your counter space, and the gap between them is what a new program has to work with.
3. Sort the process list by memory. Look at the top entry: it is probably your browser. Note how one program appears as many processes.
4. Sort by CPU instead. On an idle machine almost everything should be near zero. Anything busy while you are doing nothing is worth being curious about.
5. Count your CPU cores (Task Manager's Performance tab, or `nproc` on Linux). This is how many things your machine can genuinely do at once.
6. Open ten browser tabs with heavy sites, then watch memory climb. Close them and watch it fall. You have just watched allocation and release — the thing a memory leak fails to do.
7. Check your free disk space. Then consider: a server that fills its disk with log files stops working, and this is one of the most common real-world outages there is.
8. Write down four numbers for your machine: cores, total RAM, RAM in use, free disk. These are the four numbers you will look at on a server dashboard for the rest of your career.

## Talk to AI
```prompt
I want to understand the resource behaviour of something before I build it. The thing is: [describe the feature, e.g. "an endpoint that generates a PDF report of all orders" or "a background job that resizes uploaded images"].

Answer these, with rough numbers rather than vague adjectives:
1. How much memory will this hold at once, and when is it released?
2. Is this CPU-heavy, memory-heavy, disk-heavy, or mostly waiting on the network?
3. What happens if the process restarts in the middle of it?
4. What does it open — files, connections — and where is each one closed?
5. At roughly what scale does this approach stop working, and what is the first thing that breaks?

Then give me the simplest change that makes it survive 100× the load. Do not write code; I want to understand the shape of the problem first.
```

## Stuck? Ask AI
```prompt
Explain the difference between RAM and disk to me as if I were twelve, then explain why that difference is the reason caches and databases are different things. Then ask me to explain it back to you and correct what I get wrong.
```

```prompt
I am looking at a server dashboard and I do not know which number matters. Give me five scenarios — different combinations of CPU, memory, disk and connection numbers — and for each one ask me what I think is wrong. Do not give the answer until I guess.
```

```prompt
Explain what a memory leak is without using code. Then explain why adding more memory does not fix it, and what actually does. Use an everyday analogy that is not about computers.
```

## Quiz

### Q1 · L2
Roughly how much faster is reading from RAM than reading from a disk?
- (a) About twice as fast
- (b) About ten times as fast
- (c) About a thousand times as fast ✅
- (d) They are about the same on modern hardware
> RAM access is on the order of 100 nanoseconds; disk access is on the order of 100,000. That ~1000× gap is the entire reason caching exists.

### Q2 · L3
A server stores each student's shopping cart in RAM only. What is the consequence?
- (a) Carts load slowly because RAM is slow
- (b) Carts are lost when the process restarts, and a second server would not see them ✅
- (c) Carts take up disk space that must be cleaned up
- (d) Nothing — this is the standard approach
> RAM is wiped on restart and is not shared between processes. This is exactly why servers are kept stateless and shared state lives in the data box.

### Q3 · L4
A process's memory usage climbs in a straight line for six hours and then the process is killed. Restarting fixes it for another six hours. What is happening, and what is the real fix?
- (a) The CPU is overloaded; add more cores
- (b) A memory leak; find and release what is never freed ✅
- (c) The disk is full; delete old logs
- (d) Too many database connections; raise the connection limit
> A steady climb ending in a kill is the signature of a leak. Adding memory or restarting on a schedule only changes how long it takes to die.

### Q4 · L3
Which statement about a program and a process is correct?
- (a) They are two words for the same thing
- (b) A process is the file on disk; a program is it running
- (c) A program is the file on disk; a process is an instance of it running ✅
- (d) A program can only ever become one process
> The program is inert on disk. A process is a running instance with memory, CPU time and a PID — and one program can run as many processes, which is why one browser tab can crash alone.

### Q5 · L4
An AI assistant suggests: "Load all the orders into memory, then filter for today's." When is this a problem?
- (a) Never; filtering in memory is always fastest
- (b) Only if the server has fewer than 4 cores
- (c) As soon as the orders table is large enough not to fit in available RAM ✅
- (d) Only on Windows servers
> It works fine on test data and fails on real data. The database should do the filtering and return only what is needed, rather than the process holding the whole table.

## Go deeper
- [Latency numbers every programmer should know](https://gist.github.com/jboner/2841832)
- [What every programmer should know about memory (introduction only)](https://lwn.net/Articles/250967/)
