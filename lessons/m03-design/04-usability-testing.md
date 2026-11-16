---
id: m03-04-usability-testing
module: m03-design
slug: usability-testing
title: "Usability Testing With Three Friends"
order: 4
duration_min: 30
is_free: false
mission: true
volatility: low
last_reviewed: 2026-11-16
---

## Key learnings
- Run a usability test on three people in one lunch break, with no budget
- Write tasks instead of asking for opinions, and stay silent while they struggle
- Separate what you observed from what you concluded
- Turn an observation into a precise correction brief and confirm the fix
- Avoid the four ways student teams accidentally coach their own testers

## Lesson

### The pain

A team shows their canteen app to eight friends and asks "what do you think?" Everyone says it looks nice. Two people suggest adding dark mode. The team spends the next week building dark mode.

At demo day, a stranger is handed the phone and asked to order a vada pav. They tap the dish photo. Nothing happens — the photo is not the tap target, the small "Add" text below it is. They tap the photo again, harder. Then they say "is it loading?" Forty seconds pass. They never order anything.

Eight friends looked at that app and none of them hit this, for one reason: nobody was ever given a task. They were given a tour. A tour tells you whether people are polite. A task tells you whether your product works.

### The idea

**Give three people a real task, say nothing, and write down exactly what they do — not what they say about it.**

Three is enough. Nielsen's research shows five users find roughly 85% of usability problems, and the first three find most of those. You are not measuring anything; you are discovering problems you are blind to. Your fourth tester will mostly confirm the first three, and your time is better spent fixing what you already found and testing again.

### The analogy

Imagine standing behind the canteen counter and watching sixty students order, without saying anything. You would learn things no survey could tell you. That three people in a row point at the second dish on the board and it is the one that keeps running out. That everybody holds the plate in their left hand and then cannot reach the water jug on the left. That the new students hesitate at the token window because the sign faces the wrong way.

Nobody *told* you any of that. You watched, and the room told you.

**Where the analogy breaks:** behind the counter you can fix confusion on the spot — "water's on the other side, bhai". In a usability test that same helpful instinct destroys the data. The moment you say "just tap the Add button", you have answered the only question you were trying to ask. The hardest skill in this chapter is keeping your mouth shut while someone you know struggles in front of you.

### Picture it

```
            ONE 10-MINUTE SESSION
┌────────────────────────────────────────────────────────┐
│ 1 min   Set up    "There are no wrong answers.         │
│                    I am testing the app, not you."     │
│ 1 min   Warm up    one easy question about their day   │
│ 6 min   TASKS      hand over the phone. SAY NOTHING.   │
│                    note: taps, pauses, words, faces    │
│ 2 min   Debrief    "What was the most annoying part?"  │
└────────────────────────────────────────────────────────┘
        ▼                              ▼
   OBSERVATIONS                   CONCLUSIONS
   "tapped photo 3x"     ───▶     "photo is not a
   (what happened)                 tap target"
                                  (your inference)
```

A ten-minute session spends six of those minutes with you silent, and the notes you take are split into what happened and what you think it means — kept separate, because the first is evidence and the second is a guess.

### Writing tasks, not questions

A task has an outcome the person can reach or fail to reach. A question has an opinion as its answer. Compare:

| Bad — a question | Good — a task |
|---|---|
| Do you like the menu screen? | You have ₹40 and 15 minutes. Get lunch ordered. |
| Is the cart easy to find? | You changed your mind about the samosa. Remove it. |
| Would you use this daily? | Order exactly what you ordered yesterday. |
| Is the sold-out marking clear? | Tell me what I cannot buy right now. |

Three rules for writing them. Give a situation, not an instruction — "you have ₹40 and 15 minutes" rather than "use the reorder feature", because the second one names the feature and so the test is over. Never use your own UI words; if your button says "Reorder", the task must say "order the same thing again". And make the task have a definite end, so you can record reached or not reached rather than a vibe.

Four tasks in six minutes is about right. Put the most important flow first, while they are still fresh.

### Running it without coaching

Four ways student teams accidentally coach their own testers, all of them well-meant:

**Explaining first.** A thirty-second "so this is the menu, and down here is the cart" tour means you have just taught them the thing you needed to see them discover. Hand over the phone with no explanation at all.

**Rescuing.** They are stuck, it is painful, they are your friend. Count to ten in your head. If you must speak, say only "what are you trying to do?" or "what do you expect would happen if you tapped that?" Never "try the button at the bottom".

**Asking leading questions.** "Was that confusing?" plants the idea. Ask "what just happened there?" instead and let them characterise it.

