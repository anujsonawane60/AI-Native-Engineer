---
id: m15-01-distribution
module: m15-content-social-automation
slug: distribution-is-the-other-half
title: "Distribution Is the Product's Other Half"
order: 1
duration_min: 15
is_free: false
mission: false
volatility: low
last_reviewed: 2026-11-16
---

## Key learnings
- Treat distribution as a system you design and measure, not an event you hope for
- Tell a compounding channel from a spiky one and budget your weeks accordingly
- Pick two channels from evidence about your users instead of posting everywhere
- Name the one number that proves a channel is working for your product
- Explain what owned, earned and paid surfaces mean when your ad budget is zero

## Lesson

### The pain

Two teams from the same batch demo in week eleven, both with working, deployed products. The first has 9 users: seven classmates, a cousin, and a mentor who signed up to be kind. They spent eleven weeks on the product and the last Sunday night on "marketing" — one post from an account with 14 followers. The second has 340 users, because in week four they wrote eleven answers to the exact questions their users were typing into Google, and from week five posted one 40-second vertical video every weekday. By week eleven four of those pages brought 20 to 30 visits a day untouched. The first team believed distribution was a thing you do at the end.

### The idea

**A product nobody can find does not exist. Distribution is a system with inputs, queues, failure modes and measurements, and it has to be built in parallel with the product, not after it.**

This module treats growth as engineering: a sitemap that matches reality, a template that turns a dataset into hundreds of honest pages, a posting queue that respects rate limits, a measurement scheme that does not flatter you. All of it is checkable.

### The analogy

Two canteen stalls sell the same vada pav at the same price. One sits in the back corridor near the staff toilets, and its owner cannot understand why business is bad when all four people who tried it came back. The other is three metres from the main door, beside the water cooler everybody visits between classes, and sells 300 a day. Quality decided who came back; position decided who ever tried.

**Where the analogy breaks:** the back-corridor stall cannot move and cannot count who walked past. Your product can sit on a dozen surfaces at once and you can measure impressions, clicks and visits precisely — so you have no excuse for guessing.

### Picture it

```
  SURFACE       IMPRESSION      VISIT        SIGNUP      RETAINED
 ┌─────────┐   ┌──────────┐   ┌────────┐   ┌────────┐   ┌────────┐
 │ search  │   │ a title  │   │ your   │   │ an     │   │ comes  │
 │ result, │──▶│ somebody │──▶│ page   │──▶│ account│──▶│ back   │
 │ reel,   │   │ actually │   │ loads  │   │ exists │   │ next   │
 │ forward │   │ reads    │   │        │   │        │   │ Monday │
 └─────────┘   └──────────┘   └────────┘   └────────┘   └────────┘
 │◀──────── this module (M15) ────────▶│
                            │◀──────── M16 and M17 ──────────▶│
```

Every user arrives through a surface that already has attention, and this module owns the first three boxes while M16 and M17 own the rest.

### Where it lives

M01 gave you a north star metric; distribution is how that number grows from outside your friend group. M11 and M12 produced the pages this module makes findable, and three dependencies carry in: M02's vitals and redirect codes into chapter 3, M07's events and consent into chapter 10, M05's rate limiting into chapter 8.

**M15 ends at the visit.** Did a human see your surface, and did they click through to you? **M16 begins at the visit:** what did they do, did they activate, did they come back, which step leaks. Arguing about onboarding screens here means you have crossed the line.

### Words to own

| Term | Plain meaning | Example |
|---|---|---|
| Distribution | How a stranger ends up in front of your product | A Google result, a forwarded reel |
| Channel | One repeatable route to strangers | Search, Instagram, WhatsApp, YouTube |
| Surface | The specific place your thing appears | A search result page, a Reels feed |
| Impression | One person's screen showed your thing once | 4,000 impressions on a post |
| Click-through rate | Clicks divided by impressions | 80 clicks from 4,000 views is 2% |
| Owned media | A surface you control | Your site, your email list |
| Earned media | Someone else chose to spread it | A student forwards your video |
| Paid media | You bought the impressions | ₹500 on an Instagram ad |
| Compounding channel | Work done once keeps paying | A page that ranks for two years |
| Spiky channel | One burst, then silence | A launch post, a WhatsApp blast |
| Channel-market fit | This channel reaches your actual users | Reels work; LinkedIn does not |
| Dark social | Sharing you cannot see in analytics | WhatsApp and DM forwards |

