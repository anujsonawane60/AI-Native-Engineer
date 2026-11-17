---
id: m16-11-privacy-dpdp-gdpr
module: m16-users-analytics-growth
slug: privacy-dpdp-gdpr
title: "Privacy, Consent, DPDP and GDPR With Real User Data"
order: 11
duration_min: 15
is_free: false
mission: false
volatility: medium
last_reviewed: 2026-11-16
---

## Key learnings
- Draw a data map of every place a user's personal data actually sits
- Store a consent record you could show someone, with a notice version and a time
- Run one erasure request end to end and name the places it is impossible
- Handle under-18 users, which a campus product certainly has
- Read your numbers knowing that consent refusal shrinks and biases them

## Lesson

### The pain

A team is three weeks from demo day with six hundred users. Somebody asks: if a user emailed tomorrow asking you to delete everything you hold about them, could you?

They start listing. Users and orders tables. Server logs with phone numbers in request bodies. The analytics and replay tools, both companies abroad. A warehouse table. Chapter one's WhatsApp chats, on three phones. A CSV a teammate emailed himself. Yesterday's backup. Twelve places; they can delete from four. And their consent was a pre-ticked box with no record of when anybody agreed, or to what.

### What M07 covered, and what is new here

M07 taught the law: free, specific, informed consent by clear affirmative action; purpose limitation and minimisation; access, correction and erasure; parental consent under eighteen; GDPR once you have EU users. New here is doing that with six hundred real people and three vendors abroad.

### The idea

**Compliance is not a banner. It is a map of where personal data sits, a record of what each person agreed to, and an erasure procedure you have actually run at least once.**

Consent is a thing that happened, so it is recorded, not merely requested: who agreed, when, which version of the notice they saw, and to what. If your notice covered showing users their past orders and you later send behavioural reminders, consent does not cover the new purpose — that is purpose limitation, and the remedy is a new notice and fresh consent, which you can only aim correctly if you stored the version. Record refusals too, and make consent withdrawable with a settings toggle, not an email.

### The analogy

The canteen asks students to sign a register for the meal plan, kept in a drawer behind the counter.

**Where the analogy breaks completely:** if a student asks for their page, the manager tears it out and hands it over. Your user's data is in a dozen places: three companies abroad, a backup that cannot be edited, a chat on a teammate's phone. Nothing here is a drawer, which is why the map comes before the policy.

### Picture it

```
  DATA MAP - one user, every place they exist

  store           what                 keep    can erase?
  ───────────────────────────────────────────────────────
  users table     phone, name          2 yr    yes
  orders table    order rows           2 yr    yes, anonymise
  server logs     phone in request     30 d    yes, if rotated
  analytics       user id + events     12 mo   via vendor API
  replay tool     recordings           30 d    via vendor UI
  warehouse       copied event rows    2 yr    yes, manual
  WhatsApp chats  name, phone, quotes  ?       on 3 phones
  teammate CSV    full export, wk 2    ?       NO - untracked
  DB backup       everything, nightly  7 d     NO - expires
  ───────────────────────────────────────────────────────
  erasable today: 6 of 9        honest gaps: 3
```

A data map lists every store, its personal data, its retention and whether one person can actually be removed from it — and the honest answer is never "all of them".

### Where it lives

Everything you send to an analytics or replay tool is processed by a company on your behalf. List each vendor, what you send, which country it sits in and their retention: under the DPDP Act transfer abroad is generally permitted except to restricted countries, and GDPR adds rules for EU users. Write one page on what you do if data leaks, since the Act requires notifying the Board and those affected.

Earlier chapters are this one applied in advance — chapter two's forbidden fields, chapter five's masking, chapter one's log on a personal phone — and M18 asks whether you could delete a user, in front of people.

**Tools today.** DPDP's rules arrived on shifting timelines, so check any deadline yourself; nothing here is legal advice.

### Words to own

| Term | Plain meaning | Example |
|---|---|---|
| Data map | Every place personal data sits, and for how long | The diagram above |
| Consent record | Stored proof of who agreed to what, when | User, time, notice v2 |
| Notice version | Which wording a person actually saw | v1 did not cover reminders |
| Purpose limitation | Using data only for the stated reason | New purpose, new consent |
| Processor | A vendor handling data on your behalf | Your analytics tool |
| Sub-processor | A vendor your vendor uses | Their cloud host |
| Retention period | How long a store keeps it | Logs 30 days |
| Erasure request | A person asking to be deleted | Run it once in advance |
| Age gate | Asking age to apply the child rules | Flag suppresses tracking |
| Breach notification | Telling the Board and the people affected | One page, written early |

### Tradeoffs and gotchas

**Nothing may load before consent** — not the analytics snippet, not the replay recorder, not a vendor's font. A banner that fires trackers before the tap collects the data anyway and documents that you knew permission was needed. A pre-ticked box, or a coloured Accept beside a grey Reject, is built to produce yes.

