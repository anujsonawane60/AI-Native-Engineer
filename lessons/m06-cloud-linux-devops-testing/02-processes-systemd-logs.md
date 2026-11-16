---
id: m06-02-processes-systemd
module: m06-cloud-linux-devops-testing
slug: processes-systemd-logs
title: "Processes, systemd, Logs and Package Management"
order: 2
duration_min: 25
is_free: false
mission: false
volatility: low
last_reviewed: 2026-11-16
---

## Key learnings
- Explain what a process is and why a server needs something to restart it
- Describe what a service manager does in one sentence
- Find the log that explains a crash instead of guessing at the cause
- Tell the difference between a system package and a project dependency
- Recognise the three states a service can be in and what each means

## Lesson

### The pain

A team gets their canteen ordering app running on a free-tier virtual machine at 11 p.m. They see it in the browser. They are thrilled. They close the terminal and go to sleep.

At 11:01 p.m. the app is gone. Closing the terminal killed the process, because the process was a child of that terminal session. Nothing was watching, nothing restarted it, and no one found out until a teammate tried the link the next morning.

They fix that, and three days later the app dies at 3 a.m. for a different reason. This time they want to know *why*, and they have no idea where to look. The terminal that showed the error is long gone. They restart it, it works, and they never learn what happened — so it happens again the following week during their demo.

### The idea

**A running program is a process; a process can die; and a server needs two things you do not have on a laptop — something that restarts the process automatically, and somewhere that keeps the error message after the process is gone.**

On Linux the first thing is a service manager, and on almost every modern distribution the service manager is called systemd. The second thing is the log. Both exist so that a machine with no human watching it keeps working, and so that when it does not, the reason survives.

### The analogy

The canteen opens at 8 a.m. whether or not the manager remembers. There is a rota pinned to the wall: who opens, what happens if they do not turn up, who covers. If the tea counter runs out of gas and shuts, someone swaps the cylinder and reopens it without the manager being asked.

That rota is systemd. It says: this counter should be open; if it closes unexpectedly, reopen it; if it fails to open five times in a row, stop trying and write it in the book.

And there is a book. Every incident gets a dated line: "3:10 a.m., cold storage tripped the fuse, reset." Nobody reads it on a good day. On a bad day it is the only thing that explains anything.

**Where the analogy breaks:** a cook notices the gas has run out and acts on judgement. systemd has no judgement whatsoever. It only knows the process exited, not whether restarting is sensible. A process that crashes because the database password is wrong will be restarted forever, failing identically every time, and the restart loop itself becomes the symptom you have to diagnose.

### Picture it

```
       systemd  (starts at boot, never exits)
          │
          │ "canteen-api should be running"
          ▼
   ┌──────────────┐   crashes    ┌──────────────┐
   │  your app    │ ───────────▶ │   exit code  │
   │  process     │              │   + message  │
   │  pid 4471    │              └──────┬───────┘
   └──────────────┘                     │
          ▲                             ▼
          │                      ┌─────────────┐
          └──── restart ─────────│  the log    │
               (automatic)       │  (kept)     │
                                 └─────────────┘
```

systemd keeps your app's process alive and restarts it when it exits, while the reason for the exit is written to a log that outlives the process itself.

### What a process actually is

A process is one running copy of a program. It has a number — a PID — that the system uses to refer to it, a user it runs as, a slice of memory, and a set of open files. When it ends it returns an exit code: `0` means it finished successfully, anything else means it failed, and that single number is what CI in chapter 12 uses to decide whether your build passed.

Three facts about processes that explain most server confusion:

A process started from a terminal usually dies with that terminal. That is the 11 p.m. failure above. This is why you hand your app to a service manager instead of running it by hand.

A process that uses too much memory is killed by the kernel without warning, and the evidence appears in the system log rather than in your app's own output. On a free-tier VM with 512 MB of RAM this is common, and the symptom — "it just disappears" — looks like a mystery until you know to look there.

