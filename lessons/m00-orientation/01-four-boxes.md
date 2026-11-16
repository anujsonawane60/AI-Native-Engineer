---
id: m00-01-four-boxes
module: m00-orientation
slug: four-boxes
title: "The Four Boxes: Client, Server, Network, Data"
order: 1
duration_min: 25
is_free: true
mission: false
volatility: low
last_reviewed: 2026-11-16
---

## Key learnings
- Name the four parts that every app you have ever used is built from
- Say what each part is responsible for, and what it is not
- Spot which box is broken when something goes wrong
- Place a new feature in the right box before you ask AI to build it
- Use the words client, server, network and data the way engineers use them

## Lesson

### The pain

A student opens the canteen app at 1:10 pm to order a thali. The menu takes eight seconds to appear. She taps "Order". A spinner turns for a while, then the screen says "Something went wrong." She taps again. Now she has two orders and one of them has taken her money.

Somebody has to find out what happened. There are only four places it could have happened — but if you do not know the four places, every bug feels like one enormous unknowable mess. That is the fear this chapter removes.

### The idea

**Every app, no matter how big, is made of four parts: a client, a server, a network between them, and data that outlives both.**

The client is the part you touch. The server is the part that decides. The network is the gap they shout across. The data is what remains when everyone goes home. Instagram has these four parts. UPI has these four parts. The canteen app has these four parts. The difference between a college project and Swiggy is not the number of boxes; it is how much traffic each box survives.

### The analogy

Think about ordering food at the campus canteen counter.

**You** are the client. You read the menu board, you decide, you say what you want. You cannot cook.

**The kitchen** is the server. It receives your order, checks whether it is possible ("paneer is finished"), prepares it, and hands something back. The kitchen does not care who you are personally — it cares whether your order is valid and paid for.

**The order window** is the network. It is the only way you and the kitchen can communicate. It is narrow, it is noisy, and at 1:10 pm there is a crowd in front of it. Everything you want to say to the kitchen must fit through that window.

**The storeroom and the register** are the data. The register remembers that you paid ₹80 today, even after you leave, even after that cook's shift ends, even if the kitchen catches fire and is rebuilt. Data is the part with memory.

**Where the analogy breaks:** in a real canteen you can lean through the window and shout "make it fast". On a network you cannot. Once your order leaves, you have no idea whether the kitchen received it, is working on it, or burned down — until something comes back. Almost every confusing bug in software comes from that one difference, and you will meet it again in every phase of this program.

### Picture it

```
   CLIENT                NETWORK                SERVER
┌───────────┐                              ┌──────────────┐
│  phone /  │ ───── order request ───────▶ │   decides:   │
│  browser  │                              │  valid? paid?│
│           │ ◀──── order response ─────── │  allowed?    │
└───────────┘                              └───────┬──────┘
                                                   │
                                            reads / writes
                                                   │
                                                   ▼
                                           ┌──────────────┐
                                           │     DATA     │
                                           │  menu, users,│
                                           │ orders, money│
                                           └──────────────┘
```

The client sends a request across the network; the server decides what to do with it and reads or writes data; a response travels back the same way. The client never touches the data directly — that is the single most important line in this diagram.

### Where it lives

For the rest of this program, every new thing you learn will be an addition to this diagram. A cache goes between the server and the data. A load balancer goes in front of the server. A CDN goes close to the client. An AI model sits beside the server as something it calls. Authentication guards the door into the server.

So when you meet a new term — and you will meet roughly two hundred — your first question is always the same: **which box does this live in, or which gap does it sit in?** If you can answer that, you understand the term well enough to use it in a conversation. If you cannot, you have not learned it yet.

### Words to own

| Term | Plain meaning | Example |
|---|---|---|
| Client | The part running on the user's device that shows things and takes input | The canteen app on your phone; Chrome on a laptop |
| Server | A program running on a computer somewhere that answers requests and decides | The program that checks whether paneer is in stock |
| Network | The connection requests travel over | College Wi-Fi, 4G, the internet |
| Request | A message from client to server asking for something | "Give me today's menu" |
| Response | What the server sends back | The menu, or an error |
| Data / database | Stored information that survives restarts | The list of orders placed today |
| Frontend | Another word for the client side of an app | The screens you tap |
| Backend | Another word for the server and data side | The kitchen and the storeroom |
| Latency | How long a round trip takes | "The menu took 8 seconds" |

### Tradeoffs and gotchas

**The client cannot be trusted.** This is the rule beginners break most often, and AI will happily help you break it. Anything running on the user's phone can be read and changed by the user. If the client decides the price, someone will change the price to ₹1. The server must re-decide everything that matters. You will see this again properly in M07 when we do security, but start suspecting it now.

**The network is the slow, unreliable part.** A server reads data in milliseconds. A request crossing a crowded campus Wi-Fi might take two seconds, or vanish. Most "make it faster" work is really "cross the network less often, or carry less when you do".

**"Something went wrong" is not a bug report.** Our student's double order is a four-box story: her first request probably did reach the kitchen and did create an order, but the response got lost on the way back. She saw failure; the server saw success. They disagreed. Knowing that this gap exists is what lets you ask the right question instead of guessing.

**Which box is broken?** A quick triage you can use from today:

