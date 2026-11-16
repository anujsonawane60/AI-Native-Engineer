---
id: m02-02-dom-forms
module: m02-the-web
slug: dom-forms-and-inputs
title: "The DOM as a Tree; Forms, Inputs, Links and Images"
order: 2
duration_min: 25
is_free: false
mission: false
volatility: low
last_reviewed: 2026-11-16
---

## Key learnings
- Explain the difference between the HTML file you sent and the DOM the browser built
- Say why a form input without a `name` sends nothing, silently
- Pair every input with a real label, and say why a placeholder is not one
- Choose the right input type so a budget Android shows the right keyboard
- Name the three form red flags that cost real money or real data

## Lesson

### The pain

A team ships a "Pre-order your thali" form. It has a name field, a phone field and a submit button, and it works — the page says "Thanks, order placed".

Three real failures are hiding in it. The phone field has no `name` attribute, so the phone number never leaves the browser; every order arrives with a blank number and the canteen cannot call anybody. The phone field is `type="text"`, so on an Android phone the user gets the full alphabet keyboard and has to hunt for digits. And neither field has a label — only grey placeholder text, which disappears the moment you start typing, so a user who is interrupted cannot tell which box was which.

None of this is visible in a screenshot. All of it is visible in ten seconds if you know where to look.

### The idea

**The browser reads your HTML once and builds a live tree of objects from it — the DOM — and from then on the DOM, not your file, is what you see and what everything changes.**

Forms are the one part of HTML that can send data to a server on their own, with no JavaScript at all. That makes them the oldest and most reliable interactive thing on the web, and also the place where a missing attribute causes silent data loss rather than an error.

### The analogy

The canteen's paper order slip.

The printed blank slip is your HTML file: the boxes, the labels next to each box, the dotted line for the total. The clipboard the counter staff actually work from is the DOM — it starts as a copy of the blank slip, but then it gets filled in, crossed out and corrected, and it is the clipboard that decides what the kitchen cooks. Nobody goes back and consults the original blank slip.

The `name` attribute is the little printed label next to each box. If a box has no label, the kitchen receives a number with nothing saying what it is, and throws it away. That is exactly what a browser does with an unnamed input.

**Where the analogy breaks:** a paper slip is written once and stays written. The DOM is rewritten constantly — by JavaScript (chapter 8), by data arriving from the network (chapter 10), by the user typing. So when you look at the page in DevTools you are looking at the clipboard as it is right now, which may share very little with the blank slip the server sent. That one difference is the single biggest source of confusion for beginners, and chapter 3 is entirely about it.

### Picture it

```
   HTML text            the browser parses it         the DOM
  ┌──────────┐                                   ┌───────────────┐
  │ <form>   │ ──── bytes over the network ────▶ │ form node     │
  │  <input> │                                   │  ├─ input     │
  │  <button>│                                   │  └─ button    │
  │ </form>  │                                   └───────┬───────┘
  └──────────┘                                           │
                                                         │ painted
                                      what you see  ◀────┘
                           what JavaScript changes  ◀────┘
```

HTML arrives as text, the browser turns it into a tree of live objects, and that tree is both what gets drawn on screen and what every later change operates on.

### Where it lives

The DOM lives entirely in the client box from M00. Nothing on the server ever sees your DOM; the server only ever sent text and later receives whatever the form or your JavaScript sends back.

Forms are the hinge between the client box and the server box. A `<form>` with an `action` and a `method` sends the values of its named inputs as an HTTP request — which is chapter 13. What the server then does with them, and whether it trusts them, is M04. And the rule you will meet there, stated early so it sinks in: **every validation you put in the browser is a courtesy to the user, never a security control.** Anyone can send the request without ever loading your page.

### Words to own

| Term | Plain meaning | Example |
|---|---|---|
| DOM | The live tree of objects the browser builds from your HTML | What DevTools shows you |
| Node | One item in that tree | A single `<li>` |
| Parent / child | The element containing, and the element contained | `<ul>` is parent of `<li>` |
| `<form>` | A group of inputs that can submit itself | The pre-order form |
| `name` | The label the server receives a value under | `name="phone"` |
| `<label>` | Visible text permanently bound to one input | `<label for="phone">` |
| `type` | What kind of input this is, which picks the keyboard | `tel`, `email`, `number` |
| `required` | Browser blocks submission if empty | On the phone field |
| `href` | Where a link goes | `href="/menu"` |
| `src` | Where an image's file lives | `src="/thali.jpg"` |
| Placeholder | Grey hint text inside an empty input | "e.g. 9876543210" |

### Tradeoffs and gotchas

**No `name`, no data — and no error.** This is the most expensive five-character mistake in web development. The field looks fine, the user fills it in, the form submits successfully, and the value is simply not in the request. You only find out when someone asks why half the orders have no phone number.

