---
id: m06-03-ssh-keys-firewalls
module: m06-cloud-linux-devops-testing
slug: ssh-keys-firewalls
title: "SSH, Key Pairs and Firewalls"
order: 3
duration_min: 25
is_free: false
mission: false
volatility: low
last_reviewed: 2026-11-16
---

## Key learnings
- Explain what a key pair is and which half never leaves your laptop
- Say why a password on a public server is a losing strategy
- Read a firewall rule and say which traffic it allows
- Recognise the three mistakes that expose a student VM within hours
- Describe how a deploy platform uses the same mechanism without you logging in

## Lesson

### The pain

A team spins up a free-tier VM to run their canteen app's database. They set the root password to something they will remember, open the database port to the whole internet so they can connect from their laptops, and get on with building.

Nine hours later the VM is at 100% CPU, the free tier's monthly compute allowance is nearly spent, and the database contains a table called `README_TO_RECOVER_YOUR_DATA`. They have been found by an automated scanner, logged into with a guessed password, and their data has been copied and deleted.

Nothing about this is unusual. A public IP address receives login attempts within minutes of existing — not because anyone targeted these students, but because the entire address space is swept continuously by bots. The team did three things wrong, and each of them is a one-line decision made in ten seconds.

### The idea

**Prove who you are with a key pair instead of a password, and open the smallest number of ports you can get away with.**

A key pair is two matching files generated together. The private key stays on your laptop forever. The public key is copied onto any server you want to reach. The server challenges you, your laptop answers using the private key, and the server checks the answer against the public key. Your secret is never sent anywhere, so there is nothing to intercept and nothing to guess.

A firewall is a list of rules about which ports accept connections from where. Everything not on the list is refused before your application ever sees it.

### The analogy

The canteen's back door has a lock that a specific key opens. The manager keeps the key on their person; a copy of the *lock* is fitted to the door. Anyone who wants in must physically hold the key. Shouting the right word through the door does not work, however many words you try.

The front of the building has exactly one opening: the serving window. Students queue there. The store room has no external door at all — deliveries come through the kitchen, which is reached through the staff door, which has the lock. Nobody can reach the rice sacks from the street, because there is no path from the street to the rice sacks.

That is a firewall: not a guard who inspects people, but an absence of doors.

**Where the analogy breaks:** a physical key can be copied by a locksmith who holds it for a minute, and a lock can be picked. A private key cannot be derived from the public key by any amount of effort available today. But a private key *can* be copied in one second by anything that can read the file — which is why a leaked private key is a far worse event than a stolen physical key, and why it must never be committed to git, pasted into a chat, or left in a shared folder.

### Picture it

```
  YOUR LAPTOP                              THE SERVER
 ┌────────────────┐                   ┌────────────────────┐
 │ private key    │                   │  public key        │
 │ (never leaves) │                   │  (a copy, safe to  │
 └───────┬────────┘                   │   share)           │
         │                            └─────────┬──────────┘
         │   1. "who are you?"                  │
         │ ◀────────────────────────────────────┘
         │   2. signed answer
         └────────────────────────────────────▶ checks it
                                               │
   ┌───────────────────────────────────────────┐│
   │ FIREWALL:  22 open to my IP only          ││
   │            443 open to everyone           ▼│
   │            5432 open to nobody        connected
   │            everything else refused         │
   └────────────────────────────────────────────┘
```

Your laptop proves its identity by signing a challenge with a private key that never travels, while the firewall decides, before any of that happens, whether the connection is even allowed to reach the server.

### Ports, and the three that matter

A port is a numbered door on an IP address. One machine, many ports, each one potentially a different service listening. M02 covered this from the browser's side; here is the server's side.

| Port | What usually listens | Who should reach it |
|---|---|---|
| 22 | SSH — remote shell | You, from your own IP |
| 80 | Plain HTTP | Everyone, to redirect to 443 |
| 443 | HTTPS | Everyone |
| 5432 | PostgreSQL | Only your app, never the internet |
| 6379 | Redis | Only your app, never the internet |
| 3000 | A dev server | Nobody, in production |

