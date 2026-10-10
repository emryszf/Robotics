# Safe learning for control, active perception, wildcard areas and physics-level robotics security (2024–2026): MSc candidate screening

**Read this first: what was accessed, and how deeply (research date Sat 10 Oct 2026).**

- **Access limits.**
  - WebFetch failed with a DNS error (`ENOTFOUND`) on every domain tried.
  - Egress policy blocks arXiv, publisher sites, `*.github.io`, pololu.com and bristol.ac.uk (HTTP 403).
  - The web-search budget for this turn, shared across agents, ran out after 21 of my searches.
  - On the coordinator's tip, I then read primary-source **GitHub repository files** through raw.githubusercontent.com.
- **Reading-depth tags** used for each source below:
  - `[R]` = repository README or source file read in full. This is primary documentation written by the authors, but it is **not** the paper.
  - `[S]` = search-engine summary only, at most abstract-level, sometimes with fragments of page text.
  - `[T]` = title or a single line only.
- **No paper was read in full or at methods level.** Quotes marked "(via search summary)" should be checked against the page before they are reused.
- **Filter 5 (nearest work read at methods level) is therefore unmet for every candidate.** "No overlap found" means only that one search found nothing; it is not evidence of novelty.
- **Prior knowledge is kept separate.** Anything from my own background knowledge is marked "UNVERIFIED (prior knowledge)" and is kept out of Cited Findings.
- **Searches planned but never run** because the budget was exhausted:
  - AV-ALOHA quantitative results;
  - Eye-Robot and ViA;
  - Hearing Touch;
  - BRL staff pages;
  - mobile-robot tactile pushing;
  - physical attacks on learned visuomotor policies;
  - CoRL 2025, RSS 2026, ICRA 2026 and IROS 2026 paper lists.

---

## 1. Safety filters on real low-cost robots (2024–2026): where has a filter EXPANDED capability, and what is the strongest simple alternative?

### Takeaway
There is evidence that safety filters can expand capability, but always in the same way: the filter lets a fast, unverified controller run inside a guard. Two examples:
- **ETH Zürich's predictive safety filter** let a deep imitation-learned controller race a miniature RC car aggressively while staying on the track (RA-L 2021).
- **ABS (RSS 2024)** uses a learned reach-avoid value network to switch between an agile policy and a recovery policy. Its paper contrasts this with earlier controllers held below 1.0 m/s for safety. It ran on a Unitree Go1 with a ZED mini camera and an Orin NX.

For a 3pi+-class robot the situation is different. It has no range sensing, and along its heading it behaves almost like a double integrator. The analytic braking-distance rule is already the control barrier function (CBF), and both CBF filters for unicycle robots (Robotarium) and numerical level-set Hamilton–Jacobi (HJ) reachability solvers in JAX are available off the shelf. So a filter only earns its place through how it handles uncertainty: braking that depends on surface and battery, dead-reckoning drift, and latency. That points to calibrated (conformal) margins. Learned CBFs and DeepReach do not fit this problem.

