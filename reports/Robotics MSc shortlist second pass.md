# Robotics MSc Shortlist: Second Pass

Oct 10, 2026 · @Lucas

The first report holds up, but its 3pi+ anchor kept out the two options I'd now put first: adaptive flight on a small quadrotor, and BRL's DOTS swarm. Reading what its session couldn't moved two candidates up and confirmed the rest. Corrections from two later reviews are folded in. This is analysis built on [the first report](https://github.com/emryszf/robotics), not proof.

## Revised ranking

Two first-tier candidates, with C7 leading instead if learning objectives are your priority. The 3pi+ week stays as a bounded pilot and fallback. Ranked by evidence that the problem is real and the method fits; access and cost are shown, not used to rank. These are conditional floors: a failed integration or an underpowered comparison is not a result.

| Tier | Candidate | Platform and spend | Depends on | Why here |
| --- | --- | --- | --- | --- |
| 1 | **N1:** adaptive flight under imperfect sensing | A borrowed BRL Crazyflie with a Flow deck; or an M5StampFly, £54 | Drone access; ground truth from the BRL flying arena or a webcam rig; a validated classical position loop | Learned and hybrid adaptation have beaten classical adaptive control on small quadrotors (DATT). How onboard estimation shifts that comparison on a small quadrotor is untested. Main risk: the flight and estimation pipeline eats the dissertation. |
| 1 | **N2:** BRL's fault-robust swarm controller on DOTS | DOTS, 20 robots | Hauert lab access, code and support, all unconfirmed | Possibly the best-supported MSc, if the lab confirms support. Open as far as the abstracts show; the manuscript's methods weren't readable. |
| 1, if learning objectives come first | C7: verifier-limited best-of-k, with TailRL as a comparator | SO-101, £165–200, or a BRL arm | Its simulation kill test, run before buying | Closest fit to TailRL and MaxRL. The CMU thesis presents the TailRL–judge coupling as a prospective programme: that supports testing it, not the effect. Longest dependency chain. |
| Pilot | C1 + C2: the shared 3pi+ week | Owned 3pi+; under £50 | Nothing | Cheap, discriminating measurements and a fallback. On its own it may stay too narrow without a sharper question. |
| 2 | C3 / C5: SO-101 error anatomy, failure detection | SO-101 | The arm | NeuralActuator, validated on an SO-101, narrows both. |
| 2 | N3: RF emitter search on Leo Rovers at Fenswood (was C4) | Hunt lab Leo Rovers | Access | Real simultaneity and less multipath outdoors; still no learning. |
| 3 | C6: Robotarium team-size scaling | Remote; £0 | Account approval | Superseded by N2 if DOTS access comes through. |

Wildcards: on-robot RL (Open Ant), tactile RL on BRL arms, and the report's magnetic bumper. Next: ask Sabine Hauert or Simon Jones about DOTS, and Arthur Richards or Tom Richardson about arena training and a Crazyflie, while the 3pi+ week runs. Prefer whichever platform already has a working experimental pipeline. Payload changes and faults make this resilience research; it would only become cyber with a defined adversary on a specified channel.

## Coverage

I could open the arXiv, Springer, BRL and vendor pages the first session couldn't, and read the CMU thesis from the copy on your computer. Most page reads went through a summariser, so treat figures as abstract-plus depth until you read the PDFs.