The pattern is simple and the students who get breached broke it: ports your *users* need are open to everyone; ports your *application* needs internally are open to the application only; ports *you* need are open to you only. A database port open to the world is the single most expensive mistake on this list, because the database is the part nobody can rebuild.

### SSH in practice, without memorising anything

SSH stands for Secure Shell. It gives you a terminal on a remote machine over an encrypted connection. In this program you will use it rarely — most of your work happens on platforms with a web terminal — but it underpins three things you use constantly.

When you push to GitHub over SSH, that is a key pair. The public key is on your account; the private key is on your laptop; nothing is typed.

When a deploy platform pulls your repository, it uses a key or a token the same way, on your behalf.

When you connect to a free-tier VM, the provider asks for your public key at creation time and installs it for you, which is why a correctly created VM has no password login at all.

Two practical rules. **Generate a key once per machine, not once per project** — one key on your laptop, its public half copied wherever needed. And **protect the private key with a passphrase**, so that a stolen laptop is not also a stolen server.

### Where it lives

SSH and firewalls sit at the boundary of everything else in this module. Chapter 7's billing trap often starts with a compromised VM mining cryptocurrency on your free tier. Chapter 8's containers get their own network, which is a firewall by another name — a container only reaches what you connect it to. Chapter 9's Kubernetes adds network policies, the same idea again with more vocabulary. Chapter 12's CI needs a credential to deploy, and that credential is a secret stored by the platform rather than a key sitting in your repo.

M07 takes this further into the OWASP material and secret handling. M00's lesson about agent permissions is the same instinct applied to a different actor: give the minimum access that works, and no more.

### Words to own

| Term | Plain meaning | Example |
|---|---|---|
| SSH | Encrypted terminal on a remote machine | Port 22 |
| Key pair | Two matched files, generated together | Private + public |
| Private key | Your secret half; never leaves your laptop | `id_ed25519` |
| Public key | The shareable half, copied to servers | `id_ed25519.pub` |
| Passphrase | A password protecting the private key file | Guards a stolen laptop |
| Port | A numbered door on an IP address | 443 for HTTPS |
| Firewall | Rules for which ports accept what, from where | Default: refuse |
| Inbound rule | Who may connect in | `443 from 0.0.0.0/0` |
| `0.0.0.0/0` | "From anywhere on the internet" | Fine for 443, fatal for 5432 |
| Least privilege | Grant the minimum access that works | One port, one source |
| Bastion / jump host | One hardened machine you reach others through | Advanced, M13 |

### Tradeoffs and gotchas

**Disabling password login is the single highest-value change you can make.** Key-only authentication removes the entire category of guessing attacks, which is what almost all of that background noise is. Most cloud images now default to it, and a tutorial that tells you to re-enable passwords is telling you to undo the protection.

**Opening a port to `0.0.0.0/0` is a decision, not a default.** It means "anyone on the internet". For 443 that is the point. For a database port it is the breach. Read every firewall rule as a sentence: "allow anyone, anywhere, to connect to my database" is a sentence you would never say out loud, yet it is the same rule.

**A private key in a repository is the worst leak available to you.** Worse than an API key, because it often unlocks the machine rather than one service. M01's rule applies with no exceptions: it goes in `.gitignore` before the first commit, and if it ever reached a commit, you generate a new pair and remove the old public key from every server today, not next week.

**Firewalls do not protect you from your own application.** An open 443 with a vulnerable app behind it is wide open. This is what M07's OWASP chapter is about, and it is why "we have a firewall" is not an answer to "is it secure".

**Your home IP changes.** Locking port 22 to your current address is good practice and will lock you out when your 4G connection moves you to a different address. Know how to reach the provider's web console before you need it.

### Verify it

Agents are confident about infrastructure and frequently generous with access. Four checks.

