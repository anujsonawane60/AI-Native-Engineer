---
id: m19-14-kinematics-slam-ros2
module: m19-frontier-electives
slug: kinematics-slam-ros2
title: "Kinematics, SLAM, ROS 2 and Embodied AI"
order: 14
duration_min: 25
is_free: false
mission: true
volatility: medium
last_reviewed: 2026-11-16
---

## Key learnings
- Describe the three layers a moving machine needs: geometry, localisation, framework
- Explain odometry drift and why loop closure exists
- Recognise ROS 2's nodes and topics as M05's pub/sub in a robot
- Define what a physical agent must do when it no longer knows where it is
- Spot a robotics plan that assumes perfect sensors or only works in simulation

## Lesson

### The pain

The canteen's delivery trolley robot is given a route: counter, hall, hostel corridor, back. It works for nine days.

On the tenth, a plate of dal is spilled near the hall entrance. The robot's wheels slip for half a second, so its wheel-turn count says 80 centimetres when it travelled 40. Nothing reports an error. Every later measurement is added to that wrong starting point, so by the hostel corridor it is off by a metre and a half and turns into a doorframe. The chairs were also rearranged for an event, so the map it navigates by is yesterday's hall.

Nobody wrote a bug. Every component did its job. Moving a body through a real room means being wrong about where you are, continuously, and the engineering is about managing that wrongness.

### The idea

**Moving a machine needs three layers: geometry, localisation and mapping, and a framework that lets the pieces talk. Embodied AI sits on top and inherits every weakness below it.**

Geometry comes in two directions. Forward: given the joint angles, where did the hand end up — always one answer. Inverse: given where you want the hand, what angles get it there — many answers, none, or one passing through a pose where the arm loses a direction of motion, called a singularity. Inverse is the hard direction.

Localisation and mapping is the dal problem. Counting wheel turns is odometry, and its errors accumulate, so the estimate drifts and never returns on its own. Simultaneous localisation and mapping fuses odometry with a sensor that sees the room — a laser scanner, a camera — to correct against landmarks.

### The analogy

A canteen waiter crosses a crowded hall with a loaded tray and his eyes shut, counting paces. For six paces he is nearly right. By twenty he is a metre out, and the error only grows, because each pace's error adds to the previous ones. Open his eyes for one second: he sees the pillar, recognises it, and corrects everything at once. That second is loop closure.

**Where the analogy breaks**, in two ways that are the chapter. The waiter knows when he has stumbled; a wheel encoder reports a confident number while the wheel spins in spilt dal. And the waiter's memory of the hall updates when the chairs move, while the robot's map is a file from yesterday that it trusts until someone replaces it.

### Picture it

```
  wheel turns ──▶ odometry ──┐
                             ├──▶ pose estimate ──▶ map update
  laser scan  ──▶ features ──┘         │
                                       ▼
                        recognised a known place?
                             │ yes ──▶ loop closure:
                             │         correct the whole
                             │         accumulated path
                             └ no  ──▶ drift keeps growing
```

A position estimate is built by fusing wheel counts with what a scanner sees, and recognising an already-mapped place is what lets the system repair the drift it has silently accumulated.

```
  drift, over one trip

  truth     ●───────●───────●───────●───────●
  estimate  ●──────●──────●─────●────●      ●
                   ↑                        ↑
              small error              a metre and a half,
              nobody noticed           reported with the same
                                       confidence as the first
```

Drift grows with distance travelled and is reported with no change in apparent confidence, which is exactly why a plan that trusts the estimate without checking it eventually drives into a wall.

```
  ROS 2 is M05's pub/sub, in a robot

  /scan        ──▶┌──────────────┐──▶ /map
  /odom        ──▶│ localisation │──▶ /pose
                  └──────────────┘
  /pose, /map  ──▶┌──────────────┐──▶ /cmd_vel ──▶ motors
                  │   planner    │
                  └──────────────┘
```

Each box is a node and each named channel is a topic, so a robot is a set of small programs publishing and subscribing exactly as in M05, with the motors subscribing at the end.

### Where it lives

M05 is the structural parent: a topic that publishes late is a stale sensor reading, which is chapter 13's delay problem destabilising a loop. M10 matters more, because a robot is an agent acting on a physical system, so what it may do has a blast radius in metres and kilograms rather than rows deleted — and the bound must hold mechanically: a torque limit, a bumper, a speed cap in the motor controller, a physical stop that works when the software has crashed.

