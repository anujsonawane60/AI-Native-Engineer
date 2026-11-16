---
id: m19-11-wifi-ble-lora-mqtt
module: m19-frontier-electives
slug: wifi-ble-lora-mqtt
title: "Wi-Fi, BLE, LoRa and MQTT"
order: 11
duration_min: 25
is_free: false
mission: false
volatility: low
last_reviewed: 2026-11-16
---

## Key learnings
- Choose a radio from range, power and payload rather than from familiarity
- Explain why LoRa's tiny payloads rule out some designs entirely
- Describe MQTT topics and quality of service as the device-side cousin of M05's pub/sub
- Design for duplicates, gaps and out-of-order arrival as normal conditions
- Spot an AI device design with an unencrypted broker or an impossible payload

## Lesson

### The pain

Three nodes, three different outcomes. The one on the canteen counter works perfectly on the canteen Wi-Fi. The one in the cold store, 300 metres away through two brick walls, connects for an hour and then drops for the rest of the day. And the one the agriculture department wanted in a grape field four kilometres out has no Wi-Fi within reach at all, no mains, and a battery that the Wi-Fi radio flattens in two days of trying to reconnect.

Then the server side goes wrong in a quieter way. The nodes that do report sometimes send the same reading twice, because the node was not sure its message got through and tried again. Sometimes a reading from 2:10 pm arrives after the one from 2:11 pm. Sometimes a node is silent for four hours and then dumps everything it buffered. The dashboard shows a tidy line chart, which is a lie.

Radios and messaging are two separate choices, both easy to get wrong, and both of them wrong in ways that look like software bugs.

### The idea

**Pick the radio from range, power and payload size; then pick the messaging pattern, which for devices is almost always publish and subscribe through a broker.**

Four radios cover nearly everything a student will meet. Wi-Fi: tens of metres, high data rate, power-hungry, needs credentials for each site. Bluetooth Low Energy: ten metres or so, very low power, but it needs a phone or a gateway nearby to be useful. LoRa: kilometres, extremely low power, and payloads of tens of bytes sent a few times an hour. Cellular, including the low-power variants, works wherever a SIM works and costs money per node per month.

Then MQTT. A device publishes a short message to a named topic on a broker, and anything interested subscribes to that topic. This is exactly M05's publish and subscribe, shrunk to fit a device with half a megabyte of memory, and the mental model transfers completely.

### The analogy

Canteen communications. Shouting across the table reaches the next person and nobody else — that is BLE. The kitchen intercom reaches every room in the building and runs off the mains — that is Wi-Fi. The notice board at the main gate can be read by the whole campus, but it holds one short line and gets rewritten twice a day — that is LoRa. And the order spike where each cook watches only the slips for their own station, while the biller clips slips on without knowing who will read them, is MQTT: topics, publishers and subscribers.

**Where the analogy breaks**, and this is the break that costs student projects the most: the intercom does not flatten its own battery. A radio is often the largest single consumer in a battery node, and connecting is more expensive than transmitting — a Wi-Fi node that wakes, scans, associates, gets an address and does a TLS handshake has spent most of its energy before it sends a single byte. Second, the notice board holds one line because somebody chose that; LoRa holds tens of bytes because of physics and regulation, and no amount of wanting will make it carry a firmware update.

### Picture it

```
            range ──▶
  power
   ▲    ┌───────────────────────────────────────────┐
   │    │ Wi-Fi: 10-50 m, megabits, mains or short  │
  high  │        battery life, per-site credentials │
   │    ├───────────────────────────────────────────┤
   │    │ Cellular / NB-IoT: anywhere a SIM works,  │
   │    │        monthly cost per node              │
   │    ├───────────────────────────────────────────┤
   │    │ BLE: 10 m, low power, needs a gateway or  │
   │    │        a phone nearby                     │
  low   ├───────────────────────────────────────────┤
   │    │ LoRa: 2-10 km, tens of bytes, hours of    │
   ▼    │        silence between messages, years on │
        │        a battery                          │
        └───────────────────────────────────────────┘
```