1. **Read every firewall rule back as an English sentence.** Port, protocol, source. If the sentence contains "anyone on the internet" and a port that is not 80 or 443, stop and ask why.
2. **Ask where each secret is stored.** The acceptable answers are: the platform's secret store, or an untracked local file. Unacceptable: in the repository, in a Dockerfile, in a CI configuration file committed to git, or pasted in the chat.
3. **Check whether it chose passwords or keys.** If a setup script sets a root password and enables password login, it has traded the main protection for convenience. Ask for the key-based version.
4. **Ask what the recovery path is.** Before applying a firewall change to a machine you reach over SSH, the question "what do I do if this locks me out?" must have an answer. An agent that applies the rule without raising this has not thought about it.

Red flags: a database port open to the world; `PermitRootLogin yes`; a private key pasted into a conversation or a config file; `chmod 777` on a `.ssh` directory — SSH will actually refuse a key whose permissions are too open, and the correct response is to narrow them, not to disable the check.

## Walkthrough
1. On your own laptop, ask your AI assistant for the single command that generates a modern key pair with a passphrase, and have it explain which of the two resulting files must never move.
2. Run it. Then list the two files and note their permissions. The private key should be readable only by you — compare that against what you learned in chapter 1.
3. Open the public key file and read it. One line, starting with a key type. Note that you could post this in a group chat with no consequence, and say in your own words why.
4. Open the private key file for one second and close it. Note the header line. This is the file that must never be committed, pasted, or synced to a shared drive.
5. Add the public key to your git hosting account, then test the connection with the provider's read-only verification command. You have just authenticated without typing a password.
6. Now the firewall. Create a free-tier VM, or open an existing project's network settings page, and find the inbound rules list.
7. Read each rule aloud as a sentence: "allow TCP port 443 from anywhere". Write the list down in that form in your team notes.
8. Find a rule you cannot justify and work out what would break if you removed it. Do not remove port 22 from your own IP while you are relying on it.
9. If your project uses a managed database, find its network settings and confirm whether it is reachable from the internet or only from your app. Screenshot whichever you find.
10. Finally, ask your AI assistant to review your inbound rule list and name the single riskiest rule and why — then check its reasoning against the port table in this chapter yourself.

## Talk to AI
```prompt
CONTEXT:      I am a student who directs AI rather than writing code. My team
              of three is building a web product. We have a free-tier Linux VM
              and a managed database, and we deploy through a hosting
              platform. We are on student budgets in India, so a compromised
              VM burning our free-tier compute is a real financial risk. None
              of us has configured a server or a firewall before.

GOAL:         Set up remote access and network rules so that an automated
              scanner finding our public IP cannot get in, and so that our
              database is unreachable from the internet.

REQUIREMENTS: 1. Explain key-pair authentication in plain words - which file
                 stays on my laptop, which one goes on the server, and why my
                 secret is never transmitted
              2. Give me the exact steps to create a key pair with a
                 passphrase, and tell me where the files land on my operating
                 system
              3. Explain why password login on a public server fails, with a
                 realistic number for how soon login attempts begin
              4. Give me a minimal inbound firewall rule set for this setup,
                 written as one plain English sentence per rule, and justify
                 every open port
              5. Tell me how to make the database reachable by my app but not
                 by the internet, and how I verify that from outside

CONSTRAINTS:  Free tier only. Plain English, second-language reader. My home
              IP address changes because I am often on mobile data. Label
              every command READ-ONLY or CHANGES-THINGS. Before any command
              that changes network rules, tell me how I would recover if it
              locked me out.

NON-GOALS:    Do not set up a bastion host, a VPN, certificate authorities, or
              fail2ban. Do not tell me to re-enable password login for
              convenience. Do not put any secret in a file I would commit.

ACCEPTANCE:   - I can say which of my two key files is safe to share
              - My inbound list contains no rule allowing the whole internet
                to reach a database port
              - I can describe how I would verify from my phone that the
                database port is closed
              - I have a written recovery path for being locked out of SSH
              - Every secret in my setup lives in the platform's secret store
                or an untracked local file, and I can say which

ASK:          First ask me three questions about my actual setup - provider,
              what the VM runs, and what the database is - so the rules match
              reality. Then propose two approaches to the access setup with
              tradeoffs before giving me any commands. No commands until I
              pick one.
```

