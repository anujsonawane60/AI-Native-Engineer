---
id: m12-10-a11y-localisation
module: m12-ship-apps-for-real
slug: mobile-accessibility-localisation
title: "Accessibility and Localisation on Mobile"
order: 10
duration_min: 15
is_free: false
mission: false
volatility: low
last_reviewed: 2026-11-16
---

## Key learnings
- Make every control announce itself correctly to a screen reader
- Survive a user whose font size is set to 200%
- Move every string out of your screens and into a catalogue
- Localise properly for Hindi and Marathi, not just translate word by word
- Spot an icon button with no label and a machine translation nobody read

## Lesson

### The pain

Three users, three failures you will never see on your own phone.

The first has low vision and keeps her Android font size at the largest setting. Your buttons have fixed heights, so labels are cut in half, and your fixed-width price row turns "₹1,250" into "₹1,2". The bug is invisible to anyone with default settings.

The second is blind and uses TalkBack. He swipes through your order screen and hears "button. button. button. image. button." Your icon-only buttons have no labels, so the app is unusable — and he will not write a review explaining that.

The third is a vegetable seller in Nashik who speaks Marathi and reads English barely at all. Your app is English-only, so she asks her nephew to operate it, and your team counts her as an active user. Then you machine-translate every string and nobody who speaks Hindi reads the output. One button now means roughly "cancel the government", and a reviewer screenshots it.

### The idea

**Accessible and multilingual are properties of how you build your screens, not features you add at the end.**

**Nothing has a fixed size, and everything has a name.** Containers grow with their text, so a 200% font scale pushes layout rather than clipping it. Every control a user can tap has a label a screen reader can read, even when the visible content is only an icon.

**No string is typed into a screen.** Every piece of visible text lives in a catalogue with a key, and the screen asks the catalogue for the current language. Adding Hindi is then one more file instead of a rewrite.

Both are cheap on day one and painful on day two hundred.

### The analogy

The canteen's board is in Marathi and English, with a photograph of each dish, and prices large enough to read from the back of the queue. When a student who cannot read the board arrives, the person at the counter reads it aloud. The counter staff are the screen reader — and notice what they need: every dish must have a *name*. A blank square with a price forces them to say "something, forty rupees", which is precisely what TalkBack says about your unlabelled icon button.

**Where the analogy breaks:** the staff can improvise and describe a dish by looking at it. A screen reader cannot look. It reads only what you wrote, so a missing label is total and silent, and nothing in your analytics will tell you.

### Picture it

```
 YOUR SCREEN                      WHAT THE USER GETS
┌────────────────────┐
│ <icon btn, no      │──TalkBack──▶ "button"            ✗ unusable
│   label>           │
├────────────────────┤
│ <icon btn,         │──TalkBack──▶ "Add to order,      ✓
│   label="Add to    │               button"
│   order">          │
├────────────────────┤
│ fixed height 44px, │──200% font─▶ "Confirm ord"       ✗ clipped
│ text "Confirm      │
│  order"            │
├────────────────────┤
│ height: auto,      │──200% font─▶ wraps to two lines  ✓
│ min 48px           │               and stays readable
├────────────────────┤
│ t("confirm_order") │──Marathi───▶ reads from mr.json  ✓
└────────────────────┘
```

The same three screen decisions — a label, a flexible height, a string key — are what separate a control that works for every user from one that silently excludes people.

### Where it lives

M03 taught accessibility for the web — contrast, touch targets, not relying on colour alone — and all of it still applies. What changes on mobile is the tooling and two pressures.

The screen readers are **TalkBack** on Android and **VoiceOver** on iOS, driven by accessibility properties you set on each element: a label, a role, a state. Put them in your M03 design system components, so a button is accessible once rather than in forty places.

**Font scaling is a system setting on every phone**, and many of your users have moved it. Android also has a separate display-size setting. Both are used far more than web zoom, which is why clipping is the most common accessibility bug in mobile apps.

For language, the catalogue is one file per locale and the app reads the device's language setting. Chapter 11's listing can be localised too: a Marathi-speaking user searching the Play Store in Marathi will not find an English-only listing.

### Words to own

| Term | Plain meaning | Example |
|---|---|---|
| Screen reader | Reads the interface aloud | TalkBack, VoiceOver |
| Accessibility label | The text a screen reader announces | "Add to order" |
| Accessibility role | What kind of control this is | button, link, header |
| Font scaling | The user's system text-size setting | 200% is common |
| Touch target | Tappable area, not just the icon | 48dp minimum on Android |
| Contrast ratio | Text-to-background difference | 4.5:1 for body text |
| i18n | Building so languages can be swapped | String catalogue, no hardcoding |
| l10n | Actually producing one language | `hi.json` reviewed by a speaker |
| Locale | Language plus region conventions | `mr-IN`, dates, ₹ grouping |
| Pluralisation | Different wording per count | "1 item" vs "2 items" |
| RTL | Right-to-left layout | Urdu mirrors the whole screen |

