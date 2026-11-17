---
id: m06-01-linux-filesystem
module: m06-cloud-linux-devops-testing
slug: linux-filesystem-permissions
title: "Why Servers Run Linux; the File System, Users and Permissions"
order: 1
duration_min: 15
is_free: false
mission: false
volatility: low
last_reviewed: 2026-11-16
---

## Key learnings
- Explain why almost every server your product touches runs Linux
- Read a Linux path and say what kind of thing lives there
- Read `rwx` permissions and ownership on a file and say who can do what
- Recognise `sudo` as a signal that something is wrong with the plan
- Name the case-sensitivity trap that breaks a first deploy

## Lesson

### The pain

A team's canteen ordering app works on a laptop. The deploy fails with `Module not found: ./components/MenuCard` — and the file is right there. On disk it is `menuCard.tsx`; Windows and macOS do not care about that difference, Linux does, and the build server is Linux.

An hour later they hit `EACCES: permission denied, open '/app/uploads/photo.jpg'`, and the agent suggests `sudo chmod 777 /app/uploads` — the error goes away and every uploaded file is now writable by anything on that machine. Both failures are the same failure: nobody knows what the server actually is.

### The idea

**A Linux server is one big tree of files, and every file has an owner and a short list of who is allowed to read it, change it or run it.**

Processes, devices, configuration and logs are all files. When something breaks, the question is almost always "which file, owned by whom, and was the thing touching it allowed to?" Linux runs servers because it is free to copy a thousand times, needs no screen, starts in seconds, and underpins every cloud provider and container image you will meet.

### The analogy

The canteen has a fixed layout: store room, kitchen, serving counter, ledger drawer. A new cook finds the salt on day one because the layout is a convention, not a choice. Each area has an entry rule — students reach the counter, cooks reach the kitchen, only the manager opens the ledger drawer. The manager has a master key and does not lend it to a cook who "just needs to get something", because the cost is everything else they could then reach.

`sudo` is the master key. The layout is the file system. The entry rules are permissions.

**Where the analogy breaks:** a cook who finds the store room locked knocks and asks, and a human decides. Linux does not negotiate. It returns `Permission denied` and stops, with no hint about which of the three permissions was missing.

### Picture it

```
  /                      the root of everything
  ├── home/
  │   └── deploy/        a human-ish user's own files
  ├── etc/               configuration files, text, no data
  ├── var/
  │   ├── log/           logs - where you look when it broke
  │   └── www/           files served to the web
  ├── usr/bin/           installed programs
  ├── tmp/               scratch space, wiped on reboot
  └── app/               your product, by convention in containers
```

Every path starts at `/` and walks down this tree; there are no drive letters, and these top-level names are a convention on every server you will touch.

Listing a file in long form gives this:

```
  -rw-r-----  1 deploy  app   412  Nov 16 09:14  .env
   │└┬┘└┬┘└┬┘    └──┬─┘ └┬┘
   │ │  │  │        │    └── group that owns it
   │ │  │  │        └─────── user that owns it
   │ │  │  └──────────────── everyone else: nothing
   │ │  └─────────────────── group: read only
   │ └────────────────────── owner: read + write
   └──────────────────────── a regular file (d = directory)
```

The first character is the file type, then three groups of three say what the owner, the owning group and everyone else may do: `r` read, `w` change or delete, `x` run as a program. On a directory `x` means "allowed to walk into it", which is why a readable folder you cannot enter is a real and confusing state.

### Where it lives

You will rarely log into a Linux box here, but Linux sits underneath the next twelve chapters:

- Your deploy platform builds your app inside a Linux container
- The CI runner in chapter 12 is a fresh Linux machine that lives ninety seconds
- Every Dockerfile in chapter 8 copies files to paths like `/app`
- Logs in M07 are files under `/var/log` before anything collects them
- When M05 scales you to three servers, it is three copies of this tree

### Words to own

| Term | Plain meaning | Example |
|---|---|---|
| Root (`/`) | The top of the file tree | `/var/log/app.log` |
| Root (the user) | The all-powerful account | Can delete anything |
| `sudo` | Do this one thing as root | `sudo apt install nginx` |
| Home directory | A user's own folder | `/home/deploy` |
| `/etc` | Configuration, as plain text | nginx's site config |
| `/var/log` | Where logs are written | First place to look |
| Owner | The user the file belongs to | `deploy` |
| Permissions | Who may read, write, execute | `rw-r-----` |
| `chmod` | Change permissions | `chmod 600 .env` |
| `chown` | Change the owner | `chown deploy .env` |
| Case-sensitive | `Menu.tsx` ≠ `menu.tsx` | Breaks your first deploy |
| Absolute path | Starts at `/`, unambiguous | `/app/src/index.ts` |

