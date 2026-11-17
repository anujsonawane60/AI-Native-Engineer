---
id: m07-09-ab-tests-privacy-consent
module: m07-security-observability-analytics
slug: ab-tests-privacy-consent
title: "A/B Tests, Significance, Privacy and Consent"
order: 9
duration_min: 15
is_free: false
mission: false
volatility: low
last_reviewed: 2026-11-16
---

## Key learnings
- Decide, before running anything, whether you have the traffic for an A/B test
- Say what statistical significance claims and what it does not
- Name the three design choices that make an experiment result trustworthy
- Apply the DPDP Act's consent requirements to your own analytics
- Spot an analytics setup that collects personal data it has no right to

## Lesson

### The pain

A team changes their Order button from grey to orange. Over two days, 23 of 190 visitors order with orange against 18 of 182 with grey — a 22% relative improvement. They ship orange and put it on a slide. Nothing changes: the difference was five people, which is what two coin flips of 190 tosses differ by on an ordinary Tuesday.

Meanwhile their analytics snippet had been sending the whole signup form to a vendor's server, phone numbers included, with no notice and no consent — the more expensive mistake, and the one nobody looked at.

### The idea

**An experiment is a comparison you designed before you looked at the data, and significance is a statement about how surprising your difference would be if nothing were really happening.**

Decide the metric, the groups, the duration and the threshold in advance and the result means something. Decide any of them afterwards and you have found a pattern in noise.

### The analogy

The canteen wants to know whether a new samosa recipe sells better. The wrong experiment is new recipe Monday, old recipe Tuesday: Monday was a holiday and Tuesday had rain, so the difference measures the week. The right experiment runs both the same day at two counters, students sent to one by a coin flip and always to the same counter, measuring plates per hundred students for two weeks and looked at once at the end.

**Where the analogy breaks:** in the canteen you can watch faces and ask why. In software you get a number and nothing else, which is why the design must be right before you start.

### Picture it

```
  1,000 students arrive        assignment is sticky:
            │                  the same student always
      ┌─────┴─────┐            gets the same variant
      ▼           ▼
  ┌─────────┐ ┌─────────┐
  │    A    │ │    B    │
  │   500   │ │   500   │
  │ control │ │ variant │
  └────┬────┘ └────┬────┘
       │           │
   52 orders   61 orders
    10.4%       12.2%
       └─────┬─────┘
             ▼
   is +1.8 points real, or luck?
   at this baseline you need
   roughly 3,800 per group
```

A thousand students split evenly produce a difference of nine orders, which looks like a result but sits well inside what chance produces — detecting a move from 10% to 12% needs some 3,800 per group.

The number you will be shown is a **p-value**: *if there were truly no difference, how often would a gap this large turn up by chance?* It does **not** mean a 97% chance your change works, and above 0.05 usually means the sample was too small to tell. Decide the **minimum detectable effect** first — halving it roughly quadruples the people needed — compute the **sample size** before you start, and never **peek**.

**So most student teams should not run A/B tests,** which is a conclusion, not a failure. With a few hundred users, watch chapter eight's funnel, sit with five people, and change what confused them.

### Privacy and consent

Analytics and experiments both run on people. Under **India's Digital Personal Data Protection Act, 2023**, you are a **data fiduciary** and your user a **data principal**.

| DPDP obligation | What it means for your product |
|---|---|
| Notice and consent | A plain-language notice before processing, and consent that is free, specific, informed and actively given — a pre-ticked box is not consent |
| Purpose limitation | Use the data for the stated purpose, not one you thought of later |
| Data minimisation | Collect what the purpose needs, not the whole form |
| Rights | The principal can ask what you hold, correct it, and have it erased |
| Children | Under-18 data needs verifiable parental consent; behavioural tracking of children is prohibited |

A college app will have first-year users under 18, so the last row is not theoretical. **GDPR** applies the moment you have EU users, which a public URL may acquire accidentally, and analytics cookies there need consent before loading. For your setup: user id in events, never phone number or email; no analytics script before consent, which generated banners get wrong; IPs truncated; a retention period set.

### Where it lives

Chapter eight gave you the events; this chapter is what you may do with them and who you must ask first.

- M04 — where the personal data lives and erasure must be answerable
- Chapter 5 — the never-log list, the same principle for logs
- M11 — the going-live checkpoint, where a consent notice stops being optional
- M16, Company tier — a real experiment programme, DPDP and GDPR with real users

### Words to own

| Term | Plain meaning | Example |
|---|---|---|
| Control / variant | The existing version, and the change | Grey button, orange button |
| Sticky assignment | The same person always sees the same variant | Hashed on user id |
| Primary metric | The one number decided in advance | Orders per visitor |
| Guardrail metric | A number that must not get worse | Error rate, refunds |
| p-value | How often chance alone would produce this gap | 0.03 |
| Minimum detectable effect | The smallest difference you could notice | 2 percentage points |
| Peeking | Checking repeatedly and stopping when it looks good | The commonest false result |
| Data principal | The person the data is about | Your user |
| Data fiduciary | Whoever decides how the data is used | You |
| Data minimisation | Collect only what the purpose needs | User id, not phone number |

