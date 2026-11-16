---
id: m15-10-measuring-content
module: m15-content-social-automation
slug: measuring-content
title: "Measuring Content: Attribution Without Lying to Yourself"
order: 10
duration_min: 35
is_free: false
mission: true
volatility: low
last_reviewed: 2026-11-16
---

## Key learnings
- Tag every link you publish so your own traffic stops arriving as "direct"
- Tell first-touch, last-click and self-reported attribution apart, and name what each one hides
- Account for dark social, because WhatsApp forwards arrive with no referrer at all
- Attach one decision-changing number to each channel instead of reporting impressions
- Spot attribution that credits the last click for work the first touch did

## Lesson

### The pain

Week nine. A team opens their analytics and reads the channel breakdown: **Direct 71%**, organic search 18%, Instagram 6%, other 5%.

They conclude the brand is strong and the Instagram work is not paying, and they stop posting reels. Traffic falls by half over three weeks. The 71% was never people typing a URL. It was WhatsApp forwards, which arrive with no referrer, plus taps on an untagged Instagram bio link. "Direct" was not a channel. It was a bucket of everything they had failed to label — and most of it came from the channel they just killed.

The second mistake is subtler and more common. Their last-click report says search drove eleven signups. True, in a sense: those eleven people typed the brand name into Google and clicked the first result. But they typed the brand name because they had seen three reels over nine days. Last click credits the final step of a journey the reels created, so the team moves effort toward search for a branded term that only exists because of Instagram.

And the third: a 90-day report showing 14,000 impressions. A mentor asks how many became users. Nobody knows.

### The idea

**Attribution is a model, not a measurement. Every model is wrong in a known direction, and your job is to know the direction rather than to chase a perfect number.**

Alongside that, one practical truth that saves students a month: at your scale, **one honest question asked of real users beats four dashboards.** A free-text "how did you hear about us?" at signup will tell you more in two weeks than any attribution report built on fifty visits.

### The analogy

Stand at the misal stall and ask a customer why they came. A senior told them about it in first year. They walked past twice last week. This morning their friend at the counter said "take the misal, not the thali."

Last-click attribution credits the friend at the counter. First-touch credits the senior from first year. Neither is wrong, and neither is complete, and the stall owner who believes only one of them will make a bad decision about where to put a board.

**Where the analogy breaks**, usefully. The stall owner can simply ask, every time, for free — which is exactly why self-reported attribution deserves a place in your toolkit rather than being dismissed as unscientific. But he also cannot see the chain: he has no idea that the senior told four people, who told eleven more in a hostel WhatsApp group he will never be in. That invisible chain is dark social, and it is probably your largest channel.

### Picture it

```
 day 1    day 3      day 9        day 10      day 10
 reel  ─▶ WhatsApp ─▶ types your ─▶ brand  ─▶ signup
 seen     forward     name into     search
                      Google
   │                                  │
   │ FIRST TOUCH credits here         │
   │                                  │ LAST CLICK
   │                                  │ credits here
   ▼                                  ▼
 the reel that started it       the search that
 is invisible in analytics      finished it
```

One real journey crosses four surfaces over ten days, and the two common attribution models credit opposite ends of it — which is why a report built on either one alone will point your effort at the wrong channel.

### Make your own links legible

Most of your "direct" traffic is traffic you failed to label. The fix is a **UTM**: parameters appended to a URL that tell your analytics where the click came from.

| Parameter | What it answers | Example |
|---|---|---|
| `utm_source` | Which platform | `instagram`, `whatsapp` |
| `utm_medium` | What kind of placement | `bio`, `reel`, `qr`, `email` |
| `utm_campaign` | Which push or piece | `evening-slots-nov` |
| `utm_content` | Which variant of the same thing | `hook-a`, `hook-b` |

Rules that prevent the usual mess. Keep everything lowercase with hyphens, because `Instagram` and `instagram` become two rows. Write the scheme down once and make all three teammates use it. Tag every external link you publish — bio links, printed QR codes, WhatsApp messages, email. And never put UTMs on links between your own pages: that restarts the session and makes internal navigation look like outside traffic.