Access is free. Robot simulators run in a browser at no cost, with a differential-drive robot, a laser scanner, a map and a navigation stack. Add wheel slip and watch the estimate diverge from the truth — the most valuable thing a simulator shows you, because on real hardware you cannot see the truth to compare against.

### Words to own

| Term | Plain meaning | Example |
|---|---|---|
| Degrees of freedom | How many independent ways it can move | A simple arm with six joints |
| Forward kinematics | Joint angles in, position out | Always one answer |
| Inverse kinematics | Desired position in, joint angles out | Many answers, or none |
| Singularity | A pose where a direction of motion is lost | An arm stretched straight |
| Odometry | Position estimated from wheel or joint motion | Counting wheel turns |
| Drift | Accumulated estimation error | A metre and a half by the corridor |
| Pose | Position plus orientation | Where and which way round |
| Localisation | Working out where you are in a known map | Matching a scan to the map |
| SLAM | Mapping and localising at the same time | First run through a new building |
| Loop closure | Recognising a known place and correcting the path | Seeing the pillar again |
| Sensor fusion | Combining several imperfect sensors | Wheels plus laser plus inertial |
| Occupancy grid | A map of cells marked free, occupied or unknown | What the planner reads |
| Costmap | A map weighted by how undesirable each cell is | Keeps clearance from walls |
| ROS 2 node / topic | A small program; a named channel between them | A planner publishing velocity |
| Sim-to-real gap | Working in simulation and failing in the world | The main robotics trap |
| Teleoperation | A human driving remotely | The usual honest fallback |

### Tradeoffs and gotchas

**Every sensor lies, differently.** Wheels lie when they slip, laser scanners lie on glass, cameras lie in low light and glare, inertial units drift. The design question is never whether a sensor is accurate but what its failure looks like and which other sensor catches it.

**Inverse kinematics is not unique.** Many joint configurations reach the same point, some through a wall or a person. A plan assuming one solution has skipped the part where you choose.

**The sim-to-real gap is mostly friction, latency and sensor noise.** A policy that works perfectly in a simulator frequently fails on hardware, and the standard mitigation is randomising the simulation rather than perfecting it.

**Embodied AI demos are cherry-picked.** A video of a learned policy folding laundry represents many attempts in one setting. The honest state of the field is rapid progress on manipulation research and no general-purpose household robot, and the bottleneck is data: there is no internet-scale corpus of robot bodies doing physical tasks.

**Liability is different from software.** A robot that injures someone is not an outage.

### Verify it

**Red flag: perfect sensors assumed.** The diagnostic test of this chapter. Ask what each sensor's error is, what its failure looks like, which other sensor catches it, and how drift gets corrected. A plan that never mentions noise, slip or drift is a plan for a simulator with the noise turned off, and it fails on its tenth run rather than its first.

**Red flag: no defined behaviour when localisation is lost.** The one that matters for safety. Ask directly: does it stop, rotate in place to re-scan, call a human, or keep driving on a stale estimate? There is only one wrong answer and it is the fourth.

**Red flag: a single inverse-kinematics solution presented as the answer.** Ask how many solutions exist and how the chosen one is screened for collisions and singularities.

**Red flag: results only from simulation.** Ask what has been demonstrated on hardware, over how many trials, with what failure rate. One video is not a success rate.

**Red flag: actuator authority with no mechanical stop.** If every safeguard is software, there is no safeguard.

## Walkthrough
1. Open a free robot simulator with a wheeled robot, a laser scanner and a navigation stack. Load the default world and drive the robot manually once around it.
2. Turn on the display that shows both the true position and the estimated position. Note that both are visible to you and only one is visible to the robot.
3. Send the robot on a long route and watch the two markers. Record the distance travelled at the point where the gap first becomes visible.
4. Introduce wheel slip, extra odometry noise, or a slippery surface if your simulator allows it. Repeat the route and record how much faster the gap grew.
5. Drive the robot back through an area it has already mapped and watch what happens to the estimate at the moment it recognises the place. Write down what that correction is called.
6. Inspect the list of topics the system is publishing. Write down four of them and what each one carries, in the vocabulary of M05's publishers and subscribers.
7. Block or disable the laser scan topic while the robot is moving. Record exactly what the robot does, and whether it stops, continues or reports anything.
8. Move an obstacle into a corridor the stored map shows as clear. Send the robot through and record whether it noticed, and how.
9. Write down the behaviour you believe the robot *should* have in each of three cases: localisation lost, scanner failed, obstacle where the map says clear. Compare with what you observed.
10. Finish with four lines: where drift came from, what corrected it, what the robot did when a sensor vanished, and the one mechanical protection you would add before letting this run near people.

