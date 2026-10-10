# Safe learning for control, active perception, wildcard areas and physics-level robotics security (2024–2026): MSc candidate screening

**Read this first: what was and was not accessed (research date Sat 10 Oct 2026).**
- **WebFetch did not work.** Every page fetch failed with a DNS error (`getaddrinfo ENOTFOUND`). The domains tried were arxiv.org, lepora.com, store.bitcraze.io, pololu.com, alphaxiv.org and pmc.ncbi.nlm.nih.gov.
- **Other routes were blocked.** Direct HTTPS from the shell was refused by the egress policy (403), and GitHub file access is restricted to the session repo.
- **The search budget ran out.** The web-search budget for this turn is shared across agents, and it was used up after 21 of my searches. Planned searches on AV-ALOHA results, Eye-Robot/ViA, Hearing Touch, Robotarium, BRL people, Pololu 3pi+ specs, SO-101 prices and physical attacks on learned policies were therefore **not run**.
- **No source was read in full or at methods level.** Every finding below comes from search-engine summaries, which are at most abstract-level and sometimes include short fragments of page text.
  - `[S]` = search-summary level.
  - `[T]` = title or one line only.
  - A quote marked "(via search summary)" is text the search tool reported as coming from the page. Check it against the page before quoting it in the final report.
- **Filter 5 (nearest work read at methods level) is unmet for every candidate.** Where I write "no overlap found", it means one search found nothing. It does not mean the idea is novel.
- **Prior knowledge is kept separate.** Anything from my own background knowledge is marked "UNVERIFIED (prior knowledge)" and kept out of Cited Findings.

---

## 1. Safety filters on real low-cost robots (2024–2026): where has a filter EXPANDED capability, and what is the strongest simple alternative?

### Takeaway
There is evidence that a safety filter can let a robot do more than it otherwise safely could. In every case it works by letting a fast but unverified controller run inside a guard:
- **ETH Zürich, RA-L 2021.** A predictive safety filter let a deep-imitation-learned controller drive a miniature RC car aggressively while staying on the track.
- **ABS, RSS 2024.** A learned reach-avoid value network switches between an agile policy and a recovery policy. The paper sets this against earlier controllers that were held below 1.0 m/s for safety.

The 3pi+ is a different case. It has no range sensing, and along its heading it behaves almost like a double integrator, so the analytic braking-distance rule is already the barrier function. A filter only earns its place through how it handles uncertainty: braking that depends on surface and battery, dead-reckoning drift, and latency. That argues for calibrated (conformal) margins. Learned CBFs and DeepReach are not justified here.

