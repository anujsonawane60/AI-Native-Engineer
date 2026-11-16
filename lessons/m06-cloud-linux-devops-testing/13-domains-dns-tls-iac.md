---
id: m06-13-domains-dns-tls-iac
module: m06-cloud-linux-devops-testing
slug: domains-dns-tls-iac
title: "Domains, DNS Records, TLS Certificates and Infrastructure as Code"
order: 13
duration_min: 25
is_free: false
mission: false
volatility: low
last_reviewed: 2026-11-16
---

## Key learnings
- Read a DNS record and say what it points at and how long it is cached
- Explain why a domain change can take hours to reach some of your users
- Say what a TLS certificate proves, and what it does not
- Describe what infrastructure as code gives you, and when a text file beats it
- Recognise a DNS or certificate answer that is guessing rather than checking

## Lesson

### The pain

A team buys `canteenorder.in` for their demo. They point it at their app, it works on a laptop, and they announce it. Four teammates can reach it. One cannot, and keeps seeing the old placeholder page for six hours, because their phone's network is holding the previous answer in a cache nobody controls.

Then the browser starts saying "Not secure" in red. Their app is on HTTPS; their custom domain is not, because the certificate was issued for the platform's subdomain and not the domain they just added. A student sees a red warning before they see a menu, and does not enter their phone number.

Three weeks later a teammate cleans up cloud resources and deletes the one nobody could identify. It was the bucket serving the dish photos, created by clicking around a console in week two. They rebuild it from memory, slightly differently, and every image URL changes.

### The idea

**A domain name is a lookup table the whole internet caches, a certificate is a time-limited proof that you control that name, and infrastructure as code is the same idea applied to everything else you created by clicking.**

All three are really one idea: the important configuration of your product should be written down somewhere, with a known owner and a known expiry, rather than living in a console's memory and in yours.

### The analogy

The canteen moves from Block A to Block C.

The new location is a fact. But the campus noticeboard still says Block A, the printed hostel map says Block A, and last year's WhatsApp group says Block A. Updating the official board is instant; the copies people already hold expire on their own schedule, and anyone holding an old copy walks to Block A and finds nothing. That is DNS, and the delay is called propagation.

The health licence pinned to the wall is the certificate. It proves an inspector verified this stall, it has a date, and it expires. A student does not have to trust the stall; they trust the body that issued it.

The recipe binder is infrastructure as code: the layout, supplier list and counter arrangement written down, so the next manager does not reconstruct it from rumour.

**Where the analogy breaks:** a noticeboard has one authority and one copy. DNS is cached by your phone, your browser, your network provider and intermediate resolvers, each with its own clock, and *none* of them answers to you. You can shorten the clock in advance; you cannot clear someone else's cache. That is the entire reason domain changes need planning instead of confidence.

### Picture it

```
  student types canteenorder.in
        │
        ▼
  ┌──────────────┐  cached?  ┌──────────────────┐
  │ phone / OS   │──────────▶│ answer, for TTL  │
  └──────┬───────┘   yes     │ seconds          │
         │ no               └──────────────────┘
         ▼
  ┌──────────────┐    ┌──────────────┐    ┌─────────────┐
  │ network      │───▶│ authoritative│───▶│ A / CNAME   │
  │ resolver     │    │ nameserver   │    │ record      │
  └──────────────┘    └──────────────┘    └──────┬──────┘
                                                 │
                                                 ▼
                                         your app's address
                                         + TLS handshake
```

A name is resolved by asking a chain of caches before reaching the nameserver that actually holds your records, and every cache in that chain keeps its answer for the record's TTL.

### The records you will actually touch

| Record | What it does | Typical use |
|---|---|---|
| A | Points a name at an IPv4 address | `canteenorder.in` → an IP |
| AAAA | Same, for IPv6 | Alongside A |
| CNAME | Points a name at another name | `www` → your platform's host |
| TXT | Holds arbitrary text | Proving you own the domain |
| MX | Where email for this domain goes | Your email provider |
| NS | Which nameservers are authoritative | Set at your registrar |

**TTL — time to live — is the number that decides your pain.** It is the seconds each cache may keep an answer. A TTL of 3,600 means someone may see the old answer for an hour after you change it. So lower the TTL to a few minutes *a day before* a planned change, make the change, then raise it again. Lowering it afterwards does nothing, because the caches already took the long value.

**Use a CNAME to your platform rather than an A record to its IP.** Platform IP addresses change without notice; the hostname does not. An A record pointing at a reassigned IP is a silent outage that will baffle you.