### Tradeoffs and gotchas

**Devanagari needs more space than Latin, in both directions.** Hindi and Marathi have marks above and below the line, so a line height tuned for English clips them, and a button saying "Pay" may need three times the width. This is the same bug as font scaling, which is why fixing one usually fixes the other. Test with real Hindi strings, not placeholder text.

**Machine translation needs a human reader, every time.** Useful as a first draft, unsafe as a final answer — especially for short strings with no context, where "Clear" could mean empty, transparent or understood. A Hindi speaker in your cohort reading the file aloud catches what no tool will.

**Do not translate everything.** Your product name, units and most proper nouns stay. And numbers need localising rather than translating: Indian digit grouping writes 12,50,000 where English writes 1,250,000.

**Never bake text into an image.** It cannot be translated, read by a screen reader, or scaled. This also applies to chapter 11's store screenshots.

**Colour alone is not a state.** A red border meaning "invalid" is invisible to a colour-blind user and to a screen reader. Pair it with text or an icon — the same rule as M03, and store reviewers notice.

### Verify it

**Red flag: an icon button with no accessibility label.** Scan the diff for every tappable element whose visible content is an icon or an image. Then verify the real way: turn TalkBack on, swipe through the screen with your eyes closed, and write down what you hear.

**Red flag: fixed heights and widths around text.** Search the styles for fixed height and width on anything containing text, then set the font size to maximum and screenshot anything that clips.

**Red flag: a string typed directly into a screen.** Each one will never be translated, because nobody will find it later. Ask AI to *list* every hardcoded string rather than fix them, then check the list yourself.

**Red flag: machine-translated Hindi that nobody read.** Have a classmate who speaks Hindi read every string aloud, in order. The ones that make them pause or laugh are your bugs.

**Red flag: a "fix" that only adds labels.** Also check that every control is reachable by swiping in a sensible order, that form errors are announced, and that focus is sensible after a modal closes.

**Verify on the actual device settings.** Four settings, one pass: font size at maximum, display size at large, TalkBack on, system language Hindi. Twenty minutes, and it finds more real defects per minute than anything else in this module.

## Walkthrough
1. On a real Android phone, set the font size and display size to their largest settings. Open every screen in your app and screenshot everything that clips, overlaps or disappears.
2. Fix the worst three by removing fixed heights and widths from anything containing text, and letting containers grow.
3. Turn on TalkBack. Put the phone down, close your eyes, and swipe through your main screen. Write down exactly what you hear, in order.
4. Add an accessibility label and role to every control that announced itself as just "button" or said nothing at all. Start with your design system components from M03 so each fix lands everywhere.
5. Check that your tap targets are at least 48dp, and that the tappable area is the whole row or button rather than just the icon.
6. Find every hardcoded user-facing string in your screens and list them. Count them; the number is usually larger than your team expects.
7. Move every one of those strings into a catalogue with a key, and make your screens read from the catalogue with the device language.
8. Produce a Hindi or Marathi file. Use machine translation as a first draft, then have a classmate who speaks that language read every string aloud and correct it.
9. Set the phone's system language to Hindi and open every screen. Look specifically for clipped Devanagari marks, overflowing buttons and English strings you missed.
10. Localise your number, date and currency formatting, including Indian digit grouping for large amounts.
11. Record in your decision log which languages you support, who reviewed each one, and the three accessibility defects you fixed.