### Tradeoffs and gotchas

**With ₹0 you cannot buy attention, so you must compound it, and compounding has a warm-up measured in weeks.** Crawlers must find you; feeds must learn who to show you to. Starting that clock in week eleven guarantees it has not finished by demo day.

**Two channels, not nine.** Twelve channels done badly produce nothing, because none reaches the frequency where an algorithm or a crawler notices you. Pick one compounding (search or YouTube) and one with fast feedback (Instagram or WhatsApp), and run both six weeks before judging either.

**Channel-market fit beats channel quality.** For a Nashik tuition-batch finder the parents are on WhatsApp and the students on Instagram, and nobody involved reads blogs or LinkedIn. There is no good channel in the abstract, only a channel where your users already are.

**Hindi, Marathi and Hinglish are not an afterthought.** Your users search `tuition near me nashik`, `नाशिक ट्यूशन फीस`, and `tution fees kitni hai` — three queries, three intents, and most teams write only for the first. That is where the uncontested traffic is. Likewise, if your user would rather watch two minutes than read 800 words, the medium follows the user, not your comfort.

### Verify it

**Red flag: a list of every platform that exists.** "Instagram, Facebook, LinkedIn, X, Threads, YouTube, Reddit, Quora and a newsletter" has no decisions in it. Demand exactly two channels, each with a reason tied to your users.

**Red flag: no reason from your users.** "Instagram has high engagement rates" describes the internet, not your startup. The right answer cites the humans — their language, their phone, their app habits.

**Red flag: a channel you cannot run at the required frequency.** A daily-video plan from three students with classes dies in nine days. Ask each channel's minimum viable frequency and check it against your timetable.

**Red flag: no measurement attached.** "Brand awareness" is not a number; "visits from search per week, read in Search Console" is.

**Red flag: paid ads in the plan.** You said ₹0 — M00's missing-context failure in a new costume.

Then write your two channels, the reason for each, the frequency, and the one number per channel into your M01 decision log.

## Walkthrough
1. Open your product's analytics and write down how many people used it last week who are not in your team, your class or your family. Be strict. That number is your real starting point.
2. Interview five actual or intended users for three minutes each. Ask only: which app did you open most this morning, what did you last search for in any language, and where do you get recommendations from.
3. Write down the languages and scripts that appeared in those answers — English, Hindi, Marathi, Hinglish, transliterated Devanagari. Note which were spoken versus typed.
4. List every surface where your product currently appears. Most teams discover the honest answer is "one home page and one Instagram account with no posts".
5. From the interviews, shortlist three candidate channels. For each, write one sentence of evidence from a real user, not from an article.
6. Classify each candidate as compounding or spiky, and write the warm-up time you expect before it returns anything.
7. Pick exactly two: one compounding, one fast-feedback. Write one sentence on why you rejected the third — that sentence is the valuable part.
8. For each chosen channel, commit to a frequency you can hold for six weeks with your actual timetable, and name the person on the team who owns it.
9. Define one number per channel that you will read every Monday, and say where you will read it from. If you cannot name the screen, the number is not real.
10. Record all of it as one decision-log entry and put a recurring Monday reminder in your phone. Nothing in this module works without the Monday.

## Talk to AI
```prompt
CONTEXT:      I am a student founder in a team of three. We do not write
              code; we direct AI. Our product: [one sentence on what it
              does and for whom]. It is deployed at [URL or "not public
              yet"]. Our north star metric from our PRD is [metric].
              Real non-friend users last week: [number].

              What I learned from interviewing five real users:
              - Apps they open most: [list]
              - What they actually searched for, in their own words
                and script: [Hindi/Marathi/Hinglish queries too]
              - Where they get recommendations: [list]
              - Their phone and network: [e.g. budget Android, slow 4G]

CONSTRAINTS:  Advertising budget is 0 rupees and will stay 0. Three
              people, roughly 6 hours a week total for this, with
              classes. Free tiers only. We are in Nashik, India. We can
              publish in English and [other languages].

GOAL:         Choose exactly TWO distribution channels and a frequency
              we can actually sustain for six weeks.

REQUIREMENTS: 1. Recommend exactly two channels: one compounding, one
                 with fast feedback. No more.
              2. For each, quote the specific line of my user evidence
                 that justifies it
              3. For each, state the minimum frequency below which the
                 channel does not work at all, and the realistic warm-up
                 time before it returns anything
              4. For each, give ONE number to read every Monday and name
                 the exact free tool and screen where I read it
              5. Name the three channels you rejected and why each was
                 wrong for THESE users, not in general

              6. Keep the weekly time cost of the whole plan under 6
                 hours for the team, and show me the arithmetic

NON-GOALS:    Do not propose paid ads, influencer payments, giveaways
              or any channel needing money. Do not write content yet.
              Do not discuss onboarding, retention or pricing - those
              are later modules.

ACCEPTANCE:   - Exactly two channels recommended
              - Every recommendation quotes my user evidence
              - Every number has a named free tool and screen behind it
              - The hours add up to under 6 and you showed the sum
              - A section titled "How this fails" naming the most likely
                reason my team abandons this plan in week three

ASK:          Before recommending anything, tell me what channels you
              would have suggested knowing only that I am an Indian
              student startup - the generic answer. Then tell me which
              piece of my user evidence most changes it, and why. Only
              then give the plan.
```