### Three models, and what each one hides

**Last click.** The final touch before the conversion gets everything. It is the default in almost every tool. It systematically over-credits search and brand terms, and under-credits anything that creates demand — video, social, word of mouth.

**First touch.** The earliest known touch gets everything. It over-credits discovery and ignores whatever actually closed the decision. It also only knows about touches you tagged.

**Self-reported.** You ask the user. It is biased toward whatever they remember most recently, and many people answer "Google" for anything involving a browser. It is also the only model that can see a conversation in a WhatsApp group.

Use last click to see what closes, first touch to see what starts, and self-reported to catch the channels the other two cannot observe at all. Disagreement between them is information, not an error to be resolved.

### Dark social, your largest invisible channel

In India this is not a footnote. A link forwarded through WhatsApp arrives at your site with no referrer, so your analytics files it as direct. The same happens inside in-app browsers and DMs.

Three cheap partial fixes. Put a UTM on the link *before* you paste it into WhatsApp, so the forwards you started are labelled. Use a distinct landing path or short link per channel, so the server log says something when the referrer does not. And ask the self-reported question, because it is the only instrument that reaches into a group chat.

Never treat "direct" as a channel deserving credit. Its size measures how much you failed to tag.

### One question, asked honestly

Add a single free-text field at signup: *"How did you hear about us?"* Free text, not a dropdown, because a dropdown only returns options you already thought of. Read every answer yourself for the first hundred users.

Two caveats. Keep it optional, and handle the answers under M07's consent rules — this is personal data in a text box. And do not read percentages off forty answers. Read the *words*: "my hostel group", "a reel", "my brother sent it". Those phrases are the channel names your dashboard lacks.

### From vanity to decision

A number is only worth collecting if it could change what you do next week.

| Stage | Vanity version | Decision version |
|---|---|---|
| Surface | Followers | Non-brand impressions per week |
| Interest | Impressions | Click-through rate per piece |
| Arrival | Pageviews | Visits per channel, tagged |
| Intent | Time on page | Signups per channel |
| Value | Total users | Weekly active from that channel |

Pick one number per channel, write it in your decision log, and read it every Monday. At four weeks, ask one question per channel: keep, change, or kill. A channel with no number attached cannot be killed, which is why teams post into the void for a whole term.

### Incrementality, the cheap version

The honest question is not "how many signups did this channel get credit for" but "how many would we have lost without it". You can approximate it for free: **stop the channel for two weeks and watch.** If nothing changes, it was not doing what the report claimed. Crude, unglamorous, and the only causal evidence available at this scale.

### Where it lives

M07 is the machinery: events, properties, consent, and defining a metric before collecting it. M01 holds your north star, and every number here should ladder up to it. Chapters 2 to 4 produce the search numbers, chapter 8 the posting records, chapter 9 the per-format comparison.

And the module boundary, stated precisely one last time. **M15 measures whether people arrived. M16 measures what they did after arriving** — activation, funnels, retention, cohorts, growth loops. M17 turns arrivals into CAC, payback and revenue. If your question is "did this content bring anyone", it is answered here; if it is "why do they leave on screen two", it is not.

### Words to own

| Term | Plain meaning | Example |
|---|---|---|
| UTM | Tags on a URL naming its source | `?utm_source=instagram` |
| Referrer | The site the browser says sent you | Empty for WhatsApp |
| Direct traffic | Visits with no known source | A bucket, not a channel |
| Dark social | Sharing you cannot observe | WhatsApp forwards, DMs |
| Last click | Credit to the final touch | Brand search gets it all |
| First touch | Credit to the earliest known touch | The reel gets it all |
| Self-reported | Asking the user directly | "My hostel group" |
| Attribution window | How far back credit may reach | 30 days |
| Holdout | Pausing a channel to test it | Two weeks off |
| Incrementality | What you would lose without it | The only causal question |
| Vanity metric | A number no decision depends on | Follower count |
| Conversion rate | Share who took the next step | 80 visits, 6 signups |