### Tradeoffs and gotchas

- **Do not A/B test what you can decide by judgement** — wording, spacing, icons. Taste is cheaper than statistics.
- **Twenty metrics means one false winner** at a 5% threshold. One primary metric decided in advance, two or three guardrails, the rest questions for next time. And novelty fades, so distrust a lift that lives entirely in days one and two.
- **Reassignment destroys an experiment silently.** A variant chosen freshly on every page load puts one person's behaviour in no group at all; assignment must come from a hash of the user id.
- **Consent refusals shrink and bias your sample:** people who refuse tracking are not a random slice.
- **Experimenting on people has an ethical floor, not only a legal one.** No different prices for different students, no variant hiding a safety warning. If you would not describe the test to the person in it, do not run it.

### Verify it

1. **Ask what the primary metric is and when it was chosen.** If it came after the data, the result is a story.
2. **Check that assignment is sticky:** reload ten times and confirm the same variant. **Assignment recomputed on every render** is the classic generated-code flaw.
3. **Ask for the sample size calculation and the planned end date.** No calculation means nobody knows what the test could detect; no end date invites peeking.
4. **Watch the network tab before you touch the consent banner.** If a tracker fires first, the banner is decoration.
5. **Inspect the payload of one analytics event, field by field.** **A phone number, email or full name in an event property** is the red flag, usually because a snippet sent the whole form object.
6. **Try to answer an erasure request.** Work out how you would delete one user id from your database, your logs and your analytics tool. If you cannot, that obligation is unmet.

Red flags: significance "achieved" after daily checking; a result based on single-digit people; no primary metric named in advance; variants reassigned per request; analytics loading before consent; personal data in event properties; a retention policy of "we keep everything".

## Walkthrough
1. Take a change your team is currently arguing about and write it as a hypothesis in one sentence: "Changing X will increase Y because Z."
2. Name the single primary metric for that hypothesis, and two guardrail metrics that must not get worse. Write all three down before going any further.
3. Find your current rate for that primary metric from chapter eight's funnel. This is your baseline, and without it nothing else can be computed.
4. Decide the smallest difference that would actually change your decision. Not the smallest you could imagine — the smallest you would act on.
5. Have AI compute the sample size per group for that baseline and that effect, and ask it to show the inputs it used. Compare the answer with how many users you will realistically have this term.
6. If the answer is more users than you will have, stop and write one sentence saying so. Then design the alternative: which five people you will watch, and which funnel step you will check afterwards.
7. Either way, now audit your analytics. Open your deployed app in a fresh browser, open the network tab, and record every request that fires before you interact with any consent banner.
8. Click into one analytics event and read every field in its payload. Write down anything that identifies a person.
9. Find your consent notice. If there is none, draft one in plain English: what you collect, why, how long you keep it, and how someone asks for deletion. Four sentences is enough to start.
10. Pick one real user id and write the exact steps you would take to erase that person from your database, your logs and your analytics tool. Note every place where you cannot.
11. Put the hypothesis, the sample-size verdict and the erasure procedure in your repository, and note in your decision log what you chose not to test and why.