**Your registrar and your DNS host can be different things.** The registrar is who you bought the name from; the DNS host is whose nameservers answer for it. Records added at the registrar do nothing if the NS records point elsewhere — the commonest reason a correct-looking record has no effect.

### TLS, in the amount you need

A TLS certificate does exactly two things: it encrypts traffic between your user and your server, and it proves that whoever holds the certificate controls that domain name. The padlock means the connection is private and the name is verified.

It does not mean the site is safe, honest, or well built. A phishing site has a valid padlock. This matters for the security judgement in M07 and for how you explain the padlock to a non-technical user.

Three practical facts. Certificates are **free and automatic** on every platform in this program, usually via Let's Encrypt — anyone telling you to pay for a basic certificate is selling something. They **expire**, typically every 90 days, with automatic renewal; an expired certificate on a student project almost always means the automation broke quietly. And they are issued **per name**: adding a custom domain requires a new certificate, which is why the red warning appeared above and why you re-check the padlock after every domain change.

### Infrastructure as code, honestly

Infrastructure as code means your cloud resources are described in text files, kept in git, and applied by a tool — so the setup is reviewable, repeatable, and recoverable. Terraform is the common one. The benefits are real: no more unidentifiable resources, a diff before a change, and the ability to rebuild from nothing.

The honest position for this course is that **you do not need the tool yet, and you absolutely need the discipline now.**

For a team of three with four resources, Terraform costs more than it returns and introduces a new way to destroy things — a careless apply can delete a database. What you need is the `STACK.md` file from chapter 6: every resource, who created it, what it is for, which region, and the non-default settings. A text file in your repository delivers most of the value for a tenth of the work.

M13 does infrastructure as code properly, with state files, plans and modules. Two things count as a real, free version of the idea today: the deploy configuration file many platforms read from your repository, and your list of environment variable names with the values removed, committed.

### Where it lives

This chapter is the last mile. Chapter 6's platform gives you a subdomain; this is how you put your own name on it. Chapter 7's region decision is why a CDN, usually a DNS change, matters for your users in Nashik. Chapter 12's pipeline is where a deploy configuration file belongs. M02 covered DNS from the browser's side; this is the owner's side. M07's uptime monitoring watches both the name and the certificate.

M11's going-live checkpoint asks for a working custom domain with a valid certificate, which makes this the chapter you return to the week before.

### Words to own

| Term | Plain meaning | Example |
|---|---|---|
| Domain / registrar | The name, and who you bought it from | `canteenorder.in` |
| Nameserver | The server that answers for your domain | Set at the registrar |
| A record | Name to IPv4 address | `canteenorder.in` → IP |
| CNAME | Name to another name | `www` → platform host |
| TXT record | Arbitrary text, used for ownership proof | Domain verification |
| TTL | Seconds a cache may keep the answer | Lower it before changes |
| Propagation | Waiting for old cached answers to expire | Up to the old TTL |
| TLS certificate | Proof you control the name, plus encryption | Free, auto-renewed |
| Certificate authority | The body that issues and vouches | Let's Encrypt |
| Apex / root domain | The name with no subdomain | `canteenorder.in` |
| Infrastructure as code | Resources described in text, in git | Terraform, in M13 |
| Drift | Reality no longer matches the written description | Someone clicked |

### Tradeoffs and gotchas

**Lower the TTL before the change, not after.** The most useful sentence in this chapter. Caches took the old value with the old clock; nothing you do now shortens their copy. A day of planning turns a six-hour mystery into a five-minute switch.

**Never announce a domain change until you have checked it from a network you do not control.** Your laptop may have a fresh answer while your users' network provider does not. Check from mobile data, and ask a teammate on a different connection.

**The apex domain has a technical limitation worth knowing by name.** A plain CNAME is not allowed on the root of a domain, which is why platforms offer special record types or ask you to use `www` with a redirect. If an instruction does not work on the apex, this is usually why rather than anything you did wrong.

**An expired certificate is an automation failure, not a certificate problem.** Renewal is automatic. If yours expired, something in the chain stopped — often a DNS record that changed, or a domain validation that can no longer complete.

**Email records are easy to destroy by accident.** Replacing your DNS records wholesale can remove the MX records that deliver your email. Copy the whole record set before you change anything.

**Infrastructure as code has its own foot-gun.** A tool that can create forty resources from a file can delete forty resources from a file. This is a genuine reason to wait until M13 and to learn the plan-before-apply habit properly rather than in a hurry.