| What you see | Usually which box |
|---|---|
| Looks wrong, but data is correct when you check | Client |
| Works on Wi-Fi, fails on mobile data | Network |
| Correct for you, wrong for another user | Server logic |
| Wrong even after reloading, for everyone | Data |
| Fast sometimes, slow at 1 pm | Server or data under load |

### Verify it

When you ask AI to build or change something, it will often hand you code without saying which box it belongs to. Before you accept it, ask yourself three questions:

1. **Which box did it change?** If you cannot tell, ask it: "Which part of this runs on the user's device, and which part runs on my server?"
2. **Did it put a decision in the client that belongs on the server?** Prices, permissions, discounts and "is this user an admin" are server decisions. Always.
3. **Did it touch the data directly from the client?** If the phone talks straight to the database with a key embedded in the app, that key is public. This is a genuinely common AI mistake, not a rare one.

A red flag phrase to watch for in AI answers: "for simplicity, we'll handle this on the frontend." Sometimes that is fine. For anything involving money, identity or permissions, it is not.

## Walkthrough
1. A student taps "Order" in the canteen app. Nothing has left her phone yet — the client has only gathered what she chose.
2. The client packages the order into a short message: which items, how many, and who is ordering. This is the request.
3. The request crosses the network. At 1:10 pm it is competing with two hundred other phones on the same Wi-Fi, so this step may be the slowest part of the whole story.
4. The server receives it and starts deciding: is this a real user, is paneer still in stock, is the price she sent the price we actually charge (it recalculates rather than trusting her number).
5. The server writes to the data: a new order row, and a reduction in the paneer count. Now the order exists even if everything else crashes.
6. The server builds a response — "order #412 confirmed, ready in 10 minutes" — and sends it back across the network.
7. The client receives the response and redraws the screen. Only now does the student see anything.
8. Trace where the double-order bug fits: step 5 succeeded, step 6 or 7 failed. The order existed; she just never heard about it.

## Talk to AI
```prompt
I am learning how apps are structured. I am going to describe an app I use, and I want you to help me split it into exactly four parts: client, server, network, and data.

The app is: [name the app, e.g. UPI / Instagram / your college portal]

For each of the four parts, tell me:
1. What that part is responsible for in this specific app
2. One concrete thing it does that the other parts cannot do
3. One thing that would visibly break for me as a user if only that part failed

Keep it to plain language. Do not use any term you have not explained in the same sentence. At the end, list any term you used that I would need to look up.
```

## Stuck? Ask AI
```prompt
Explain the difference between a client and a server to me as if I have never used a computer, using an example that is not about restaurants or food. Then ask me two questions to check whether I actually understood it, and wait for my answers before telling me if I am right.
```

```prompt
I think I understand the four boxes: client, server, network, data. Quiz me with five short scenarios where something goes wrong in an app, and for each one ask me which of the four boxes is most likely at fault. Do not tell me the answer until I have guessed. Then explain why.
```

```prompt
Here is my understanding in my own words: [paste your explanation]. Tell me specifically what I got wrong or imprecise, and what important idea I left out entirely. Be blunt rather than encouraging.
```

## Quiz

### Q1 · L2
In the canteen analogy used in this chapter, what plays the role of the **network**?
- (a) The cook who prepares the food
- (b) The order window you must speak through ✅
- (c) The register that records your payment
- (d) The storeroom holding the ingredients
> The network is the only channel between client and server, and everything must fit through it. The cook is the server, the register and storeroom are data.

### Q2 · L3
A student taps "Order" once. Her money is deducted and an order is created on the server, but her screen shows "Something went wrong," so she taps again and ends up with two orders. Where did the first attempt actually fail?
- (a) The request never reached the server
- (b) The server rejected the order as invalid
- (c) The server succeeded, but the response did not get back to the client ✅
- (d) The database refused to save the order
> The order existed, so the server and database both worked. The failure was on the return trip across the network — the client and server ended up disagreeing about what happened.

### Q3 · L4
An AI assistant proposes: "To keep it simple, let the phone app calculate the final price and send that amount to the payment gateway." What is the main problem?
- (a) Phones are too slow to do multiplication
- (b) Anyone can modify what runs on their own device and pay ₹1 ✅
- (c) Payment gateways cannot accept amounts from an app
- (d) The price should be stored in the network layer
> The client cannot be trusted, because the user controls it. Anything involving money, identity or permissions must be decided again on the server.

### Q4 · L3
An app works correctly on college Wi-Fi but frequently fails on mobile data. Which box should you suspect first?
- (a) The client
- (b) The network ✅
- (c) The server
- (d) The data
> The variable that changed is the connection, not the code or the stored data. Behaviour that tracks the connection points at the network.

### Q5 · L3
Which statement about the four boxes is correct?
- (a) The client reads and writes the database directly, and the server only draws screens
- (b) The server shows the interface, and the client stores the data
- (c) The client shows and collects, the server decides, and only the server touches the data ✅
- (d) The network decides whether a request is valid before passing it on
> The client gathers input and displays results; the server holds the decisions; data access goes through the server. The network only carries messages — it never decides anything.

## Go deeper
- [MDN: How does the internet work?](https://developer.mozilla.org/en-US/docs/Learn/Common_questions/Web_mechanics/How_does_the_Internet_work)
- [MDN: Client-server overview](https://developer.mozilla.org/en-US/docs/Learn/Server-side/First_steps/Client-Server_overview)