### Tradeoffs and gotchas

- **`chmod 777` is never the right answer.** It means "anyone on this machine may read, change or run this": the symptom goes, the hole stays. The real fix changes the *owner*, or runs the process as the user that already owns the file.
- **Do not run your app as root.** It works, and any bug in your code then has full control of the machine. Deploy platforms use an unprivileged user by default.
- **Case sensitivity bites exactly once and costs you an evening.** Pick one file-naming convention on day one, put it in your M10 rules file, and check capitalisation first when a deploy cannot find a file you can see.
- **`.env` should be `600`.** Owner-only read and write. M01 kept it out of git; this keeps it away from other users on the same machine.

### Verify it

You will meet Linux mostly through an AI explaining an error or proposing a command.

1. **Does the command need `sudo`, and should it?** A *system* package legitimately needs it; your *project's* dependencies never do. `sudo` inside your own project folder means the agent is routing around a permissions problem and leaving root-owned files you cannot edit.
2. **Does it widen permissions or fix ownership?** Ask for the owner fix instead of the permission-bit fix; there almost always is one.
3. **Is the path absolute and plausible?** A command writing to `/etc` or `/usr` to configure *your app* is misplaced.
4. **Did it check the obvious thing first?** For "module not found" on a file that exists, the first hypothesis is capitalisation, not caches.

Red flags: `sudo` for a project dependency; `chmod 777` as a fix; instructions that assume you are root; a confident claim about a file's permissions without having listed the file first.

## Walkthrough
1. Open a free browser-based Linux environment — a cloud dev workspace or your deploy platform's web shell. You need a terminal, not a VM of your own.
2. Run `pwd` and read the answer out loud. That absolute path is where you are standing in the tree from the diagram above.
3. Run `ls /` and compare what you see with the diagram. Name three directories you recognise and one you do not.
4. Run `ls -l /var/log`. Read one line fully: file type, owner, group, size, date, name. Say in plain words who can read it.
5. Create a throwaway file with `echo "secret" > scratch.txt`, then `ls -l scratch.txt`. Note the permissions it got by default.
6. Run `chmod 600 scratch.txt` and list it again. Describe exactly what changed in the nine letters, and who lost access.
7. Try to read a file you do not own: `cat /etc/shadow`. Read the error. This is what `Permission denied` looks like, and it is the error you will see in deploy logs.
8. Now the case-sensitivity drill. Run `touch Menu.txt`, then `cat menu.txt`. Note that Linux treats these as two different names, then write that sentence into your team's notes.
9. Ask your AI assistant: "explain the output of `ls -l` line by line for this file, and tell me which permission a web server would need to serve it." Compare its answer to what you worked out yourself.
10. Finally, ask it for a command that fixes a permission error by changing ownership rather than by using `chmod 777`, and have it explain why that is safer.

## Talk to AI
```prompt
CONTEXT:      I am a student learning to direct AI rather than write code. My
              team is building a web product that will be deployed to a Linux
              server by a hosting platform. I develop on Windows/macOS. I have
              never administered a Linux machine and I will not be memorising
              commands.

GOAL:         Understand enough about the Linux file system, users and
              permissions to read a deploy error and judge whether a fix you
              propose is safe.

REQUIREMENTS: 1. Explain the standard top-level directories (/etc, /var/log,
                 /home, /tmp, /usr/bin, /app) and what my product's files,
                 configuration, logs and secrets each belong in
              2. Teach me to read one line of `ls -l` output completely -
                 type, owner, group, and the nine permission letters
              3. Explain what r, w and x mean differently on a FILE versus on
                 a DIRECTORY, with a concrete example of each
              4. Explain what sudo is, give me two examples where it is
                 correct and two where its presence means something is wrong
              5. Explain the case-sensitivity difference between my laptop and
                 a Linux server, and the single most common deploy failure it
                 causes

CONSTRAINTS:  Plain English, second-language reader. No shell scripting. Do
              not teach me vi. Assume I will run at most five commands.
              For every command you show me, label it READ-ONLY or
              CHANGES-THINGS, and for anything that changes things, say what
              it would take to undo.

NON-GOALS:    Do not cover user management, groups, SELinux, ACLs, or
              networking. Do not give me a cheat sheet to memorise.

ACCEPTANCE:   - I can look at `-rw-r-----  deploy app .env` and say in one
                sentence who can read it and who cannot
              - I can name the directory I would look in first when my
                deployed app stops responding
              - I can explain why `chmod 777` makes an error disappear
                without fixing the cause
              - I can state the ownership-based alternative to chmod 777

ASK:          Before explaining anything, ask me two questions about what I
              am actually deploying so your examples use my real paths. Then
              explain. At the end, give me three short scenarios and ask ME
              who has access - do not give me the answers first.
```