## Talk to AI
```prompt
CONTEXT:      I am a student founder. I do not write code and I have no
              robot; I use a free robot simulator. I am reviewing a plan,
              not building it. The plan: a wheeled robot carries trays
              between a canteen counter and a hostel corridor, through a
              hall where furniture is rearranged and people walk.

GOAL:         A review of the plan that assumes every sensor is imperfect
              and states what happens in each failure.

REQUIREMENTS: 1. List the sensors such a robot would use. For each: what it
                 measures, its typical error, what its failure looks like
                 from the outside, and which other sensor would catch that
                 failure
              2. Explain where drift comes from, how loop closure corrects
                 it, and how the system can tell it is lost rather than
                 merely slightly wrong
              3. Define required behaviour for five failures: localisation
                 lost, scanner failed, obstacle where the map says clear,
                 a person standing in the path, and the map being a day out
                 of date
              4. Describe the software as nodes and topics, naming what
                 each topic carries, and say which topic going stale is
                 most dangerous
              5. State which safety protections must be mechanical and
                 still work if the software crashes mid-motion
              6. Tell me honestly what in this plan would work in a
                 simulator and fail on real hardware, and why

CONSTRAINTS:  No code. No formulas and no kinematics mathematics. Free
              simulators only; assume I cannot buy a robot. Plain English
              for a second-year student reading in a second language.

NON-GOALS:    Do not recommend a specific robot or vendor. Do not design a
              learned policy. Do not promise a timeline for embodied AI.

ACCEPTANCE:   - Every sensor row names a failure mode and a cross-check
              - Your localisation-lost behaviour is an explicit action, not
                "handle the error"
              - At least one safety protection in your answer is mechanical
              - You name at least two specific things that would work in
                simulation and fail in the hall

ASK:          Give me the sensor table first, then the five failure
              behaviours. Do not design the happy path at all until those
              two are done.
```

## Mission

**Objective.** Produce a sense-decide-act review of the part of your own product that acts on the real world — naming every input's failure mode and the defined behaviour when each one fails — and a written statement of the limit that holds when your software is wrong.

If your startup has any physical or real-world-acting component — a delivery robot, a drone, a kiosk, a vending unit, a camera, a device that switches something — review that. If your product is purely software, review the agent or automation in it that acts without a human checking each action: a bot that sends messages, an automation that moves money, an agent with write access to a customer's data. The loop is the same shape — it senses imperfect inputs, decides, and acts — and so are the failures.

**Steps.**
1. **Frame** — draw your loop in three boxes: what it senses, what it decides, what it acts on. One sentence each.
2. **Inputs** — list every input the decision depends on. For each: what it measures, how wrong it can be, what a *silent* failure looks like, and whether anything else would catch it.
3. **Find the drift** — name the input whose errors accumulate rather than cancel, the equivalent of odometry. Write how it would be corrected and how the system notices it has gone too far.
4. **Failure behaviours** — for each input, write the required behaviour when it fails: stop, degrade, re-sense, hand to a human. "Handle the error" is not an answer.
5. **Lost state** — write the single most important one explicitly: what happens when the system no longer knows the state of the world it is acting on.
6. **Brief** — use the Talk to AI prompt, adapted to your loop. Record its sensor table and its five failure behaviours.
7. **Verify in simulation** — run a free robot simulator, deliberately disable one sensor mid-run, and record what the robot actually did. Then write how that same failure shape appears in your own loop.
8. **Document** — write the decision-log entry: the limit that holds when your software is wrong, who can stop the system, and the one failure behaviour you have not yet defined.

**Upload.**
- Your three-box loop diagram, in text or a sketch → post caption or attached file
- Your input table with error, silent-failure and cross-check columns
- Your five failure behaviours, each as an explicit action
- A screenshot or recording from the robot simulator showing what happened when you disabled a sensor
- The AI conversation including its sensor table → share link in `ai_chat_url`
- Your decision-log entry, 150 words maximum
- Reflection, 150 words max → the input you had been silently trusting, and one thing the AI got wrong

