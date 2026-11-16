---
id: m15-01-distribution
module: m15-content-social-automation
slug: distribution-is-the-other-half
title: "Distribution Is the Product's Other Half"
order: 1
duration_min: 25
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

Two teams from the same batch demo in week eleven. Both products work. Both are deployed, fast on a cheap Android phone, and genuinely useful.

The first team has 9 users. Seven are their own classmates, one is a cousin, one is a mentor who signed up to be kind. They spent eleven weeks on the product and the last Sunday night on "marketing", which meant one Instagram post from a brand-new account with 14 followers and a WhatsApp message to a family group.

The second team has 340 users and 60 who came back last week. They did not build a better product. In week four they wrote eleven answers to the exact questions their users were already typing into Google, put each answer on its own page, and from week five posted one 40-second vertical video every weekday showing the product solving one small problem. By week eleven, four of those pages were bringing 20 to 30 visits a day without anybody touching them, and one video had been forwarded into roughly 200 WhatsApp groups.

The first team believed the product was the work and distribution was a thing you do at the end. That belief is the single most expensive one in this course, and it is almost universal among engineers.

### The idea

**A product nobody can find does not exist. Distribution is a system with inputs, queues, failure modes and measurements — exactly like the systems you have been building for fourteen modules — and it has to be built in parallel with the product, not after it.**

This module treats growth as engineering. You will build a sitemap that matches reality, a page template that turns a dataset into hundreds of honest pages, a posting queue that respects rate limits, and a measurement scheme that does not flatter you. None of it is vibes. All of it is checkable.

### The analogy

Two stalls in a college canteen sell the same vada pav at the same price, made the same way.

The first is in the back corridor near the staff toilets. Nobody walks past it unless they already know it is there. The owner cannot understand why business is bad — the vada pav is excellent, and he can prove it, because the four people who have tried it all came back.

The second is three metres from the main door, beside the water cooler everybody visits between classes, with the price painted in letters readable from the queue. It sells 300 a day.

Quality decided who came back. Position decided who ever tried.

**Where the analogy breaks**, in two ways that matter. First, the back-corridor stall cannot move; your product can appear on a dozen surfaces at once and you can change them any afternoon. Second, nobody can tell the canteen owner exactly how many people walked past and did not stop — but on the web you can measure impressions, clicks and visits precisely, which means you have no excuse for guessing. The canteen explains why position matters; it badly understates how much control and how much data you actually have.

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

Every user arrives through a surface that already has attention, and this module is about the first three boxes: being on a surface, earning the impression, and getting the visit. What happens after the visit — activation, retention, revenue — is M16 and M17.

### Where it lives

M01 gave you a north star metric. Distribution is how the number in that metric gets bigger from outside your friend group. M11 got your MVP deployed and M12 got your app listed; both of those produced the pages and listings this module now tries to make findable.

The boundary with M16 is worth stating precisely, because the two modules are easy to blur. **M15 ends at the visit.** Did a human see your surface, and did they click through to you? **M16 begins at the visit:** what did they do, did they activate, did they come back, which step leaks. If you find yourself arguing about onboarding screens here, you have crossed the line. If you find yourself arguing about whether anybody is arriving at all, you are in the right module.

Three other dependencies carry straight in. M02's Core Web Vitals, rendering and redirect status codes are the raw material of chapter 3. M07's event and consent work is what makes chapter 10's measurement legal and real. M05's rate limiting is literally the constraint chapter 8 is built around, because social APIs reject you hard and fast.

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

**With ₹0 you cannot buy attention, so you must compound it.** Paid ads are a machine for turning money into visits, and that machine is closed to you this term. What is open: search pages that keep working, a video archive that keeps getting discovered, and things worth forwarding. All three are slow to start and hard to stop, which is the correct trade when you have time but no money.

**Two channels, not nine.** Every AI answer to "how do I get users" lists twelve platforms. Twelve channels done badly produce nothing, because none of them reaches the frequency where an algorithm or a crawler notices you. Pick two: one compounding (usually search or YouTube) and one with fast feedback (usually Instagram or WhatsApp). Run both for six weeks before you judge either.

**Channel-market fit beats channel quality.** For a Nashik tuition-batch finder, the parents are on WhatsApp and the students are on Instagram, and nobody involved reads blogs or LinkedIn. For a tool sold to developers, the reverse. There is no good channel in the abstract — only a channel where your users already are. Go and ask five real users which app they opened most this morning. That answer outranks any article.

**Hindi, Marathi and Hinglish are not an afterthought.** Your users search `tuition near me nashik`, `नाशिक ट्यूशन फीस`, and `tution fees kitni hai` — three different queries with three different intents, and most teams write for only the first. Chapter 2 takes this seriously because it is where the uncontested traffic is.

**YouTube beats blogging for a lot of Indian audiences.** If your user would rather watch two minutes than read 800 words, a blog post is a page nobody will finish. Search still matters, because YouTube is a search engine, and your page still matters, because it is where the signup lives — but the medium should follow your user, not your comfort.

**The best distribution is a product feature.** A shareable receipt, a public profile, a result card worth screenshotting: these put a surface inside the product. That is M16's growth-loop material, and the reason this module is placed next to it.

**Do not wait for "ready".** Distribution has a warm-up period measured in weeks — crawlers have to find you, feeds have to learn who to show you to. Starting that clock in week eleven guarantees it has not finished by demo day.

### Verify it

Ask an AI for a distribution plan and you will get something confident and useless within three seconds. Here is how to tell whether it understood your situation.

**Red flag: a list of every platform that exists.** "Post on Instagram, Facebook, LinkedIn, X, Threads, YouTube, Pinterest, Reddit, Quora and start a newsletter" is a plan with no decisions in it. Demand exactly two channels and a stated reason tied to your users.

**Red flag: no reason from your users.** The right answer cites something about the humans — their language, their phone, their app habits, the time of day they are free. If the justification is "Instagram has high engagement rates", it is describing the internet, not your startup.

**Red flag: a channel you cannot run at the required frequency.** A daily-video plan from a team of three with classes is a plan that dies in nine days. Ask what the minimum viable frequency of each channel is and whether you can actually sustain it for six weeks.

**Red flag: no measurement attached.** Every proposed channel needs one number that would tell you in three weeks whether to keep going. "Brand awareness" is not that number. "Visits from search per week, measured in Search Console" is.

**Red flag: paid ads in the plan.** If you said ₹0 and the plan still assumes a budget, your constraint was ignored — exactly the failure M00 warned about when a brief's CONSTRAINTS section is missing or unread.

Write your two chosen channels, the reason for each, the frequency you commit to, and the one number per channel into your decision log from M01. In week eleven, that entry is the difference between "we tried marketing" and "we ran two channels for six weeks and here is what each returned".

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
