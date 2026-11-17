---
id: m00-05-terminal-files-encoding
module: m00-orientation
slug: terminal-files-encoding
title: "The Terminal, the File System and How Data Is Encoded"
order: 5
duration_min: 15
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

An AI agent tells you: "Run `npm install` in the project root." You do not know what a terminal is or where "the project root" is. Nothing difficult, just unexplained.

Later, a student's name prints on a receipt as `Pri?a Sh?rma` while being correct in the database: an encoding bug that costs an afternoon.

### The idea

**The terminal is a text conversation with your computer; the file system is the tree it talks about; and encoding is the agreement about how letters become numbers.**

You will not write code in this program, but you will read instructions that assume all three.

### The analogy

Two ways to ask the canteen for yesterday's sales. Walk in and flip through the register: a graphical interface — obvious, slow, hard to repeat. Or hand the manager a note: "total sales, yesterday, grouped by item, only items over ₹50." That is the terminal: harder to write, but exact, repeatable, and sendable without being in the room. That last part is why servers work this way — **a server has no screen**.

**Where the analogy breaks:** a manager interprets a vague note charitably. A terminal does not — it does exactly what you typed, including deleting the wrong thing without asking.

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

Everything hangs off a single root called `/`; a path is the route down that tree, and "the project root" is the folder holding your config files — `canteen/` above, not `src/`.

Paths are what beginners misread most:

| Path | Means |
|---|---|
| `/home/aarav/projects` | Absolute — starts at root, unambiguous anywhere |
| `projects/canteen` | Relative — starts from wherever you currently are |
| `.` | Right here |
| `..` | One level up |
| `~` | Your home directory |
| `./src/app.js` | `app.js`, inside `src`, inside here |
| `../canteen/.env` | Up one level, then down into `canteen` |

### Where it lives

The terminal is how you reach the server box from chapter 1; paths matter at deploy time, because "the project root" is where build commands expect to run.

Encoding lives at every boundary *between* boxes. A computer stores numbers, so text needs a conversion table — the **encoding**. ASCII had 128 slots: English letters, digits and punctuation, no Devanagari, no `₹`. Countries then invented incompatible tables, so the same bytes meant different letters depending on who opened the file — which is where `Pri?a Sh?rma` comes from. **UTF-8 solved it** by giving every character in every script one agreed number. Use UTF-8 end to end — database, connection, page, export — and the problem disappears; a mismatch at any hop revives it.

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

### Tradeoffs and gotchas

**In UTF-8, one character is not one byte.** `a` is one byte, `प` three, an emoji four or more. So "under 20 characters" and "under 20 bytes" differ, and प्रिया fails a check Priya passes.

**A file name is a claim, not a fact.** Renaming an image to `report.pdf` gives you an image with a misleading name, so "we only allow `.jpg`" is a label, not a check. A leading dot means hidden, not safe — hence published `.env` files.

**1 KB is not 1 KB.** Storage is sold in powers of ten (1,000 bytes) and measured by the OS in powers of two (1,024). Hence a "500 GB" drive showing 465 GB.

**The terminal never asks "are you sure?"** A feature when you run the same deploy a hundred times; a disaster the one time you typo a path.

**Case sensitivity differs by system.** On Linux, which your server runs, `Menu.js` and `menu.js` are two files; on Windows and macOS, usually one. Hence "works on my laptop, module not found on deploy".

### Verify it

Recognise commands rather than memorise them: does this one read, or destroy?

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

Three checks before you run it:

1. **Does it delete, overwrite or move anything?** `rm`, `mv`, `cp`, `>`, `DROP`. If so, read the path character by character — one stray space turns "delete this folder" into "delete everything".
2. **Does it need `sudo`?** Most project commands do not; an agent reaching for admin rights is usually solving a path problem the wrong way.
3. **Where does it think I am?** Most "it did not work" moments are a working-directory mistake. Run `pwd` first.

There is no recycle bin on a server, so ask first: "what does this change, and what happens if I run it twice?" An agent that cannot answer clearly is proposing something you should not run.

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