### Tradeoffs and gotchas

**Small numbers lie loudly.** With 40 visits and 3 signups, a "7.5% conversion rate" is noise with a decimal point. Report counts until the counts are large, and never compare two percentages built on fewer than a hundred events each.

**Do not change four things at once.** If you fix your sitemap, change your hook, add a channel and redesign the landing page in the same week, you have learned nothing about any of them.

**Give a channel eight weeks, not two.** Search has a warm-up measured in months. Killing a compounding channel at two weeks is how teams conclude that SEO does not work.

**Consent is not optional.** Tie every event to M07's consent state, treat self-reported answers as personal data, and avoid anything resembling fingerprinting.

**Beware benchmark numbers from AI.** "Average CTR is 2.4%" is usually fabricated. Your baseline is your own previous four weeks.

**Your own visits pollute everything.** Three teammates reloading fifty times is a measurable share of your traffic. Exclude yourselves.

**Write the decision, not the number.** "Instagram: 212 tagged visits, 9 signups — keep, and move from daily to three a week" is a decision-log entry. "Instagram is doing well" is not.

### Verify it

**Red flag: attribution that credits the last click for work the first touch did.** Ask for both models side by side. If search gets all eleven signups under last click and almost none under first touch, the demand was created somewhere else and your report is pointing you the wrong way.

**Red flag: "direct" treated as a channel.** If direct is your largest row, the finding is "we have not tagged our links", not "our brand is strong".

**Red flag: impressions reported as results.** Ask what happened to them at every step down to a signup. A report that stops at the top of the funnel was designed not to be checked.

**Red flag: percentages from tiny numbers.** Ask for the raw counts behind every rate. Below about a hundred events, insist on counts only.

**Red flag: no self-reported question.** If nobody asks users, your view excludes dark social — probably your largest channel in India.

**Red flag: UTMs on internal links.** Check your navigation. Internal UTMs inflate traffic and corrupt every channel number you have.

**Red flag: invented benchmarks.** Any industry average with no named source is fabrication. Demand that each number be labelled measured or estimated, exactly as in chapter 2.

**Red flag: a metric with no decision attached.** For each number on your scoreboard, ask what you would do differently if it halved. No answer means delete the row.

## Walkthrough
1. Open your analytics and write down the channel breakdown for the last 30 days, including the exact size of "direct". That number is your ignorance, measured.
2. List every external link you have published — bio links, QR codes, WhatsApp messages, emails, comments. Note which have UTMs. Most teams find the answer is none.
3. Write your UTM scheme on one line: allowed sources, allowed mediums, campaign naming, all lowercase with hyphens. Share it with both teammates.
4. Re-tag every live external link using the scheme. Replace your bio link and regenerate any printed QR code.
5. Check your internal navigation for UTMs and remove any you find, then confirm your own team's visits are excluded from the reports.
6. Add one optional free-text field at signup: "How did you hear about us?" Keep it optional and handle the answers under your consent rules.
7. Build a one-page scoreboard with one row per channel and exactly four columns: tagged visits, signups, the self-reported count, and your decision.
8. Pull the same period under last click and under first touch, where your tool allows it, and write one sentence on where the two disagree.
9. After four weeks, read the self-reported answers as words, not percentages. Write down every channel name that appears there and is missing from your dashboard.
10. Make one decision per channel — keep, change or kill — with the number that justifies it, and record it in your decision log. Then pick one channel to pause for two weeks as a holdout.