## Talk to AI
```prompt
CONTEXT:      I am a student in a team of three. I direct AI and verify its
              output; I do not write code. Our product: [describe it in two
              sentences]. We have [number] users, roughly [number] visits a
              week. Our users are students in India; some may be under 18,
              and our URL is public so we may get EU visitors. We collect
              these personal fields: [list them]. Our current conversion
              rate on [the action we care about] is [number]%. We use
              [analytics tool, or "none yet"]. India's DPDP Act applies to
              us; GDPR may apply to EU visitors.

GOAL:         An honest answer to whether we can run a meaningful A/B test
              at our size, and a privacy and consent setup for our
              analytics that would survive being read by a regulator.

REQUIREMENTS: 1. Given my baseline rate, compute the sample size per group
                 needed to detect a 2-point and a 5-point absolute change.
                 Show your inputs and assumptions. Then tell me plainly
                 whether my traffic supports either.
              2. If it does not, design the alternative: how many people I
                 should watch using the product, what I should look for, and
                 which single funnel number I should check afterwards.
              3. Explain a p-value in one paragraph, then give me three
                 sentences people commonly say about significance that are
                 wrong, and correct each one.
              4. Explain sticky assignment, how I can test it myself in a
                 browser in under a minute, and what exactly goes wrong if
                 the variant is recomputed on every page load.
              5. For my field list, tell me which fields may appear in an
                 analytics event, which must never, and what to send
                 instead.
              6. Draft a plain-English consent notice of at most six
                 sentences covering what I collect, why, how long I keep it,
                 and how someone requests deletion. Tell me where it must
                 appear and what must not load before consent is given.
              7. Tell me what the DPDP Act requires specifically because
                 some of my users may be under 18.
              8. Give me the step-by-step procedure for honouring one
                 erasure request across my database, my logs and my
                 analytics tool, and name the places where this is usually
                 impossible.

CONSTRAINTS:  Free tiers only. Explain every term on first use. Plain
              English, no legal jargon without a translation. Where the law
              is genuinely uncertain or has changed, say so and point me at
              the primary source rather than guessing. You are not my
              lawyer and I am not treating this as legal advice.

NON-GOALS:    Do not design the event taxonomy; I have one. Do not write
              code. Do not recommend a consent-management vendor. Do not
              tell me we are "compliant" — give me checks, not verdicts.

ACCEPTANCE:   - I know whether an A/B test is possible at my size, with the
                arithmetic shown
              - I can correct three wrong statements about significance
              - I can test sticky assignment myself in under a minute
              - My consent notice is six sentences or fewer and a
                first-year student could understand it
              - I can state what must not load before consent
              - I have a written erasure procedure, including its gaps

ASK:          First ask me up to five questions whose answers would change
              your advice, including anything about my users' ages and
              locations. Then answer requirement 1 only, with the
              arithmetic visible, and stop.
```

## Stuck? Ask AI
```prompt
Explain a p-value to me as if I have never studied statistics, using a canteen taste test with two samosa recipes. Then give me five sentences that people commonly say about significance, where some are correct and some are wrong, and ask me to sort them before you tell me the answers.
```

```prompt
Explain what "peeking" means in an A/B test and why it produces false results. Then walk me through a concrete example: show me what happens to the apparent significance if I check the numbers every day for twenty days on two groups that are genuinely identical.
```

```prompt
Explain India's DPDP Act to me in plain English as it applies to a student product with a few hundred users: what notice means, what counts as valid consent, what purpose limitation stops me doing, and what rights my users have. Then tell me the three things a student team most commonly gets wrong, and how I would check each one in my own app today.
```

## Quiz

### Q1 · L4
Your test shows a p-value of 0.03. Which statement is correct?
- (a) There is a 97% chance the change works
- (b) If there were truly no difference, a gap this large would occur about 3% of the time ✅
- (c) The change improves conversion by 3%
- (d) The result will hold for at least three months
> A p-value describes how surprising your data would be under the assumption of no effect. It says nothing about the size of the effect, nothing about whether it persists, and it is not the probability that you are right.

### Q2 · L5
With 190 visitors per group and a 10% baseline, your variant shows a 2-point lift. What should you conclude?
- (a) Ship it; a 20% relative improvement is substantial
- (b) Nothing — at this size the sample cannot distinguish a 2-point lift from chance ✅
- (c) That the control is broken and should be investigated
- (d) Run it another two days and ship if it holds
> Detecting a 2-point move at a 10% baseline needs thousands per group. Option (d) is worse than (a), because stopping as soon as it looks good is exactly the peeking that manufactures false winners.

### Q3 · L4
AI builds your A/B test so the variant is chosen randomly on each page load. What breaks?
- (a) Nothing, as long as the split stays near 50/50
- (b) Users move between groups, so neither group measures a consistent experience ✅
- (c) The p-value is computed on too small a sample
- (d) The control group will be systematically larger
> Assignment must be sticky — derived from a stable value like a hash of the user id. Reassignment blends the groups, invalidates the comparison, and produces no warning of any kind.

### Q4 · L4
Your consent banner appears on load, and the analytics script loads at the same time. Why is this worse than having no banner?
- (a) It slows the page down for no benefit
- (b) The data is collected anyway, plus a false claim of having asked ✅
- (c) Banners are not required under the DPDP Act
- (d) It reduces the sample size of your experiments
> Consent must precede processing and must be a clear affirmative action. A banner that fires trackers before the click collects the data without permission and documents that you knew permission was needed.

### Q5 · L3
Which field is acceptable inside an analytics event property?
- (a) The user's phone number, for matching across devices
- (b) An internal user id ✅
- (c) The user's email address, hashed for privacy
- (d) The full signup form, so nothing is lost
> Minimisation means sending the least that answers your question, and an internal id does that while keeping the mapping inside your own database. A hashed email is still personal data, and sending the whole form is how phone numbers end up at a vendor.

## Go deeper
- [Evan Miller: How Not To Run An A/B Test](https://www.evanmiller.org/how-not-to-run-an-ab-test.html)
- [PRS India: Digital Personal Data Protection Act, 2023](https://prsindia.org/billtrack/digital-personal-data-protection-bill-2023)
