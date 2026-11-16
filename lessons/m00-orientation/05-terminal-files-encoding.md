---
id: m00-05-terminal-files-encoding
module: m00-orientation
slug: terminal-files-encoding
title: "The Terminal, the File System and How Data Is Encoded"
order: 5
duration_min: 25
is_free: true
mission: false
volatility: low
last_reviewed: 2026-11-16
---

## Key learnings
- Explain what a terminal is and why engineers prefer it for some work
- Read a file path and say where it points
- Say what a file extension does and does not guarantee
- Explain why text needs an encoding, and what UTF-8 solves
- Recognise the three classic encoding and size traps before they cost you a day

## Lesson

### The pain

An AI agent tells you: "Run `npm install` in the project root." You do not know what a terminal is, what "the project root" means, or how to get there. You are stuck at the first instruction, and the thing blocking you is not difficult — it is just unexplained.

Later, a student's name appears on a receipt as `Pri?a Sh?rma`. The data is correct in the database. It is correct in the admin panel. It is broken only on the receipt. That is an encoding bug, and it will cost you an afternoon unless you know the shape of it.

Neither of these is advanced. Both are assumed.

### The idea

**The terminal is a text conversation with your computer; the file system is the tree it talks about; and encoding is the agreement about how letters become numbers.**

Three unglamorous things. You will not write code in this program, but you will read instructions that assume all three, and you will debug problems caused by all three.

### The analogy

Imagine two ways to ask the canteen for yesterday's sales.

The first: walk in, look at the register, flip pages, read it. That is a graphical interface — you see everything, you point at what you want, and it is obvious but slow and hard to repeat.

The second: hand the manager a written note — "total sales, yesterday, grouped by item, only items over ₹50." That is the terminal. It is harder to write than pointing, but it is exact, it is fast, you can keep a copy of the note, you can hand the same note over every week, and you can send it by message without being in the room.

That last part is the real reason servers are run this way: **a server has no screen.** It is in a data centre somewhere. Written notes are the only way to talk to it.

**Where the analogy breaks:** a manager will interpret a vague note charitably. A terminal will not. It does exactly what you typed, including deleting the wrong thing without asking. This is why you will read commands before running them, and why permission prompts exist.

### Picture it

```
  /                         the root — everything starts here
  ├── home/
  │   └── aarav/            your home directory  (shortcut: ~)
  │       └── projects/
  │           └── canteen/  <-- "the project root"
  │               ├── package.json
  │               ├── .env          secrets — never commit this
  │               └── src/
  │                   └── app.js
  ├── etc/                  system configuration lives here
  └── var/
      └── log/              logs live here
```

Everything on a Unix-like machine hangs off a single root called `/`. A path is just the route down that tree, and "the project root" means the folder holding your project's config files — `canteen/` above, not `src/`.

### Reading a path

Paths are the thing beginners misread most, and the rules are small:

| Path | Means |
|---|---|
| `/home/aarav/projects` | Absolute — starts at root, unambiguous anywhere |
| `projects/canteen` | Relative — starts from wherever you currently are |
| `.` | Right here |
| `..` | One level up |
| `~` | Your home directory |
| `./src/app.js` | `app.js`, inside `src`, inside here |
| `../canteen/.env` | Up one level, then down into `canteen` |

Windows uses `C:\Users\Aarav\projects` with backslashes instead, which is why instructions sometimes do not work verbatim on Windows — the separator differs. On Windows, WSL gives you a Linux-style tree so the instructions match.

A leading dot, as in `.env` or `.git`, means **hidden**. It is not security; it is a convention that keeps clutter out of listings. Critically, hidden does not mean safe: `.env` holds your secrets and is hidden, which is exactly why people forget it exists and accidentally publish it.

### Extensions promise nothing

`report.pdf` is a *name*. Renaming an image to `report.pdf` does not make it a PDF; it makes it an image with a misleading name. The extension is a hint to humans and to the operating system about which program to open it with — nothing more.

This matters the moment your product accepts uploads. A file called `photo.jpg` can contain anything at all. "We only allow `.jpg`" is not a check; it is a label. Real validation looks inside the file. You will meet this properly in M07, but the instinct starts here: **the name is a claim, not a fact.**

### Encoding: how letters become numbers

A computer stores numbers. Text has to be converted, and the conversion table is called an **encoding**.

Old encodings like ASCII had 128 slots — enough for English letters, digits and punctuation, and nothing else. No Devanagari, no emoji, no `₹`. Different countries invented incompatible tables for their own scripts, so the same bytes meant different letters depending on who opened the file. That is where `Pri?a Sh?rma` comes from: the bytes were written with one table and read with another.

**UTF-8 solved this** by giving every character in every script a single agreed number, and storing common characters in fewer bytes. It is now the default on the web, and the rule is simple: use UTF-8 everywhere, end to end — database, server, page, file export — and the problem disappears. Mixed encodings anywhere in the chain bring it back.