Two processes cannot listen on the same port. `EADDRINUSE` means an old copy of your app is still running. The fix is finding and stopping it, not changing the port.

### Logs: the part students skip

A log is a file your program appends lines to as it runs. That is all. Its value is entirely in the fact that it persists after the thing that wrote it has gone.

Three places matter. Your application's own log, which is whatever your code prints — and on a managed platform, this is what you see in the dashboard's "Logs" tab. The service manager's log, which records every start, stop, crash and restart of your service. And the kernel log, which is where out-of-memory kills and disk failures appear.

The discipline to build now, and the one chapter 11 depends on: **when something breaks, read the log before forming a theory.** Students consistently invent a cause, ask an AI to fix the invented cause, and spend an hour on a problem that the first ten lines of the log would have named. M07 goes much deeper on structured logs and what is worth logging; for now the skill is simply going and looking.

### Package management, and the dependency it is not

Linux installs software through a package manager — `apt` on Ubuntu and Debian, which is what nearly every tutorial and cloud image you meet will use. One command fetches a program, its dependencies, and its service definition, from a trusted repository.

The distinction that matters, and that agents get wrong:

A **system package** is software the whole machine shares: the database server, the web server, the language runtime itself. Installing one is a root-level act and `sudo` is correct.

A **project dependency** is a library your product uses: a web framework, a date library. It belongs inside your project folder, is listed in your project's dependency file, and is installed by your language's own tool with no `sudo` anywhere. Your repository is the record of it; the machine is not.

Mixing these up is the most common way an agent breaks a working environment, which is why chapter 1 flagged `sudo` on a project dependency as a red flag and why chapter 8's containers largely remove the problem by giving each app its own private file system.

### Where it lives

On a managed platform — the zero-config kind in chapter 6 — you never touch systemd. The platform runs the service manager for you, restarts your app, and shows you the log in a web page. Knowing what it is doing is still what lets you read that page.

On your own VM, you write a short service file and systemd does the rest. In chapter 8 a container runs a single process and the container runtime does the restarting, which is systemd's job moved up a level. In chapter 9 Kubernetes does the same job again, across many machines. Same idea at three scales: declare what should be running, let something else keep it that way.

### Words to own

| Term | Plain meaning | Example |
|---|---|---|
| Process | One running copy of a program | Your API, pid 4471 |
| PID | The number identifying a process | `4471` |
| Exit code | A number meaning pass or fail | `0` = success |
| Service / daemon | A process meant to run forever | Your API, nginx |
| systemd | The thing that keeps services running | Starts them at boot |
| `systemctl status` | Is it running, and what it last said | Your first check |
| Log | Lines a program appends as it runs | `/var/log/app.log` |
| OOM kill | The kernel killing a memory hog | Common on 512 MB VMs |
| Package manager | Installs system software | `apt install nginx` |
| System package | Software the machine shares | PostgreSQL |
| Project dependency | A library your code imports | Listed in your repo |
| `EADDRINUSE` | Port already taken by another process | An old copy is alive |

### Tradeoffs and gotchas

**Automatic restart hides failures as well as fixing them.** A service that silently restarts twenty times an hour looks healthy from outside. The restart count is itself a metric worth watching, which is exactly the point M07 makes about alerting on the thing rather than the symptom.

**A restart loop on a configuration error is infinite.** Wrong database password, missing environment variable, bad port — systemd will restart forever and the log will contain the same twelve lines over and over. The tell is identical timestamps a few seconds apart.

**Restarting fixes the symptom and destroys the evidence.** The instinct under pressure is to restart. Copy the log first. Thirty seconds of copying saves you the second outage.

**`sudo apt install` on your laptop does not travel.** Anything you install by hand on a machine exists only on that machine. If your app needs it, it belongs in your Dockerfile or your platform configuration, or the next deploy will not have it. This is the "works on my machine" problem that chapter 8 exists to solve.

**Free-tier VMs run out of memory, not CPU.** On 512 MB, a build step is far more likely to be killed than to be slow. If a process vanishes during a build, suspect memory before anything clever.