**You have under-18 users, so decide deliberately.** A college product has first-years who are seventeen; a child's data needs verifiable parental consent, and behavioural tracking of children is prohibited. Either ask for age and set a flag that suppresses analytics and replay, or stay ignorant by accident, which is no defence.

**Run one erasure before anybody asks, and write down where you failed.** A nightly backup cannot be edited, so record that it expires in seven days; one untracked CSV defeats every other control. A procedure with named gaps is real; one with none has never been run.

**Refusal shrinks and biases your data.** People who decline tracking are not a random slice, so every funnel and cohort in this module rests on a self-selected subset; say so when you present numbers.

### Verify it

Five checks, each done by you on the live product, not read from a configuration.

1. **Open the app in a fresh browser, network tab recording, and touch nothing.** Any analytics or replay request before consent is the red flag.
2. **Record a fake password and OTP, then watch your own replay.** An unwatched setting is not a guarantee.
3. **Ask to see one stored consent record**, with a person, a time and a notice version.
4. **Run one erasure end to end and count the stores you could not clear.**
5. **Check what happens to a user who says they are seventeen.**

Red flags: a pre-ticked box; Reject smaller than Accept; an erasure procedure that stops at the database.

## Walkthrough
1. List every place a user's personal data could sit, including your own phones, laptops, exports, backups and every third-party tool. Aim for more than ten entries; fewer means you have not looked.
2. For each store, write what personal data it holds, why it is there, how long it is kept, and whether one individual can be removed from it. Mark the honest "no" rows.
3. List every vendor you send data to, what you send, and which country it sits in. Read each vendor's stated retention and write it beside them.
4. Open your live app in a fresh browser with the network tab recording and touch nothing. Write down every request that fires before you interact with any consent control.
5. Read every field of one real analytics event and one real replay frame. Note anything that identifies a person, including a third party named in free text.
6. Check your consent record: find the stored row for one real user, with a time and a notice version. If it does not exist, design it.
7. Read your consent notice aloud to someone outside your team and ask them to tell you what you collect and why. Rewrite anything they could not repeat back.
8. Check the consent design for dark patterns: pre-ticks, unequal button styling, and how many taps refusing takes compared with accepting.
9. Pick one real user id and run the erasure for real, across every store on your map. Record what you cleared and what you could not.
10. Decide your under-18 position and implement it: either an age flag that suppresses behavioural tracking and replay, or a written statement of why you do not collect age and what that exposes you to.
11. Write the one-page breach plan — who finds out, who is told, what users are told, when — and commit the data map, the consent record design, the erasure procedure with its gaps, and the plan together with today's date.

## Talk to AI
```prompt
CONTEXT:      I am a student in a team of three in India. I direct AI and
              verify its output; I do not write code. Our product is live
              with [number] real users: [describe it in two sentences]. The
              personal fields we collect are: [list every one]. The places
              data sits, as far as I know: [list them, including our own
              phones, laptop exports, backups, analytics, replay tool,
              warehouse, WhatsApp chats]. Vendors we send data to: [list
              them]. Our users are college students in India and some are
              under 18. Our URL is public so we may have EU visitors. Our
              current consent setup is: [describe it honestly, even if it
              is a pre-ticked box or nothing]. India's DPDP Act applies to
              us; GDPR may apply to EU visitors. You are not my lawyer and
              I am not treating this as legal advice.

GOAL:         A data map, a consent record design, an erasure procedure I
              have actually run with its gaps named, an under-18 position
              and a one-page breach plan.

REQUIREMENTS: 1. Turn my list into a data map: store, what personal data,
                 why, retention period, and whether one individual can be
                 erased. Then tell me which stores I have probably
                 forgotten - think about exports, backups, support chats
                 and screenshots.
              2. Design the consent record: the exact fields to store,
                 including notice version, and explain what breaks later if
                 I do not store the version.
              3. Tell me what must not load before consent in my setup, and
                 the exact browser check I run to prove it in under a
                 minute.
              4. Audit my current consent design for dark patterns -
                 pre-ticks, unequal buttons, asymmetric tap counts - and
                 rewrite my notice in at most six sentences a first-year
                 student could repeat back.
              5. Give me the step-by-step erasure procedure across every
                 store on my map, and name in advance the stores where it
                 will be impossible and what I should write down instead.
              6. Give me my two honest options for under-18 users, what
                 each one requires me to build, and what each one leaves me
                 exposed to. Then tell me which one you would choose and
                 why.
              7. For each vendor, tell me what I need to find out and where
                 to look: what they hold, which country, their retention,
                 and whether they use sub-processors.
              8. Write the one-page breach plan: who finds out first, who I
                 must notify, what I tell users, and in what order.
              9. Tell me how consent refusal changes the funnel and cohort
                 numbers I have been computing all module, and the sentence
                 I should say when presenting them.
             10. Name the three fields I collect that I could stop
                 collecting entirely, and what I would lose.

CONSTRAINTS:  Free tier only. Plain English - translate every legal term on
              first use. Where the law has changed or is uncertain, say so
              and point me at the primary source rather than guessing;
              DPDP's rules have been phased in and I need to check the
              current position myself. Do not tell me I am "compliant" -
              give me checks, not verdicts. Do not recommend a
              consent-management vendor.

NON-GOALS:    Do not write code. Do not design my event taxonomy. Do not
              draft terms of service or a privacy policy in legal language.
              Do not modify any files.

ACCEPTANCE:   - My data map has more than ten stores and marks the
                non-erasable ones
              - My consent record stores a notice version
              - I know the one-minute browser check for pre-consent loading
              - My notice is six sentences or fewer and plain
              - My erasure procedure names its own gaps
              - I have a deliberate under-18 position, not an accidental
                one
              - I can state how refusal biases my numbers

ASK:          First ask me up to five questions about my fields, my stores,
              my vendors and my users' ages. Then answer requirement 1
              only, and stop.
```