One consequence worth internalising: **in UTF-8, one character is not one byte.** `a` is one byte, `प` is three, and a single emoji can be four or more. So "name must be under 20 characters" and "name must be under 20 bytes" are different rules, and a student called प्रिया can fail a length check that a student called Priya passes. This is a real bug that ships regularly in Indian products.

### Where it lives

The terminal is how you reach the server box from chapter 1 — a server has no screen, so a written instruction is the only channel there is. Paths start mattering the moment you deploy, because "the project root" is where every build command expects to be run from. And encoding matters at every boundary *between* boxes: the database, the connection to it, the page and any file export each declare one, and a mismatch at any single hop produces the broken name.

All three return in M06, where you reach a free virtual machine over SSH and read its logs to find out why something failed — and where the Linux case-sensitivity rule from this chapter becomes the most common first-deploy failure there is.

### Words to own

| Term | Plain meaning | Example |
|---|---|---|
| Terminal / shell | A text interface for giving commands | bash, zsh, PowerShell |
| CLI | Command-line interface | `git`, `npm` |
| Path | The route to a file in the tree | `/home/aarav/app.js` |
| Absolute path | Starts from the root `/` | Works from anywhere |
| Relative path | Starts from where you are now | `./src` |
| Working directory | The folder you are currently "in" | Shown by `pwd` |
| Project root | The folder holding the project's config | Where `package.json` lives |
| Dotfile | A hidden file, name starting with `.` | `.env`, `.gitignore` |
| Encoding | The map from characters to bytes | UTF-8 |
| UTF-8 | The universal encoding; the modern default | Handles every script and emoji |
| Byte | 8 bits; the unit of storage | One ASCII letter |

### Commands as vocabulary

You will not memorise commands. You will *recognise* them, so that when an AI agent proposes one you can tell whether it reads or destroys. That distinction is the whole point.

| Command | Does | Dangerous? |
|---|---|---|
| `pwd` | Prints where you are | No |
| `ls` | Lists what is here | No |
| `cd projects` | Moves into a folder | No |
| `cat file.txt` | Prints a file's contents | No |
| `cp a b` | Copies a to b | Overwrites b |
| `mv a b` | Moves or renames | Overwrites b |
| `rm file` | Deletes, with no recycle bin | **Yes** |
| `rm -rf folder` | Deletes a folder and everything in it, silently | **Very** |
| `sudo <anything>` | Runs it as the all-powerful administrator | **Depends entirely** |

Two of those have no undo. There is no recycle bin on a server. When an agent proposes `rm -rf` or `sudo`, that is the moment to read carefully rather than approve reflexively — which is exactly what the next chapters on AI mistakes and agent permissions are about.

### Tradeoffs and gotchas

**Three classic traps, all cheap to avoid:**

1. **The `?` or `�` in names.** Mixed encodings somewhere in the chain. Check every hop — database charset, connection charset, page charset, file export — and make all of them UTF-8.
2. **A length check that rejects Indian names.** The check is counting bytes, not characters. Count characters.
3. **A file that is 1 KB in one place and 1,024 bytes in another.** Storage is sold in powers of ten (1 KB = 1,000 bytes) and measured by operating systems in powers of two (1 KiB = 1,024 bytes). This is why a "500 GB" drive shows as 465 GB. Nobody is lying; two units share one name.

**The terminal has no confirmation and no history of what it destroyed.** Graphical tools ask "are you sure?" Terminals assume you meant it. This is a feature when you are running the same deployment a hundred times, and a disaster the one time you typo a path.

**Case sensitivity differs by system.** On Linux — which is what your server runs — `Menu.js` and `menu.js` are two different files. On Windows and macOS they are usually the same file. This produces the genuinely baffling bug where a project works perfectly on your laptop and fails on deploy with "module not found", because of one capital letter.

### Verify it

Before you run any command an AI gives you, three checks, in this order:

1. **Does it delete, overwrite or move anything?** Look for `rm`, `mv`, `cp`, `>`, `truncate`, `DROP`. If yes, read the path character by character. A space in the wrong place turns "delete this folder" into "delete everything".
2. **Does it need `sudo`?** If so, ask why. Most project commands do not. An agent reaching for administrator rights to install a project dependency is usually solving a path problem the wrong way.
3. **Where does it think I am?** Most "it did not work" moments are a working-directory mistake. Run `pwd` and `ls` first and confirm you are where the instructions assume.

A good habit with agents: **ask it to explain the command before running it.** "What does this command do, what does it change, and what happens if I run it twice?" An agent that cannot answer the third question clearly is proposing something you should not run.