**A placeholder is not a label.** It vanishes when the user types, it is usually too low-contrast to read in sunlight on a cheap screen, screen readers treat it inconsistently, and when autofill puts a value in, the user has lost all record of what the box was for. Use a real `<label>` whose `for` matches the input's `id`. The side benefit is that tapping the label focuses the input, which matters a lot on a 5-inch screen.

**`type` picks the keyboard, and on a phone that is the whole experience.** `type="tel"` gives a big digit pad. `type="email"` puts `@` on the main keyboard. `type="number"` gives digits but also spinner arrows and will silently reject a leading zero — so for an Indian phone number, `type="tel"` is right and `type="number"` is wrong.

**Links and buttons do different jobs.** A link goes somewhere and must have an `href`; that is what makes it openable in a new tab, copyable, bookmarkable and crawlable. A button performs an action here. An AI will often give you a `<a>` with no `href` styled as a button, which is neither.

**Every image needs width and height.** Without them the browser does not know how much room to leave, so the page jumps when the image loads and the user taps the wrong thing. That jump has a name and a score — chapter 16's CLS.

**Enter should submit.** Inside a real `<form>` with a submit button, pressing Enter in a text field submits it. Rebuild the form out of `<div>` elements and that stops working, and nobody notices because everyone testing it uses a mouse.

### Verify it

The checks here are mostly about looking at what actually travelled, not at what the page claims.

1. **Fill the form and watch the request.** Open DevTools, go to the Network panel, submit the form, click the request and read the payload. Every field you typed must appear, under a sensible key. A missing key means a missing `name`. This single check catches the most common failure in this chapter.
2. **Check `for` and `id` pair up.** For each input, there must be a `<label for="x">` and an input with `id="x"`, spelled identically. Fastest test: click the label text and confirm the cursor jumps into the field.
3. **Open it on a phone, not a simulator.** Tap each field and look at the keyboard you get. A phone number field that offers letters is a wrong `type`.
4. **Submit it empty, then submit it with nonsense.** A form with no handling for either is a form that will show your users a blank screen or a raw error. Both states must say something useful in plain language.
5. **Press Tab from the top and then Enter.** The order should be top to bottom, with nothing skipped, and Enter in the last text field should submit.

Red flags worth naming when you push back on AI output: inputs with no `name`; placeholders used in place of labels; `type="number"` on a phone field; an `<a>` with no `href` used as a button; `<img>` with no `width`/`height`; and a form that is a pile of `<div>` elements with a click handler instead of a real `<form>`. The last one is the one that breaks Enter, autofill and the browser's own validation all at once.

One more, subtler: if the AI tells you it "added validation" and the only validation is in the browser, that is not wrong but it is not finished. Say so, and note it for M04.

## Walkthrough
1. Find a real form you did not build — your college's enquiry form, a food delivery signup, any page with a login. Open it on a laptop.
2. Right-click the first input and choose "Inspect". You are now looking at the DOM, not the file. Note the element, its `type`, its `name` and its `id`.
3. Look immediately above and below it for a `<label>`. If there is one, check that its `for` value matches the input's `id` exactly. If there is no label, the grey text you see is a placeholder.
4. Click on the label's text. If the cursor jumps into the field, the pairing is correct. If nothing happens, it is not a real label or the `for` is wrong.
5. Open the Network panel in DevTools, tick "Preserve log", then fill the form with obviously fake values and submit it. Find the request that was sent.
6. Click that request and read its payload or form data. Compare it line by line with what you typed. Write down anything you typed that did not arrive, and anything that arrived under a confusing key.
7. Now open the same page on your phone. Tap each field in turn and photograph or note the keyboard you get. Decide for each whether the `type` was chosen correctly for the data being asked for.
8. Back on the laptop, press Tab from the top of the page and walk through the whole form with the keyboard only. Note the tab order and whether Enter in the last field submits.
9. Submit the form completely empty and record exactly what the page tells you. Then submit it with a deliberately invalid value — letters in the phone field — and record that too.
10. Write a short findings list: every missing `name`, every placeholder-as-label, every wrong `type`, and what each one costs the business in one sentence.
11. Take the worst finding and write the one ACCEPTANCE line that would have caught it, in the style of M00's Director's Brief.