### Verify it

When an AI explains a crash or proposes a service setup, check four things.

1. **Did it read the log, or guess?** Ask: "what in the log output tells you that?" If the answer does not quote your actual text, the diagnosis is a plausible invention. Paste the real lines and ask again.
2. **Did it propose a restart as the fix?** A restart is a mitigation. Ask what would have to be true for the problem to not recur, and whether the log supports that.
3. **Is it installing a project dependency with `sudo`?** Still the red flag from chapter 1. Project dependencies go in the project's dependency file; system packages go in the Dockerfile or platform config, never typed once into a terminal and forgotten.
4. **Does the service definition say what happens on failure?** A service file with no restart policy and no log destination is half a service. Ask what happens on the fifth consecutive crash.

One more red flag specific to this chapter: an answer that tells you to run your app with `&` at the end, or inside `screen` or `nohup`, as a production solution. Those keep a process alive past your terminal; they do not restart it when it dies, and nobody is watching.

## Walkthrough
1. In a free browser Linux environment or your platform's web shell, run `ps aux | head -20`. Every line is one process. Find the user column and notice how few run as root.
2. Start something long-running — `sleep 300 &` will do — then run `ps aux | grep sleep` and write down its PID.
3. Stop it by PID with `kill <pid>`, then list again and confirm it is gone. You have just done by hand what a service manager does automatically.
4. Run `systemctl status` with no arguments and read the top few lines: how many services are loaded, and how many have failed.
5. Pick one real service — `systemctl status cron` or `systemctl status ssh` — and read its output fully. Identify the state word (`active`, `inactive`, `failed`), the PID, and the last few log lines it prints.
6. Now read logs properly. Run `journalctl -u ssh -n 30` and look at the shape: timestamp, host, service, message. Note that these lines exist whether or not anyone was watching.
7. Deliberately cause a port clash. Start any small server on port 3000, then start a second one on the same port. Read the error, note that it names the port, and reason out loud about which process is at fault.
8. Open your deploy platform's dashboard and find its "Logs" view. Compare what you see there with the `journalctl` output. Same idea, different packaging.
9. Trigger something in your app — load a page, submit a form — and watch the line appear in the platform log. That round trip is the thing you will rely on in chapter 11 and in M07.
10. Ask your AI assistant to explain the difference between `apt install` and your language's own dependency install, using your actual project as the example, and to tell you which of your current dependencies would be lost if the machine were rebuilt.

## Talk to AI
```prompt
CONTEXT:      I am a student who directs AI rather than writing code. My team
              is building a web product that will run as a long-lived server
              process. I will deploy to a managed platform, and I also have a
              free-tier Linux VM with 512 MB of RAM to experiment on. I have
              never set up a service or read a system log.

GOAL:         Understand how a server keeps my app running without me
              watching, and learn to find the log that explains a crash
              instead of guessing at the cause.

REQUIREMENTS: 1. Explain what a process is, what a PID is, and why a process
                 started in a terminal dies when I close the terminal
              2. Explain in plain words what a service manager does, and show
                 me a minimal systemd service file for a web app, with a
                 comment on every line
              3. Explain the three states a service can be in and what the
                 practical next step is for each
              4. Show me the three logs that matter - my app's own output, the
                 service manager's record, and the kernel log - and tell me
                 which one records an out-of-memory kill
              5. Explain clearly the difference between a system package and a
                 project dependency, using my stack, and tell me which of the
                 two should never be installed with sudo

CONSTRAINTS:  Plain English, second-language reader. Free tier only. No shell
              scripting. Label every command READ-ONLY or CHANGES-THINGS.
              Assume I will not memorise flags - I want to know what to ask
              for, not how to type it.

NON-GOALS:    Do not cover writing timers, socket activation, cgroup tuning,
              or log rotation configuration. Do not give me a command cheat
              sheet.

ACCEPTANCE:   - I can explain why `node server.js &` is not a deployment
              - I can name which log to read when my app "just disappears"
              - I can look at a restart-loop log and say why restarting
                forever will not fix a wrong database password
              - I can decide, for any new dependency, whether it goes in my
                project file or in my machine configuration

ASK:          First ask me what my app is written in and where I am deploying
              it, so the service file and log commands match reality. Then
              explain. Finish by giving me four short crash scenarios and
              asking ME which log I would open first, without telling me the
              answers.
```