Each radio trades range against data rate and power, and the honest reading of this picture is that long range and large payloads are not available together at low power.

```
  MQTT: nobody addresses anybody

  node A ──publish──▶┌────────┐◀──subscribe── dashboard
  node B ──publish──▶│ broker │◀──subscribe── alert service
  node C ──publish──▶└────────┘◀──subscribe── data store
                      topics:
                      canteen/fridge/3/temp
                      canteen/fridge/3/status
```

Publishers name a topic rather than a recipient and subscribers choose topics rather than senders, so adding a new consumer of fridge data requires no change to any device.

```
  what the server must expect as NORMAL

  duplicate   same reading twice   ──▶ idempotent writes
  gap         four hours missing   ──▶ "last heard" timestamp
  out of order 2:11 before 2:10    ──▶ trust the device clock
                                        only if it is set
  burst       100 buffered at once ──▶ rate limits, backpressure
```

Duplicates, gaps, reordering and bursts are ordinary behaviour for devices rather than bugs, so the server design must absorb all four without producing a misleading chart.

### Where it lives

This is M05 with a radio attached. The queue and pub/sub chapter gave you the pattern, the delivery guarantees and the reason at-least-once delivery means duplicates. MQTT's quality-of-service levels are the same three options you already met: fire and forget, at least once, and exactly once at the cost of extra round trips. Most device fleets use at-least-once and make every write idempotent, which is the same conclusion M05 reached.

M02 is the other half. MQTT normally runs over TCP, and over TLS when you configure it to — on the default unencrypted port everything including credentials crosses the network in the clear. That is chapter 12's subject and it starts here.

Indian specifics worth knowing. The 865 to 867 MHz band is available licence-free for low-power devices, which is why LoRa deployments for agriculture and cold chain are practical here; a single gateway on a tall building or a solar pole can cover a cluster of villages. Cellular low-power coverage depends entirely on the operator in that district, so it is a thing to verify on the ground rather than from a coverage map. And for a cold-chain truck moving between districts, cellular is usually the only honest answer, with buffering for the stretches with no signal.

### Words to own

| Term | Plain meaning | Example |
|---|---|---|
| Wi-Fi | Short-range, high-rate, power-hungry radio | The canteen access point |
| BLE | Very low power, very short range | A phone talking to a sensor |
| Gateway | A device that bridges a local radio to the internet | A LoRa gateway on a roof |
| LoRa / LoRaWAN | Long range, tiny payloads, the network around it | A field node 4 km out |
| Duty-cycle limit | A regulatory cap on how often you may transmit | Why LoRa nodes stay quiet |
| NB-IoT / LTE-M | Low-power cellular for devices | A SIM in a moving truck |
| MQTT | A lightweight publish and subscribe protocol | What most fleets use |
| Broker | The server that routes messages between topics | Hosted or self-run |
| Topic | A named channel in a hierarchy | canteen/fridge/3/temp |
| QoS 0 / 1 / 2 | Fire and forget, at least once, exactly once | Most fleets use 1 |
| Retained message | The broker keeps the last value for new subscribers | A dashboard's first reading |
| Last will | A message the broker sends if a device vanishes | "fridge 3 offline" |
| Backhaul | The link from the gateway onward to the internet | The roof gateway's SIM |
| Idempotent | Applying it twice has the same effect as once | Safe handling of duplicates |

### Tradeoffs and gotchas

**Connecting costs more than sending.** On Wi-Fi, scan plus association plus address plus TLS handshake can dominate a node's energy use. A node that reports every ten seconds and one that reports once an hour may have nearly the same battery life if the second one reconnects each time.

**LoRa cannot carry what people assume.** Tens of bytes, a few messages an hour, with duty-cycle limits. No JSON blobs, no images, no firmware updates. Designs that reach for LoRa for range and then send structured text are the most common failure in this chapter.

**BLE always needs something nearby.** It is a superb link between a sensor and a phone in the same room and a non-answer for an unattended node, unless you are also deploying a gateway.

**At-least-once means duplicates, always.** Plan for them with an identifier per reading and writes that tolerate repetition. The alternative — exactly-once delivery — costs more round trips and more battery, and still does not save you from a node that resends after a reboot.