## Talk to AI
```prompt
CONTEXT:      I am directing AI to build web pages rather than writing them
              myself. I am working on a pre-order form for a college canteen.
              Users are students in Nashik on budget Android phones, often on
              slow 4G, frequently in bright sunlight. Some use screen readers.
              There is no backend yet, so the form should submit to a
              placeholder endpoint.

GOAL:         A pre-order form that reliably delivers every field the canteen
              needs, and that a distracted student can fill in one-handed.

REQUIREMENTS: 1. Fields: name, Indian mobile number, dish choice from six
                 options, quantity, and an optional note
              2. Every field has a real label element bound by for and id
              3. Every field has a name attribute, because the server reads
                 those keys
              4. Input types chosen so the correct phone keyboard appears
              5. A real form element with a real submit button, so Enter
                 submits and browser autofill works
              6. Clear, plain-language messages for empty and invalid input

CONSTRAINTS:  Plain HTML, no framework, no JavaScript yet. No placeholder
              text used as a substitute for a label. Quantity must not allow
              zero or negative values. Keep the whole file readable in one
              screen-and-a-half. Must be usable at 360px wide.

NON-GOALS:    No payment. No login. No styling beyond what is needed to be
              legible. Do not add client-side JavaScript validation yet, and
              do not claim the form is secure.

ACCEPTANCE:   - Submitting the filled form sends every field, each under a
                clearly named key I can read in the Network panel
              - Clicking any label moves the cursor into its field
              - On an Android phone the mobile-number field shows a digit pad
              - Tab order runs top to bottom with nothing skipped, and Enter
                in the last text field submits
              - Submitting it empty produces a message naming which field is
                missing, not a blank screen

ASK:          Before writing any markup, list each field with the element,
                the type, the name and the label text you intend to use, as
                a table, and flag any choice where a reasonable developer
                would disagree. Wait for my approval. Then output the HTML,
                and separately tell me which of my acceptance criteria your
                markup does NOT yet satisfy and why.
```

## Stuck? Ask AI
```prompt
Explain the difference between the HTML file a server sends and the DOM the browser builds, using an analogy that is not about trees or family relationships. Then give me five situations and ask me whether each one changes the file, the DOM, or both, without telling me the answers first.
```

```prompt
Here is a form's markup: [paste it]. Do not rewrite it. For each input, tell me in a table: does it have a name, does it have a real bound label, is its type right for the data, and what exactly breaks for the user or the business if it is wrong. Rank the problems by what they cost.
```

```prompt
I do not understand why people say browser validation is not security. Walk me through, step by step and concretely, how someone could send data to my server without ever loading my page, and what that means I must do on the server side instead. Do not give me code, give me the reasoning.
```

## Quiz

### Q1 · L2
What is the DOM?
- (a) The HTML file as stored on the server
- (b) The live tree of objects the browser builds from the HTML, and then draws and changes ✅
- (c) The CSS rules currently applied to the page
- (d) The list of network requests a page has made
> The file is text and is sent once. The DOM is built from it, is what gets painted, and is what JavaScript and the user then modify — so it can drift far from the original file.

### Q2 · L3
A student fills in a working form and submits it. The name arrives at the server but the phone number does not, and no error appears anywhere. What is the most likely cause?
- (a) The phone field is missing its `name` attribute ✅
- (b) The phone field is missing its `required` attribute
- (c) The phone number was too long for the field
- (d) The browser blocked the phone number for privacy
> A form sends only the values of inputs that have a `name`. Without one, the field is simply omitted from the request — successfully, silently, and with nothing in the UI to hint at it.

### Q3 · L3
Why is a placeholder a poor substitute for a label on a budget Android phone?
- (a) Placeholders cannot be translated
- (b) It disappears when the user types, is low-contrast in sunlight, and leaves autofilled fields unlabelled ✅
- (c) Placeholders prevent the form from submitting
- (d) Placeholders are ignored by the browser
> All three problems compound on a cheap screen outdoors. A real `<label>` also makes the text a tap target that focuses the field, which matters on a 5-inch screen.

### Q4 · L4
For an Indian ten-digit mobile number, which input type is the right choice and why?
- (a) `type="number"`, because the value is numeric
- (b) `type="text"`, because it accepts anything
- (c) `type="tel"`, because it shows a digit pad without mangling leading zeros or adding spinners ✅
- (d) `type="search"`, because it gives a clean keyboard
> `type="number"` is for quantities: it adds spinner arrows and treats the value as a number, which breaks leading zeros and formatting. `tel` is designed for phone numbers.

### Q5 · L4
An AI tells you it "added validation" to your form, and all of it is in the browser. What is the correct response?
- (a) Accept it — browser validation is sufficient
- (b) Accept it as good UX, and record that server-side validation is still required ✅
- (c) Reject it and remove the browser validation
- (d) Ask it to move all validation into the database
> Browser validation improves the experience and prevents honest mistakes. It stops nothing deliberate, because a request can be sent without ever loading your page — so the server must check again, which is M04's job.

## Go deeper
- [MDN: Your first form](https://developer.mozilla.org/en-US/docs/Learn_web_development/Extensions/Forms/Your_first_form)
- [MDN: Introduction to the DOM](https://developer.mozilla.org/en-US/docs/Web/API/Document_Object_Model/Introduction)