## Talk to AI
```prompt
CONTEXT:      React Native app built with Expo, Android first, users in
              Nashik and around Maharashtra. Product: [one sentence].
              Our users: students and small vendors on budget Android
              phones. Many prefer Marathi or Hindi; some read English with
              difficulty. Some have low vision and use large system fonts.
              Our design system components: [list the shared ones, e.g.
              Button, Card, ListRow, PriceTag]
              Screens in the app: [list them]
              Languages today: English only, strings written directly in
              the screens.

GOAL:         Every control announces itself correctly to TalkBack, no
              screen clips at 200% font scale, and adding a language is one
              file rather than a rewrite.

REQUIREMENTS: 1. For each of my design system components, the accessibility
                 properties it must expose and what a screen reader should
                 announce for it
              2. A rule for sizing anything containing text so a 200% font
                 scale wraps instead of clipping, with the specific style
                 patterns to avoid
              3. A plan to extract every hardcoded string into a catalogue:
                 the structure, the key naming convention, and how I find
                 strings I have missed
              4. Guidance specific to Devanagari: line height, vertical
                 space, and what breaks that would not break in English
              5. What to localise beyond words: numbers with Indian digit
                 grouping, currency, dates, and pluralisation
              6. A list of what must NOT be translated and why
              7. A twenty-minute manual test plan on a real Android phone
                 covering font size, display size, TalkBack and system
                 language, with what to look for in each state

CONSTRAINTS:  Free tiers only. No paid translation service. We have Hindi
              and Marathi speakers in our cohort who can review. Budget
              Android phones, so avoid anything that costs performance.
              Assume some users rely entirely on a screen reader.

NON-GOALS:    Do not redesign our screens. Do not add an RTL language yet.
              Do not propose a paid localisation platform. Do not translate
              our product name.

ACCEPTANCE:   - Every component in my list has a stated screen-reader
                announcement
              - Your sizing rule names the specific style properties that
                cause clipping
              - Your extraction plan includes how I detect strings I missed
              - You have named at least three things that break in
                Devanagari but not in English
              - Your test plan is runnable in twenty minutes on one phone
              - You have told me explicitly that machine translation needs
                a human reviewer, and where that risk is highest

ASK:          Start with the component-by-component accessibility table and
              the sizing rule. No code until I have agreed what each
              component should announce.
```

## Stuck? Ask AI
```prompt
Explain what a screen reader actually does on a phone and what information it needs from the app in order to be useful. Then describe exactly what a blind user experiences on a screen made of five icon-only buttons with no labels. Ask me to predict what they hear before you tell me.
```

```prompt
Here are the visible strings from one screen of my app: [paste them]. Translate them into Hindi as a first draft, then for each one tell me how confident you are, what context you were missing, and which ones a human reviewer must check most carefully. Flag any short string where the English word is ambiguous.
```

```prompt
My app looks fine normally but breaks when a user sets their phone font size to the largest setting. Explain why this happens, which style choices cause it, and the three patterns I should use instead. Then give me a checklist I can run on each screen in under two minutes to find the problem before my users do.
```

## Quiz

### Q1 · L3
A user with TalkBack swipes through your screen and hears "button. button. button." What is missing?
- (a) A heading structure on the screen
- (b) Accessibility labels on icon-only controls ✅
- (c) Higher contrast on the button backgrounds
- (d) Larger touch targets
> A screen reader can only read what you wrote. An icon conveys nothing to it, so every control whose visible content is an icon needs a label naming the action.

### Q2 · L3
Why is clipping the most common accessibility bug in mobile apps specifically?
- (a) Mobile screens are small
- (b) Font and display size are system settings that many users have increased, and fixed-size containers cannot grow with the text ✅
- (c) React Native does not support text wrapping
- (d) Android reduces font size on budget devices
> Unlike web zoom, phone font scaling is a routine setting on every device. Combined with fixed heights and widths, it cuts labels in half for a user you will never meet.

### Q3 · L4
Your team machine-translates every string into Hindi and ships it the same afternoon. What is the risk?
- (a) The app bundle becomes too large
- (b) Short strings with no context mistranslate confidently, and nobody who reads Hindi has checked ✅
- (c) Hindi cannot be rendered on Android without a font download
- (d) Translation breaks the string catalogue structure
> Machine translation is a good first draft and an unsafe final answer, especially for one-word buttons where "Clear" could mean three different things. A cohort member reading the file aloud costs an hour.

### Q4 · L4
Which of these must NOT be translated?
- (a) Button labels
- (b) Error messages
- (c) Your product's name ✅
- (d) Empty-state explanations
> Product names, units and most proper nouns stay as they are. Numbers, by contrast, need localising rather than translating — Indian grouping writes 12,50,000 where English writes 1,250,000.

### Q5 · L5
An AI "fixes" your accessibility by adding labels to every control and declares the screen accessible. What is still unverified?
- (a) Nothing; labels are the whole requirement
- (b) Swipe order, whether errors are announced when they appear, and where focus lands after a modal closes ✅
- (c) Whether the labels are spelled correctly
- (d) Whether the app supports Hindi
> Labels are necessary and not sufficient. Accessibility is about navigating and understanding the screen over time, which is why the real verification is turning TalkBack on and using the app with your eyes closed.

## Go deeper
- [Android developers: make apps more accessible](https://developer.android.com/guide/topics/ui/accessibility/apps)
- [Expo: localization with expo-localization](https://docs.expo.dev/versions/latest/sdk/localization/)
