# Multi-robot and swarm learning on real low-cost robots (2024–2026), BRL swarm work, and dissertation-candidate screening

Research date: Saturday 10 October 2026. Researcher: sub-agent for task #3 (multi-robot and swarm learning).

**Access limits (read first).** The session's egress policy returned HTTP 403 for these hosts:
- Papers and indexes: arxiv.org (all mirrors), IEEE Xplore, Springer, ResearchGate, Semantic Scholar, OpenAlex, OpenReview, PMLR, PMC, Zenodo, nature.com, IRIDIA, Sheffield, Hugging Face, alphaXiv, archive.org.
- Bristol and BRL: bristol.ac.uk and its research portal, hauertlab.com, huntlab.uk, brl.ac.uk.
- Other: robotarium.gatech.edu, docs.espressif.com, pololu.com, iieta.org, dl.ifip.org.

The WebFetch tool had no DNS. **Readable:** raw GitHub files, `git clone` of GitHub repositories (including committed PDFs and project-page sources), and PyPI. The shared web-search budget (200 calls per turn across all agents) ran out mid-task, so several planned searches never ran; they are listed under Gaps. **Only two paper documents were read in full, both from GitHub:** the MARBLER supplementary material (5 pages) and Ravichandar's 2026 research statement. The JaxRobotarium abstract was read in full from its lab-website post. All other paper-level claims come from search-engine excerpts, which are abstract-level or less. Treat them as leads to verify, not as methods-level readings.

**Reading-depth tags used on every source:**
- **[R]**: primary file read directly (GitHub README, source code or config) at the relevant section or in full.
- **[S]**: search-engine excerpt only (abstract-level or less; full text blocked).
- **[T]**: title or metadata only.
- **[M]**: GitHub or PyPI metadata (dates, versions, repository descriptions).
- **PREPRINT** marks arXiv-only work. A venue is given only where a source states it.

---

## 1. What has genuinely moved in decentralised multi-robot learning, with real-robot evidence on small or cheap platforms (2024–2026)?

### Takeaway
What moved in 2024–2026 is infrastructure, not capability. GPU or JAX-vectorised simulators plus open deployment pipelines (VMAS→Cambridge RoboMaster; JaxRobotarium→Robotarium) cut training to minutes and made zero-shot deployment routine. Real-robot evidence is still small-N (5 RoboMasters; tens of Robotarium robots) and on simple tasks (navigation, passage, formation). Almost every successful deployment learns only a high-level decision and leaves low-level motion to a classical controller and a safety filter (barrier certificates, waypoint controllers, "semantic actions"). That reduces the "learned decentralised policy on real robots" claim to "learned decision layer over classical control".