### Verify it

DNS and certificates are the area where AI's confident guesses are hardest to distinguish from facts, because everything looks correct until a cache disagrees. Four checks.

1. **Make it tell you what to run, and run it yourself.** DNS state is observable. A lookup command shows the current record and its remaining TTL, and `curl -i` from chapter 4 shows what the name actually serves. An answer about your DNS that was not preceded by a lookup is a guess.
2. **Check the record at the right place.** Confirm which nameservers are authoritative for your domain before concluding a record is wrong. Records added in the wrong console are the most common cause of "I did exactly what it said and nothing happened".
3. **Verify the certificate covers the exact name you are using**, including with and without `www`. Open both in a browser and look at the padlock. A certificate for the platform subdomain does not cover your custom domain.
4. **Before any destructive infrastructure step, make it show you what will be deleted.** For a console action, ask what else the resource is attached to. For a tool, the plan output. "This will remove these three things" is a sentence you must see before agreeing.

Red flags: an instruction to point an A record at a platform's IP address rather than a CNAME to its hostname; advice to "wait for propagation" with no mention of TTL; a suggestion to buy a certificate; a DNS change that replaces your whole record set without preserving MX records; and any infrastructure command run without showing the plan first.

## Walkthrough
1. Find who your registrar is and where your DNS records are actually hosted. They may be the same or different. Write both down — this is the fact that saves you an hour later.
2. List every DNS record currently on your domain, including MX and TXT. Copy the whole list into a file in your repository before you change anything.
3. Ask AI for the read-only lookup command for your operating system, and run it against your domain. Read the record type, the value and the TTL in the output.
4. Note the TTL. If it is an hour or more and you plan a change this week, lower it now to a few minutes and note the time — you must wait out the old value before the new TTL helps.
5. Add your custom domain in your platform's dashboard and read the exact record it asks for. Note whether it wants a CNAME, an apex-specific record type, or a TXT record to prove ownership.
6. Add the record at the correct place — the DNS host, not necessarily the registrar — and then run your lookup command again until you see the new value.
7. Open the domain in a browser and check the padlock. If it warns, find the certificate status in your platform's domain settings and wait for issuance rather than changing anything.
8. Check both `canteenorder.in` and `www.canteenorder.in`, or your equivalents. Confirm one redirects to the other and that both show a valid certificate.
9. Verify from a network you do not control: open it on your phone on mobile data, and ask a teammate on a different connection. Only now is it safe to share the link.
10. Find your certificate's expiry date in the browser's certificate details and write it down. Then confirm your platform renews automatically, and note where you would see a failure.
11. Finally, update `STACK.md`: every resource you own, who created it, its region, and the non-default settings. Then export your environment variable *names* with the values removed and commit that list. That is your free, honest infrastructure as code for now.

## Talk to AI
```prompt
CONTEXT:      I am a student on an AI-native engineering program. I direct AI
              rather than writing code. My team of three is building
              [describe your product in two sentences], deployed on [your
              platform] on a free tier. We have bought the domain [your
              domain] from [your registrar]. Our users are students in India
              on budget Android phones and slow 4G. We have a demo in [number]
              days and the domain must work and show a valid padlock. We also
              have cloud resources that were created by clicking in consoles
              and nobody has written down.

GOAL:         Get our custom domain serving our app over HTTPS with no browser
              warning, without a propagation surprise, and write down our
              infrastructure so a deleted resource can be rebuilt.

REQUIREMENTS: 1. Explain A, CNAME, TXT and MX records and TTL in plain words,
                 and tell me which ones my setup needs
              2. Tell me how to find out whether my registrar or someone else
                 is hosting my DNS, and why adding a record in the wrong place
                 silently does nothing
              3. Give me the exact ordered steps to point my domain at my
                 platform, including the apex versus www question, and tell me
                 what to do about TTL BEFORE I make the change
              4. Explain what a TLS certificate proves and what it does not,
                 and tell me why adding a custom domain needs a new one
              5. Give me the read-only commands to verify the record and the
                 certificate myself, and tell me what a correct result looks
                 like
              6. Tell me what to write down about our existing cloud resources
                 so we could rebuild them, without introducing a new tool

CONSTRAINTS:  Free tier only - do not suggest buying a certificate. Plain
              English, second-language reader. Label every command READ-ONLY
              or CHANGES-THINGS. Preserve our existing email records. Do not
              have me replace my whole record set. We cannot afford downtime
              in the [number] days before our demo, so tell me the risk of
              each step and the point of no return.

NON-GOALS:    Do not introduce Terraform or any infrastructure-as-code tool -
              a text file in our repository is what we want for now. Do not
              set up a CDN, email sending, or subdomains for staging. Do not
              change our hosting platform.

ACCEPTANCE:   - My domain and its www form both load my app with a valid
                padlock, checked on mobile data and on a teammate's network
              - I can state the TTL of my records and what that means for my
                next change
              - My existing MX records still work
              - I know my certificate's expiry date and where a renewal
                failure would show
              - Every cloud resource we own is in a committed file with its
                purpose, region and owner

ASK:          First ask me who my registrar is, where my DNS is hosted, which
                platform I deploy on, and how many days until the demo. Then
              tell me the order of operations and what the riskiest step is,
              before giving me any instruction to carry out.
```