## Stuck? Ask AI
```prompt
Explain public and private keys to me as if I have never heard of encryption, using an analogy that is not about padlocks or mailboxes. Then tell me, concretely, what an attacker who obtains my public key can do with it, and what one who obtains my private key can do with it. Finish by asking me three questions to check I actually understood which file is which.
```

```prompt
Here is my cloud firewall's inbound rule list: [paste it exactly, including ports and source ranges]. Rewrite each rule as one plain English sentence of the form "allow <who> to connect to <what>". Then rank the rules from most to least risky and explain the top one. Do not change anything - I want the reading, not a fix.
```

```prompt
I think a private key or a server credential may have been committed to our git repository. Tell me, in priority order, exactly what I should do in the next ten minutes, and explain why deleting the file in a new commit does not solve it. Then tell me what I should do in the next day.
```

## Quiz

### Q1 · L2
In a key pair, which half goes onto the server you want to connect to?
- (a) The private key, because the server needs to verify you
- (b) The public key ✅
- (c) Both, so they can be matched locally
- (d) Neither — the pair is only used by your laptop
> The public key is installed on the server and is safe to share. The private key never leaves your machine; it signs a challenge, and the signature is what travels.

### Q2 · L3
Why is password authentication a losing strategy on a machine with a public IP address?
- (a) Passwords are transmitted in plain text over SSH
- (b) The address space is swept continuously by bots, so guessing attempts begin within minutes ✅
- (c) Linux stores passwords in a file anyone can read
- (d) Passwords expire too quickly to be practical
> SSH encrypts the password in transit, so interception is not the issue. Volume is: an exposed port receives automated login attempts constantly, and only one guess has to work.

### Q3 · L4
Which firewall rule should make you stop and ask why?
- (a) Allow TCP 443 from `0.0.0.0/0`
- (b) Allow TCP 22 from your office IP only
- (c) Allow TCP 5432 from `0.0.0.0/0` ✅
- (d) Allow TCP 80 from `0.0.0.0/0`
> 80 and 443 are what users need and belong open. 5432 is PostgreSQL. Read as a sentence it says "let anyone on the internet connect to my database" — the one mistake nobody can rebuild from.

### Q4 · L4
A tutorial tells you to set `PermitRootLogin yes` and re-enable password authentication so your teammates can log in more easily. What are you actually trading?
- (a) Nothing — it is the same security with a different interface
- (b) You restore the entire guessing-attack surface and hand it the most powerful account ✅
- (c) Only convenience, since root is protected separately
- (d) Encryption, which only applies to key-based logins
> Key-only login removes guessing attacks completely. Re-enabling passwords brings them back, and allowing root login means a single successful guess owns the machine. Add teammates' public keys instead.

### Q5 · L5
You are about to apply a new firewall rule set to a VM you can only reach over SSH. What must be true before you apply it?
- (a) The rules have been reviewed by a teammate
- (b) You have a written recovery path for being locked out, such as the provider's web console ✅
- (c) You have taken a snapshot of the disk
- (d) Port 22 is open to `0.0.0.0/0` as a fallback
> The change can cut the connection you are using to make it. A snapshot does not help you log in, and a wide-open port 22 defeats the purpose. Knowing the out-of-band console path is what makes the change safe to attempt.

## Go deeper
- [Ubuntu Server: OpenSSH and firewall configuration](https://ubuntu.com/server/docs)
- [Mozilla's OpenSSH configuration guidelines](https://infosec.mozilla.org/guidelines/openssh)
