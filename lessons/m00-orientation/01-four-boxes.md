---
id: m00-01-four-boxes
module: m00-orientation
slug: four-boxes
title: "The Four Boxes: Client, Server, Network, Data"
order: 1
duration_min: 15
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

A student opens the canteen app at 1:10 pm and taps "Order". A spinner turns, then the screen says "Something went wrong." She taps again — and now she has two orders, both of which have taken her money.

There are only four places this could have gone wrong. If you do not know the four, every bug feels like one unknowable mess.

### The idea

**Every app, no matter how big, is made of four parts: a client, a server, a network between them, and data that outlives both.**

The client is the part you touch. The server decides. The network is the gap they shout across. The data is what remains when everyone goes home. Instagram, UPI and the canteen app all have exactly these four parts; the difference between a college project and Swiggy is not the number of boxes but how much traffic each box survives.

### The analogy

Ordering at the campus canteen counter. **You** are the client: you read the menu, you decide, you say what you want, and you cannot cook. **The kitchen** is the server: it takes your order, checks whether it is possible ("paneer is finished"), hands something back. **The order window** is the network: narrow, noisy, crowded at 1:10 pm, the only channel between you. **The register** is the data: it remembers you paid ₹80 today even after you leave.

**Where the analogy breaks:** in a real canteen you can lean through the window and shout "make it fast". On a network you cannot. Once your order leaves you have no idea whether the kitchen got it, is working on it, or burned down — until something comes back.

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

The client sends a request across the network; the server decides and reads or writes data; a response travels back the same way. The client never touches the data directly — the most important line in this diagram.

Her double order fits on this diagram. The server saved it and wrote the row; the response never made it back across the network. She saw failure, the server saw success, and the next chapter follows that tap through every hop to show exactly where it was lost.

### Where it lives

Every new thing you learn in this program is an addition to this diagram: a cache between server and data, a load balancer in front of the server, a CDN close to the client, an AI model beside the server as something it calls, authentication guarding the door in. So when you meet a new term, ask the same question first: **which box does this live in, or which gap does it sit in?**

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

**The client cannot be trusted.** Anything on the user's phone can be read and changed by the user. If the client decides the price, someone will change it to ₹1, so the server must re-decide everything that matters. AI will happily help you break this rule.

**The network is the slow, unreliable part.** A server reads data in milliseconds; a request crossing crowded campus Wi-Fi might take two seconds, or vanish. Most "make it faster" work is really "cross it less often, carry less".

**Client and server can disagree.** She saw failure; the server saw success. Knowing that gap exists is what lets you ask the right question.

**Which box is broken?** A triage you can use from today:

| What you see | Usually which box |
|---|---|
| Looks wrong, but data is correct when you check | Client |
| Works on Wi-Fi, fails on mobile data | Network |
| Correct for you, wrong for another user | Server logic |
| Wrong even after reloading, for everyone | Data |
| Fast sometimes, slow at 1 pm | Server or data under load |

### Verify it

AI hands you code without saying which box it belongs to. Three checks before you accept it:

1. **Which box did it change?** If you cannot tell, ask: "which part of this runs on the user's device, and which on my server?"
2. **Is a decision in the client that belongs on the server?** Prices, permissions, discounts and "is this user an admin" are server decisions, always.
3. **Does the client touch the data directly?** A database key embedded in a phone app is a public key — a common AI mistake.

Red flag: "for simplicity, we'll handle this on the frontend." Sometimes fine; never for money, identity or permissions.

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