## Stuck? Ask AI
```prompt
Here is the output of a DNS lookup for my domain: [paste the complete output]. Explain every part of it: the record type, the value it points at, the TTL and what that TTL means for me in practice. Then tell me whether this record is pointing where I think it is, and give me the read-only command that would show me what that address actually serves.
```

```prompt
My browser shows a certificate warning on my custom domain, but my platform's own subdomain works fine over HTTPS. Explain why adding a custom domain requires a new certificate, what has to be true for it to be issued, and the three most common reasons issuance gets stuck. For each, tell me the read-only check that would confirm it. Do not tell me to buy a certificate.
```

```prompt
Explain infrastructure as code and then argue honestly against using it for a three-person student team with four cloud resources. Tell me what I would gain, what it would cost me in time and risk, and what a plain text file in my repository should contain to get most of the benefit. Then ask me what resources we actually have so you can tell me what to write down about each one.
```

## Quiz

### Q1 · L3
You change your domain's A record, and one teammate still sees the old site six hours later. Why?
- (a) The new record was saved incorrectly
- (b) Their network's resolver cached the old answer for the record's previous TTL, and you cannot clear it ✅
- (c) Their browser needs a hard refresh
- (d) The certificate has not been issued yet
> Every cache in the chain holds its answer for the TTL it was given. Lowering the TTL after the change does nothing; it has to be lowered a day before, so caches take the short value.

### Q2 · L4
Your platform offers both a CNAME to its hostname and an A record to its current IP address. Which should you choose, and why?
- (a) The A record, because it removes one lookup and is faster
- (b) The CNAME, because platform IP addresses change without notice while the hostname does not ✅
- (c) Either — they are functionally identical
- (d) The A record, because CNAMEs are not supported by most resolvers
> An A record freezes an address the platform may reassign, producing a silent outage with no change on your side. The hostname is the stable thing, so point at the name.

### Q3 · L3
What does a valid TLS certificate actually prove to your user?
- (a) That the site is safe and trustworthy
- (b) That the connection is encrypted and the domain name is verified ✅
- (c) That the site's owner has been identified by name
- (d) That the site does not store personal data
> Encryption plus name verification, nothing more. A phishing site can hold a perfectly valid certificate, which is why the padlock is not a safety judgement.

### Q4 · L4
Your custom domain shows a certificate warning while your platform's subdomain works over HTTPS. What is the most likely cause?
- (a) The platform's certificate expired
- (b) Certificates are issued per name, and one has not yet been issued for your new domain ✅
- (c) You need to buy a certificate for a custom domain
- (d) The A record points at the wrong IP
> The existing certificate covers the platform subdomain only. Adding a domain triggers issuance for that name, which requires the DNS record to be resolving first — and certificates on these platforms are free.

### Q5 · L5
Why does this course tell a three-person team to keep a committed text file describing their cloud resources rather than adopt an infrastructure-as-code tool?
- (a) Infrastructure as code does not work on free tiers
- (b) The file captures most of the value — nothing unidentifiable, rebuildable from a record — without the setup cost or the risk that one careless apply deletes a database ✅
- (c) Text files are automatically applied by most platforms
- (d) The tools are only useful once you run a cluster
> The discipline is needed now; the tool is not. A tool that can create forty resources from a file can also destroy them, which is a real reason to learn it properly in M13 rather than in a hurry before a demo.

## Go deeper
- [Cloudflare Learning: DNS records explained](https://www.cloudflare.com/learning/dns/dns-records/)
- [Let's Encrypt: how certificate issuance and renewal work](https://letsencrypt.org/how-it-works/)