**Self-check.** Tick every box before you submit.
- [ ] My three boxes name what is sensed, what is decided and what is acted on
- [ ] Every input has a "what a silent failure looks like" line
- [ ] I named the input whose errors accumulate rather than cancel
- [ ] Every failure behaviour is an explicit action, not "handle the error"
- [ ] I defined what happens when the system no longer knows the state of the world
- [ ] I actually disabled a sensor in a simulator and recorded the result
- [ ] I named a limit that holds even if my software is completely wrong
- [ ] I named who can stop the system and how long that takes
- [ ] I admitted at least one failure behaviour I have not yet defined
- [ ] My reflection names an input I had been trusting without checking

**Stretch.** Take the most impressive embodied AI demonstration video you can find, and write 150 words on what the video does not show: how many attempts, in how many environments, with what failure rate, and what would break first in a canteen hall.

## Stuck? Ask AI
```prompt
Explain to me, as if I have never seen a computer, why counting your steps with your eyes shut goes wrong slowly rather than suddenly, and why opening your eyes for one second fixes all of it at once. Use no formulas, keep it under 200 words, and end with the one question I should ask about any machine that thinks it knows where it is.
```

```prompt
I do not understand why asking "what joint angles put the hand here" can have many answers or none. Explain it with an everyday example using my own arm, no mathematics, and then ask me which of the possible answers a robot should refuse and why. Wait for my answer.
```

```prompt
Give me five robot or automation failures described only by what an observer saw - one drifting into a wall, one stopping dead in a doorway, one confidently reporting a wrong position, one working in simulation and failing in a room, one injuring something. Ask me which layer failed in each and what behaviour should have been defined. Do not reveal your answers until I have done all five.
```

## Quiz

### Q1 · L2
What is odometry drift?
- (a) A sensor failing suddenly and reporting nothing
- (b) Accumulated position error from estimating motion, which grows and does not self-correct ✅
- (c) The robot's map becoming out of date
- (d) The delay between a scan and a decision
> Each small error is added to the previous estimate, so the gap grows with distance travelled. Nothing in the odometry itself ever brings it back.

### Q2 · L3
What does loop closure do?
- (a) Ends a navigation task cleanly
- (b) Recognises an already-mapped place and corrects the accumulated path ✅
- (c) Closes a control loop by adding feedback
- (d) Shuts down the robot when it is lost
> Seeing a known landmark again lets the system repair the whole drifted trajectory at once, which is the single most valuable correction in a mapping run.

### Q3 · L3
A robot's wheels slip for half a second on a spilt plate of food. What is the most dangerous property of the resulting error?
- (a) It stops the robot immediately
- (b) It is reported with the same apparent confidence as a correct estimate ✅
- (c) It only affects orientation, not position
- (d) It disappears on the next scan automatically
> A silent failure is the hard case. The estimate is simply wrong, with no error flag, and every later measurement is added to the wrong starting point.

### Q4 · L4
A robotics plan says nothing about what happens when localisation is lost. Why is that the most serious omission?
- (a) Because mapping cannot start without a known position
- (b) Because the only wrong answer is to keep driving on a stale estimate, and that is the default ✅
- (c) Because inverse kinematics requires a valid pose
- (d) Because the planner will crash without a pose
> Stopping, re-scanning or calling a human are all acceptable. Continuing to act on an estimate the system has no reason to trust is how a machine drives into a person.

### Q5 · L5
An AI proposes a tray-carrying robot with a learned policy, demonstrated only in simulation. What is the most useful question to ask?
- (a) Which simulator was used
- (b) How many trials on real hardware, in how many environments, with what failure rate ✅
- (c) How many degrees of freedom the robot has
- (d) Whether the policy uses a camera or a laser scanner
> The sim-to-real gap is the field's main trap, and it is mostly friction, latency and sensor noise. A success rate on hardware is the only evidence that answers it.

## Go deeper
- [ROS 2 documentation: concepts, nodes and topics](https://docs.ros.org/en/jazzy/index.html)
- [Webots: free open-source robot simulator user guide](https://cyberbotics.com/doc/guide/index)
