---
id: m16-11-privacy-dpdp-gdpr
module: m16-users-analytics-growth
slug: privacy-dpdp-gdpr
title: "Privacy, Consent, DPDP and GDPR With Real User Data"
order: 11
duration_min: 30
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

A team is three weeks from demo day with six hundred real users. Somebody asks one question: if a user emailed tomorrow asking you to delete everything you hold about them, could you?

They start listing. The users and orders tables. The server logs, which contain phone numbers in request bodies. The analytics tool, a company in another country. The replay tool, holding four hundred recordings including one where a masking rule was wrong. A warehouse table built for a dashboard. The WhatsApp chats from chapter one's recruitment, on three personal phones. A CSV a teammate exported in week two and emailed to himself. Yesterday's database backup. A mission screenshot with a real phone number in it.

Twelve places. They can delete from four. The consent they collected was a pre-ticked box with no record of when anybody agreed or to what, so they cannot even show what they were permitted to do.

### What M07 covered, and what is new here

M07's chapter nine taught the law: data fiduciary and data principal; notice and consent that is free, specific, informed and a clear affirmative action; purpose limitation; minimisation; rights of access, correction and erasure; verifiable parental consent under eighteen and no behavioural tracking of children; and GDPR the moment you have EU users. Re-read it for the definitions; this chapter does not restate them.

What is new is doing it with six hundred real people, a replay tool, three vendors in other countries and chats on your own phone. The data map. A consent record you could produce. One erasure run end to end. An age gate on a college product. And what refusal does to every number in this module.

### The idea

**Compliance is not a banner. It is a map of where personal data sits, a record of what each person agreed to, and a procedure you have actually run at least once.**

The banner is the part everyone builds and the part that proves nothing. Those three artefacts are what "can you delete this person?" is answered with.

### The analogy

The canteen asks students to sign a register for the monthly meal plan: name, room number, phone. The register lives in a drawer behind the counter.

**Where the analogy breaks completely:** the register is one book in one drawer, and if a student asks for their page back the manager tears it out and hands it over. Your user's data is in a dozen places, three owned by companies in other countries, one a backup that cannot be edited without restoring it, one a chat on a teammate's phone. Nothing about your situation is a drawer, which is why the map comes before the policy.

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

### The consent record

Consent is a thing that happened, so it has to be recorded, not merely requested. Store four fields when someone agrees: who, when, which version of the notice they saw, and what they agreed to.

The version matters more than teams expect. If your notice said "we use your order history to show you your past orders" and six weeks later you start sending behavioural reminders, your consent does not cover the new purpose — that is purpose limitation, and the remedy is a new notice and fresh consent, which you can only identify as necessary if you know which version each person saw.

Two further rules. A refusal is recorded as deliberately as an acceptance, because an unanswered banner is not consent. And consent must be withdrawable as easily as it was given — a settings toggle, not an email.

### Consent that works on a phone on slow data

**Nothing may load before consent.** Not the analytics snippet, not the replay recorder, not a vendor's font. This check takes thirty seconds in a network tab and almost nobody runs it. A banner that fires trackers before the tap is worse than no banner: it collects the data anyway and documents that you knew permission was needed.

**The banner must not break the product on a slow connection.** A full-screen modal weighing two hundred kilobytes that renders after four seconds on 3G is an activation killer as well as a legal artefact.

**Dark patterns are not consent.** A pre-ticked box, a coloured Accept beside a smaller grey Reject, or a flow where refusing takes three taps and accepting takes one. A design built to produce yes is not a free and informed affirmative action.

### Running one erasure, for real

Pick a real user id and do it, once, before anybody asks: delete or anonymise the rows in your database; purge or rotate the logs holding the identifier; call the analytics vendor's deletion endpoint; delete the replay recordings; delete the copied warehouse rows; delete the chat and contact on every team phone; delete the CSV, if you can find it.