### Cited Findings
**Prorok lab (Cambridge) line**
- Blumenkamp, Morad, Gielis, Li & Prorok, "A Framework for Real-World Multi-Robot Systems Running Decentralized GNN-Based Policies", IEEE ICRA 2022, pp. 8772–8778 (arXiv 2111.01777). The paper presents a system for fully decentralised execution of GNN policies on an open-source ROS 2 framework. The case study "requires tight coordination between robots". It warns that "real-world wireless communication is noisy, and messages can be lost or delayed" [S] — [ADS listing](https://ui.adsabs.harvard.edu/abs/2021arXiv211101777B/abstract), [arXiv](https://arxiv.org/pdf/2111.01777)
  - A secondary summary says it recommends multicast and fewer 802.11 retries to cut latency, and that both communication delay and robot dynamics drive performance loss [S, secondary aggregator; treat cautiously] — [Lacuna summary](https://lacuna.tiptreesystems.com/work/a-framework-for-real-world-multi-robot-systems-running-decentralized-gnn-based/wrk_3f2f30e37c5aabff3f4dc9db148b0813)
  - The training code trains a multi-agent PPO policy in RLlib ("training will stop automatically after 5000 training iterations") and exports TorchScript for ROS 2 inference [R] — [rl_multi_agent_passage README](https://github.com/proroklab/rl_multi_agent_passage)
- Cambridge RoboMaster (Blumenkamp, Shankar, Bettini, Bird & Prorok), arXiv 2405.02198, v2 October 2024, **PREPRINT** (no venue confirmed). It uses customised DJI RoboMaster S1s, which the authors place "between small robots with too little compute or actuation and larger robots that are impractical for indoor multi-robot tests". The stack is ROS 2 estimation and control with ad-hoc peer-to-peer communication. It runs MARL policies trained in VMAS zero-shot. Demonstrations: 5 robots pass a narrow constriction by breaking and re-forming formation, and 2 robots do SLAM-based collision avoidance [S] — [arXiv](https://arxiv.org/abs/2405.02198v2), [TechXplore](https://techxplore.com/news/2024-05-team-agile-multi-robot-platform.html)
  - The repository README still says "Paper: TBD" (repo updated 23 Sep 2026) [R][M] — [cambridge-robomaster](https://github.com/proroklab/cambridge-robomaster)
  - The build documentation says "detailed bill of materials. TBD". Compute is a Jetson Orin on a D131 carrier with an Intel M.2 WiFi module, or a Raspberry Pi variant. Networking is set up by an ad-hoc `nmcli` script. No latency or packet-loss figures appear in the docs [R] — [hardware_setup.md](https://github.com/proroklab/cambridge-robomaster/blob/master/docs/hardware_setup.md), [software_setup.md](https://github.com/proroklab/cambridge-robomaster/blob/master/docs/software_setup.md)
  - **The paper's platform-comparison data** (`paper/robot_comparison.csv`) [R] — [robot_comparison.csv](https://github.com/proroklab/cambridge-robomaster/blob/master/paper/robot_comparison.csv)
    - The `price` column has **no currency stated**: Cambridge RoboMaster 689, e-puck 850, Kilobot 14, Crazyflie 225, Duckietown 150, TurtleBot 3 850, Khepera IV 3180, Mona 120, Colias 40, Pheeno 270.
    - Maximum velocity (m/s): RoboMaster 4.45, e-puck 0.13, Kilobot 0.01, Duckietown 0.5, Crazyflie 1.
    - The Pololu 3pi+ is not in the table.
- VMAS, DARS 2022 [R] — [VMAS README](https://github.com/proroklab/VectorizedMultiAgentSimulator)
  - Fidelity limitation, quoted: "Holonomic point-robot simulation has been chosen to focus on general high-level problems, without learning low-level custom robot controls through MARL."
  - Optional `DiffDriveDynamics`, `KinematicBicycleDynamics` and `Drone` models exist.
  - Scenarios include `transport` (packages default to 50× agent mass), `passage`, `give_way`, `flocking` and `discovery`.
  - Latest release 1.5.2, 10 Nov 2025 [M] — [PyPI](https://pypi.org/pypi/vmas/json)
- BenchMARL, *JMLR* 25(217), 2024 [R] — [README](https://github.com/facebookresearch/BenchMARL). Latest release 1.5.2, 7 Feb 2026 [M] — [PyPI](https://pypi.org/pypi/benchmarl/json)
- HetGPPO, "Heterogeneous Multi-Robot Reinforcement Learning" (Bettini, Shankar & Prorok, AAMAS 2023), and SND (behavioural-diversity metric, arXiv 2305.02128, PREPRINT) [R] — [HetGPPO README](https://github.com/proroklab/HetGPPO). I could not verify hardware-experiment details (search budget exhausted).
- Prorok-lab repositories created or active in 2025–2026 [M/R] — [repo list](https://github.com/proroklab)
  - D4orm "Denoising Multi-Robot Trajectories", T-RO 2026: model-based diffusion refines trajectories to be "kinodynamically feasible, collision-free"; it has distributed variants [R] — [d4orm](https://github.com/proroklab/d4orm)
  - "Concrete Multi-Agent Path Planning Enabling Kinodynamically Aggressive Maneuvers", npj Robotics 2026: a LaCAM-based MAPF solver with execution constraints, shown with 8 drones [R] — [agile-mapf](https://github.com/proroklab/agile-mapf)
  - GNN-guided MAPF search: LaGAT (AAAI-26) and HMAGAT (hypergraph, ICLR 2026) [M]
  - "Remotely Detectable Robot Policy Watermarking", arXiv 2512.15379 (Dec 2025, PREPRINT), evaluated on VMAS Navigation with "ground-truth action", "onboard sensor" and "remote camera feed" detection modes [R] — [RobotPolicyWatermarking](https://github.com/proroklab/RobotPolicyWatermarking)
  - An ESP32 UDP receiver for drone control (ESP-IDF v5.5.1; branch `release/corl-osh-workshop`) [R] — [sanity_receiver](https://github.com/proroklab/sanity_receiver)
  - CoViS-Net, a "Cooperative Visual Spatial Foundation Model for Multi-Robot Applications" [M] — [CoViS-Net](https://github.com/proroklab/CoViS-Net)

**Robotarium line (Georgia Tech)**
- MARBLER (Torbati, Lohiya, Singh, Nigam & Ravichandar), MRS 2023, pp. 57–63, Best Paper Award [R] — [MARBLER README](https://github.com/GT-STAR-Lab/MARBLER)
  - It joins the Robotarium to Gym and EPyMARL, compares MAPPO, QMIX, VDN and QMIX_NS on 4 scenarios, and finds VDN best. The authors report "few differences between simulation and real-world" performance, but real results come from only 10 episodes against 100 in simulation [S] — [arXiv 2307.03891](https://arxiv.org/pdf/2307.03891), [project page](https://shubhlohiya.github.io/MARBLER/)
  - Maintenance limitation (README): it works only with simulator commit 6bb184e; it "will run with the most recent push to the Robotarium but it will crash during training"; it needs Python 3.8 [R]
  - **MARBLER supplementary material (5 pages, read in full)** [R] — [supplementary.pdf](https://github.com/shubhlohiya/MARBLER/blob/gh-pages/assets/supplementary.pdf)
    - All scenarios use 5 discrete actions (left, right, up, down, stop) held for a fixed time.
    - Speeds are about 6–25 cm/s; teams have 4–6 robots.
    - **Most scenarios give "full communication so each robot had all other robots' observations concatenated together".** Material Transport alone uses "full communication with limited bandwidth" (a message from 0 to 3).
    - Warehouse needs learned collision avoidance "as the barrier certificates will slow them down significantly".
    - MAPPO is "much more influenced by its seed than VDN"; VDN was best for all scenarios after 5M timesteps.
    - Hyperparameters were searched over 2 seeds per algorithm.
- JaxRobotarium (Jain, Liu, Kailas & Ravichandar), **CoRL 2025** (venue confirmed by the lab's own paper post), arXiv 2505.06771 [R: abstract in full] — [lab post](https://github.com/GT-STAR-Lab/gt-star-lab.github.io/blob/main/papers/_posts/2025-09-28-jain-jaxrobotarium.md), [PMLR listing](https://proceedings.mlr.press/v305/jain25a.html), [arXiv](https://arxiv.org/abs/2505.06771)
  - The abstract states that established MARL platforms (SMAC, MPE) "lack robotics relevance and hardware deployment", and that MARBLER "lacks support for parallelization and GPU/TPU execution".
  - It claims "realistic robot dynamics and safety constraints", eight scenarios (four new ones that bring RWARE and Level-Based Foraging to robots), that the simulator "retains high simulation fidelity", and 20× training and 150× simulation speed-ups.
  - Up to 150× faster simulation and 20× faster training than MARBLER.
  - 8 scenarios and 4 MARL algorithms, with a sim-to-real study of more than 200 real deployments.
  - Hardware performance was close to simulation for many task–algorithm pairs; some pairs showed noticeable differences, and simple randomisation (noisy actions) reduced them.
  - The authors note it "is not a panacea for sim-to-real gap" [S]
- JaxRobotarium action interface (README) [R] — [JaxRobotarium README](https://github.com/GT-STAR-Lab/JaxRobotarium)
  - Discrete actions "are interpreted as waypoints to follow" by a go-to-goal controller with a barrier function. Continuous actions are unicycle velocity commands. Each environment step runs 30 Robotarium steps by default.
- JaxRobotarium deployment path [R] — [deploy README](https://github.com/GT-STAR-Lab/JaxRobotarium/blob/main/jaxrobotarium/deploy/README.md)
  - Flax policies are re-implemented as PyTorch actors with safetensors weights and uploaded through the Robotarium web interface.
  - Scenario files present include navigation, warehouse, material_transport, arctic_transport, foraging, discovery and rware [R, file probe].
- JaxMARL (NeurIPS 2024 Datasets & Benchmarks) now lists JaxRobotarium ("open access sim2real through the Robotarium") and JaxNav ("2D geometric navigation for differential drive robots") [R] — [JaxMARL README](https://github.com/FLAIROx/JaxMARL). Version 0.2.0 released 10 Sep 2026 [M] — [PyPI](https://pypi.org/pypi/jaxmarl/json)

**Other platforms**
- Crazyflie: "Decentralized Control of Quadrotor Swarms with End-to-end Deep RL" (arXiv 2109.07735) reports zero-shot transfer to Crazyflie 2.0 with a Deep-Sets/attention model, not a GNN [S] — [arXiv](https://arxiv.org/pdf/2109.07735)
  - The excerpt says controllers run at 500 Hz, which makes communication latency "exceptionally tight" for multi-layer GNNs. This may be the summariser's inference, not the paper's statement.
  - Venue conflict: the excerpt says IROS 2020; my recollection is CoRL 2021. Unverified.
- AttentionSwarm (arXiv 2503.07376, PREPRINT): MADRL with attention control barrier functions on Crazyflies [S] — [arXiv](https://arxiv.org/html/2503.07376)
- A hypergraph decentralised-RL preprint claims a 16–22% sim-to-real performance drop against 45–68% for baselines, and 78% less communication overhead, with swarms of up to 50 agents in simulation. The hardware platform was not visible [S; unverified] — [ResearchGate](https://www.researchgate.net/publication/397090848_Decentralized_reinforcement_learning_for_scalable_embodied_intelligence_in_robotic_swarms)
- IDEA (Liu, Zhang et al.), arXiv 2606.26575, 25 Jun 2026, PREPRINT [S] — [arXiv](https://arxiv.org/abs/2606.26575)
  - It argues that learned multi-agent methods "break under sim-to-real gaps" because they act in low-level control spaces.
  - It maps discrete semantic actions to closed-loop low-level controllers to "bound the sim-to-real performance gap without explicitly modeling low-level physical parameters".
  - It adds communication-based action synchronisation for "inter-agent action timing mismatches".
- Duckietown: a MARL survey (arXiv 2408.09675, PREPRINT) describes MAPPO deployed on real robots in a Duckietown-based setup after the sim-to-real gap is addressed [S; details not verified] — [arXiv](https://arxiv.org/pdf/2408.09675)
- Mixed-reality digital twins for hybrid sim-to-real MARL (arXiv 2403.10996, PREPRINT) [T] — [arXiv](https://arxiv.org/pdf/2403.10996)
- Further leads [T]:
  - "Graph Neural Planning and Predictive Control for Multi-Robot Communication-Constrained Unlabeled Motion Planning" (arXiv 2605.19209, PREPRINT) — [arXiv](https://arxiv.org/html/2605.19209v1)
  - LPAC, GNN perception-action-communication loops for coverage (arXiv 2401.04855) — [arXiv](https://arxiv.org/pdf/2401.04855)
  - "Physics-Informed MARL for Distributed Multi-Robot Problems" (arXiv 2401.00212) — [arXiv](https://arxiv.org/pdf/2401.00212)

### Inferences
- **The real advance is turnaround time.** "Train in minutes, deploy the same day on 5–20 real robots" now exists in open source (JaxRobotarium on the Robotarium; VMAS on the RoboMaster). That makes systematic sim-to-real measurement cheap for the first time. It does not show that learned decentralised control beats anything.
- **Learned layers sit on classical control in every 2024–2026 deployment found.** MARBLER and JaxRobotarium wrap policies in barrier certificates and waypoint controllers. IDEA (2026) makes semantic actions an explicit design principle. Expect a gap measured on such stacks to reflect interaction and timing effects more than motor physics.
- **"Decentralised" is often nominal in the Robotarium benchmarks.** MARBLER's supplementary shows most scenarios give every robot all other robots' observations, concatenated. Policies then run per robot but see global state, and their input size grows with N, so they cannot transfer across team sizes. A genuinely decentralised, N-transferable study needs local or neighbourhood observations (GNN, Deep Sets or k-nearest neighbours). That choice is justified by the scaling question itself (filter 4), not imposed to create a need (filter 3).
- **Prorok-lab output has moved towards learning inside planners.** Public 2025–2026 repositories cover GNN-guided MAPF search (LaGAT, HMAGAT), diffusion-based trajectory optimisation (D4orm) and kinodynamic MAPF, more than end-to-end MARL on robots. This is my inference from repository metadata, not a stated change of direction.
- **No cheap differential-drive MARL deployment with acceleration-limited dynamics was found.** VMAS is holonomic by default and the Robotarium simulator is kinematic (Section 6). The student's earlier failure mode (a simulator with no acceleration limits) would recur unless the dynamics are added and identified.

### Gaps
- The main papers of Blumenkamp 2022, Cambridge RoboMaster, MARBLER and JaxRobotarium were blocked; only MARBLER's supplementary and JaxRobotarium's abstract were read in full. Their limitations and future-work sections were not read, and any team-size analysis in JaxRobotarium is unknown. Read it before committing to Candidate C1.
- No 2024–2026 real-robot MARL papers on e-puck, Kilobot or Duckiebot were found. The searches were incomplete because the budget ran out.
- HetGPPO and DiCo hardware details, and any 2025–2026 Prorok-lab real-robot MARL paper, were not verified.

---

## 2. Where does learning genuinely beat classical multi-robot methods on real robots, and where has it been shown not to?

### Takeaway
Head-to-head comparisons on physical robots between a learned policy and the *strongest* classical method are rare. I found none for ORCA, consensus, CBBA or response thresholds. The best real-robot evidence in swarms runs the other way: neural controllers produced by learning or evolution cross the reality gap worse than modular, hand-structured ones (the AutoMoDe line on e-pucks). Where learned controllers "win", it is usually against weak baselines (no mitigation, or a non-learning default) and in simulation.

### Cited Findings
- **Collective transport (classical strongest baseline).** Chen, Gauci, Li, Kolling & Groß, *IEEE T-RO* 2015, "Occlusion-based cooperative transport with a swarm of miniature mobile robots" [S] — [Sheffield supplementary page](https://sheffield.ac.uk/naturalrobotics/supp/2014-002), [CORE PDF](https://files01.core.ac.uk/download/pdf/46665030.pdf)
  - Robots push only where the object occludes their view of the goal. The method is fully decentralised and needs no explicit communication.
  - It is proved for convex objects in the plane; 20 e-pucks succeeded in 43 of 45 trials.
- **Extensions of the occlusion strategy**
  - Concave objects: robots fill the cavity so the object becomes effectively convex (Nirgude et al., arXiv 2409.02436, PREPRINT) [S] — [arXiv](https://arxiv.org/abs/2409.02436)
  - Obstacles: a sub-goal behaviour handles obstacles that block line of sight, with results from a physics-based simulator only (arXiv 2605.13006, May 2026, PREPRINT) [S] — [arXiv](https://arxiv.org/pdf/2605.13006)
  - **No learned occlusion-based policy was found** [S].
- Learned transport is either related but not occlusion-driven, or in simulation:
  - CoLF, VLM-guided leader–follower policies (arXiv 2602.07776) [T] — [arXiv](https://arxiv.org/pdf/2602.07776)
  - Cooperative grasping with two mobile manipulators (arXiv 2509.03638) [S] — [arXiv](https://arxiv.org/html/2509.03638)
  - BRL (Section 5): a neuroevolved fault-mitigation module beats *no mitigation* (Lee & Hauert, EvoApplications 2025) [S] — [Springer](https://link.springer.com/chapter/10.1007/978-3-031-90062-4_19)
  - BRL: a MARL recurrent shared policy is "more robust to faults than a non-learning baseline" in box transport (Alboraei, Hauert & Lee, ANTS 2026) [S] — [Springer](https://link.springer.com/chapter/10.1007/978-3-032-26123-6_15)
- **Collision avoidance.** I found no direct learned-versus-ORCA comparison on hardware [S].
  - In simulation, a learned policy is reported "much better than previous decentralized methods" (arXiv 1709.10082) — [arXiv](https://arxiv.org/pdf/1709.10082)
  - V-RVO reports ORCA "tends to fail" in dense benchmarks of 25–70 agents because its constraints are not satisfied (arXiv 2102.13281) — [arXiv](https://arxiv.org/pdf/2102.13281)
  - A 2025 MPPI paper notes that ORCA-style methods "assume … perfect sensing" of neighbours' positions, velocities and shapes. It also says purely learned reactive behaviours "may fail to generalize to unseen scenarios". Its tests ran in Gazebo with TurtleBots, not on hardware [S] — [arXiv 2507.20293](https://arxiv.org/html/2507.20293v1), [arXiv 2511.09331](https://arxiv.org/html/2511.09331v2)
- **Reality gap of learned versus structured swarm controllers (real e-pucks)**
  - AutoMoDe controllers "appear to be robust to the so called reality gap" in aggregation and foraging. A follow-up compared them with neuroevolution on e-pucks [S] — [AutoMoDe 2014](https://www.researchgate.net/publication/260816457_AutoMoDe_A_novel_approach_to_the_automatic_design_of_control_software_for_robot_swarms)
  - Modular methods (TF-WS-B, TF-HV, TF-L2, Mandarina) "transfer better from simulation to reality than those based on neuroevolution" (IRIDIA 2024, concurrent design criteria) [S] — [IRIDIA PDF](https://iridia.ulb.ac.be/~mbiro/paperi/GarPagStuBir2024AIS.pdf)
  - Ligot & Birattari, *Swarm Intelligence* 2020: "a performance drop may appear in a simulation-only setting" (a "pseudo-reality"). A design method can rank better in simulation and worse in reality [S] — [IRIDIA supplement](https://iridia.ulb.ac.be/supp/IridiaSupp2019-002/index.html), [Frontiers review](https://www.frontiersin.org/journals/robotics-and-ai/articles/10.3389/frobt.2023.1134841/full)
- **Learned MARL on real Robotarium robots, compared only with other learners.** MARBLER compares four MARL algorithms; no classical controller baseline appears in the excerpts or in its supplementary (read in full). In JaxRobotarium, the hardware gap is small for many task–algorithm pairs and reduced by action-noise randomisation [S] (Section 1).
- **Learned low-level control is fragile.** IDEA (2026) argues low-level learned control is fragile under dynamics mismatch, so it keeps learning at the semantic level [S] (Section 1).
- **Patrol (BRL, simulation only).** Heterogeneity in an individual learning trait ("latent inhibition") changes team monitoring performance; mostly high-LI robots plus one low-LI robot performed best (York, Madin, O'Dowd & Hunt, ICDL 2024) [S] — [arXiv 2403.01181](https://arxiv.org/abs/2403.01181), [IEEE](https://ieeexplore.ieee.org/document/10645013/)

### Inferences
- **Learning has a structural case (still unproven on cheap real robots) only where a classical method's assumptions break:**
  - ORCA's "perfect neighbour state" under noisy sensing.
  - Occlusion transport for non-convex objects, obstacles, or too few robots. Chen et al. used 20; a student can field 3–6.
  - Fault combinations too numerous to hand-design (BRL's 2025–2026 line).
  - Heterogeneous roles that are hard to hand-specify.
- **Learning has no structural case for small teams doing navigation or allocation** (3–6 robots, few tasks, all-to-all radio). Greedy or auction allocation, a CBF plus go-to-goal controller, or ORCA should be near-optimal, which repeats the student's earlier "ten-line controller wins" result. This is an inference: no 2024–2026 real-robot learned-versus-CBBA or response-threshold comparison was found.
- **The strongest documented real-robot pattern is the AutoMoDe finding.** Less expressive, modular controllers cross the reality gap better. Any project that pits a neural policy against a structured controller on real robots should expect to reproduce it, and should design for that as the likely, still useful, outcome (filter 8).

### Gaps
- Real-robot learned-versus-classical comparisons for task allocation (CBBA, auction, response threshold), consensus and flocking: not found, search incomplete.
- The limitations section of Chen et al. 2015 was not read; the 43/45 figure comes from a search excerpt of the supplementary page.
- Full methods of the ANTS 2026 and EvoApps 2025 papers (simulator, robot count, baseline definitions) were not read.

---

## 3. Real communication constraints (ESP32 WiFi / ESP-NOW) and learned communication under loss and latency

### Takeaway
At the team sizes a student can field (3–7), measured ESP-NOW performance indoors is milliseconds to tens of milliseconds of latency with a few percent loss. That is likely below what would break a 10–20 Hz decentralised controller. "Robust learned communication under real WiFi loss" is therefore likely a manufactured gap on this hardware unless a cheap measurement shows otherwise. No real-world evaluation of learned communication under measured loss and latency was found; that search did not run.

### Cited Findings
- **ESP-NOW protocol facts** (Espressif ESP-IDF documentation source) [R, relevant section] — [esp_now.rst](https://github.com/espressif/esp-idf/blob/master/docs/en/api-reference/network/esp_now.rst)
  - "The default ESP-NOW bit rate is 1 Mbps."
  - Maximum payload is 250 bytes (v1.0) or 1470 bytes (v2.0).
  - The send callback reports MAC-layer success only: "It is not guaranteed that application layer can receive the data". Espressif recommends application-level acknowledgements and sequence numbers.
  - At most 20 paired devices; "encrypting multicast vendor-specific action frame is not supported".
- **Multi-robot ESP-NOW mesh, indoor corridors** (IIETA *JESA*; year and volume not verified; methodology not checked; low-tier venue, so treat with caution) [S] — [IIETA](https://iieta.org/journals/jesa/paper/10.18280/jesa.590507), [PDF](https://www.iieta.org/download/file/fid/194339)
  - Packet delivery ratio above 98% with 3 robots, above 95% with 5 and above 92% with 7.
  - Latency below 20 ms with 3 robots and below 40 ms with 7.
- **ESP-NOW outdoors** (IFIP WONS 2025) [S] — [WONS 2025 PDF](https://dl.ifip.org/db/conf/wons/wons2025/1571077625.pdf)
  - Bench one-way delay about 1059 µs for a 1-byte payload, measured between two cable-connected boards.
  - Field tests at 21 distances and 205 forest positions (delay, jitter, reliability). I did not obtain the per-distance figures.
- **Anecdotal ESP-NOW broadcast timing** (not peer-reviewed): about 800 µs fixed sender delay and up to about 5 ms jitter for a 1-byte broadcast to 5 receivers [S] — [Arduino Forum](https://forum.arduino.cc/t/esp-now-broadcast-transmisison-latency/1347159)
- Gielis, Shankar & Prorok, "A Critical Review of Communications in Multi-robot Systems", *Current Robotics Reports* 3(4):213–225, 2022 [S] — [Springer](https://link.springer.com/article/10.1007/s43154-022-00090-9), [arXiv](https://arxiv.org/pdf/2206.09484)
  - Multi-robot control often assumes synchronous execution, which "imposes a hard limit on message delay".
  - Commonly deployed protocols use best-effort delivery as "the standard paradigm".
- Hybrid TDMA/CSMA for robots (arXiv 2509.06119, PREPRINT): CSMA/CA collisions and backoff cause "long and unpredictable queuing latency". In emulation, most failures were missed deadlines rather than outright loss [S] — [arXiv](https://arxiv.org/html/2509.06119v2)
- The Prorok lab controls drones over ESP32 UDP broadcast: `udp_broadcast.py` and a ROS 2 bridge, built on ESP-IDF v5.5.1 [R] — [sanity_receiver](https://github.com/proroklab/sanity_receiver)
- The Robotarium's GTernal robots communicate through an MQTT broker over WiFi (`MQTT_HOST`, `MQTT_PORT=1884` in the firmware configuration) [R] — [GTernal env_variables.sh](https://github.com/robotarium/GTernal/blob/main/config/env_variables.sh)
- IDEA (2026) adds communication-based action synchronisation for inter-agent timing mismatch [S] (Section 1).
- Blumenkamp 2022 identifies loss and delay as real-world concerns [S] (Section 1).

### Inferences
- **Estimated budget for a 3pi+ team.** Assume a 20 Hz control loop (50 ms period) and states broadcast as roughly 20–50-byte ESP-NOW packets. The sourced latencies (about 1 ms on the bench; under 20–40 ms in multi-robot corridors) sit below or around one control period, and loss is a few percent. A one-packet hold-last-value scheme would hide most of it. This is my estimate, and it is exactly what a first-week measurement should confirm or refute.
- **The likely failure mode is a manufactured gap.** If measured loss and latency are negligible, studying "robustness to WiFi loss" needs injected impairment. That fails filter 3 (perturbations chosen so learning matters) and repeats rejected idea (5).
- **What is real, and worth measuring, is the loss *structure*.** Burst length, motor EMI and congestion from building WiFi at BRL decide whether "hold last value plus constant-velocity extrapolation" (the strongest simple baseline) suffices.

### Gaps
- No real-robot study of learned (GNN or MARL) communication under *measured* WiFi or ESP-NOW loss and latency was found; that search was cut by the budget.
- Per-distance WONS 2025 figures, the IIETA methodology (clock synchronisation, payload, rate) and ESP32-C3-specific figures were not obtained.
- ESP32-C3 Wi-Fi Fine Timing Measurement (FTM) ranging accuracy: not found.

---

## 4. LLM-assisted and automatic swarm design: robotics-first or method-first?

### Takeaway
LLM-for-swarm work from 2024–2026 is **method-first**. The robots are a backdrop for language-to-controller generation, and wherever real robots appear, the physics is handled by a classical safety filter or planner. Automatic (off-line) swarm design (AutoMoDe and its descendants) is the robotics-first counterpart: the reality gap is its explicit subject. Neither looks like a good fit for a one-robot MSc. I found no Hauert-lab or BRL LLM-swarm paper, though the search was incomplete.

### Cited Findings
- **LLM2Swarm** (Strobel, Dorigo & Fritz), NeurIPS 2024 Workshop on Open-World Agents; arXiv 2410.11387 [R README + S] — [README](https://github.com/Pold87/LLM2Swarm), [arXiv](https://arxiv.org/pdf/2410.11387)
  - Indirect integration: "LLMs are used to synthesize and validate the robot controllers".
  - Direct integration: "each robot locally executes a separate LLM instance".
  - Evidence is "proofs of concept (showcases)".
  - Prerequisites are an OpenAI API key, the ARGoS 3.0.0-beta59 simulator and ARGoS-Epuck, so the showcases ran on simulated e-pucks.
- **Swarm-GPT**, NeurIPS 2023 robot-learning workshop (arXiv 2312.01059) [S] — [arXiv](https://arxiv.org/pdf/2312.01059)
  - The LLM proposes music-driven waypoints, and a model-based planner guarantees collision-free, feasible motion.
  - Mean sim-to-real RMSE was 28.7 mm; it was shown at 3 live events.
- **SwarmGPT** (Schuck, Dahanaggamaarachchi et al., TUM, Schoellig group), arXiv 2412.08428; RA-L 2025 per a reference list [S] — [arXiv](https://arxiv.org/html/2412.08428v2)
  - A safety filter makes "minimal corrections" to the LLM's output.
  - It ran up to 200 drones in simulation and up to 20 real drones.
- **GenSwarm**: LLM-generated multi-robot code policies deployed on real robots; npj Robotics according to reference lists [S] — [arXiv 2503.23875](https://arxiv.org/pdf/2503.23875), [npj Robotics](https://www.nature.com/articles/s44182-025-00065-w)
  - The code README [R] says the project "heavily relies on" in-house "Cluster Robots", a "Vicon System … for precise localization" and "Custom Servers", on ROS Noetic (end-of-life ROS 1).
  - The instruction-to-code pipeline is reproducible with an LLM API key, but "code deployment to robots and their execution, cannot be reproduced" without that hardware.
  - Branch names include `feature-orca` and `feature-comparative` [M] — [GenSwarm repo](https://github.com/WindyLab/GenSwarm)
- **CommandSwarm** (arXiv 2605.07764, May 2026, PREPRINT) [S] — [arXiv](https://arxiv.org/html/2605.07764)
  - It turns natural language into XML behaviour trees; "the LLM proposes a tree, while the parser and whitelist decide whether the tree is executable".
  - Fine-tuning a 10B model on 2,063 synthetic pairs raised syntactic validity from 0% to 72%.
- Survey: "Large Language Models for Multi-Robot Systems" (arXiv 2502.03814) [T] — [arXiv](https://arxiv.org/html/2502.03814v5)
- A 2025 Konstanz master's thesis concludes that LLMs are viable for natural-language-to-swarm-controller generation, but "further research with larger models is needed" [S] — [KOPS](https://kops.uni-konstanz.de/handle/123456789/75647)
- **BRL behaviour-tree lineage (no LLM)**
  - Jones, Studley, Hauert & Winfield, "Evolving behaviour trees for swarm robotics" (DARS 2016 proceedings; on Kilobots, "which have very limited memory") [S] — [Bristol portal](https://research-information.bris.ac.uk/en/publications/evolving-behaviour-trees-for-swarm-robotics(4a4cbff7-7446-451a-beb4-e282131167d6).html)
  - Hogg, Hauert, Harvey & Richards, "Evolving behaviour trees for supervisory control of robot swarms", *Artificial Life and Robotics* 2020 [S] — [Springer](https://link.springer.com/article/10.1007/s10015-020-00650-2)
  - A 2019 Bristol study evolved human-understandable swarm behaviours on the robots themselves rather than on an external computer [S; source URL ambiguous among the result set] — [Logistics Manager](https://www.logisticsmanager.com/university-of-bristol-lays-out-a-case-for-swarm-robotics)
- **Automatic design** (Birattari / IRIDIA)
  - Modular methods transfer better than neuroevolution (Section 2).
  - Another preprint assembles automatically generated neural "behaviour repertoires" into probabilistic finite-state machines and tests them on real robots [S] — [ResearchGate](https://www.researchgate.net/publication/373542905_Automatic_modular_design_of_robot_swarms_based_on_repertoires_of_behaviors_generated_via_novelty_search)
  - Automatic design of stigmergy-based behaviours (*Communications Engineering* 2024) [T] — [Nature](https://www.nature.com/articles/s44172-024-00175-7)
  - Transferability of control software across platforms (arXiv 2305.16126) [T] — [arXiv](https://arxiv.org/pdf/2305.16126)
- **Hunt, "Temperament Engineering: Designing Strategic Behavioural Diversity in Robot Swarms"** (arXiv 2609.29423, Sep 2026, PREPRINT) [S] — [arXiv](https://arxiv.org/abs/2609.29423)
  - It proposes designing the *distribution* of behavioural temperaments across the swarm as "an offline, anticipatory design input".

### Inferences
- **Classification against the eight filters:**
  - LLM2Swarm (simulation only; robots as backdrop) and CommandSwarm (an NL→BT pipeline) fail filters 1 and 4: the method is chosen for recency, not the robot's problem.
- GenSwarm's real-robot results rest on Vicon localisation and in-house robots (README). The LLM writes code on top of perfect state, so the robotics is solved by infrastructure.
  - SwarmGPT and Swarm-GPT have a real-robot application (choreography), but the robotics is solved by the safety filter; the LLM is a design interface.
  - GenSwarm is the most robotics-relevant, because generated code meets real physics, but its details were not verified.
- **The only robotics-first LLM question is narrow:** "do LLM-generated controllers cross the reality gap better or worse than evolved or modular ones?" It needs a swarm (filter 6) and has no structural reason to expect LLMs to help (filter 4).
- **Temperament engineering is design-of-heterogeneity, not LLM work.** It is relevant to BRL supervision (Hunt) but has no real-robot evidence yet.

### Gaps
- Hauert-lab or BRL LLM-swarm work: none found; the search was incomplete. "No overlap found" is not established absence.
- GenSwarm's real-robot platform, team size and success rates were not verified (the dedicated search did not run).
- No AutoMoDe paper from 2025–2026 surfaced; the search was incomplete.

---

## 5. Bristol Robotics Laboratory swarm lineage and current work (people, papers, platforms, 2024–2026)

### Takeaway
BRL swarm work in 2024–2026 has two methods-level centres:
- **Hauert lab:** fault detection and mitigation for collective transport and intralogistics, moving from data-driven metrics (RA-L 2022) through neuroevolution (EvoApps 2025) to MARL (ANTS 2026), plus distributed spatial awareness and large public human-swarm events.
- **Hunt lab (with O'Dowd):** small robot teams (4–10), behavioural heterogeneity and temperament, patrol, and emitter localisation (SSRR 2025, simulation).

The flagship platform is DOTS: 20 robots, private 5G, motion capture, portal access per the 2022 preprint. A 2023 Bristol MSc dissertation used DOTS robots. Winfield's recent output found here is governance and ethics, which the student does not want.

### Cited Findings
**Hauert lab (Professor of Swarm Engineering; "a team of 20 researchers working on making swarms for people")** [S] — [Thinking Digital](https://thinkingdigital.co.uk/?p=3784), [Bristol profile](https://www.bristol.ac.uk/people/person/Sabine-Hauert-f714ce91-f09a-47e1-b573-f4e120cac448/)
- **DOTS** (Jones, Milner, Sooriyabandara & Hauert; Bristol with Toshiba Research Europe), arXiv 2203.13809, **PREPRINT** [S] — [arXiv](https://arxiv.org/abs/2203.13809)
  - "20 fast agile robots with high sensing and computational performance, and real-world payload capability."
  - An arena "equipped with private 5G, motion capture, multiple cameras", "openly accessible via an online portal".
  - A platform-agnostic develop, simulate and deploy pipeline, demonstrated on swarm logistics (search and retrieve cargo carriers).
  - Follow-on papers model the robot as a 2 kg, 250 mm disc moving holonomically at up to 1 m/s.
- A University of Bristol MSc Biorobotics dissertation on swarming intralogistics built a mixed-reality app for DOTS robots (2023) [M] — [GitHub repository](https://github.com/Massittha/Mixed-Reality-application-for-DOTS-robots)
- **Fault-tolerance line**
  - Lee, Milner & Hauert, "A Data-Driven Method for Metric Extraction to Detect Faults in Robot Swarms", *IEEE RA-L* 7(4):10746–10753, 2022 [S] — [dblp](https://dblp1.uni-trier.de/rec/conf/tas/LeeH23.html)
  - Lee & Hauert, "A Data-Driven Method to Identify Fault Mitigation Strategies in Robot Swarms", ANTS 2024 [S] — [Springer](https://link.springer.com/chapter/10.1007/978-3-031-70932-6_2)
  - Lee & Hauert, "Evolving Dynamic Fault Mitigation Strategies in a Robot Swarm for Collective Transport", EvoApplications 2025, LNCS 15612, pp. 305–322 [S] — [Springer](https://link.springer.com/chapter/10.1007/978-3-031-90062-4_19), [Bristol portal](https://research-information.bris.ac.uk/en/publications/evolving-dynamic-fault-mitigation-strategies-in-a-robot-swarm-for/)
    - No separate fault-detection stage. An evolved "mitigation module" maps locally observed metrics to mitigation actions taken by faulty *and* healthy robots.
    - It handles six fault types independently and beats a no-mitigation baseline.
  - **Alboraei, Hauert & Lee, "Multi-Agent Reinforcement Learning of a Fault-Robust Controller in an Intralogistics Robot Swarm", ANTS 2026, LNCS 16515** (online 22 May 2026; oral session 9 Jun 2026) [S] — [Springer](https://link.springer.com/chapter/10.1007/978-3-032-26123-6_15), [Zenodo](https://zenodo.org/records/22870565)
    - A shared recurrent policy trained with centralised training and decentralised execution maps local state to predefined mitigation actions, without explicit fault detection, on box transport.
    - It is "more robust to faults than a non-learning baseline" and generalises to more faults than seen in training.
  - Milner, Sooriyabandara & Hauert, "Swarm Performance Indicators: metrics for robustness, fault tolerance, scalability and adaptability" (arXiv 2311.01944) [T] — [arXiv](https://ar5iv.labs.arxiv.org/html/2311.01944)
  - The line is cited by others, e.g. Carminati et al., AAMAS 2024 [S] — [AAMAS PDF](https://ifaamas.org/Proceedings/aamas2024/pdfs/p253.pdf)
- **Spatial awareness.** Jones & Hauert, "Distributed spatial awareness for robot swarms", *Autonomous Robots* 49(4):41 (2025), plus a DARS chapter published 2 Nov 2025. Characterised in simulation, with two example algorithms [S] — [Springer journal](https://link.springer.com/article/10.1007/s10514-025-10228-1), [PMC](https://pmc.ncbi.nlm.nih.gov/articles/PMC12640350/)
- **Human-swarm interaction**
  - MOSAIX, robot tiles for social human-swarm interaction (ICRA 2022); "Deliberative Democracy with Robot Swarms" (IROS 2022); "Self-Organised Saliency Detection and Representation in Robot Swarms" (RA-L 2021); all with O'Dowd as co-author [S] — [O'Dowd profile](https://bristol.ac.uk/people/person/Paul-O'Dowd-d54e9ad6-41de-4eef-81c6-1ee227ced8dc)
  - "Express Yourself…" (May 2025): 63 MOSAIX robots used as smart sticky notes at a science museum [S] — [Hauert research works](https://www.researchgate.net/scientific-contributions/Sabine-Hauert-2223852616)
- **Assurance and governance (flagged as not wanted by the student)**
  - Winfield, Swana, Ives & Hauert, "On the ethical governance of swarm robotic systems in the real world", *Phil. Trans. R. Soc. A* 2025 [S] — [Bristol portal](https://research-information.bris.ac.uk/en/publications/on-the-ethical-governance-of-swarm-robotic-systems-in-the-real-wo/)
  - "Autonomous Robotic Swarms: A Corroborative Approach for V&V" (ERAS 2025) [S]
  - AERoS [T] — [Springer](https://link.springer.com/chapter/10.1007/978-3-031-40953-0_28)
  - "Search Space Illumination of Robot Swarm Parameters for Trustworthy Interaction" [T] — [Springer](https://link.springer.com/chapter/10.1007/978-3-031-51497-5_13)
- Decentralised negotiation for multi-object collective transport (Herranz, Hauert & Jones, 2022) [T, cited in] — [arXiv 2609.05331](https://arxiv.org/pdf/2609.05331)
- A Bristol project on trustworthy robot swarms for city-scale logistics, 2025–2030 [S; funder and details not verified]. A 2026 Hauert talk: "Out-of-the-box robot swarms for everyday applications" [S].

**Hunt lab (Edmund R. Hunt, Senior Lecturer in Robotics)** [S] — [Bristol portal](https://research-information.bris.ac.uk/en/persons/edmund-r-hunt/), [Hunt Lab people](https://www.huntlab.uk/people/)
- Research links collective animal behaviour with swarm robotics, for environmental monitoring and human–robot teaming.
- The lab works with "smaller numbers of robots (e.g. 4-10)" to make real-world deployment practical.
- **Morris, Pelham & Hunt, "Spatially Intelligent Patrol Routes for Concealed Emitter Localization by Robot Swarms", SSRR 2025, pp. 108–114** (arXiv 2510.14018, 15 Oct 2025) [S] — [arXiv](https://arxiv.org/abs/2510.14018v1)
  - Differential evolution designs geometric patrol routes for a **4-robot** swarm, with no prior knowledge of emitter location, power or frequency.
  - Triangular routes are "the most efficient per meter".
  - Directional antennas achieved 98.75% detection against 80.25% for omnidirectional.
  - Mean errors were 1.01–1.30 m (directional) and 1.67–1.90 m (omnidirectional); omnidirectional failures clustered at the map periphery.
  - **Simulation only.**
- York, Madin, O'Dowd & Hunt, ICDL 2024: latent-inhibition heterogeneity in patrol, simulation [S] (Section 2).
- "Heterogeneity in Multi-Robot Environmental Monitoring for Resolving Time-Conflicting Tasks" [T] — [ResearchGate](https://www.researchgate.net/publication/398512956_Heterogeneity_in_Multi-Robot_Environmental_Monitoring_for_Resolving_Time-Conflicting_Tasks)
- Temperament engineering (Sep 2026 PREPRINT) [S] (Section 4).
- Human–robot trust: movement synchrony in one-human, two-robot teams correlates with self-reported trust [S] — [Bristol portal](https://research-information.bris.ac.uk/en/persons/edmund-r-hunt/)

**Paul O'Dowd** (Senior Lecturer, School of Engineering Mathematics and Technology; core team of the Bristol robotics group) [S] — [profile](https://bristol.ac.uk/people/person/Paul-O'Dowd-d54e9ad6-41de-4eef-81c6-1ee227ced8dc)
- Co-supervises PhD students Connor York (with Hunt) and Jay Hamill.
- Co-authored MOSAIX, deliberative democracy and saliency detection, and an ICRA 2025 paper with Alhafnawi, Hunt, Lemaignan and Hauert.
- Has robotic drawing and painting work. Earlier swarm/embodied-evolution work was not verified in this session.

### Inferences
- **For a learning-oriented student, the Hauert fault-mitigation line is the most active BRL multi-robot *learning* thread** (neuroevolution, then MARL, in 2025–2026). Its open methods-level question is real-robot transfer. Feasibility hinges on DOTS access (unconfirmed for 2027).
- **The Hunt lab's small-team (4–10) focus best matches what one student can field or borrow.** Its key 2025–2026 results (SSRR 2025, ICDL 2024) are simulation-only, so physical validation is open (no overlap found, which is not established novelty).
- **O'Dowd bridges the Hunt and Hauert labs and the student's own MSc teaching team.** I have not verified that he runs the 3pi+ RSS unit.

### Gaps
- Not found (search budget exhausted):
  - Winfield's 2024–2026 internal-model or swarm technical papers, and whether he is still active.
  - O'Dowd's link to the RSS 3pi+ unit.
  - The robot platform the Hunt lab uses.
  - Jonathan Lawry's collective-learning work: the search returned nothing relevant.
- Whether DOTS is still portal-accessible in 2026 and open to MSc students in 2027: unknown.
- Onboard evolution of behaviour trees on robots (Jones, Winfield, Hauert & Studley, 2018–2019) and the "Xpuck" platform were not verified in this session (recalled lineage only).

---

## 6. Practical routes to multi-robot experiments for a student who owns ONE robot

### Takeaway
The Robotarium is the strongest one-robot route in 2026:
- **It is active.** The simulator was pushed in August 2026 and server libraries are dated 2026.
- **It is moving to the GTernal robot.**
- **Its 2026 API exposes onboard sensing,** not just motion-capture poses: 7 distance sensors, IMU, magnetometer, encoders.
- **It accepts PyTorch policies,** through JaxRobotarium's deploy path.

Borrowing 4–6 3pi+ robots is the hands-on route, but the 3pi+ has no inter-robot sensing, so the overhead camera must be ground truth only, never a policy input. Two further routes are time-multiplexing a single robot for static-environment sensing tasks, and BRL facilities (DOTS) subject to access. GPU simulators do not model the dynamics that matter for a 3pi+ (acceleration limits, motor lag, contact) without additions.

### Cited Findings
**Robotarium status and capabilities**
- Free "for academic research or educational purposes" worldwide; users test in simulation, then submit through the web [S] — [Robotarium paper (ResearchGate)](https://www.researchgate.net/publication/363585954_The_Robotarium_Globally_Impactful_Opportunities_Challenges_and_Lessons_Learned_in_Remote-Access_Distributed_Control_of_Multirobot_Systems), [ICRA 2017](http://ames.caltech.edu/robotarium-icra_2017.pdf)
- An October 2025 Georgia Tech podcast reports more than 16,500 experiments "submitted from every continent except Antarctica" [S] — [GT AI podcast](https://georgiatech.ai/2025/10/30/ep12swarms/)
- Harish Ravichandar's research statement (PDF created 29 Jul 2026; read in full) describes MARBLER as "an open-source, zero-cost, and turnkey framework for standardized hardware evaluation … by leveraging Georgia Tech's Robotarium". That is the most recent first-party sign that remote access remains free [R] — [research statement 2026](https://github.com/GT-STAR-Lab/gt-star-lab.github.io/blob/main/team/pdfs/harish-ravichandar-research-statement-2026.pdf)
- Fleet size conflicts across sources: 20, 22, or "up to 25 GRITSBot X" (2021), against "close to 100" ground and aerial robots in a press interview [S] — [2021 presentation](https://seattlerobotics.org/wp-content/uploads/2021/11/2021-04-17_RobotariumPresentation.pdf), [Digital Trends](https://www.digitaltrends.com/cool-tech/georgia-tech-robotarium/)
- **2026 activity** [M] — [robotarium org](https://github.com/robotarium)
  - Python simulator pushed 4 Aug 2026; MATLAB simulator pushed 23 Apr 2026.
  - New GTernal hardware repository (Raspberry Pi plus Teensy) pushed 26 Sep 2025.
- **Server environment** (`libraries.txt`): torch 2.10.0, numpy 2.2.6, scipy 1.15.3, gym 0.26.2 and `gtsam-develop` (Feb 2026 build) are present; JAX is not listed [R] — [libraries.txt](https://github.com/robotarium/robotarium_python_simulator/blob/master/libraries.txt)
- **2026 simulator constants**, "Matching ARobotarium.m" [R] — [robotarium_abc.py](https://github.com/robotarium/robotarium_python_simulator/blob/master/rps/robotarium_abc.py)
  - Time step 0.033 s; maximum linear speed 0.2 m/s; wheel radius 0.016 m; base 0.11 m; arena 3.2 × 2.0 m; at most 50 robots in simulation.
  - **7 distance sensors:** 1.2 m range, 3% fractional noise, 3.5% dropout, 1.4% outliers.
  - **Encoders:** 28 CPR with a 100.37:1 gearbox and 0.25-count noise.
  - **IMU:** accelerometer σ 0.012–0.053 m/s², gyro σ 0.0012–0.0024 rad/s, plus magnetometer and an orientation channel with σ 0.31°.
- **API methods:** `get_poses`, `get_distances`, `get_accelerations`, `get_gyros`, `get_magnetic_fields`, `get_orientations`, `get_encoders` [R]
- **Simulator dynamics:** unicycle kinematic integration with wheel-speed saturation. Accelerations are computed only to synthesise IMU readings, so **motion has no acceleration limits** [R] — [robotarium.py](https://github.com/robotarium/robotarium_python_simulator/blob/master/rps/robotarium.py)
- **Deploying learned policies:** JaxRobotarium converts Flax policies to PyTorch with safetensors; the user sets the robot count and uploads [R] (Section 1).

**Borrowed 3pi+ robots**
- 3pi+ 32U4 sensing (Pololu library README): line sensors, front bump sensors, LSM6DS33 accelerometer and gyro, LIS3MDL compass, encoders, OLED, buzzer. **There are no proximity or inter-robot sensors** [R] — [Pololu 3pi+ library](https://github.com/pololu/pololu-3pi-plus-32u4-arduino-library)
- 3pi+ price and top speed: not found (pololu.com blocked).

**Overhead tracking**
- WhyCon (circle markers) claims "millimiter precision" with "low-cost web cameras"; default marker diameter 122 mm. This is a self-claim in the README [R] — [WhyCon README](https://github.com/LCAS/whycon)
- ArUco plus webcam accuracy figures: **not found** (search budget exhausted).

**GPU simulators**
- VMAS is holonomic by default, with an optional differential-drive model and customisable drag, friction and mass [R] (Section 1).
- JaxMARL includes MPE (particles), MABrax, SMAX, JaxNav (differential drive, 2D geometric) and JaxRobotarium [R] (Section 1).

**BRL facilities**
- DOTS arena with motion capture and 5G (2022 claim) [S]; MSc precedent (2023) [M] (Section 5).
- Mixed reality (one real robot with virtual teammates) has a dedicated 2024 preprint [T] — [arXiv 2403.10996](https://arxiv.org/pdf/2403.10996)

### Inferences
- **What to ask for (all estimated, to verify with BRL):**
  - *From the MSc robot pool (likely the RSS unit team; O'Dowd unverified):* 4–6 3pi+ 32U4 OLED Standard Edition robots with 30:1 motors, matching the student's own so dynamics are comparable. Also M5Stamp C3 boards with I2C harnesses (or permission to fit them), NiMH AAA cells and chargers.
  - *Loan window and permissions:* a loan from about March to August 2027, after teaching needs end; permission to reflash firmware.
  - *Space:* an area of at least 2 × 2 m with a ceiling mount for a webcam.
  - *From BRL:* 1–2 sessions on motion capture (DOTS arena or another Vicon/OptiTrack space) to calibrate webcam tracking error, plus a named contact for DOTS access (Suet Lee or Sabine Hauert).
  - *From the Robotarium:* an account. Then confirm in week 1 that real GTernal sensor channels are returned to user code.
- **Webcam resolution (estimated by arithmetic):** a 1920-pixel-wide image spanning a 2 m arena gives about 1.0 mm per pixel. Marker position error of a few mm is plausible after calibration, but lens distortion and marker tilt dominate. The student must measure it against motion capture or a surveyed grid; no sourced figure was found.
- **Hidden-perfect-state trap.** If overhead-camera poses are fed into borrowed-3pi+ policies, every robot has global state, and decentralisation becomes a manufactured constraint (filter 3). With no inter-robot sensors, the 3pi+ can only share odometry over radio. Use the camera for evaluation only.
- **Time-multiplexing.** For static, continuously present phenomena (an RF emitter, a fixed field), one robot driving each "team member's" route in sequence gathers the same spatial samples as a simultaneous team. It cannot test interaction effects (collisions, congestion, communication). It also removes one source of realism, which must be stated.
- **Simulator fidelity.** No surveyed GPU simulator models 3pi+ motor lag, acceleration limits or wheel slip out of the box. VMAS's 2D contact physics could approximate pushing, but its parameters would need identification on the real robot. This is the same trap as rejected idea (1).

### Gaps
- Official 2026 Robotarium status (queue times, current robot count, the GRITSBot X to GTernal transition, onboard sensors returned on hardware) was not readable; the site was blocked.
- ArUco or webcam accuracy figures, 3pi+ price and speed, and ESP32-C3 board prices were not found.
- Availability of the MSc 3pi+ pool for dissertations, and BRL motion-capture access rules, are unverified (sibling task #7 covers BRL facilities).

---

## 7. Candidate dissertation questions: eight-filter verdicts, first-week kill tests, and "tempting but fails"

### Takeaway
Of the four suggested hypotheses, only **"how the reality gap scales with team size"** survives provisionally, and only through the Robotarium with JaxRobotarium, not on borrowed 3pi+ robots. **Cooperative localisation from cheap sensors** survives only as a classical-estimation project with a high first-week kill risk. **Learned versus occlusion-based transport** fails filters 3 and 6 on this hardware. **Robustness to real WiFi loss** probably fails filter 3, which a cheap measurement can settle in days.

Research also surfaced one more provisional survivor: **physical validation of Hunt-lab swarm patrol routes for RF emitter localisation** (SSRR 2025). It is robotics-first, defence-relevant and feasible with one robot, but it is not a learning project. Ranking by evidence: C1 > C2 > C3 > C4 (conditional on access) > C5 > C6. Nothing passes filter 5 yet, because no closest work could be read beyond abstract level.

### Cited Findings
- JaxRobotarium: more than 200 real deployments; the gap is small for many task–algorithm pairs, noticeable for some, and reduced by action-noise randomisation [S] (Section 1).
- The 2026 Robotarium API exposes onboard sensing, the server has torch and gtsam, and simulator motion has no acceleration limits [R] (Section 6).
- SSRR 2025 emitter localisation: 4 robots, simulation-only, with route-shape and antenna effects quantified [S] (Section 5).
- Occlusion transport is proved for convex objects; 20 e-pucks, 43/45 trials [S] (Section 2).
- ESP-NOW: bench ~1 ms one-way delay; 3–7 robots indoors at PDR >92–98% and latency <20–40 ms [S]; application-layer delivery is not guaranteed [R] (Section 3).
- BRL MARL fault-robust transport controller (ANTS 2026), simulation implied but not confirmed [S] (Section 5).
- The 3pi+ has no inter-robot sensing [R] (Section 6).

### Inferences

#### C1. Team-size scaling of the sim-to-real gap for decentralised learned policies on the Robotarium (provisional survivor; ranked first)
- **(a) Research question.** For decentralised MARL policies trained in JaxRobotarium and deployed on the Robotarium, does the sim-to-real performance gap grow with team size N (about 2→16) or with density? Is any growth removed by per-robot actuation randomisation, or is it driven by interaction effects (barrier-certificate interventions, near-collisions, congestion, timing)?
- **(b) Why now:**
  - JaxRobotarium (CoRL 2025) brought train-to-deploy down to about 10 minutes and ran more than 200 real deployments.
  - JaxMARL 0.2.0 (10 Sep 2026) lists JaxRobotarium.
  - The Robotarium simulator was updated in August 2026 (GTernal, sensor models).
  - The GT lab's July 2026 research statement says access is "zero-cost".
- **(c) Nearest work, and what remains open.**
  - JaxRobotarium (CoRL 2025; abstract read in full, sim-to-real section not read) is the nearest. **Whether it already analyses gap against N is unknown**: "no overlap found" is not novelty. Read its sim-to-real section first.
  - MARBLER (MRS 2023; supplementary read in full): teams of 4–6, mostly full-observation "communication"; real results from 10 episodes.
  - Ligot & Birattari 2020 (pseudo-reality).
  - **Overlap risk:** Ravichandar's July 2026 research statement names as a future direction that high-level multi-robot planners treat execution time and success likelihood "as direct inputs, even though these quantities emerge from embodiment, interaction, and team composition". The JaxRobotarium group may already be pursuing the interaction-driven gap. Contact them, or differentiate, for example by adding the 3pi+ cross-platform arm or genuinely local observations.
- **(d) Strongest simple baseline.** "Per-robot calibration plus action-noise randomisation removes the gap at every N" (JaxRobotarium's own fix). It might fail because interaction effects scale with neighbour count, and per-robot noise cannot represent them. Show it by plotting the gap against N with and without randomisation, against logged barrier interventions and minimum inter-robot distances. A classical controller arm (assignment, go-to-goal and barrier certificates) also runs at every N, to test whether learned policies have a *different* gap profile.
- **(e) First-week test.**
  1. Apply for a Robotarium account.
  2. Train the JaxRobotarium navigation scenario at N = 4 and N = 12 on the RTX 3060.
  3. In simulation, log the fraction of steps where barrier functions modify actions at N = 2, 4, 8 and 16.
  4. Submit 3 real runs at each of N = 4 and N = 12.

  **Kill if any of these holds:**
  - Access or queue turnaround makes about 30 to 60 real runs impossible by summer 2027 (estimated need).
  - JaxRobotarium already reports gap against N.
  - Barrier interventions are negligible at N = 16 *and* real N = 12 runs match simulation within run-to-run spread, so there is nothing to scale.
- **(f) Hardware and compute:**
  - Robotarium access: free for academic use (sourced; 2026 status unverified).
  - Robots: up to about 20 (sourced but conflicting); arena 3.2 × 2.0 m, 0.2 m/s maximum (sourced).
  - Compute: RTX 3060 for JAX training (estimated sufficient for small MLP/RNN policies; the 10-minute claim is on unspecified hardware).
  - No purchases (estimated £0). The student's own 3pi+ is optional, for a cross-platform point at N ≤ 1–5 if robots are borrowed.
- **(g) Headline figure and outcomes.**
  - Headline: normalised sim-to-real return gap against N, for 2–3 tasks × 2 algorithms plus the classical controller, with the share explained by interaction metrics.
  - Best realistic: a density-driven scaling of the gap plus a cheap simulator fix (for example, simulated barrier-delay or interaction noise) that removes it.
  - Floor: "within Robotarium's safety stack, no measurable gap growth up to N = X". Still a quantified boundary.
- **(h) Filter-by-filter verdict:**
  1. Pass, weak-to-moderate: physical multi-robot behaviour is the subject, but mediated by the Robotarium's controllers.
  2. Pass: baseline named, with a mechanism for failure.
  3. Pass with caveat: the policies use motion-capture poses in both sim and real (the same information, nothing withheld), so the gap measured is dynamics and interaction, not perception. Because MARBLER-style scenarios concatenate all robots' observations, use neighbourhood observations to make policies N-transferable. That is justified by the scaling question, not by a manufactured need.
  4. Pass: learned policies are the object of study; JAX is justified by the number of policies needed.
  5. **Incomplete:** nearest work read at excerpt level only.
  6. Pass, conditional on Robotarium access.
  7. Pass.
  8. Pass.
- **(i) Skills built:** MARL in JAX (reuses the student's JAX skills), sim-to-real experimental design, remote-hardware logistics, statistics on small-sample real runs.
- **(j) Bristol/BRL people:** none directly; Hauert lab adjacent (reality gap, fault transfer). External: Harish Ravichandar (Georgia Tech STAR lab, MARBLER and JaxRobotarium).
- **Main risks:** it is hands-off (no building); the Robotarium fleet transition may change robots mid-project; the gap may be tiny because of the safety stack.

#### C2. Physical validation of swarm patrol-route geometry for RF emitter localisation (Hunt lab; provisional survivor; not a learning project)
- **(a) Research question.** Do the patrol-route geometries that differential evolution found best in simulation for concealed-emitter localisation by a 4-robot team keep their ranking and accuracy with real indoor 2.4 GHz RSSI sampled from moving low-cost robots? (Simulation ranking: triangle > square/hexagon; 1.67–1.90 m error with omnidirectional antennas.)
- **(b) Why now.** Morris, Pelham & Hunt, SSRR 2025 (arXiv 15 Oct 2025) is simulation-only. The Hunt lab's 2026 work continues the small-team focus.
- **(c) Nearest work.** SSRR 2025, read at abstract level; its propagation model is unknown. The classical RSSI-localisation literature was not searched (gap). Open: physical validation (no overlap found, which is not established novelty).
- **(d) Strongest simple baseline.** Fixed lawnmower or geometric routes with least-squares or particle-filter multilateration under a fitted log-distance path-loss model. It might fail because indoor multipath, robot-body shadowing and the M5Stamp C3's antenna orientation make RSSI non-monotonic in distance, so geometry optimised under idealised propagation may lose its advantage. Show it by comparing simulation-predicted against measured error per route shape. Any adaptive or learned routing is justified only if a classical information-gain planner fails.
- **(e) First-week test.** Make a Pico 2 W or spare ESP32 a fixed beacon. Drive the 3pi+ (M5Stamp C3 logging RSSI) on straight passes at several offsets, with webcam or odometry ground truth.
  - **Kill if** RSSI against distance has no usable monotonic trend beyond about 1–2 m (estimated threshold). Then every route geometry is equivalent and localisation error exceeds the arena half-width.
  - If the trend is clean, the study measures how much of the simulated accuracy survives.
- **(f) Hardware and compute:**
  - Owned: 3pi+ with M5Stamp C3, Pico 2 W, webcam (measured: in hand).
  - Single-robot time-multiplexing reproduces the 4-robot sample pattern for a static emitter (estimated valid; it loses simultaneity).
  - Optional directional antenna (price not found) and 3 borrowed 3pi+ for simultaneous runs.
  - Compute trivial. Estimated spend £0–£50 (unsourced).
- **(g) Headline figure and outcomes.**
  - Headline: real against simulated localisation error and detection rate per route geometry and antenna type.
  - Best: the geometry ranking holds, with a quantified degradation factor.
  - Floor: a measured boundary that "moving-robot indoor RSSI cannot support better than X m", making geometry irrelevant. Useful to the Hunt lab either way.
- **(h) Filter-by-filter verdict:**
  1. Pass: RF sensing from a moving robot is the subject.
  2. Pass.
  3. Pass: real propagation; time-multiplexing must be stated.
  4. Pass for a classical, optimisation-based design; learning is not justified, which is a fit issue, not a filter failure.
  5. Incomplete.
  6. Pass.
  7. Pass.
  8. Pass.
- **(i) Skills built:** embedded RF logging (ESP32 promiscuous or ESP-NOW RSSI), estimation, experimental design. Defence and SIGINT-adjacent relevance (Cheltenham, Bristol defence employers).
- **(j) Bristol/BRL people:** Edmund Hunt, Adam Morris, Timothy Pelham; Paul O'Dowd co-supervises Hunt-lab students.
- **Note:** this overlaps with the active-localisation thread (sibling task #2): "when should a robot deviate from its route to reduce emitter uncertainty" is a natural extension *if* a classical information-gain baseline fails.

#### C3. Decentralised cooperative localisation from cheap sensors (survives only as classical estimation; high kill risk)
- **(a) Research question.** With only encoders, IMU and short-range distance sensing, how much does sharing relative observations reduce position drift relative to independent encoder-plus-gyro dead reckoning, as a function of team size and encounter rate? Platforms: Robotarium GTernals remotely, or borrowed 3pi+ robots with added ranging. When is it not worth the communication?
- **(b) Why now.** The 2026 Robotarium API and simulator expose encoders, IMU, magnetometer and 7 noisy ToF channels, and the server ships `gtsam-develop`. This apparently makes onboard-sensing estimation on about 20 real robots possible remotely. Hardware returning these channels is unverified.
- **(c) Nearest work.** Not read: the cooperative-localisation and UWB-swarm literature searches never ran, so filter 5 fails for now.
- **(d) Strongest simple baseline.** Independent dead reckoning, with motion capture as evaluation ground truth. It might fail because heading drift grows without bound. But the student's own earlier finding (rejected idea 4) was that 3pi+ dead reckoning "may be accurate enough", which is a strong prior that the baseline will *not* fail over short tasks.
- **(e) First-week test.** Simulate 8 GTernals for 5 minutes with the sourced Robotarium noise models, and log the student's 3pi+ dead-reckoning drift against webcam ground truth over the same duration. **Kill if** the drift stays below about half a robot diameter (about 5 cm; estimated threshold) over the task duration.
- **(f) Hardware:** Robotarium (free, sourced); own 3pi+ (measured: in hand). Any UWB or extra ranging hardware: price and accuracy not found.
- **(g) Headline figure.** Drift reduction (%) against N and encounter rate. Floor: "dead reckoning suffices below T minutes".
- **(h) Filter-by-filter verdict:**
  1. Pass.
  2. Pass.
  3. Pass, if motion capture is kept for evaluation only (standard practice, not withholding).
  4. Pass for EKF or factor-graph estimation; **fails for any learned variant**.
  5. Fail as of now.
  6. Conditional.
  7. Pass.
  8. Pass.
- **(i) Skills built:** estimation, factor graphs (gtsam), multi-robot data association.
- **(j) Bristol/BRL people:** Jones and Hauert (distributed spatial awareness).

#### C4. Real-robot transfer of BRL's MARL fault-robust transport controller (conditional on DOTS access)
- **(a) Research question.** Does the CTDE recurrent mitigation policy of Alboraei, Hauert & Lee (ANTS 2026) keep its advantage over evolved and non-learning mitigation when moved from simulation to physical DOTS robots, and for which fault types (stalled wheel, sensor dropout, slow robot) does it break?
- **(b) Why now.** ANTS 2026 (published May 2026); EvoApps 2025.
- **(c) Nearest work.** Both papers above, read at abstract level; the simulator and fault models are unknown.
- **(d) Strongest simple baseline.** A hand-coded rule ("if my progress < threshold, stop pushing and yield"), or the evolved module. It might fail because faults are not locally distinguishable from congestion or contact, which is the papers' own rationale for detection-free learned mitigation.
- **(e) First-week test.** Confirm DOTS access and the sim-to-hardware pipeline with Suet Lee. Run the published controller on 3–5 DOTS robots with one injected wheel-stall fault. **Kill if** there is no access by about January 2027 (estimated), or if the non-learning baseline matches the learned policy on hardware in pilot runs.
- **(f) Hardware:** DOTS (unconfirmed); compute is a 12 GB GPU for retraining (estimated adequate).
- **(g) Headline figure.** Hardware transport success against fault type, comparing learned, evolved and rule-based mitigation. Floor: a measured reality gap of fault mitigation.
- **(h) Filter-by-filter verdict:**
  1. Pass.
  2. Pass.
  3. Pass with care: faults are realistic, but the fault set must not be chosen to favour learning.
  4. Pass: combinatorial fault space.
  5. Incomplete.
  6. **Conditional:** depends on access.
  7. Pass.
  8. Pass.
- **(i) Skills built:** MARL, hardware swarm experimentation, fault injection.
- **(j) Bristol/BRL people:** Suet Lee, Sabine Hauert, Youssef Alboraei.
- **Main risk:** it is the host lab's own agenda; it only works if they want it.

#### C5. Robustness of learned decentralised policies to real ESP-NOW loss and latency (probably fails filter 3; cheap to settle)
- **(a) Research question.** On 3–5 3pi+ robots sharing state over ESP-NOW, does a learned GNN or MARL policy degrade more or less than a classical controller using hold-last-value plus constant-velocity extrapolation under *measured* loss and latency?
- **(b) Why now.** Weak: there is no new enabling release. The ESP-NOW v2.0 1470-byte payload in ESP-IDF is the only recent change found.
- **(c) Nearest work.** Blumenkamp 2022, Gielis 2022 and IDEA 2026, all at excerpt level only.
- **(d) Strongest simple baseline.** Hold-last-value plus extrapolation. It fails only if loss is bursty and long.
- **(e) First-week test.** Buy 2–3 more ESP32-C3 boards (price not found; the Pico 2 W's CYW43439 radio is not ESP-NOW-compatible, so it cannot serve). Broadcast 50-byte states at 20 Hz with sequence numbers and timestamps, on robots with motors running, across 0.5–3 m in the BRL lab at busy times. Log the packet delivery ratio, latency distribution and burst lengths. **Kill if** PDR ≥ 99%, p99 latency ≤ 10 ms, and bursts ≤ 2 packets (estimated thresholds against a 50 ms period).
- **(h) Filter-by-filter verdict:**
  1. Pass.
  2. At risk.
  3. **Likely fail** (needs injected loss).
  4. At risk.
  5. Incomplete.
  6. Pass.
  7. Pass.
  8. Pass: a measured "comm is not the bottleneck" result is useful but thin for a whole dissertation.
- **(i) Skills built:** embedded networking.
- **(j) Bristol/BRL people:** none specific found.
- **Note:** run the measurement anyway as a cheap by-product; it informs C1 and C2.

#### C6. Behavioural heterogeneity in physical patrol on a taped graph (borderline)
- **(a) Research question.** Does the ICDL 2024 simulated result (mostly high-latent-inhibition robots plus one low-LI robot performs best) survive on 3–5 3pi+ robots that line-follow a taped graph? On real robots, intersection misdetection and congestion add unplanned heterogeneity.
- **(c) Nearest work.** York et al. 2024 and Hunt 2026, at abstract level.
- **(d) Strongest simple baseline.** A single-cycle (TSP-tour) patrol strategy (named from domain knowledge; not sourced this session).
- **(e) First-week test.** On one robot, measure intersection-detection failures and the coefficient of variation of edge traversal times on a 6–8-node taped graph. **Kill the "reality" angle if** both are near zero: execution then equals the simulator's abstraction.
- **(h) Filter-by-filter verdict:**
  1. **Borderline:** the graph abstraction makes the robot a backdrop.
  2. Pass.
  3. Risk: anomaly events must be synthetic.
  4. **Borderline:** the "learning" is a probabilistic rule.
  5. Incomplete.
  6. Pass with borrowed robots.
  7. Pass.
  8. Pass.
- **(j) Bristol/BRL people:** Hunt, York, Madin, O'Dowd.

#### Tempting but fails (named filter)
- **Learned versus occlusion-based collective transport on 3pi+.**
  - **Filter 3:** the 3pi+ cannot perceive goal occlusion, so occlusion must be emulated from an overhead camera, which hands every robot global state, and a centralised planner then beats both.
  - **Filter 6:** 3–6 robots against the 20 used by Chen et al.
  - **Filter 2:** the occlusion strategy is provably correct for convex objects, so learning wins only on objects chosen to break it, which is also a filter 3 problem.
  - Learned transport is already being done in simulation at BRL (ANTS 2026). It is only rescued as C4 (DOTS).
- **GNN or MARL policies on borrowed 3pi+ using overhead-camera "virtual sensors".** Filter 3: decentralisation becomes a manufactured constraint over hidden perfect state.
- **LLM-generated swarm behaviour trees or controllers** (LLM2Swarm or CommandSwarm style). Filters 1 and 4 (method-first), and 6 (needs a swarm).
- **Learned communication scheduling (when to talk) over ESP-NOW.** Filter 2 (threshold event-triggering is a strong simple baseline) and filter 3 (bandwidth is not binding at N ≤ 6).
- **Byzantine or faulty-robot resilient learned consensus versus W-MSR.** Filters 2 and 3: with all-to-all broadcast in small teams, classical resilient consensus works, and the attack surface collapses as in rejected ideas (2) and (5).
- **AutoMoDe-style automatic design for a 3pi+ swarm.** Filter 6 (no proximity or inter-robot sensing; too few robots) and filter 5 (IRIDIA dominates).
- **Learned task allocation versus CBBA or response thresholds with 3–6 robots.** Filter 2: auction or greedy allocation is near-optimal at this scale. This is my inference; no real-robot comparison was found.
- **Group-relative objectives (TailRL/MaxRL) for multi-robot teams.** Filter 4; repeats rejected idea (3).
- **Physical validation of robot-policy watermark detectability through a webcam** (Prorok lab, arXiv 2512.15379). Single-robot, and only borderline on filter 1 (a security question with the robot as medium). Outside this task's scope; flag it to the wildcard researcher (task #6).

### Gaps
- **Filter 5 is incomplete for every candidate.** No closest-work paper could be read beyond excerpts. Before any commitment, the student should read in full:
  - JaxRobotarium (C1).
  - Morris, Pelham & Hunt, SSRR 2025 (C2).
  - Alboraei, Hauert & Lee, ANTS 2026 and Lee & Hauert, EvoApps 2025 (C4).
  - York et al., ICDL 2024 (C6).
- **Searches planned but not run** (budget exhausted):
  - GenSwarm details.
  - Prorok-lab 2025 real-robot MARL follow-ups.
  - Learned communication under real loss.
  - ArUco accuracy.
  - Official 2026 Robotarium status.
  - Prices of 3pi+, ESP32-C3 and UWB modules.
  - ESP32 FTM accuracy.
  - Winfield, Lawry and the Hunt-lab platform.
  - Real-robot learned-versus-classical task allocation.
  - Kilobot and e-puck MARL from 2024–2026.
- **Sources I could not access** (HTTP 403 at the egress proxy): arxiv.org and mirrors, ieeexplore.ieee.org, link.springer.com, researchgate.net, semanticscholar.org, openreview.net, proceedings.mlr.press, research-information.bris.ac.uk, bristol.ac.uk, hauertlab.com, huntlab.uk, brl.ac.uk, robotarium.gatech.edu, docs.espressif.com, pololu.com, iieta.org, dl.ifip.org, huggingface.co, alphaxiv.org, archive.org. Links to these hosts above were cited from search excerpts, not read.
- **Read directly [R]:**
  - READMEs: MARBLER, JaxRobotarium (with its deploy README), rl_multi_agent_passage, VMAS, BenchMARL, JaxMARL, Robotarium Python simulator, GTernal, LLM2Swarm, Proroklab sanity_receiver, RobotPolicyWatermarking, cambridge-robomaster (plus docs and `paper/robot_comparison.csv`), HetGPPO, d4orm and agile-mapf, GenSwarm, WhyCon, the Pololu 3pi+ library.
  - Source files: Robotarium `robotarium_abc.py`, `robotarium.py`, `sensors.py` and `libraries.txt`; GTernal `env_variables.sh`; ESP-IDF `esp_now.rst`.
  - Documents read in full via `git clone`: the MARBLER supplementary PDF (5 pages) and Ravichandar's research statement (6 pages, Jul 2026). The GT STAR lab paper posts for JaxRobotarium (full abstract and CoRL 2025 venue) and MARBLER were also read.
  - Release data: PyPI for VMAS, BenchMARL and JaxMARL.
- **GitHub repository searches** for code from SSRR 2025 (emitter patrol), the BRL fault-mitigation papers and the occlusion-transport extensions returned nothing.
