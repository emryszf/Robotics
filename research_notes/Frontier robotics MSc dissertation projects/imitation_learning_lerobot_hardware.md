# Imitation learning and generative policies on low-cost hardware; LeRobot / SO-101 in the UK; sub-£200 sensors for the Pololu 3pi+ (research notes, as of Saturday 10 October 2026)

Access legend for every citation below: **[R]** = primary page read in full through a GitHub-hosted copy (github.com and raw.githubusercontent.com were the only hosts the fetch tool could reach; the tool returns a model-written summary, so spot-check numbers before the final report). **[V]** = raw file read verbatim with curl from raw.githubusercontent.com, not model-summarised; added after a coordinator tip. **[P]** = partly read. **[S]** = known only from a web-search result summary; the page itself was not opened, so treat it as abstract-level or weaker. **[A]** = abstract-level, via a third party's summary. **[T]** = title or listing only. "PREPRINT" = arXiv-only as far as could be checked. Hugging Face blog posts are cited from their GitHub copies (canonical URL huggingface.co/blog/&lt;slug&gt;, which was blocked). The session's egress policy blocked arxiv.org, huggingface.co, pmc.ncbi.nlm.nih.gov, every vendor site and *.github.io. The shared web-search budget (200 searches per turn across all agents) ran out after this researcher's 23rd search. All prices are snapshots from the search index, not live checkout prices on 10 Oct 2026. The full access log is at the end of Section 7.

---

## 1. LeRobot's current status: latest release, supported robots and policies, documentation quality, community

### Takeaway
On 10 Oct 2026 the latest LeRobot release is **v0.6.1 (3 Aug 2026)**, which followed v0.6.0 (6–7 Jul 2026). The library now covers ACT, Diffusion and VQ-BeT; about a dozen VLAs (π0, π0.5, π0-FAST, SmolVLA, GR00T N1.7, X-VLA, …); world-model policies; reward models; HIL-SERL; and real-time chunking (RTC). SO-100/101 remains the reference low-cost arm. The docs are broad, but every minor release has broken something. The newest hard community figures I could source date from June 2025.