### Cited Findings
- **Predictive safety filter for racing.** Tearle, Wabersich, Carron & Zeilinger, "A predictive safety filter for learning-based racing control", *IEEE RA-L* 6 (2021) 7635–7642, ETH Zürich. `[S]` — [arXiv 2102.11907](https://arxiv.org/pdf/2102.11907); [Semantic Scholar](https://www.semanticscholar.org/paper/A-Predictive-Safety-Filter-for-Learning-Based-Tearle-Wabersich/14a0f4a0f3b54b9c2db20df14b78afe1bd86df44)
  - The filter keeps the car within the track boundaries for any potentially unsafe input, including learning-based controllers.
  - "the safe set is extended in real-time through predictive control backup trajectories" (via search summary).
  - Demonstrated on a physical miniature RC car, with human-in-the-loop racing and with deep imitation learning.
  - The authors note that performance improves with a more accurate model, a longer horizon or a larger terminal set.
  - No lap-time numbers were retrieved.
- **Earlier filter from the same group:** Wabersich & Zeilinger, predictive safety filter for learning-based control of constrained nonlinear systems. `[T]` — [arXiv 1812.05506](https://arxiv.org/pdf/1812.05506)
- **ABS: Agile But Safe** — He, Zhang, Xiao, He, Liu & Shi, **RSS 2024**.
  - Method `[S]`:
    - an agile policy paired with a recovery policy;
    - a learned reach-avoid value network, inspired by HJ reachability, chooses when to switch and guides the recovery policy;
    - the value network is trained with a discounted reach-avoid Bellman equation on data from the agile policy;
    - a separate network turns depth images into ray distances;
    - trained in simulation, run onboard.
  - Paper framing: "Existing studies either develop conservative controllers (< 1.0 m/s) to ensure safety, or focus on agility without considering potentially fatal collisions" (via search summary).
  - The reported peak of about 3.1 m/s comes from a secondary summary and is **unverified**.
  - Code README `[R]`:
    - hardware is a Unitree Go1 EDU, a ZED mini camera and an Orin NX (16 GB);
    - the codebase includes a **PPO-Lagrangian agile-policy variant**, i.e. a constrained-RL baseline;
    - the reach-avoid value design cites Hsu et al., "Safety and liveness guarantees through reach-avoid reinforcement learning" (RSS 2021);
    - licence is CC BY-NC 4.0.
  - Sources: [RSS 2024 proceedings p059](https://roboticsproceedings.org/rss20/p059.html); [arXiv 2401.17583](https://arxiv.org/html/2401.17583v3); [alphaxiv summary](https://alphaxiv.org/overview/2401.17583v3); [LeCAR-Lab/ABS README](https://raw.githubusercontent.com/LeCAR-Lab/ABS/main/README.md)
- **Tunable real-time safety filters via set-based CBFs** (arXiv 2507.07805, PREPRINT, Jul 2025). `[S]` — [arXiv 2507.07805](https://arxiv.org/pdf/2507.07805)
  - A single class-K parameter sets how aggressively the filter intervenes.
  - Hardware test on a permanent-magnet synchronous motor drive: an explicit QP inside a **150 µs** sampling window, with a **28.04 µs worst-case execution time** (sourced).
  - In simulation, a low parameter gives early, heavily damped intervention; a high one allows a more aggressive approach to the boundary.
- **Neural CBFs for safe navigation** (arXiv 2407.19907, PREPRINT). Hardware tests at 20 Hz, indoors and outdoors. The filter changed an unsafe reference command so the robot stopped before an obstacle. This shows restriction, not expansion. `[S]` — [arXiv 2407.19907](https://arxiv.org/pdf/2407.19907)
- **Latent CBF** — "How to Train Your Latent CBF" (arXiv 2511.18606, PREPRINT, Nov 2025). Reports a "45% decrease in action override magnitude relative to the corresponding LR filter" for its CBF-with-GP filter (setting not confirmed). `[S]` — [arXiv 2511.18606](https://arxiv.org/pdf/2511.18606)
- **Survey.** Hsu, Hu & Fisac, "The Safety Filter: A Unified View of Safety-Critical Control in Autonomous Systems", *Annual Review of Control, Robotics, and Autonomous Systems* 7(1):47–72 (2024). `[S]` — [Princeton portal](https://collaborate.princeton.edu/en/publications/the-safety-filter-a-unified-view-of-safety-critical-control-in-au/); [arXiv 2309.05837](https://arxiv.org/abs/2309.05837v1)
- **RL under a safety filter.** Oh, Nguyen, Hu & Fisac, "Provably Optimal Reinforcement Learning Under Safety Filtering" (arXiv 2510.18082, **Oct 2025**; listed in the IASEAI proceedings on ojs.aaai.org, year not confirmed). A least-restrictive filter (one that prevents all violations while intervening as little as possible) lets an RL agent converge to the optimal safety-constrained policy. `[S]` — [arXiv 2510.18082](https://arxiv.org/pdf/2510.18082v2); [IASEAI listing](https://ojs.aaai.org/index.php/IASEAI/article/view/43046)
- **Conformal Decision Theory.** Lekeufack, Angelopoulos, Bajcsy, Jordan & Malik, **ICRA 2024**, pp. 11668–11675. `[S]` — [arXiv 2310.05921](https://arxiv.org/pdf/2310.05921). The venue and pages come from a later paper's reference list as reported by the search tool, probably [arXiv 2509.25124](https://arxiv.org/pdf/2509.25124); that attribution is uncertain.
  - Test domains:
    - robot navigation around people, using the Stanford Drone Dataset, with the reward tuned online "to be safe but efficient";
    - manufacturing, where the method "directly calibrates the speed of the conveyor belt under a robot to achieve high-throughput and successful robot grasps" (via search summary);
    - trading.
  - No physical-hardware experiment was confirmed.
- **Off-the-shelf filters and reachability tools** `[R]`:
  - The **Robotarium Python simulator** ships `barrier_certificates.py`, with "SI and unicycle barrier certificates" solved as quadratic programs with CVXOPT. It includes `uni_barriers.py` and `dead_reckoning.py` examples and models the GTernal robot. — [robotarium_python_simulator README](https://raw.githubusercontent.com/robotarium/robotarium_python_simulator/master/README.md)
  - **`hj_reachability`** (StanfordASL) is a JAX package of numerical solvers for Hamilton–Jacobi–Isaacs PDEs on grids. It lists toolboxLS, helperOC, beacls and optimized_dp as its inspirations, and its TODO list says it would like to "incorporate neural-network-based PDE solvers", citing DeepReach. — [hj_reachability README](https://raw.githubusercontent.com/StanfordASL/hj_reachability/main/README.md)
  - **DeepReach** (Bansal & Tomlin) is an **ICRA 2021** method for *high-dimensional* reachability; its example is `air3D`. — [DeepReach README](https://raw.githubusercontent.com/smlbansal/deepreach/master/README.md)
- **Other 2024–2026 leads, title only and not assessed** `[T]`:
  - closed-form dual-barrier CBF on incrementally built occupancy grids ([arXiv 2605.05182](https://arxiv.org/html/2605.05182v2), May 2026);
  - input-to-state-safe CBF whole-body control for humanoids ([arXiv 2605.25546](https://arxiv.org/pdf/2605.25546));
  - policy CBFs built at runtime ([arXiv 2410.11157](https://arxiv.org/html/2410.11157));
  - "Drive Fast, Learn Faster: On-Board RL for High Performance Autonomous Racing" ([arXiv 2505.07321](https://arxiv.org/pdf/2505.07321));
  - "Safety with Agency", a human-centred filter for AI-assisted motorsport ([arXiv 2504.11717](https://arxiv.org/pdf/2504.11717));
  - conformalised semantic maps for safe planning ([arXiv 2509.25124](https://arxiv.org/pdf/2509.25124));
  - conformal policy learning under distribution shift ([arXiv 2311.01457](https://arxiv.org/pdf/2311.01457));
  - Ames et al.'s CBF review ([arXiv 1903.11199](https://arxiv.org/pdf/1903.11199));
  - "How to Train Your Neural CBF" ([arXiv 2310.15478](https://arxiv.org/pdf/2310.15478)).

  All are PREPRINTS unless stated otherwise.

### Inferences
- **"Expansion" is relative.** In both hardware precedents (Tearle 2021, ABS 2024), the gain is measured against the conservative controller that would otherwise be deployed. The filter is what makes it acceptable to run a fast, unverified policy. Neither precedent was on a robot as sensor-poor as the 3pi+.
- **On a 3pi+, the simple rule is already the barrier.** Hazards are either in a map or sensed only on contact (line sensors at an edge, bump sensors). The state-dependent braking rule `v ≤ sqrt(2·a_min·(d−m)) − a_min·τ` is the CBF condition for a near double-integrator (standard kinematics; inference).
  - Unicycle CBF filters already exist (Robotarium), and low-dimensional HJ problems can be solved numerically on a grid in JAX (`hj_reachability`, a level-set-style solver).
  - Learned CBFs and DeepReach, which exists for high-dimensional systems, are therefore not justified by the problem structure (filter 4).
  - For novelty, the filter itself is solved tooling. Anything new has to come from how uncertainty is handled.
- **The one framing worth carrying forward.** A filter changes what a 3pi+ can do only if it replaces a global speed cap with a limit that depends on state and conditions, and whose margin is calibrated online for surface, battery and dead-reckoning drift.
  - Conformal Decision Theory's "calibrate the speed knob for throughput under a risk bound" maps directly onto this. This is **Candidate A**.
  - The Pololu library exposes `readBatteryMillivolts()` `[R]`, so battery level is observable on the robot.
- **Computing cost is small.** Explicit-QP CBF filters run in tens of microseconds on an embedded drive (28.04 µs, sourced). A filter on an ESP32-C3 at 100 Hz–1 kHz should be cheap (estimated; different CPU).
- **Theory–hardware gap, relevant only if RL is needed.** Oh et al. (Oct 2025) assume a perfect, least-restrictive filter. On cheap hardware the filter's model will be wrong (slip, latency), so "how much does an imperfect filter bias what RL learns?" is open. It only matters if the task genuinely needs RL; in the student's earlier checkpoint-navigation test it did not.
- **Robotarium context.** The Robotarium uses barrier certificates so that users' untrusted code can run on shared robots (UNVERIFIED, prior knowledge: Wang, Ames & Egerstedt, T-RO 2017). This is the clearest "filter enables a new capability" story on cheap differential-drive robots. It is multi-robot and, by now, established.

### Gaps
- No lap-time or speed numbers for Tearle 2021; ABS's 3.1 m/s is from a secondary source only.
- No 2024–2026 paper found that shows a filter expanding capability on a robot as cheap as a 3pi+. **This search was incomplete.**
- Not checked: Wang, Ames & Egerstedt (T-RO 2017), Robey 2020, Dawson 2023, Lindemann 2023, and shielding papers. These names come from the brief.
- Hardware use in Conformal Decision Theory is not confirmed.
- No head-to-head comparison of a CBF filter against a braking-distance rule was found.

---

## 2. Active perception beyond localisation: is any of it feasible with a webcam plus £200?

### Takeaway
The 2024–25 active-vision manipulation results do not transfer to a webcam and £200 as published:
- **AV-ALOHA** (PREPRINT) uses a whole 7-DoF arm just to carry the camera.
- **Observe Then Act** was tested in simulation only (RLBench).

An SO-101 follower arm costs about $122 in parts (excluding printing), so a budget active-vision rig is borderline-feasible. The strongest cheap baseline, though, is simply one or two extra static webcams. AV-ALOHA's static-versus-active comparison could not be retrieved.

On the 3pi+, the credible form of "active perception beyond localisation" is active system identification: spend a short probing manoeuvre to identify braking before committing to speed. This is folded into Candidate A.

### Cited Findings
- **AV-ALOHA**, "Active Vision Might Be All You Need" — Chuang, Lee, Gao, Naddaf-Sh & Soltani (UC Berkeley / UC Davis), arXiv 2409.17435 v1/v2, **PREPRINT** (no venue found). `[S]` — [arXiv 2409.17435](https://arxiv.org/pdf/2409.17435); [v2](https://arxiv.org/html/2409.17435v2)
  - Extends ALOHA 2 with an extra 7-DoF arm that carries only a stereo camera and seeks the best viewpoint.
  - Released as an open-source "low-cost" teleoperation system.
- **Observe Then Act** — Guokang Wang et al., arXiv 2409.14891 v1–v3. `[S]` — [arXiv 2409.14891](https://arxiv.org/pdf/2409.14891); [v3](https://arxiv.org/html/2409.14891v3)
  - A camera next-best-view policy chained with a gripper next-best-pose policy, trained by few-shot RL.
  - Tested on 8 viewpoint-constrained RLBench tasks in simulation, with "consistent gains over baselines".
  - A summary says v3 was accepted at an IEEE venue in Jan 2025; which venue is unconfirmed, so treat it as PREPRINT.
- **SO-101 costs** (TheRobotStudio SO-ARM100 README) `[R]` — [README](https://raw.githubusercontent.com/TheRobotStudio/SO-ARM100/main/README.md)
  - Follower-arm parts: **$121.94 / €124.3**.
  - Leader plus follower: **$229.88 / €226.3**.
  - Both totals exclude 3D-printed parts.
  - The repo has wrist-camera mounts for 32×32 UVC modules, RealSense D405/D435 and a generic webcam.
  - It also lists a dual-arm mobile build costing $660.
- **UC Davis "Eye, Robot" active-vision blog.** `[T]` — [UC Davis blog](https://www.ucdavis.edu/index%2ephp/blog/eye-robot-researchers-give-machines-new-perspective-active-vision)

### Inferences
- **Feasibility.**
  - Perception-only active vision, such as a pan-tilt webcam on hobby servos, is cheap (price not sourced).
  - Coupling active vision to manipulation needs an arm. An SO-101 follower (about $122 in parts, sourced, plus UK VAT, shipping and printing, not sourced) uses most of the £200.
  - Teleoperated demonstrations would also need the leader arm, which pushes the total past budget (estimated).
- **Strongest baseline and kill test.** The baseline is more static cameras, or a wrist camera. Kill the idea if a second static webcam matches the active camera on the chosen occluded task (filters 2 and 3).
- **On the 3pi+.** Actively scanning a single-beam time-of-flight sensor on a servo is a manufactured gap if a fixed multizone time-of-flight sensor would do the job (not sourced). The better framing is information for speed (Candidate A).
- **Ranking.** Active vision ranks below Candidates A–D on the available evidence. It overlaps #4's low-cost arm scope.

### Gaps
- AV-ALOHA's quantitative results were not retrieved.
- ViA, Eye-Robot and 2025–26 next-best-view-for-manipulation work were not checked.
- Pan-tilt kit and webcam prices were not sourced.

---

## 3. Wildcard (a): tactile sensing — the TacTip line from Lepora's group at BRL, magnetic skins, and fit to a cheap arm

### Takeaway
Tactile sensing is the best-evidenced and best-supervised wildcard at Bristol:
- Nathan Lepora's group produced TacTip, DigiTac and Tactile Gym 2.0 (RA-L 2022), which includes a **tactile object-pushing environment** with zero-shot sim-to-real on a ~£2k Dobot MG400.
- The group published 2025 reviews in T-RO and IJRR.

Magnetic skins are now very cheap:
- **eFlesh:** under $5 of magnets, TPU on a hobby printer, and the ReSkin/AnySkin magnetometer board (MLX90393 chips read by a QT Py microcontroller).
- **AnySkin:** commercial skin at $48, with an SO-101 mount variant at $128.

The student owns no arm. The cheapest tactile question on hardware the student already has is a printed magnetic tactile bumper on the 3pi+ for pushing objects (Candidate D).

### Cited Findings
- **Lepora's 2025 output**, from his papers page as shown in search summaries (the page itself could not be opened). `[S]` — [lepora.com/papers](https://lepora.com/papers/)
  - "Tactile Robotics: Past and Future" (*IJRR* 2025);
  - "Tactile Robotics: An Outlook" (*T-RO* 2025, with Luo, Yuan, Althoefer, Cheng and Dahiya);
  - "Shear-based Grasp Control for Multi-fingered Underactuated Tactile Robotic Hands" (*T-RO* 2025, Ford et al.);
  - StereoTacTip (*IEEE/ASME T-Mech* 2025);
  - "Educational SoftHand-A", built from LEGO MINDSTORMS (*IROS* 2025);
  - "NeuralTouch" (in submission).
- **Tactile Gym** (dexterousrobot/tactile_gym README) `[R]` — [README](https://raw.githubusercontent.com/dexterousrobot/tactile_gym/main/README.md); [Tactile Gym 2.0 README](https://raw.githubusercontent.com/ac-93/tactile_gym/main/README.md)
  - Tactile Gym 2.0 is Lin, Lloyd, Church & Lepora, *IEEE RA-L* 7(4):10754–10761 (2022). Tactile Gym 1.0 is Church et al., CoRL 2021 (PMLR 164).
  - Environments:
    - `edge_follow`;
    - `surface_follow`, including a `-v2` for 4-DoF robots (MG400, Magician);
    - `object_roll`;
    - **`object_push`** — "push the object along the trajectory" with a tactile sensor;
    - `object_balance`.
  - Observation modes are oracle, tactile, visual and visuotactile.
  - The 2.0 README says the group integrated "a low-cost off-the-shelf industrial-level-accuracy desktop robot DOBOT MG400 for three learning environments" and "transferred the learned policies into the real world without any further fine-tuning". It covers DigiTac, DIGIT and TacTip sensors.
- **DigiTac and the MG400 rig.** DigiTac uses a desktop Dobot MG400 costing about **£2k**. Tactile Gym 2.0 lists the MG400 at about **$2.7k** (0.05 mm accuracy, 440 mm reach, 0.75 kg payload) against about **$45k** for a UR5 set-up (sourced via summary). `[S]` — [arXiv 2206.13657](https://arxiv.org/abs/2206.13657); [Bristol research portal](https://research-information.bris.ac.uk/en/publications/digitac-a-digit-tactip-hybrid-tactile-sensor-for-comparing-low-co/); [arXiv 2207.10763](https://arxiv.org/pdf/2207.10763)
- **AnyRotate:** "Gravity-Invariant In-Hand Object Rotation with Sim-to-Real Touch". `[R, title only]` — [AnyRotate README](https://raw.githubusercontent.com/maxyang27896/AnyRotate/main/README.md)
- **eFlesh** — Pattabiraman, Huang, Panozzo, Zorin, Pinto & Bhirangi (NYU), arXiv 2506.09994, **PREPRINT, Jun 2025**. `[S]` — [arXiv](https://arxiv.org/html/2506.09994v1); `[R]` — [eFlesh README](https://raw.githubusercontent.com/notvenky/eFlesh/main/README.md)
  - Reported accuracy: contact localisation RMSE **0.5 mm**; force RMSE **0.27 N** normal and **0.12 N** shear (sourced).
  - Magnets cost **< $5** (sourced).
  - Repo build details:
    - TPU 95A printed on a Bambu Lab X1 Carbon;
    - N52 neodymium magnets, 1/8″ × 3/8″ for standard sensors and 1/16″ × 3/16″ for fingertips;
    - "We use the rigid magnetometer PCBs used in Reskin and AnySkin", read by a QT Py;
    - models are "a simple two layered MLP with 128 nodes".
  - Also demonstrated: slip detection on a Hello Stretch robot, and visuo-tactile policies with an "average success rate of >90%" on four precise manipulation tasks.
- **ReSkin board.** The ReSkin "5X" board uses the **MLX90393** magnetometer, which the README notes is "currently very low-stock". Boards are ordered from PCB vendors such as PCBWay, and "shipping is often the biggest bottleneck in price and time". `[R]` — [ReSkin circuits README](https://raw.githubusercontent.com/raunaqbhirangi/reskin_sensor/main/circuits/README.md)
- **AnySkin** (NYU, 2024; venue not confirmed).
  - The starter kit is a magnetometer board, a QWIIC cable and an Adafruit QT Py.
  - Readings drift over time: "Over time, the sensor measurement will drift and you may see a non-zero measurement without any load being applied", so the zero point must be recalibrated. `[R]` — [AnySkin README](https://raw.githubusercontent.com/raunaqbhirangi/anyskin/main/README.md)
  - WowRobo lists a **$48.00** "touch skin only" variant and a **$128.00** "SO-101 structural part" variant. A reseller lists **$49.92** without saying which variant, which **conflicts**. `[S]` — [WowRobo](https://shop.wowrobo.com/products/enhanced-anyskin-premium-crafted-editionwowskin); [reseller](https://www.voltsprays.com/product/wowrobo-robotics-enhanced-anyskin-premium-crafted-edition-wowskin/); [SO-ARM100 add-on list](https://github.com/adityachandrath/SO-ARM100)
- **"Seeing Touch from Motion: A Unified Modality-Aware Visuo-Tactile Policy..."** (arXiv 2606.29941, **Jun 2026**, PREPRINT; authors not confirmed). `[T]` — [arXiv 2606.29941](https://arxiv.org/pdf/2606.29941)

### Inferences
- **Fitting touch to a £100–£200 arm.**
  - An SO-101 follower (about $122 in parts, sourced) plus AnySkin ($48 skin, or $128 for the SO-101 variant) comes to roughly $170–$250 before VAT, shipping and printing (estimated). That is at or over £200.
  - Imitation learning with touch on an SO-101 is crowded (2025–26 visuo-tactile policy papers; eFlesh's own Visuo-Skin policies), so it fails filter 5.
- **On the student's own hardware:** a magnetic tactile bumper on the 3pi+ for pushing objects (Candidate D).
  - Nearest Bristol work: Tactile Gym's `object_push`, a tactile pushing environment on an arm with a sim-to-real pipeline `[R]`.
  - Mobile-robot tactile pushing itself was not searched.
- **Platform-specific risks.**
  - The 3pi+ carries an LIS3MDL magnetometer (see §6) and motors with magnetic parts, so magnetic interference both ways must be measured in week 1. The encoder type is UNVERIFIED.
  - MLX90393 supply was low-stock.
  - AnySkin drifts and needs re-zeroing.

### Gaps
- Not checked: how AnyRotate and Bi-Touch work, whether a "Tactile Gym 3" exists (none was found under the repos tried), and what a TacTip costs to build.
- MLX90393 breakout and 5X board prices were not sourced.
- Whether Lepora's group offers MSc projects, and has MG400 or Magician arms to lend, is for #7.

---

## 4. Wildcard (b): acoustic and vibration sensing

### Takeaway
Contact-microphone sensing is well evidenced for manipulation, and the hardware is cheap:
- SonicSense (CoRL 2024: 83 objects, a heuristic exploration policy, a hand costing "just over $200" per press reports);
- ManiWAV (CoRL 2024: two contact mics on a UMI gripper);
- VibeCheck (April 2025) and Audio-VLA (November 2025).

On the student's wheeled robot, though, the classical baseline is strong: IMU and encoder vibration and slip signatures. Acoustic terrain classification is also an old topic. No acoustic candidate survives on hardware the student owns.

### Cited Findings
- **SonicSense** — Liu & Chen (Duke), **CoRL 2024**. `[R]` — [SonicSense README](https://raw.githubusercontent.com/generalroboticslab/SonicSense/main/README.md); `[S]` — [arXiv 2406.17932](https://arxiv.org/pdf/2406.17932); [The Robot Report](https://www.therobotreport.com/sonicsense-lets-robots-perceive-objects-via-in-hand-acoustic-vibration/); [Unite.AI](https://www.unite.ai/sonicsense-gives-robots-human-like-sensing-abilities-through-acoustic-vibrations/)
  - README: "a diverse set of 83 real-world objects"; "a simple but effective heuristic exploration policy" with end-to-end learned fusion; CAD and hardware instructions in the repo; the code acknowledges PyLX-16A (LX-16A servos).
  - Build cost "just over $200", from press reports quoting Duke.
- **ManiWAV** (**CoRL 2024**). `[R]` — [ManiWAV README](https://raw.githubusercontent.com/real-stanford/maniwav/main/README.md); `[S]` — [arXiv 2406.19464](https://arxiv.org/html/2406.19464)
  - Built on the Universal Manipulation Interface (UMI), a handheld data-collection gripper, with two contact mics (an Amazon part), wired to the GoPro media mod's external mic port.
  - A game-capture card streams vision and audio together.
- **VibeCheck** (arXiv 2504.15535, PREPRINT, Apr 2025). `[T]` — [arXiv 2504.15535](https://arxiv.org/pdf/2504.15535)
- **Audio-VLA** (arXiv 2511.09958, PREPRINT, Nov 2025). Adds contact audio to a vision-language-action model, and notes that Hearing Touch and ManiWAV both use piezo contact mics. `[S]` — [arXiv 2511.09958](https://arxiv.org/pdf/2511.09958)

### Inferences
- **3pi+ variant: a contact mic for slip, terrain or contact detection.**
  - Strongest baseline: the IMU at its maximum rate plus encoder–gyro slip checks.
  - Kill test: if IMU and encoder features alone reach ≥ ~95% (estimated threshold), the mic adds nothing.
  - Verdict: likely killed, and the area is classical (e.g., Valada & Burgard, IJRR 2017; UNVERIFIED, prior knowledge).
- **No Bristol link** was found this session.

### Gaps
- Not checked: Hearing Touch's venue and method; piezo prices; any acoustic-sensing group at BRL.

---

## 5. Wildcard (c): robot self-modelling

### Takeaway
There is strong 2025 evidence for self-models learned from a camera:
- Jacobian fields (*Nature*, June 2025), with data from a pneumatic hand, an Allegro hand, a Handed Shearing Auxetics platform and a Poppy arm;
- Hu, Lin & Lipson (*Nature Machine Intelligence*, 2025).

The full 3D Jacobian-field pipeline was trained on 4×A100 or 4×A8000 GPUs, so it does not fit a 12 GB RTX 3060 at the published scale; only the 2D variant fits. For the student it would also need an arm, and it would have to beat classical camera-based kinematic calibration. On the 3pi+, self-modelling reduces to system identification (filter 4).

### Cited Findings
- **Jacobian fields.** Li, Zhang, Chen, Matusik, Liu, Rus & Sitzmann, "Controlling diverse robots by inferring Jacobian fields with deep networks", *Nature* 643(8070):89–95, published 25 Jun 2025. `[S]` — [PMC12221969](https://www.ncbi.nlm.nih.gov/pmc/articles/PMC12221969/); [RePEc](https://ideas.repec.org/a/nat/nature/v643y2025i8070d10.1038_s41586-025-09170-0.html); [arXiv 2407.08722](https://arxiv.org/pdf/2407.08722); `[R]` — [NJF README](https://raw.githubusercontent.com/sizhe-li/neural-jacobian-field/main/README.md)
  - Video is mapped to a visuomotor Jacobian field, and control then uses one camera.
  - The preprint says training data came from 12 consumer RGB-D cameras.
  - README, robots in the dataset: a pneumatic hand mounted on an arm, an Allegro hand, a Handed Shearing Auxetics platform, and a Poppy arm.
  - README FAQ, compute: training was tested on "4 × A8000s / 4 × A100s"; the default of 50M steps implies an estimated "1300 hours ... on an NVIDIA A40", though one can stop early; inference on "a single RTX 4090". The 2D tutorials take about 30 minutes each.
  - Follow-up **VERA** (arXiv 2605.27817, May 2026; code released 21 Jul 2026) turns a frozen video model into a policy through a Jacobian inverse-dynamics model, on a Franka arm and an Allegro hand.
- **Self-simulation from video.** Hu, Lin & Lipson, "Teaching Robots to Build Simulations of Themselves", *Nature Machine Intelligence* 2025 (arXiv 2311.12151). `[S]` — [arXiv 2311.12151](https://arxiv.org/pdf/2311.12151); [Columbia news](https://engineering.columbia.edu/about/news/robots-learn-how-move-watching-themselves-0)
  - A kinematic self-model is built from a short video from one ordinary camera and used for planning, anomaly detection and damage recovery.
  - Earlier work used a 4-DoF arm, about 1,000 trajectories, under 35 hours of training, and reached about 4 cm accuracy (Columbia news).
  - One secondary report gives *Science Robotics* as the venue, which conflicts with the arXiv journal reference.

### Inferences
- **Arm-based candidate:** a self-model of an SO-101 versus ArUco kinematic calibration. It probably fails filter 2, because calibration is likely to be enough. The 3D Jacobian-field version fails filter 6 on compute (sourced 4×A100 training); a 2D variant is feasible but weaker.
- **3pi+ variant:** this is system identification, so it fails filter 4. It is also covered by #1.
- **No Bristol link** was found this session.

### Gaps
- Not retrieved: quantitative accuracy figures; whether the Poppy arm counts as low-cost (UNVERIFIED, prior knowledge says it is a 3D-printed open-source arm); Chen et al. (*Science Robotics* 2022) not checked.

---

## 6. Wildcard (d): RL-trained low-level control on microcontrollers and Crazyflie-class drones

### Takeaway
"Train in seconds, run on a microcontroller" has moved from a single platform to a peer-reviewed adaptive foundation policy:
- **2024:** Learning to Fly in Seconds (RA-L 2024) trained in 18 s on an M1 MacBook Pro.
- **2026:** RAPTOR (*Science Robotics* 11(114), 2026) is a 2,084-parameter recurrent policy that adapts in context to 10 real quadrotors from 32 g to 2.4 kg. It has reference deployments for PX4, Betaflight, Crazyflie and the **M5StampFly**, an M5Stack micro-drone (ESP32-S3-based: UNVERIFIED, prior knowledge).

Inference speed varies about 160-fold between chips (Teensy 4.1 to ESP32-C3) for a two-layer [64, 64] network (sourced):

| Chip | Time per inference | Rate |
|---|---|---|
| Teensy 4.1 | 41 µs | — |
| Crazyflie | 293 µs | 3.4 kHz |
| ESP32 (Xtensa) | 279 µs | — |
| ESP32-C3, the student's chip | 6,645 µs | ~150 Hz |

Crazyflie hardware costs more than the £200 budget even before a radio (UK prices, sourced):

| Model | UK price | Stock at search time |
|---|---|---|
| Crazyflie 2.1+ | £214.95–£221.54 | sold out / restocking |
| Crazyflie 2.1 Brushless | £395.79–£461.54 | sold out / restocking |

### Cited Findings
- **Learning to Fly in Seconds** — Eschmann, Albani & Loianno, **RA-L 2024**, arXiv 2311.13081. `[S]` — [arXiv 2311.13081](https://arxiv.org/abs/2311.13081); [NSF PAR](https://par.nsf.gov/servlets/purl/10557794); [GitHub](https://github.com/arplaboratory/learning-to-fly); [ResearchGate](https://www.researchgate.net/publication/380286884_Learning_to_Fly_in_Seconds)
  - Direct RPM control after "only 18 seconds of training on a consumer-grade laptop". The RLtools README `[R]` says "Trained in 18s on a 2020 MacBook Pro (M1) using RLtools TD3".
  - Repo limitations (via search summary): take-off "has a big sim2real gap (ground-effect, z-ranger/optical-flow estimation behavior) and a large setpoint error", so take off with the stock controller and then switch. The policy regulates global yaw to 0.
  - Unverified excerpt: motor delays are not modelled, "which can lead to instabilities in deployment"; over 200 flights with under 0.05 m RMSE at up to 2.0 m/s.
- **RLtools** — *JMLR* 25(301):1–19 (2024). `[R]` — [RLtools README](https://raw.githubusercontent.com/rl-tools/rl-tools/main/README.md); [microcontroller inference table](https://raw.githubusercontent.com/rl-tools/media/master/microcontroller_inference.png); `[S]` — [arXiv 2306.03530](https://arxiv.org/pdf/2306.03530)
  - Embedded support: Teensy for both inference and training; Crazyflie, ESP32 and PX4 for inference.
  - The Quick Start's quadrotor SAC example trains in about **4 s on an Apple M3** and about **6 s on a Zen 5** CPU.
  - Inference for a two-layer [64, 64] MLP, generic / DSP library / optimised (sourced, image read):

| Platform | Generic | DSP library | Optimised |
|---|---|---|---|
| Crazyflie | 743 µs | 478 µs | **293 µs (3.4 kHz)** |
| Pixhawk 6C | 133 µs | 93 µs | **53 µs (18.8 kHz)** |
| Teensy 4.1 | 64 µs | 45 µs | **41 µs (24.3 kHz)** |
| ESP32 (Xtensa) | 4,282 µs | **279 µs (3.6 kHz)** | 333 µs |
| **ESP32-C3 (RISC-V)** | 8,716 µs | 6,950 µs | **6,645 µs (150 Hz)** |

  - The paper claims "the first-ever training of a deep RL algorithm directly on a microcontroller (the Teensy 4.1)" `[S]`.
- **RAPTOR** — Eschmann, Albani & Loianno, "RAPTOR: A foundation policy for quadrotor control", ***Science Robotics* 11(114):eaec1481 (2026)**, doi 10.1126/scirobotics.aec1481. arXiv 2509.11481 (v2, 6 Apr 2026). **Peer-reviewed**, so not a preprint. `[R]` — [RAPTOR README](https://raw.githubusercontent.com/rl-tools/raptor/master/README.md); `[S]` — [arXiv 2509.11481](https://arxiv.org/abs/2509.11481v2); [alphaxiv](https://www.alphaxiv.org/overview/2509.11481); [chatpaper](https://chatpaper.com/de/paper/188273)
  - From the abstract `[S]`:
    - "A tiny, three-layer policy with only 2084 parameters is sufficient for zero-shot adaptation to a wide variety of platforms";
    - adaptation happens in context, through a recurrent hidden layer;
    - meta-imitation training: 1,000 sampled quadrotors, an RL teacher for each, distilled into one student;
    - tested on 10 real quadrotors from 32 g to 2.4 kg, covering tracking, indoor and outdoor flight, wind, poking and propeller changes.
  - From the README `[R]`:
    - the policy "is trained for control at 100 Hz", and can run faster (e.g. 400–500 Hz) by advancing its latent state only every N steps;
    - deployment reference implementations exist for **PX4, Betaflight, Crazyflie and M5StampFly**;
    - `pip install foundation-policy==1.0.1`, and the training simulator is "l2f";
    - "the pre-training of the 1000 teachers will take a long time"; teacher checkpoints are provided.
- **Crazyflie IMU configuration**, from the firmware's BMI088 driver (crazyflie-firmware `src/hal/src/sensors_bmi088_bmp3xx.c`; the BMI088 is used on the 2.1-series per the driver name, while older boards may differ: UNVERIFIED) `[R, configuration lines]` — [source file](https://raw.githubusercontent.com/bitcraze/crazyflie-firmware/master/src/hal/src/sensors_bmi088_bmp3xx.c)
  - **BMI088** IMU read at 1,000 Hz.
  - Gyro ±2000 dps, set to `BMI088_GYRO_BW_116_ODR_1000_HZ`.
  - Accelerometer ±24 g at 1,600 Hz ODR.
  - Software 2-pole low-pass filters: gyro **80 Hz**, accelerometer **30 Hz**.
  - BMP3xx barometer at 50 Hz.
- **M5StampFly.** The m5stack README is minimal. The StampFly operation manual (M5Fly-kanazawa/StampFly2024June) describes Stabilize and Acro modes, an altitude-hold mode that is "in development", an impact-stop function and serial telemetry. Price and IMU part are **not sourced**. `[R]` — [M5StampFly README](https://raw.githubusercontent.com/m5stack/M5StampFly/main/README.md); [StampFly manual](https://raw.githubusercontent.com/M5Fly-kanazawa/StampFly2024June/main/README.md)
- **Prices**, retrieved via search on 10 Oct 2026 (page dates unknown) `[S]`:
  - Crazyflie 2.1+: Bitcraze store **$240.00** ([store](https://store.bitcraze.io/products/crazyflie-2-1-plus)); RobotShop UK **£221.54**, "re-stocking soon" ([link](https://uk.robotshop.com/products/bitcraze-crazyflie-21)); Unmanned Tech Shop **£214.95**, sold out ([link](https://www.unmannedtechshop.co.uk/products/crazyflie-2-1-drone-hacker-development-platform-kit)). The UTS URL says "2.1 kit", so the model is ambiguous.
  - Crazyflie 2.1 Brushless: RobotShop UK **£461.54**, restocking ([link](https://uk.robotshop.com/products/crazyflie-21-brushless-version-open-source-micro-quadcopter-drone)); UTS **£395.79**, sold out ([link](https://www.unmannedtechshop.co.uk/de/products/crazyflie-2-1-brushless-hacker-drone)).
  - A Lighthouse swarm bundle is listed at £5,495; whether the listing is current is unclear ([listing](https://www.unmannedtechshop.co.uk/collections/all?page=30)).
- **On-device CNN training on an ESP32-S3** (webmcu-ai, arXiv 2604.23012 / 2604.22834, PREPRINT, Apr 2026): about 9 min per training run and 159 ms per frame. Which of the two papers holds the figure is uncertain. `[S]` — [arXiv 2604.23012](https://arxiv.org/pdf/2604.23012)
- **3pi+ hardware** (Pololu library README) `[R]` — [README](https://raw.githubusercontent.com/pololu/pololu-3pi-plus-32u4-arduino-library/master/README.md)
  - ATmega32U4 microcontroller.
  - On-board "LSM6DS33 accelerometer and gyro, and an LIS3MDL compass".
  - Battery voltage readable with `readBatteryMillivolts()`.
  - The library was last updated in 2022 (v1.1.3), so current production boards may differ.

### Inferences
- **Why now (Candidate B).** RAPTOR is peer-reviewed in 2026, ships deployment code for small flight controllers, including Crazyflie and M5StampFly, and gives a concrete recipe: randomised dynamics, teachers, then a distilled recurrent student. That makes "a tiny in-context adaptive controller on a cheap ground robot" a sharp transfer. It lands directly on the student's open item, adaptive motion control under wheel, surface or payload changes (#1's scope).
- **Compute fit.**
  - On the ESP32-C3, a [64, 64] MLP manages only about **150 Hz** (sourced). RAPTOR's 2,084-parameter network is smaller, so it might reach roughly 300–450 Hz (**estimated** from parameter count; the recurrent unit's cost is not sourced). That is enough for a 100 Hz policy.
  - The Pico 2 W (RP2350, which has a hardware FPU; UNVERIFIED, prior knowledge) has no RLtools benchmark.
  - The ATmega32U4 is not a realistic host (estimated).
- **The simulator must be upgraded.** It needs actuator limits, motor dynamics, latency and battery sag. The earlier failure came partly from having no acceleration limits.
- **Baseline risk.** On a wheeled robot, the main effect of changing surface is a traction limit, and no controller can create traction. An adaptive policy can only avoid demanding more acceleration than the traction allows. That pulls B towards A's problem, which is knowing the limit (inference).
- **Cheaper drone route.** The M5StampFly is a RAPTOR reference target (its chip, an ESP32-S3, is UNVERIFIED prior knowledge). If it is cheap (price not sourced), it may be a budget alternative to a Crazyflie for Candidate C. Ask BRL's aerial robotics lab, or check the M5Stack store.

### Gaps
- Not found: prices for the Crazyradio 2.0, Flow deck v2 and M5StampFly, and the M5StampFly's IMU part.
- Not retrieved: RAPTOR's baselines, its compute and wall-clock training time, and its limitations section.
- No ESP32-S3 or RP2350 inference figures.

---

## 7. Wildcard (e): other cheap, evidenced areas — whiskers, and printed skins on mobile robots

### Takeaway
TU Delft's barometer-based whiskers for tiny drones (October 2025; reportedly *Nature Communications*, September 2026) run tactile navigation on a 192 KB-RAM microcontroller. That is the same RAM as the Crazyflie's STM32F405 (UNVERIFIED, prior knowledge).

On a ground robot, though, a cheap time-of-flight ranger is a strong baseline, so whiskers on the 3pi+ fail the "real gap" filter except in narrow cases such as transparent obstacles. The more defensible cheap tactile idea is the magnetic skin bumper (Candidate D).

### Cited Findings
- **Whisker-based tactile flight.** Ye, de Croon & Hamaza (TU Delft), "Whisker-based Tactile Flight for Tiny Drones", arXiv 2510.03119 (3 Oct 2025). `[S]` — [arXiv](https://arxiv.org/abs/2510.03119v1); [ScienceBlog](https://scienceblog.com/?p=601630); [Dronezine](https://www.dronezine.it/?p=460083); [The Debrief](https://thedebrief.org/bio-inspired-whiskers-let-tiny-drones-navigate-total-darkness-by-touch/)
  - A **3.2 g** apparatus of barometer-based whiskers; depth estimation to "**sub-6 mm**"; runs on a **192-KB RAM** microcontroller (sourced).
  - Reported published in *Nat Commun* on **18 Sep 2026**, DOI 10.1038/s41467-026-77366-7 (**unverified**).
  - Earlier: a biomorphic whisker sensor, ICRA 2024.
- **Whiskers with a spiking network and central pattern generator** (Wiley *Concurrency and Computation: Practice and Experience* 2026, first published Dec 2025): whisker navigation, including in total darkness. `[S]` — [Wiley](https://onlinelibrary.wiley.com/doi/10.1002/cpe.70467)
- **Whisker preprints, titles only** `[T]`: [arXiv 2503.00133](https://arxiv.org/pdf/2503.00133); [arXiv 2507.23305](https://arxiv.org/pdf/2507.23305); [arXiv 2410.14005](https://arxiv.org/html/2410.14005v1).
- **Classic reference:** Jung & Zelinsky (1996), whisker-based mobile robot navigation. `[S]` — [IEEE Xplore](https://ieeexplore.ieee.org/document/570842/)

### Inferences
- **Whiskers on the 3pi+** fail filter 3 against a time-of-flight baseline unless transparent or specular obstacles are the explicit target. They go on the "tempting but fails" list.
- **Bristol's whisker heritage** (BRL; UNVERIFIED, prior knowledge) could make a whisker variant supervisable if a real gap were found.

### Gaps
- BRL whisker staff were not verified.
- Mobile tactile-bumper pushing was **not searched**.

---

## 8. Robotics–security overlaps where physics or sensing is the subject: how crowded?

### Takeaway
Injecting signals into MEMS inertial sensors, acoustically and now by laser, is a crowded and maturing security area:
- Son et al. 2015, WALNUT 2017 and Tu et al. 2018;
- NDSS 2023 recovery work;
- MARS (2025);
- the HEMA teaching platform (2025);
- 2025 laser injection (17 of 25 devices);
- Banshee (July 2026).

The only angle that strengthens a robotics question, rather than serving as backdrop, is how learned or adaptive low-level controllers respond to physically injected sensor corruption. One search found nothing on this. On a Crazyflie, the gyro's 116 Hz bandwidth at 1 kHz output rate, plus an 80 Hz software low-pass filter, means injection would matter mainly through aliasing or non-linearity, which is the WALNUT mechanism.

### Cited Findings
- **Un-Rocking Drones** (**NDSS 2023**). Sound resonates the proof masses of MEMS IMUs on drones; the paper recovers benign values, and finds that sampling jitter strongly affects crashes. `[S]` — [NDSS paper](https://www.ndss-symposium.org/wp-content/uploads/2023/02/ndss2023_f112_paper.pdf); [NDSS page](https://www.ndss-symposium.org/ndss-paper/un-rocking-drones-foundations-of-acoustic-injection-attacks-and-recovery-thereof)
- **MARS** (arXiv 2505.00924, PREPRINT, May 2025). "Deep Auto Encoders (DAE) have been deployed to learn the acoustic resonance signature; however, as we show in this work, they are unable to cope with sensor saturation, unexpected sensor measurements, and communication drops" (via search summary). `[S]` — [arXiv](https://arxiv.org/html/2505.00924v1)
- **HEMA** — "A Hands-on Exploration Platform for MEMS Sensor Attacks" (arXiv 2507.06439, PREPRINT, Jul 2025). `[T]` — [arXiv](https://arxiv.org/pdf/2507.06439)
- **Continuous-wave laser injection into IMUs** (2025): 17 of 25 tested devices "could be implicitly controlled". Attribution came via search summary; verify before use. `[S]` — [ResearchGate](https://www.researchgate.net/publication/398056610_Signal_Injection_Attack_on_Inertial_Measurement_Unit_using_Continuous-Wave_Laser)
- **SoK: sensor spoofing against robotic vehicles.** Gyroscopes and accelerometers are "susceptible to resonant acoustic interference". `[S]` — [arXiv 2205.04662](https://arxiv.org/pdf/2205.04662)
- **Change-point detection mitigation** (MhIMU framework, 2024). `[T]` — [ResearchGate](https://www.researchgate.net/publication/383317892_Acoustic_Attack_Mitigation_Approach_for_MEMS_Inertial_Sensors_Using_Change_Point_Detection_on_MhIMU_Framework)
- **Banshee** (arXiv 2607.09930, PREPRINT, Jul 2026). `[S]` — [arXiv](https://arxiv.org/pdf/2607.09930)
  - Acoustic injection against gimbal-stabilised tracking, giving "consistent, directionally biased gimbal angular offsets in real time".
  - Its related-work table lists KITE as a speaker-plus-piezo attack that biases a drone controller's localisation.
  - It describes WALNUT as exploiting "aliasing and clipping", and Nashimoto et al. as attacking sensor fusion with sound.
- **Crazyflie IMU pipeline (firmware BMI088 driver):** sensors read at 1 kHz, gyro bandwidth 116 Hz, plus an 80 Hz 2-pole software LPF `[R]` (see §6).
- **3pi+ IMU:** LSM6DS33 accelerometer/gyro and LIS3MDL magnetometer, per the Pololu library README `[R]` (see §6).
- **Learned controllers under injection:** one search found **no paper combining** acoustic IMU injection with learned or neural controllers. This is absence of evidence from one search. `[S]`

### Inferences — crowding assessment
| Sub-area | Crowding | Strengthens a robotics question? |
|---|---|---|
| New acoustic/laser attack demos on MEMS IMUs | **Crowded**: at least 10 papers 2015–2026, now with a teaching platform | No; fails filter 5 |
| Learned detectors and recovery for injected IMU data | **Active** (MARS critiques autoencoders) | Weakly |
| How learned or *adaptive* low-level controllers respond to physical injection | **No overlap found (one search)** | **Yes.** Does in-context adaptation mistake injection for a dynamics change? (Candidate C) |
| Attacks on a ground robot's IMU heading or Hall-effect encoders | Replicates the Tu 2018 / ABS wheel-speed-spoofing lineage (UNVERIFIED, prior knowledge); same family as the rejected localisation-spoofing idea | No; fails filters 5 and 1 |

- **Physics that shapes the kill test.** With in-sensor and software low-pass filtering, a resonance drive in the kHz range shows up only through aliasing, clipping or rectification, as a DC bias or a low-frequency component. That is what the test should look for (inference from the firmware settings and Banshee's description of WALNUT).
- **Safety:** high-volume acoustic tests need hearing protection and lab permission.

### Gaps
- Son 2015, Trippel 2017 and Tu 2018 were not re-checked; venues come from the brief.
- The KITE primary paper was not found.
- 2024–26 physical attacks on learned visuomotor policies were **not searched**.
- Whether the BMI088 or LSM6DS33 is susceptible to acoustic resonance is **unknown**.

---

## 9. New since October 2025 that opens an MSc-scale question on cheap hardware

### Takeaway
The October 2025 – October 2026 items that open usable questions are:
- **RAPTOR**, now *Science Robotics* 2026, with Crazyflie and M5StampFly reference deployments → Candidates B and C.
- **Oh et al.** (October 2025), on RL under least-restrictive filters → a hardware question only if the task needs RL.
- **Cheap printed tactile sensing:** eFlesh (June 2025) plus TU Delft's whiskers (October 2025 / September 2026) → Candidate D.

The rest mostly signal crowding or high-dimensional settings.

### Cited Findings
- RAPTOR, *Science Robotics* 11(114) (2026); arXiv v2 6 Apr 2026. — [RAPTOR README](https://raw.githubusercontent.com/rl-tools/raptor/master/README.md) `[R]`
- Whisker-based tactile flight (3 Oct 2025; reportedly *Nat Commun* 18 Sep 2026). — [arXiv 2510.03119](https://arxiv.org/abs/2510.03119v1) `[S]`
- Provably optimal RL under safety filtering (Oct 2025). — [arXiv 2510.18082](https://arxiv.org/pdf/2510.18082v2) `[S]`
- Latent CBF (Nov 2025). — [arXiv 2511.18606](https://arxiv.org/pdf/2511.18606) `[S]`
- Audio-VLA (Nov 2025). — [arXiv 2511.09958](https://arxiv.org/pdf/2511.09958) `[S]`
- Whisker spiking-network navigation (Dec 2025). — [Wiley](https://onlinelibrary.wiley.com/doi/10.1002/cpe.70467) `[S]`
- On-device CNN training on an ESP32-S3 (Apr 2026). — [arXiv 2604.23012](https://arxiv.org/pdf/2604.23012) `[S]`
- Dual-barrier CBF and humanoid input-to-state-safe CBF (May 2026). — [arXiv 2605.05182](https://arxiv.org/html/2605.05182v2); [arXiv 2605.25546](https://arxiv.org/pdf/2605.25546) `[T]`
- VERA, Jacobian inverse dynamics with video models (arXiv 2605.27817, May 2026; code Jul 2026). — [NJF README](https://raw.githubusercontent.com/sizhe-li/neural-jacobian-field/main/README.md) `[R]`
- Seeing Touch from Motion (Jun 2026). — [arXiv 2606.29941](https://arxiv.org/pdf/2606.29941) `[T]`
- Banshee (Jul 2026). — [arXiv 2607.09930](https://arxiv.org/pdf/2607.09930) `[S]`

### Inferences
- Only RAPTOR and the tactile/whisker work open questions that run on hardware the student owns or nearly owns.
- VERA and the 2026 CBF preprints target expensive arms or humanoids.

### Gaps
- No systematic scan of the CoRL 2025, RSS 2026, ICRA 2026 or IROS 2026 paper lists was possible. This section is **incomplete**.

---

## 10. Which candidate questions survive all eight filters, and what is each one's first-week kill test?

### Takeaway
None can be certified as passing all eight filters yet, because filter 5 (methods-level reading of the nearest work) is unmet. On paper, four candidates survive filters 1–4 and 6–8:
- **A — calibrated braking safety filter, or "information for speed", on the 3pi+.** Uses owned hardware and has a crisp kill test.
- **B — RAPTOR-style tiny in-context adaptive controller on the 3pi+.** Peer-reviewed method evidence, but a high risk that the classical baseline wins.
- **D — printed magnetic tactile bumper for pushing objects with the 3pi+.** Best Bristol supervision link (Tactile Gym `object_push` lineage); real risk of magnetic interference.
- **C — adaptive learned flight control under acoustic gyro injection.** Least crowded angle; needs a BRL Crazyflie or an M5StampFly (price unknown).

Everything else fails a named filter (list below).

### Cited Findings
- Key anchors for each candidate:
  - **A:** [Tearle 2021](https://arxiv.org/pdf/2102.11907), [ABS](https://roboticsproceedings.org/rss20/p059.html), [Conformal Decision Theory](https://arxiv.org/pdf/2310.05921), [Oh et al. 2025](https://arxiv.org/pdf/2510.18082v2), [Robotarium barrier certificates](https://raw.githubusercontent.com/robotarium/robotarium_python_simulator/master/README.md), [hj_reachability](https://raw.githubusercontent.com/StanfordASL/hj_reachability/main/README.md), [3pi+ library](https://raw.githubusercontent.com/pololu/pololu-3pi-plus-32u4-arduino-library/master/README.md).
  - **B:** [RAPTOR](https://raw.githubusercontent.com/rl-tools/raptor/master/README.md), [RLtools and its microcontroller table](https://raw.githubusercontent.com/rl-tools/media/master/microcontroller_inference.png), [Learning to Fly in Seconds](https://arxiv.org/abs/2311.13081).
  - **C:** [Un-Rocking Drones](https://www.ndss-symposium.org/ndss-paper/un-rocking-drones-foundations-of-acoustic-injection-attacks-and-recovery-thereof), [MARS](https://arxiv.org/html/2505.00924v1), [Banshee](https://arxiv.org/pdf/2607.09930), [Crazyflie BMI088 configuration](https://raw.githubusercontent.com/bitcraze/crazyflie-firmware/master/src/hal/src/sensors_bmi088_bmp3xx.c), Crazyflie prices from [RobotShop UK](https://uk.robotshop.com/products/bitcraze-crazyflie-21).
  - **D:** [eFlesh](https://raw.githubusercontent.com/notvenky/eFlesh/main/README.md), [ReSkin board](https://raw.githubusercontent.com/raunaqbhirangi/reskin_sensor/main/circuits/README.md), [AnySkin](https://raw.githubusercontent.com/raunaqbhirangi/anyskin/main/README.md), [Tactile Gym](https://raw.githubusercontent.com/dexterousrobot/tactile_gym/main/README.md), [lepora.com/papers](https://lepora.com/papers/).

### Inferences

#### Candidate A — information for speed: a calibrated braking safety filter on the 3pi+ (stretch goal: active friction or position probing)

**(a) Research question.** A 3pi+ works near a mapped hazard edge that it can sense only when its downward sensors reach it. Two questions follow:
- How much average speed does an online-calibrated (conformal) braking-distance filter recover over a fixed worst-case margin, and over a lookup table of per-surface margins, at the same edge-violation rate, across surfaces and battery levels?
- Stretch: when is a short probing or re-localising manoeuvre worth its time?

**(b) Why now.** The "why now" is moderate; no paper from October 2025 onwards opened this specifically.
- Conformal Decision Theory (ICRA 2024) calibrates a speed knob online for throughput under a risk guarantee.
- Oh et al. (Oct 2025) show that least-restrictive filtering preserves RL optimality, which argues for calibrated rather than worst-case margins.
- Latent CBF (Nov 2025) treats reducing filter override as a goal.

**(c) Nearest work.**
- Tearle et al., RA-L 2021: predictive safety filter on a miniature car.
- Conformal Decision Theory, ICRA 2024: online speed calibration, tested on datasets and modelled settings.
- Robotarium unicycle barrier certificates: established tooling, so the filter is not the novelty.
- Still open: calibrated margins on a robot that senses hazards only at contact, where braking depends on the surface; and the value of probing.
- All read at summary or README level only.

**(d) Strongest simple baselines.**
1. A fixed braking rule, `v ≤ sqrt(2·a_min·(d−m)) − a_min·τ`, tuned on the worst surface. This is the analytic CBF.
2. **A per-surface lookup table:** classify the surface from line-sensor reflectance, then apply that surface's fixed margin.
3. A reactive stop when the line sensor reaches the edge.

They could fail if achievable deceleration drifts continuously within a surface class: battery voltage (readable with `readBatteryMillivolts()`), wheel dust, or surfaces the robot has not seen. The worst-case margin then wastes speed, and the lookup table misjudges conditions. A stopping-distance model conditioned on these inputs, with conformal calibration of its residuals, is the structurally justified learned component.

**(e) First-week kill test.**
- About 240 stops: 10 stops × 4 speeds × 3 floor surfaces × 2 battery levels. Measure from encoder counts, with an overhead webcam and an ArUco marker as ground truth.
- **Kill if** the 95th-percentile stopping distance varies across conditions by less than the robot's own dead-reckoning error at the hazard, or by less than ~5 mm (estimated threshold).
- **Kill also if** the 3pi+ cannot reach speeds at which braking distance exceeds the offset between its line sensors and its wheels.

**(f) Hardware and compute.**
- The 3pi+ and the webcam are owned. Its top speed, mass and encoder resolution are **not sourced**: the Pololu product page is blocked, and the library README does not give them.
- Edge rig with a foam catch: **estimated** £0–£20.
- A laptop is enough.
- Optional request to BRL: motion-capture time.

**(g) Headline figure and outcomes.**
- Headline: "At ≤ 1% edge violations, X% higher mean speed than a fixed worst-case margin, and Y% higher than a per-surface lookup table."
- Best realistic: a 15–30% gain on grippy surfaces (estimated).
- Floor: "A fixed margin costs less than 5% of speed below v m/s on this robot." That is still a measured boundary.

**(h) Filter-by-filter verdict.**

| Filter | Verdict |
|---|---|
| 1 Robotics first | Pass |
| 2 Baseline named | Pass: three named baselines |
| 3 Gap is real | Conditional on the week-1 test; ground truth is used only for evaluation |
| 4 Method fits the problem | Pass |
| 5 Nearest work read at methods level | **Not met** |
| 6 Feasible | Pass |
| 7 First-week test | Pass |
| 8 Useful if the method loses | Pass |

**(i) Skills built.** Safety-critical control, conformal and online calibration, embedded real-time control, experimental design. Relevant to GNC and autonomy roles.

**(j) Bristol people.** None found this session (#7).

**Overlap note.** This candidate unifies the student's two open items, adaptive motion control and active localisation. Both become information-gathering actions whose value is measured in safe speed. Consider merging it with #1's and #2's candidates.

#### Candidate B — a RAPTOR-style tiny in-context adaptive controller on the 3pi+

**(a) Research question.** Can a recurrent policy of about 2k parameters, meta-trained over randomised differential-drive dynamics (mass, motor constants, friction, latency, battery), track velocity and heading on a real 3pi+ under payload and surface changes better than PID with feedforward and a recursive-least-squares adaptive controller, while running on the robot?

**(b) Why now.** RAPTOR, *Science Robotics* 2026: a 2,084-parameter recurrent policy that adapts zero-shot to 10 real quadrotors (32 g–2.4 kg), with deployment code for small flight controllers (Crazyflie, M5StampFly).

**(c) Nearest work.**
- RAPTOR and Learning to Fly in Seconds.
- Still open:
  - whether in-context adaptation transfers to wheeled robots, where traction limits and motor saturation dominate;
  - whether it beats simple adaptive baselines. RAPTOR's own baselines were not retrieved.

**(d) Strongest simple baselines.**
- PID with integral action and feedforward.
- Recursive-least-squares adaptation of the feedforward gain.
- Gain scheduling on battery voltage.

They could fail when payload changes the acceleration response in ways a fixed gain only corrects slowly, during transients. Surface changes impose traction limits that no controller can beat, which shrinks B's possible upside.

**(e) First-week kill test.**
- Measure a tuned PID's tracking RMSE on an aggressive velocity profile at three conditions: nominal, about +50% mass (mass not sourced), and a low-friction surface.
- **Kill if** the degradation is under ~15–20% (estimated threshold), or if recursive least squares closes the gap within a day.

**(f) Hardware and compute.**
- ESP32-C3 inference: a [64, 64] MLP runs at 150 Hz (**sourced**). RAPTOR-sized networks are **estimated** at 300–450 Hz, enough for a 100 Hz policy.
- The RTX 3060 is **estimated** to be enough to meta-train about 1,000 low-dimensional teachers. RAPTOR's README warns its teacher pre-training "will take a long time".
- The simulator needs an actuator model.

**(g) Headline figure and outcomes.**
- Headline: tracking RMSE under unseen changes for the tiny policy versus PID versus recursive least squares.
- Best realistic: 20–40% lower RMSE (estimated).
- Floor: "PID plus recursive least squares is enough on a 3pi+, and adaptation matters only above X payload." That is a boundary result.

**(h) Filter-by-filter verdict.**

| Filter | Verdict |
|---|---|
| 1 Robotics first | Pass |
| 2 Baseline named | Pass, but high "classical wins" risk |
| 3 Gap is real | Conditional on the kill test |
| 4 Method fits the problem | Pass, if the dynamics variation is real |
| 5 Nearest work read at methods level | Partial (abstract and README only) |
| 6 Feasible | Pass |
| 7 First-week test | Pass |
| 8 Useful if the method loses | Pass |

**(i) Skills built.** Sim-to-real, JAX RL and distillation, neural-network inference on a microcontroller, adaptive control.

**(j) Bristol people.** None found.

**Overlap note.** Merge this with #1's adaptive-control item.

#### Candidate C — adaptive learned flight control versus PID under physically injected gyro corruption

**(a) Research question.** When acoustic or ultrasonic resonance corrupts a micro-drone's MEMS gyro, does a RAPTOR-style in-context adaptive policy mistake the injected signal for a change in dynamics, and fail differently from the stock PID cascade and from a non-adaptive learned policy? And does a physically motivated pre-filter or consistency check restore it?

**(b) Why now.**
- RAPTOR (*Science Robotics* 2026) has Crazyflie and M5StampFly deployments.
- Injection attacks keep advancing (MARS 2025, HEMA 2025, Banshee 2026).
- One search found no study of learned controllers under injection.

**(c) Nearest work.**
- Un-Rocking Drones (NDSS 2023).
- MARS (2025).
- KITE, via Banshee's table.
- Still open: how learned or adaptive controllers respond (one search; not established novelty).

**(d) Strongest simple baseline.** The stock PID with a notch filter at the injected frequency. It fails if the injected frequency is unknown, or if the injection reaches the passband through aliasing or clipping. On the Crazyflie the gyro already has 116 Hz bandwidth and an 80 Hz software filter `[R]`, so in-band effects must come from those mechanisms.

**(e) First-week kill test.**
- Put the drone on a stand or tether, sweep tones (speaker or ultrasonic transducer), and log the gyro.
- **Kill if** no DC bias or low-frequency component appears above the noise floor at any frequency, at the sound level available.
- **Kill also if** RAPTOR firmware cannot be flown on the available drone within a week.

**(f) Hardware and compute.**
- **Crazyflie 2.1+:** £214.95–£221.54 UK (sourced; out of stock or restocking).
- **Crazyflie 2.1 Brushless:** £395.79–£461.54 (sourced).
- **Crazyradio 2.0 and M5StampFly:** prices not found.
- **Ask BRL for:** one or two Crazyflies with a Crazyradio and a Flow deck or motion capture, a cage or tether, and permission for high-volume acoustic tests. Alternatively, check M5StampFly pricing.
- Speaker and amplifier: **estimated** £20–£50. Hearing protection is required.

**(g) Headline figure and outcomes.**
- Headline: crash rate and attitude error under injection, for the adaptive policy versus the non-adaptive policy versus PID.
- Best realistic: identify a failure mode of in-context adaptation, with a mitigation.
- Floor: "The BMI088 is not susceptible at accessible sound levels", plus a characterisation sweep. That is still a boundary result.

**(h) Filter-by-filter verdict.**

| Filter | Verdict |
|---|---|
| 1 Robotics first | Pass |
| 2 Baseline named | Pass |
| 3 Gap is real | Pass |
| 4 Method fits the problem | Pass |
| 5 Nearest work read at methods level | Partial |
| 6 Feasible | **Fails on own budget** for a Crazyflie; conditional on BRL kit or a cheap M5StampFly |
| 7 First-week test | Pass |
| 8 Useful if the method loses | Pass |

**(i) Skills built.** Flight control, MEMS sensor physics, signal processing, hardware security testing. Relevant to GNC, defence and cyber roles.

**(j) Bristol people.** BRL aerial robotics; names not verified (#7).

#### Candidate D — a printed magnetic tactile bumper for pushing objects with the 3pi+

**(a) Research question.** Does an eFlesh- or ReSkin-style printed magnetic bumper, reporting contact location and force, let a 3pi+ push unknown objects to a goal more reliably than a passive V-shaped pusher or bump-switch feedback, using onboard sensing only?

**(b) Why now.**
- eFlesh (June 2025): under $5 of magnets, printed TPU, 0.5 mm contact localisation, and an open CAD-to-sensor tool.
- AnySkin is now sold commercially ($48).

**(c) Nearest work.**
- Bristol's Tactile Gym `object_push`: tactile pushing on an arm, with a sim-to-real pipeline `[R]`.
- eFlesh, AnySkin and ReSkin: sensing papers, not about mobile pushing.
- Mobile-robot tactile pushing was **not searched**, so overlap is unknown.

**(d) Strongest simple baselines.**
- A passive V-shaped or concave pusher, driven open-loop on dead reckoning.
- Reactive correction from the bump switches.
- An overhead-camera closed loop, used **only** as an upper bound in evaluation.

The baselines could fail because a V-pusher only stabilises objects that match its geometry. Contact location is the key unobserved variable in pushing (UNVERIFIED, prior knowledge of pushing mechanics).

**(e) First-week kill test.**
1. Run 20 open-loop pushes for each of 3 shapes with a printed V-pusher. **Kill if** ≥ 90% reach the goal.
2. Bench-test the skin with the motors at full power. **Kill if** interference from the motors and the onboard LIS3MDL area pushes contact-location error above ~5 mm (estimated threshold).

**(f) Hardware and compute.**
- A TPU-capable printer: **ask BRL**. eFlesh used a Bambu X1C.
- Magnets: under $5 (sourced).
- A ReSkin 5X or MLX90393 board: price not found. Note the MLX90393 low-stock warning `[R]`.
- The ESP32-C3 for I2C and the webcam for evaluation are owned.
- A two-layer, 128-unit MLP regressor, the kind eFlesh uses `[R]`, runs on a CPU.

**(g) Headline figure and outcomes.**
- Headline: push success rate and final pose error versus the V-pusher and bump-switch baselines, across shapes.
- Best realistic: touch closes most of the gap to camera-based pushing on irregular objects.
- Floor: "A V-pusher is enough for shapes X." That is a boundary result.

**(h) Filter-by-filter verdict.**

| Filter | Verdict |
|---|---|
| 1 Robotics first | Pass |
| 2 Baseline named | Pass |
| 3 Gap is real | Pass: onboard-only sensing; the camera is only an upper bound |
| 4 Method fits the problem | Pass |
| 5 Nearest work read at methods level | **Not met** |
| 6 Feasible | Pass, given printer access |
| 7 First-week test | Pass |
| 8 Useful if the method loses | Pass |

**(i) Skills built.** Sensor fabrication, magnetometer signal processing, contact mechanics, embedded I2C, small learned regressors.

**(j) Bristol people.** Nathan Lepora's tactile robotics group, verified this session through his papers listing, the DigiTac Bristol portal entry and the Tactile Gym repos (my.bristol.ac.uk project sites).

#### Ranking by evidence (not by fit to the student)
1. **A.** Plausible physics, precedents for filters expanding capability, owned hardware, ready-made filter tooling, and a crisp kill test. Weaknesses: novelty unverified, and learning plays a small part.
2. **B.** Strongest *method* evidence: a peer-reviewed 2026 paper with embedded code. It still has no evidence that a 3pi+ needs adaptation, and traction limits cap its upside.
3. **D.** Strong sensor evidence and the best supervision link. No evidence yet on mobile tactile pushing, and an interference risk.
4. **C.** The least crowded angle on learned controllers, but the attack physics is crowded and it is infeasible on budget without BRL kit or a cheap M5StampFly.

#### Tempting but fails (filter named)
1. **Learned CBF, DeepReach or a neural reach-avoid filter on the 3pi+.** Fails 4: the state is low-dimensional, the analytic braking CBF exists, and `hj_reachability` solves the grid case in JAX. Also fails 2.
2. **RL under a safety filter on the 3pi+, to test Oh et al. 2025.** Fails 2 and 3: a hand-coded controller already beat the learners on navigation.
3. **Robotarium-style multi-robot CBF on several 3pi+ robots.** Fails 5: the tooling is packaged (`barrier_certificates.py`) and the topic is established. Also fails 6 (several robots and global state needed). In #3's scope.
4. **Conformal safety around moving people or obstacles on the 3pi+.** Fails 6 (no exteroceptive sensing) and 3 (an overhead camera would hand it perfect state).
5. **An actively scanned single-beam time-of-flight sensor.** Fails 3: a fixed multizone time-of-flight sensor probably removes the need (not sourced).
6. **Pan-tilt or arm-mounted active vision.** Borderline on 6 (an SO-101 follower is $121.94 in parts, before VAT, shipping and printing) and at risk on 2 (an extra static webcam). In #4's scope.
7. **Visuo-tactile imitation learning on an SO-101 with AnySkin.** Fails 5 (crowded). In #4's scope.
8. **A Jacobian-field self-model of an SO-101.** Fails 6 on compute: the published 3D pipeline trained on 4×A100. Likely fails 2: calibration is probably enough.
9. **A self-model of the 3pi+.** Fails 4: it is system identification.
10. **A contact mic for terrain or slip on the 3pi+.** Fails 5 and 2.
11. **Whiskers on the 3pi+.** Fails 3: time-of-flight is the baseline.
12. **Replicating acoustic gyro injection against the 3pi+'s heading control.** Fails 5 (Tu 2018 lineage) and 1 (security is only the backdrop).
13. **Hall-encoder spoofing on the 3pi+ with an IMU consistency check.** Fails 5 (classical fault detection; the rejected localisation-spoofing family) and 3 (a contrived threat model).
14. **Replicating Learning to Fly in Seconds on a Crazyflie.** Fails 6 (budget) and 5 (a replication).

### Gaps
- **Filter 5 is outstanding for all four candidates.** Read these at methods level:
  - for **A:** Tearle 2021 and the Conformal Decision Theory experiments, then search "conformal safety margin mobile robot hardware" and "braking distance learned speed limit";
  - for **B:** RAPTOR's baselines and limitations in *Science Robotics*, then search "in-context adaptation wheeled robot";
  - for **C:** the Un-Rocking Drones, MARS and KITE primaries, then search "neural flight controller sensor injection";
  - for **D:** Lloyd & Lepora's tactile pushing and Tactile Gym `object_push` sim-to-real, then search "tactile bumper pushing mobile robot".
- **Bristol/BRL names from prior knowledge are UNVERIFIED** and should go to #7:
  - Martin Pearson: whiskers;
  - Arthur Richards: aerial robotics and control;
  - Efi Psomopoulou, Benjamin Ward-Cherrier and John Lloyd: tactile (Lloyd is a co-author of Tactile Gym 2.0 `[R]`);
  - Sabine Hauert: swarms;
  - Kerstin Eder: verification;
  - Helmut Hauser and Jonathan Rossiter: soft robotics;
  - Tom Richardson: UAVs.

  Confirmed this session: Lepora, and Tactile Gym co-authors Lin, Lloyd and Church, as authors only, not their current roles.
- **Not sourced:** 3pi+ top speed, mass and encoder counts per revolution; whether current 3pi+ production still uses the LSM6DS33; prices for the Crazyradio 2.0, Flow deck v2, M5StampFly, MLX90393 board, piezo mics and pan-tilt kits.
- **All "estimated" thresholds** in the kill tests are judgement calls, to be set after a pilot.