**Clocks drift and nodes boot with no clock at all.** A microcontroller with no network time thinks it is 1 January 1970. Timestamp at the server as well as the device, and keep both.

**Per-site Wi-Fi credentials do not scale.** Nine fridges in one canteen is fine. Two hundred nodes across forty villages, each with a different access point and a password that changes, is an operational problem that pushes you to cellular or LoRa for reasons that have nothing to do with radio physics.

### Verify it

**Red flag: LoRa plus a large payload.** The diagnostic test of this chapter. If a design chooses LoRa for range and then sends anything longer than a few dozen bytes, or proposes firmware updates over it, the author does not know what LoRa is. Ask for the payload size in bytes and the messages per hour, and check them against the duty-cycle limit.

**Red flag: MQTT on the default unencrypted port.** Ask whether TLS is used, which port, and how the device authenticates. "Username and password over port 1883" means the credentials are readable by anyone on the path.

**Red flag: one shared credential for the whole fleet.** Ask whether each device has its own identity. One shared password means one compromised device compromises every device, and you cannot revoke a single node.

**Red flag: no handling for duplicates, gaps or reordering.** Ask what the server does with the same reading arriving twice, and what the dashboard shows when a node is silent for four hours. If the answer is "it won't happen", the design has not met a real device.

**Red flag: permanent connectivity assumed.** Ask what the node does while offline: buffer how many readings, in what memory, and what it discards when full. A node with no offline story loses exactly the data from the incident you cared about.

## Walkthrough
1. Open a free hosted MQTT test broker's web client. Subscribe to a topic under a prefix you invent for yourself, so you are not reading other people's traffic.
2. From a second browser tab or a second client, publish a short message to that topic. Confirm it appears in the subscriber. Note that neither side addressed the other.
3. Publish three messages with a wildcard subscription active, using topics that differ only in the last level. Record which messages the wildcard received.
4. Publish a message with the retained flag set. Disconnect your subscriber, reconnect it, and record whether the old message arrives again.
5. Set a last-will message on a client, then disconnect it abruptly rather than cleanly. Record what the broker sent and how long it took.
6. Publish the same message twice at quality of service one. Write down what your subscriber received and what your server would have to do to stay correct.
7. In a free in-browser microcontroller simulator, make a simulated node publish a temperature reading to your topic every ten seconds, and watch it arrive in your web client.
8. Count the bytes in the message your node sends. Write that number down, then look up a typical LoRa maximum payload and say in one line whether your message would fit.
9. Design a daily message budget for a field node on LoRa: payload in bytes, messages per hour, total bytes per day, and whether the duty-cycle limit allows it.
10. Finish with four lines: the radio you would choose for the cold store 300 metres away, the one for the field 4 km away, the one for a truck crossing districts, and the single deciding factor in each case.

## Talk to AI
```prompt
CONTEXT:      I am a student in India with no hardware, working in free
              in-browser simulators and on a free hosted MQTT broker. I do
              not write code; I need to judge a connectivity design.

              Three nodes to connect: one on a canteen counter with Wi-Fi in
              range; one in a cold store 300 m away through brick walls with
              no Wi-Fi and no mains; one in a grape field 4 km from the
              nearest building, battery and solar only.

GOAL:         A defensible radio and messaging choice for each node, with
              the numbers behind it.

REQUIREMENTS: 1. A table of the three nodes with the chosen radio, the
                 single deciding factor, the expected payload size in
                 bytes, the reporting interval, and the main failure mode
              2. For the LoRa node, a message budget: payload bytes,
                 messages per hour, bytes per day, and whether duty-cycle
                 limits allow it - say which limit you assumed
              3. An MQTT design: the topic hierarchy, which quality of
                 service and why, whether retained messages and a last will
                 are used, and how each device authenticates
              4. What the server must do about duplicates, gaps,
                 out-of-order arrival and a sudden burst from a node that
                 was buffering
              5. What each node does while offline: how many readings it
                 buffers, where, and what it discards first
              6. One paragraph on how this changes if the fleet grows from
                 three nodes to two hundred across forty villages

CONSTRAINTS:  No code. Free and simulator-based only; assume I cannot buy a
              gateway. Payloads in bytes, not adjectives. Assume no mains
              power at two of the three sites. Treat Indian licence-free
              band rules as something I must verify.

NON-GOALS:    Do not design the dashboard. Do not pick a cloud vendor. Do
              not propose a custom protocol.

ACCEPTANCE:   - Every radio choice names one deciding factor
              - Your LoRa budget has a byte count and a stated duty-cycle
                assumption
              - Your MQTT design says explicitly whether TLS is used and
                how each device is identified
              - You state what the dashboard shows when a node has been
                silent for four hours

ASK:          Give me the three-node table first. If any of my three
              requirements is impossible with the radio you chose, say so
              before designing anything else.
```

