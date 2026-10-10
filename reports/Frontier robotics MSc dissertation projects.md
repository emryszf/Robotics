# Measure the robot before you choose

**Research date: Saturday 10 October 2026.** Seven research strands produced nineteen candidate questions and more than sixty rejected directions. The two that survive scrutiny best both run on the Pololu 3pi+ you already own, and the same near-free week of measurements settles both. The strongest is a **physical-limits study**. You find where the 3pi+ stops behaving like the kinematic model that let a ten-line controller beat your learners. You then build the smallest simulator that predicts what happens there, which MuJoCo's 2026 DC-motor, delay and system-identification releases make cheap. Only if week one shows real slip do you go on to ask whether learned adaptation beats classical traction control at the friction limit. The second is **calibrate or decide**: when the robot must choose when to re-localise, or how fast to approach a hazard, does the gain come from calibrating its own uncertainty or from a richer, possibly learned, decision rule? Five further candidates follow in order of evidence. First is an error-anatomy study of a cheap SO-101 arm, which would consume the whole £200. Then a physical test of Edmund Hunt's simulated RF-emitter patrol routes, on hardware you own but with no learning in it, and runtime failure detection on the SO-101. Last are team-size scaling of the sim-to-real gap on the remote Robotarium, and a verifier-limited best-of-k study, the one place MaxRL/TailRL-style objectives legitimately belong in robotics. The wildcard is a printed magnetic tactile bumper in the lineage of Nathan Lepora's tactile group. Two caveats govern everything. The session's network policy blocked most primary literature, so **no candidate yet passes filter 5**, and "no overlap found" below always means an incomplete search, never established novelty. And your own record says classical methods usually win on this hardware, so each project is designed so that a measured boundary, not a learned victory, is the minimum deliverable.

## 1. Research date and coverage: verified to 10 October 2026, mostly at abstract depth

Treat every claim below as verified only as far as the notes verified it on **10 October 2026**. Seven strands fed this report:

| Strand | Scope |
|---|---|
| Adaptive control | Adaptive control and sim-to-real on the 3pi+ |
| State estimation | State estimation and active localisation |
| Multi-robot | Multi-robot and swarm learning |
| Imitation learning | Imitation learning, LeRobot and low-cost hardware |
| VLAs and RL | Small VLAs, RL post-training and verifier-based objectives |
| Safe learning | Safe learning, wildcards and physics-level security |
| BRL | BRL staff, facilities and the MSc timeline |

That evidence has hard limits. The session's network policy **blocked arXiv, OpenReview, the publishers (IEEE Xplore, Science, Springer, ScienceDirect, Frontiers), Hugging Face, `*.github.io` project pages, Pololu, ST and other vendor sites, and every University of Bristol, UWE and BRL website**. The page-fetching tool failed on DNS for every host except GitHub. The web-search budget, 200 calls per turn shared across all agents, ran out in every strand, and one strand managed only about ten searches. The researchers therefore read whatever GitHub could serve. Most papers were seen only as abstracts, search snippets or READMEs; a handful of full texts and datasheets came from copies committed to GitHub.

| Reading depth | What it covers |
|---|---|
| **Read in full, primary** | Pololu's 3pi+ 32U4 Arduino library source. Bristol's Robotics Science & Systems (RSS) teaching repos: labsheets, example CSVs and the 2026 `StampC3_Template`. MuJoCo's changelog, `dcmotor` reference, and sysid and MJX docs. LeRobot docs and SO-follower code. The Robotarium simulator source. ESP-IDF's ESP-NOW docs. READMEs for RLtools, RAPTOR, AnyCar, ASAP (partial), Neural-Fly, RiskRL, FAIL-Detect, JaxRobotarium, MARBLER, eFlesh, AnySkin, Tactile Gym and others. |
| **Full or near-full texts from GitHub copies** | HIL-SERL paper (pp. 1–6, 9–14, 19–21). TailRL main text (pp. 1–15). MaxRL preprint (pp. 1–8, 11–13). SRT paper (pp. 1–2). MARBLER supplementary (5 pp.). Harish Ravichandar's July 2026 research statement. The ST LSM6DS33 and LIS3MDL datasheet tables. A saved copy of Pololu's 3pi+ 2040 user's guide. |
| **Abstract, snippet or README only** | Nearly every other paper, including RiskRL and the When-to-Localize POMDP, CLAPS, Conformal Decision Theory, Tearle et al., ABS, AnyCar, ASAP, RMA, Neural-Fly, RAPTOR, JaxRobotarium, the SSRR 2025 emitter paper, ANTS 2026, the June 2026 SO-101 benchmark, DREAM-Chunk, DP-ACT, CoVer-VLA, RoboMonkey and eFlesh. Also all Pololu product and guide pages, and every BRL staff and programme page. |
| **Not accessed** | Pololu's schematic and product pages. Gibbs & Candès's adaptive conformal inference (ACI). Classical adaptive-control papers for differential-drive robots. The CMU thesis "The Tail and the Judge". The limitations sections of AnyCar, ASAP and RMA. Live UK vendor prices. BRL rules on MSc equipment access, the allocation process, past dissertation titles and the submission date. |

Three consequences follow for your confidence. First, **filter 5 is at best "partial" for every candidate**: the strongest overlapping papers are named, but their methods and limitations sections were mostly not read. Second, success rates, speed-ups and accuracies taken from abstracts are provisional until you read the paper. Third, every statement about BRL staff rests on search-engine snippets of institutional pages. The one exception is Paul O'Dowd's GitHub, which was read directly. Nothing documents which BRL equipment MSc students may use, and nothing here claims anyone is available to supervise. Where the notes disagree, the stronger source is as follows.