## Talk to AI
```prompt
CONTEXT:      I am a student founder. I do not write code; I direct AI.
              My product: [one sentence]. My north star metric from our
              PRD: [metric]. My channels: [list them].

              What I measured myself over the last 30 days:
              - channel breakdown including direct: [paste the rows]
              - total visits: [X], total signups: [X]
              - external links I publish, and which carry UTMs:
                [list them honestly]
              - self-reported answers so far, verbatim: [paste them, or
                "we do not ask yet"]
              - my own team's visits excluded? [yes / no]

GOAL:         A one-page scoreboard and a UTM scheme that make my real
              channels distinguishable, and one honest decision per
              channel.

REQUIREMENTS: 1. A UTM scheme: allowed sources, allowed mediums, a
                 campaign naming rule, all lowercase with hyphens
              2. Tell me which of my published links must be re-tagged,
                 and which must NOT carry UTMs at all, and why
              3. Explain what share of my "direct" row is probably dark
                 social, how you reached that guess, and what would
                 narrow it
              4. A scoreboard of one row per channel and exactly four
                 columns, with every number sourced from a named free
                 tool and screen
              5. For each channel, the one number that would change my
                 decision, and what I would do if it halved
              6. Last click versus first touch for my data: say where
                 they would disagree and which channel each one unfairly
                 credits

CONSTRAINTS:  Free tools only. Fewer than 100 signups, so no percentages
              built on small counts. Consent rules apply to anything a
              user types. I cannot run paid experiments.

NON-GOALS:    Do not give me industry benchmark numbers. Do not report
              impressions as a result. Do not propose a dashboard with
              more than six numbers on it. Do not discuss onboarding,
              retention or funnels inside the product - that is a later
              module.

ACCEPTANCE:   - Every number is labelled MEASURED, with the tool and
                screen named, or GUESSED
              - No percentage is computed on fewer than 100 events
              - "Direct" is never treated as a channel deserving credit
              - Every scoreboard row has a decision attached, not just a
                number
              - A section titled "What your data cannot tell you",
                naming the cheapest next measurement

ASK:          First, tell me the single most likely wrong conclusion
              somebody would draw from the numbers I pasted, and why it
              is wrong. Then build the scheme and the scoreboard.
```

## Mission

**Objective.** A live, tagged measurement setup for your own startup and a one-page scoreboard that ends in one written decision per channel — keep, change or kill — each justified by a number you measured yourself.

**Steps.**
1. **Frame** — write down the size of your "direct" row today. That is the measurement you are trying to shrink.
2. **Scheme** — write your UTM scheme on one line and agree it with both teammates. Lowercase, hyphens, a fixed list of sources and mediums.
3. **Tag** — re-tag every external link you publish, including the bio link and any QR code. Remove every internal UTM. Exclude your own team's traffic.
4. **Ask** — add the optional free-text "how did you hear about us?" field at signup, under your consent rules.
5. **Direct** — brief AI to build the four-column scoreboard, with every number tied to a named free tool and screen, and no percentages on small counts.
6. **Compare** — pull the same period under last click and under first touch and write one sentence on where they disagree and which channel each over-credits.
7. **Read** — after at least two weeks, read the self-reported answers as words. Write down every channel name that appears there but not in your dashboard.
8. **Decide** — one decision per channel, with the number beside it. Pause one channel for two weeks as a holdout and predict, in writing, what will happen.
9. **Verify** — click one of your own tagged links from a phone and confirm it appears in the right row. Confirm your own visits are excluded.
10. **Document** — one decision-log entry naming the channel you were most wrong about, and the number that corrected you.

**Upload.**
- Your Director's Brief → attached file or post caption
- Your channel breakdown before and after tagging, with the direct row visible in both → screenshots
- Your written UTM scheme → text
- A list of every external link with its tagged URL → table or screenshot
- Your signup form showing the self-reported question live → screenshot
- Your one-page scoreboard, four columns, one row per channel → screenshot
- Verbatim self-reported answers, with anything identifying removed → text
- Your holdout prediction, written before the two weeks start → text
- Your AI conversation → share link in `ai_chat_url`
- Your live product → `demo_url`
- Reflection, 150 words max → one tradeoff you chose, one thing the AI got wrong