## Walkthrough
1. Open a terminal. On Windows, install WSL or use Git Bash so the commands here match; on macOS open Terminal; on Linux you already have one. GitHub Codespaces also gives you one in the browser with nothing to install.
2. Type `pwd` and press Enter. This prints your working directory — the answer to "where am I". Every relative path is interpreted from here.
3. Type `ls` to list what is here. Then `ls -a` to include hidden dotfiles. Notice how much was invisible a moment ago.
4. Type `cd ..` and then `pwd` again. You have moved one level up the tree. Type `cd ~` to jump straight home.
5. Make a folder to experiment in: `mkdir terminal-practice`, then `cd terminal-practice`, then `pwd` to confirm where you landed.
6. Create a file with some non-English text in it, using your normal editor — include a Devanagari name and the `₹` symbol. Then `cat` the file in the terminal. If the characters survive, your whole chain is UTF-8.
7. Check a file's real size: `ls -l` shows bytes. Compare the byte count to the number of characters you typed. For text with Devanagari or emoji, bytes will exceed characters — that is the trap from this lesson, visible.
8. Finally, practise the safety habit. Write out the command `rm -rf ~/terminal-practice` **without running it**, and say out loud what would happen if you mistyped the path as `~ /terminal-practice` with a space. Then delete the folder deliberately and carefully.

## Talk to AI
```prompt
I am going to paste a terminal command that I was told to run. Before I run it, I want you to act as a careful reviewer.

The command is: [paste it]

Tell me:
1. In one plain sentence, what this does
2. Exactly what it creates, changes, overwrites or deletes
3. Whether any part of it is irreversible, and which part
4. Whether it needs administrator rights, and if so, why — and whether there is a way to avoid that
5. What happens if I run it twice by accident
6. What I should check before running it, and how I would confirm afterwards that it worked

If any part of this command could destroy something I care about, say so first and plainly, before the rest of the explanation.
```

## Stuck? Ask AI
```prompt
I do not understand file paths. Explain absolute paths, relative paths, `.`, `..` and `~` using one example tree, then give me five paths and ask me where each one points. Do not tell me the answers until I have guessed all five.
```

```prompt
A student's name shows as "Pri?a" on a receipt but is correct in the database. Walk me through every place an encoding could be wrong, in the order I should check them. Assume I know nothing about character encodings.
```

```prompt
Explain why a name length check can reject an Indian name written in Devanagari but accept the same name written in English letters. Then tell me the correct way to write that check.
```

## Quiz

### Q1 · L3
A project works on your laptop but fails on deploy with "module not found", and the only difference you can spot is `Menu.js` versus `menu.js`. Why?
- (a) The server has a slower disk
- (b) Linux file names are case-sensitive; macOS and Windows usually are not ✅
- (c) The file was corrupted during upload
- (d) `.js` files must always be lowercase
> Your laptop treated the two names as one file; the Linux server treats them as two. This is one of the most common first-deploy failures there is.

### Q2 · L2
What does a file extension like `.pdf` actually guarantee about a file?
- (a) That the file contains a valid PDF document
- (b) Nothing — it is a name, and the contents can be anything ✅
- (c) That the file was created by a PDF program
- (d) That the file is safe to open
> The extension is a hint for humans and for choosing a program. Real validation has to look inside the file, which matters the moment you accept uploads.

### Q3 · L4
A form rejects the name "प्रिया" as too long but accepts "Priya", with the same limit of 20. What is wrong?
- (a) Devanagari is not supported by UTF-8
- (b) The check counts bytes rather than characters ✅
- (c) The database column is too small
- (d) The form is using ASCII instead of UTF-8
> In UTF-8 a Devanagari character takes about three bytes. A byte-based length check therefore discriminates against non-English names, which ships in real Indian products regularly.

### Q4 · L3
Which of these commands is irreversible and has no recycle bin?
- (a) `ls -a`
- (b) `cd ..`
- (c) `rm -rf folder` ✅
- (d) `pwd`
> `ls`, `cd` and `pwd` only look or move. `rm -rf` deletes a folder and its contents silently and permanently — the single command most worth reading twice.

### Q5 · L3
An instruction says "run this in the project root". Where is that?
- (a) The `/` directory at the top of the file system
- (b) The folder containing the project's config files, such as `package.json` ✅
- (c) The `src` folder where the code lives
- (d) Your home directory
> "Root" here means the top of *the project*, not the top of the machine. It is the folder holding config files — and getting this wrong is the most common reason a command appears not to work.

## Go deeper
- [The Absolute Minimum Every Software Developer Must Know About Unicode](https://www.joelonsoftware.com/2003/10/08/the-absolute-minimum-every-software-developer-absolutely-positively-must-know-about-unicode-and-character-sets-no-excuses/)
- [Ubuntu: The Linux command line for beginners](https://ubuntu.com/tutorials/command-line-for-beginners)