Then write down where you failed. A nightly backup cannot be edited, so the honest answer is that it expires in seven days and you record that limit. A replay somebody has already watched cannot be unwatched.

Writing the gaps down is the deliverable. A procedure with three named gaps is real; one with none has never been run.

### Under-18 users, which you have

A college product has first-year students who are seventeen. That is not an edge case: processing a child's data requires verifiable parental consent, and behavioural tracking aimed at children is prohibited outright.

Two honest options. Ask for age and, for anyone under eighteen, run no behavioural analytics or replays on them — a flag that suppresses tracking. Or do not collect age, in which case you do not know, and not knowing is not a defence when your users are visibly a college cohort. The second is what most teams pick by accident. Pick the first deliberately, and write down what the flag suppresses.

### Vendors, countries and a breach plan

Everything you send to an analytics or replay tool is processed by a company on your behalf, and that company may use others. Write down each vendor, what you send, which country the data sits in, and their stated retention. Under the DPDP Act transfer abroad is generally permitted except to countries the government restricts; GDPR adds its own requirements for EU users. The list takes twenty minutes.

Then one page on what you do if data leaks: who finds out first, who is told, what users are told and when. The Act requires notifying both the Data Protection Board and affected individuals, and a plan written in advance is the difference between a bad week and a disaster.

**Tools today.** The Act passed in 2023 and its rules have been phased in since, on timelines that have moved more than once. Treat anything here about deadlines, thresholds or the Board's procedures as a pointer to check, not settled fact. Nothing here is legal advice.

### Where it lives

Chapter two's forbidden-fields line and chapter five's masking are this chapter's rules applied earlier. Chapter one's WhatsApp recruitment log is personal data on a personal phone, which teams never classify as such. Chapter ten's exit interviews collect fresh personal data from people who have left.

M04 is where the rows live and where erasure must be answerable. M07's never-log list is the same discipline for logs. M13's warehouse makes a second copy of everything, which is a second place to erase from. M17's legal chapter is the company-level version, and M18 is where you are asked whether you could delete a user, in front of people.

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

**Refusal shrinks and biases your data, and there is no legitimate way around it.** People who decline tracking are not a random slice, so every funnel and cohort in this module rests on a self-selected subset. Say so when you present numbers.

**Server-side events are not a consent loophole.** Moving a tracker to your backend changes who delivers the data, not whether you had permission to process it.

**"We will add consent before launch" is already too late if you are live.** The data you collect today is collected under whatever notice exists today.

**Untracked exports are the hardest gap to close.** One CSV on one laptop defeats every other control you have. Ban them, and find the existing one first.

**Minimisation is cheaper than protection.** A field you never collect cannot leak, cannot be asked for and cannot be fined over. The best privacy work in a student product is deleting a form field.

**Your own missions contain personal data.** Recruitment logs, replay stills, interview quotes. Initialise names, blur numbers, treat submissions as publication.

### Verify it

Six checks, each done by you on the live product, not read from a configuration.

1. **Open the app in a fresh browser with the network tab recording and touch nothing.** **Any analytics or replay request firing before consent** is the most common red flag.
2. **Read every field of one real analytics event.** A phone number, email, name or typed user note is a breach, usually there because a generated snippet sent the whole object.
3. **Record yourself typing a fake password and OTP, then watch your own replay.** **A replay that captured a password or OTP field** is the red flag here; an unwatched setting is not a guarantee.
4. **Ask to see one stored consent record.** Without a row carrying a person, a time and a notice version, you cannot show what you were permitted to do.
5. **Run one erasure end to end and count the stores you could not clear.** No named gaps means it has not been run.
6. **Check what happens to a user who says they are seventeen.** "Nothing different" means you are behaviourally tracking children.

Red flags: a pre-ticked consent box; Reject styled smaller than Accept; a notice in legal English a first-year could not read; no retention period on any store; no vendor list; replays kept "until the quota rolls over"; an erasure procedure that stops at the database; and a notice that changed without anyone re-consenting.

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