## Stuck? Ask AI
```prompt
Explain to me, as if I have never seen a computer, why a radio that reaches four kilometres can only send a few words at a time while one that reaches thirty metres can send a video. Use one everyday comparison, no formulas, under 200 words, and end with the one trade-off I should remember.
```

```prompt
I do not understand publish and subscribe. Explain it using a canteen order spike where cooks watch only their own station's slips, then ask me what happens when I want to add a new screen showing only cold drinks orders. Wait for my answer before telling me whether I am right.
```

```prompt
Give me five device messages arriving at a server: one duplicate, one four hours late, one out of order, one with a 1970 timestamp, one from a node that had buffered a hundred readings. Ask me what each one would do to a line chart and what check would have caught it. Do not reveal your answers until I have done all five.
```

## Quiz

### Q1 · L2
What is LoRa's defining limitation?
- (a) It only works indoors
- (b) Very small payloads sent infrequently ✅
- (c) It requires a phone nearby
- (d) It cannot be encrypted
> LoRa buys kilometres of range and years of battery life by sending tens of bytes a few times an hour. That is why firmware updates and structured text are off the table.

### Q2 · L3
A battery node on Wi-Fi reports once an hour and dies in a week. What is the most likely cause?
- (a) The sensor draws too much current
- (b) Reconnecting each time costs most of the energy budget ✅
- (c) One message an hour is too frequent for Wi-Fi
- (d) The broker is rejecting messages
> Scan, associate, address and handshake can dominate a node's energy use. Connecting is far more expensive than sending, which is why reporting interval alone does not predict battery life.

### Q3 · L3
Your fleet uses MQTT at quality of service one. What must the server be designed for?
- (a) Messages arriving exactly once, in order
- (b) Duplicate messages, so writes must be idempotent ✅
- (c) Messages being silently dropped with no retry
- (d) Each device holding an open connection permanently
> At-least-once delivery means a device that is unsure will resend. The standard answer is an identifier per reading and writes that produce the same result when applied twice.

### Q4 · L4
An AI designs a farm sensor on LoRa that sends a structured text payload of about 200 bytes every thirty seconds. What is wrong?
- (a) 200 bytes is too small to be useful
- (b) The payload is far too large and the interval far too frequent for LoRa's limits ✅
- (c) LoRa cannot reach a farm
- (d) Text payloads are not supported by LoRa
> LoRa payloads are tens of bytes and duty-cycle rules keep transmissions rare. This design would not work, and it is the most common LoRa mistake in AI-produced plans.

### Q5 · L4
A device design uses MQTT with one username and password shared across the whole fleet, on the default port. Name the two problems.
- (a) The topic hierarchy is wrong and QoS is too high
- (b) Credentials travel unencrypted, and one compromised device compromises every device ✅
- (c) The broker cannot handle more than one client per credential
- (d) Retained messages would be disabled
> The default port carries no TLS, so credentials are readable on the path. A shared credential also means you cannot revoke a single node, which chapter 12 treats as a serious failure.

## Go deeper
- [HiveMQ: MQTT Essentials](https://www.hivemq.com/mqtt-essentials/)
- [The Things Network: LoRaWAN architecture](https://www.thethingsnetwork.org/docs/lorawan/architecture/)