**Testing friends who know your product.** Your teammates cannot be testers. They know where everything is. Find people who have never seen it — a hostel neighbour, someone from another branch, the canteen staff themselves for the staff flow.

One more thing that costs nothing and changes everything: ask them to think aloud. "Say what you are thinking as you go, even if it sounds silly." You will hear "I'll just... hmm, where's the price?" which is worth more than any answer they could give you afterwards.

### Observation versus conclusion

The single most common error in a student test report is writing conclusions and calling them observations. Keep two columns.

An observation is something a camera would have captured. "Tapped the dish photo three times." "Paused 8 seconds on the payment screen." "Said 'is this the total?'" "Scrolled past the reorder block twice without stopping."

A conclusion is your interpretation. "The photo looks tappable but is not." "The total is ambiguous." "The reorder block does not read as interactive."

Why it matters: conclusions are where your bias enters, and you need to be able to re-examine them later when a different conclusion fits the same observation better. Also, when two testers produce the same observation and you had two different conclusions, that is the signal to look harder rather than to pick one.

A finding earns a fix when two of three testers hit it. One tester hitting something strange is interesting; two is a pattern. The exception: if one tester hits something that loses money or data — a double order, a payment that charges twice — fix it regardless of count.

### Where it lives

Usability testing is the cheapest quality tool you will ever have and the one most teams skip because it feels awkward rather than because it is hard. It sits directly after the heuristic review in chapter 3 — you run the checklist yourself first so your testers are not wasted on problems you could have found alone. It runs on wireframes, which is why chapter 2 told you to keep them lo-fi. In M07 you will see the same drop-offs in analytics as numbers; testing is how you learn *why* the number is what it is, which no dashboard can tell you. M11's checkpoints will ask what you changed because of a user, and this is where that evidence comes from. M12 adds a version of this with assistive technology.

### Words to own

| Term | Plain meaning | Example |
|---|---|---|
| Usability test | Watching someone attempt a real task | Three students, ten minutes each |
| Task | A situation with a reachable outcome | "You have ₹40. Get lunch." |
| Think aloud | Asking them to narrate their thinking | "Where's the price?" |
| Observation | What a camera would have recorded | "Tapped the photo three times" |
| Conclusion | Your interpretation of observations | "The photo looks tappable" |
| Facilitator | The person running the session | You, saying almost nothing |
| Leading question | A question that plants its own answer | "Was that confusing?" |
| Task success | Did they reach the outcome, yes or no | 1 of 3 ordered successfully |
| Time on task | How long it took them | 48 seconds to order |
| Severity | Users affected times how much it hurts | Double order = highest |

### Tradeoffs and gotchas

**Three testers tell you what is broken, not how many people it affects.** You cannot say "25% of users have this problem" from three people. That is what M07's analytics are for. Qualitative testing finds problems; quantitative measurement sizes them. Confusing the two produces confident nonsense.

**Testing too late makes the findings unaffordable.** If you test the week before demo day, every finding becomes a reason not to fix it. Test the lo-fi wireframes, then test again after the build.

**Your friends will be kind, which is the problem.** Expect the debrief to be gentler than the observations. Trust the observations. If someone struggled for forty seconds and then said "yeah it was fine", the forty seconds is the data.

**Do not test everything.** Pick the one flow that matters most. A session that tries to cover the whole app covers nothing properly and exhausts the tester.

**Record whatever you can, with permission.** Screen recording on the phone plus audio is plenty. You will miss things in the moment; three people and no notes is an afternoon wasted. Ask first, and in India be explicit that you are recording the screen and voice, not their face.

### Verify it

Two verification jobs here: checking your own test, and checking what AI does with your findings.

Checking your test: **Did you talk?** Count your own interventions. More than two or three in six minutes and you coached. **Were your tasks free of your own UI words?** If a task named a button, that task is void. **Can you separate observations from conclusions?** If your notes have no "tapped", "paused", "said" or "scrolled" in them, you recorded opinions. **Did at least one tester fail at something?** Three clean successes on a first test almost always means the tasks were too easy or you helped.

Checking AI's fix: give it one observation, not a redesign request. "Two of three testers tapped the dish photo expecting it to add the item; it is not a tap target" is a brief. "Make the menu better" is not. Then confirm three things: the specific behaviour changed, the tap target is actually big enough on a real phone rather than in theory, and nothing else on the screen moved. Ask for the diff in plain words — "list exactly what you changed and why" — and compare against what you asked for.

Red flags: AI responding to a usability finding with a visual redesign; a fix that adds a tooltip or help text instead of fixing the thing (help text is heuristic 10, and it is the last resort, not the first); a claim that the tap target is "now 44px" without you checking it on a device; and any fix that quietly changes a second screen you did not mention.

