---
id: m12-10-a11y-localisation
module: m12-ship-apps-for-real
slug: mobile-accessibility-localisation
title: "Accessibility and Localisation on Mobile"
order: 10
duration_min: 25
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

Three real users, three failures you will never see on your own phone.

The first has low vision and keeps her Android font size at the largest setting. Your app's buttons have fixed heights, so the labels are cut in half. Your price row is a fixed-width box, so "₹1,250" becomes "₹1,2". Two screens are unusable and she cannot tell you why, because the error is invisible to anyone with default settings.

The second is blind and uses TalkBack. He swipes through your order screen and hears: "button. button. button. image. button." Your icon-only buttons have no labels, so the screen reader has nothing to read. The app is not slow or broken for him — it is simply unusable, and he will not be writing a review explaining that.

The third is a vegetable seller in Nashik who speaks Marathi, reads Hindi with effort and English barely at all. Your app is English-only. She asks her nephew to operate it for her. Your product has acquired a user who cannot use it alone, and your team counts her as an active user.

Then your team tries to fix the third one by running every string through machine translation. Nobody who speaks Hindi reads the output before it ships. One of your buttons now says something that is grammatically correct and means roughly "cancel the government", and a reviewer screenshots it.

### The idea

**Accessible and multilingual are properties of how you build your screens, not features you add at the end.**

Two commitments make almost all of it work.

**Nothing has a fixed size, and everything has a name.** Containers grow with their text so a 200% font scale pushes layout rather than clipping it. Every control a user can tap has a label a screen reader can read, even when the visible content is only an icon.

**No string is typed into a screen.** Every piece of visible text lives in a catalogue with a key, and the screen asks the catalogue for the current language. Adding Hindi then becomes one more file instead of a rewrite — and you get a complete list of your own user-facing copy for free, which is worth having anyway.

Both of these are cheap on day one and painful on day two hundred. That is the whole argument for doing them now.

### The analogy

The canteen's board is in Marathi and English, with a photograph of each dish. Someone who reads neither language can still point at a picture. The prices are large enough to read from the back of the queue. And when a student who cannot read the board at all arrives, the person at the counter reads it aloud.

The counter staff are the screen reader. Notice what they need in order to do the job: every dish has to have a *name*. If one item on the board were just a blank square with a price, the staff could only say "something, forty rupees" — which is precisely what TalkBack says about your unlabelled icon button.

**Where the analogy breaks:** the staff can improvise. They can describe a dish they have never been told the name of, by looking at it. A screen reader cannot look. It reads only what you wrote, so a missing label is total and silent — the app simply does not exist for that user, and nothing in your analytics will tell you.

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

M03 taught accessibility and design systems for the web: contrast, touch targets, not relying on colour alone, respecting reduced motion. All of it still applies. What changes on mobile is the tooling and two specific pressures.

The screen readers are **TalkBack** on Android and **VoiceOver** on iOS, and they are driven by accessibility properties you set on each element: a label, a role, a state. React Native exposes these directly. Your design system components from M03 are the right place to put them, so that a button is accessible once rather than in forty places.

**Font scaling is a system setting on every phone**, and a substantial number of your users have moved it. Android also has a separate display-size setting that scales everything. Both are far more commonly used than web zoom, which is why clipping is the single most common accessibility bug in mobile apps.

For language, the catalogue lives in your app as one file per locale and the app reads the device's language setting. Chapter 11's store listing can be localised too, which matters: a Marathi-speaking user searching the Play Store in Marathi will not find an English-only listing.

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

**Devanagari needs more vertical space than Latin.** Hindi and Marathi text has marks above and below the line, so a line height tuned for English clips them. Test every screen with real Hindi strings, not placeholder text, and give text rows room to breathe.

**Translated strings are longer, and sometimes much longer.** A button that says "Pay" in English may need three times the width in another language. This is the same bug as font scaling, which is why fixing one usually fixes the other.

**Machine translation needs a human reader, every time.** It is genuinely useful as a first draft and genuinely unsafe as a final answer, especially for short strings with no context where "Clear" could mean empty, transparent or understood. You have Hindi and Marathi speakers in your cohort. One hour of someone reading the file aloud catches what no tool will.

**Do not translate everything.** Your product name usually stays. So do units and most proper nouns. And numbers need localising rather than translating: Indian digit grouping writes 12,50,000 where English writes 1,250,000, and a user reading "1,250,000" will do a double-take.

**Never bake text into an image.** A graphic with English words in it cannot be translated, cannot be read by a screen reader, and cannot scale with the font setting. This also applies to your store screenshots in chapter 11.

**Colour alone is not a state.** A red border meaning "invalid" is invisible to a colour-blind user and to a screen reader. Pair it with text or an icon — the same rule as M03, and store reviewers do notice.

**Respect reduce-motion.** Both platforms expose a system setting for users who get motion sickness or find animation distracting. Honour it. On budget phones it also helps performance, which is chapter 9's benefit for free.

**Accessibility is also commercially useful.** A user who can operate your app alone is a user who comes back. The vegetable seller who needs her nephew is not a retained user, and M16 will show you exactly that in the retention curve.

### Verify it

**Red flag: an icon button with no accessibility label.** Scan the diff for every tappable element whose visible content is an icon, an image or an emoji. Each one needs a label. Then verify the real way: turn TalkBack on, swipe through the screen with your eyes closed, and write down what you hear. If you hear "button" with no name, you found it.

**Red flag: fixed heights and widths around text.** Search the styles for fixed height and width values on anything containing text. Then set your phone's font size to maximum and open every screen. Screenshot anything that clips, overlaps or scrolls off.

**Red flag: a string typed directly into a screen.** Grep your screens for quoted user-facing text. Each one is a string that will never be translated, because nobody will find it later. Ask AI to list every hardcoded string it can find rather than to fix them — then check the list is complete yourself.

**Red flag: machine-translated Hindi that nobody read.** The test is not technical. Hand the Hindi file to a classmate who speaks Hindi and have them read every string aloud, in order. Mark the ones that make them pause or laugh. Those are your bugs, and no automated check would have found them.

**Red flag: an accessibility "fix" that only adds labels.** Labels are necessary and not sufficient. Also check: can you reach every control by swiping, in a sensible order? Do form errors get announced when they appear? Is the focus sensible after a modal closes? Ask AI to verify the navigation order, not just the naming.

**Verify on the actual device settings.** Four settings, one pass: font size at maximum, display size at large, TalkBack on, and the system language set to Hindi. Open every screen in each state. It takes twenty minutes, and it finds more real defects per minute than anything else in this module.

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