| Source | Read depth | Used for |
| --- | --- | --- |
| The Tail and the Judge (CMU-RI-TR-26-90, July 2026) | §7.2 and §8.2–8.4 in full, from your local copy | C7 novelty |
| [RAPTOR](https://arxiv.org/abs/2509.11481) | Platforms, baselines, state estimation and limitations, via a summarised page | N1 |
| [DATT](https://arxiv.org/abs/2310.09053) (CoRL 2023) | Baselines, results and limitations, via a summarised page | N1 |
| [JaxRobotarium](https://arxiv.org/abs/2505.06771) | Sim-to-real section, via a summarised page | C6, N2 |
| [RiskRL](https://arxiv.org/abs/2411.02788) and [When to Localize?](https://arxiv.org/abs/2411.08281) | Abstracts | C2 |
| [Morris, Pelham & Hunt](https://arxiv.org/abs/2510.14018) | Abstract | C4, N3 |
| [Alboraei, Lee & Hauert](https://link.springer.com/chapter/10.1007/978-3-032-26123-6_15) (ANTS 2026) | Abstract | N2 |
| [DOTS testbed](https://arxiv.org/abs/2203.13809) (2022) | Abstract | N2 |
| [BRL flying-arena tutorial](https://starlinguas.github.io/starling_controller_templates/brl_flight/) | Summarised page | N1 |
| [M5StampFly at The Pi Hut](https://thepihut.com/products/m5stamp-fly-with-m5stamps3) | Price and specs, 10 Oct 2026 | N1 |
| [Amano & Azumi](https://arxiv.org/abs/2609.28969), [The Open Ant](https://arxiv.org/abs/2607.18488), [Tac2Real](https://arxiv.org/abs/2603.28475) | Abstracts | C1, N4, N5 |
| NeuralActuator (arXiv 2607.11734) | Abstract, plus a news summary ([ModelWire](https://themodelwire.com/article/neural-actuator-model-cuts-sim-to-real-error-on-low-cost-robots-01KXF8F9BKB07AYMNEP2TBK3YA)) | C3, C5 |

Not done: full methods reads of ASAP, AnyCar, Neural-Fly, CLAPS and ACI, and BRL's rules on MSc access to the arena, DOTS or lab arms.

**What moved.** The frontier that fits an MSc moved towards small, open policies and platforms that work on real hardware (RAPTOR, the Open Ant) and towards modelling cheap hardware honestly (NeuralActuator, simulator tuning). It did not move towards bigger models on a 12 GB GPU, which is why the first report's frontier cuts were right.

## Verdict on the first report

It is careful and mostly right. Its one structural flaw came from my prompt: the 3pi+ and £200 pinned the shortlist to a plant its own analysis calls benign.

What holds:

- It is honest about access and claims nothing beyond what it read.
- Every candidate has pre-registered kill criteria and a measured-boundary floor.
- The shared 3pi+ week is clever: slip drives both top candidates, so one week decides both.
- The physical reasoning is sound: the regulated 8 V motor rail, friction unobservable before slip, and 100 ms of tracker latency costing about 15 cm at 1.5 m/s.
- The "tempting but fails" list is its most useful part. It stops you reopening dead ideas.

Where it bends:

- Six of its seven candidates and its wildcard run on the 3pi+ or one £200 arm, yet it concludes RL "will keep losing" on the 3pi+. The anchor and the conclusion pull against each other.
- BRL kit appears only as "what to ask for". The Vicon flying arena, DOTS, Leo Rovers and the UR5 tactile hands are in its notes, but never a candidate's main platform.
- Little of it is frontier. The top two are good engineering science; the frontier items were rightly cut for a 12 GB GPU, but nothing replaced them.
- Filter 5 (nearest work read) is unmet everywhere, through no fault of the writer.

Keep its top two as a bounded pilot and fallback. Don't treat its ranking as final now that the 3pi+ is optional and BRL kit can be requested.

## What gap-filling changed

Two candidates moved up and none died. The biggest blocked read, the CMU thesis, strengthens Candidate 7 rather than sinking it.

| Candidate | What I checked | What it showed | Effect |
| --- | --- | --- | --- |
| C7: best-of-k | The Tail and the Judge, §7.2 and §8.2–8.4 | It calls the TailRL–judge coupling "synthesis motivating a research program, not an established result". Its future work proposes tracking the selection gap against group size, with and without tail-likelihood training. Every experiment is simulated (DeepMind Control, HumanoidBench, Atari). Its TailRL chapters have "open results slots": numbers not yet claimed. | Up one. Your TailRL arm becomes a physical test of a named, open conjecture. The hypothesis isn't yours; the test would be. |
| C6: team-size scaling | JaxRobotarium's sim-to-real section | Its scenarios use 3–4 robots (from the training configs), and it has no analysis against team size. The largest gaps came where "inflection points" amplify small differences. The authors conclude single-robot accuracy "does not necessarily imply accurate multi-robot simulations". | Up. Its main kill condition is cleared. Better run on DOTS (N2). |
| C2: calibrate or decide | RiskRL and When to Localize? | Both report numerical experiments only, and neither arXiv page names a venue. | No change. Real re-localisation actions stay open. |
| C4: RF patrol | Morris, Pelham & Hunt | Simulation only; the abstract names no propagation model. | No change. Running outdoors on Leo Rovers lowers the multipath risk (N3). |
| C1: physical limits | Amano & Azumi (Sept 2026) | Bayesian-optimisation tuning of a differential-drive simulator, then zero-shot DreamerV3 transfer to a WHILL wheelchair. | No change, but "identify the simulator" isn't new. The contribution has to be the fidelity ablation. |
| C3 / C5: SO-101 | NeuralActuator (July 2026; abstract) | Validated on an SO-101, among other arms. Predicts a torque surrogate, external force and a motor-condition score without force sensors; the abstract gives no numbers. | Down, more for C5: its motor-condition score overlaps C5's telemetry detector, so C5 needs it as a baseline. |

## N1. Adaptive flight under imperfect sensing

**Question.** How does state-estimation quality change the relative robustness of recurrent learned and classical quadrotor control under one specified change in dynamics?

**Why now.** [RAPTOR](https://arxiv.org/abs/2509.11481) (*Science Robotics* 2026, per the first report) flew ten real quadrotors zero-shot, from 31.9 g to about 2.4 kg, with one 2,084-parameter recurrent policy. They include a 38 g M5StampFly and a Crazyflie, and the code and weights are open. It already flew outdoors on its largest drone, and its supplement adds an accelerometer-based fix for velocity delay, so "RAPTOR without motion capture" is not new in itself. It reports no classical baselines, no motor damage, and nothing on how a small quadrotor's onboard estimation shifts it against strong classical control. Its inputs include absolute position and velocity, so onboard flight means position from integrated optical flow, which drifts.

**Nearest work and what's open.** [DATT](https://arxiv.org/abs/2310.09053) (CoRL 2023) is a hybrid: a learned tracking policy fed by an L1 disturbance estimator. On a 40 g Crazyflie 2.1 it beat L1 adaptive control and L1-MPC, which crashed on infeasible trajectories, though only on planar paths. Neural-Fly set the template for comparing learned adaptation with L1 and INDI. So the contrast is learned, hybrid and classical adaptation, not a simple learned-versus-classical divide. Open, as far as this pass found: how estimation quality changes that contrast on a small quadrotor.

**Baseline and why it might fail.** Two classical baselines, so learning can't win against a weak one. Fixed gains tuned once on the unmodified drone are the fair test, because adaptation's claim is needing no retuning; gains retuned for the perturbation give an upper bound. On a Crazyflie, the cascaded PID covers position down to rate. Mellinger covers position and attitude but relies on the platform mass, so a payload change tests exactly that assumption. INDI is documented as an attitude controller, so confirm and validate the position loop it runs under before it counts. The StampFly's manual reportedly has no position hold, so there you would build and validate the cascaded loop first. Classical control was developed to stabilise exactly these systems, so expect it to be strong.

**First-week test and kill.** Validate the classical position loop on the unmodified drone first. Then run a 2 × 2 in a netted space: motion capture against onboard sensing, crossed with RAPTOR's released policy against classical control. Use one perturbation fixed in advance, a payload at two masses; propeller swaps and delay come later, if at all. Score tracking error against ground truth and against the onboard estimate, so estimator drift isn't counted as control error. Set a practically meaningful margin and the number of independent flights before testing; equivalence within that margin is a floor result, not a kill. Switch platform or candidate if either controller can't fly reliably with motion capture within the pilot's time box: a failed integration is not a result.

| Need | Status | Label |
| --- | --- | --- |
| M5StampFly | £54 at The Pi Hut; sold out on 10 Oct 2026 | Sourced |
| Its sensors | ESP32-S3; BMI270 IMU; PMW3901 optical flow; two VL53L3 range sensors | Sourced |
| On-board policy cost | Under 10% of compute on RAPTOR's smallest platforms | Sourced |
| Ground truth | BRL flying arena: Vicon, 10 × 5 × 4 m flying volume, nets. Or a webcam ArUco rig, used for evaluation only | Arena sourced; webcam accuracy estimated |
| Retraining with more delay randomisation | Skip teacher pre-training with the released teacher checkpoints; use the released policy as-is for the core study | Sourced (README) |

**Better platform.** A borrowed BRL Crazyflie with a Flow deck gives stock PID, Mellinger and INDI controllers, onboard flow-based velocity, and a RAPTOR reference implementation. Ask before buying a StampFly, and set a stock alert if you still need one.

**Crash budget.** Spare propellers, motors and batteries, safe LiPo handling, and flights only inside nets.

**Headline figure.** Tracking error at each payload mass for RAPTOR and the classical controllers, in two panels: motion capture and onboard sensing.

**Best realistic and floor.** Best: explain a failure mechanism, such as delayed or drifting velocity breaking the policy's adaptation, and show a targeted mitigation. That needs controlled degradation: add delay or noise to the motion-capture feed in steps matching what you measure onboard. Floor: a controlled comparison showing when estimation errors undermine adaptation, judged against the pre-set margin.

**BRL fit (topical only).** Arthur Richards (control, aerial), Tom Richardson (Bristol Flight Lab) and Shane Windsor (bio-inspired flight). The arena flies Coex Clover drones through the Starling workflow, with testing arranged through a supervisor.

**Skills.** Flight control (PID, INDI), optical-flow state estimation, RL deployment on a microcontroller and flight testing. All of it reads directly as GNC.

**What you'd be learning.** With the released weights, this studies deployment and in-context adaptation of an existing learned controller; nothing is trained on the robot. If training RL matters to you, add one arm: re-run RAPTOR's final training stage from its released teacher checkpoints, using the delay you measure on your drone.

**Risks.** Cheap drones crash and break, arena access needs training, StampFly stock is patchy, and onboard position drifts.

**Filters.** 1 pass. 2 pass, once the classical position loop is validated. 3 pass: onboard estimation is the real deployment condition. 4 pass: adapting to changed dynamics under imperfect sensing is the question, and DATT shows hybrid adaptation can win. 5 partial: RAPTOR and DATT read via summaries. 6 pass, given a drone and ground truth. 7 pass. 8 pass.

## N2. BRL's fault-robust swarm controller on DOTS, across team sizes

**Question.** Does the Hauert lab's fault-robust transport policy keep its edge over the non-learning baseline on real DOTS robots, and does its sim-to-real gap change with team size, holding density and the fraction of faulty robots fixed?

**Why now.** [Alboraei, Lee & Hauert](https://link.springer.com/chapter/10.1007/978-3-032-26123-6_15) (ANTS 2026) trained a shared-parameter recurrent MARL policy that maps local state to mitigation actions. It beat a non-learning baseline and scaled to more faults than it saw in training. The abstract names no hardware, and DOTS appears only in its references. Separately, [JaxRobotarium](https://arxiv.org/abs/2505.06771) concluded that single-robot accuracy doesn't guarantee accurate team simulation, and never varied team size.

**Nearest work and what's open.** JaxRobotarium and MARBLER on the Robotarium, and IRIDIA's e-puck work, which the first report says found modular controllers cross the reality gap better than neural ones. Open: BRL's own policy on real robots, and the gap from 4 up to 20 robots.

**Baseline and why it might fail.** The paper's non-learning baseline, plus per-robot noise randomisation (JaxRobotarium's fix). Faults and congestion both scale with neighbour count, which per-robot noise cannot model.

**First-week test and kill.** Confirm DOTS access and support, get the simulator and policy from the lab, and ask whether the policy uses onboard sensing or motion-capture state. Run baseline and policy with 4 and 12 robots at the same density. Kill if access can't support enough independent runs by summer 2027 to resolve a meaningful margin. Agreement between simulation and hardware within that margin is a positive result, not a kill.

**Hardware and compute.** [DOTS](https://arxiv.org/abs/2203.13809) has 20 robots, motion capture and private 5G, and was described in 2022 as an "open access testbed" (sourced). Its 2026 status, MSc access and onboard sensors are unknown (ask). The simulator is the lab's (ask).

**Headline figure.** Sim-to-real gap against team size and fault count, learned policy against baseline.

**Best realistic and floor.** Best: a density-driven gap and a simulator fix that removes it. Floor: "BRL's policy transfers within X up to N robots", which the host lab wants to know either way.

**BRL fit.** Sabine Hauert (strong), Simon Jones (designed DOTS; research staff), Edmund Hunt (moderate).

**Why it beats the Robotarium version.** It would be hands-on, local and inside a lab's own agenda, if the lab confirms it can support you.

**Filters.** 1 pass. 2 pass. 3 pass: motion capture is fine for coordination and congestion claims; it only rules out claims about onboard perception. 4 pass: the learned policy is the object of study. 5 partial: abstracts only. 6 conditional on access. 7 pass. 8 pass.

## Other new avenues

Three more routes open once the 3pi+ is optional. Each depends on access or a build I couldn't price.

- **N3. RF emitter search on Leo Rovers at Fenswood.** The report's Candidate 4, run simultaneously on several Hunt-lab Leo Rovers at the Fenswood field site. Open ground should give a cleaner signal-strength trend with distance than indoors, so its main kill risk shrinks. It is still not a learning project unless an adaptive information-gain planner beats the optimised fixed routes. Fit: Edmund Hunt, the paper's senior author.
- **N4. On-robot RL from scratch.** [The Open Ant](https://arxiv.org/abs/2607.18488) (July 2026) is an open-source physical version of Gymnasium's Ant. SARSA(λ) and SAC learned to walk on the hardware in about an hour, and simulation policies transferred. A question: when the robot changes (a worn leg, added mass), does learning on the robot recover faster than redoing sim-to-real? It is the closest fit to your RL interest here, but build cost and parts are unknown, so treat it as a watch item.
- **N5. Tactile RL on BRL arms.** The report's magnetic bumper is the 3pi+ version. The stronger version uses Nathan Lepora's TacTip sensors and Tactile Gym on a lab arm. GPU tactile simulators for online RL are new ([Tac2Real](https://arxiv.org/abs/2603.28475), March 2026). A question: which parts of the tactile model must be right for zero-shot transfer? It depends entirely on joining that group's setup.

## Tempting but fails: additions

These extend the report's list for the new platforms. Filter numbers match your prompt.

| Direction | Fails | Why |
| --- | --- | --- |
| Training a new quadrotor foundation policy | 4, 6 | Method-first, and RAPTOR already exists in the open. |
| Agile flips or racing on a StampFly or Crazyflie | 6, 3 | DATT notes a thrust-to-weight ratio under 2 rules out agile manoeuvres; the gap would be the airframe's. |
| TailRL for flight control | 4 | No rare-success structure, and physical rollouts can't be repeated from one context. |
| Learned communication over DOTS's 5G | 3, unless grounded | Injecting loss is fair only if the loss model matches a measured or specified operating condition. |
| LLM-planned swarm behaviour on DOTS | 1, 4 | Method-first; the robots become a backdrop. |
| A new tactile simulator | 4, 6 | A tool rather than a question, and beyond MSc scale. |

## Where your interests may still steer you

Drones are the new pull, so N1 needs the scepticism the 3pi+ ideas got.

- N1 hits RL, hardware and defence at once. Trust the kill test over enthusiasm: classical control may simply win, and a drone that crashes every second flight eats weeks.
- N1 with released weights is deployment research, not RL training. Decide whether that satisfies you, or add the retraining arm.
- The thesis read supports testing the TailRL–judge coupling; it doesn't establish it. And choosing among generated candidates before acting is not the same as running several real attempts and keeping the best.
- Overhead tracking is fine for multi-robot coordination studies. It only rules out claims about onboard perception or genuine decentralisation.
- Supervision beats topic. Pick the platform that already has a working pipeline and someone who knows its limits.

## Reading list for the new candidates

Read these in full before agreeing a topic; most were read here only through summaries. For the 3pi+ week, the report's own list still applies.

| # | Read | Why |
| --- | --- | --- |
| 1 | [RAPTOR](https://arxiv.org/abs/2509.11481): limitations and deployment sections; code at [rl-tools/raptor](https://github.com/rl-tools/raptor) | N1's starting point and its stated gaps |
| 2 | [DATT](https://arxiv.org/abs/2310.09053), CoRL 2023 | How to compare learned adaptation with L1 on a Crazyflie |
| 3 | Neural-Fly, *Science Robotics* 2022 | The L1 and INDI comparison design |
| 4 | The StampFly firmware source and Bitcraze's controller documentation | The classical baselines you would fly |
| 5 | [Alboraei, Lee & Hauert](https://link.springer.com/chapter/10.1007/978-3-032-26123-6_15), ANTS 2026 | The policy N2 would transfer |
| 6 | [DOTS testbed](https://arxiv.org/abs/2203.13809) | Sensors, localisation and access |
| 7 | [JaxRobotarium](https://arxiv.org/abs/2505.06771), sim-to-real section | What is known about multi-robot gaps |
| 8 | The Tail and the Judge, §6 and §8.2–8.4 (your local copy) | Candidate 7's conjecture and diagnostic |

## Sources

Pages opened on 10 October 2026, plus the thesis read from your computer.

- [First report and strand notes](https://github.com/emryszf/robotics) (emryszf/robotics)
- The Tail and the Judge, S. Ramasubramanian, CMU-RI-TR-26-90 (your local copy)
- [RAPTOR: A Foundation Policy for Quadrotor Control](https://arxiv.org/abs/2509.11481)
- [DATT: Deep Adaptive Trajectory Tracking for Quadrotor Control](https://arxiv.org/abs/2310.09053)
- [JaxRobotarium](https://arxiv.org/abs/2505.06771)
- [When to Localize? A Risk-Constrained RL Approach](https://arxiv.org/abs/2411.02788)
- [When to Localize? A POMDP Approach](https://arxiv.org/abs/2411.08281)
- [Spatially Intelligent Patrol Routes for Concealed Emitter Localization](https://arxiv.org/abs/2510.14018) by Robot Swarms
- [Multi-Agent RL of a Fault-Robust Controller in an Intralogistics Robot Swarm](https://link.springer.com/chapter/10.1007/978-3-032-26123-6_15)
- [DOTS: An Open Testbed for Industrial Swarm Robotic Solutions](https://arxiv.org/abs/2203.13809)
- [Starling: flying your controllers in the BRL Flight Arena](https://starlinguas.github.io/starling_controller_templates/brl_flight/)
- [M5Stamp Fly with M5StampS3, The Pi Hut](https://thepihut.com/products/m5stamp-fly-with-m5stamps3)
- [Sim-to-Real Aware End-to-End Learning Environment for Micromobility](https://arxiv.org/abs/2609.28969)
- [The Open Ant](https://arxiv.org/abs/2607.18488)
- [Tac2Real](https://arxiv.org/abs/2603.28475)
- [ModelWire on NeuralActuator](https://themodelwire.com/article/neural-actuator-model-cuts-sim-to-real-error-on-low-cost-robots-01KXF8F9BKB07AYMNEP2TBK3YA)
- [RAPTOR README](https://raw.githubusercontent.com/rl-tools/raptor/master/README.md)
- [NeuralActuator](https://arxiv.org/abs/2607.11734)
- [Bitcraze: Crazyflie controllers](https://www.bitcraze.io/documentation/repository/crazyflie-firmware/master/functional-areas/sensor-to-control/controllers/)