## Stuck? Ask AI
```prompt
My deployed app keeps stopping and starting on its own. Here are the last 40 lines of the log: [paste them exactly, including timestamps]. Tell me first whether this looks like a restart loop and what evidence in the text says so. Then give me the three most likely causes in order, and for each, the single read-only check that would confirm or rule it out. Do not propose a fix until I tell you which check I ran.
```

```prompt
Explain the difference between a system package installed with a package manager and a project dependency installed by my language's own tool. Use a canteen app as the example. Then give me six things an app might need - a database server, a date library, an image converter, a web framework, a PDF tool, a logging library - and ask me which category each belongs in. Do not give me the answers first.
```

```prompt
I keep fixing outages by restarting my app, and the problem comes back. Explain why restarting both fixes the symptom and destroys the evidence, and give me a short checklist of what to capture BEFORE I restart anything, in the order I should capture it, assuming I have two minutes and a panicking teammate.
```

## Quiz

### Q1 · L2
What is the main job of a service manager like systemd?
- (a) To compile your application before it runs
- (b) To keep declared services running and restart them when they exit ✅
- (c) To install system packages and their dependencies
- (d) To collect metrics about CPU and memory usage
> It is a supervisor. You declare that something should be running; it starts it at boot, notices when the process exits, and starts it again according to the policy you set.

### Q2 · L3
A student runs `node server.js &`, sees the site working, and closes the terminal. The site goes down. What does this demonstrate?
- (a) The app had a bug that only appears after a few seconds
- (b) The process was tied to that terminal session and nothing was supervising it ✅
- (c) The port was released by the operating system
- (d) The app needed `sudo` to stay running
> `&` backgrounds a process within the session; it does not hand it to a supervisor. Nothing restarted it, and nothing was ever going to. This is the gap a service manager fills.

### Q3 · L4
Your service log shows the same twelve lines repeating every four seconds, with the error `password authentication failed`. What is happening, and what will automatic restart achieve?
- (a) A memory leak; restarting will clear it
- (b) A restart loop on a configuration error; restarting will never fix it ✅
- (c) Normal startup behaviour; the log is noisy
- (d) A port clash; restarting will eventually find a free port
> systemd has no judgement — it only knows the process exited. A wrong credential produces an identical failure every time, forever. The tell is identical messages a few seconds apart.

### Q4 · L4
On a 512 MB free-tier VM, your build step disappears mid-run with no error in your app's output. Where does the evidence most likely live?
- (a) Your application log, under a different log level
- (b) The kernel log, as an out-of-memory kill ✅
- (c) The package manager's history file
- (d) Nowhere — processes killed abruptly leave no trace
> When memory runs out the kernel kills a process outright; your app never gets to write anything. The record of the kill is in the system and kernel log, not in your output.

### Q5 · L5
An agent diagnoses your outage confidently as "a database connection pool exhaustion issue" without having seen any of your logs. What is the right response?
- (a) Accept it — pool exhaustion is a common cause
- (b) Ask it to implement the fix and see whether the outage recurs
- (c) Ask what in the log output supports that, then paste the actual lines and ask again ✅
- (d) Restart the service and check whether the diagnosis still applies
> A diagnosis that cites no evidence from your system is a plausible invention. Demanding that a claim point at real text is the core verification habit of this module and of chapter 11.

## Go deeper
- [Ubuntu Server documentation: service management and logging](https://ubuntu.com/server/docs)
- [systemd for administrators, the readable introduction](https://www.freedesktop.org/wiki/Software/systemd/)