| Point | What the notes say | Stronger source |
|---|---|---|
| **IMU part on the 3pi+ 32U4** | All strands: ST LSM6DS33 plus LIS3MDL. One warns the library dates from 2022, so newer boards might differ. The LSM6DSO belongs to the 3pi+ 2040. | **Pololu's library source.** It detects only the LSM6DS33 (WHO_AM_I 0x69) and the LIS3MDL ([IMU.cpp](https://github.com/pololu/pololu-3pi-plus-32u4-arduino-library/blob/master/src/Pololu3piPlus32U4IMU.cpp)), and the 2040 guide names the LSM6DSO for the 2040 ([saved guide](https://github.com/FREECPP/Bachelor-Thesis/blob/main/Skripte/00_Weitergabe_Micromouse/doku/Pololu_docs.htm)). Read WHO_AM_I on your board in week one. |
| **Top speed** | 1.5 m/s from a search extract of the 32U4 guide. Another strand found 1.5 m/s only for the 3pi+ 2040. | **The 32U4-specific extract** ([0J83](https://www.pololu.com/docs/0J83/all)). It is consistent with the shared chassis and motors, and with an extrapolation of Bristol's speed data to about 1.57 m/s (estimated). |
| **Battery as a perturbation** | The safe-learning strand proposes battery level as a braking factor. The adaptive-control strand says the motors run from a regulated 8 V rail. | **The 32U4 product extract plus Bristol's labsheet** ("VM: is the regulated voltage supplied to the motors at 8v") ([product 3737](https://www.pololu.com/product/3737); [SL0](https://github.com/paulodowd/SEMTM0042_43/blob/main/Labsheets/Supp/SL0_MeasuringBatteryVoltage.ipynb)). The 2040's battery-sag warning concerns a different board. Treat battery as a check, not a design variable. |
| **Is the RSS platform a 3pi+ with an ESP32-C3?** | The BRL strand calls it unverified because its README access was denied. The multi-robot strand had not confirmed that O'Dowd runs the unit. | **The 2026 template source, read directly by the adaptive-control strand.** The StampC3 is I2C master to the 3pi+ at 0x08 and runs at 100 Hz ([Robot.cpp](https://raw.githubusercontent.com/paulodowd/RoboticsScienceAndSystems/main/StampC3_Template/Robot.cpp); [Controller.h](https://raw.githubusercontent.com/paulodowd/RoboticsScienceAndSystems/main/StampC3_Template/Controller.h)). The unit catalogue names O'Dowd as unit director of SEMTM0043, on a page with a possibly provisional "2033/34" label ([UPC](https://upc.bristol.ac.uk/unit-programme-catalogue/UnitDetails.jsa?ayrCode=33/34&unitCode=SEMTM0043)). |
| **BRL job titles** | Stale pages call Hauert a Reader, Hunt a Lecturer and O'Dowd a Lecturer in Computer Science, place Kunze at Oxford, and list Studley as co-director. | **Current individual profiles, cross-checked with the October 2026 UoB news release.** Richards and Kunze co-direct BRL; Hauert is Professor of Swarm Engineering; Hunt and O'Dowd are Senior Lecturers ([UoB news](https://www.bristol.ac.uk/news/2026/october/new-98-million-robotics-hub-to-benefit-south-west-businesses.html)). |
| **Hunt lab's robots** | Not found by the multi-robot strand. | **The BRL strand: Leo Rovers, earlier Kilobots** ([Hunt Lab](https://www.huntlab.uk/?p=10)). |

Before you agree a topic, read the full texts below. These are what would move filter 5 from "partial" towards "pass".

| Candidate | Read in full | Access on 10 Oct 2026 |
|---|---|---|
| 1. Physical limits | ASAP (RSS 2025) limitations. AnyCar (arXiv 2409.15783; venue unverified, so treat as PREPRINT) limitations. Neural-Fly (*Science Robotics* 2022) baselines. RAPTOR (*Science Robotics* 2026) baselines and limitations. A classical adaptive-control reference for differential drive. | READMEs and abstracts only. The classical references were never searched. |
| 2. Calibrate or decide | RiskRL (arXiv 2411.02788 v2) and When to Localize? (SSRR 2024). CLAPS (PREPRINT 2512.10294). Gibbs & Candès ACI (NeurIPS 2021). AI-IMU (*IEEE T-IV* 2020). Conformal Decision Theory (ICRA 2024). Tearle et al. (*RA-L* 2021). | Abstracts, plus RiskRL's and AI-IMU's code READMEs. ACI not retrieved. |
| 3. SO-101 error anatomy | DREAM-Chunk (PREPRINT 2606.18589). SO-101 benchmark (PREPRINT 2606.08881). DP-ACT (*Scientific Reports* 2026). The 2026 STS3215 backlash test-stand paper. "Revisiting open-loop" (PREPRINT 2608.15938). | Abstracts and third-party summaries only. |
| 4. RF patrol | Morris, Pelham & Hunt (SSRR 2025), especially its propagation model. A classical RSSI-localisation reference. | Abstract only. Classical literature not searched. |
| 5. Failure detection | FAIL-Detect (RSS 2025) real-robot results. Sentinel/STAC and FIPER. | README only. Sentinel and FIPER not found. The lerobot_far docs were read in full. |
| 6. Robotarium scaling | JaxRobotarium (CoRL 2025) sim-to-real section. The MARBLER main paper. Ligot & Birattari (2020). | Abstract only. The MARBLER supplementary was read in full. |
| 7. Best-of-k | "The Tail and the Judge" (CMU-RI-TR-26-90). CoVer-VLA (PREPRINT 2602.12281). Stroebl et al. (PREPRINT 2411.17501). DMPO. | The thesis host was blocked. The rest were read as summaries or READMEs. |
| Wildcard: tactile bumper | Tactile Gym 2.0 (*RA-L* 2022) `object_push` sim-to-real. Lloyd & Lepora on tactile pushing. eFlesh (PREPRINT 2506.09994). | READMEs and abstracts. Mobile tactile pushing was not searched. |

## 2. What moved: tools for measuring robots, not proof that learning wins

What has genuinely moved since late 2024 is the infrastructure for honest measurement at MSc scale, not evidence that learned controllers beat classical ones on cheap hardware. **MuJoCo's 2026 releases** cover what your kinematic simulator left out ([changelog](https://raw.githubusercontent.com/google-deepmind/mujoco/main/doc/changelog.rst)). Version 3.5.0 (12 Feb 2026) brought official MuJoCo Warp, a system-identification toolbox, and actuator and sensor delays. Version 3.7.0 (14 Apr 2026) added a `dcmotor` actuator with back-EMF, cogging, inductance and LuGre friction, set from datasheet stall torque and no-load speed, and 3.12.0 (20 Aug 2026) gave it voltage input. **RAPTOR** (*Science Robotics*, 2026) showed a 2,084-parameter recurrent policy adapting in context to ten real quadrotors from 32 g to 2.4 kg, with microcontroller deployment code ([README](https://raw.githubusercontent.com/rl-tools/raptor/master/README.md)). On your ESP32-C3, RLtools measured a [64, 64] network at 6.6–8.7 ms per inference ([RLtools](https://raw.githubusercontent.com/rl-tools/rl-tools/master/README.md)). Multi-robot learning gained turnaround but not capability. JaxRobotarium (CoRL 2025) trains in minutes and reports more than 200 real Robotarium deployments ([lab post](https://github.com/GT-STAR-Lab/gt-star-lab.github.io/blob/main/papers/_posts/2025-09-28-jain-jaxrobotarium.md)), yet every 2024–2026 deployment found still wraps learned decisions in classical controllers and barrier certificates ([IDEA, PREPRINT](https://arxiv.org/abs/2606.26575)). Cheap-arm learning got honest evaluation. LeRobot reached v0.6.1 (3 Aug 2026) with SO-101 support, DAgger rollouts and reward models ([releases](https://github.com/huggingface/lerobot/releases)). The first independent SO-101 benchmark (PREPRINT, June 2026) measured **π0.5 at 56.25%, ACT at 33.75% and SmolVLA at 32.5%**, with execution instability the main failure source ([2606.08881](https://arxiv.org/abs/2606.08881)). Verifier-guided best-of-N became crowded, and verifier error began to bite: CoVer-VLA (PREPRINT, Feb 2026) reported RoboMonkey's verifier making π0 worse than no verification at all ([CoVer-VLA](https://arxiv.org/abs/2602.12281)). MaxRL (ICML 2026) and TailRL (PREPRINT, Sept 2026) gave pass@k objectives a principled form, but they contain no robot experiments, and their gains appear only when initial success is about 1% or rarer ([TailRL](https://github.com/Zanette-Labs/TailRL-website/blob/main/static/TailRL.pdf)). "When to localise" was posed as a chance-constrained POMDP and as risk-constrained RL, both only in simulation ([RiskRL](https://arxiv.org/pdf/2411.02788)). Conformal SE(2) prediction sets reached a small real differential-drive robot (CLAPS, PREPRINT, Dec 2025) ([CLAPS](https://arxiv.org/html/2512.10294v1)). The net effect for you: identifying, calibrating and evaluating a cheap robot has never been cheaper. Meanwhile the method-novelty niches are either crowded or beyond a 12 GB GPU: action-chunk scheduling, VLA reinforcement learning, LLM-designed swarms and world models.

## 3. Seven candidates ranked by evidence, with the two 3pi+ studies leading

Where strands asked the same research question, I merged their candidates. **Candidate 1** merges four. From the adaptive-control strand it takes the simulator-fidelity ladder and the traction-limit adaptation study. It adds the safe-learning strand's RAPTOR-style tiny adaptive controller and the imitation strand's optical-flow slip instrument. All four ask where the 3pi+ leaves the kinematic regime, and whether anything learned helps there. **Candidate 2** merges the state-estimation strand's "calibration versus decision in when to re-localise" with the safe-learning strand's calibrated braking filter. Both ask whether calibrated uncertainty or a richer decision rule drives a consequential choice. **Candidate 3** absorbs the imitation strand's scripted-versus-learned boundary map as one chapter. The ranking weighs four things: evidence that the gap is real, a stated physical reason the baseline might fail, feasibility with kit you own, and the depth of reading behind each claim. It deliberately ignores fit to your interests.

| Rank | Candidate | Draws on (notes) | Platform and spend | Evidence the gap is real | Main risk |
|---|---|---|---|---|---|
| 1 | Where the 3pi+ leaves the kinematic regime | adaptive_control (C1, C2); safe_learning (B); imitation (D) | Owned 3pi+; about £0–£50 (estimated) | Your own documented failure; Bristol's measured dead-band and surface data; a first-principles traction estimate | Effects may be small outside aggressive manoeuvres |
| 2 | Calibrate or decide: re-localisation and approach speed | state_estimation (A); safe_learning (A) | Owned 3pi+; under £30 of arena materials (estimated) | Datasheet gyro bias and drift; nearest work is simulation-only | Dead reckoning may already be good enough |
| 3 | Error anatomy of learned manipulation on an SO-101 | imitation (A, C) | Follower arm, £165–£200 (estimated) | Measured servo backlash; an independent benchmark that blames execution | Uses the whole budget; kit stock; a parallel GitHub project |
| 4 | Do Hunt's simulated emitter-patrol routes survive real RF? | multi_robot (C2) | Owned 3pi+, Stamp C3, Pico 2 W; £0–£50 (estimated) | Simulation-only predecessor with quantified route effects | Indoor RSSI may be unusable; not a learning project |
| 5 | Can a cheap arm tell it is failing? | imitation (B) | Same arm as rank 3 | Recovery gains shown only under staged failures | Telemetry may already catch most failures |
| 6 | Does the sim-to-real gap grow with team size? | multi_robot (C1) | Remote Robotarium; £0 | More than 200 real deployments with uneven gaps | Hands-off; access; overlap with the JaxRobotarium group |
| 7 | Simulate more, gain less? Verifier-limited best-of-k | vla_rl (A) | Same arm as rank 3 | LLM theory; one VLA report; DMPO on hardware | Strongest overlap (a CMU thesis) unread; heaviest build |

### One week of 3pi+ measurements decides both leading candidates

Candidates 1 and 2 rest on the same physical unknowns: stopping distance, slip, step response and dead-reckoning error. Run one pre-registered measurement week with kit you own, ideally in November–December 2026, before the topic is agreed in early 2027. Write the kill thresholds down before you collect any data.

| Day | Task |
|---|---|
| 0 | **Set up.** Read WHO_AM_I on your board: 0x69 means LSM6DS33. Set up logging, and build overhead-webcam ArUco ground truth. Measure that ground truth against a ruler grid, because its accuracy was never sourced. The 16-bit encoder counters wrap after roughly 9.2 m of travel with 32 mm wheels (estimated), so log count deltas ([Encoders.h](https://github.com/pololu/pololu-3pi-plus-32u4-arduino-library/blob/master/src/Pololu3piPlus32U4Encoders.h)). |
| 1–2 | **Wheels-off tests.** Apply PWM steps at 25/50/75/100%, plus reversals and chirps, logged at about 1 kHz on the 32U4 over USB serial. Fit a first-order-plus-dead-band-plus-delay model, and the `dcmotor` datasheet model. |
| 2–3 | **Surface tests.** On three or four real lab surfaces, do full-PWM launches and stops: 10 stops × 4 speeds per surface. Compare IMU x-acceleration with encoder-derived acceleration. Run constant-radius turns at rising speed, comparing gyro z-rate with encoder yaw rate. Repeat a subset at two battery levels to confirm the 8 V regulation. |
| 3–4 | **Odometry calibration.** Run UMBmark clockwise and counter-clockwise squares. Estimate at-rest gyro bias. Estimate the gyro scale factor from N spins over a radial tape line. |
| 5 | **Course runs and replay.** Run a pre-registered course open-loop about 10 times per surface. Replay your old hand-coded controller and one policy trained in your kinematic simulator, both in simulation and on the robot. |

How to read the results:

| Week-one outcome | What it means |
|---|---|
| No detectable slip, and dead-reckoning error well inside the task tolerance | Both 3pi+ candidates die. Pivot cheaply to rank 3 or 4, because nothing has been bought. |
| Slip at full PWM | Candidate 1 lives. The same slip probably also widens stopping-distance spread and odometry error, which keeps Candidate 2 alive. |
| Dead-reckoning error above tolerance, or a consistency statistic (NEES) that differs across surfaces | Candidate 2 lives. |

The outcomes are correlated because slip drives all of them. That is why one week decides the whole 3pi+ branch.

### Candidate 1. Where the 3pi+ leaves the kinematic regime: a fidelity ladder, then adaptation at the traction limit

**Research question.** Which physical effects must a simulator contain for controllers designed or trained in it to transfer zero-shot to the 3pi+, as a function of how aggressive the manoeuvre is? The candidate effects are acceleration limits, DC-motor back-EMF, surface-dependent dead-band, loop delay, encoder quantisation and wheel slip. A conditional second question follows only if week one shows slip with real headroom: does a small, history-based learned adaptive controller beat classical traction control at the friction limit?

**Why now.** Four developments open this up.

| Development | Why it matters here |
|---|---|
| **MuJoCo 2026 releases** | System identification and delays arrived in 3.5.0 (12 Feb 2026). A datasheet-parameterised `dcmotor` arrived in 3.7.0 (14 Apr 2026), and voltage input in 3.12.0 (20 Aug 2026) ([changelog](https://raw.githubusercontent.com/google-deepmind/mujoco/main/doc/changelog.rst)). |
| **ASAP (RSS 2025)** | Made learned "delta action" alignment a standard comparator ([RSS page](https://roboticsconference.org/program/papers/66/)). |
| **RAPTOR (*Science Robotics* 2026)** | Supplies a recipe for a tiny in-context adaptive policy ([README](https://raw.githubusercontent.com/rl-tools/raptor/master/README.md)). |
| **Your own result** | A simulator with no acceleration limits let a ten-line controller beat every learner. That documents the gap rather than manufacturing it. |

**Nearest work and what remains open.**

| Work | What it does | Gap or limitation, and read depth |
|---|---|---|
| ASAP | Learns corrective "delta actions" from the mismatch between real and simulated humanoid trajectories, using motion capture | Secondary summaries say two Unitree G1s broke and the full 23-DoF delta model gave way to a 4-DoF ankle model; stated future work is fewer real samples. Summary level only; primary limitations unread ([README](https://raw.githubusercontent.com/LeCAR-Lab/ASAP/main/README.md)) |
| AnyCar | Pairs a transformer dynamics model with MPPI (sampling-based model-predictive control) on 1/10–1/16 cars with an onboard PC | Baselines are learned specialists, not classical adaptive control. Limitations section not accessed ([arXiv](https://arxiv.org/abs/2409.15783)) |
| Neural-Fly | Quadrotor learned adaptation | Alone among these, it benchmarks against L1 adaptive control, incremental nonlinear dynamic inversion (INDI) and a no-learning adaptive ablation. README level only ([README](https://raw.githubusercontent.com/aerorobotics/neural-fly/main/README.md)) |
| A-RMA (biped extension of RMA) | Adapts the legged-robot adaptation method RMA to bipeds | Notes that the extrinsics estimator "can be imperfect" ([A-RMA](https://hybrid-robotics.berkeley.edu/publications/IROS2022_AdaptingRMA_Biped.pdf)) |

Two things remain open, and for both no overlap was found in an incomplete search. One is a systematic fidelity ablation on a microcontroller-scale differential-drive robot. The other is a learned-versus-classical-adaptive comparison on one. A strand recalled that Tan et al. (RSS 2018) and Hwangbo et al. (2019) found actuator and latency modelling essential. That is unverified prior knowledge, so check it.

**Strongest simple baseline, and why it might fail.**

| Question | Baseline |
|---|---|
| Fidelity | A calibrated kinematic model with a first-order lag plus speed and acceleration limits fitted from step tests: your simulator plus about three parameters |
| Adaptation | PI wheel loops on the 32U4 at 500 Hz or faster; a gyro heading loop; a per-surface acceleration limit; an encoder-versus-IMU slip detector; and a recursive-least-squares (RLS) friction update |

They might fail for four physical reasons:

| Mechanism | Evidence |
|---|---|
| Back-EMF makes available acceleration fall with speed, so a constant limit is wrong | Motor physics |
| Full-PWM launches look traction-limited | About 36 m/s² is motor-limited against 3–10 m/s² friction-limited (both estimated from first principles) |
| The dead-band is surface-dependent | Bristol measured a dead-band of PWM −12 to +14, and a carpet-to-table speed ratio of 0.52 at PWM 20 against 0.93–0.97 at PWM 48–116 (computed from their measured data) ([L1 labsheet](https://github.com/paulodowd/SEMTM0042_43/blob/main/Labsheets/Core/L1_Motors_SEMTM0042_43.ipynb); [surfaces.csv](https://raw.githubusercontent.com/paulodowd/SEMTM0042_43/refs/heads/main/ExampleData/surfaces.csv)) |
| Wheel speed is closed across I2C at 100 Hz | The 2026 RSS middleware has no wheel-speed setpoint register, adding roughly 10–15 ms of effective delay (estimated) ([i2c_datatypes.h](https://raw.githubusercontent.com/paulodowd/RoboticsScienceAndSystems/main/StampC3_Template/i2c_datatypes.h)) |

The deeper lesson is that friction is unobservable from encoders or the IMU until slip begins. Neither RLS nor a learned history encoder can therefore anticipate a low-friction patch. **Learning's only structural edge is faster detection of slip onset**, and that has to be measured, not assumed.

**First-week test and kill criterion.** Use the shared week above.

| Kill this… | …if |
|---|---|
| The fidelity question, or shrink it to a chapter | All three hold: the first-order-plus-delay model predicts held-out wheel speed to within about 1 count per 10 ms; no slip is detectable at full PWM or in maximum-speed turns on any practical surface; and real trajectories match the calibrated kinematic-plus-lag simulator within camera error |
| The adaptation half | No slip appears at full PWM, or a single conservative acceleration limit performs within run-to-run noise of per-surface tuned limits |

**Hardware and compute.**

| Need | Status | Label |
|---|---|---|
| 3pi+ 32U4 Standard Edition, Stamp C3, USB webcam | Owned | per your brief |
| 358.3 encoder counts per wheel revolution; 1.5 m/s top speed; regulated 8 V motor rail | Platform facts | sourced ([User's Guide](https://www.pololu.com/docs/0J83/all); [product 3737](https://www.pololu.com/product/3737)) |
| Printed ArUco markers, surfaces, low-friction sheet, payload weights | To buy | well under £50, estimated |
| Optional PAA5100JE/PMW3901 downward optical-flow breakout as a slip instrument | Price unverified; a Python library supports both over SPI | £20–£30 estimated; library sourced ([pmw3901-python](https://github.com/pimoroni/pmw3901-python)) |
| System identification on the desktop CPU | Batched `mujoco.rollout` across threads | sourced ([sysid README](https://raw.githubusercontent.com/google-deepmind/mujoco/main/python/mujoco/sysid/README.md)) |
| RTX 3060 for JAX or MJWarp training | MJWarp needs an NVIDIA GPU. The differentiable MJX-JAX path lacks `dcmotor`, but a stateless motor can be written as an affine actuator. | GPU need sourced ([mjx.rst](https://raw.githubusercontent.com/google-deepmind/mujoco/main/doc/mjx.rst)); affine workaround inferred |
| On-board policy on the ESP32-C3 | A [64, 64] MLP takes 6.6–8.7 ms per inference, so use 16–32 units or fixed point. A RAPTOR-sized network may reach about 300–450 Hz. | timing sourced ([RLtools](https://raw.githubusercontent.com/rl-tools/rl-tools/master/README.md)); sizes and rates estimated |

**Headline figure.** A "fidelity ladder" plots sim-to-real error against the model component added, with manoeuvre aggressiveness as the series. Error means RMS position error and the gap in completion time. It covers a hand-written controller, an MPC and an RL policy, each trained in each simulator. If the adaptation half survives, a second panel shows completion time and stopping error across surface × payload for learned, classical-adaptive and fixed-conservative controllers, alongside slip-detection latency.

**Best realistic and floor outcomes.** The best realistic outcome is an aggressiveness threshold above which back-EMF, slip and delay modelling become necessary, with RL trained in the identified simulator transferring where kinematic-simulator RL failed. For the adaptation half, the realistic best is a tie at steady state, with learning ahead only on abrupt surface transitions (estimated). The floor is a validated, parameterised 3pi+ simulator with uncertainty bounds, plus a measured "first-order plus delay suffices up to X" boundary. That floor is useful to every RSS cohort.

**BRL supervision fit** (topical overlap only; no claim about availability).

| Person | Fit | Evidence |
|---|---|---|
| Paul O'Dowd | Strongest | MSc Programme Director and RSS unit director. His GitHub holds the 2026 StampC3 template, an ArUco tracker, a 2-D differential-drive simulator and the Fixed16 fixed-point library ([github.com/paulodowd](https://github.com/paulodowd)) |
| Arthur Richards | Good | Professor of Robotics and Control with a robust-MPC background, matching the identification-plus-MPC baseline ([UoB profile](https://www.bristol.ac.uk/people/person/Arthur-Richards-d4aa20a8-75fa-4b1a-8400-b2387ed04fe5/)) |
| Kunze, Hunt, Hauert, Winfield | None visible | — |

Looking wider, no 2023–2026 BRL publication on learning-based control was found. The nearest people are Efi Psomopoulou, who works on control for physical interaction, and Nathan Lepora's group, who work on sim-to-real RL for touch. UWE advertised a Lecturer in Robot Learning from June to July 2026; the appointee was not found ([jobs.ac.uk](https://www.jobs.ac.uk/job/DRU598/)).

**Skills it builds.** System identification and DC-motor modelling, AVR and ESP32 firmware, high-rate logging, sensor fusion, MuJoCo and JAX simulation, and sim-to-real RL with domain randomisation. Also traction control and experimental design, which GNC, embedded and autonomy employers all read directly.

**BRL equipment and what to ask for.**

| Ask | Purpose |
|---|---|
| Sessions on O'Dowd's ArUco tracking server, which sends poses at 10 Hz ([tracker README](https://raw.githubusercontent.com/paulodowd/ProcessingTrackingVisualiser/main/README.md)), or two or three 2-hour motion-capture sessions with a marker kit for a 97 mm robot | Ground truth to validate the webcam |
| A one-week loan of 5–10 RSS 3pi+ robots | Measure unit-to-unit spread |
| Permission to reflash the 32U4 middleware | Run wheel-speed PI on the 32U4 |

None of these is essential: a webcam with printed markers is a realistic fallback.

| Filter | Verdict | Reason |
|---|---|---|
| 1 Robotics first | Pass | The robot's physics is the subject. |
| 2 Baseline named | Pass | Named, with physical failure mechanisms: back-EMF, traction, dead-band, delay. |
| 3 Gap is real | Pass, conditional on week one | Your own failure documents it. The risk is that it is small: surface changes alter mid-speed gain by only 3–7%. |
| 4 Method justified | Pass for identification; weak for learned adaptation | Friction is unobservable before slip. |
| 5 Nearest work read | Partial | ASAP, AnyCar, Neural-Fly and RAPTOR were read at README or abstract level. |
| 6 Feasible | Pass | Owned hardware; on-board policies must be tiny. |
| 7 First-week test | Pass | Kill criteria fixed in advance. |
| 8 Useful if learning loses | Pass | The validated simulator and the fidelity boundary are the product. |

### Candidate 2. Calibrate or decide: when to re-localise, and how fast to approach a hazard

**Research question.** A 3pi+ runs pre-registered waypoint tasks in an arena with sparse tape lines and walls. A tuned uncertainty-threshold rule decides when it should re-localise and, as a second decision, how fast it should approach a hazard edge. Compared with a hindsight oracle, how much of that rule's shortfall comes from miscalibrated uncertainty, and how much from the rule itself? Calibration options are adaptive covariance, or conformal and adaptive-conformal updates at each re-localisation event. Richer rules are a one-step expected-cost rule, chance-constrained POMCP, or recurrent soft actor-critic (SAC) in the style of RiskRL.

**Why now.**

| Source | What it shows |
|---|---|
| *When to Localize? A POMDP Approach* (SSRR 2024) and RiskRL (arXiv 2411.02788, v2 2025; venue unverified) | Together they defined the chance-constrained re-localisation problem, but only in simulation ([POMDP paper](https://arxiv.org/abs/2411.08281); [RiskRL](https://arxiv.org/pdf/2411.02788)) |
| CLAPS (PREPRINT, Dec 2025) | Put conformal SE(2) prediction sets on a real small differential-drive robot, an MBot ([CLAPS](https://arxiv.org/html/2512.10294v1)) |
| Two 2026 preprints | Argue that learned or filter covariances lack coverage guarantees or are overconfident ([May 2026](https://arxiv.org/pdf/2605.00233); [Sept 2026](https://arxiv.org/pdf/2609.14271)) |
| Conformal Decision Theory (ICRA 2024) | Calibrates a speed "knob" online for throughput under a risk bound ([CDT](https://arxiv.org/pdf/2310.05921)) |
| Oh et al. (Oct 2025) | Argue that least-restrictive safety filtering preserves RL optimality, which points to calibrated rather than worst-case margins ([Oh et al.](https://arxiv.org/pdf/2510.18082v2)) |

**Nearest work and what remains open.** All of the following were read at abstract or README level only.

| Work | What it does | Limitation |
|---|---|---|
| RiskRL | A particle filter with twin recurrent SAC under chance constraints; reports at least 26% higher success in unseen environments (v2) | Its public code is Gymnasium grid worlds ([README](https://github.com/raaslab/when-to-localize-riskrl)) |
| The POMDP paper | Chance-constrained POMDP for when to re-localise | Lists "high compute time" and a need for "well-defined transition and observation models"; one version reports more than 20 minutes of inference for a 55-waypoint path |
| CLAPS | Calibrates action-outcome sets | Never triggers re-localisation |
| Tearle et al. (*RA-L* 2021) | Ran a predictive safety filter on a miniature RC car ([arXiv](https://arxiv.org/pdf/2102.11907)) | Not a re-localisation method |

What remains open:

| Open question | Why it matters |
|---|---|
| Real re-localisation actions that cost time and can fail | The simulation work assumes otherwise |
| One-dimensional partial fixes from a tape line | A line crossing fixes only one coordinate |
| Calibration of the belief that drives the decision | A miscalibrated belief can make a good rule look bad |
| Adaptive conformal inference under sparse, decision-dependent, partial labels | The robot only learns its true error when it chooses to re-localise |

No overlap was found for any of these, in an incomplete search.

**Strongest simple baseline, and why it might fail.**

| Decision | Baselines |
|---|---|
| Re-localisation | A trace(P) threshold tuned on training runs; periodic or distance-based re-localisation; a one-step expected-cost rule |
| Speed | The analytic braking rule v ≤ √(2·a_min·(d−m)) − a_min·τ, tuned on the worst surface, which already *is* the control barrier function for a near-double-integrator; and a per-surface margin looked up from line-sensor reflectance |

These baselines might fail for three reasons:

| Failure mode | Detail |
|---|---|
| The filter's σ is wrong exactly when it matters | During slip in acceleration and turns, at surface transitions, and under temperature-driven gyro bias. The LSM6DS33's zero-rate level drifts by ±0.05 dps/°C ([datasheet](https://github.com/dddrrreee/cs240lx-20spr/blob/master/labs/8-i2c-accel/docs/LSM6DS33.pdf)), so a 5 °C warm-up leaves about 0.25 °/s of bias: roughly 15° of heading per minute if uncompensated (estimated) |
| Detour cost varies with position | A fixed threshold ignores where the nearest line is |
| A missed line is a failure | If the error exceeds the line's capture region, re-localisation itself fails |

There is a teaching point here. Uncertainty grows roughly monotonically between resets, which gives the problem an optimal-stopping structure. In that structure, threshold rules are often near-optimal *if their input is calibrated*. So the likeliest result is "calibration matters more than the rule". That is publishable, and it is a strong classical-wins prior you should state up front. A free classical fix also exists: re-estimate gyro bias at every stop the encoders detect.

**First-week test and kill criterion.** Use the shared week, adding about 10 open-loop course runs per real lab surface. Measure final error against a task tolerance τ, taken from a stated task such as ±1–2 cm docking. Also compute the normalised estimation error squared (NEES), a standard filter-consistency check. Add about 240 braking stops: 10 stops × 4 speeds × 3 surfaces × 2 battery levels.

| Outcome | Action |
|---|---|
| **Kill:** the 95th-percentile calibrated dead-reckoning error over the whole course is below τ on every surface | Report the critical course length L\* and move on. Do not lengthen the course or tighten τ to rescue the idea; that would fail filter 3. |
| **Soft kill:** NEES is already consistent across surfaces | The calibration arm has little to fix |
| **Kill the braking arm:** stopping-distance spread is below the dead-reckoning error at the hazard, or below about 5 mm (estimated) | Calibrated margins cannot beat a fixed margin |
| **Kill the braking arm:** the robot cannot reach speeds where braking distance exceeds the line-sensor-to-wheel offset | The hazard can always be caught reactively |

Your earlier finding that dead reckoning "may suffice" is a strong prior towards the first kill.

**Hardware and compute.**

| Need | Status | Label |
|---|---|---|
| 3pi+, Stamp C3 telemetry firmware, webcam | Owned | per your brief |
| Tape, fiducials, arena boards, edge rig with a foam catch | A few pounds; under £30; £0–£20 | estimated |
| Webcam ground-truth accuracy | Sub-centimetre over about 1 m² with fiducials is plausible, but must be measured | estimated |
| Control-barrier-function quadratic program on the ESP32-C3 | 28.04 µs worst case on an embedded motor drive, which suggests it is cheap on the C3 | drive figure sourced ([arXiv 2507.07805](https://arxiv.org/pdf/2507.07805)); C3 transfer estimated (different CPU) |
| Simulator | A noise model fitted to your logs, not the kinematic simulator | — |
| RTX 3060 | Ample for an EKF, small networks and POMCP over a low-dimensional belief | estimated |

**Headline figure.** For re-localisation, a regret decomposition: "Of the regret (time plus failures) between a tuned threshold and the hindsight oracle, X% is removed by calibration and Y% by a better rule." The design is factorial: four estimators (a fixed-Q EKF, adaptive-Q, a learned covariance adapter, and ACI) crossed with three rules (threshold, expected cost, and POMCP or RL). Plot empirical coverage at 90% nominal by surface. Vary line density as an independent variable so that line placement cannot manufacture the gap. For braking: "At ≤1% edge violations, X% more mean speed than a worst-case margin and Y% more than a per-surface lookup."

**Best realistic and floor outcomes.** The best realistic outcome is a clean decomposition in which a calibrated threshold matches POMCP or RL within noise. That is an informative negative result for learned decision rules. Separately, expect a 15–30% speed gain on grippy surfaces (estimated). The floor is dead reckoning within tolerance for tabletop tasks. You would then report L\*(τ, surface), a characterisation of 3pi+ odometry calibration and coverage, and "a fixed margin costs under 5% of speed below v m/s".

**BRL supervision fit** (topical overlap only).

| Person | Fit | Notes |
|---|---|---|
| Paul O'Dowd | Strong | Owns the platform. His GitHub includes ArUco tracking and BLE/WiFi RSSI localisation tests. |
| Lars Kunze | Overlaps your "consequential autonomy" interest | Professor in Safety for Robotics and Autonomous Systems at UWE and BRL Co-Director. Works on safe autonomy, and on causal and explainable AI, including learnt reward profiles (ICRA 2025) ([UWE profile](https://people.uwe.ac.uk/Person/LarsKunze)). His Responsible Technology roles lean towards governance, so frame the project as estimation and decision-making, not assurance. |
| Arthur Richards | Fits the chance-constraint side | Robust MPC and guidance |
| Edmund Hunt, Sabine Hauert | Partial | Gaussian-belief-propagation estimation |
| Alan Winfield | Not a fit for what you want | His consequence engine predates 2023, and his 2024–2026 output is ethics, standards and accident investigation ([Robot Talk, Mar 2026](https://robohub.org/?p=219232)) |
| Kerstin Eder | Avoid as primary | Works on verification and validation, which sits close to assurance |

**Skills it builds.** EKF design, UMBmark and gyro calibration, and the NEES/NIS consistency tests. Conformal and adaptive-conformal methods, POMDPs and constrained RL, and control barrier functions. Embedded logging and pre-registered experimental design.

**BRL equipment and what to ask for.** Ask for 2–4 hours of motion capture, or O'Dowd's ArUco tracker, to validate the webcam. Ask for any 3pi+ odometry logs from past RSS cohorts, optionally a second 3pi+ to check unit-to-unit variation, and floor space of about 2 × 2 m for a taped arena (estimated).

| Filter | Verdict | Reason |
|---|---|---|
| 1 Robotics first | Pass | The robot's sensing and decisions are the subject. |
| 2 Baseline named | Conditional | Threshold, expected-cost and braking baselines are named. The failure mechanism must be shown in weeks 1–3. |
| 3 Gap is real | Conditional | Only if a pre-registered τ beats dead-reckoning accuracy on real surfaces. The camera is for evaluation only. |
| 4 Method justified | Pass | ACI is justified by sparse, delayed, policy-dependent labels, and POMCP by the chance constraint. RL is one arm, not the thesis. |
| 5 Nearest work read | Partial | RiskRL, the POMDP paper, CLAPS and CDT were read at abstract or README level only. |
| 6 Feasible | Pass | Owned hardware. |
| 7 First-week test | Pass | Kill criteria are fixed in advance. |
| 8 Useful if learning loses | Pass | The factorial decomposition is informative whichever arm wins. |

### Candidate 3. Error anatomy of learned manipulation on a £200 arm

**Research question.** On an SO-101, how much of an ACT policy's end-point error comes from three sources: load-dependent backlash and compliance that the joint encoders cannot see, servo tracking error that they can see, and policy variance? And as task clearance tightens, does shortening the open-loop execution horizon recover more success than classical compensation does?

**Why now.**

| Source | What it shows |
|---|---|
| Independent SO-101 benchmark (PREPRINT, June 2026) | Execution instability was the main failure source ([2606.08881](https://arxiv.org/abs/2606.08881)) |
| DP-ACT (*Scientific Reports*, Aug 2026) | Raised ACT from 62% to 82% on an SO100 with dynamic-programming correction, and blamed limited repeatability. Seen only as an abstract via a third-party summary ([summary](https://github.com/deepcharles/macs3050-journal/issues/82)) |
| A 2026 servo test stand | Measured STS3215 backlash at 0.62° unloaded and 1.30° loaded. Seen only as a search summary ([ScienceDirect](https://www.sciencedirect.com/science/article/pii/S2468067226000271)) |
| Chunking-stability theory (ICLR 2026) and "Revisiting open-loop" (PREPRINT, Aug 2026) | Make testable predictions about execution horizon ([2507.09061](https://arxiv.org/abs/2507.09061); [2608.15938](https://arxiv.org/abs/2608.15938)) |
| LeRobot v0.6.1 | Made the follower's P coefficient configurable |

**Nearest work and what remains open.**

| Work | Overlap | What it leaves out |
|---|---|---|
| DREAM-Chunk (PREPRINT, June 2026) | The strongest methods overlap. Selects action chunks robustly against "hardware execution errors" on the SO-101, reporting 10% → 65% on a perturbed insertion task ([arXiv](https://arxiv.org/abs/2606.18589)) | Uses external perturbations and does not attribute error |
| `so101-error-anatomy` GitHub project (Sept 2026) | Plans the same decomposition ([repo](https://github.com/tucasteixeira-debug/so101-error-anatomy)) | Has no data yet |
| BID | In simulation, open-loop chunks of 8 fall from 0.884 to 0.582 under noise while single-step execution holds 0.805 ([bid_diffusion](https://github.com/YuejiangLIU/bid_diffusion)) | Simulation only |

What remains open is the hardware analogue: external ground truth linked to both a policy-design variable and a classical compensator. No overlap was found in an incomplete search.

**Strongest simple baseline, and why it might fail.** Engineer the classical fixes properly. Approach targets from one side, add static dead-band compensation, and tune the servo gains. LeRobot ships the follower with **P = 16, I = 0, D = 32** ([config](https://github.com/huggingface/lerobot/blob/main/src/lerobot/robots/so_follower/config_so_follower.py)), so the first fix is to enable integral gain or add a host-side integral outer loop. Also build a scripted ArUco-plus-IK pick-and-place, which becomes the scripted-versus-learned chapter. These baselines might fail for three reasons. Backlash roughly doubles under load. Gravity torque changes sign across the workspace and on contact. And gear-stage play is invisible to the output-side encoder. To show this, plot residual error after compensation against the predicted change in gravity-torque sign.

**First-week test and kill criterion.** Once the arm is built, measure ArUco and webcam noise with the arm unpowered. Then use `lerobot-replay` to replay one trajectory 30 times to each of 5 poses, with and without a payload of about 100 g, logging Goal − Present. Finally, repeat with integral gain switched on.

| Outcome | Action |
|---|---|
| **Kill:** 3σ scatter is about 1 mm or less and insensitive to load direction, while ACT fails mainly at grasping or perception | Hardware error is not the bottleneck |
| **Escalate:** ArUco noise is at least as large as the backlash signal | Move to a dial gauge or motion capture |
| **Re-scope:** integral gain alone removes most of the error | Focus on the encoder-invisible component |

**Hardware and compute.**

| Need | Status | Label |
|---|---|---|
| DIY follower: servos | 6 × £19.20 = £115.20, on-demand stock ([RobotShop UK](https://uk.robotshop.com/products/feetech-12v-30kgcm-magnetic-encoding-servo-sts3215)) | computed from a sourced unit price |
| DIY follower: board, PSU, cable and clamp | About $33 in the BOM; no UK prices found | estimated |
| DIY follower: printed parts | £29.74, or free if BRL prints them | sourced |
| DIY follower: total | £165–£200 | estimated |
| Two-arm motor kit plus skeleton | £203.99 + £29.74 = £233.73; sold out when indexed ([Unmanned Tech](https://www.unmannedtechshop.co.uk/products/so-arm101-low-cost-ai-arm-servo-motor-kit-pro-for-lerobot-without-3d-printed-parts)) | computed from price snapshots |
| Phone teleoperation of the follower alone | Supported; demonstration quality against a leader arm is undocumented | sourced ([phone_teleop](https://github.com/huggingface/lerobot/blob/main/docs/source/phone_teleop.mdx)) |
| ACT training on the RTX 3060 | 2–6 GB VRAM | sourced ([hardware guide](https://github.com/huggingface/lerobot/blob/main/docs/source/hardware_guide.mdx)) |
| Diffusion policy training | 8–14 GB at batch 8, so needs a smaller batch | sourced (smaller batch estimated) |
| Recording 50 demonstrations | 1–2 h | estimated |
| Wrist camera | £10–£25 | estimated |

**Headline figure.** "On the SO-101, X% of ACT's end-point variance is mechanical. At ≤1 mm clearance, shortening the horizon from 100 to k steps recovers Y percentage points, against Z for outer-loop compensation."

**Best realistic and floor outcomes.** The best realistic outcome is an error decomposition with confidence intervals, plus a clearance crossover curve. The floor is an ISO 9283-style repeatability and backlash map of the SO-101, which every LeRobot user would find useful.

**BRL supervision fit.** None of the six named staff fits; Richards's control work is model-based and not about manipulation. The closest wider fit is Efi Psomopoulou, Senior Lecturer from 2025 in control for physical interaction and tactile grasp control ([UoB profile](https://www.bristol.ac.uk/people/person/Efi-Psomopoulou-88afab93-e2fc-4a7a-be90-ccfcd945e1ad/)). Others with related work:

| Person | Area |
|---|---|
| Nathan Lepora | Tactile dexterity |
| Antonia Tzemanaki | Dexterous Manipulation and Wearable Robotics |
| Guanqun Cao (UWE) | Tactile sensing, VLAs |

No 2023–2026 BRL imitation-learning publication was found. Dandan Zhang, who worked on robot learning, has moved to Imperial ([Imperial](https://profiles.imperial.ac.uk/d.zhang17)).

**Skills it builds.** Metrology and ISO 9283 statistics, servo identification and control, kinematics and IK, imitation-learning training and evaluation, and statistically powered experiment design. At n = 20 trials, the 95% interval on 50% success is about ±22 percentage points (computed), so plan 50 or more trials per condition.

**BRL equipment and what to ask for.**

| Ask | Why |
|---|---|
| Any SO-100/101 leader or follower arm, or a Koch or OMX arm | Saves the budget or supplies a leader |
| 3D-printer access with PLA+ and TPU 95A | Printed parts |
| Motion capture, or a dial indicator and height gauge | Ground truth beyond ArUco |
| A GPU of 24 GB or more | Only if you need SmolVLA at a normal batch size |

| Filter | Verdict | Reason |
|---|---|---|
| 1 Robotics first | Pass | Arm physics is the subject. |
| 2 Baseline named | Conditional pass | The integral-gain and outer-loop fixes must be engineered properly. |
| 3 Gap is real | Pass | Only with a pre-registered clearance sweep and no injected perturbations. |
| 4 Method justified | Pass | A visual closed loop is justified because part of the error is unobservable to the joint sensors. |
| 5 Nearest work read | Partial | DREAM-Chunk was read at abstract level, and a parallel GitHub project exists. |
| 6 Feasible | Pass, tight | Uses the whole £200; the kit was out of stock when indexed. |
| 7 First-week test | Pass | Runs once the arm is built. |
| 8 Useful if learning loses | Pass | The error budget stands on its own. |

### Candidate 4. Do Hunt's simulated emitter-patrol routes survive real radio?

**Research question.** In simulation, differential evolution ranked patrol-route shapes for a four-robot team searching for a concealed emitter. Do those shapes keep their ranking and accuracy when the received signal strength (RSSI) is real indoor 2.4 GHz, measured from a moving low-cost robot?

**Why now.** Morris, Pelham & Hunt (SSRR 2025; arXiv, 15 Oct 2025) reported these results from simulation only ([arXiv](https://arxiv.org/abs/2510.14018v1)):

| Simulated result | Value |
|---|---|
| Most efficient route shape per metre | Triangular |
| Detection rate | **98.75%** with directional antennas, **80.25%** with omnidirectional |
| Mean localisation error | 1.01–1.30 m directional; 1.67–1.90 m omnidirectional |
| Where omnidirectional failures occurred | Clustered at the map periphery |

The Hunt lab's 2026 work keeps its small-team focus.

**Nearest work and what remains open.** The SSRR paper was read at abstract level only, so its propagation model is unknown, and the classical RSSI-localisation literature was not searched. What remains open is physical validation; no overlap was found in an incomplete search.

**Strongest simple baseline, and why it might fail.** The baseline is fixed lawnmower or geometric routes, with least-squares or particle-filter multilateration under a fitted log-distance path-loss model. Three things could make RSSI non-monotonic in distance and erase any advantage of the optimised geometry:

| Effect | Mechanism |
|---|---|
| Indoor multipath | Reflections add and cancel |
| Body shadowing | The robot's own body blocks the signal |
| Antenna orientation | The M5Stamp C3's antenna gain varies with heading |

To show this, compare simulated and measured error for each route shape. Adaptive or learned routing is justified only if a classical information-gain planner fails.

**First-week test and kill criterion.** Make the Pico 2 W, or a spare ESP32, a fixed beacon. Drive the 3pi+, with the C3 logging RSSI, on straight passes at several offsets, using webcam or odometry ground truth. **Kill** if RSSI shows no usable monotonic trend with distance beyond about 1–2 m (estimated threshold). In that case every route geometry is equivalent and localisation error exceeds half the arena width.

**Hardware and compute.**

| Need | Status | Label |
|---|---|---|
| 3pi+, Stamp C3, Pico 2 W, webcam | Owned | per your brief |
| One robot driving each "team member's" route in turn | Reproduces the four-robot sample pattern for a static emitter, but loses simultaneity, which you must state | valid (estimated) |
| Directional antenna | Optional; price not found | — |
| Up to three borrowed 3pi+ | Optional | — |
| Spend | £0–£50 | estimated |
| Compute | Trivial | estimated |

**Headline figure.** Real against simulated localisation error and detection rate, for each route geometry and antenna type.

**Best realistic and floor outcomes.** At best, the geometry ranking holds with a measured degradation factor. The floor is "moving-robot indoor RSSI cannot support better than X m", which makes route geometry irrelevant. That result is useful to the Hunt lab either way.

**BRL supervision fit.**

| Person | Fit |
|---|---|
| Edmund Hunt | Strongest. Senior author of the paper. His RAEng fellowship, "robots for a safer world", covers multi-robot systems for monitoring, emergency services and security ([RAEng](https://www.raeng.org.uk/programmes-and-prizes/programmes/uk-grants-and-prizes/support-for-research/research-awardees/research-fellowships-awardees/2021-2026/dr-edmund-hunt)). Whether that fellowship ended or was extended in 2026 was not found. |
| Paul O'Dowd | Strong. Co-supervises Hunt-lab PhD students and keeps a BLE/WiFi RSSI localisation repository. |
| Sabine Hauert | Partial, through swarm engineering. |
| Arthur Richards, Lars Kunze, Alan Winfield | No overlap. |

More widely, the co-authors Adam Morris and Timothy Pelham, and Simon Jones (Gaussian-belief-propagation spatial awareness), are relevant.

**Skills it builds.** Embedded RF logging, multilateration and particle filters, field-style experimental design, and SIGINT-adjacent signal work that defence employers recognise without it being a security project.

**BRL equipment and what to ask for.** Ask for arena space in BRL's real RF environment, and for up to three borrowed 3pi+ robots for simultaneous runs. Ask the Hunt lab for the SSRR simulator or its propagation model, and optionally for time at the Fenswood field site it uses ([Hunt Lab](https://www.huntlab.uk/?p=10)).

| Filter | Verdict | Reason |
|---|---|---|
| 1 Robotics first | Pass | RF sensing from a moving robot is the subject. |
| 2 Baseline named | Pass | Classical multilateration and fixed routes. |
| 3 Gap is real | Pass | Real propagation; time-multiplexing is stated. |
| 4 Method justified | Pass | Classical optimisation. Learning is not justified, which is a fit issue, not a filter failure. |
| 5 Nearest work read | Incomplete | Abstract only; propagation model unknown. |
| 6 Feasible | Pass | Owned hardware. |
| 7 First-week test | Pass | The RSSI-trend kill test. |
| 8 Useful if learning loses | Pass | A measured RSSI boundary. |

### Candidate 5. Can a cheap arm tell it is failing?

**Research question.** An SO-101 runs imitation-learned policies, and some of its runs fail naturally. Five failure modes are in scope: empty grasp, slip, stall or collision, misplacement, and wrong object. Which of these can free servo telemetry detect at runtime, using Present_Load, Present_Current and |Goal − Present|? And when do learned progress or uncertainty signals detect them earlier, or with fewer false alarms?

**Why now.** Two recent developments set this up.

| Development | What it showed |
|---|---|
| FAR MSc thesis (defended 12 Jun 2026) | A proprioceptive recovery monitor changed success by +8/+4/+42 percentage points for SmolVLA and +10/0/+68 for π0.5, but the large gains came only under staged interference. False positives cut success from 70% to 60% and from 80% to 60% at two positions. The thesis reported no detection metrics ([experiments](https://github.com/HyuanTan/lerobot_far/blob/main/docs/experiments.md)). |
| LeRobot reward models | v0.5.0 (Mar 2026) added SARM. v0.6.0 (Jul 2026) added Robometer, TOPReward, and rollout logging with intervention flags ([v0.6.0 blog](https://github.com/huggingface/blog/blob/main/lerobot-release-v060.md)). |

**Nearest work and what remains open.**

| Work | Status |
|---|---|
| lerobot_far | Documentation read in full |
| FAIL-Detect (RSS 2025) | The strongest generic method. It learns uncertainty scores from training data alone and thresholds them with a conformal band. Released code is simulation-only, and its real-robot results were not checked ([README](https://github.com/CXU-TRI/FAIL-Detect)). |
| Sentinel/STAC, FIPER | Not verified |

What remains open is per-class detectability under natural failures on cheap hardware, comparing telemetry and learned signals head to head. No overlap was found in an incomplete search.

**Strongest simple baseline, and why it might fail.** The baseline is thresholds on joint load, current and position error, plus a gripper-closed check and a webcam object-in-bin check. It might fail for two reasons. First, semantic failures such as a wrong object produce normal telemetry. Second, LeRobot caps the gripper at 50% torque, and at 25% on overload, "to avoid burnout" ([so_follower config](https://github.com/huggingface/lerobot/blob/main/src/lerobot/robots/so_follower/config_so_follower.py)). Gripper load therefore saturates by design, which may blur "holding" and "stalled on an object" (inference). To show it, plot ROC curves per failure class.

**First-week test and kill criterion.** Run a scripted grasp loop, with no learning, on three objects: 30 successes and 30 staged empty or slipping grasps. Log telemetry at 30 Hz and compute AUROC (the area under the ROC curve). Then run 20 ACT rollouts to see the natural failure mix. **Kill** if telemetry AUROC is about 0.95 or higher for the classes that make up about 80% or more of natural failures (thresholds estimated). The project would then collapse into a replication of FAR.

**Hardware and compute.**

| Need | Status | Label |
|---|---|---|
| Arm, plus ACT and diffusion training | As in Candidate 3 | — |
| TOPReward weights | About 16 GB, more than your 12 GB GPU ([topreward doc](https://github.com/huggingface/lerobot/blob/main/docs/source/topreward.mdx)) | sourced |
| Other reward models' VRAM | Not found | — |
| Sampling N action chunks to measure their spread | Multiplies inference cost; BID roughly doubled compute in its setting | BID figure sourced; N× estimated |
| Natural-failure dataset for the floor outcome | About 200 rollouts | estimated |

**Headline figure.** "Servo telemetry alone catches X% of natural SO-101 failures at ≤Y% false alarms; a learned progress model adds Z% recall or W s of lead time on class C."

**Best realistic and floor outcomes.** At best, a detectability matrix plus a measured recovery gain under natural conditions. The floor is a labelled natural-failure dataset and a characterised telemetry detector whose false alarms have been costed.

**BRL supervision fit.**

| Person | Fit |
|---|---|
| Lars Kunze | Good. His PACER fellowship is building a service that analyses recorded event-data logs to anticipate safety issues and explain them causally (snippet level; the exact page is unconfirmed) ([UWE profile](https://people.uwe.ac.uk/Person/LarsKunze)). That directly overlaps runtime failure detection, if you keep the work technical. |
| Sabine Hauert | Good. Her group's data-driven fault-detection line for swarms, from *RA-L* 2022 onwards, is a genuine methods neighbour ([UoB profile](https://www.bristol.ac.uk/person/Sabine-Hauert-f714ce91-f09a-47e1-b573-f4e120cac448/)). |
| Paul O'Dowd | Partial. He supervises PhDs on robot security monitoring. |
| Hunt, Richards, Winfield | Little overlap. |
| Efi Psomopoulou | Wider fit, on grasp and slip detection. |
| Guanqun Cao | Wider fit (UWE; tactile sensing, VLAs). |

**Skills it builds.** Embedded telemetry, ROC and power analysis, generative policies, and runtime monitoring for consequential autonomy.

**BRL equipment and what to ask for.** Everything in Candidate 3's list, plus a GPU of at least 16 GB for TOPReward-class reward models.

| Filter | Verdict | Reason |
|---|---|---|
| 1 Robotics first | Pass | The arm's sensing and behaviour are the subject. |
| 2 Baseline named | Pass | Telemetry thresholds, with a mechanism for the silent classes. |
| 3 Gap is real | Pass, natural failures only | Staged failures only as a secondary test; FAR's large gains were all staged. |
| 4 Method justified | Pass | Generative policies supply samples; telemetry is free. |
| 5 Nearest work read | Partial | A four-month-old MSc thesis; FAIL-Detect's code is simulation-only. |
| 6 Feasible | Pass | Reward-model VRAM unknown. |
| 7 First-week test | Pass | Scripted-grasp AUROC test. |
| 8 Useful if learning loses | Pass | The detectability map. |

### Candidate 6. Does the sim-to-real gap grow with team size?

**Research question.** Take decentralised MARL policies trained in JaxRobotarium and deployed on the Robotarium. Does the sim-to-real performance gap grow with team size (roughly 2 to 16 robots) or with robot density? If it does, does per-robot actuation randomisation remove the growth, or is it driven by interaction effects such as barrier-certificate interventions, near-collisions and timing?

**Why now.**

| Development | What it offers |
|---|---|
| JaxRobotarium (CoRL 2025) | More than 200 real deployments. The gap was small for many task–algorithm pairs but noticeable for some, and simple action-noise randomisation reduced it ([lab post](https://github.com/GT-STAR-Lab/gt-star-lab.github.io/blob/main/papers/_posts/2025-09-28-jain-jaxrobotarium.md)). |
| JaxMARL 0.2.0 (10 Sep 2026) | Lists JaxRobotarium ([PyPI](https://pypi.org/pypi/jaxmarl/json)). |
| Robotarium simulator update (Aug 2026) | Adds sensor models for the GTernal robot ([robotarium_abc.py](https://github.com/robotarium/robotarium_python_simulator/blob/master/rps/robotarium_abc.py)). |
| Ravichandar's research statement (July 2026) | Still calls access "zero-cost" ([statement](https://github.com/GT-STAR-Lab/gt-star-lab.github.io/blob/main/team/pdfs/harish-ravichandar-research-statement-2026.pdf)). |

**Nearest work and what remains open.** Only JaxRobotarium's abstract was read in full, so whether its sim-to-real section already analyses the gap against team size is unknown. MARBLER's supplementary, read in full, shows something more important. Most of its scenarios give every robot all other robots' observations concatenated together ([supplementary](https://github.com/shubhlohiya/MARBLER/blob/gh-pages/assets/supplementary.pdf)). "Decentralised" is therefore nominal, and the input size grows with the number of robots. Ravichandar's statement also names embodiment- and interaction-driven execution as a future direction, which makes overlap a live risk. No overlap was found in an incomplete search.

**Strongest simple baseline, and why it might fail.** The baseline is JaxRobotarium's own fix: "per-robot calibration plus action-noise randomisation removes the gap at every team size". Alongside it, run a classical controller arm (assignment, go-to-goal and barrier certificates) at every team size. The baseline might fail because interaction effects scale with neighbour count, which per-robot noise cannot represent. To show it, plot the gap against team size, with and without randomisation, alongside logged barrier interventions and minimum inter-robot distances.

**First-week test and kill criterion.** Apply for a Robotarium account and train the navigation scenario with 4 and 12 robots on the RTX 3060. In simulation, log how often the barrier functions modify actions with 2, 4, 8 and 16 robots. Then submit three real runs each with 4 and 12 robots.

**Kill** the candidate if any of these hold:

| Kill condition | Why it kills |
|---|---|
| Access or queue times make 30–60 real runs impossible by summer 2027 | Too few runs to measure a scaling trend |
| JaxRobotarium already reports the gap against team size | The question is answered |
| Barrier interventions are negligible at 16 robots, and real 12-robot runs match simulation within run-to-run spread | There is nothing to scale |

**Hardware and compute.**

| Need | Status | Label |
|---|---|---|
| Purchases | £0 | estimated |
| Robotarium access | Free for academic use; 2026 status unverified | sourced |
| Fleet size | About 20 robots; sources conflict | sourced |
| Arena | 3.2 × 2.0 m; robots capped at 0.2 m/s | sourced |
| Server software | Ships PyTorch but not JAX, so deploy through JaxRobotarium's Flax-to-PyTorch path ([libraries.txt](https://github.com/robotarium/robotarium_python_simulator/blob/master/libraries.txt)) | sourced |
| RTX 3060 | Enough for small MLP or RNN policies | estimated |

One warning: **the Robotarium simulator integrates unicycle kinematics with no acceleration limits** ([robotarium.py](https://github.com/robotarium/robotarium_python_simulator/blob/master/rps/robotarium.py)). That is your old failure mode.

**Headline figure.** The normalised sim-to-real return gap plotted against team size, for two or three tasks × two algorithms plus the classical controller, with the share of the gap explained by interaction metrics.

**Best realistic and floor outcomes.** The best realistic outcome is density-driven scaling of the gap, plus a cheap simulator fix, such as simulated barrier delay or interaction noise, that removes it. The floor is "within the Robotarium's safety stack, no measurable growth in the gap up to N robots".

**BRL supervision fit.**

| Person | Fit | Why |
|---|---|---|
| Sabine Hauert | Good | Swarm engineering, and a 2026 MARL fault-robust controller ([ANTS 2026](https://link.springer.com/chapter/10.1007/978-3-032-26123-6_15)) |
| Edmund Hunt, Paul O'Dowd | Moderate | Multi-robot behaviour and heterogeneity |
| Arthur Richards | Partial | Distributed control of vehicle teams |
| Lars Kunze, Alan Winfield | None | — |

The natural external contact is Ravichandar at Georgia Tech, who is not at BRL.

**Skills it builds.** MARL in JAX, sim-to-real experimental design, remote-hardware logistics, and statistics on small samples of real runs. Be honest with yourself that this is hands-off, and you said you like building.

**BRL equipment and what to ask for.** Nothing is essential. Ask for a co-supervisor in multi-robot learning, and optionally for borrowed 3pi+ robots to add a cross-platform data point.

| Filter | Verdict | Reason |
|---|---|---|
| 1 Robotics first | Pass, weak to moderate | Mediated by the Robotarium's controllers. |
| 2 Baseline named | Pass | Randomisation fix plus a classical controller. |
| 3 Gap is real | Pass with caveat | The same motion-capture state is used in simulation and reality. Neighbourhood observations are justified by the scaling question itself. |
| 4 Method justified | Pass | Learned policies are the object of study. |
| 5 Nearest work read | Incomplete | Abstract only. |
| 6 Feasible | Conditional | Depends on Robotarium access. |
| 7 First-week test | Pass | — |
| 8 Useful if learning loses | Pass | A measured scaling boundary. |

### Candidate 7. Simulate more, gain less? Verifier-limited best-of-k on a low-cost arm

**Research question.** An SO-101 pushes an object to a goal around an obstacle. A simulator scores k candidate actions (behaviour-cloning proposals or MPPI samples) and the best is executed. How does real-world success scale with k? Can the point where it plateaus or reverses be predicted from a cheap calibration of the simulator's false-positive rate?

**Why now.**

| Source | What it shows |
|---|---|
| RoboMonkey (CoRL 2025) | Test-time scaling for VLAs, sampling and verifying 16 candidates in 650 ms, about 1.5 Hz ([page source](https://github.com/robomonkey-vla/robomonkey-vla.github.io)) |
| CoVer-VLA (PREPRINT, Feb 2026) | Reported that RoboMonkey's verifier can make π0 worse than no verification ([arXiv](https://arxiv.org/abs/2602.12281)) |
| VERITAS (PREPRINT, June 2026) and DiVeR (PREPRINT, Oct 2026) | Show the area is live ([VERITAS](https://arxiv.org/abs/2606.18247); [DiVeR](https://arxiv.org/abs/2610.04933)) |
| "The Tail and the Judge" (CMU thesis, 28 Jul 2026) | Ties best-of-k training to judges that planners exploit ([thesis](https://publications.ri.cmu.edu/storage/publications/2026/07/MSR_Thesis__1__20260731134034.pdf)) |
| MuJoCo Menagerie | Ships an SO-101 model, but it is not system-identified ([Menagerie](https://github.com/google-deepmind/mujoco_menagerie)) |

**Nearest work and what remains open.** The strongest overlap is the judge chapter of the CMU thesis. It argues that a planner maximising predicted return "will seek out exactly the regions where its world model is most optimistically wrong". It was seen only as a search summary, and its experimental domain is unknown. **That is the biggest novelty risk on this list.** Other relevant work:

| Work | Finding |
|---|---|
| FOREWARN | Traces most of its own failures to world-model prediction error |
| DMPO | Learns a residual on MPPI and beats the best MPC by up to 27% with fewer samples on a real quadrotor ([README](https://github.com/jisacks/dmpo)) |
| Stroebl et al. (PREPRINT) | Show for LLMs that verifier false positives set a ceiling that resampling cannot remove, and that the optimal k can be below 10 ([arXiv](https://arxiv.org/abs/2411.17501v2)) |

What remains open is a calibrated physical-robot measurement of success against k, set against a measured false-positive rate, with a physics simulator as the verifier. No overlap was found in an incomplete search.

**Strongest simple baseline, and why it might fail.** Three baselines apply. The first is k = 1, executing the policy's or MPPI's nominal plan. The second is MPPI with the same simulator and the same total rollout budget, and the third is a hand-coded push primitive with replanning. They might fail because contact and friction mismatch bias any simulator-based selection, and the argmax grows that bias with k (the winner's curse). Whether the primitive already solves the task is unknown.

**First-week test and kill criterion.** Do this in simulation, before buying anything. Build two MuJoCo scenes from the Menagerie SO-101: a "verifier" with nominal friction and mass, and a "truth" scene with plausibly different friction, mass and servo gains. Generate k ∈ {1, …, 64} candidates by Gaussian/MPPI sampling and from a scripted multimodal set (left- and right-push families). Record the truth success of each verifier-selected candidate, along with the verifier's true- and false-positive rates.

**Kill** the candidate if any of these hold:

| Kill condition | Threshold |
|---|---|
| The false-positive rate is near zero across plausible mismatches | — |
| k = 1 or the primitive already succeeds often in the truth scene | ≥90% truth success (estimated) |
| Truth success against k is indistinguishable from the perfect-verifier curve | — |

Then the week-two physical kill: stop if webcam and ArUco markers cannot track the object to within task tolerance at a usable rate. As k grows without bound, success tends to the verifier's precision: p·TPR / (p·TPR + (1−p)·FPR). With p = 0.3, TPR = 0.9 and FPR = 0.1, that is about 0.79 whatever the k. This is the notes' derivation, labelled an estimate.

**Hardware and compute.**

| Need | Status | Label |
|---|---|---|
| Arm | As in Candidate 3 | — |
| ACT on the RTX 3060 | Fits | sourced |
| Diffusion policy | Fits at reduced batch | estimated |
| MuJoCo rollout speed | Not found; measure in week one | — |
| Physical trials with manual resets | About 100 trials per few hours | estimated, unmeasured |
| VLA | Not needed | — |

**Optional TailRL arm.** Train the proposal generator in the verifier scene with RLOO and with TailRL, using your JAX code and the papers' group size of N = 16. Then test on hardware whether TailRL proposals show a *larger* predicted-minus-real gap. Always report k = 1 success as well, because the TailRL paper itself reports that the objective "lowers single-rollout correctness" in its code task. Train against the deterministic simulator verifier, since TailRL assumes deterministic rewards. Kill this arm if, in simulation, verifier error is uncorrelated with verifier score.

**Headline figure.** Real success against log k, for k = 1, behaviour-cloning proposals with the simulator verifier, and MPPI. Overlay the ceiling predicted from the measured p, TPR and FPR, and plot the predicted-minus-realised outcome of the selected candidate as k grows.

**Best realistic and floor outcomes.** The best realistic outcome is a measured k\* beyond which real success plateaus or declines, predicted within a confidence interval. The floor is that simulator verification gives no real gain over k = 1, with the measured false-positive rate explaining why.

**BRL supervision fit.**

| Person | Fit | Why |
|---|---|---|
| Alan Winfield | Conceptual only | His consequence engine simulates candidate actions before selecting one, the natural local framing. In Blum, Winfield & Hafner (2018), a corridor "danger ratio" fell from about 20% to zero at 8–10 simulations per time-step ([Frontiers](https://www.frontiersin.org/journals/robotics-and-ai/articles/10.3389/frobt.2017.00074/epub)). His 2024–2026 output is ethics and standards. |
| Arthur Richards | Good | Robust MPC suits the MPPI baseline |
| Lars Kunze | Partial | Learnt reward profiles |
| Edmund Hunt, Sabine Hauert, Paul O'Dowd | None | — |
| Efi Psomopoulou or Nathan Lepora | Wider | Contact physics |

**Skills it builds.** MuJoCo modelling and system identification, sampling-based MPC, imitation learning with LeRobot, camera calibration and experimental statistics. Optionally, your JAX group-relative RL.

**BRL equipment and what to ask for.** Motion-capture time or a calibrated overhead rig; any BRL arm if you are not buying one; and printer access.

| Filter | Verdict | Reason |
|---|---|---|
| 1 Robotics first | Pass | Contact, friction and servo physics are the subject. |
| 2 Baseline named | Conditional | The primitive or MPPI may saturate. |
| 3 Gap is real | Pass, if… | …the mismatch comes from the real robot and an unidentified model, not injected perturbations; the week-one simulation pair is only a proxy. |
| 4 Method justified | Pass, if… | …demonstrations confirm the task is multimodal. |
| 5 Nearest work read | Partial | The CMU thesis is unread. |
| 6 Feasible | Pass (estimated) | Uses the whole £200. |
| 7 First-week test | Pass | In simulation, before any purchase. |
| 8 Useful if learning loses | Pass | The ceiling measurement is the product. |

## 4. Wildcard: a printed magnetic skin that lets the 3pi+ feel what it pushes

Tactile sensing is not among your stated interests, which is exactly why it belongs here. Of everything outside your list, it combines the best-evidenced cheap sensor, the strongest BRL supervision lineage and a robot you already own.

**Research question.** Can a printed magnetic bumper, in the style of eFlesh or ReSkin and reporting contact location and force, let a 3pi+ push unknown objects to a goal more reliably than a passive V-shaped pusher or bump-switch feedback? The robot may use onboard sensing only.

**Why now.** eFlesh (PREPRINT, June 2025) prints the sensor in TPU on a hobby printer with under $5 of magnets ([README](https://raw.githubusercontent.com/notvenky/eFlesh/main/README.md)). It reports sourced accuracy of 0.5 mm RMSE for contact location, 0.27 N for normal force and 0.12 N for shear force. AnySkin is now sold as a $48 skin, though a reseller lists $49.92 for an unspecified variant ([WowRobo](https://shop.wowrobo.com/products/enhanced-anyskin-premium-crafted-editionwowskin)).

**Nearest work and what remains open.** The closest Bristol work is Tactile Gym 2.0 (*RA-L* 2022). Its `object_push` environment pushes objects along a trajectory with a tactile sensor, and its learned policies transferred zero-shot to a desktop Dobot MG400 arm ([Tactile Gym README](https://raw.githubusercontent.com/dexterousrobot/tactile_gym/main/README.md)). eFlesh, AnySkin and ReSkin are sensing papers, not pushing papers. Mobile-robot tactile pushing was never searched, so overlap is unknown and filter 5 is not met.

**Strongest simple baseline, and why it might fail.** The baseline is a printed passive V-shaped or concave pusher, driven open-loop on dead reckoning, plus reactive correction from the existing bump switches. An overhead-camera closed loop serves only as an evaluation upper bound. The baseline might fail because a V-pusher only stabilises objects whose geometry suits it. Contact location is the key unobserved variable in pushing (prior knowledge of pushing mechanics, unverified in the notes).

**First-week test and kill criterion.** Run 20 open-loop pushes for each of three shapes with a printed V-pusher, and kill the idea if 90% or more reach the goal. Then bench-test the skin with the motors at full power. Pololu warns that the 3pi+'s own magnetometer is disturbed by the robot's motor and encoder magnets ([User's Guide 5.6](https://www.pololu.com/docs/0J83/5.6)), and the same physics threatens a magnetic skin. Kill the idea if that interference pushes contact-location error above about 5 mm (estimated).

**Hardware and compute.**

| Need | Status | Label |
|---|---|---|
| TPU-capable printer (eFlesh used a Bambu X1C) | Ask BRL | sourced |
| Magnets | Under $5 | sourced |
| ReSkin 5X or MLX90393 board | Price not found; the README warns the MLX90393 is "currently very low-stock" ([circuits README](https://raw.githubusercontent.com/raunaqbhirangi/reskin_sensor/main/circuits/README.md)) | sourced warning |
| AnySkin readings | Drift and need re-zeroing ([AnySkin README](https://raw.githubusercontent.com/raunaqbhirangi/anyskin/main/README.md)) | sourced |
| Contact-estimation model | A two-layer, 128-unit MLP that runs on a CPU | sourced architecture |

**Headline figure and outcomes.** Plot push success rate and final pose error across shapes against the V-pusher and bump-switch baselines. The best realistic outcome is that touch closes most of the gap to camera-based pushing on irregular objects. The floor is "a V-pusher is enough for shapes X".

**BRL supervision fit.** Of the six people you named, only O'Dowd fits, as owner of the 3pi+ add-on boards (I2C extension, IR communication, M5 hat). Looking wider, the strongest fit at BRL is Nathan Lepora's Tactile Robotics Group: TacTip, DigiTac, Tactile Gym, and 2025 reviews in *T-RO* and *IJRR* ([UoB group](https://www.bristol.ac.uk/engineering/research/tactile-robotics/); [papers](https://lepora.com/papers/)). Others are Psomopoulou (tactile grasp control), Cao at UWE (tactile sensing), Ben Ward-Cherrier (status unverified) and Martin Pearson (whiskers; no outputs found after 2023).

**Skills and equipment.** The project builds sensor fabrication, magnetometer signal processing, contact mechanics, embedded I2C and small learned regressors. Ask BRL for TPU-capable printer access, and ask the Tactile Gym team for advice on the `object_push` sim-to-real pipeline.

| Filter | Verdict |
|---|---|
| 1 Robotics first | Pass |
| 2 Baseline named | Pass |
| 3 Gap is real | Pass: onboard sensing only, with the camera used only for evaluation |
| 4 Method justified | Pass |
| 5 Nearest work read | **Not met** |
| 6 Feasible | Pass, given printer access |
| 7 First-week test | Pass |
| 8 Useful if the method loses | Pass |

**A second wildcard: magnetic self-interference and floor maps (a GNC estimation project, not learning).** The question is whether the 3pi+'s own magnetic interference can be compensated well enough for two things: usable magnetometer heading at speed, and repeatable floor-level magnetic maps for re-localisation. The interference has three parts: hard iron, motor current, and rotating encoder magnets that alias at low sampling rates. The evidence is unusually good. Pololu itself documents the interference. The LIS3MDL datasheet gives a typical zero-gauss offset of ±1 gauss (100 µT) against 3.2 mgauss of noise at ±12 gauss ([datasheet copy](https://github.com/Arunvishnu123/iot-project/blob/master/docs/lis3mdl.pdf)). And the motor shaft turns at roughly 445 rev/s at top speed (estimated), while the library samples the magnetometer at 10 Hz, so shaft-rate interference aliases at almost every useful speed.

The physics-first baseline is a Tolles–Lawson-style regression, the classical aircraft magnetic-compensation model. Hager et al. (PREPRINT, Mar 2026) keep exactly that and put a neural network only on the residual ([arXiv](https://arxiv.org/pdf/2603.08265)). Christensen et al. (IROS 2017) already learned compensation of a robot's own distortion ([DFKI](https://robotik.dfki-bremen.de/en/research/publications/9087)).

Week one has two tests. First, a lifted-wheel PWM sweep with the magnetometer logged at its maximum rate (155 Hz in ultra-high-performance mode) alongside the encoders. Second, a survey of a 10 cm grid over about 1 m² on two separate days. Kill the idea if any of these holds (thresholds estimated): the map's spatial variation is below about three times the residual self-interference; the day-to-day correlation is below about 0.8; or the anomaly correlation length is shorter than the dead-reckoning error. The learning arm almost certainly fails filter 4 unless the physics residual stays well above sensor noise.

The defence link is real: the Royal Navy–University of Sussex quantum magnetometer trials were announced on 15 Oct 2025 ([Royal Navy](https://www.royalnavy.mod.uk/news/2025/october/15/20251015-rn-university-of-sussex-quantum-trials)). But this project is a robot-scale teaching analogue, not a contribution to that field.

## 5. Tempting but fails: the directions that break a filter

The filter numbers match your eight filters. Rows marked "conditional" fail only on current evidence or access.

**3pi+ control and simulation**

| Direction | Fails | Why |
|---|---|---|
| Adaptive control under wheel, surface or payload change, in general form (your open item) | 2, 3 | The motor rail is regulated at 8 V. Mid-speed surface changes alter gain by only 3–7%. Payload effects are linear in parameters. Wheel diameter is odometry calibration. Survives only as Candidate 1's traction-limit half. |
| Battery-state adaptation | 3 | Regulated motor rail ([SL0](https://github.com/paulodowd/SEMTM0042_43/blob/main/Labsheets/Supp/SL0_MeasuringBatteryVoltage.ipynb)). |
| RL time-optimal line following | 2, 3 | Pololu's own PID example caps the Standard Edition at 50% speed. The task is limited by five sensors with no preview, not by control ([LineFollowerPID](https://raw.githubusercontent.com/pololu/pololu-3pi-plus-32u4-arduino-library/master/examples/LineFollowerPID/LineFollowerPID.ino)). |
| Waypoint racing with webcam or ArUco tracking | 3 | 100 ms of tracker latency at 1.5 m/s is 15 cm (estimated). The gap would be the instrument's, not the robot's. |
| Off-board WiFi learned control | 2, 4 | On-board PI wins at wheel level. No WiFi jitter figure was sourced. |
| "Tiny policy on a microcontroller" as the question | 1, 4 | An embedded benchmark. RLtools already reports microcontroller inference. |
| Differentiable-simulation policy optimisation; a GPU simulator as the contribution | 4 | Method-driven. A vmapped JAX planar model is enough. |
| Friction prediction from IR reflectance; fleet meta-learned dynamics | 2 (3) | A lookup table or a 1–2 minute per-robot identification probably wins (estimated). |
| Learned CBF, DeepReach or neural reach-avoid filter | 4, 2 | The state is low-dimensional, the analytic braking CBF exists, and `hj_reachability` solves the grid case in JAX ([README](https://raw.githubusercontent.com/StanfordASL/hj_reachability/main/README.md)). |
| RL under a safety filter; simulate-before-acting or MPPI with learned proposals on the 3pi+ | 2, 3, 4 | A 2-D action space, weak multimodality, cheap rollouts — and a hand-written controller already won. |
| A "self-model" of the 3pi+ | 4 | It is system identification. |

**3pi+ estimation and sensing**

| Direction | Fails | Why |
|---|---|---|
| IMU-only learned odometry (RoNIN, TLIO, AirIO, Tartan IMU) | 2, 3 | The encoders already measure velocity. |
| Learned gyro denoising | 2 | Zero-rate bias updates at every stop leave little to learn. |
| KalmanNet-style learned Kalman gain | 4 | The dynamics are known. The problem is non-stationary noise. |
| End-to-end RL "when to re-localise" as the headline | 4, 2 | RiskRL already does it in simulation. Keep it as one arm of Candidate 2. |
| Standalone learned slip detection | 3, 2 | Encoder–gyro disagreement plus χ² innovation gating is strong. |
| Cooperative multi-robot localisation | 5 now; 6 | Needs extra robots and inter-robot sensing the 3pi+ lacks. The learned variant also fails 4. |
| Whiskers; actively scanned single-beam ToF; contact microphone | 3; 3; 5, 2 | A fixed ToF ranger or the IMU/encoder signatures are the baselines. |
| Learned reactive navigation (VL53L5CX or LiDAR); visual imitation on a XIAO ESP32-S3 | 2 (4) | Vector-field histograms (VFH), potential fields, SLAM, colour or ArUco tracking, and a cheap optical-flow chip. |
| Conformal safety around moving people | 6, 3 | No exteroception. An overhead camera would hand the robot perfect state. |
| 1-D magnetic route re-localisation as a standalone project; quantum magnetometer navigation | 3; 6 | Tape markers are trivially better on a bench. Quantum sensors are far beyond £200 (estimated). |

**Learning objectives and RL post-training**

| Direction | Fails | Why |
|---|---|---|
| TailRL/MaxRL versus RLOO for robot policies, in simulation or real | 4, 2, 6 | No rare-success structure. Groups of rollouts from one context need identical physical resets. The papers assume deterministic (TailRL) or perfectly verified (MaxRL) rewards. RLOO-style advantages already lift a one-demo policy from 4% to 97%+ in simulation ([RIPT-VLA](https://github.com/Ariostgx/ript-vla)). |
| Tail objectives for robust sim-to-real | 4, 5 | CVaR-ensemble training such as EPOpt covers this (prior knowledge, unverified). Your earlier tests found small effects from the objective. |
| Group-relative objectives for multi-robot teams | 4 | Repeats your rejected federated-TailRL idea. |
| RL post-training of a VLA on the SO-101 | 6, 5 | SimpleVLA-RL uses 8 × A800 80 GB GPUs ([README](https://github.com/PRIME-RL/SimpleVLA-RL)). The area is crowded in simulation. |
| HIL-SERL replication on the SO-101 | 4, 5 as a research question | A documented pipeline with no question attached. It makes a good warm-up. |
| Retry-aware pass@k policies; DSRL steering trained with TailRL as a standalone project | 2, 3; 4 | Scripted retries are strong, and physical attempts are not i.i.d. DSRL steering belongs only as Candidate 7's optional arm. |

**Manipulation, VLAs and world models**

| Direction | Fails | Why |
|---|---|---|
| Real-time chunking (RTC) or latency-aware chunking on the SO-101 | 5, likely 3 | At least seven papers between June 2025 and Aug 2026. SmolVLA succeeds at about 78% with synchronous and asynchronous inference alike ([SmolVLA blog](https://github.com/huggingface/blog/blob/main/smolvla.md)). |
| Quantised Jetson Orin Nano inference driving the arm | 3, 5 | The desktop can drive a tethered arm. Quantised ACT on an 8 GB Jetson is already published (PREPRINT) ([2608.03938](https://arxiv.org/abs/2608.03938)). |
| VLA benchmarking; world-model policies; video world models as verifiers | 4, 5, 6 | The June 2026 benchmark exists, and π0.5 fine-tuning needs 24–40 GB. NVIDIA's Jetson VLA tutorials cover the AGX Thor only ([tutorials](https://github.com/NVIDIA-AI-IOT/jetson-generative-ai-playground/tree/main/src/content/tutorials/vla)). |
| Best-of-N with VLM reward-model verifiers from LeRobot | 6, plus a design flaw | TOPReward weights are about 16 GB. These models judge video after execution, so they cannot rank actions beforehand. |
| A learned backlash compensator on its own; leader-versus-phone teleoperation data quality | 2; 6 | Classical outer loops suffice. A leader plus follower exceeds £200 in the UK. |
| Visuo-tactile imitation with AnySkin; a Jacobian-field self-model; pan-tilt active vision | 5; 6, 2; 2 | Crowded. The Jacobian-field model was trained on 4 × A100 GPUs. An extra static webcam is the baseline. |
| Evaluation statistics only; a Jetson VLA deployment benchmark | 1; 1, 4 | Neither asks a robotics question. Fold them in as chapters. |

**Multi-robot and swarms**

| Direction | Fails | Why |
|---|---|---|
| Learned versus occlusion-based collective transport on 3pi+ robots | 3, 6, 2 | The 3pi+ cannot perceive occlusion. The classical strategy is provably correct for convex objects and ran with 20 e-pucks in 43 of 45 trials ([Sheffield](https://sheffield.ac.uk/naturalrobotics/supp/2014-002)). |
| GNN or MARL on borrowed 3pi+ robots with overhead-camera "virtual sensors" | 3 | Decentralisation becomes a manufactured constraint over hidden perfect state. |
| LLM-generated swarm controllers (LLM2Swarm, CommandSwarm) | 1, 4, 6 | Method-first. LLM2Swarm's showcases ran on simulated e-pucks ([README](https://github.com/Pold87/LLM2Swarm)). |
| Learned communication scheduling, or robustness to ESP-NOW loss | 2, 3 | Indoor ESP-NOW delivered packets above 92–98% with under 20–40 ms latency for 3–7 robots (low-tier venue) ([IIETA](https://iieta.org/journals/jesa/paper/10.18280/jesa.590507)). Studying loss would mean injecting it. Run the measurement anyway as a by-product. |
| Learned Byzantine-resilient consensus; learned task allocation for 3–6 robots | 2, 3 | W-MSR (classical resilient consensus) and auction or greedy allocation are near-optimal at this scale (inference). |
| AutoMoDe-style automatic design for a 3pi+ swarm | 6, 5 | No proximity or inter-robot sensing. IRIDIA dominates the area. |
| Behavioural heterogeneity on a taped graph | 1, 4 borderline | The graph abstraction makes the robot a backdrop. |
| Moving BRL's MARL fault-robust transport policy onto real DOTS robots | 6, conditional | Depends on unconfirmed DOTS access, and it is the host lab's own agenda. |

**Security**

| Direction | Fails | Why |
|---|---|---|
| Learned adaptive flight control under acoustic gyro injection | 6 conditional, 5 partial | It is robotics-first, but the Crazyflie 2.1+ costs £214.95–£221.54 and was sold out or restocking ([RobotShop UK](https://uk.robotshop.com/products/bitcraze-crazyflie-21)). Revive it only if BRL lends a drone. |
| Replicating acoustic injection or Hall-encoder spoofing on the 3pi+ | 5, 1, 3 | Crowded (ten or more papers since 2015), and security would be only the backdrop. |
| Magnetometer or localisation spoofing | Already rejected | No new evidence. |
| Replicating "Learning to Fly in Seconds"; detecting robot-policy watermarks with a webcam | 6, 5; 1 | Budget and replication; a security question with the robot only as the medium. |

## 6. Where your interests are steering you wrong

**Learning objectives are your strongest pull and the clearest mismatch.** MaxRL and TailRL were both read in full, and both rest on assumptions that a physical robot violates. MaxRL assumes binary rewards from a perfect verifier and says it "does not extend directly to continuous or arbitrarily valued rewards". TailRL assumes a deterministic reward, so that "all randomness in the reward comes from the policy" ([MaxRL preprint](https://github.com/Zanette-Labs/MaxRL/blob/master/site/static/pdf/MaxRL_Preprint.pdf); [TailRL](https://github.com/Zanette-Labs/TailRL-website/blob/main/static/TailRL.pdf)). Both need many rollouts from the same context; the maze and code experiments use 16. TailRL's advantage is "concentrated where high-reward rollouts are attainable but rare": initial maze success ranged from 0.83% to 0.012%. In its code task, the objective even "lowers single-rollout correctness". A physical robot has stochastic outcomes and manual resets, and your checkpoint task had no rare-success structure. That is why TailRL tied RLOO and a ten-line controller won. The legitimate robotics home for these objectives is training proposals for a simulator-verified planner, which is Candidate 7's optional arm. The interesting hypothesis there is not that TailRL wins. It is that tail objectives *amplify* verifier error. Keep the objective as an instrument, not the subject.

**RL as the default controller for the 3pi+ will keep losing.** The plant is benign. The motor rail is regulated at 8 V, mid-speed gain changes by only 3–7% between surfaces, and payload effects are linear in parameters. Learning therefore earns a place only at the physical limits or under non-stationary noise. That is exactly where both leading candidates go, and it is why each plans for classical methods winning.

**Swarms with one robot invite manufactured gaps.** Your 3pi+ has no proximity or inter-robot sensing ([Pololu library](https://github.com/pololu/pololu-3pi-plus-32u4-arduino-library)). Learned coordination on borrowed robots would therefore need an overhead camera, and that camera hands every robot global state. BRL's 2025–2026 swarm learning is neuroevolution and MARL in simulation ([EvoApps 2025](https://link.springer.com/chapter/10.1007/978-3-031-90062-4_19); [ANTS 2026](https://link.springer.com/chapter/10.1007/978-3-032-26123-6_15)). The best real-robot evidence, from IRIDIA's e-puck work, says modular controllers cross the reality gap better than neural ones ([IRIDIA](https://iridia.ulb.ac.be/~mbiro/paperi/GarPagStuBir2024AIS.pdf)). If multi-robot work is non-negotiable, the honest routes are Candidate 6, which is remote and hands-off, or Candidate 4, which studies sensing geometry with no learning.

**The cyber reflex points at a crowded field.** At least ten papers since 2015 have injected signals into MEMS sensors, and three of your five rejected ideas grew from the same reflex. Security strengthens a robotics question only where physics is the subject. The one such idea found, adaptive flight control under acoustic gyro injection, needs a drone your budget cannot buy. Bring the adversarial mindset to evaluation instead. Pre-register thresholds, test on the worst surfaces, cost your false alarms, and hunt for hidden perfect state.

**Embedded skill is your instrument, not your question.** "A tiny policy on a microcontroller" is a benchmark, and RLtools has already measured it. What makes Candidates 1 and 2 credible is 1 kHz logging on the 32U4, wheel-speed PI running on the 32U4, and magnetometer sampling synchronised to the encoders.

**"Consequential autonomy" has a local pull towards ethics, governance and assurance.** Winfield's consequence engine sits inside a body of work that is now mostly ethics and standards. Kunze's Responsible Technology roles lean towards governance. The technical version of your interest is Candidates 2 and 5: decisions under calibrated uncertainty, scored in regret and false alarms.

**What to do next.**

| When | Action |
|---|---|
| November 2026 | Run the shared 3pi+ week; it costs nothing. |
| December 2026 | Do the full-text reads in Section 7. |
| Until the 3pi+ branch is settled | Hold the £200. If Candidate 7 tempts you, run its simulation kill test before buying an arm. |
| Now | Ask Paul O'Dowd (UoB MSc Programme Director) and Martin Pearson (named as programme leader on the UWE course page) for the EMATM0055 dates and the project-allocation process; the notes found neither ([UWE MSc](https://courses.uwe.ac.uk/H67B1/robotics)). |
| Teaching block 2 | The dissertation starts around late January 2027 (estimated, unverified). |

The 2026/27 catalogue lists "Robot Learning for Control" and "Advanced Control & Dynamics" among the 15-credit TB-2 options, and either would feed the top two candidates ([UPC](https://upc.bristol.ac.uk/unit-programme-catalogue/RouteStructure.jsa?byCohort=N&ayrCode=26/27&programmeCode=4EMAT002T)). For your career, a measured boundary backed by an identified model reads well to GNC, embedded and autonomy employers. Do not bend the project towards any particular advert.

## 7. Reading list for the top two candidates, in the order you will need it

Items 1–4 serve both candidates, 5–7 serve Candidate 1 and 8–10 serve Candidate 2. The access column records how far the research strands got on 10 October 2026. Whatever it says, you need to read every item yourself. Items 6–10 are the ones whose full texts nobody has yet read, and they decide filter 5.

| # | Read | When and why | Access on 10 Oct 2026 |
|---|---|---|---|
| 1 | Pololu, [3pi+ 32U4 User's Guide](https://www.pololu.com/docs/0J83/all) §5.4–5.7, with the [Arduino library source](https://github.com/pololu/pololu-3pi-plus-32u4-arduino-library) (motors, encoders, IMU) | Before your first log: encoder counts, PWM timer, IMU registers and the power path | Guide pages blocked, search extracts only. Library source read in full on GitHub. |
| 2 | Bristol RSS repos: [SEMTM0042_43](https://github.com/paulodowd/SEMTM0042_43) labsheets L1, L3, SL0 and SL3 plus the example CSVs, and the [RoboticsScienceAndSystems](https://github.com/paulodowd/RoboticsScienceAndSystems) StampC3 template | Week 0: your logging and middleware starting point, and Bristol's measured dead-band and surface data | Read on GitHub: template headers and CSVs in full, labsheets in their relevant sections |
| 3 | ST [LSM6DS33 datasheet](https://github.com/dddrrreee/cs240lx-20spr/blob/master/labs/8-i2c-accel/docs/LSM6DS33.pdf), DocID027423 Rev 4, Table 3 | Days 1–3: gyro bias, drift and noise budget; accelerometer noise for slip detection | Read directly from a GitHub-hosted copy. Confirm against ST's current revision. |
| 4 | Borenstein & Feng (1996), [UMBmark](https://cs.au.dk/~ocaprani/legolab/DigitalControl.dir/NXT/Lesson10.dir/paper58.pdf), *IEEE T-RA* 12(6) | Days 3–4: the odometry-calibration protocol | Search extract only. A copy exists but was not read. |
| 5 | MuJoCo [changelog 3.5.0–3.12.0](https://raw.githubusercontent.com/google-deepmind/mujoco/main/doc/changelog.rst), [`dcmotor` XML reference](https://raw.githubusercontent.com/google-deepmind/mujoco/main/doc/XMLreference.rst), [sysid README](https://raw.githubusercontent.com/google-deepmind/mujoco/main/python/mujoco/sysid/README.md) and [MJX parity table](https://raw.githubusercontent.com/google-deepmind/mujoco/main/doc/mjx.rst) | Weeks 2–4: building the rungs of the identified simulator | Read in full on GitHub |
| 6 | He et al., [ASAP](https://roboticsconference.org/program/papers/66/) (RSS 2025) | When you add the learned-residual rung to the ladder | README partial plus secondary summaries; primary limitations unread. Read in full. |
| 7 | O'Connell et al., [Neural-Fly](https://www.science.org/doi/abs/10.1126/scirobotics.abm6597) (*Science Robotics*, May 2022) | When you design the adaptation comparison: its L1, INDI and no-learning ablation are the template | README only; paper blocked. Read in full. |
| 8 | Shek et al., [RiskRL](https://arxiv.org/pdf/2411.02788) (arXiv 2411.02788 v2; venue unverified), with Williams et al., [When to Localize?](https://arxiv.org/abs/2411.08281) (SSRR 2024) | If Candidate 2 survives week one: the decision-rule arms and their stated limitations | Abstracts, plus RiskRL's code README (grid worlds). Read both in full. |
| 9 | Marques, Ghaffari & Berenson, [CLAPS](https://arxiv.org/html/2512.10294v1) (PREPRINT, Dec 2025) | When you build the calibration arm: the only conformal SE(2) result on a real small differential-drive robot | Abstract only. Read in full. |
| 10 | Lindemann et al., [conformal prediction for verification and control](https://arxiv.org/pdf/2409.00536) (PREPRINT, v3 Aug 2025). Use it as the route into Gibbs & Candès's adaptive conformal inference (NeurIPS 2021), which has no link in the notes. | When you implement online recalibration under sparse, decision-dependent labels | Survey abstract only; ACI itself not retrieved. Read both. |