## Stuck? Ask AI
```prompt
I have built a product that works and almost nobody uses it. Here is what it does: [one sentence]. Here is who it is for: [who]. Do not give me a marketing plan. Instead, ask me eight questions, one at a time, that would let you work out where my intended users already spend attention. Wait for each answer before asking the next.
```

```prompt
Explain the difference between a compounding distribution channel and a spiky one, as if to someone who has never marketed anything. Use two examples from Indian products people actually know. Then give me three channels and ask me to classify each one before you tell me the answer.
```

```prompt
Here is my distribution plan: [paste it]. Do not improve it. Instead, tell me honestly which parts are decisions and which parts are lists. For every channel I named, tell me the minimum weekly effort it needs to work at all, and then tell me which ones my team of three with classes will have quietly dropped by week three.
```

## Quiz

### Q1 · L2
What does "compounding channel" mean?
- (a) A channel where you spend more money each month
- (b) A channel where work done once keeps producing visits later ✅
- (c) A channel that combines several platforms into one post
- (d) A channel whose audience grows by referral only
> A page that ranks, or a video that keeps being discovered, pays out for months after the work. A launch post pays once. With no ad budget, compounding is the only kind of leverage you have.

### Q2 · L3
Two teams ship equally good products. One gets 9 users, the other 340. What does this most likely tell you?
- (a) The second product is better engineered
- (b) The second team started distribution early and ran it as a system ✅
- (c) The first team's product had a worse onboarding flow
- (d) The second team must have spent money on ads
> Quality decides who comes back; position decides who ever arrives. The difference here was eleven weeks of parallel distribution work versus one Sunday night of it.

### Q3 · L4
An AI returns a distribution plan listing ten platforms to post on weekly. What is the strongest objection?
- (a) Ten platforms is too few for real reach
- (b) It contains no decisions, and no channel will reach a sustainable frequency ✅
- (c) Posting weekly is too frequent for a small team
- (d) It should have recommended paid ads first
> A list is not a plan. Spread across ten surfaces, a three-person team cannot hit the frequency any feed or crawler needs to notice them, so all ten return nothing.

### Q4 · L4
Your team argues about whether your signup screen asks for too many fields. Which module does that belong to, and why?
- (a) M15, because signup is part of distribution
- (b) M16, because M15 ends at the visit and M16 begins at it ✅
- (c) M03, because it is a design question
- (d) M11, because the MVP defines the signup flow
> The boundary is the visit. Getting a human to your page is this module; what they do once they are there — activation, drop-off, retention — is behaviour analytics in M16.

### Q5 · L5
Your users are parents in Nashik who mostly use WhatsApp, and students who mostly use Instagram. An AI recommends LinkedIn and a weekly blog because "they build authority". What is actually wrong?
- (a) LinkedIn and blogs are outdated channels
- (b) The recommendation describes the internet in general instead of where these specific users already are ✅
- (c) Authority is not a valid distribution goal
- (d) Blogs cannot be measured properly
> There is no good channel in the abstract, only a channel where your users already spend attention. A recommendation that cites "authority" rather than your user evidence has ignored your context — the classic missing-context failure from M00.

## Go deeper
- [Google Search Central: SEO starter guide](https://developers.google.com/search/docs/fundamentals/seo-starter-guide)
- [Reforge: the four fits framework on channel-market fit](https://www.reforge.com/blog/brian-balfour-four-fits-growth-framework)