## Stuck? Ask AI
```prompt
Pick one real user id and walk me through deleting that person from every place they could exist in a small student product: database, server logs, analytics vendor, session replay tool, warehouse copy, WhatsApp chats on team phones, laptop exports and nightly backups. For each one, tell me the concrete step - and for the ones where it is impossible, tell me exactly what I should write down instead of pretending.
```

```prompt
Explain, in plain English for a second-year student, what the DPDP Act requires of me specifically because some of my college users are under 18. Then give me my two realistic options, what each costs me to build, and what each leaves me exposed to. Tell me where to check the current status of the rules rather than relying on your memory.
```

```prompt
Here is my consent banner design: [describe it, including button colours, wording and how many taps refusing takes]. Audit it for dark patterns against the requirement that consent be free, specific, informed and a clear affirmative action. Then rewrite my notice in six plain sentences and tell me what must not load before the user taps.
```

## Quiz

### Q1 · L4
Your consent banner appears on page load and your analytics snippet loads at the same moment. Why is this worse than having no banner at all?
- (a) It slows the first load on slow connections
- (b) The data is processed without permission, and the banner documents that you knew permission was needed ✅
- (c) Banners are not required for products with under a thousand users
- (d) It reduces the sample size of your experiments
> Consent must precede processing and be a clear affirmative action. A banner that fires trackers before the tap collects the data anyway and leaves written evidence that the requirement was understood.

### Q2 · L5
You run an erasure request and find you can clear six of nine stores. What is the correct outcome?
- (a) Report the request as fulfilled, since the main stores were cleared
- (b) Record the procedure with the three gaps named, including the backup's expiry ✅
- (c) Delay all erasure requests until every store is clearable
- (d) Delete the backup entirely so the gap disappears
> A procedure with named gaps is a real procedure; one with none has not been run. The backup expiring in seven days is a limit you document, and the untracked CSV is why exports get banned from that point on.

### Q3 · L4
Your notice said order history is used to show users their past orders. Six weeks later you start sending behavioural reminders based on that history. What does the law require?
- (a) Nothing; the data was already collected with consent
- (b) A new notice and fresh consent, because the purpose has changed ✅
- (c) An update to your retention period only
- (d) Notification of the Data Protection Board
> This is purpose limitation. Consent is specific to a stated purpose, so a new purpose needs new consent — and you can only tell who needs re-consenting if the consent record stores which notice version each person saw.

### Q4 · L3
A first-year user tells you they are seventeen. Under the DPDP Act, what changes?
- (a) Nothing, as long as consent was collected
- (b) Behavioural tracking of them is prohibited and processing needs verifiable parental consent ✅
- (c) You must delete their account
- (d) Only marketing emails are restricted
> A campus product certainly has under-18 users, so this is routine rather than exceptional. The practical implementation is a flag that suppresses behavioural analytics and replay for that account.

### Q5 · L4
A teammate suggests moving tracking to your server so the consent banner no longer costs you data. What is wrong with this?
- (a) Server-side events are less accurate than client events
- (b) It changes who delivers the data, not whether you had permission to process it ✅
- (c) It would break sticky assignment in experiments
- (d) Nothing; server-side collection is exempt
> Consent governs processing, not the transport. Chapter two moves money events server-side for accuracy, which is a different motive; doing it to escape a refusal is circumventing permission you were required to obtain.

## Go deeper
- [MeitY: Digital Personal Data Protection Act and the notified rules](https://www.meity.gov.in/data-protection-framework)
- [EDPB: guidelines on consent under the GDPR](https://www.edpb.europa.eu/our-work-tools/general-guidance/guidelines-recommendations-best-practices_en)