## Walkthrough
1. Pick exactly one flow to test — the single most important path in your product. Write its outcome as one sentence you can mark reached or not reached.
2. Write four tasks as situations, with no UI words in them. Read each one aloud and strike any word that appears on a button or label in your app.
3. Decide in advance what you will record: screen recording with audio, plus a two-column notes page headed "observed" and "concluded".
4. Recruit three people who have never seen your product and are not in your team. A hostel neighbour, someone from another branch, a canteen staff member if you are testing the staff flow.
5. Run session one. Read the setup line, hand over the phone without explaining anything, read task one, then stay silent for six minutes. Write only what happens.
6. Debrief with two open questions: "what was the most annoying part?" and "what did you expect to happen when you tapped that?" Do not defend anything.
7. Run sessions two and three without changing anything between them. Resist the urge to fix after the first — you need to know whether it repeats.
8. Build a findings table: observation, how many testers hit it, your conclusion, severity. Mark anything two or more testers hit, plus anything that costs money or data even at one.
9. Write one correction brief for your top finding, stating the observation, the expected versus actual behaviour, and a NON-GOAL limiting what may change.
10. Apply the fix, then re-test that one task with a fourth person. Confirm the behaviour changed, on a real phone, and that nothing else moved.

## Talk to AI
```prompt
CONTEXT:      I am a student founder in a team of three. My product is
              [one sentence]. The single most important flow is [name it,
              with its outcome]. My users are students and staff on
              budget Android phones in India, often on slow 4G, and many
              read English as a second language.

              My interface, screen by screen, top to bottom, including
              the exact text on every button and label: [paste]

GOAL:         A complete, runnable usability test plan for three testers
              and ten minutes each, that I can execute today with no
              budget and no tools beyond a phone.

REQUIREMENTS: 1. A setup script I read aloud, max 40 words, that makes
                 clear I am testing the app and not the person
              2. Exactly four tasks, written as situations with a
                 reachable outcome
              3. For EACH task, list the words I must not say because
                 they appear in my own interface
              4. A note-taking sheet with two separate columns,
                 "observed" and "concluded", and three examples of each
                 so I can tell them apart
              5. The two debrief questions to ask at the end, and three
                 leading questions I must avoid
              6. A prediction: the three places in MY described flow
                 where you expect testers to get stuck, and why

CONSTRAINTS:  Ten minutes per session, six of them silent. No paid tools,
              no lab, no incentives. Testers are strangers to the
              product. I must be able to run this in a canteen or
              corridor. Tasks must not name any button or label from my
              interface.

NON-GOALS:    Do not redesign my interface. Do not suggest new features.
              Do not give me a survey, a questionnaire, or a rating
              scale. Do not write any code.

ACCEPTANCE:   - Every task describes a situation, never an instruction to
                use a named feature
              - No task contains a word that appears on one of my buttons
              - Each task has an outcome I can mark reached or not
                reached, with no judgement
              - The observed/concluded examples are clearly different in
                kind, not just in wording
              - Your three stuck-point predictions each name a specific
                element from what I pasted

ASK:          Before writing the plan, tell me which ONE of my four tasks
              you think is least worth running and why, and suggest a
              better one. Then write the full plan.
```

## Mission

**Objective.** Run a real usability test of your own product's most important flow on three people who have never seen it, and ship one fix driven by what you observed rather than what anyone said.

**Steps.**
1. **Frame** — write the one flow you are testing and its outcome in a single sentence that can be marked reached or not reached.
2. **Brief** — use the Director's Brief above to produce your test plan: setup script, four situation-based tasks, forbidden words, notes sheet, debrief questions.
3. **Prepare** — check every task for UI words and strike them. Set up screen recording with audio and get each tester's permission out loud.
4. **Direct** — run three sessions of ten minutes with three strangers to your product, changing nothing between sessions. Six minutes of each session must be you silent.
5. **Verify** — build the findings table: observation, tester count, conclusion, severity. Keep observations and conclusions in separate columns.
6. **Iterate** — write one precise correction brief for the highest-severity finding, including expected versus actual and a NON-GOAL. Apply it.
7. **Re-test** — run the same task with a fourth person. Confirm the behaviour changed on a real phone and that nothing else on the screen moved.
8. **Document** — one decision-log entry: what you observed, what you changed, what you deliberately chose not to fix, and why.