### Cited Findings
**Releases and dates**
- The releases page, fetched 10 Oct 2026, lists newest first: v0.6.1 "03 Aug", v0.6.0 "06 Jul", v0.5.1 "07 Apr", v0.5.0 "09 Mar", v0.4.4 "27 Feb", v0.4.3 "22 Jan". The page shows no years, and nothing newer than v0.6.1 is listed — [LeRobot releases](https://github.com/huggingface/lerobot/releases) [P: first 100k of 137k characters]
- The HF blog index gives the years: lerobot-release-v040 **Oct 24, 2025**; lerobot-release-v050 **Mar 09, 2026**; lerobot-release-v060 **Jul 07, 2026**; grabette Jul 21, 2026; smolvla Jun 3, 2025; async-robot-inference Jul 10, 2025; lerobot-datasets-v3 Sep 16, 2025 — [HF blog _blog.yml](https://github.com/huggingface/blog/blob/main/_blog.yml) [R]
- Conflict: a search summary said v0.5.0 dated "from April 2026" — [TechApple v0.6.0 article](https://global.techapple.com/2026/07/lerobot-v0-6-0-closing-the-robot-learning-loop/) [S]. The GitHub release page (09 Mar) and the blog index (Mar 09, 2026) both disagree with it. 7 April is the date of v0.5.1.
- **v0.6.1**: LeKiwi support in `lerobot-replay` and the rollout/eval CLI; Dynamixel tables for XH540-W150, XC330-T288 and XC330-T181; wall-x and xvla refactored; **breaking change**: `lerobot.types` renamed to `lerobot.lerobot_types`; "configurable SO follower P coefficient"; RealSense exposure, gain and white-balance options; DataLoader multiprocessing defaults to `spawn` — [releases](https://github.com/huggingface/lerobot/releases) [P]
- **v0.6.0** ("Imagine, Evaluate, Improve"):
  - World-model policies: VLA-JEPA, LingBot-VA, FastWAM.
  - VLAs: GR00T N1.7 (replaces N1.5), MolmoAct2, EO-1, Multitask DiT, EVO1.
  - Unified reward-models API (`lerobot.rewards`): HIL-SERL classifier, SARM, Robometer, TOPReward.
  - `lerobot-rollout` CLI with strategies `base`, `sentry`, `highlight`, `episodic` and `dagger`, plus an `intervention` flag on correction frames.
  - RTC supported by EVO1 and by the rollout inference backends.
  - Six simulation benchmarks: LIBERO-plus, RoboTwin 2.0, RoboCasa365, RoboCerebra, RoboMME, VLABench.
  - Infrastructure: FSDP; HF Jobs cloud training; 12-bit depth video; LeLab browser UI; Isaac Teleop for the SO-101; async vectorised eval "up to 2x faster"; about 40% fewer base dependencies.
  - Source: [v0.6.0 blog (GitHub copy)](https://github.com/huggingface/blog/blob/main/lerobot-release-v060.md) [R]
- **v0.6.0 breaking changes**: `pip install lerobot` no longer pulls in dataset or training dependencies (use extras such as `lerobot[training]`); minimum PyTorch 2.7; `eval_freq` renamed `env_eval_freq`; the `sac` policy type is replaced by `gaussian_actor` under a rebuilt RL API; GR00T N1.5 users must pin `lerobot==0.5.1`; diffusion and VQ-BeT now load pretrained vision-encoder weights by default — [releases](https://github.com/huggingface/lerobot/releases) [P]
- **v0.5.1**: multitask Diffusion Transformer policy; relative actions for pi0, pi0.5 and pi0_fast; **DAgger-style human-in-the-loop data collection**; `LeRobotDataset` split into a reader and a writer — [releases](https://github.com/huggingface/lerobot/releases) [P]
- **v0.5.0**:
  - Robots: Unitree G1 (first humanoid); OpenArm and OpenArm Mini; EarthRover (first mobile robot); OMX arm; Damiao and RobStride CAN motors. "We've unified the SO-100 and SO-101 implementations into a single, cleaner codebase".
  - Policies: Pi0-FAST; **RTC for flow-matching policies** (`--policy.rtc_config.enabled=true`); Wall-X; X-VLA; SARM; PEFT/LoRA.
  - Claims "10x faster image training" and "3x faster encoding", with no baseline stated.
  - "over 200 merged PRs and over 50 new contributors since v0.4.0"; LeRobot paper accepted at ICLR 2026.
  - Source: [v0.5.0 blog](https://github.com/huggingface/blog/blob/main/lerobot-release-v050.md) [R]
- **v0.4.0 (Oct 2025)**: LeRobotDataset v3.0 (chunked episodes, streaming, OXE scale ">400GB"); LIBERO ("over 130 tasks") and Meta-World ("over 50"); pi0 and pi0.5; GR00T N1.5; plugin system for third-party hardware; Reachy 2; **phone teleoperation of a follower arm, including in end-effector space**; multi-GPU training via Accelerate (time "to half with 2 GPUs and to a third with 3") — [v0.4.0 blog](https://github.com/huggingface/blog/blob/main/lerobot-release-v040.md) [R]

**Robots and policies listed now**
- Hardware in the README: SO100 (covers SO-100 and SO-101), LeKiwi, Koch, HopeJR, OMX, EarthRover, Reachy2, OpenARM, Unitree G1, reBot B601, plus gamepads, keyboards and phones. Community plugins: xArm, UR5e, Franka, AgileX Piper, Trossen WidowX, ARX5, I2RT YAM, GELLO, SpaceMouse, Meta Quest — [LeRobot README](https://github.com/huggingface/lerobot) [R]
- Policies in the README:
  - Imitation learning: ACT, Diffusion, VQ-BeT, Multitask DiT.
  - Reinforcement learning: HIL-SERL, TDMPC, and QC-FQL "coming soon".
  - VLAs: Pi0, Pi0Fast, Pi0.5, FineART-VLA, GR00T N1.7, SmolVLA, XVLA, EO-1, MolmoAct2, WALL-OSS, EVO1, DM05.
  - World models: VLA-JEPA, LingBot-VA, FastWAM, LaWAM, FLUX 3 Action.
  - Reward models: SARM, TOPReward, Robometer.
  - Source: [README](https://github.com/huggingface/lerobot) [R]
- Citable reference: Cadene et al., "LeRobot: An Open-Source Library for End-to-End Robot Learning", **ICLR 2026**, arXiv 2602.22818 — [README](https://github.com/huggingface/lerobot) [R]; paper not opened.

**Documentation**
- `docs/source` holds about 98 .mdx/.md pages, including per-policy READMEs, hardware_guide, phone_teleop, rtc, hilserl, il_robots, so100/so101, lekiwi, koch, omx, rebot_b601, earthrover_mini_plus, third_party_sensors and action_representations — [docs directory](https://github.com/huggingface/lerobot/tree/main/docs/source) [R]
- **hardware_guide.mdx** gives training-VRAM envelopes (peak, batch 8, AdamW):

  | Policy group | VRAM | Note |
  |---|---|---|
  | ACT, VQ-BeT, TDMPC | ~2–6 GB | "RTX 3060" suggested |
  | Diffusion, Multitask DiT | ~8–14 GB | |
  | SmolVLA | ~10–16 GB | |
  | pi0, pi0_fast, pi05, xvla, wall_x | ~24–40 GB | "tight at BS 1" on 24 GB |
  | GR00T, EO-1 | ~24–40 GB | |

  Wall-clock (5 epochs, ~50 episodes, ~45k frames, 640×480): ACT on an RTX 4090/3090 ~30–60 min; Diffusion ~2–4 h; SmolVLA on L4/A10G ~3–6 h; "real runs can differ by ±50%". The guide gives no inference VRAM or latency figures and no Jetson guidance — [hardware_guide.mdx](https://github.com/huggingface/lerobot/blob/main/docs/source/hardware_guide.mdx) [R]
- so101.mdx covers motor IDs, calibration and leader-arm teleop. It says nothing about 7.4 V vs 12 V, overheating or calibration drift — [so101.mdx](https://github.com/huggingface/lerobot/blob/main/docs/source/so101.mdx) [R]
- feetech.mdx is only a firmware-update tutorial. Baud is "typically 1000000"; "Do not disconnect power or USB during firmware updates, it will potentially brick the motor" — [feetech.mdx](https://github.com/huggingface/lerobot/blob/main/docs/source/feetech.mdx) [R]
- il_robots.mdx:
  - "We suggest recording at least 50 episodes, with 10 episodes per location"; defaults are 60 s per episode and 60 s per reset; examples use 30 fps.
  - "Avoid adding too much variation too quickly, as it may hinder your results."
  - A loop running at 25 Hz still produces a dataset labelled 30 Hz, and in the example the cameras were the bottleneck.
  - Support is directed to Discord.
  - Source: [il_robots.mdx](https://github.com/huggingface/lerobot/blob/main/docs/source/il_robots.mdx) [R]
- **async.mdx** gives the only inference-memory figures in the docs: "[PI0] occupies 14GB of memory at inference time, while [SmolVLA] requires only ~2GB". Other guidance:
  - `chunk_size_threshold` "values around 0.5-0.6 work well"; `actions_per_chunk` defaults to 50.
  - Larger `actions_per_chunk` "might also result in less precise actions, due to the compounding errors".
  - "reduce your fps if you consistently run out of actions in queue".
  - Source: [async.mdx](https://github.com/huggingface/lerobot/blob/main/docs/source/async.mdx) [V]
- **smolvla.mdx**:
  - "We recommend recording ~50 episodes"; the reference dataset had 50 episodes over 5 cube positions × 10. "We tried similar dataset with 25 episodes, and it was not enough leading to a bad performance".
  - Fine-tuning `smolvla_base` (450M) for 20k steps "will roughly take ~4 hrs on a single A100 GPU" (example batch 64).
  - The `lerobot-rollout` example marks RTC as optional, "use when running on low power hardware".
  - Source: [smolvla.mdx](https://github.com/huggingface/lerobot/blob/main/docs/source/smolvla.mdx) [V]
- **action_representations.mdx**: three action forms, with UMI terminology:
  - **Absolute** is the LeRobot default.
  - **Relative** (an offset from the current state at prediction time) is used by the pi family: "there is no error accumulation across chunks"; it is compatible with RTC.
  - **Delta** (step-to-step) makes errors accumulate; "UMI explicitly argues against this representation".
  - Relative conversion is implemented for pi0, pi0.5 and pi0_fast.
  - Source: [action_representations.mdx](https://github.com/huggingface/lerobot/blob/main/docs/source/action_representations.mdx) [V]
- **third_party_sensors.mdx** lists community plugins: Xense vision-based tactile sensors, Berxel depth, ZMQ networked cameras and a dummy camera. No low-cost ToF or optical-flow sensor plugins are listed — [third_party_sensors.mdx](https://github.com/huggingface/lerobot/blob/main/docs/source/third_party_sensors.mdx) [V]

**Community**
- GitHub on 10 Oct 2026: **28.1k stars, 5.9k forks, 168 watchers, 1,811 commits** on main — [LeRobot repo page](https://github.com/huggingface/lerobot) [R]
- After the June 2025 worldwide hackathon: "10k+ Discord members", "100+ Github Contributors", "2M+ datasets downloads", "10k+ datasets uploaded to the hub, the equivalent of 260 days of recording" — [Thomas Wolf on X](https://x.com/Thom_Wolf/status/1934365247357296739) [S]
- The hackathon organisation page lists winners, demo videos and a dataset collection — [LeRobot-worldwide-hackathon](https://huggingface.co/LeRobot-worldwide-hackathon) [S]. A participant's write-up says the top 30 teams were showcased and Discord was the coordination hub — [Kamath Robotics](https://kamathrobotics.com/lerobot-worldwide-hackathon) [S]
- SmolVLA was pretrained on "487 curated datasets" from the community, "focused on the SO100 robotic arm, standardized at 30 FPS", "around 10 million frames", "fewer than 30k training episodes" — [SmolVLA blog](https://github.com/huggingface/blog/blob/main/smolvla.md) [R]

### Inferences
- A dissertation should pin a LeRobot version (e.g., v0.6.1) from day one. Every minor release from v0.4 to v0.6.1 changed install extras, imports or CLI flags.
- On the student's RTX 3060 (12 GB), from the sourced envelopes:
  - ACT trains comfortably.
  - Diffusion fits only at a reduced batch (estimated).
  - SmolVLA is borderline: 10–16 GB at batch 8, so it needs a frozen vision encoder and a batch of about 2–4 (estimated).
  - π0, π0.5 and GR00T fine-tuning are out of reach locally (24–40 GB) and need HF Jobs or a BRL GPU.
- Inference on the Jetson Orin Nano 8 GB: SmolVLA (~2 GB, sourced) should fit, while π0 (14 GB, sourced) cannot. That is an estimate from the sourced memory figures; Jetson latency was not found.
- The community is large and active (monthly releases, tens of thousands of stars, thousands of Hub datasets). However, the headline success figures are mostly author-reported. Independent SO-101 evaluations are much lower (Section 3).
- Since Oct 2025, the release notes show the library moving from "collect and train ACT" towards closing the loop: DAgger and intervention logging, reward/progress models, rollout tooling and RTC. For a dissertation this lowers the engineering cost of failure-handling and human-in-the-loop experiments more than it opens new questions in itself.

### Gaps
- 2026 Discord membership, Hub dataset count and contributor count: not found (huggingface.co blocked; search budget exhausted).
- UK-specific LeRobot community (meetups, UK hackathon nodes, UK courses using the SO-101): not found.
- 2026 hackathons: none found in the one search run.
- Release notes beyond the first 100k characters of the releases page (older v0.4.x notes): not read.

---

## 2. SO-101 bill of materials; UK prices, stock and VAT (10 Oct 2026); follower-only use; what fits in about £200

### Takeaway
A two-arm SO-101 kit costs just over £200 in the UK. The cheapest listing seen was £203.99 for the motor kit without printed parts, plus £29.74 for the printed skeleton, and that kit was sold out when indexed. **With £200, the realistic purchase is a follower arm only.** LeRobot officially supports teleoperating a follower alone by phone (end-effector IK), keyboard or gamepad. All UK prices are search-index snapshots; no vendor page could be opened live.

### Cited Findings
**Bill of materials (TheRobotStudio and LeRobot docs)**
- Servo counts: a two-arm build needs 7× STS3215 at 1/345 (C001), 2× at 1/191 (C044) and 3× at 1/147 (C046). The **follower is 6× 1/345**. The leader is 1× 1/345, 2× 1/191 and 3× 1/147 — [SO-ARM100 repo](https://github.com/TheRobotStudio/SO-ARM100) [R]
- Leader ratios per joint: shoulder pan 1/191, shoulder lift 1/345, elbow 1/191, wrist flex 1/147, wrist roll 1/147, gripper 1/147. This lets the leader "sustain its own weight" and "be moved without requiring much force" — [so101.mdx](https://github.com/huggingface/lerobot/blob/main/docs/source/so101.mdx) [R]
- Power: the 7.4 V STS3215 is listed at 16.5 kg·cm stall at 6 V. The **12 V version is 30 kg·cm and needs a 12 V 5 A+ supply**. The leader is always 7.4 V, and 7.4 V was "found to be sufficient" for the follower — [SO-ARM100 repo](https://github.com/TheRobotStudio/SO-ARM100) [R]
- Electronics per arm: a Waveshare bus-servo board (set both jumpers to channel B for USB), a USB-C cable, a power supply, a table clamp and a screwdriver — [SO-ARM100 repo](https://github.com/TheRobotStudio/SO-ARM100) [R]; [so101.mdx](https://github.com/huggingface/lerobot/blob/main/docs/source/so101.mdx) [R]
- **DIY BOM totals (no printed parts, no cameras)**: two-arm **$229.88 / €226.30**; **follower only $121.94 / €124.30**. Line items include the board ($10.60 / €11.40), PSU ($10 / €15.70), USB-C ($7) and clamp ($5–9). No GBP prices or UK vendors are listed — [SO-ARM100 repo](https://github.com/TheRobotStudio/SO-ARM100) [R]
- Printing: PLA+ with TPU 95A for the compliant gripper parts; 15% infill; supports everywhere except slopes steeper than 45°; files for 220×220 mm and 205×250 mm beds. Print the gauge parts first. Printing services are linked for those without a printer — [SO-ARM100 repo](https://github.com/TheRobotStudio/SO-ARM100) [R]
- Official kit sellers named: Seeed Studio, WowRobo, RobotEd, Robonine, PartaBot, ForgeMotion Labs, RoboSEasy, NeoBot, Autodiscovery, and Phospho (SO-100 follower only). **None is UK-based** — [SO-ARM100 repo](https://github.com/TheRobotStudio/SO-ARM100) [R]
- SO-ARM101 vs SO-ARM100: wiring revised "to avoid disconnections at joint 3"; gear ratios optimised; the leader can follow the follower (for human-in-the-loop RL); 12 servos in total. The two-arm kit launched at $220 (Standard) and $240 (Pro) — [CNX Software, 2 May 2025](https://www.cnx-software.com/2025/05/02/so-arm101-open-source-dual-robotic-arm-kit-works-with-hugging-faces-lerobot/) [S]

**UK listings (search-index snapshots; live pages blocked on 10 Oct 2026)**

| Item | Vendor | Price | Stock / VAT | Source |
|---|---|---|---|---|
| SO-ARM101 Motor Kit Pro, without 3D-printed parts (12 servos and boards, both arms) | Unmanned Tech Shop (UK) | **£203.99** | **Sold out** when indexed; VAT not captured | [Unmanned Tech](https://www.unmannedtechshop.co.uk/products/so-arm101-low-cost-ai-arm-servo-motor-kit-pro-for-lerobot-without-3d-printed-parts) [S] |
| SO-ARM101 3D-printed skeleton | Unmanned Tech Shop | **£29.74** | not captured | [Unmanned Tech skeleton](https://www.unmannedtechshop.co.uk/products/so-arm101-low-cost-ai-arm-3d-printed-skeleton) [S] |
| Seeed SO-ARM101 Pro motor kit, w/o printed parts; follower is "12V 1:345" on all joints | RobotShop UK | **£271.98** sale | "tax included, shipping calculated at checkout"; stock not captured | [RobotShop UK](https://uk.robotshop.com/products/seeed-studio-so-arm101-ai-arm-servo-motor-kit-pro-lerobot-w-o-3d-printed-parts) [S] |
| Seeed SO-ARM101 3D-printed parts | RobotShop UK | **£36.92** (page also shows £59.99, inconsistent) | — | [RobotShop UK parts](https://uk.robotshop.com/products/seeedstudio-so-arm101-ai-arm-3d-printed-parts) [S] |
| Feetech STS3215 12 V 30 kg·cm, single | RobotShop UK | **£19.20** | tax included; "on-demand" item ordered from the supplier | [RobotShop UK servo](https://uk.robotshop.com/products/feetech-12v-30kgcm-magnetic-encoding-servo-sts3215) [S] |
| Pro kit w/o printed parts | Amazon UK (Global Store) | **£335.50** | includes estimated import charges | [Amazon UK](https://www.amazon.co.uk/Robotic-Arm-Kit-Servo-Motors/dp/B0FH8CPXP7) [S] |
| Assembled, 2 leaders + 2 followers | eBay UK | ≈ **£730.50** | — | [eBay UK](https://www.ebay.co.uk/itm/388694425801) [S] |

**Non-UK reference prices (UK shipping and import VAT not included)**
- Seeed Pro servo motor kit **$277.99** — [Seeed Studio](https://www.seeedstudio.com/SO-ARM101-Low-Cost-AI-Arm-Kit-p-6426.html) [S]. Hiwonder DIY kit (unassembled) **$269.99** — [Hiwonder](https://www.hiwonder.com/products/lerobot-so-101) [S]
- WowRobo's own store: two-arm unassembled **$259**; assembled leader + follower + camera **$299**; several variants sold out. An older snapshot showed $199 for printed parts plus servos — [WowRobo](https://shop.wowrobo.com/products/so-arm101-diy-kit-assembled-version-1) [S]
- OpenELAB US: **$325.99** sale ($363 regular) — [OpenELAB US](https://openelab.com/products/wowrobo-robotics-so-arm101-diykit?variant=56349875339588&country=US&currency=USD) [S]. OpenELAB EU: assembled leader + follower + camera **€604.95** — [OpenELAB EU](https://openelab.io/products/wowrobo-robotics-so-arm101-diy) [S]
- MakerMods: "SO-101 Robot Arm Kit: Leader + Follower, $300" — [MakerMods](https://www.makermods.ai/so101) [T]
- PartaBot sells "SO-ARM101 (Follower only)", price not captured — [PartaBot](https://partabot.com/products/so-arm101-follower-only) [T]. PartaBot also sells assembled versions, LeKiwi and Koch — [roboticscenter.ai wiki](https://roboticscenter.ai/wiki/so-101) [S]
- Third-party DIY estimate: follower only about $122, follower + leader about $230 — [roboticscenter.ai](https://www.roboticscenter.ai/hardware/so-101) [S]

**Is a follower arm alone usable? (LeRobot)**
- **Phone teleop**:
  - iOS uses the HEBI Mobile I/O app (ARKit); Android uses the WebXR `teleop` package. Install with `pip install lerobot[phone]`.
  - Examples target the **SO-100/SO-101 follower**.
  - Pipeline: phone pose → end-effector deltas → IK via Placo (a Pinocchio wrapper) with `SO101/so101_new_calib.urdf`.
  - Safety: workspace bounds and `max_ee_step_m` (default 0.10).
  - No latency figures are given.
  - Source: [phone_teleop.mdx](https://github.com/huggingface/lerobot/blob/main/docs/source/phone_teleop.mdx) [R]
- **Keyboard teleop** exists but needs a global key backend: it works on X11, Windows and macOS, not on Wayland or headless machines — [il_robots.mdx](https://github.com/huggingface/lerobot/blob/main/docs/source/il_robots.mdx) [R]
- **HIL-SERL**: "A leader arm is optional if you use a gamepad or keyboard". It learns in end-effector space and needs a URDF. The SO-101 leader is recommended because its reduced gearing lets it "move and track the follower arm", which makes takeover smoother — [hilserl.mdx](https://github.com/huggingface/lerobot/blob/main/docs/source/hilserl.mdx) [R]

**Alternatives supported by LeRobot, with sourced costs**
- **LeKiwi** (mobile base + SO-101 pair): 12 V full build **$482 / €545.8**; 5 V **$499 / €526**. Base only: 12 V $251.5 / €307.8; 5 V $248 / €295; wired $184 / €235. The base uses 3× STS3215, a Raspberry Pi 5 (4 GB) and 2 USB cameras. No UK vendors are listed — [LeKiwi BOM](https://github.com/SIGRobotics-UIUC/LeKiwi/blob/main/BOM.md) [R]; [LeKiwi repo](https://github.com/SIGRobotics-UIUC/LeKiwi) [R]
- **Koch "Low-Cost Robot Arm"** (Dynamixel): follower about $258 (2× XL430-W250, 4× XL330-M288); leader about $183 (6× XL330-M077); about **$430** for both — [low_cost_robot](https://github.com/AlexanderKoch-Koch/low_cost_robot) [R]
- **XLeRobot** (dual-arm mobile): about **$660 / €680** basic, excluding printing, tools, shipping and taxes. A WowRobo developer assembly kit was announced at $579 (2025-09-09) — [XLeRobot](https://github.com/Vector-Wangel/XLeRobot) [R]
- **Grabette** (Pollen Robotics handheld data-collection gripper, similar to UMI): about **€490**, plus the Gripette robot gripper at about **€120** (blog dated Jul 21, 2026) — [Grabette blog](https://github.com/huggingface/blog/blob/main/grabette.md) [R]
- OMX, reBot B601, EarthRover, HopeJR, OpenArm, Reachy2 and G1 are supported (README), but their prices were not found.
  - **reBot B601**: an open-source Seeed Studio arm, with a 6-DOF follower on Damiao or RobStride CAN motors and a "reBot Arm 102" leader. The docs give no price — [rebot_b601.mdx](https://github.com/huggingface/lerobot/blob/main/docs/source/rebot_b601.mdx) [V, searched for specs and price]
  - **OMX** (ROBOTIS): Dynamixel-based, needs the Dynamixel SDK. The docs give no price — [omx.mdx](https://github.com/huggingface/lerobot/blob/main/docs/source/omx.mdx) [V, searched for specs and price]
- The processors guide uses phone → SO-100 follower as its worked example. It notes that "relative EE deltas or joint position are often preferred as learning features". Its example sets `max_ee_step_m=0.20`, whereas phone_teleop.mdx gives a default of 0.10, so the two docs disagree — [processors_robots_teleop.mdx](https://github.com/huggingface/lerobot/blob/main/docs/source/processors_robots_teleop.mdx) [V, searched for key terms]

### Inferences
**What fits in £200** (estimates labelled; USD→GBP assumed at about £0.74–0.80 per $, not sourced for 10 Oct 2026):
- **Option 1, DIY follower only (fits, just).**
  - 6 × £19.20 = **£115.20** for 12 V servos from RobotShop UK (computed from the sourced unit price; on-demand stock).
  - Add a bus board, 12 V 5 A PSU, USB-C cable and clamp. UK prices were not found; the BOM lists about $10.60, $10, $7 and $5.
  - Add printed parts: £29.74 (Unmanned Tech), or free if BRL prints them.
  - Estimated total **£165–£200** delivered.
  - Alternatively, a PartaBot or WowRobo follower-only kit (price not found) plus import costs.
- **Option 2, two-arm motor kit (just over).** £203.99 + £29.74 = **£233.73** (computed). That is about £34 over budget, or about £204 if BRL prints the parts. The kit was sold out when indexed.
- **Option 3, imported kits** (WowRobo $259/$299, Seeed $277.99, Hiwonder $269.99). Adding shipping and UK import VAT (standard rate assumed 20%, not sourced here) puts these **well above £200** (estimated).
- Assembled kits in the UK, from £271.98 (no printed parts) to about £730 (2+2), are **out of budget**.

**Other inferences**
- A follower-only arm is enough for imitation learning in LeRobot through phone EE teleop. However, how phone/EE demonstrations compare with leader-arm demonstrations in quality and policy success is undocumented. Ask BRL for a leader arm before designing around phone demos.
- LeKiwi, Koch, XLeRobot, OpenArm and Grabette all exceed a £200 budget on sourced BOMs alone.
- Choosing the 12 V follower (30 kg·cm vs 16.5 kg·cm) should reduce sag under load. Whether it changes backlash is unknown.

### Gaps
- No live UK vendor page could be opened. Stock on 10 Oct 2026 is unverified for every listing. VAT inclusion was captured only for RobotShop UK ("tax included") and Amazon UK (import charges included).
- The Pi Hut: one search found no SO-ARM101 listing. Pimoroni and Cool Components were not searched (budget). Feetech direct pricing was not found.
- UK prices for the Waveshare board, the 12 V 5 A PSU and a wrist camera: not found.
- PartaBot follower-only price and stock: not found.
- Prices for OMX, reBot B601, EarthRover and HopeJR: not found.

---

## 3. Documented SO-100 / SO-101 performance, hardware realities and failure modes

### Takeaway
I found **no published repeatability or payload specification for the SO-101**. The hard numbers that do exist are at the servo level:

- STS3215 resolution is 4096 steps per revolution (0.088° per step).
- Measured backlash is ~0.6° unloaded and ~1.3° loaded per servo, i.e. 7–15 encoder counts.

Published success rates range from 78% (author-reported SmolVLA on SO-100) down to 32–56% (an independent four-task SO-101 benchmark). Independent work attributes most failures to *execution*, not semantics.

### Cited Findings
**Servo and arm mechanics**
- STS3215: **4096 steps/rev** (model number 777). Control-table registers include P/D/I coefficients (addresses 21/22/23), Acceleration (41), Torque_Limit (48), **Present_Load (60)**, **Present_Temperature (63)** and **Present_Current (69)** — [LeRobot feetech tables.py](https://github.com/huggingface/lerobot/blob/main/src/lerobot/motors/feetech/tables.py) [R]. 360°/4096 = **0.088° per step** (computed).
- Open-source backlash test stand for low-cost UART servos (2026; the journal is inferred as *HardwareX* from the ScienceDirect ISSN in the article ID):

  | Condition | Counts | Degrees | mm |
  |---|---|---|---|
  | Single STS3215, unloaded | **7.03** | **0.62°** | **1.08** |
  | Single STS3215, loaded | **14.78** | **1.30°** | **2.27** |
  | Two coupled servos | ~5–8 | — | — |

  The paper states that "even a small amount of backlash can reduce positioning accuracy, degrade repeatability" — [ScienceDirect](https://www.sciencedirect.com/science/article/pii/S2468067226000271) [S]; [PMC mirror](https://pmc.ncbi.nlm.nih.gov/articles/PMC13087586/) [S, blocked]. The mm values match a 100 mm lever arm (100 × tan 0.62° = 1.08 mm, computed). The count and degree values match the 4096-count encoder (7.03 × 0.088° = 0.62°, computed).
- Robo9 bench tests of the STS3215: datasheet gearbox backlash "≤ 0.5°"; measured free play about 0.8–0.9° (about 1.3 mm at 100 mm); 3 mm at a 160 mm arm (≈1.07°). They note that "the encoder cannot observe motion within the internal gear stages". Protocol: a 10 cm vertical lever, 20-step increments, three cycles in each direction, three iterations — [Robo9 STS3215 test](https://robonine.com/testing-of-feetech-sts3215-servomotor-backlash-repeatability-and-torque/) [S]. A separate Robo9 article covers backlash compensation — [Robo9 compensation](https://robonine.com/backlash-compensation-in-sts3215-servo-actuators/) [T]
- "SO-101: Error Anatomy" (a GitHub project dated 2026-09):
  - Plans to attribute SO-101 placement error to mechanical (backlash/compliance), control (servo tuning) and policy variance.
  - Uses ground truth independent of the arm's sensors and ISO 9283-style statistics.
  - Estimates each source by variance subtraction, assuming independence, which the author says must be checked.
  - Status: hardware bring-up, "no measurement data yet"; 8 commits, 0 stars.
  - Source: [so101-error-anatomy](https://github.com/tucasteixeira-debug/so101-error-anatomy) [R]

- **LeRobot's SO follower configuration** (current main branch):
  - Position-mode gains written to each STS3215 at connect: `position_p_coefficient: int = 16`, `position_i_coefficient: int = 0`, `position_d_coefficient: int = 32`. **The integral gain is zero.**
  - Gripper only: `Max_Torque_Limit` 500 ("50% of max torque to avoid burnout"), `Protection_Current` 250 ("50% of max current to avoid burnout"), `Overload_Torque` 25 ("25% torque when overloaded").
  - `max_relative_target` (a per-step safety clamp on commanded motion) defaults to `None`.
  - Failed `sync_read`s are retried because "Feetech buses can occasionally return a corrupted status packet ('Incorrect status packet!'), especially when several joints move at once".
  - Source: [config_so_follower.py](https://github.com/huggingface/lerobot/blob/main/src/lerobot/robots/so_follower/config_so_follower.py) [V]; [so_follower.py](https://github.com/huggingface/lerobot/blob/main/src/lerobot/robots/so_follower/so_follower.py) [V]

**Published success rates**
- SmolVLA paper (SO-100; authors' own setup; PREPRINT, arXiv 2506.01844): SmolVLA (450M) averaged **78.3%** (pick-place 75, stacking 90, sorting 70), vs **ACT 48.3%** and **π0 61.7%**. Sources disagree on π0's size (3.3B or 3.5B) — [arXiv 2506.01844](https://arxiv.org/abs/2506.01844) [S]; [MarkTechPost, 3 Jun 2025](https://www.marktechpost.com/2025/06/03/hugging-face-releases-smolvla-a-compact-vision-language-action-model-for-affordable-and-efficient-robotics/) [S]
- SmolVLA blog:
  - "51.7%" without community pretraining, "78.3%" with it.
  - On the SO-101 cube task, async and sync inference both reach "≈78%" success, with completion "9.7s vs. 13.75s" and "19 vs. 9" cubes in a fixed window.
  - Training needs "a single consumer GPU"; inference can "run on CPU".
  - Source: [SmolVLA blog](https://github.com/huggingface/blog/blob/main/smolvla.md) [R]
- **Independent SO-101 benchmark** (Yi Yu & Xinchuan Qiu, Hiroshima University; arXiv 2606.08881, June 2026, PREPRINT):
  - Four tasks (pen transfer, colour sorting, object packing, pen placement), 20 episodes per task.
  - **π0.5 56.25%, ACT 33.75%, SmolVLA 32.5%**. Wall-X was also tested; its figure was not captured.
  - "Execution instability is the main failure source"; recovery ability "differs substantially between architectures"; uses a failure taxonomy and recovery-aware metrics.
  - Source: [arXiv 2606.08881](https://arxiv.org/abs/2606.08881) [S]. The rates match 80 trials per policy (45/80, 27/80, 26/80; computed).
- Single-developer **ACT reproduction on SO-101** (rollouts named 2026-08-12):
  - Setup: 50 demos (17,879 frames); wrist plus external camera at 640×480/30 fps; chunk size 100; 100k steps on one A10G in about 4.5 h.
  - 20 trials: reached 20/20, **grasped 12/20, placed 10/20**; centre 6/8 vs corners 4/12.
  - "Reaching is solved; grasping is where the robot struggles."
  - Source: [khushi02/so101-imitation-learning-ACT](https://github.com/khushi02/so101-imitation-learning-ACT) [R]
- **Chalmers/University of Gothenburg MSc thesis "FAR"** (defended 12 Jun 2026):
  - Task: SO-101 cube into box; 100 demos at 20 Hz; n = 50 per row; Jetson client with remote GPU server.
  - Baselines: SmolVLA 60% (no RTC) / 64% (RTC); π0.5 72% / 68%. When the experimenter lifts the cube as the gripper closes ("interference"), these fall to 40% (SmolVLA) and 32% (π0.5).
  - A proprioceptive failure-aware recovery monitor with retries changed success by **+8/+4/+42 pp (SmolVLA)** and **+10/0/+68 pp (π0.5)**.
  - **False positives lowered success** at two positions (70→60%, 80→60%).
  - Stated limitations: "A principled selector that incorporates contact history, failure signature, or task-progress estimates remains future work"; under jitter the RTC delay estimate lags, so the "effective horizon drifts out of calibration".
  - Source: [lerobot_far experiments.md](https://github.com/HyuanTan/lerobot_far/blob/main/docs/experiments.md) [R]; [lerobot_far README](https://github.com/HyuanTan/lerobot_far) [R]
- **DP-ACT** (Yuhang Li & Deping Li, *Scientific Reports*, Aug 2026, doi:10.1038/s41598-026-63709-3): SO100 with Feetech actuators; three-stage precision assembly; 160 demos, 100 trials. Success rose from **62% (ACT) to 82%** with dynamic-programming local trajectory correction inside ACT inference. The abstract says ACT struggles with reliability on low-cost manipulators that have limited mechanical repeatability — [summary of the abstract](https://github.com/deepcharles/macs3050-journal/issues/82) [A]; [DOI](https://doi.org/10.1038/s41598-026-63709-3) [T]
- **DREAM-Chunk** (arXiv 2606.18589, June 2026, PREPRINT): tested on SO-101 and Franka with SmolVLA and π0.5. It is motivated by "hardware execution errors" and raised open-loop π0.5 from **10% to 65%** on a precise insertion task under external perturbation — [arXiv 2606.18589](https://arxiv.org/abs/2606.18589) [S]

**Failure modes and practicalities**
- Grasp failures dominate the ACT reproduction (the gripper failed to close in 8 of 20 trials), and workspace edges are under-covered — [khushi02](https://github.com/khushi02/so101-imitation-learning-ACT) [R]
- Silent empty grasps and slips; latency tails (SmolVLA under RTC: queue wait mean ≈58 ms, **p95 413 ms**; SmolVLA needs at least 512×512 input vs 224×224 for π0.5). For π0.5 under RTC the source lists mean ≈43.5 ms and p95 = 43 ms, which is internally inconsistent and should be checked — [lerobot_far experiments.md](https://github.com/HyuanTan/lerobot_far/blob/main/docs/experiments.md) [R]
- Setup: "the power supply cable might disconnect as you manipulate the board"; Linux may need `sudo chmod 666` on the USB port — [so101.mdx](https://github.com/huggingface/lerobot/blob/main/docs/source/so101.mdx) [R]
- A slow control loop silently mislabels data (25 Hz recorded as 30 Hz); cameras were the bottleneck — [il_robots.mdx](https://github.com/huggingface/lerobot/blob/main/docs/source/il_robots.mdx) [R]
- HIL-SERL on SO-100/101: learning is "very sensitive to the intervention strategy"; it is sensitive to background distractions (cropping needed); resolutions other than 128×128 and 64×64 are "not been extensively tested"; the only quantified result is that interventions cut steps-to-max-reward "by a quarter" — [hilserl.mdx](https://github.com/huggingface/lerobot/blob/main/docs/source/hilserl.mdx) [R]

### Inferences
- **Magnitude check** (computed or estimated):
  - Backlash is 7–15 encoder counts per joint, i.e. 1–2 mm at a 100 mm lever.
  - Summed along the shoulder–elbow–wrist chain at full reach (reach not found), several millimetres of end-effector play is plausible (estimated).
  - That matters for small-object grasps and insertion but not for drops into a bin.
- **The STS3215's magnetic encoder appears to sense the output shaft.** The test stand reports backlash in encoder counts, and Robo9 says internal gear stages are unobservable. This is inference, not confirmed. If true, the arm's error splits into two parts:
  - **Visible**: servo tracking or steady-state error under load, i.e. Goal_Position − Present_Position. A classical host-side outer loop can remove it.
  - **Invisible**: gear-stage play plus link and printed-part compliance, which joint sensors cannot see.

  Any error-budget study must separate the two, or a classical fix will quietly make the learned contribution disappear.
- **Zero integral gain makes steady-state error under gravity load expected.** LeRobot's default servo gains are P = 16, I = 0, D = 32. So the "visible" part of the error is probably material by default, and simply enabling I or adding a host-side outer-loop integral is a strong, almost free classical baseline. This is inference; it has not been measured.
- **The torque-capped gripper may contribute to grasp failures.** The gripper is limited to 50% torque and drops to 25% on overload. This could contribute to the grasp failures reported in the ACT reproduction and to slips, but that is speculative and untested. It also means gripper Present_Load and Present_Current are bounded signals.
- **Evaluation counts and protocol dominate conclusions.**
  - The gap between 78% (author-reported) and 32–56% (independent) shows how much protocol and task choice matter.
  - Most studies use 20 trials per condition. At n = 20, the 95% interval on 50% is about **±22 pp** (computed, normal approximation).
  - An MSc comparison needs about 50 or more trials per condition, or a paired design.
- **FAR's large gains come only from staged interference.** Under natural conditions FAR changes success by 0 to +10 pp at n = 50, where the standard error of a difference is about 9 pp (computed). So the evidence that recovery monitors help with *natural* failures is weak so far.

### Gaps
- SO-101 payload, reach and repeatability: not found. Vendor and spec pages were blocked, and the search summary reported "no published repeatability measurement for the SO-101".
- Full text of the backlash test-stand paper (load magnitude, lever, method details): not read.
- Per-task tables and the failure taxonomy of 2606.08881: not read. Wall-X numbers: not captured.
- STS3215 thermal behaviour (overheating under sustained holding torque) and calibration drift: no sourced data. The docs are silent and the LeRobot issue tracker was not searched.

---

## 4. Methods-level state of the art (2024–2026): what has genuinely moved in action chunking, diffusion and flow matching on cheap hardware, and what remains open

### Takeaway
**What has genuinely moved:**

- **Small VLAs on consumer hardware.** SmolVLA (Jun 2025) trains on one consumer GPU and runs on CPU, and community-data pretraining lifted its SO-100 success from 51.7% to 78.3%.
- **Inference-delay handling for chunked flow policies.**
  - RTC (NeurIPS 2025) reached LeRobot v0.5.0 in March 2026.
  - Training-time conditioning followed in December 2025.
- **Explanations of why chunking helps.**
  - The stability theory, published at ICLR 2026.
  - "Revisiting open-loop" (Aug 2026), showing that long open-loop execution mainly compensates for short context.
- **First independent cheap-arm evaluations and hardware-error-aware chunk selection**, from June to August 2026.

The chunk-boundary and latency niche is now crowded, with at least eight papers in about fourteen months. What remains open on cheap arms concerns hardware error, honest evaluation and failure handling, not new samplers.

### Cited Findings
**Foundations**
- ACT (Zhao et al., **RSS 2023**): predicting a chunk of k actions reduces the effective horizon by a factor of k. The authors hypothesise that action chunking is the key design choice — [RSS 2023 PDF](https://www.roboticsproceedings.org/rss19/p016.pdf) [S]
- Diffusion Policy (Chi et al., RSS 2023; journal version in IJRR): **not accessed this session** (see Gaps).

**Why chunking helps: theory and controlled studies**
- Zhang, Pfrommer, Pan, Matni and Simchowitz show that action chunking and exploratory data collection avoid exponential compounding error in continuous-control behaviour cloning, with **control-theoretic stability as the hidden mechanism**. Posted 11 Jul 2025, v5 26 Nov 2025. It is listed for **ICLR 2026** on OpenReview under the changed title "Action Chunking and Data Augmentation Yield Exponential Improvements in Behavior Cloning for Continuous Spaces" — [arXiv 2507.09061](https://arxiv.org/abs/2507.09061) [S]; [OpenReview](https://openreview.net/forum?id=jiWXDvw1Lf) [S]
- **Zeng, Agarwal, Bati, Lee, Ancha, Tedrake** (arXiv 2608.15938, Aug 2026, PREPRINT):
  - "long open-loop execution primarily helps short-context policies imitate 'non-Markovian demonstrations'".
  - "when policies are provided with a sufficiently long context, open-loop execution is no longer beneficial and the most reactive, closed-loop policies perform best".
  - Four simulated tasks (Push-T, FurnitureSim, IsaacLab gear insertion, Franka Kitchen) and two real tasks; code is public, but the hardware inference code is proprietary.
  - Source: [arXiv 2608.15938](https://arxiv.org/abs/2608.15938) [S]; [GitHub code index](https://github.com/Michaelszeng/Revisiting-Open-Loop-Execution-in-Robotics) [R]

**Handling inference delay and chunk boundaries**
- **RTC** (Black, Galliker, Levine; **NeurIPS 2025**; arXiv 2506.07339):
  - Method: generates the next chunk while the current one executes, "freezing" actions guaranteed to execute and "inpainting" the rest; no retraining.
  - Evaluation: 12 dynamic Kinetix tasks and 6 real bimanual tasks.
  - Limitations from the Discussion: it "adds some computational overhead compared to methods that sample directly from the base policy"; "there are more dynamic settings that could benefit even more"; legged locomotion appears only in simulation.
  - Source: [NeurIPS 2025 listing](https://neurips.cc/virtual/2025/poster/117747) [S]; [arXiv 2506.07339](https://arxiv.org/abs/2506.07339) [S]
  - The public simulation code evaluates "2048 trials per level" across 12 Kinetix levels with "an exhaustive sweep over inference delay and execution horizon for all methods". Training-time RTC is reproduced by fine-tuning with `simulated_delay` 5 for 8 epochs — [real-time-chunking-kinetix README](https://github.com/Physical-Intelligence/real-time-chunking-kinetix) [V]
- **Training-Time Action Conditioning for Efficient Real-Time Chunking** (arXiv 2512.05964, Dec 2025, PREPRINT) underlies LeRobot's "trained" RTC mode, which requires a Pi05 checkpoint trained with `rtc_training_max_delay > 0` — [rtc.mdx](https://github.com/huggingface/lerobot/blob/main/docs/source/rtc.mdx) [R]; [arXiv 2512.05964](https://arxiv.org/abs/2512.05964) [T]
- LeRobot RTC parameters: `execution_horizon` (typically 8–12; larger values give smoother transitions and less reactivity); `inference_delay` passed at runtime; `max_guidance_weight` 10.0 for 10-step flow matching; guided or trained modes. The page reports **no quantitative results** — [rtc.mdx](https://github.com/huggingface/lerobot/blob/main/docs/source/rtc.mdx) [R]
- A critique of RTC argues that, because it is applied only at inference time, it "can cause spurious multimodal switching across chunk boundaries" ("Learning Native Continuation for Action Chunking Flow Policies" / Legato; arXiv 2602.12978, Feb 2026, PREPRINT) — [arXiv 2602.12978](https://arxiv.org/abs/2602.12978) [S]
- More entries from 2025–26, titles only, all PREPRINT:
  - Action-Prior Denoising for Smooth Real-Time Chunking — [arXiv 2605.25537](https://arxiv.org/abs/2605.25537) [T]
  - FutureRTC (anticipatory-conditioned chunking) — [arXiv 2607.24008](https://arxiv.org/abs/2607.24008) [T]
  - Temporal Action Selection for Action Chunking — [arXiv 2511.04421](https://arxiv.org/abs/2511.04421) [T]
  - LiPo post-optimisation smoothing of chunks — [arXiv 2506.05165](https://arxiv.org/abs/2506.05165) [T]
- **BID** (Bidirectional Decoding; Liu et al.; **ICLR 2025**; a proceedings PDF exists, though one listing shows it as a workshop poster):
  - Samples several chunks and picks one by backward coherence and forward contrast.
  - Beat vanilla, warm-start and EMA sampling on 7 simulated and 2 real dynamic tasks; roughly doubled compute.
  - Limitation: it "may still be expensive for high-frequency control on low-cost robots".
  - Open direction: generating "quality yet diverse" chunks under batch-size limits.
  - Source: [ICLR 2025 proceedings PDF](https://proceedings.iclr.cc/paper_files/paper/2025/file/0d78dd998f7b9ac79604d47a2d79bb0d-Paper-Conference.pdf) [S]; [ICLR virtual page](https://iclr.cc/virtual/2025/32377) [S]
  - The code README cites it as a main-conference paper ("The Thirteenth International Conference on Learning Representations", 2025). Authors: Yuejiang Liu, Jubayer Ibn Hamid, Annie Xie, Yoonho Lee, Max Du, Chelsea Finn (Stanford IRIS Lab).
  - Push-T results in the README (ah = number of actions executed open-loop):

    | Condition | Vanilla ah=1 | Vanilla ah=8 | Warm-start | EMA | BID |
    |---|---|---|---|---|---|
    | Deterministic | 0.846 | 0.884 | 0.887 (ah=8) | 0.866 (ah=8) | **0.928** (ah=8) |
    | Stochastic (noise 1.0) | 0.805 | **0.582** | 0.852 (ah=1) | 0.823 (ah=1) | **0.889** (ah=1) |

    So open-loop execution helps without disturbances and collapses under stochastic perturbation.
  - Source: [bid_diffusion README](https://github.com/YuejiangLIU/bid_diffusion) [V]
  - A **BID + LeRobot** port exists, but it implements BID only on VQ-BeT in the Push-T simulation (closed loop ah=1, open loop ah=5). There is no real-robot or SO-100 experiment — [bid_lerobot README](https://github.com/Jubayer-Hamid/bid_lerobot) [V]
- **DREAM-Chunk** (Wenxi Chen, …, Yuejiang Liu, …, Yan Gu; arXiv 2606.18589, June 2026, PREPRINT):
  - Method: sample candidate chunks, predict their latent futures with a lightweight world model, and "keep only the chunk whose predicted latent state best matches what the robot actually experiences".
  - Motivated by "stochastic dynamics, hardware execution errors, and partially observed state".
  - SO-101 and Franka; 10% → 65% on perturbed insertion with π0.5.
  - Source: [arXiv 2606.18589](https://arxiv.org/abs/2606.18589) [S]. Yuejiang Liu is also a BID author (inferred from the author lists).
- SmolVLA async inference gave the same success (≈78%), about 30% faster completion and twice the throughput in a fixed window — [SmolVLA blog](https://github.com/huggingface/blog/blob/main/smolvla.md) [R]

**Cheap and embedded hardware**
- Ge et al., "Bridging the Resource Gap" (arXiv 2411.11406, Nov 2024, PREPRINT): model compression plus Temporal Ensemble with Dropped Actions (TEDA) for edge deployment. Symmetric quantisation was required on the target device, and input-shape unification made it about 6× faster — [arXiv 2411.11406](https://arxiv.org/abs/2411.11406) [S]
- **"Bimanual Manipulation Within an 8 GB Budget: Zero-Copy Sensing and Quantized ACT on an Entry-Level Jetson"** (arXiv 2608.03938, Aug 2026, PREPRINT):
  - TensorRT INT8 calibration quantises the ResNet18 backbone but accepts **0 of 145 transformer layers**.
  - INT8 gives **28%** further latency reduction over FP16.
  - Full precision is feasible at `n_action_steps = 100`, but not with the per-step re-prediction that temporal ensembling needs.
  - Source: [arXiv 2608.03938](https://arxiv.org/abs/2608.03938) [S]. A secondary blog claims Diffusion Policy failed to converge in that setting — [blog summary](https://book.st-hakky.com/en/news/8gb-jetson-zero-copy-quantized-act) [S, low trust]
- NVIDIA's Jetson AI Lab lists LeRobot ACT/diffusion on Jetson, including the 8 GB Orin Nano (shown with a warning mark) — [Jetson AI Lab](https://www.jetson-ai-lab.com/lerobot) [S]
- Low-cost bilateral control (ALPHA-α / Bi-ACT) avoids temporal ensembling because precise current control needs a high rate: inference runs at 70 Hz, commands at 100 Hz — [arXiv 2411.09942](https://arxiv.org/abs/2411.09942) [S]; [Bi-ACT arXiv 2401.17698](https://arxiv.org/abs/2401.17698) [T]. Both are PREPRINT as far as checked.
- DP-ACT on SO100 (*Scientific Reports* 2026): local dynamic-programming correction inside ACT inference "while keeping the control-loop frequency the same"; 62% → 82% — [summary](https://github.com/deepcharles/macs3050-journal/issues/82) [A]

**Data and tooling**
- LeRobot's recipe: at least 50 episodes, with variation added gradually — [il_robots.mdx](https://github.com/huggingface/lerobot/blob/main/docs/source/il_robots.mdx) [R]. For SmolVLA, 25 episodes was "not enough leading to a bad performance" — [smolvla.mdx](https://github.com/huggingface/lerobot/blob/main/docs/source/smolvla.mdx) [V]
- Action representation is now a switchable variable in LeRobot. Absolute is the default; relative is available for the pi family and claims "no error accumulation across chunks" — [action_representations.mdx](https://github.com/huggingface/lerobot/blob/main/docs/source/action_representations.mdx) [V]
- **Runtime failure detection for generative policies: FAIL-Detect** (Chen Xu et al., TRI; **RSS 2025**, arXiv 2503.08558), "Can We Detect Failures Without Failure Data? Uncertainty-Aware Runtime Failure Detection for Imitation Learning Policies":
  - Learns scalar uncertainty scores from (observation embedding, action) pairs *in the training data only*, i.e. no failure data.
  - logpZO and RND are "the best performing ones" among several baselines.
  - Decides with a conformal-prediction band.
  - Works with both diffusion and flow-matching policies.
  - The released code covers robomimic simulation tasks (square, transport, tool_hang, can). Real-robot experiments were not checked.
  - Source: [FAIL-Detect README](https://github.com/CXU-TRI/FAIL-Detect) [V]
- DAgger-style collection arrived in v0.5.1, and `lerobot-rollout --strategy dagger` with intervention flags in v0.6.0 — [releases](https://github.com/huggingface/lerobot/releases) [P]; [v0.6.0 blog](https://github.com/huggingface/blog/blob/main/lerobot-release-v060.md) [R]
- Grabette: Diffusion Policy trained on 200 handheld demonstrations of grasping a cup, evaluated on an OpenArm. **No quantitative results** are reported — [Grabette blog](https://github.com/huggingface/blog/blob/main/grabette.md) [R]

### Inferences
- **What an MSc can add that is not "collect demos, train ACT, report success rate"**:
  1. Attribute a learned policy's error to hardware vs policy, using external metrology on a cheap arm.
  2. Test whether chunking's stability-based benefit (ICLR 2026 theory) survives hysteretic, backlash-prone actuation, given the Aug 2026 finding that context, not open-loop length, is what matters.
  3. Runtime failure detection and recovery under *natural* rather than staged failures.
  4. Pre-registered head-to-heads against scripted controllers.
  5. Statistically powered evaluation (50 or more trials per condition, or paired designs).
- **Avoid**:
  - New chunk samplers or schedulers (RTC, training-time RTC, Legato, action-prior denoising, FutureRTC, TAS, DREAM-Chunk and BID already cover this ground).
  - On-device quantisation (the Aug 2026 8 GB Jetson paper).
  - VLA leaderboard-style comparisons (the June 2026 SO-101 benchmark).
- **Latency is unlikely to be the binding constraint for quasi-static SO-101 tasks.** SmolVLA's sync and async modes had the same success and differed only in time. RTC-style gains need dynamic tasks.
- DREAM-Chunk is the strongest methods-level overlap for any "hardware-error-aware chunking on SO-101" idea. Its perturbations were external, and its contribution is a selection mechanism, not an error attribution.

### Gaps
- Full texts of ACT, Diffusion Policy, RTC, BID, SmolVLA and 2507.09061 were not readable. Their limitations and future-work sections are quoted only where a search snippet gave them.
- Not searched (budget exhausted): flow-matching policy variants (streaming, one-step, consistency); data quality and scaling studies; curation methods; noise-space RL fine-tuning of diffusion/flow policies; Q-chunking.
- Runtime failure detection: only FAIL-Detect's README was read. Its real-robot results, and other detectors (Sentinel/STAC, FIPER), were not verified.
- Whether 2607.24008, 2605.25537 or 2511.04421 test on cheap arms: unknown.
- The second real task in 2608.15938 and the hardware used: unknown.

---

## 5. The strongest non-learned baseline for typical SO-101 tasks, and which task classes make it fail

### Takeaway
For the tasks most SO-101 demos use (rigid pick-and-place into a bin, colour sorting, stacking), the baseline to beat is a scripted pipeline: a fixed webcam, colour or ArUco detection, camera-to-base calibration, inverse kinematics, and approach from one side. LeRobot already ships the IK and the URDF. **I found no published scripted-vs-learned comparison on SO-100/101.** The failure classes below come from reasoning, not from sourced measurements.

### Cited Findings
- LeRobot already uses IK (Placo, a Pinocchio wrapper) with the SO-101 URDF (`so101_new_calib.urdf` from the SO-ARM100 repo), plus workspace bounds and per-step end-effector limits, for phone teleop — [phone_teleop.mdx](https://github.com/huggingface/lerobot/blob/main/docs/source/phone_teleop.mdx) [R]
- `lerobot-find-joint-limits` and end-effector-space control exist for HIL-SERL — [hilserl.mdx](https://github.com/huggingface/lerobot/blob/main/docs/source/hilserl.mdx) [R]
- `lerobot-replay` replays a recorded episode "to test how repeatable the robot's motions are" — [il_robots.mdx](https://github.com/huggingface/lerobot/blob/main/docs/source/il_robots.mdx) [R]
- Learned-policy success on simple SO-101 tabletop tasks is 32.5–56.25% in the independent benchmark — [arXiv 2606.08881](https://arxiv.org/abs/2606.08881) [S]. ACT reached 50% on clip-into-bowl, with grasp failures dominant — [khushi02](https://github.com/khushi02/so101-imitation-learning-ACT) [R]
- SmolVLA's real-world tasks were pick-place, stacking and sorting — [SmolVLA blog](https://github.com/huggingface/blog/blob/main/smolvla.md) [R]

### Inferences
- **Baseline specification** (estimated 1–2 weeks of work):
  - A fixed overhead or oblique webcam (the student owns one).
  - An ArUco board for camera-to-base calibration.
  - HSV segmentation or ArUco-tagged objects.
  - A top-down grasp at the detected centroid, from a fixed approach height.
  - Placo/URDF IK.
  - Always approaching from the same direction, to take up backlash.
- **Where scripted control should win:** rigid, well-separated, colour-distinct or tagged objects with centimetre-level tolerance. These are exactly the tasks in most LeRobot demos and in the SmolVLA and 2606.08881 evaluations. Given 33–56% learned success there, a well-built scripted baseline may well beat ACT and SmolVLA on them (estimated; untested).
- **Where scripted control should fail** (reasoning):
  - Deformables (cloth, cable): there is no reliable shape model.
  - Tight-clearance insertion: the only force sensing is Present_Load/Present_Current.
  - Clutter and occlusion: single-view detection breaks.
  - Transparent or reflective objects: colour and ArUco detection fail.
  - Moving targets: there is no prediction.
  - Multi-step tasks that need recovery: there is no replanning.
- **A fair test** pre-registers an *external* task list (for example, the four tasks of 2606.08881), uses equal trial counts, and reports confidence intervals. This avoids choosing tasks so that learning wins (filter 3).

### Gaps
- No scripted-vs-learned head-to-head on SO-100/101 was found. One search was attempted and the budget ran out before a second.
- ArUco pose accuracy with the student's webcam at bench distance: not found. It must be measured, and it bounds every ground-truth claim above.

---

## 6. Sub-£200 sensor additions for the Pololu 3pi+, and whether each opens a genuine frontier question

### Takeaway
**No current UK price could be sourced for any of these sensors.** Vendor sites were blocked and the search budget was exhausted, so every price below is an *estimate*. On reasoning alone:

- A **downward optical-flow sensor** (PAA5100JE or PMW3901) is the only add-on that makes the robot's own physics (wheel slip) directly measurable. Its value is as an *instrument* for the still-open "adaptive motion control under surface or payload change" question, not as a frontier in itself.
- **Multizone time-of-flight** fits active localisation.
- A **camera, 2D LiDAR or UWB** mostly enable problems that already have strong classical solutions.

### Cited Findings
- 3pi+ 32U4 hardware: ATmega32U4, integrated motor drivers, encoders, OLED (newer version), buzzer, buttons, line sensors, front bump sensors, and an **LSM6DS33 accelerometer/gyro plus LIS3MDL compass** — [Pololu 3pi+ 32U4 Arduino library](https://github.com/pololu/pololu-3pi-plus-32u4-arduino-library) [R]. Encoder counts per revolution, wheel diameter and expansion pinout are in the user's guide, which was not read — [Pololu 3pi+ user's guide](https://www.pololu.com/docs/0J83) [T]
- Pimoroni's Python library supports both the **PMW3901 and the PAA5100JE over SPI**; the PAA5100JE "has a slightly different init routine". The README gives no range or frame-rate specs and no prices — [pimoroni/pmw3901-python](https://github.com/pimoroni/pmw3901-python) [R]
- LeRobot has a third-party sensors docs page (not read) and an EarthRover mobile-robot integration (v0.5.0) — [docs directory](https://github.com/huggingface/lerobot/tree/main/docs/source) [R]; [v0.5.0 blog](https://github.com/huggingface/blog/blob/main/lerobot-release-v050.md) [R]

### Inferences
Prices marked "estimated" are recollections of 2024–25 UK list prices, **unverified for 10 Oct 2026**; check before ordering. A zero-cost instrument also helps here: the owned USB webcam, used overhead with ArUco markers, gives ground-truth pose for any 3pi+ experiment (its accuracy must be measured first).

- **Downward optical flow (PAA5100JE "near" or PMW3901)**
  - Price: estimated **£20–£30** per breakout.
  - Integration: SPI, which uses SPI-capable pins on the M5Stamp C3. Feasible but not verified (estimated).
  - Question it opens: at the 3pi+'s achievable accelerations, how much do slip and surface change corrupt encoder odometry and velocity control, and can encoder–flow disagreement drive online adaptation?
  - Is it real? Only if slip at realistic accelerations is material. That is exactly the untested premise behind the open adaptive-control idea, and it ties to the earlier failure in which the simulator had no acceleration limits. The sensor is a cheap, direct test of whether that gap is real.
  - Baseline: EKF or complementary fusion of encoders, gyro and flow, with CUSUM change detection on the residuals. This is strong, so a learned estimator is unlikely to be justified (filter 2 and 4 risk).
  - Unknowns: the right mount height and how surface texture affects quality.
- **VL53L1X (single-zone ToF)**
  - Price: estimated **£12–£25**.
  - A narrow beam makes "turn to look" an explicit, costly sensing action, which would make the open active-localisation question (when to spend time on a localisation-improving action) bite.
  - But picking a narrow sensor *in order to* create that cost risks filter 3 (a manufactured gap).
- **VL53L5CX (8×8 multizone ToF)**
  - Price: estimated **£20–£35**.
  - Uses I2C, so it can share the existing bus.
  - Enables local obstacle maps and range-based localisation in a known arena, without a manufactured sensing cost. This is the better active-localisation sensor.
  - Learned reactive navigation would face VFH or potential-field baselines and would likely fail filter 2.
- **XIAO ESP32-S3 Sense (camera + MCU)**
  - Price: estimated **£12–£20**.
  - Enables onboard vision and TinyML. Learned visual navigation would face colour or ArUco tracking.
  - Learned visual odometry would lose to a cheap optical-flow chip. **Weak.**
- **2D LiDAR (LD06, LD19, STL-19P, RPLIDAR C1)**
  - Price: estimated **£60–£100**.
  - Large and power-hungry relative to a roughly 10 cm robot (estimated).
  - Opens SLAM and scan matching, which are classically solved. As a ground-truth instrument, the overhead webcam is cheaper. **Weak for this dissertation.**
- **UWB (DWM3000-class)**
  - Price: estimated **£20–£40 per module**. 2D trilateration needs at least 3 anchors plus a tag, so roughly **£80–£160** including microcontrollers (estimated).
  - Opens multi-robot relative localisation only if BRL lends more 3pi+ robots.
  - The classical trilateration plus EKF baseline is strong. Defer to the multi-robot and state-estimation research streams.

### Gaps
- All UK prices (Pimoroni, The Pi Hut, Cool Components, Pololu, Mouser/Farnell for the DWM3000): **not found**.
- Specs not sourced: PAA5100JE and PMW3901 working range and maximum speed; VL53L5CX field of view and rate; LiDAR size and current draw; 3pi+ encoder resolution and wheel diameter.
- Closest work on optical-flow slip estimation for small differential-drive robots, and on active localisation with ToF, was not searched (budget). The parallel adaptive-control and state-estimation researchers cover these.

---

## 7. Which candidate questions survive all eight filters, and what is each one's first-week kill test?

### Takeaway
**Two candidates survive conditionally.** Both run on a follower-only SO-101, which fits about £200 (tight):

- **(A) Error budget.** How much of a learned policy's error on a backlash-prone arm is mechanical, and does closed-loop replanning beat classical compensation as clearance tightens?
- **(B) Runtime failure detection.** Which natural failures can the arm detect from free servo telemetry, and when do learned progress or uncertainty signals do better?

**(C)**, a pre-registered scripted-vs-learned boundary map, is a safe fallback with low novelty, or a chapter inside A or B. **The 3pi+ sensor ideas** are instruments for other candidates, not standalone imitation-learning projects. Ideas built on chunk scheduling or RTC, Jetson deployment, VLA benchmarking or world models fail filter 5 (crowded), filter 3 (manufactured gap) or filter 6 (compute or budget). In every case, novelty means "no overlap found in an incomplete search", not established novelty.

### Cited Findings
These are the key facts used in the verdicts; details and access levels are in Sections 1–6.
- Execution instability is the main failure source in the independent SO-101 VLA benchmark, where success was 32.5–56.25% — [arXiv 2606.08881](https://arxiv.org/abs/2606.08881) [S]
- ACT's reliability suffers on low-repeatability low-cost arms; dynamic-programming correction lifted success from 62% to 82% on SO100 — [DP-ACT summary](https://github.com/deepcharles/macs3050-journal/issues/82) [A]
- STS3215 backlash is 0.62° unloaded and 1.30° loaded, against a 0.088° step — [backlash test stand](https://www.sciencedirect.com/science/article/pii/S2468067226000271) [S]; [tables.py](https://github.com/huggingface/lerobot/blob/main/src/lerobot/motors/feetech/tables.py) [R]
- A parallel SO-101 error-decomposition project exists, with no data yet (Sep 2026) — [so101-error-anatomy](https://github.com/tucasteixeira-debug/so101-error-anatomy) [R]
- The FAR proprioceptive monitor on SO-101 gave 0 to +10 pp under natural conditions and +42/+68 pp under staged interference, and false positives cost success — [lerobot_far experiments](https://github.com/HyuanTan/lerobot_far/blob/main/docs/experiments.md) [R]
- LeRobot v0.6.0 rollout strategies (sentry, dagger), intervention flags and reward models (SARM, Robometer, TOPReward) — [v0.6.0 blog](https://github.com/huggingface/blog/blob/main/lerobot-release-v060.md) [R]
- Servo telemetry registers are readable: Present_Load, Present_Current, Present_Temperature — [tables.py](https://github.com/huggingface/lerobot/blob/main/src/lerobot/motors/feetech/tables.py) [R]
- Default SO follower servo gains are P = 16, I = 0, D = 32, and the gripper is torque-limited to 50% (25% on overload) "to avoid burnout" — [config_so_follower.py](https://github.com/huggingface/lerobot/blob/main/src/lerobot/robots/so_follower/config_so_follower.py) [V]; [so_follower.py](https://github.com/huggingface/lerobot/blob/main/src/lerobot/robots/so_follower/so_follower.py) [V]
- FAIL-Detect (RSS 2025) detects failures of diffusion and flow policies without failure data, using learned uncertainty scores and a conformal band; released code is simulation-only (robomimic) — [FAIL-Detect README](https://github.com/CXU-TRI/FAIL-Detect) [V]
- In simulation, open-loop chunks collapse under stochastic perturbation (BID Push-T: ah=8 falls from 0.884 to 0.582; ah=1 holds 0.805) — [bid_diffusion README](https://github.com/YuejiangLIU/bid_diffusion) [V]
- The chunk-handling niche is crowded: RTC (NeurIPS 2025), training-time RTC (Dec 2025), Legato (Feb 2026), action-prior denoising (May 2026), DREAM-Chunk (Jun 2026), FutureRTC (Jul 2026), "Revisiting open-loop" (Aug 2026) — sources in Section 4
- Quantised ACT on an 8 GB Jetson is already published (Aug 2026) — [arXiv 2608.03938](https://arxiv.org/abs/2608.03938) [S]
- 12 GB VRAM envelopes — [hardware_guide.mdx](https://github.com/huggingface/lerobot/blob/main/docs/source/hardware_guide.mdx) [R]

### Inferences

#### Candidate A: Error anatomy of learned manipulation on a low-cost arm, and closed-loop replanning vs classical compensation (ranked 1st on evidence)
- **(a) Research question.** On an SO-101, how much of a learned policy's end-point error comes from each of three sources: (i) load-dependent backlash and compliance invisible to joint encoders, (ii) servo tracking error that the encoders can see, and (iii) policy variance? And as task clearance tightens, does shortening the open-loop execution horizon (visual closed loop) recover more success than classical compensation?
- **(b) Why now.** Evidence that has appeared since June 2026 points at execution error on cheap arms:
  - The independent SO-101 benchmark (arXiv June 2026) found execution instability dominant.
  - DP-ACT (*Scientific Reports*, Aug 2026) blames limited repeatability.
  - The STS3215 backlash measurements were published in 2026.
  - The chunking-stability theory (ICLR 2026) and "Revisiting open-loop" (arXiv Aug 2026) make testable predictions about open-loop horizon.
  - LeRobot v0.6.1 (3 Aug 2026) exposed the SO follower's P coefficient, so servo gain is now a first-class variable.
- **(c) Nearest work.** None of it attributes arm-level, load-dependent error with external ground truth *and* links it to a policy-design variable (execution horizon) and to a classical compensator. That is the open slot.
  - **DREAM-Chunk** (2606.18589) is the strongest methods overlap: chunk selection robust to "hardware execution errors" on SO-101. It uses external perturbations and does not attribute error.
  - **DP-ACT** corrects trajectories but, judging from the abstract, does not separate the error sources.
  - **so101-error-anatomy** plans the same decomposition but has no data and no horizon or compensation intervention.
  - **The backlash test stand** works at servo level only.
  - **Simulation evidence on the horizon trade-off** already exists. In BID's Push-T table, vanilla ah=8 falls from 0.884 to 0.582 under noise while ah=1 holds 0.805. What is untested (no overlap found) is the hardware analogue: hysteretic actuation as the "noise" on a cheap arm.
- **(d) Strongest simple baseline, and why it might fail here.**
  - Baseline: approach from one side, static dead-band compensation, and servo-gain tuning. LeRobot ships the SO follower with **P = 16, I = 0, D = 32**, so the first classical fix is to enable the servo's I-gain or add a host-side integral outer loop on Goal − Present. Also a scripted ArUco + IK pick-and-place.
  - Why it should fail on this platform: backlash roughly doubles under load (0.62° → 1.30°), and the sign of gravity torque at the shoulder and elbow flips across the workspace and on contact. Gear-stage play is invisible to the output-side encoder (Robo9), so joint-side compensation is partly blind.
  - How to show it: plot the residual end-point error after compensation against the predicted gravity-torque sign change, measured externally.
- **(e) First-week test, and what would kill it.**
  1. Measure ArUco and webcam noise with the arm unpowered.
  2. With `lerobot-replay`, replay one trajectory 30 times to 5 poses, with and without a payload of about 100 g, and record end-point scatter.
  3. Log Goal − Present for every joint.
  4. Repeat with I > 0, or a host-side integral. If that alone removes most end-point error, only the encoder-invisible component remains for the closed-loop arm of the study, so re-scope or pivot.

  Kill if hardware 3σ scatter is about 1 mm or less and insensitive to load direction (estimated threshold), while the first ACT rollouts fail mainly at grasping or perception (as in the ACT reproduction). Then hardware is not the bottleneck: pivot to B or C. Also kill, or escalate to a dial gauge or BRL motion capture, if ArUco noise is at least as large as the backlash signal.
- **(f) Hardware and compute.**
  - Follower arm: estimated £165–£200, from sourced component prices (Section 2).
  - Webcam: owned. Wrist camera: price not found, estimated £10–£25.
  - Printed ArUco markers.
  - Dial gauge: price not found.
  - Ask BRL for printing, a leader arm and motion-capture time.
  - ACT training needs 2–6 GB VRAM (sourced), so the RTX 3060 is fine. Diffusion needs 8–14 GB at batch 8 (sourced), so reduce the batch (estimated).
  - Recording 50 demos takes about 1–2 h (estimated from 60 s episodes plus resets).
  - Budget is tight: little or nothing remains for a second camera or a gauge if the follower costs about £190.
- **(g) Headline figure and outcomes.** "On SO-101, X% of ACT's end-point variance is mechanical. At ≤1 mm clearance, shortening the execution horizon from 100 to k steps recovers Y pp, vs Z pp for outer-loop compensation."
  - Best realistic outcome: a decomposition with confidence intervals, plus a clearance crossover curve.
  - Floor: an ISO 9283-style repeatability and backlash map of the SO-101, with a measured statement of when hardware error matters. That is useful to every LeRobot user even if no learned variant wins.
- **(h) Filter-by-filter verdict.**

  | Filter | Verdict | Why |
  |---|---|---|
  | 1 Robotics first | Pass | Arm physics is the subject. |
  | 2 Baseline | Conditional pass | Baseline named with a mechanism; it must be engineered properly. |
  | 3 Real gap | Pass | Only if clearances are a pre-registered sweep and no artificial perturbation is injected. |
  | 4 Method fit | Pass | Visual closed loop is justified because part of the error cannot be observed by joint sensors. |
  | 5 Nearest work | Partial | Nearest is DREAM-Chunk; novelty not established. |
  | 6 Feasibility | Pass (tight) | £200 follower-only; 12 GB GPU. |
  | 7 First-week test | Pass | Replay test as above. |
  | 8 Useful if the method loses | Pass | The error budget stands on its own. |

- **(i) Skills built.** Metrology and ISO 9283 statistics; servo identification and control; kinematics and IK; IL training and evaluation; experimental design. These map onto robotics, GNC and applied-ML roles.
- **(a′) Optional second design variable.** Absolute vs relative actions. LeRobot implements relative actions only for the pi family, so using them with ACT needs porting (estimated small effort). It is a cheap way to test whether re-referencing to the measured state each chunk absorbs steady-state offsets — [action_representations.mdx](https://github.com/huggingface/lerobot/blob/main/docs/source/action_representations.mdx) [V]
- **(j) Bristol/BRL people.** None appeared in my own sources. The parallel BRL staff notes ([brl_staff_facilities.md](./brl_staff_facilities.md), whose underlying sources are search-snippet or partial reads) report:
  - **Efi Psomopoulou** (UoB Senior Lecturer from 2025): robot control and physical interaction, tactile grasp control; co-investigator on the Horizon Europe MANiBOT bimanual-manipulation project.
  - **Nathan Lepora** (UoB Tactile Robotics Group): 3D-printed tactile sensors; sim-to-real RL on a UR5.
  - **Antonia Tzemanaki** (UoB): Dexterous Manipulation and Wearable Robotics group.
  - **Guanqun Cao** (UWE Lecturer): tactile sensing, VLAs.
  - A **UWE "Lecturer in Robot Learning"** post was advertised Jun–Jul 2026; appointee unknown.
  - The same notes found **no 2023–2026 BRL imitation-learning publication**.

  Psomopoulou is the closest fit for a physical-interaction and error-budget dissertation.

#### Candidate B: When can a cheap arm tell it is failing? Free servo telemetry vs learned progress or uncertainty signals under natural failures (ranked 2nd)
- **(a) Research question.** On an SO-101 running imitation-learning policies, which *naturally occurring* failure classes can be detected at runtime from servo telemetry (Present_Load, Present_Current, |Goal − Present|)? The classes in scope are empty grasp, slip, stall or collision, misplacement and wrong object. When does a learned signal detect them earlier or with fewer false alarms? Candidate learned signals are LeRobot reward or progress models (SARM, Robometer, TOPReward) and the dispersion of sampled action chunks.
- **(b) Why now.**
  - The FAR MSc thesis (12 Jun 2026) showed proprioceptive recovery helps but brings false positives, and it reported no detection metrics.
  - The SO-101 benchmark (Jun 2026) introduced recovery-aware metrics.
  - LeRobot v0.5.0 (Mar 2026) added SARM, and v0.6.0 (Jul 2026) added Robometer, TOPReward and rollout `sentry`/`dagger` logging with intervention flags.
- **(c) Nearest work, and what remains open.** The open slot is per-class detectability with natural failures on cheap hardware, comparing telemetry and learned signals head-to-head.
  - **lerobot_far**: a proprioceptive gripper monitor with retries. It reports changes in success rate, not precision or recall. Its README says it reads Present_Load, while its experiments doc describes gripper position or feedback, so the two conflict.
  - **CARE** (arXiv 2609.24118) and **RePO-VLA** (arXiv 2605.09410) address VLA failure recovery (titles only, PREPRINT).
  - **FAIL-Detect** (RSS 2025) is the strongest generic methods-level overlap for the "learned signal" arm.
    - It learns uncertainty scores (logpZO, RND) from training-data (observation, action) pairs, with no failure data, and thresholds them with conformal prediction.
    - It supports diffusion and flow policies.
    - Its released code covers robomimic simulation tasks only.
    - Source: [FAIL-Detect README](https://github.com/CXU-TRI/FAIL-Detect) [V]
    - Not checked: its real-robot results, and whether it was ever compared with proprioceptive or telemetry baselines.
  - Other generic detectors (e.g., Sentinel/STAC, FIPER) were **not verified**. Their repository README could not be found at the path guessed. Reading their methods sections is week-one work and the main novelty risk.
- **(d) Strongest simple baseline, and why it might fail here.**
  - Baseline: thresholds on joint load, current and position error, plus a check that the gripper closed, plus a webcam check that the object is in the bin.
  - Why it might fail: semantic failures (misplacement, wrong object) produce normal telemetry. Load readings may be coarse or noisy (estimated). FAR's false positives cost 10–20 pp at two positions. LeRobot also caps gripper torque at 50%, dropping to 25% on overload, so gripper load saturates by design. That may blur the line between "holding" and "stalled on an object" (inference).
  - How to show it: ROC curves per failure class.
- **(e) First-week test, and what would kill it.**
  1. Run a scripted grasp loop (no learning) on 3 objects: 30 successes and 30 staged empty or slipping grasps. Log telemetry at 30 Hz and compute AUROC.
  2. Run 20 ACT rollouts to see the natural failure mix.

  Kill if telemetry AUROC is about 0.95 or higher for the failure classes that make up about 80% or more of natural failures (estimated thresholds). That leaves no room for learned signals, and the project collapses into a replication of lerobot_far.
- **(f) Hardware and compute.**
  - The same arm as Candidate A.
  - ACT and Diffusion training as above.
  - Sampling N chunks for dispersion multiplies inference cost. BID roughly doubled compute in its setting (sourced); N× is an estimate.
  - Size and VRAM of the reward or progress models: not found. They may need cloud or BRL GPUs.
- **(g) Headline figure and outcomes.** "Servo telemetry alone catches X% of natural SO-101 failures at ≤Y% false alarms; a learned progress model adds Z% recall or W s lead time on class C."
  - Best realistic outcome: a detectability matrix plus a measured recovery gain under natural conditions.
  - Floor: a labelled natural-failure dataset (about 200 rollouts, estimated) and a characterised telemetry-only detector, including what its false alarms cost.
- **(h) Filter-by-filter verdict.**

  | Filter | Verdict | Why |
  |---|---|---|
  | 1 Robotics first | Pass | The arm's sensing and behaviour are the subject. |
  | 2 Baseline | Pass | Telemetry thresholds named, with a mechanism for the silent classes. |
  | 3 Real gap | Pass | Only on natural failures; staged interference as a secondary test. FAR's large gains were all staged. |
  | 4 Method fit | Pass | Generative policies supply samples; telemetry is free. |
  | 5 Nearest work | Partial | Closest on SO-101 is a 4-month-old MSc thesis (lerobot_far). Closest generic method is FAIL-Detect (RSS 2025, simulation code). Telemetry vs learned on a cheap arm: no overlap found, but the search was incomplete. |
  | 6 Feasibility | Pass | Reward-model VRAM unknown. |
  | 7 First-week test | Pass | Scripted grasp loop plus natural-failure census. |
  | 8 Useful if the method loses | Pass | The detectability map is useful regardless. |

- **(i) Skills built.** Embedded telemetry; ROC and power analysis; generative policies; runtime monitoring for consequential autonomy decisions. This is relevant to autonomy and defence roles.
- **(j) Bristol/BRL people.** None appeared in my own sources. From the parallel BRL notes ([brl_staff_facilities.md](./brl_staff_facilities.md)): Psomopoulou (tactile grasp control: grasp and slip detection is the overlap) and Cao (UWE; tactile sensing, VLAs) are the nearest. There is no known BRL work on runtime failure detection.

#### Candidate C: Pre-registered scripted-vs-learned boundary map on SO-101 (fallback, ranked 3rd)
- **(a) Research question.** On an externally fixed task list, where does a calibrated scripted controller beat ACT (and SmolVLA where it fits in 12 GB) on SO-101? Which task properties predict the crossover: deformability, clearance, clutter or object optics?
- **(b) Why now.** Low independent success rates (Jun 2026), and LeRobot's IK and URDF tooling (v0.4.0+) make the baseline cheap to build.
- **(c) Nearest work.** None found (search incomplete).
- **(d) Baseline.** The baseline *is* the subject; Section 5 lists the failure classes.
- **(e) First-week test.** Build scripted pick-and-place for one rigid block. If it cannot become reliable because calibration or backlash defeats IK accuracy, that becomes Candidate A rather than killing the area.
- **(f) Hardware and compute.** The same arm. Compute: ACT only.
- **(g) Headline figure and outcomes.** "Scripted X% vs ACT Y% on rigid tasks; learning wins only on {…}." Floor: a head-to-head table with confidence intervals.
- **(h) Filter-by-filter verdict.**

  | Filter | Verdict | Why |
  |---|---|---|
  | 1 Robotics first | Partial | Task physics decides the winner, but the work is evaluation-heavy. |
  | 2 Baseline | Pass | By construction. |
  | 3 Real gap | Pass | Only with an external task list. |
  | 4 Method fit | Partial | No new method. |
  | 5 Nearest work | Unknown | Search incomplete. |
  | 6 Feasibility | Pass | |
  | 7 First-week test | Pass | |
  | 8 Useful if the method loses | Pass | |

  Low novelty. Best used as a chapter inside A or B.
- **(i) Skills built.** Calibration, IK, perception pipelines, evaluation.
- **(j) Bristol/BRL people.** None appeared in my own sources. The parallel BRL notes found no 2023–2026 BRL imitation-learning publication ([brl_staff_facilities.md](./brl_staff_facilities.md)).

#### Candidate D: 3pi+ with downward optical flow as a slip instrument (pass to the adaptive-control stream; not an imitation-learning candidate)
- **(a) Research question.** At the 3pi+'s achievable accelerations, how large are slip-induced odometry and velocity errors across ordinary lab surfaces and payloads? Does flow-based feedback or adaptation reduce path error beyond a well-tuned encoder controller with acceleration limits?
- **(b) Why now.** No specific 2025–26 release found (search exhausted), so this is weak.
- **(c) Nearest work.** Not searched.
- **(d) Baseline.** Encoder PID with acceleration limiting and gyro heading, or an EKF fusing encoders, gyro and flow. Likely strong.
- **(e) First-week test.** On 3 surfaces, do 20 straight runs and in-place turns at maximum acceleration, comparing encoders against overhead-ArUco ground truth. Kill if slip error is below about 2% everywhere (estimated threshold); then both the sensor and the adaptive-control premise lose their footing.
- **(f) Hardware and compute.** Sensor about £20–£30 (estimated); webcam owned.
- **(h) Filter-by-filter verdict.**

  | Filter | Verdict |
  |---|---|
  | 1 Robotics first | Pass |
  | 2 Baseline | Risky |
  | 3 Real gap | Pass on ordinary floors |
  | 4 Method fit | Weak for learning |
  | 5 Nearest work | Unknown |
  | 6 Feasibility | Pass |
  | 7 First-week test | Pass |
  | 8 Useful if the method loses | Pass, as a characterisation |

  Keep it as an instrument.
- **(j) Bristol/BRL people.** Paul O'Dowd, per the parallel BRL notes ([brl_staff_facilities.md](./brl_staff_facilities.md)): MSc Robotics Programme Director and Robotics Science & Systems unit director. His GitHub has a run of Pololu 3pi+ add-on boards (an I2C extension board, IR communication boards, a joystick board and an M5Core2 hat), so he is the natural person to ask about mounting a flow sensor on the 3pi+.

#### Tempting but fails (with the filter it fails)

| Idea | Fails | Why |
|---|---|---|
| RTC or latency-aware chunking as the main question on SO-101 | Filter 5, likely 3 | At least 7 papers from Jun 2025 to Aug 2026. Quasi-static tasks show no success loss from latency: sync vs async SmolVLA both ≈78%. |
| Quantised on-device inference on the Jetson Orin Nano driving the SO-101 | Filters 3 and 5 | The desktop RTX 3060 can drive a tethered arm, so the latency gap is manufactured. 2608.03938 and 2411.11406 exist, and lerobot_far already ran a Jetson client. |
| VLA benchmarking (SmolVLA vs ACT vs π0.5) on new SO-101 tasks | Filters 4 and 5; 6 locally | The June 2026 benchmark exists. π0.5 fine-tuning needs 24–40 GB. |
| World-model policies (VLA-JEPA, FastWAM, LingBot-VA) on SO-101 | Filters 4 and 6 | Chosen for recency. VRAM unknown, likely above 12 GB. |
| Leader-arm vs phone-teleop data-quality study | Filter 6, weak on 1 | Leader plus follower costs over £200 in the UK. |
| Learned backlash compensator on its own | Filter 2 | Classical dead-band and outer-loop compensation is the natural, probably sufficient baseline. Robo9 already treats STS3215 compensation. |
| LeKiwi, Koch, XLeRobot or Grabette platforms | Filter 6 | $482–$660 or €490+ on sourced BOMs. |
| 3pi+ learned reactive navigation with VL53L5CX or LiDAR | Filter 2 | VFH, potential fields and SLAM. |
| 3pi+ visual imitation with the XIAO ESP32-S3 Sense | Filters 2 and 4 | Colour or ArUco tracking; cheap flow chip. |
| 3pi+ UWB multi-robot coordination | Filter 6 | Needs more robots and anchors; BRL unconfirmed. |
| Evaluation statistics only ("how many trials does a cheap-arm benchmark need?") | Filter 1 | It is statistics, not robotics. Still, build its power analysis into A, B or C. |
| Unresolved, not failed: HIL-SERL RL vs DAgger per human-hour on SO-101 | Filter 2 risk | DAgger is a strong baseline; LeRobot calls HIL-SERL "very sensitive to the intervention strategy". The original HIL-SERL code targets a **Franka with impedance controllers over ROS** (RAM insertion, USB pick-and-insert, handover, egg flip) and claims "near-perfect success rates". Its README cites it as arXiv 2410.21845 (2024); a later journal version was not verified — [hil-serl README](https://github.com/rail-berkeley/hil-serl) [V]. Whether the method's benefits survive a position-controlled, backlash-prone servo arm with no impedance control is untested (no overlap found); this is a real physical difference. Noise-space RL fine-tuning not verified. Pass to the RL post-training stream. |

#### What to ask BRL for, exactly
1. Any SO-100 or SO-101 leader or follower arms, or Koch or OMX arms. This saves the £200 or provides a leader for demonstrations.
2. 3D-printer access with PLA+ and TPU 95A, for the SO-101 printed parts.
3. Motion-capture time (OptiTrack or Vicon), or a dial indicator and height gauge, for ISO 9283-style ground truth.
4. A GPU with at least 24 GB, on a cluster or workstation, for SmolVLA at normal batch size or π0.5 fine-tuning.
5. A supervisor or co-supervisor in manipulation or physical interaction. Psomopoulou, Lepora and Tzemanaki at UoB, and Cao at UWE, per the parallel BRL notes. For the 3pi+ instrument idea, O'Dowd. Also ask whether the UWE "Lecturer in Robot Learning" post advertised Jun–Jul 2026 has been filled.

### Gaps
- The novelty of A and B is "no overlap found in an incomplete search". Not searched (shared search budget exhausted):
  - Generic runtime failure-detection literature for diffusion and flow policies.
  - Backlash-aware or compliance-aware imitation learning.
  - Scripted-vs-learned comparisons.
  - The HIL-SERL paper.
  - Noise-space RL fine-tuning.
  - BRL staff.
- BRL staff names come from the parallel notes ([brl_staff_facilities.md](./brl_staff_facilities.md)), which are themselves mostly search-snippet level. BRL equipment (arms, printers, motion capture, GPUs) is not verified.
- Sentinel/STAC and FIPER code READMEs: not found at the repository paths tried; methods not verified.
- All thresholds in the kill tests are estimates and must be set against measured ArUco noise and the task tolerances.

#### Source access log
- **Read in full via GitHub (model-summarised fetch):**
  - LeRobot code and docs: [README](https://github.com/huggingface/lerobot); [docs directory](https://github.com/huggingface/lerobot/tree/main/docs/source); docs pages so101, phone_teleop, hardware_guide, rtc, il_robots, hilserl, feetech; [feetech tables.py](https://github.com/huggingface/lerobot/blob/main/src/lerobot/motors/feetech/tables.py).
  - HF blog (GitHub copies): lerobot-release-v040, lerobot-release-v050, lerobot-release-v060, smolvla, grabette, _blog.yml.
  - Hardware repos: [TheRobotStudio SO-ARM100](https://github.com/TheRobotStudio/SO-ARM100); [LeKiwi README](https://github.com/SIGRobotics-UIUC/LeKiwi) and [BOM.md](https://github.com/SIGRobotics-UIUC/LeKiwi/blob/main/BOM.md); [Koch low_cost_robot](https://github.com/AlexanderKoch-Koch/low_cost_robot); [XLeRobot](https://github.com/Vector-Wangel/XLeRobot); [pimoroni/pmw3901-python](https://github.com/pimoroni/pmw3901-python); [Pololu 3pi+ 32U4 library](https://github.com/pololu/pololu-3pi-plus-32u4-arduino-library).
  - Community and student projects: [so101-error-anatomy](https://github.com/tucasteixeira-debug/so101-error-anatomy); [khushi02 SO-101 ACT](https://github.com/khushi02/so101-imitation-learning-ACT); [lerobot_far README](https://github.com/HyuanTan/lerobot_far) and [experiments.md](https://github.com/HyuanTan/lerobot_far/blob/main/docs/experiments.md); [Revisiting-Open-Loop code index](https://github.com/Michaelszeng/Revisiting-Open-Loop-Execution-in-Robotics).
- **Read verbatim via curl of raw GitHub files [V]** (after the coordinator's tip):
  - LeRobot docs: async.mdx, smolvla.mdx, third_party_sensors.mdx, in full; action_representations.mdx, processors_robots_teleop.mdx, omx.mdx and rebot_b601.mdx, searched for key terms.
  - LeRobot code: `src/lerobot/robots/so_follower/so_follower.py` and `config_so_follower.py`.
  - Paper code READMEs: [FAIL-Detect](https://github.com/CXU-TRI/FAIL-Detect), [real-time-chunking-kinetix](https://github.com/Physical-Intelligence/real-time-chunking-kinetix), [bid_diffusion](https://github.com/YuejiangLIU/bid_diffusion), [bid_lerobot](https://github.com/Jubayer-Hamid/bid_lerobot), [hil-serl](https://github.com/rail-berkeley/hil-serl).
  - Not found at the guessed path: agiachris/STAC README.
- **Consulted (local):** the parallel BRL notes [brl_staff_facilities.md](./brl_staff_facilities.md), for staff names only; their own sources are mostly search-snippet level.
- **Partly read:** [LeRobot releases page](https://github.com/huggingface/lerobot/releases) (first 100k of 137k characters).
- **Abstract-level via a third party:** DP-ACT (*Scientific Reports* 2026), via [a course-journal issue](https://github.com/deepcharles/macs3050-journal/issues/82).
- **Search-result summaries only (pages not opened):**
  - Papers: SmolVLA paper; 2606.08881; DREAM-Chunk; RTC (NeurIPS 2025); BID (ICLR 2025); 2507.09061 / ICLR 2026; 2608.15938; Legato 2602.12978; 2411.11406; 2608.03938; ALPHA-α 2411.09942; ACT RSS 2023 PDF; the backlash test-stand paper (ScienceDirect/PMC).
  - Other: Robo9 STS3215 tests; Thomas Wolf X post; hackathon pages; every vendor listing (Unmanned Tech, RobotShop UK, Amazon UK, eBay UK, Seeed, Hiwonder, WowRobo, OpenELAB, roboticscenter.ai); CNX Software; MarkTechPost; TechApple; the Jetson blog summary; Jetson AI Lab.
- **Title or listing only:** arXiv 2512.05964, 2605.25537, 2607.24008, 2511.04421, 2506.05165, 2401.17698, 2609.24118, 2605.09410; MakerMods; PartaBot follower-only; Robo9 compensation article; Pololu 3pi+ user's guide.
- **Could not access (egress policy denied):** arxiv.org, huggingface.co, pmc.ncbi.nlm.nih.gov, seeedstudio.com, unmannedtechshop.co.uk, robonine.com, shop.pimoroni.com, thepihut.com, physicalintelligence.company, tonyzhaozh.github.io (and therefore probably all *.github.io project pages). The GitHub REST API for huggingface/lerobot was also refused (repository not attached to the session).
- **Not done because the search budget ran out:**
  - UK sensor prices.
  - Diffusion Policy limitations.
  - Flow-matching variants; data quality, scaling and curation studies.
  - Failure-detection literature.
  - HIL-SERL and noise-space RL papers.
  - Scripted-vs-learned comparisons.
  - Optical-flow slip estimation on small robots.
  - BRL people.
  - UK LeRobot community events.
  - LeRobot GitHub issues on SO-101 overheating and calibration drift.