## Stuck? Ask AI
```prompt
I got this error from my deploy and I do not understand it: [paste the exact error text, including the file path]. Explain in plain English what Linux was trying to do, which file and which user were involved, and the three most likely causes in order of probability. Do not give me a fix yet - first tell me what command I should run to find out which cause it actually is, and whether that command is read-only.
```

```prompt
Explain the difference between a case-sensitive and a case-insensitive file system to someone who has only used Windows. Then give me five pairs of file names and import statements and ask me which pairs would break on a Linux server. Do not tell me the answers until I have guessed all five.
```

```prompt
An AI assistant told me to run `sudo chmod -R 777` on my project folder to fix a permission error. Explain what that command actually does, what it exposes, and why it is the wrong fix. Then give me the correct approach based on file ownership, and tell me how I would undo the 777 if I had already run it.
```

## Quiz

### Q1 · L2
On a Linux server, where would you look first for the output of a web app that has stopped responding?
- (a) `/etc`
- (b) `/var/log` ✅
- (c) `/usr/bin`
- (d) `/home`
> `/etc` holds configuration as plain text, `/usr/bin` holds installed programs, and `/home` holds user files. Logs live under `/var/log` by convention on every Linux distribution.

### Q2 · L3
Your deploy fails with `Module not found: ./components/MenuCard`, but the file is clearly in the folder. What is the first thing to check?
- (a) Whether the dependency is missing from the package file
- (b) Whether the build cache needs clearing
- (c) Whether the file name's capitalisation matches the import exactly ✅
- (d) Whether the file has execute permission
> Your laptop's file system is case-insensitive; the Linux build server is not. `menuCard.tsx` and `MenuCard` are different names there, and this is the most common first-deploy failure in the course.

### Q3 · L4
An agent proposes `sudo chmod -R 777 /app/uploads` to fix an upload failure. What is wrong with accepting it?
- (a) Nothing — it is the standard fix for upload permissions
- (b) It removes the error but grants read, write and execute to every user and process on the machine ✅
- (c) It only works on directories, not on the files inside them
- (d) It will be reverted on the next deploy, so it is pointless
> `777` is the widest possible permission. The right fix is almost always to change the file's owner so the process that needs to write already owns the directory, rather than opening it to everything.

### Q4 · L4
Which of these uses of `sudo` is a signal that something is wrong with the plan?
- (a) `sudo apt install postgresql` on a fresh VM
- (b) `sudo npm install` inside your own project folder ✅
- (c) `sudo systemctl restart nginx`
- (d) `sudo ufw allow 443`
> Installing a system-wide package or managing a system service legitimately needs root. Your project's own dependencies never do — and installing them as root leaves files in your repo that your normal user cannot then edit.

### Q5 · L3
A directory has permissions `r--------` for its owner. The owner can list the names inside it but gets `Permission denied` trying to open any file within. Why?
- (a) The files inside are owned by a different user
- (b) On a directory, `x` means permission to enter it, and it is missing ✅
- (c) `r` on a directory only applies to hidden files
- (d) The directory needs `w` before anything inside can be read
> On a directory, `r` lets you read the list of names and `x` lets you traverse into it to reach the contents. Read without execute produces exactly this confusing half-working state.

## Go deeper
- [Ubuntu Server: the Linux file system and permissions](https://ubuntu.com/server/docs)
- [The Linux Filesystem Hierarchy Standard, explained plainly](https://tldp.org/LDP/Linux-Filesystem-Hierarchy/html/)