**Self-check.** Tick every box before you submit.
- [ ] Every external link I publish carries a UTM following my written scheme
- [ ] No internal link on my site carries a UTM
- [ ] My own team's visits are excluded from the reports
- [ ] I clicked one tagged link myself and confirmed which row it landed in
- [ ] My "direct" row is smaller than it was, and I can say why
- [ ] The self-reported question is live, optional, and handled under consent rules
- [ ] No percentage on my scoreboard is computed from fewer than 100 events
- [ ] Every scoreboard row ends in keep, change or kill with a number beside it
- [ ] I wrote my holdout prediction down before the holdout started
- [ ] My reflection names a tradeoff in my own words

**Stretch.** Run the holdout to completion and write 100 words comparing what happened with your written prediction. That gap is the most honest measurement in this module.

## Stuck? Ask AI
```prompt
My analytics says 71% of my traffic is "direct". Do not interpret that as brand strength. Instead, list every real-world way a visit ends up labelled direct, ask me which of my own published links carry campaign tags, and then tell me what I can and cannot conclude from my data as it stands.
```

```prompt
Explain first-touch, last-click and self-reported attribution as if I have never used an analytics tool, using one person who saw a reel, got a WhatsApp forward, then searched my brand name and signed up. For each model, say which touch gets the credit and what decision it would push me toward wrongly.
```

```prompt
Here are my numbers for the last 30 days: [paste visits, signups and channel rows]. Do not calculate conversion rates. Tell me which comparisons my sample size cannot support, which single number I am missing to make a real decision, and the cheapest free way to get it.
```

## Quiz

### Q1 · L3
Your analytics shows "Direct 71%". What is the most likely explanation?
- (a) Your brand is strong and people type your URL from memory
- (b) Most of your published links are untagged, and WhatsApp forwards arrive with no referrer ✅
- (c) Search engines are stripping referrer data
- (d) Your analytics tool is misconfigured and must be replaced
> Direct is a bucket of unlabelled traffic, not a channel. A large direct row measures how much you failed to tag, and in India most of it is dark social.

### Q2 · L4
Eleven people signed up after searching your brand name, having each seen three reels first. Last-click attribution credits search. What is the risk?
- (a) None; the search was the final step so it earned the credit
- (b) You move effort to search for a branded term that only exists because of the reels ✅
- (c) The signups will be counted twice
- (d) Search traffic is always lower quality than social
> Last click over-credits whatever closes and under-credits whatever creates demand. Reading first touch beside it is what exposes the reel's real contribution.

### Q3 · L4
You have 40 visits and 3 signups from a channel. Your teammate reports a 7.5% conversion rate and wants to double down. What should you say?
- (a) Agree; 7.5% is above average for most channels
- (b) Three events cannot support a rate — report counts and wait for larger numbers ✅
- (c) Recalculate using first-touch attribution instead
- (d) Compare it to the industry benchmark before deciding
> Percentages built on single-digit counts are noise with a decimal point. Report raw counts until the counts are large, and never compare two rates built on fewer than a hundred events.

### Q4 · L2
Why should you never put UTMs on links between your own pages?
- (a) They slow down page loads
- (b) They restart the session, so internal navigation looks like new external traffic ✅
- (c) Search engines treat them as duplicate content
- (d) They are only valid on external domains
> Internal UTMs inflate your traffic and corrupt every channel number you have. Tag external links only — bio links, QR codes, messages and emails.

### Q5 · L5
After four weeks you cannot tell whether your Instagram work is producing users. You have no budget. What is the most honest test available?
- (a) Compare your click-through rate to an industry benchmark
- (b) Pause Instagram for two weeks and see what actually changes ✅
- (c) Switch your reports from last click to first touch
- (d) Add more tracking parameters and wait another month
> A holdout is crude and it is the only causal evidence you can afford. The real question is not how much credit a channel was assigned, but what you would lose without it.

## Go deeper
- [Google Analytics Help: collect campaign data with custom URLs](https://support.google.com/analytics/answer/10917952)
- [MeitY: Digital Personal Data Protection Act, 2023](https://www.meity.gov.in/data-protection-framework)