**Upload.**
- Your test plan, including the four tasks and the forbidden-word list → post caption or attached file
- The AI conversation that produced the plan → share link in `ai_chat_url`
- Your findings table with observations and conclusions in separate columns → screenshot or text
- Evidence from the sessions → screen recordings, or your handwritten notes photographed
- Before and after of the one thing you fixed → screenshots, and `demo_url` if it is deployed
- Your re-test result with the fourth person → one line per task, reached or not reached
- Reflection, 150 words max → the gap between what testers said and what they did, and one thing you were wrong about

**Self-check.** Tick every box before you submit.
- [ ] All three testers were strangers to my product and not on my team
- [ ] No task contains a word that appears on a button or label in my app
- [ ] Every task has an outcome I marked reached or not reached
- [ ] I counted my own interventions and there were three or fewer per session
- [ ] I changed nothing between the three sessions
- [ ] My notes contain observations a camera could have recorded, in their own column
- [ ] At least one tester failed at something, and I wrote down exactly where
- [ ] My fix brief names an observation, not an opinion, and includes a NON-GOAL
- [ ] I re-tested the fix on a real phone with a fourth person
- [ ] My decision log names something I chose NOT to fix, and why
- [ ] My reflection names a case where what a tester said contradicted what they did

**Stretch.** Run one session on the staff or admin flow with an actual canteen worker or shopkeeper rather than a student. The gap between how a customer and an operator read the same product is usually the most useful thing you will learn all month.

## Stuck? Ask AI
```prompt
Here are four usability tasks I wrote: [paste them]. For each one, tell me whether it is a situation or a disguised instruction, list every word in it that would give away part of my interface, and rewrite the weak ones. Do not add tasks of your own.
```

```prompt
Here are my raw notes from one usability session: [paste them]. Sort every line into either "observation — a camera could have recorded this" or "conclusion — this is my interpretation". Where a line is a conclusion, tell me what the underlying observation probably was and what I should have written down instead.
```

```prompt
Two of three testers did this: [describe exactly what you observed, with no interpretation]. Give me three different explanations for that behaviour, ranked by how likely they are, and for each one a single specific change that would test the explanation. Do not redesign my screen and do not pick for me.
```

## Quiz

### Q1 · L3
Why do three testers find most of your usability problems?
- (a) Three is the number at which results become statistically significant
- (b) The same blocking problems recur across testers, so the first few surface most of them ✅
- (c) Most apps only have three serious problems
- (d) More than three testers start copying each other's behaviour
> Testing is discovery, not measurement. Blocking problems are hit by almost everyone, so the first few testers reveal them — and a fourth mostly confirms what you already know.

### Q2 · L4
A tester struggles for forty seconds, fails the task, and then tells you "yeah, it was fine". What do you record?
- (a) That the flow is acceptable, since the tester said so
- (b) The forty seconds and the failure — behaviour over opinion ✅
- (c) Both, weighted equally, and average them
- (d) Nothing, because the session was inconclusive
> Friends are kind, especially in a debrief. What they did is evidence; what they say about it afterwards is filtered through politeness.

### Q3 · L4
Which of these is a valid usability task?
- (a) Do you find the reorder feature easy to use?
- (b) Tap the Order Again button at the top of the screen
- (c) You ate here yesterday and want exactly the same thing. Get it ordered. ✅
- (d) Rate the menu screen out of ten
> A task gives a situation with a reachable outcome and never names your own UI words. Naming the button answers the question you were trying to ask.

### Q4 · L3
One of three testers accidentally placed two identical orders by double-tapping. Nobody else hit it. What do you do?
- (a) Ignore it — the rule is two of three testers
- (b) Fix it, because anything that costs the user money overrides the count rule ✅
- (c) Add help text explaining not to double-tap
- (d) Run three more sessions before deciding
> The two-of-three rule prioritises ordinary friction. A finding that loses money or data is severe at any frequency, and help text never fixes a missing pending state.

### Q5 · L5
Your finding is "two of three testers tapped the dish photo expecting it to add the item". AI responds with a full visual redesign of the menu screen. What has gone wrong, and what do you do?
- (a) Nothing — a redesign will solve it along with other issues
- (b) The brief lacked a NON-GOAL, so restate the observation with a scope limit and reject the redesign ✅
- (c) Accept it but ask for the old colours back
- (d) Add a tooltip to the photo instead
> One observation, one bounded change. An unbounded brief invites a rewrite you cannot review, and you then cannot tell which change fixed the finding and which broke something else.

## Go deeper
- [Nielsen Norman Group: Usability testing 101](https://www.nngroup.com/articles/usability-testing-101/)
- [Nielsen Norman Group: Thinking aloud — the number one usability tool](https://www.nngroup.com/articles/thinking-aloud-the-1-usability-tool/)