### Cited Findings
- **Predictive safety filter for racing.** Tearle, Wabersich, Carron and Zeilinger, "A predictive safety filter for learning-based racing control", *IEEE RA-L* 6 (2021) 7635–7642, ETH Zürich. `[S]` — [arXiv 2102.11907](https://arxiv.org/pdf/2102.11907); [Semantic Scholar](https://www.semanticscholar.org/paper/A-Predictive-Safety-Filter-for-Learning-Based-Tearle-Wabersich/14a0f4a0f3b54b9c2db20df14b78afe1bd86df44)
  - The filter keeps the vehicle within the track for any potentially unsafe input, including learning-based ones.
  - "the safe set is extended in real-time through predictive control backup trajectories" (via search summary).
  - It was shown on a physical miniature RC car, with human-in-the-loop racing and with deep imitation learning.
  - The authors note that performance improves with a more accurate model, a longer horizon or a larger terminal set.
  - No lap times or success rates were retrieved.
- **Earlier filter from the same group.** Wabersich & Zeilinger, predictive safety filter for learning-based control of constrained nonlinear systems. `[T]` — [arXiv 1812.05506](https://arxiv.org/pdf/1812.05506)
- **ABS: Agile But Safe** (He, Zhang et al., CMU/ETH), **RSS 2024**. `[S]` — [RSS 2024 proceedings p059](https://roboticsproceedings.org/rss20/p059.html); [arXiv 2401.17583](https://arxiv.org/html/2401.17583v3); [alphaxiv summary](https://alphaxiv.org/overview/2401.17583v3)
  - An agile policy is paired with a recovery policy.
  - A learned reach-avoid value network, inspired by Hamilton–Jacobi reachability, decides when to switch and guides the recovery policy. It is trained with a discounted reach-avoid Bellman equation on data from the agile policy.
  - Depth images are converted to ray distances; everything is trained in simulation and run onboard.
  - The paper's framing: "Existing studies either develop conservative controllers (< 1.0 m/s) to ensure safety, or focus on agility without considering potentially fatal collisions" (via search summary).
  - A peak of about 3.1 m/s on a Unitree Go1 comes only from a secondary summary, so it is **unverified**.
- **Tunable real-time safety filters via set-based CBFs** (arXiv 2507.07805, PREPRINT, Jul 2025). `[S]` — [arXiv 2507.07805](https://arxiv.org/pdf/2507.07805)
  - A single class-K parameter sets how aggressively the filter intervenes.
  - Hardware test: a permanent-magnet synchronous motor drive, with an explicit QP inside a **150 µs** sampling window and a **28.04 µs worst-case execution time** (sourced).
  - In simulation, a lower parameter gives early, heavily damped intervention and a higher one allows a more aggressive approach to the boundary.
- **Neural CBFs for safe navigation** (arXiv 2407.19907, PREPRINT). `[S]` — [arXiv 2407.19907](https://arxiv.org/pdf/2407.19907)
  - Hardware tests indoors and outdoors at 20 Hz.
  - The filter changed an unsafe reference command to stop the robot before an obstacle. This shows restriction, not expansion.
- **"How to Train Your Latent CBF"** (arXiv 2511.18606, PREPRINT, Nov 2025). `[S]` — [arXiv 2511.18606](https://arxiv.org/pdf/2511.18606)
  - The CBF with a Gaussian process gives a "45% decrease in action override magnitude relative to the corresponding LR filter" (sourced). The setting was not confirmed.
- **Survey.** Hsu, Hu & Fisac, "The Safety Filter: A Unified View of Safety-Critical Control in Autonomous Systems", *Annual Review of Control, Robotics, and Autonomous Systems* 7(1):47–72 (2024). It offers a unified framework for comparing and combining filters. `[S]` — [Princeton portal](https://collaborate.princeton.edu/en/publications/the-safety-filter-a-unified-view-of-safety-critical-control-in-au/); [arXiv 2309.05837](https://arxiv.org/abs/2309.05837v1)
- **Optimal RL under a filter.** Oh, Nguyen, Hu & Fisac, "Provably Optimal Reinforcement Learning Under Safety Filtering" (arXiv 2510.18082, **Oct 2025**; listed in the IASEAI proceedings on ojs.aaai.org, year not confirmed). `[S]` — [arXiv 2510.18082](https://arxiv.org/pdf/2510.18082v2); [IASEAI listing](https://ojs.aaai.org/index.php/IASEAI/article/view/43046)
  - Claim: a least-restrictive filter, which prevents every violation while intervening as little as possible, lets an RL agent converge to the optimal safety-constrained policy.
- **Conformal Decision Theory.** Lekeufack, Angelopoulos, Bajcsy, Jordan & Malik, **ICRA 2024**, pp. 11668–11675. `[S]` — [arXiv 2310.05921](https://arxiv.org/pdf/2310.05921). The venue and pages come from a later paper's reference list as reported by the search tool. That paper is probably [arXiv 2509.25124](https://arxiv.org/pdf/2509.25124), but the attribution is uncertain.
  - Three test domains:
    - robot navigation around people, using the Stanford Drone Dataset, where the method tunes the reward online "to be safe but efficient";
    - manufacturing, where it "directly calibrates the speed of the conveyor belt under a robot to achieve high-throughput and successful robot grasps" (via search summary);
    - stock trading.
  - No physical-hardware experiment was confirmed.
- **Other 2024–2026 leads, title only and not assessed** `[T]`:
  - closed-form dual-barrier CBF on incrementally built occupancy grids for holonomic robots ([arXiv 2605.05182](https://arxiv.org/html/2605.05182v2), May 2026, PREPRINT);
  - input-to-state-safe CBF whole-body control for humanoids ([arXiv 2605.25546](https://arxiv.org/pdf/2605.25546), PREPRINT);
  - policy CBFs built at runtime ([arXiv 2410.11157](https://arxiv.org/html/2410.11157));
  - "Drive Fast, Learn Faster: On-Board RL for High Performance Autonomous Racing" ([arXiv 2505.07321](https://arxiv.org/pdf/2505.07321), PREPRINT);
  - "Safety with Agency", a human-centred filter for AI-assisted motorsport ([arXiv 2504.11717](https://arxiv.org/pdf/2504.11717), PREPRINT);
  - conformalised semantic maps for safe planning ([arXiv 2509.25124](https://arxiv.org/pdf/2509.25124), Sep 2025, PREPRINT);
  - conformal policy learning under distribution shift ([arXiv 2311.01457](https://arxiv.org/pdf/2311.01457));
  - Ames et al. CBF review ([arXiv 1903.11199](https://arxiv.org/pdf/1903.11199));
  - "How to Train Your Neural CBF" ([arXiv 2310.15478](https://arxiv.org/pdf/2310.15478)).

### Inferences
- **"Expansion" is relative.** In both hardware precedents (Tearle 2021; ABS 2024), the gain is measured against the conservative controller you would otherwise deploy: the filter is what makes it acceptable to run a fast, unverified policy. Neither was on a robot as sensor-poor as the 3pi+.
- **On the 3pi+, the simple rule is the CBF.** Hazards are either in a map or sensed only on contact (line sensors at an edge, bump sensors at a wall). For a near double-integrator, the state-dependent braking rule `v ≤ sqrt(2·a_min·(d − m)) − a_min·τ` is the barrier condition (standard kinematics; inference).
  - That makes learned CBFs and DeepReach, which targets high-dimensional reachability, unjustified by the problem structure (filter 4).
  - A low-dimensional Hamilton–Jacobi problem can be solved on a grid (UNVERIFIED, prior knowledge).
- **The one framing worth carrying forward:** a filter changes what a 3pi+ can do only if it replaces a global speed cap with a limit that depends on state and conditions, and whose margin is recalibrated online for surface, battery and dead-reckoning drift.
  - Conformal Decision Theory's idea of calibrating a speed knob for throughput under a risk bound maps directly onto this. This is **Candidate A**.
- **Embedded cost is small.** Explicit-QP CBF filters run in tens of microseconds on an embedded drive (28.04 µs sourced). A filter on an ESP32-C3 at 100 Hz–1 kHz is therefore likely cheap (estimated; different CPU).
- **A theory–hardware gap exists, but only matters if RL is needed.** Oh et al. (Oct 2025) assume a perfect, least-restrictive filter. On cheap hardware the filter's model is wrong (slip, latency), so "how much does an imperfect filter bias what RL learns?" is open. It only matters if the task actually needs RL. In the student's checkpoint test it did not: a hand-coded controller won.

### Gaps
- **Missing numbers.** No lap-time or speed figures were retrieved for Tearle 2021, and ABS's ~3.1 m/s is secondary only.
- **No cheap-robot example found.** I found no 2024–2026 paper where a filter expanded capability on a robot as cheap as a 3pi+. The search was incomplete: targeted searches for Robotarium/GRITSbot, F1TENTH and Duckietown were never run.
- **Papers named in the brief were not checked this session:** Wang, Ames & Egerstedt (T-RO 2017), Robey et al. 2020, Dawson et al. 2023, DeepReach, Lindemann et al. 2023, and shielding papers.
- **Hardware in Conformal Decision Theory not confirmed.**
- **No head-to-head comparison found** between a CBF filter and a plain braking-distance rule on any robot.

---

## 2. Active perception beyond localisation: is any of it feasible with a webcam plus £200?

### Takeaway
The 2024–25 active-vision manipulation results are out of reach on a webcam and £200 without an arm:
- **AV-ALOHA** (PREPRINT) uses a whole 7-DoF arm just to carry the camera.
- **Observe Then Act** was evaluated only in simulation (RLBench).

The strongest cheap baseline is one or two extra static webcams, and that is cheap enough to kill most active-vision claims on this budget. I could not retrieve AV-ALOHA's comparison against static cameras, so I cannot say where active vision actually won.

On the 3pi+, the credible form of "active perception beyond localisation" is active system identification: a short probing manoeuvre to learn the braking on the current surface before committing to speed. This is folded into Candidate A.

### Cited Findings
- **AV-ALOHA.** "Active Vision Might Be All You Need" — Chuang, Lee, Gao, Naddaf-Sh & Soltani (UC Berkeley / UC Davis), arXiv 2409.17435 v1/v2, **PREPRINT** (no venue found). `[S]` — [arXiv 2409.17435](https://arxiv.org/pdf/2409.17435); [v2 html](https://arxiv.org/html/2409.17435v2)
  - Extends ALOHA 2 with an extra 7-DoF arm that carries only a stereo camera and is tasked with finding the best viewpoint.
  - Released as an open-source, "low-cost" teleoperation system.
- **Observe Then Act.** Guokang Wang et al., arXiv 2409.14891 v1–v3. `[S]` — [arXiv 2409.14891](https://arxiv.org/pdf/2409.14891); [v3](https://arxiv.org/html/2409.14891v3)
  - Chains a camera next-best-view policy with a gripper next-best-pose policy.
  - Trained with few-shot RL inside a sensor–motor coordination framework.
  - Evaluated on 8 viewpoint-constrained RLBench tasks (simulation), with "consistent gains over baselines".
  - A summary says v3 was accepted in Jan 2025 at an IEEE venue; which one is unconfirmed, so treat it as PREPRINT.
- **UC Davis blog** on the "Eye, Robot" active-vision work exists. `[T]` — [UC Davis blog](https://www.ucdavis.edu/index%2ephp/blog/eye-robot-researchers-give-machines-new-perspective-active-vision)

### Inferences
- **Feasibility on budget.**
  - Perception-only active vision, such as a webcam on a pan-tilt servo bracket, fits the budget (price not sourced; estimated at tens of pounds).
  - Active vision coupled to manipulation needs an arm, which the student does not own. The SO-101 is covered by the imitation-learning / low-cost-arms researcher (#4).
  - AV-ALOHA's two task arms plus a camera arm are far beyond the budget.
- **Strongest baseline and kill test.** The baseline is more static cameras, or a wrist camera. Kill the idea if a second static webcam (£ not sourced) matches the active camera's success on the chosen occluded task (filters 2 and 3).
- **On the 3pi+ (no camera).** Active scanning with a single-beam time-of-flight sensor on a servo creates a manufactured gap if a fixed multizone time-of-flight sensor would do the job (spec and price not sourced). The better framing is information for speed: spend a manoeuvre to identify friction or position, and unlock a higher safe speed (Candidate A).
- **Ranking.** Active vision ranks below Candidates A–D on the evidence available.

### Gaps
- AV-ALOHA's quantitative results (which tasks active vision helped, and the static-camera comparison) were not retrieved.
- Vision in Action (ViA, 2025), Eye-Robot (Kerr et al., 2025) and next-best-view-for-manipulation work from 2025–26 were not checked (search budget used up).
- Prices for a pan-tilt kit, a second webcam and an SO-101 were not sourced.

---

## 3. Wildcard (a): tactile sensing — the TacTip line from Lepora's group at BRL, magnetic skins, and fit to a cheap arm

### Takeaway
Tactile sensing is the best-evidenced wildcard and has the strongest supervision link at Bristol:
- Lepora's TacTip, DigiTac and Tactile Gym line, plus his 2025 reviews in T-RO and IJRR.
- Magnetic skins have become very cheap:
  - **eFlesh:** under $5 of magnets plus a hobby 3D printer and a magnetometer board;
  - **AnySkin:** commercial skin listed at $48, with an SO-101 mount variant at $128.

The obstacle is that the student owns no arm. The cheapest tactile question on hardware the student already has is a printed magnetic tactile bumper on the 3pi+ (Candidate D).

### Cited Findings
- **Lepora's 2025 output**, from his papers page as shown in the search summary. The page itself could not be opened. `[S]` — [lepora.com/papers](https://lepora.com/papers/)
  - "Tactile Robotics: Past and Future" (*IJRR* 2025);
  - "Tactile Robotics: An Outlook" (*IEEE T-RO* 2025, with Luo, Yuan, Althoefer, Cheng and Dahiya);
  - "Shear-based Grasp Control for Multi-fingered Underactuated Tactile Robotic Hands" (*T-RO* 2025, Ford et al.);
  - StereoTacTip (*IEEE/ASME T-Mech* 2025);
  - "Educational SoftHand-A", built from LEGO MINDSTORMS (*IROS* 2025);
  - "NeuralTouch", on sim-to-real tactile control (in submission).
- **DigiTac and Tactile Gym 2.0.** `[S]` — [arXiv 2206.13657](https://arxiv.org/abs/2206.13657); [Bristol research portal](https://research-information.bris.ac.uk/en/publications/digitac-a-digit-tactip-hybrid-tactile-sensor-for-comparing-low-co/); [Tactile Gym 2.0, arXiv 2207.10763](https://arxiv.org/pdf/2207.10763)
  - DigiTac adapts the DIGIT sensor with a 3D-printed TacTip-style skin.
  - Its test rig uses a desktop Dobot MG400 arm at about **£2k** (sourced via summary).
  - Tactile Gym 2.0 lists the MG400 at about **$2.7k**, with 0.05 mm accuracy, 440 mm reach and 0.75 kg payload, against about **$45k** for a UR5 set-up (sourced via summary).
- **Pose and shear-based tactile servoing** (Lepora group). `[T]` — [arXiv 2312.08411](https://arxiv.org/pdf/2312.08411)
- **eFlesh.** Pattabiraman, Huang, Panozzo, Zorin, Pinto & Bhirangi (NYU), arXiv 2506.09994, **PREPRINT, Jun 2025**. `[S]` — [arXiv 2506.09994](https://arxiv.org/html/2506.09994v1)
  - Built with a hobby 3D printer (e.g., Bambu X1 Carbon) and TPU filament; magnets (**< $5**, sourced) are press-fitted during a paused print, and an off-the-shelf magnetometer board reads the field.
  - Reported accuracy: contact localisation RMSE **0.5 mm**; force RMSE **0.27 N** normal and **0.12 N** shear (sourced).
  - An open-source tool converts CAD (OBJ/STL) into printable sensor files.
- **AnySkin** (NYU, 2024; venue not confirmed this session). `[S]` — [WowRobo product page](https://shop.wowrobo.com/products/enhanced-anyskin-premium-crafted-editionwowskin); [reseller listing](https://www.voltsprays.com/product/wowrobo-robotics-enhanced-anyskin-premium-crafted-edition-wowskin/); [SO-ARM100 add-on list](https://github.com/adityachandrath/SO-ARM100); [ResearchGate](https://www.researchgate.net/publication/383985119_AnySkin_Plug-and-play_Skin_Sensing_for_Robotic_Touch); [TechXplore, Oct 2024](https://techxplore.com/news/2024-10-sensor-large-scale-robotics-applications.html)
  - WowRobo's "Enhanced AnySkin" page lists a **$48.00** "touch skin only" variant and a **$128.00** "SO-101 structural part" variant (sourced; mounting parts are sold separately).
  - A reseller lists **$49.92** (marked down from $99.84) without saying which variant, which **conflicts** with the WowRobo prices.
  - A GitHub add-on list for the SO-100/SO-101 includes AnySkin for the gripper.
- **"Seeing Touch from Motion: A Unified Modality-Aware Visuo-Tactile Policy with Tactile Motion Correlation"** (arXiv 2606.29941, **Jun 2026**, PREPRINT). It surfaced in a Lepora-related search; authorship not confirmed. `[T]` — [arXiv 2606.29941](https://arxiv.org/pdf/2606.29941)

### Inferences
- **Can touch be fitted to a £100–£200 arm such as an SO-101?** Yes in principle: WowRobo sells an SO-101 AnySkin mount. But the student owns no arm, and an SO-101 plus AnySkin probably uses up the £200 (SO-101 price not sourced; see #4).
- **Imitation learning with touch on the SO-101 is crowded.** There is a stream of 2025–26 visuo-tactile and multimodal policy papers, for example arXiv 2606.29941 and Audio-VLA. It fails filter 5 without a sharper question.
- **On the student's own hardware:** a printed magnetic skin as a bumper on the 3pi+ would give contact location and force while pushing objects (Candidate D).
  - Possible Bristol link: tactile pushing by Lloyd & Lepora on an arm with a TacTip (UNVERIFIED, prior knowledge).
- **Platform-specific risk:** the 3pi+'s motor magnets and magnetic encoders sit near any magnetometer-based skin (encoder type UNVERIFIED). Interference has to be measured in week 1.

### Gaps
- Not checked: AnyRotate (CoRL 2024), Bi-Touch, whether a "Tactile Gym 3" exists, TacTip build cost, ReSkin cost (access and search budget).
- Magnetometer breakout prices were not sourced.
- Whether Lepora's group has MSc projects or arms to lend is for #7 (BRL staff and facilities).

---

## 4. Wildcard (b): acoustic and vibration sensing

### Takeaway
Contact-microphone sensing is well evidenced for manipulation and cheap:
- SonicSense and ManiWAV at CoRL 2024;
- VibeCheck (April 2025) and Audio-VLA (November 2025);
- the SonicSense hand cost "just over $200", according to Duke via press reports.

On the student's wheeled robot, though, the classical baseline is strong: vibration and slip signatures from the IMU and encoders. Acoustic terrain classification is also an old topic. I found no acoustic candidate that survives on the student's own hardware.

### Cited Findings
- **SonicSense** (Duke; arXiv 2406.17932; **CoRL 2024**). `[S]` — [arXiv 2406.17932](https://arxiv.org/pdf/2406.17932); [The Robot Report](https://www.therobotreport.com/sonicsense-lets-robots-perceive-objects-via-in-hand-acoustic-vibration/); [Unite.AI](https://www.unite.ai/sonicsense-gives-robots-human-like-sensing-abilities-through-acoustic-vibrations/)
  - A four-finger hand with a contact microphone in each fingertip; it taps, grasps and shakes objects, and the contact mics reject ambient noise.
  - Build cost "just over $200", from secondary press reporting Duke.
  - A later paper describes its exploration policy as heuristic.
- **ManiWAV** (arXiv 2406.19464; **CoRL 2024**). `[S]` — [arXiv 2406.19464](https://arxiv.org/html/2406.19464)
  - An "ear-in-hand" handheld gripper with a piezo contact mic in one finger.
  - Audio goes through a GoPro's mic port, synchronised with video, so demonstrations can be collected in the wild for policy learning.
- **VibeCheck** — active acoustic tactile sensing for contact-rich manipulation (arXiv 2504.15535, **PREPRINT, Apr 2025**). `[T]` — [arXiv 2504.15535](https://arxiv.org/pdf/2504.15535)
- **Audio-VLA** (arXiv 2511.09958, **PREPRINT, Nov 2025**). `[S]` — [arXiv 2511.09958](https://arxiv.org/pdf/2511.09958)
  - Adds contact-audio perception to a vision-language-action model.
  - Notes that Hearing Touch and ManiWAV both use piezo contact mics.

### Inferences
- **Why now:** cheap piezo contact mics plus the 2025 multimodal policies. The momentum is in manipulation and VLAs, which are crowded and partly in #5's scope.
- **3pi+ variant: a contact mic for slip, terrain or contact detection.**
  - Strongest baseline: the IMU at its maximum data rate plus encoder–gyro slip checks.
  - Kill test: if IMU and encoder features alone classify surfaces and slip at ≥ ~95% (threshold estimated), the mic adds nothing.
  - Verdict: likely killed. The area is also classical (e.g., Valada & Burgard, IJRR 2017; UNVERIFIED, prior knowledge).
- **No Bristol connection** was found this session.

### Gaps
- Hearing Touch's venue and method were not checked.
- Piezo contact-mic prices were not sourced.
- No BRL acoustic-sensing group was identified (search budget used up).

---

## 5. Wildcard (c): robot self-modelling

### Takeaway
There is strong 2025 evidence that a self-model learned from a single camera can control diverse manipulators, including cheap ones:
- Jacobian fields (*Nature*, July 2025);
- Hu, Lin & Lipson (*Nature Machine Intelligence*, 2025).

For this student it would need an arm, and it would have to beat classical camera-based kinematic calibration. On the 3pi+, "self-modelling" reduces to system identification, where grey-box models are the right tool (filter 4).

### Cited Findings
- **Jacobian fields.** Li, Sitzmann et al. (MIT), "Controlling diverse robots by inferring Jacobian fields with deep networks", *Nature* 643(8070):89–95 (online 25 Jun 2025; DOI 10.1038/s41586-025-09170-0). `[S]` — [PMC12221969](https://www.ncbi.nlm.nih.gov/pmc/articles/PMC12221969/); [RePEc](https://ideas.repec.org/a/nat/nature/v643y2025i8070d10.1038_s41586-025-09170-0.html); [arXiv 2407.08722](https://arxiv.org/pdf/2407.08722)
  - Networks map video of a robot to its visuomotor Jacobian field, and control then needs only one camera.
  - It "makes no assumptions about the robots' materials, actuation or sensing, and is trained without expert intervention".
  - Tested on manipulators "that vary in actuation, materials, fabrication and cost".
  - The preprint says training data came from **12 consumer RGB-D cameras**, so "single camera" applies only to control after training.
- **Self-simulation from video.** Hu, Lin & Lipson, "Teaching Robots to Build Simulations of Themselves", *Nature Machine Intelligence* 2025 (arXiv 2311.12151). `[S]` — [arXiv 2311.12151](https://arxiv.org/pdf/2311.12151); [Columbia news](https://engineering.columbia.edu/about/news/robots-learn-how-move-watching-themselves-0)
  - Builds a kinematic self-model from a short video taken by one ordinary camera, then uses it for motion planning, anomaly detection and damage recovery.
  - Columbia's news item says earlier self-modelling work used a 4-DoF arm, about 1,000 random trajectories, under 35 hours of training, and reached about 4 cm accuracy.
  - One secondary report named *Science Robotics* as the venue, which conflicts with the arXiv journal reference. I take *NMI* as correct.

### Inferences
- **Arm-based candidate:** "does a vision-learned self-model beat ArUco-based kinematic calibration on a cheap servo arm with backlash and compliance?" Classical calibration is probably enough unless the task tolerances are tight. Provisional fail on filters 2 and 6 (no arm owned).
- **3pi+ variant:** a self-model of the 3pi+ is just system identification, so it fails filter 4. It is also covered by #1 (adaptive control).
- **No Bristol link** was found this session.

### Gaps
- Not retrieved: the quantitative accuracy and full robot list for Jacobian fields, the cost of the robots used, and the method details of Chen et al. (*Science Robotics* 2022).

---

## 6. Wildcard (d): RL-trained low-level control on microcontrollers and Crazyflie-class drones

### Takeaway
"Train in seconds, deploy on a microcontroller" has moved on fast:
- **2024:** a single platform — *Learning to Fly in Seconds* (RA-L 2024) trained a Crazyflie policy in 18 s on a laptop.
- **2025–26:** a 2,084-parameter recurrent "foundation" policy that adapts in-context to 10 real quadrotors from 32 g to 2.4 kg (RAPTOR, PREPRINT, Sep 2025, v2 Apr 2026).

Crazyflie hardware is over the £200 budget even before buying a radio:

| Model | UK price (sourced) | Stock at search time |
|---|---|---|
| Crazyflie 2.1+ | £214.95–£221.54 | sold out / restocking |
| Crazyflie 2.1 Brushless | £395.79–£461.54 | sold out / restocking |

So Crazyflie work needs BRL equipment. The on-budget version is a RAPTOR-style tiny adaptive policy on the 3pi+'s ESP32-C3. There, a classical baseline (PID with feedforward, or a recursive-least-squares adaptive controller) may make learning pointless.

### Cited Findings
- **Learning to Fly in Seconds** — Eschmann, Albani & Loianno, **RA-L 2024**, arXiv 2311.13081. `[S]` — [arXiv 2311.13081](https://arxiv.org/abs/2311.13081); [NSF PAR copy](https://par.nsf.gov/servlets/purl/10557794); [GitHub repo](https://github.com/arplaboratory/learning-to-fly); [ResearchGate](https://www.researchgate.net/publication/380286884_Learning_to_Fly_in_Seconds)
  - Sim-to-real transfer "for direct RPM control after only **18 seconds** of training on a consumer-grade laptop" (via search summary).
  - Repo limitations: take-off "has a big sim2real gap (ground-effect, z-ranger/optical-flow estimation behavior) and a large setpoint error", so take off with the stock controller and switch mid-air. The policy also regulates global yaw to 0.
  - A ResearchGate excerpt (version unclear) says motor delays are not modelled, "which can lead to instabilities in deployment". It also reports over 200 flights with under 0.05 m RMSE at up to 2.0 m/s. **Unverified.**
- **RLtools** (arXiv 2306.03530). `[S]` — [arXiv 2306.03530](https://arxiv.org/pdf/2306.03530)
  - Claims "the first-ever training of a deep RL algorithm directly on a microcontroller (the Teensy 4.1)".
  - Teensy 4.1 inference: **64 µs** generic, **45 µs** with the vendor DSP library, **41 µs** optimised (≈24.3 kHz) (sourced).
  - **3.4 kHz** inference on the Crazyflie (sourced).
  - No ESP32 figures were found.
- **RAPTOR** — Eschmann, Albani & Loianno (UC Berkeley / TII), arXiv 2509.11481, **v2 6 Apr 2026, PREPRINT**. `[S]` — [arXiv 2509.11481](https://arxiv.org/abs/2509.11481v2); [alphaxiv overview](https://www.alphaxiv.org/overview/2509.11481); [chatpaper abstract](https://chatpaper.com/de/paper/188273); [SOTAVerified](https://sotaverified.org/papers/250911481)
  - "A tiny, three-layer policy with only 2084 parameters is sufficient for zero-shot adaptation to a wide variety of platforms."
  - In-context adaptation comes from recurrence in the hidden layer.
  - Training is meta-imitation: sample 1,000 quadrotors, train an RL teacher for each, then distil them into one adaptive student.
  - Tested on 10 real quadrotors from 32 g to 2.4 kg, differing in motor, frame, propeller and flight controller. Tests covered trajectory tracking, indoor and outdoor flight, wind, poking and propeller changes.
  - Code is listed as available; the project page is raptor.rl.tools.
- **Prices**, retrieved via search on 10 Oct 2026 (page dates unknown). `[S]`
  - Crazyflie 2.1+: **$240.00** at the Bitcraze store ([store](https://store.bitcraze.io/products/crazyflie-2-1-plus)); **£221.54** at RobotShop UK, "re-stocking soon" ([RobotShop UK](https://uk.robotshop.com/products/bitcraze-crazyflie-21)); **£214.95** at Unmanned Tech Shop UK, sold out ([UTS](https://www.unmannedtechshop.co.uk/products/crazyflie-2-1-drone-hacker-development-platform-kit)). The UTS page URL says "2.1 kit", so it is ambiguous whether this is the 2.1 or the 2.1+.
  - Crazyflie 2.1 Brushless: **£461.54** at RobotShop UK, restocking ([RobotShop UK](https://uk.robotshop.com/products/crazyflie-21-brushless-version-open-source-micro-quadcopter-drone)); **£395.79** at Unmanned Tech Shop, sold out ([UTS](https://www.unmannedtechshop.co.uk/de/products/crazyflie-2-1-brushless-hacker-drone)).
  - A Lighthouse swarm bundle with Brushless drones is listed at £5,495; whether that listing is current is unclear ([UTS listing](https://www.unmannedtechshop.co.uk/collections/all?page=30)).
- **On-device training on ESP32-class chips.** The "webmcu-ai" series (arXiv 2604.23012 / 2604.22834, **PREPRINT, Apr 2026**) reports a CNN on an ESP32-S3 that trains in about **9 min per run** and infers at **159 ms per frame** (6.3 FPS). Which of the two papers holds the figure is uncertain. It shows on-MCU learning is practical on ESP32-class chips, though not RL. `[S]` — [arXiv 2604.23012](https://arxiv.org/pdf/2604.23012); [arXiv 2604.22834](https://arxiv.org/pdf/2604.22834)
- **System identification of quadrotors with motor delays** (arXiv 2404.07837). `[T]` — [arXiv 2404.07837](https://arxiv.org/pdf/2404.07837)

### Inferences
- **Why now.** RAPTOR (Sep 2025 / Apr 2026) is the first evidence I found that a ~2k-parameter recurrent policy adapts across very different real platforms. That makes "a tiny in-context adaptive controller on a cheap ground robot" a sharp, testable transfer. It lands directly on the student's open item, adaptive motion control under wheel, surface or payload changes (#1's scope). This is **Candidate B**.
- **Compute fit (estimated).**
  - 2,084 float32 parameters ≈ 8.3 KB (arithmetic).
  - That fits easily in the ESP32-C3's RAM (~400 KB) and the Pico 2 W's (UNVERIFIED, prior knowledge).
  - The ATmega32U4's ~2.5 KB RAM cannot hold the weights (UNVERIFIED, prior knowledge).
- **The simulator must be upgraded.** It needs actuator limits, motor dynamics, latency and battery sag. The student's earlier failure was partly caused by having no acceleration limits.
- **Baseline risk.** PID with integral action already absorbs a payload change in steady state. Adaptation only pays if transients, saturation or slip degrade tracking materially, which is exactly what the kill test checks.
- **Crazyflie path.** Ask BRL's aerial robotics lab for:
  - a Crazyflie 2.1+ or Brushless;
  - a Crazyradio 2.0;
  - a Flow deck v2, or Lighthouse or motion-capture coverage.

  This also enables **Candidate C**.

### Gaps
- Crazyradio 2.0 and Flow deck v2 prices were not found.
- Not retrieved for RAPTOR: training compute and time, baseline comparisons, and its exact Crazyflie firmware path.
- Not verified this session: the Crazyflie's IMU part (BMI088 per prior knowledge) and whether the ESP32-C3 has a hardware FPU (prior knowledge: it does not).

---

## 7. Wildcard (e): other cheap, evidenced areas — whiskers, and printed skins on mobile robots

### Takeaway
TU Delft's barometer-based whiskers for tiny drones (October 2025; reportedly *Nature Communications*, September 2026) show tactile navigation running on a microcontroller with 192 KB of RAM. On a ground robot, though, a cheap time-of-flight ranger is a strong baseline, so whiskers on the 3pi+ fail the "real gap" filter except in narrow cases such as transparent obstacles. The more defensible cheap tactile idea for the 3pi+ is the magnetic skin bumper (Candidate D).

### Cited Findings
- **Whisker-based tactile flight.** Ye, de Croon & Hamaza (TU Delft), "Whisker-based Tactile Flight for Tiny Drones", arXiv 2510.03119 (**3 Oct 2025**). `[S]` — [arXiv 2510.03119](https://arxiv.org/abs/2510.03119v1); [ScienceBlog](https://scienceblog.com/?p=601630); [Dronezine](https://www.dronezine.it/?p=460083); [The Debrief](https://thedebrief.org/bio-inspired-whiskers-let-tiny-drones-navigate-total-darkness-by-touch/)
  - A **3.2-gram** apparatus of barometer-based whisker sensors.
  - Tactile depth estimation to "**sub-6 mm**" accuracy, running fully onboard a **192-KB RAM** microcontroller (sourced).
  - Press reports say it was published in *Nature Communications* on **18 Sep 2026**, DOI 10.1038/s41467-026-77366-7 (**unverified**).
  - Earlier work: "A Biomorphic Whisker Sensor for Aerial Tactile Applications", ICRA 2024.
- **Whiskers with a spiking network.** Zhang et al. pair a bio-inspired whisker with a spiking neural network and a central pattern generator, and report navigation including in total darkness. *Concurrency and Computation: Practice and Experience*, 2026 (first published Dec 2025). `[S]` — [Wiley](https://onlinelibrary.wiley.com/doi/10.1002/cpe.70467)
- **Other whisker preprints** `[T]`:
  - magnetically actuated vision-based whisker array ([arXiv 2503.00133](https://arxiv.org/pdf/2503.00133));
  - whisker-based active tactile perception for contour reconstruction ([arXiv 2507.23305](https://arxiv.org/pdf/2507.23305));
  - whisker sim-to-real for underwater contact tracking ([arXiv 2410.14005](https://arxiv.org/html/2410.14005v1)).
- **Classic reference.** Jung & Zelinsky's whisker-based mobile robot navigation (1996), designed for high-speed, close wall-following. `[S]` — [IEEE Xplore](https://ieeexplore.ieee.org/document/570842/)

### Inferences
- **Whiskers on the 3pi+** fail filter 3: a time-of-flight sensor is the baseline, and it works in darkness too. They survive only if transparent or specular obstacles are the explicit target. They go on the "tempting but fails" list.
- **Bristol has whisker heritage** (Pearson et al. at BRL; UNVERIFIED, prior knowledge). It could make a whisker variant supervisable if a real gap were found.

### Gaps
- BRL whisker staff and current activity were not verified (search budget used up; see #7).
- I found no 2024–2026 mobile-robot tactile-bumper pushing paper. **This search was never run**, so the absence is not evidence of a gap.

---

## 8. Robotics–security overlaps where physics or sensing is the subject: how crowded?

### Takeaway
Injecting signals into MEMS inertial sensors, acoustically and now by laser, is a crowded and maturing security area:
- the 2015–2018 foundations (Son et al., WALNUT, Tu et al.);
- NDSS 2023 recovery work;
- 2025 defences (MARS);
- a 2025 hands-on teaching platform (HEMA);
- 2025 laser injection (17 of 25 devices);
- a July 2026 attack on gimbal-stabilised tracking (Banshee).

The only angle where physics-level security strengthens a robotics question, rather than just being the backdrop, is how learned or adaptive low-level controllers respond to physically injected sensor corruption. One search found no paper on this, which is not evidence of novelty. It also needs a Crazyflie-class platform that the budget cannot buy (Candidate C).

### Cited Findings
- **Un-Rocking Drones** (**NDSS 2023**, KAIST). `[S]` — [NDSS paper PDF](https://www.ndss-symposium.org/wp-content/uploads/2023/02/ndss2023_f112_paper.pdf); [NDSS page](https://www.ndss-symposium.org/ndss-paper/un-rocking-drones-foundations-of-acoustic-injection-attacks-and-recovery-thereof)
  - Sound resonates the proof masses of MEMS IMUs on drones; the paper proposes recovering the benign sensor values.
  - It found that sampling jitter strongly affects whether a drone crashes during an attack.
- **MARS** (arXiv 2505.00924, **PREPRINT, May 2025**). `[S]` — [arXiv 2505.00924](https://arxiv.org/html/2505.00924v1)
  - Model-based anomaly detection and recovery for UAVs.
  - "Deep Auto Encoders (DAE) have been deployed to learn the acoustic resonance signature; however, as we show in this work, they are unable to cope with sensor saturation, unexpected sensor measurements, and communication drops" (via search summary).
- **HEMA** — "A Hands-on Exploration Platform for MEMS Sensor Attacks" (arXiv 2507.06439, **PREPRINT, Jul 2025**). `[T]` — [arXiv 2507.06439](https://arxiv.org/pdf/2507.06439)
- **Continuous-wave laser injection into IMUs** (ResearchGate, 2025). 17 of 25 tested devices with MEMS inertial sensors "could be implicitly controlled". Which page this figure came from is uncertain; check before use. `[S]` — [ResearchGate](https://www.researchgate.net/publication/398056610_Signal_Injection_Attack_on_Inertial_Measurement_Unit_using_Continuous-Wave_Laser)
- **SoK: Rethinking Sensor Spoofing Attacks against Robotic Vehicles** (arXiv 2205.04662). Gyroscopes and accelerometers are both "susceptible to resonant acoustic interference". `[S]` — [arXiv 2205.04662](https://arxiv.org/pdf/2205.04662)
- **Change-point detection against acoustic attacks on MEMS IMUs** (MhIMU framework, 2024). `[T]` — [ResearchGate](https://www.researchgate.net/publication/383317892_Acoustic_Attack_Mitigation_Approach_for_MEMS_Inertial_Sensors_Using_Change_Point_Detection_on_MhIMU_Framework)
- **Banshee** (arXiv 2607.09930, **PREPRINT, Jul 2026**). `[S]` — [arXiv 2607.09930](https://arxiv.org/pdf/2607.09930)
  - Target-switch attacks on gimbal-stabilised visual tracking through acoustic injection, producing "consistent, directionally biased gimbal angular offsets in real time".
  - Its related-work table lists **KITE** as a speaker-plus-piezo attack that biases a drone controller's localisation.
  - It describes WALNUT as exploiting "aliasing and clipping" for precise control of MEMS accelerometers, and Nashimoto et al. as attacking sensor fusion with sound.
- **Learned controllers under injection.** One search combining acoustic IMU injection with learned, RL or neural controllers found **no paper combining them**. This is absence of evidence from a single search. `[S]` — same results as above.

### Inferences — crowding assessment
| Sub-area | Crowding | Strengthens a robotics question? |
|---|---|---|
| New acoustic/laser attack demos on MEMS IMUs | **Crowded**: at least 10 papers 2015–2026, now with a teaching platform | No; new demos fail filter 5 |
| Learned detectors and recovery for injected IMU data | **Active / moderately crowded** (MARS critiques autoencoders) | Weakly |
| How learned or *adaptive* low-level controllers respond to physical injection | **No overlap found (one search)** | **Yes.** Does in-context adaptation mistake injection for a dynamics change? This is an identifiability question (Candidate C) |
| Attacks on a cheap ground robot's IMU heading or its Hall-effect encoders | Replicates the Tu 2018 / ABS wheel-speed-spoofing lineage (UNVERIFIED, prior knowledge); same family as the student's rejected localisation-spoofing idea | No; fails filters 5 and 1 |

- **Practical constraints:** high sound levels need hearing protection and lab permission. Ultrasonic transducer prices were not sourced.

### Gaps
- Son 2015, Trippel 2017 (WALNUT) and Tu 2018 were not re-checked; their venues come from the brief.
- The primary KITE paper and its year were not found. An earlier claim that KITE achieved "stable and controllable injections" in Oct 2025 could not be attributed to a URL, so it is excluded.
- 2024–2026 physical attacks on learned visuomotor policies (e.g., adversarial patches) were **not searched** (search budget used up).
- Whether the Crazyflie's BMI088 or the 3pi+'s IMU is susceptible to acoustic resonance is **unknown**.

---

## 9. New since October 2025 that opens an MSc-scale question on cheap hardware

### Takeaway
Six items from October 2025 to October 2026 matter; three of them open usable questions:
- **RAPTOR v2** (April 2026): tiny adaptive learned controllers. Opens Candidates B and C.
- **Oh et al.** (October 2025): RL under least-restrictive safety filters. Opens a hardware question only if the task needs RL.
- **eFlesh** (June 2025, just before the window) together with the **TU Delft whiskers** (October 2025): sub-$5 printed tactile skins and MCU-only tactile navigation. Opens Candidate D.
- The rest mostly signal crowding: Banshee (July 2026) and Seeing Touch from Motion (June 2026).

### Cited Findings
- **RAPTOR v2** (6 Apr 2026): a 2,084-parameter in-context adaptive policy tested on 10 real quadrotors. — [arXiv 2509.11481](https://arxiv.org/abs/2509.11481v2) `[S]`
- **Whisker-based tactile flight** (3 Oct 2025; reported *Nat Commun* 18 Sep 2026): 3.2 g, sub-6 mm depth, 192 KB-RAM microcontroller. — [arXiv 2510.03119](https://arxiv.org/abs/2510.03119v1) `[S]`
- **Provably optimal RL under safety filtering** (Oct 2025). — [arXiv 2510.18082](https://arxiv.org/pdf/2510.18082v2) `[S]`
- **Latent CBF** (Nov 2025): 45% less override magnitude. — [arXiv 2511.18606](https://arxiv.org/pdf/2511.18606) `[S]`
- **Audio-VLA** (Nov 2025). — [arXiv 2511.09958](https://arxiv.org/pdf/2511.09958) `[S]`
- **Whisker spiking-network navigation** (Wiley, online Dec 2025). — [Wiley](https://onlinelibrary.wiley.com/doi/10.1002/cpe.70467) `[S]`
- **On-device CNN training on an ESP32-S3** (Apr 2026). — [arXiv 2604.23012](https://arxiv.org/pdf/2604.23012) `[S]`
- **Dual-barrier CBF on occupancy grids** (May 2026) and **humanoid input-to-state-safe CBF** (May 2026). — [arXiv 2605.05182](https://arxiv.org/html/2605.05182v2); [arXiv 2605.25546](https://arxiv.org/pdf/2605.25546) `[T]`
- **Seeing Touch from Motion** (Jun 2026). — [arXiv 2606.29941](https://arxiv.org/pdf/2606.29941) `[T]`
- **Banshee** (Jul 2026). — [arXiv 2607.09930](https://arxiv.org/pdf/2607.09930) `[S]`

### Inferences
- Of these, only RAPTOR v2 and the tactile/whisker work open questions that run on hardware the student owns, or nearly owns.
- The safety-filter preprints of 2025–26 target high-dimensional systems (humanoids, latent spaces), which the 3pi+ does not need (filter 4).

### Gaps
- No systematic scan was possible of CoRL 2025, RSS 2026, ICRA 2026 or IROS 2026 accepted papers (search budget used up). This section is therefore **incomplete**.

---

## 10. Which candidate questions survive all eight filters, and what is each one's first-week kill test?

### Takeaway
No candidate can yet be certified as passing all eight filters, because the nearest work for each was read only at search-summary level (filter 5). On paper, four candidates survive filters 1–4 and 6–8:
- **A — calibrated braking safety filter / information-for-speed on the 3pi+.** Strongest evidence base; uses owned hardware.
- **B — RAPTOR-style tiny adaptive controller on the 3pi+.** Strong method evidence, high risk that a classical baseline wins.
- **D — printed magnetic tactile bumper for pushing on the 3pi+.** Best Bristol supervision link; real risk of magnetic interference.
- **C — adaptive learned flight control under acoustic IMU injection.** The least crowded angle, but it needs BRL Crazyflies.

Each has a first-week test that could kill it. Everything else fails a named filter (list below).

### Cited Findings
- Evidence for each candidate is cited in Sections 1–9. The key anchors are:
  - **A:** [Tearle 2021](https://arxiv.org/pdf/2102.11907), [ABS RSS 2024](https://roboticsproceedings.org/rss20/p059.html), [Conformal Decision Theory, ICRA 2024](https://arxiv.org/pdf/2310.05921), [Oh et al. 2025](https://arxiv.org/pdf/2510.18082v2).
  - **B:** [RAPTOR](https://arxiv.org/abs/2509.11481v2), [Learning to Fly in Seconds](https://arxiv.org/abs/2311.13081), [RLtools](https://arxiv.org/pdf/2306.03530).
  - **C:** [Un-Rocking Drones](https://www.ndss-symposium.org/ndss-paper/un-rocking-drones-foundations-of-acoustic-injection-attacks-and-recovery-thereof), [MARS](https://arxiv.org/html/2505.00924v1), [Banshee](https://arxiv.org/pdf/2607.09930), plus the Crazyflie prices from [RobotShop UK](https://uk.robotshop.com/products/bitcraze-crazyflie-21) and the [Bitcraze store](https://store.bitcraze.io/products/crazyflie-2-1-plus).
  - **D:** [eFlesh](https://arxiv.org/html/2506.09994v1), [AnySkin via WowRobo](https://shop.wowrobo.com/products/enhanced-anyskin-premium-crafted-editionwowskin), [DigiTac](https://arxiv.org/abs/2206.13657), [lepora.com/papers](https://lepora.com/papers/).

### Inferences

#### Candidate A — information-for-speed: a calibrated braking safety filter on the 3pi+ (stretch goal: active friction or position probing)

**(a) Research question.** A 3pi+ works near a mapped hazard edge that it can only sense once its downward sensors reach it. Two questions follow:
- How much average speed does an online-calibrated (conformal) braking-distance filter recover over a fixed worst-case margin, at the same edge-violation rate, across surfaces and battery levels?
- Stretch: when is a short probing or re-localising manoeuvre worth the time it costs?

**(b) Why now.** The "why now" is moderate. No paper from October 2025 onwards opened this specifically.
- Conformal Decision Theory (ICRA 2024) calibrates a speed knob online for throughput under a risk guarantee.
- Oh et al. (Oct 2025) show that least-restrictive filtering preserves RL optimality, which motivates calibrated margins rather than worst-case ones.
- Latent CBF (Nov 2025) treats reducing filter override as a goal in its own right.

**(c) Nearest work.**
- Tearle et al., RA-L 2021: predictive safety filter on a miniature RC car with aggressive learned driving.
- Conformal Decision Theory, ICRA 2024: online calibration of a speed-like parameter, tested on datasets and modelled settings.
- Still open: calibrated margins on a robot that senses hazards only at contact, where braking depends on the surface; and the value of probing.
- Both papers were read at search-summary level only. The "no overlap" here comes from an incomplete search.

**(d) Strongest simple baselines.**
1. A fixed state-dependent braking rule, `v ≤ sqrt(2·a_min·(d−m)) − a_min·τ`, tuned on the worst surface. This is the analytic CBF.
2. A **surface-classifier lookup table**: identify the surface from line-sensor reflectance, then apply a fixed margin per surface.
3. A reactive "stop when the line sensor sees the edge".

Why they might fail:
- If the achievable deceleration `a_min` varies continuously with battery voltage, wheel dust or a surface the robot has not seen, the worst-case margin wastes speed.
- The lookup table fails when conditions drift within a "class" of surface.
- Show this by measuring the spread of stopping distances.

**(e) First-week kill test.**
- Set-up: about 240 stops — 10 stops × 4 speeds × 3 floor surfaces × 2 battery levels. Measure stop distance from encoder counts, with an overhead webcam and an ArUco marker as ground truth.
- **Kill if** the 95th-percentile stopping distance varies across conditions by less than the robot's own dead-reckoning error at the hazard, or by less than ~5 mm (threshold estimated). A fixed margin would then be close to optimal.
- **Kill also if** the 3pi+ cannot reach speeds at which braking distance exceeds the offset between its line sensors and its wheels. The reactive rule would then be enough.

**(f) Hardware and compute.**
- The 3pi+ and the webcam are owned. The 3pi+'s top speed, mass and encoder resolution are **not sourced** (the Pololu page fetch failed).
- An edge rig with a foam catch: **estimated** £0–£20.
- A laptop is enough; no GPU needed.
- Optional request to BRL: motion-capture time for ground truth.

**(g) Headline figure and outcomes.**
- Headline: "At ≤ 1% edge violations, the calibrated filter gives X% higher mean speed than a fixed worst-case margin (and Y% versus the surface lookup table), across N surfaces."
- Best realistic: a 15–30% speed gain on high-friction surfaces at the same violation rate (estimated).
- Floor: "On this robot, a fixed margin costs less than 5% of speed below v m/s." That is still a measured boundary.

**(h) Filter-by-filter verdict.**

| Filter | Verdict |
|---|---|
| 1 Robotics first | Pass: braking physics, slip, sensing latency |
| 2 Baseline named | Pass: three named baselines, with testable failure modes |
| 3 Gap is real | Conditional: settled by the week-1 test; ground truth is used only for evaluation, never for control |
| 4 Method fits the problem | Pass: online risk calibration under drift; no learned CBF |
| 5 Nearest work read at methods level | **Not met** |
| 6 Feasible | Pass |
| 7 First-week test | Pass |
| 8 Useful if the method loses | Pass |

**(i) Skills built.** Safety-critical control (CBFs and reachability), conformal and online calibration, embedded real-time control, experimental design. Relevant to GNC and autonomy roles.

**(j) Bristol people.** None found this session (ask #7 about BRL control and safety staff).

**Overlap note.** This candidate unifies the student's two open items, adaptive motion control and active localisation. Probing for friction and re-localising are both information-gathering actions whose value can be measured as safe speed gained. Consider merging it with #1's and #2's candidates.

#### Candidate B — a tiny in-context adaptive controller (RAPTOR-style) on the 3pi+'s ESP32-C3

**(a) Research question.** Can a recurrent policy of about 2k parameters, meta-trained in simulation over randomised differential-drive dynamics (mass, motor constants, friction, latency, battery voltage), track velocity and heading on a real 3pi+ under payload and surface changes better than PID with feedforward and a recursive-least-squares adaptive controller?

**(b) Why now.** RAPTOR (Sep 2025, v2 6 Apr 2026) is the first evidence that a 2,084-parameter recurrent policy adapts zero-shot, in context, across 10 real quadrotors from 32 g to 2.4 kg. RLtools and Learning to Fly in Seconds show that training takes seconds and deployment onto a microcontroller works.

**(c) Nearest work.**
- RAPTOR on quadrotors, plus Learning to Fly in Seconds.
- Still open:
  - whether in-context adaptation transfers to wheeled robots, where slip and motor saturation dominate;
  - whether it beats simple adaptive baselines. RAPTOR's own baselines were not retrieved.

**(d) Strongest simple baselines.**
- PID with integral action and feedforward, tuned at nominal.
- Recursive-least-squares adaptation of the feedforward gain.
- Gain scheduling on the measured battery voltage.

They could fail when a payload or surface change alters the acceleration limits and causes slip transients that a fixed-gain loop cannot anticipate.

**(e) First-week kill test.**
- Measure the tracking RMSE of a well-tuned PID on an aggressive velocity profile at three conditions: nominal, about +50% mass (mass not sourced), and a low-friction surface.
- **Kill if** the degradation is under ~15–20% (threshold estimated), or if the recursive-least-squares baseline closes the gap within a day.

**(f) Hardware and compute.**
- The 3pi+, ESP32-C3 and RTX 3060 are owned.
- Meta-training about 1,000 low-dimensional teachers on the 3060 is **estimated** to be feasible.
- The simulator must gain an actuator model.
- Payload weights: **estimated** about £5.

**(g) Headline figure and outcomes.**
- Headline: tracking RMSE under unseen changes for the tiny policy versus PID versus recursive least squares.
- Best realistic: 20–40% lower RMSE under change, with equal nominal performance (estimated).
- Floor: "PID plus recursive least squares is enough on a 3pi+; adaptation matters only above X payload." That is a measured boundary.

**(h) Filter-by-filter verdict.**

| Filter | Verdict |
|---|---|
| 1 Robotics first | Pass |
| 2 Baseline named | Pass, but high risk this is the "classical wins" failure mode again |
| 3 Gap is real | Conditional on the kill test |
| 4 Method fits the problem | Pass if the dynamics variation is real |
| 5 Nearest work read at methods level | Partial (RAPTOR at abstract level only) |
| 6 Feasible | Pass, once the simulator is upgraded |
| 7 First-week test | Pass |
| 8 Useful if the method loses | Pass |

**(i) Skills built.** Sim-to-real, JAX RL and distillation, neural-network inference on a microcontroller, adaptive control. Relevant to embedded, GNC and applied-ML roles.

**(j) Bristol people.** None found this session.

**Overlap note.** This is #1's open item with a specific method attached. Merge it there.

#### Candidate C — adaptive learned flight control versus PID under physically injected gyro corruption (Crazyflie on loan from BRL)

**(a) Research question.** When acoustic or ultrasonic resonance drives a Crazyflie's MEMS gyro, does a RAPTOR-style in-context adaptive policy mistake the injected signal for a change in dynamics, and fail differently from the stock PID cascade and from a non-adaptive learned policy? And does a physically motivated pre-filter restore it?

**(b) Why now.**
- RAPTOR (2025/26) puts adaptive learned low-level control on Crazyflie-class microcontrollers.
- Injection attacks keep advancing (MARS, May 2025; HEMA, July 2025; Banshee, July 2026).
- One search found no study of learned controllers under injection.

**(c) Nearest work.**
- Un-Rocking Drones (NDSS 2023): recovery for classical controllers.
- MARS (2025): model-based detection and recovery.
- KITE, via Banshee's table: a localisation attack on a drone controller.
- Still open: how learned or adaptive controllers respond. The single search finding nothing is not the same as established novelty.

**(d) Strongest simple baseline.** The stock PID cascade with a notch filter at the injected frequency. It fails if the frequency is unknown or if the injection aliases into the control band (the WALNUT-style aliasing mechanism, per Banshee's description).

**(e) First-week kill test.**
- Put a Crazyflie on a test stand or tether, sweep tones from a speaker or ultrasonic transducer, and log the gyro.
- **Kill if** there is no injected component above the noise floor at any frequency at the sound level available. Whether the BMI088 is susceptible is unknown.
- **Kill also if** the RAPTOR firmware cannot be flown on the BRL Crazyflie within a week.

**(f) Hardware and compute.**
- **Crazyflie 2.1+:** £214.95–£221.54 UK (sourced), out of stock or restocking at search time.
- **Crazyflie 2.1 Brushless:** £395.79–£461.54 (sourced).
- **Crazyradio 2.0:** price not found.
- Together these **exceed the £200 budget**.
- **Ask BRL for:** one or two Crazyflies with a Crazyradio 2.0, a Flow deck or motion-capture coverage, a cage or tether, and permission to run high-volume acoustic tests.
- Speaker and amplifier: **estimated** £20–£50. Hearing protection is required.

**(g) Headline figure and outcomes.**
- Headline: crash rate and attitude error under injection, for the adaptive policy versus the non-adaptive policy versus PID.
- Best realistic: identify a failure mode of in-context adaptation, such as adapting to phantom dynamics, together with a mitigation.
- Floor: "The BMI088 is not susceptible at accessible sound levels", plus a characterisation sweep. That is still a boundary result.

**(h) Filter-by-filter verdict.**

| Filter | Verdict |
|---|---|
| 1 Robotics first | Pass: sensor physics and controller dynamics are the subject |
| 2 Baseline named | Pass |
| 3 Gap is real | Pass: the injection is physical, not simulated |
| 4 Method fits the problem | Pass |
| 5 Nearest work read at methods level | Partial: the attack side is crowded; the learned-controller angle was searched only once |
| 6 Feasible | **Fails on own budget**; conditional on BRL kit |
| 7 First-week test | Pass |
| 8 Useful if the method loses | Pass |

**(i) Skills built.** Flight control, MEMS sensor physics, signal processing, hardware security testing. Relevant to GNC, defence and cyber roles around Bristol and Cheltenham.

**(j) Bristol people.** BRL aerial robotics; names not verified this session (#7).

#### Candidate D — a printed magnetic tactile bumper for pushing objects with the 3pi+

**(a) Research question.** Does a 3D-printed magnetic tactile bumper, in the style of eFlesh or AnySkin and reporting contact location and force, let a 3pi+ push unknown objects to a goal more reliably than a passive V-shaped pusher or bump-switch feedback, using onboard sensing only?

**(b) Why now.**
- eFlesh (June 2025) makes printable magnetic skins with under $5 of magnets and 0.5 mm contact localisation.
- AnySkin is now sold commercially ($48 for the skin).
- TU Delft's whiskers (October 2025) show tactile navigation running on a 192 KB-RAM microcontroller.

**(c) Nearest work.**
- Tactile pushing with a TacTip on an arm by Lepora's group (UNVERIFIED, prior knowledge).
- eFlesh and AnySkin, which are sensing papers with no mobile pushing.
- Mobile-robot tactile pushing was **not searched**, so overlap is unknown.

**(d) Strongest simple baselines.**
- A passive V-shaped or concave pusher, which self-centres the object mechanically, driven open-loop on dead reckoning.
- Reactive correction from the bump switches.
- An overhead-camera closed loop, used **only** as an upper bound in evaluation.

The baselines could fail because a V-pusher only stabilises objects that match its geometry: irregular, asymmetric or low-friction objects rotate off. Contact location is the key unobserved variable in pushing (classical pushing mechanics; UNVERIFIED, prior knowledge).

**(e) First-week kill test.**
1. Run 20 open-loop pushes for each of 3 object shapes with a printed V-pusher. **Kill if** ≥ 90% reach the goal within tolerance.
2. Bench-test the skin on the robot while the motors run at full power. **Kill if** interference from the motor and encoder magnets pushes contact-location error above ~5 mm (threshold estimated).

**(f) Hardware and compute.**
- A TPU-capable 3D printer: **ask BRL**.
- Magnets: under $5 (sourced).
- Magnetometer breakout(s): price not found; **estimated** £5–£15 each.
- The ESP32-C3 for I2C and the webcam for evaluation are owned.
- A small neural network as a contact regressor runs on a CPU.

**(g) Headline figure and outcomes.**
- Headline: push success rate and final pose error versus the V-pusher and bump-switch baselines, across object shapes.
- Best realistic: touch closes most of the gap to camera-based pushing on irregular objects.
- Floor: "A V-pusher is enough for shapes X; touch helps only for Y." That is a measured boundary.

**(h) Filter-by-filter verdict.**

| Filter | Verdict |
|---|---|
| 1 Robotics first | Pass |
| 2 Baseline named | Pass |
| 3 Gap is real | Pass: onboard-only sensing is a real constraint; the camera is used only as an upper bound |
| 4 Method fits the problem | Pass: contact location is the state that pushing needs |
| 5 Nearest work read at methods level | **Not met** |
| 6 Feasible | Pass, given printer access |
| 7 First-week test | Pass |
| 8 Useful if the method loses | Pass |

**(i) Skills built.** Sensor fabrication, magnetometer signal processing, contact mechanics, embedded I2C, small learned regressors.

**(j) Bristol people.** Nathan Lepora's tactile robotics group at Bristol. Confirmed this session only through his papers listing and the DigiTac entry on the Bristol research portal.

#### Ranking by evidence (not by fit to the student)
1. **A.** The physics is plausible but unmeasured. There are precedents for filters expanding capability (Tearle 2021, ABS 2024, Conformal Decision Theory 2024). It uses owned hardware and has a crisp kill test. Weaknesses: novelty unverified, and learning plays only a small part.
2. **B and D, joint second.**
   - B has strong method evidence (RAPTOR on real hardware) but no evidence yet that a 3pi+ needs adaptation, and a high chance the classical baseline wins.
   - D has strong sensor evidence (eFlesh figures) and the best supervision link, but no evidence yet on tactile pushing with a mobile robot, and a real interference risk.
3. **C.** The least crowded angle, on learned controllers, but the attack physics is crowded and it is not feasible on budget without BRL equipment.

#### Tempting but fails (filter named)
1. **Learned CBF, DeepReach or a neural reach-avoid filter on the 3pi+.** Fails 4: the state is low-dimensional, so the analytic braking CBF or a grid-based Hamilton–Jacobi solution is enough. Also fails 2.
2. **RL trained under a safety filter on the 3pi+, to test Oh et al. 2025.** Fails 2 and 3: a hand-coded controller already beat the learners on navigation, so the task does not need RL.
3. **Robotarium-style multi-robot CBF collision avoidance with several 3pi+ robots.** Fails 5 (crowded since 2017, per the brief's citation of Wang, Ames & Egerstedt) and 6 (needs several robots and global state). Also in #3's scope.
4. **Conformal safety around moving people or obstacles on the 3pi+.** Fails 6: the robot has no exteroceptive sensing. Also fails 3: an overhead camera would hand it perfect state.
5. **An actively scanned single-beam time-of-flight sensor on a servo.** Fails 3: a fixed multizone time-of-flight sensor probably removes the need (not sourced).
6. **Pan-tilt active vision for manipulation.** Fails 6 (no arm) and is at risk on 2 (one extra static webcam).
7. **Visuo-tactile imitation learning on an SO-101 with AnySkin.** Fails 5 (crowded 2025–26). Also in #4's scope.
8. **A visual self-model of an SO-101 versus kinematic calibration.** Fails 6 (no arm) and is likely to fail 2.
9. **A self-model of the 3pi+.** Fails 4: it is system identification.
10. **A contact microphone for terrain or slip sensing on the 3pi+.** Fails 5 (a classical area) and 2 (the IMU and encoders are probably enough).
11. **Whiskers on the 3pi+.** Fails 3 (time-of-flight baseline) unless transparent obstacles are the explicit target.
12. **Replicating acoustic gyro injection against the 3pi+'s heading control.** Fails 5 (the Tu 2018 lineage) and 1 (security is only the backdrop).
13. **Magnetic spoofing of the 3pi+'s Hall-effect encoders, with an IMU consistency check.** Fails 5 (classical fault detection; the same family as the rejected localisation-spoofing idea) and 3 (the threat model is contrived).
14. **Replicating Learning to Fly in Seconds on a Crazyflie.** Fails 6 (budget: £214.95–£221.54 for the drone alone, out of stock) and 5 (a replication).

### Gaps
- **Filter 5 is outstanding for all four candidates.** Before the report recommends any of them, read these at methods level:
  - for **A:** the methods and limitations sections of Tearle 2021 and Conformal Decision Theory, then search for "conformal safety margin mobile robot hardware" and "braking distance learned speed limit mobile robot";
  - for **B:** RAPTOR's baselines and limitations, then search for "in-context adaptation wheeled robot";
  - for **C:** the Un-Rocking Drones, MARS and KITE primaries, and search for "neural flight controller sensor injection";
  - for **D:** Lloyd & Lepora's tactile pushing, and search for "tactile bumper pushing mobile robot".
- **Bristol/BRL names come from prior knowledge, are UNVERIFIED, and should be passed to #7 to check:**
  - Martin Pearson: whiskers, BRL/UWE;
  - Arthur Richards: aerial robotics and control;
  - Efi Psomopoulou, Benjamin Ward-Cherrier and John Lloyd: tactile;
  - Sabine Hauert: swarms;
  - Kerstin Eder: verification;
  - Helmut Hauser and Jonathan Rossiter: soft robotics and morphology;
  - Tom Richardson: UAVs.

  Only Nathan Lepora was confirmed this session.
- **Not sourced:** Pololu 3pi+ specifications (top speed, mass, IMU part numbers, encoder counts per revolution), the SO-101 price, the Crazyradio 2.0 price, the Flow deck v2 price, and prices for piezo mics, magnetometer breakouts and pan-tilt kits.
- **All "estimated" thresholds** in the kill tests (5 mm, 15–20%, 90%, 95%) are judgement calls, to be set after a pilot.
