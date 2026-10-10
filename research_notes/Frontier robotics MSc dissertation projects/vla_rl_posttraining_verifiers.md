# Small VLAs, RL post-training of robot policies, and sample-then-verify / best-of-k / tail objectives (2024–2026) — where MaxRL/TailRL belong in robotics, and which dissertation questions survive (research date: Sat 10 Oct 2026)

**Read this first: how much I could check, and how to read the tags**
- **Blocked hosts (session egress policy).** These hosts were blocked by policy (proxy 403, or WebFetch DNS failure): arxiv.org, huggingface.co (including LeRobot's hosted docs and model cards), openreview.net, every *.github.io project page, pi.website (Physical Intelligence blog), jetson-ai-lab.com, developer.nvidia.com, science.org, frontiersin.org, ri.cmu.edu, publications.ri.cmu.edu, semanticscholar.org, proceedings.mlr.press, roboticsproceedings.org and alphaxiv.org. I did **not** route around the blocks.
- **What I read instead.** github.com and raw.githubusercontent.com were reachable, so primary sources are READMEs, LeRobot doc sources, Jetson AI Lab page sources, and project-page HTML read from the authors' own GitHub repos.
- **Full papers read from author-hosted PDFs.** Following the coordinator's tip, I downloaded PDFs the authors committed to their own GitHub repos (via raw.githubusercontent.com) and read them locally:
  - **HIL-SERL**, from `hil-serl/hil-serl.github.io/static/hil-serl-paper.pdf`: pages 1–6, 9–14 and 19–21 read (abstract, method, task descriptions, results tables, discussion and limitations).
  - **TailRL**, from `Zanette-Labs/TailRL-website/static/TailRL.pdf`: pages 1–15 read (all of the main text through the conclusion; appendices not read).
  - **MaxRL preprint**, from `Zanette-Labs/MaxRL/site/static/pdf/MaxRL_Preprint.pdf`: pages 1–8 and 11–13 read (method, estimator, ImageNet and maze set-up, conclusion and limitations). This is the preprint; the ICML 2026 camera-ready may differ.
  - **"Can Large Reasoning Models Self-Train?" (SRT)**, from the same MaxRL repo: pages 1–2 read (abstract and introduction).
- **Search budget ran out.** The web-search budget (200 calls per turn, shared by all parallel agents) ran out partway through. These planned searches were **not** run:
  - real-robot numbers for PA-RL, ConRFT and πRL (HIL-SERL's numbers were later verified from the authors' PDF);
  - π*0.6/RECAP;
  - multiple-choice learning (MCL) and winner-takes-all (WTA) losses, and learned MPPI/CEM proposals;
  - follow-ups to Winfield's consequence engine;
  - community reports of VLAs on Jetson Orin Nano;
  - a targeted search for "robot proposals trained for best-of-k".
  Treat every "not found" below as **search incomplete, not absent**.
- **Reading-status tags.** [README/doc read] = primary GitHub source read in full. [page source read] = project page HTML read from GitHub. [search summary] = search-engine abstract or snippet only, not opened. [listing only] = title or ID seen in results only.
- **Paper status.** arXiv-only papers are marked PREPRINT; peer-reviewed ones give venue and year.

## Q1. Which VLAs/generalist policies can be fine-tuned on a 12 GB RTX 3060 and run on a Jetson Orin Nano 8 GB? (sourced requirements only)

### Takeaway
- **12 GB RTX 3060.** Only sub-1B models have documented requirements in this range:
  - VLA-Adapter's README puts the RTX 3060 by name in a "10–12 GB" tier (9.6 GB, or 8.6 GB elsewhere in the same README, at batch 1 with LoRA rank 64).
  - LeRobot documents SmolVLA training at ~10–16 GB (batch 8) and ~2 GB at inference.
  - Every π0-class or larger model (openpi π0/π0-FAST/π0.5, GR00T N1.7, OpenVLA, OpenVLA-OFT, and X-VLA in LeRobot's grouping) is documented above 12 GB for fine-tuning.
- **Jetson Orin Nano 8 GB.** I found no documented VLA run. NVIDIA's Jetson AI Lab VLA tutorials (GR00T N1.7, openpi π0.5) target Jetson AGX Thor only.

### Cited Findings
**Requirements table** (all "sourced" unless marked not found; batch sizes as documented)

| Model | Size | Fine-tuning requirement | Inference requirement | Jetson / edge statement | Status |
|---|---|---|---|---|---|
| π0 / π0-FAST / π0.5 (openpi) | not stated in README | LoRA ">22.5 GB" (e.g. RTX 4090); full ">70 GB" (A100 80GB/H100); "These estimations assume a single GPU" — [openpi README](https://github.com/Physical-Intelligence/openpi) | ">8 GB" (e.g. RTX 4090) — [openpi README](https://github.com/Physical-Intelligence/openpi); LeRobot: "PI0 occupies 14GB of memory at inference time" — [LeRobot async.mdx](https://github.com/huggingface/lerobot/blob/main/docs/source/async.mdx) | Jetson not mentioned in openpi README; Jetson AI Lab has a π0.5-on-**Thor** tutorial only — [Jetson AI Lab source](https://github.com/NVIDIA-AI-IOT/jetson-generative-ai-playground/tree/main/src/content/tutorials/vla) | README read |
| GR00T N1.7 (current GA; N1.6/N1.5 previous) | "Base model (3B params)" | "1 or more GPUs with 40 GB+ VRAM recommended" (H100/L40; A6000 works but slower) | "1 GPU with 16 GB+ VRAM (e.g., RTX 4090, L40, H100, Jetson AGX Thor/Orin, DGX Spark)" | Orin Nano not mentioned; tested on JetPack 7.2 — [Isaac-GR00T README](https://github.com/NVIDIA/Isaac-GR00T) | README read |
| OpenVLA | "all 7.5 billion parameters" | LoRA on "a single A100 GPU with 80 GB VRAM", works "as long as it has at least ~27 GB"; "~72 GB" at batch 16; full FT "a full node of 8 A100 GPUs" | not found (README gives no inference memory) | no consumer-GPU or Jetson mention — [OpenVLA README](https://github.com/openvla/openvla) | README read |
| OpenVLA-OFT | (OpenVLA-based) | "Between 1-8 GPUs with 27-80 GB" | "~16 GB VRAM" (LIBERO), "~18 GB VRAM" (ALOHA) | none — [OpenVLA-OFT README](https://github.com/moojink/openvla-oft) (PREPRINT arXiv 2502.19645) | README read |
| SmolVLA | "our pretrained 450M model" | "20k steps will roughly take ~4 hrs on a single A100"; ~50 episodes recommended ("25 ... was not enough") — [LeRobot smolvla.mdx](https://github.com/huggingface/lerobot/blob/main/docs/source/smolvla.mdx); training VRAM "~10–16GB" at batch 8, "RTX 4080+ / L4 / A10G" — [LeRobot hardware_guide.mdx](https://github.com/huggingface/lerobot/blob/main/docs/source/hardware_guide.mdx) | "SmolVLA requires only ~2GB" — [LeRobot async.mdx](https://github.com/huggingface/lerobot/blob/main/docs/source/async.mdx) | Jetson not named; code comment "RTC optional, use when running on low power hardware" — [smolvla.mdx](https://github.com/huggingface/lerobot/blob/main/docs/source/smolvla.mdx) | doc read |
| VLA-Adapter | Qwen2.5-0.5B backbone ("Tiny-scale", 0.5B) | 10–12 GB tier lists "NVIDIA GeForce RTX 2080Ti, 3060, 3080, 4070, 4080, and 5070"; batch 1, LoRA r64: "only requires 9.6GB of VRAM" vs "requir[es] only 8.6GB" (README contradicts itself); 24 GB tier ~20 GB; RTX 5090 32 GB: LIBERO-Object "99.2% ... in just 8 hours" | not found as VRAM; "0.036s" to generate 8 action chunks (hardware not stated in what I read) | Jetson not mentioned; ALOHA real-world support added 2026-03-16 — [VLA-Adapter README](https://github.com/OpenHelix-Team/VLA-Adapter) | README read |
| X-VLA | "X-VLA-0.9B"; ICLR 2026 | not found in README; LeRobot groups `xvla` with "Large VLA ... ~24–40GB" at batch 8 — [hardware_guide.mdx](https://github.com/huggingface/lerobot/blob/main/docs/source/hardware_guide.mdx) | not found | "edge devices" mentioned generically, no Jetson — [X-VLA README](https://github.com/2toinf/X-VLA) | README read |
| NORA / NORA-Long / NORA-1.5 | not found in README | not found | not found | none; supports LeRobot dataset format; WidowX demos — [NORA README](https://github.com/declare-lab/nora) | README read |
| MiniVLA | Qwen2.5 0.5B backbone + residual-VQ action chunks (2024-12-09) | not found (examples use 8 GPUs) | not found; "Converting and deploying MiniVLA models ... is not supported yet!" — [openvla-mini README](https://github.com/Stanford-ILIAD/openvla-mini) | none | README read |
| Evo-1 (in LeRobot) | InternVL3 backbone; params not stated | "measured run used two H100 GPUs" | not found | none — [LeRobot evo1.mdx](https://github.com/huggingface/lerobot/blob/main/docs/source/evo1.mdx) | doc read |
| FLUX 3 Action (Black Forest Labs, 2026) | "a 7B world action model"; SO-101 checkpoint `flux-3-action-so101` | single-GPU default; "still needs GPU and robot validation" — [LeRobot flux3.mdx](https://github.com/huggingface/lerobot/blob/main/docs/source/flux3.mdx) | text-encoder CPU offload "moves about 8.3 GiB" | `minimum_jetson: "Thor"` — [Jetson AI Lab model page source](https://github.com/NVIDIA-AI-IOT/jetson-generative-ai-playground/blob/main/src/content/models/flux-3-action.md) | doc/page read |

**Other sourced points**
- **LeRobot training-VRAM bands (batch 8, AdamW):**
  - Light BC (`act`, `vqbet`, `tdmpc`): "~2–6GB", with "Laptop GPU (RTX 3060)" listed as a suitable starter GPU.
  - Diffusion: "~8–14GB" ("RTX 4070+").
  - Large VLAs (`pi0`, `pi0_fast`, `pi05`, `xvla`): "~24–40GB" ("24 GB tight at BS 1").
  - `groot`/`eo1`: "~24–40GB".
  - The guide says "Memory scales roughly linearly with batch size". When memory-bound it suggests smaller batches, gradient accumulation, or `freeze_vision_encoder=True` for SmolVLA. It calls its time figures "order-of-magnitude" (±50%).
  - Source: [LeRobot hardware_guide.mdx](https://github.com/huggingface/lerobot/blob/main/docs/source/hardware_guide.mdx) [doc read].
- **LeRobot LoRA (PEFT).** It supports LoRA for "pre-trained policies (e.g., SmolVLA, π₀, ...)" but gives no VRAM savings figures — [peft_training.mdx](https://github.com/huggingface/lerobot/blob/main/docs/source/peft_training.mdx) [doc read].
- **openpi platform support.** "tested with Ubuntu 22.04, we do not currently support other operating systems". The PyTorch port (Sept 2025) does **not** support π0-FAST, mixed precision, FSDP, LoRA or EMA. Remote inference is supported: "the model can run on a different server and stream actions to the robot via a websocket connection" — [openpi README](https://github.com/Physical-Intelligence/openpi) [README read].
  - Note: the student's desktop runs Windows, and openpi only supports Ubuntu 22.04.
- **Jetson AI Lab VLA tutorials are Thor-only.** The folder contains only `groot_n17_on_thor.md` and `openpi_on_thor.md` — [folder listing](https://github.com/NVIDIA-AI-IOT/jetson-generative-ai-playground/tree/main/src/content/tutorials/vla).
  - GR00T N1.7 on Thor (4 denoising steps, batch 1): PyTorch eager "~126 ms" (8.0 Hz) down to TensorRT mixed NVFP4 "~40 ms" (25.1 Hz). "Engines are ... GPU-architecture-specific and must be rebuilt for a different device" — [groot_n17_on_thor.md](https://github.com/NVIDIA-AI-IOT/jetson-generative-ai-playground/blob/main/src/content/tutorials/vla/groot_n17_on_thor.md).
  - π0.5 (`pi05_libero`, action horizon 10) on Thor: PyTorch BF16 "~132 ms" total; TensorRT FP8 "~54 ms"; FP8+NVFP4 "~49 ms" — [openpi_on_thor.md](https://github.com/NVIDIA-AI-IOT/jetson-generative-ai-playground/blob/main/src/content/tutorials/vla/openpi_on_thor.md).
  - Neither page mentions Orin or Orin Nano [page sources read].
- **jetson-containers.** Has OpenVLA and LeRobot containers (OpenVLA images 6.5 GB, 2024-08-26, and 6.7 GB, 2025-02-27; these are image sizes, not runtime memory). Neither README names a tested device, speed or Orin Nano memory note — [openvla package](https://github.com/dusty-nv/jetson-containers/tree/master/packages/physicalAI/vla/openvla), [lerobot package](https://github.com/dusty-nv/jetson-containers/tree/master/packages/physicalAI/lerobot) [README read].
- **LeRobot async inference.** It decouples "action prediction from action execution", with the server assumed to have "better computational resources than the robot controller". It works with "all the policies supported by LeRobot". No latency figures are given — [async.mdx](https://github.com/huggingface/lerobot/blob/main/docs/source/async.mdx) [doc read].
- **New VLA-class policies in LeRobot (2026).** The docs folder now includes `eo1`, `evo1`, `fastwam`, `flux3`, `lingbot_va`, `molmoact2`, `multi_task_dit`, `xvla` (via hardware guide) and others. It has no Jetson page — [LeRobot docs listing](https://github.com/huggingface/lerobot/tree/main/docs/source) [listing only].

### Inferences
- **RTX 3060 (12 GB), fine-tuning:**
  - **ACT:** sourced as fitting.
  - **Diffusion policy:** estimated to fit at batch < 8. Reasoning: ~8–14 GB at batch 8, and memory roughly linear in batch.
  - **SmolVLA:** estimated to fit only at reduced batch and/or with a frozen vision encoder. Reasoning: ~10–16 GB at batch 8, plus the guide's own memory-saving advice.
  - **VLA-Adapter:** sourced as fitting at batch 1 with LoRA. The README's 9.6 vs 8.6 GB contradiction should be settled by measurement.
  - **Ruled out on documented figures:** π0/π0.5 (LoRA >22.5 GB), GR00T (40 GB+), OpenVLA (~27 GB+) and OFT (27–80 GB).
- **Jetson Orin Nano 8 GB:**
  - SmolVLA's documented ~2 GB inference footprint (hardware unspecified) is the only VLA number below 8 GB. It is *estimated* to fit, but the speed is **not found**.
  - The documented architecture that fits the hardware is LeRobot async inference: the Orin Nano or a PC as robot client, and the policy server on the RTX 3060 desktop. Its latency over the student's network is not found and would need measuring.
- **Fit to the student's platforms.** VLAs are irrelevant to the 3pi+ (no camera). For an SO-101, a VLA is not needed for the one candidate that survives (Q7); ACT or diffusion policies are cheaper, and LeRobot documents ACT on an RTX 3060.
- **If a VLA-scale project were pursued anyway,** ask BRL exactly for at least one 40–80 GB GPU (A100/H100/L40-class) for fine-tuning. For VLA RL, ask for 4–8 such GPUs; see Q2 for RIPT-VLA's 3–6 GPUs and SimpleVLA-RL's 8×A800 80 GB.

### Gaps
- **Not found:** any documented VLA (SmolVLA, VLA-Adapter, OpenVLA, π0) running on a Jetson Orin Nano 8 GB, with speed or memory. Community reports were not searched (budget exhausted).
- **Not found:** OpenVLA inference memory in its README; X-VLA, NORA and MiniVLA requirements; VLA-Adapter inference VRAM.
- **Not accessed:** the SmolVLA paper (PREPRINT arXiv 2506.01844) and Hugging Face blog/model cards (huggingface.co blocked), so their claims about consumer GPUs and CPUs are unverified.
- **Not checked:** the dates of GR00T N1.7's release and X-VLA's ICLR acceptance (READMEs give none).

## Q2. Which RL post-training methods have real-robot evidence at a scale an MSc could reproduce?

### Takeaway
- **Reproducible at MSc scale (documented):** only HIL-SERL and DSRL have documented real-robot paths at that scale.
  - **HIL-SERL**, verified from the paper: 100% success on Franka tasks after 1–2.5 h of real-world training (6 h for timing-belt assembly), with 20–30 demonstrations, SpaceMouse corrections and a single RTX 4090. LeRobot packages it for SO-100/101 (one NVIDIA GPU plus a gamepad; "a few hours" claimed).
  - **DSRL:** RL over a diffusion policy's noise space, needing only black-box access to the policy.
- **Out of reach:** the VLA-scale methods (SimpleVLA-RL, VLA-RL, RIPT-VLA, πRL) are simulation-first and multi-GPU (3–16 GPUs, A800-class). Their real-robot evidence is thin or undocumented in their READMEs.
- **Relevant to the student's code:** RIPT-VLA uses exactly the leave-one-out (RLOO) advantage the student has implemented.

### Cited Findings
- **HIL-SERL (Luo, Xu, Wu, Levine; UC Berkeley).** The paper is dated "Oct 2024" (PREPRINT arXiv 2410.21845). The Science Robotics venue was **not verified** (science.org blocked).
  - Sources: [paper PDF hosted in the authors' project repo](https://github.com/hil-serl/hil-serl.github.io/blob/main/static/hil-serl-paper.pdf) [pp. 1–6, 9–14, 19–21 read]; [rail-berkeley/hil-serl README](https://github.com/rail-berkeley/hil-serl) [README read].
  - **Headline claims.**
    - Abstract: "near-perfect success rates and fast cycle times within just 1 to 2.5 hours of training", and "an average 2x improvement in success rate and 1.8x faster execution" over imitation-learning baselines.
    - Table 1a lists per-task training times of 1–2.5 h for every task except **Timing Belt Assembly, which took 6 h**. So the "1 to 2.5 hours" headline excludes the hardest task.
    - Table 1a averages: BC (HG-DAgger with the same human data) reaches 49.7% success; HIL-SERL reaches 100% ("+101%"). Cycle time drops from 9.6 s to 5.4 s.
    - All metrics come from "100 trials per task", except IKEA whole assembly (10 trials).
  - **Compute.** Training time "includes all scripted motion, policy rollouts, intended stops, as well as onboard computation which is carried on a single NVIDIA RTX 4090 GPU".
  - **Robot and method.**
    - Franka arm(s): single- and dual-arm tasks such as RAM, SSD and USB insertion, IKEA, car dashboard, handover, timing belt, Jenga whipping and object flipping.
    - Actor at "10 Hz"; ImageNet-pretrained ResNet-10 encoder; RLPD off-policy learner.
    - SpaceMouse human interventions; demo buffer "usually on the range of 20-30" demonstrations.
    - A binary reward classifier per task, trained from positive and negative demonstrations. The authors "also collect additional potential false positive or false negative examples if necessary". This shows reward-verifier errors have to be patched by hand in real-robot RL.
  - **Ablation (Table 1b): without human corrections, learning fails.** "HIL-SERL no demo no itv" scores 0 on RAM insertion, dashboard assembly and object flipping. "HIL-SERL no itv" averages 49. On the same tasks, Diffusion Policy averages 34, BC 31 and HG-DAgger 39 (DP and BC trained on 200 demonstrations). The strongest other RL baseline, IBRL, averages 57: 75 / 0 / 95.
  - **Limitations quoted (Section 6):**
    - "it remains uncertain whether this method can be further extended to tasks with significantly longer horizons, where the sample complexity issue becomes more pronounced";
    - "we did not perform extensive randomization in our experiments, nor did we test the method's generalization capability in unstructured environments";
    - "each task still requires training from scratch".
  - **Its classical-baseline argument (authors' claim, Section 5.2).** Mixed-integer contact planners (Hogan & Rodriguez 2016; Aceituno-Cabezas & Rodriguez 2020) "can quickly become computationally intractable as the planning horizon increases" and "require accurate state estimators".
- **LeRobot HIL-SERL (SO-100/SO-101).**
  - **Hardware:** "A Nvidia GPU" plus "A real robot with a follower and leader arm (optional if you use the keyboard or the gamepad)".
  - **Training claim:** a policy "on a real robot in just a few hours".
  - **Suggested starter tasks:** "Push cube to a goal region" or "Pick and lift cube"; episodes about 5–10 s.
  - **Example configs:** 15 demonstration episodes and 20 reward-classifier episodes; the reward classifier is ResNet-10 based.
  - **Architecture:** actor/learner processes over gRPC running SAC; 128×128 images by default ("64×64 if you need to save GPU memory").
  - **Caveats quoted:** "Avoid extremely long horizon tasks"; it "is very sensitive to the intervention strategy" and takes "a few runs" to learn.
  - Source: [LeRobot hilserl.mdx](https://github.com/huggingface/lerobot/blob/main/docs/source/hilserl.mdx) [doc read].
- **DSRL — "Steering Your Diffusion Policy with Latent Space RL", CoRL 2025.**
  - **Method:** RL over the diffusion policy's starting noise. It "does not change the policy's weights". Variants are DSRL-NA (distils "a Q-function learned on the original action space") and DSRL-SAC. Diffusion policies must be sampled with DDIM.
  - **Tuning caveat:** `action_magnitude` and `utd` "typically need tuning on new environments".
  - Source: [ajwagen/dsrl README](https://github.com/ajwagen/dsrl) [README read].
  - **Real-world claims (project page):**
    - It "enables effective real-world autonomous policy improvement" and needs "only black-box access to the BC policy".
    - A Franka single-task diffusion policy "trained on 10 demonstrations before and after DSRL adaptation".
    - WidowX multi-task (BridgeData V2) mushroom/blocks/drawer.
    - Steering π0 with "the publically available DROID weights" (toaster, spoon tasks).
    - No episode counts, times or success rates are given in the HTML.
  - Source: [project page source](https://github.com/diffusion-steering/diffusion-steering.github.io) [page source read].
- **ConRFT, RSS 2025** ("Proceedings of Robotics: Science and Systems, RSS 2025"; arXiv 2502.05450).
  - Fine-tunes Octo "on the top of [HIL-SERL]" on a Franka, "with human interventions".
  - The README gives no tasks, times or success rates — [cccedric/conrft README](https://github.com/cccedric/conrft) [README read].
- **PA-RL** (Policy Agnostic RL; PREPRINT arXiv 2412.06685; venue not stated).
  - Offline RL then online fine-tuning of diffusion policies or OpenVLA, with a critic ensemble and a particle-choosing rule such as "max_q_value". In effect this is best-of-k selection by a learned Q, then training.
  - Real-robot work "is based on the Bridge Data Robot repo".
  - Caveat: the computer connected to the robot "may not be powerful enough to run OpenVLA at a reasonable speed".
  - No numbers in the README — [PolicyAgnosticRL README](https://github.com/MaxSobolMark/PolicyAgnosticRL) [README read].
- **RIPT-VLA** (Interactive Post-Training; PREPRINT arXiv 2505.17016).
  - **Advantages:** "leave-one-out advantage estimation" with PPO-style updates ("LOOP"). `rloo_batch_size` defaults to 8 rollouts per context, and dynamic sampling is on by default.
  - **Benchmarks:** LIBERO only; Meta-World is still a TODO; no real-robot experiments.
  - **Compute:** QueST runs recommend 3 GPUs (6 for LIBERO-90); OpenVLA-OFT runs recommend 4.
  - **Claims:** 94.3% on LIBERO-90; 97.5 across four LIBERO suites with OFT; from one demo, "4% success ... reaches 97+ after 15 iterations" with sparse binary rewards.
  - Source: [Ariostgx/ript-vla README](https://github.com/Ariostgx/ript-vla) [README read].
- **SimpleVLA-RL, ICLR 2026** (arXiv 2509.09674).
  - Binary outcome rewards; exploration via "dynamic sampling, adaptive clipping, temperature tuning".
  - **Sim results:** OpenVLA-OFT reaches 97.6 on LIBERO-Long; one trajectory per task goes "from 17.3 to 91.7".
  - **Real-world:** claims a "~relatively 300%" improvement, with no hardware or details given.
  - **Compute:** "1 node with 8 NVIDIA A800 GPUs, each having 80GB memory" (multi-node: 16 A800).
  - "Support real-world RL" is still on the roadmap.
  - Source: [PRIME-RL/SimpleVLA-RL README](https://github.com/PRIME-RL/SimpleVLA-RL) [README read].
- **VLA-RL** (a 2025 Notion blog; arXiv not opened).
  - "Improving OpenVLA-7B with outcome-based multi-task reinforcement learing"; LIBERO evaluation; "Built with Ray+vLLM+LoRA+FSDP".
  - No real-robot work; "Support SERL-style Real-world RL" is unchecked — [GuanxingLu/vlarl README](https://github.com/GuanxingLu/vlarl) [README read].
- **πRL / RLinf.** "π_RL: Online RL Fine-tuning for Flow-based Vision-Language-Action Models" (PREPRINT arXiv 2510.25889).
  - RLinf news: π0/π0.5 RL fine-tuning (2025/10); "real-world RL with Franka" (2025/12); "DSRL for Pi0" (2026/02); "RECAP" (2026/04); GR00T-N1.6/N1.7 (2026/06); π0-FAST RL (2026/09).
  - Supported hardware includes "AgileX Piper, and SO101" among others.
  - **No real-world results or compute numbers in the README** — [RLinf README](https://github.com/RLinf/RLinf) [README read].
  - **But RLinf's own SO101 page says "RLinf does not yet provide a supported real-world task or training workflow for SO101"**. It covers only hardware checks: joint motion, gripper, leader-arm teleoperation — [RLinf so101.rst](https://github.com/RLinf/RLinf/blob/main/docs/source-en/rst_source/examples/embodied/so101.rst) [doc read].
- **RECAP (the π*0.6 method) as reproduced in RLinf.**
  - Expansion: "RL with Experience and Corrections via Advantage-conditioned Policies".
  - "RECAP is an offline policy optimization method that requires no online environment interaction". Four stages: compute returns, train a value model, compute advantages, then classifier-free-guidance training of the policy.
  - Value model: SigLIP2-so400m + Gemma3-270M with a 201-bin categorical critic.
  - Advantages are labelled by quantile: the top 30% are positive by default.
  - Policy: π0.5 (OpenPI, PyTorch).
  - RLinf reproduction: LIBERO-10 Task 0 goes from 48.8% (few-shot SFT) to 66.5% after one RECAP iteration. Caveats: a single task and single iteration, results depend on `failure_reward` and the quantile, and "the value model can overfit".
  - The doc calls RECAP "especially suited for real-robot scenarios where large-scale online sampling is impractical", but reports no real-robot results.
  - The doc gives no paper citation or date; GPU requirements not specified.
  - Source: [RLinf recap.rst](https://github.com/RLinf/RLinf/blob/main/docs/source-en/rst_source/examples/embodied/recap.rst) [doc read].
- **Deep Model Predictive Optimization (DMPO; Sacks, Rana, Huang, Spitzer, Shi, Boots).** PREPRINT arXiv 2310.04590; venue not stated.
  - Included here because it is RL post-training of a *sampling-based controller*, with real-robot evidence. It "learns the inner-loop of an MPC optimization algorithm directly via experience" (a learned residual on MPPI).
  - **Real quadrotor:** it "can outperform the best MPC algorithm by up to 27% with fewer samples" and an end-to-end model-free RL policy "by 19%", "with 4.3X less memory" — [jisacks/dmpo README](https://github.com/jisacks/dmpo) [README read].

### Inferences
- **The realistic MSc path for real-robot RL is LeRobot HIL-SERL on an SO-101.**
  - Cost: one GPU plus tens of demonstration and classifier episodes, then hours of supervised interaction.
  - The original HIL-SERL ran on "a single NVIDIA RTX 4090 GPU" (24 GB). I found no documented figure for a 12 GB card in LeRobot's HIL-SERL. *Estimated* feasible on an RTX 3060, because the learner is SAC with a ResNet-10 encoder at 128×128 (or 64×64) images. This should be measured in week 1.
  - The ablation shows human corrections are not optional (0% without demonstrations or interventions). Budget human time for the whole training run, not just the demonstrations.
  - It uses SAC (critic-based, off-policy), not group-relative objectives. Group-relative methods (RLOO, MaxRL, TailRL) need several rollouts from the *same* context (RIPT-VLA defaults to 8). On a physical robot that means repeated identical resets, which is impractical without automated resets.
  - So group-relative or tail objectives are, in practice, a **simulation** tool in robotics.
  - *Estimated*: the human reset time for 8 rollouts × many contexts is the binding cost; not measured.
- **The regime where TailRL would beat RLOO is narrower than the student might hope.** RIPT-VLA reports RLOO-style advantages lifting a one-demo policy from 4% to 97+% in simulation. TailRL's reported win over GRPO/RLOO is from initial success rates of 0.83% down to 0.012% (Q4). So the case for a tail objective over RLOO needs genuinely rare success, around ≤1% (*inference*).
- **DMPO is the strongest sourced evidence that learned proposals can beat MPPI at a fixed sample budget, and it is on hardware.** This is the methods-level overlap any "learned proposals for sampling-based control" dissertation must beat or distinguish itself from.

### Gaps
- **Unverified headline numbers** (sources blocked and search budget exhausted): ConRFT's success and minutes of online fine-tuning, PA-RL's real-world gains, πRL's LIBERO/ManiSkill numbers, and SimpleVLA-RL's real-robot hardware.
- **HIL-SERL figures are now verified** from the authors' PDF (see Cited Findings).
- **Not read in the HIL-SERL paper:** pages 7–8 and 15–18 (some method details and analysis) and the supplementary material (per-task set-up, total human-intervention time).
- **π\*0.6 / RECAP (Physical Intelligence, Nov 2025).** pi.website was blocked and not searched, so PI's own claims are **unread**: real-robot tasks, throughput and failure-rate gains, hours of operation, robot fleet and compute.
  - The method's name and structure are sourced from RLinf's reproduction (Cited Findings above).
  - *Background knowledge, unverified:* PI reported espresso-making, laundry-folding and box-assembly tasks.
- **No community SO-101 RL results** (HIL-SERL or RLinf) were found; not searched.
- **No 2026 real-robot RL post-training papers** beyond the RLinf news items were searched (budget exhausted).

## Q3. Test-time sampling plus verification for robot policies, and how verifier error limits the gains

### Takeaway
- **The field is crowded.** Verifier-guided best-of-N for robot policies became very busy between 2024 and Oct 2026: V-GPS, RoboMonkey, FOREWARN, CoVer-VLA, VERITAS, DiVeR, plus world-model evaluators.
- **Verifier error already bites.** The clearest robot evidence is CoVer-VLA's report that RoboMonkey's verifier made π0 *worse* than no verification on most tasks. FOREWARN traces most of its failures to world-model prediction error. A CMU thesis (July 2026) argues that a planner maximising predicted return seeks out where its world model is "most optimistically wrong".
- **No systematic robot measurement found.** I found no robot study that measures the LLM-style "fLaws" curve: success versus k under a measured verifier false-positive rate. The search was incomplete.

### Cited Findings
- **V-GPS (Nakamoto, Mees, Kumar, Levine; CoRL 2024).**
  - Re-ranks K sampled actions from off-the-shelf generalist policies with a language-conditioned Cal-QL Q-function learned offline; no fine-tuning or weight access. Reported across Octo variants, RT-1-X and OpenVLA over 12 tasks — [search summary](https://www.alphaxiv.org/abs/2410.13816); code [nakamotoo/V-GPS](https://github.com/nakamotoo/V-GPS).
  - Limitations (secondary summaries): needs diverse stochastic proposals, adds inference compute, and is limited on unseen language or objects — [Moonlight review](https://www.themoonlight.io/en/review/steering-your-generalists-improving-robotic-foundation-models-via-value-guidance) [search summary].
- **RoboMonkey (Kwok et al.; CoRL 2025, PMLR v305).**
  - Samples a few VLA actions, fits a Gaussian to the translation and rotation components, sets the gripper by majority vote, and ranks candidates with a VLM-based verifier trained on synthetic action-preference data.
  - "the relationship between action error and the number of generated samples follows an exponentiated power law".
  - Claims: +25% absolute OOD (Bridge V2), +9% in-distribution (SIMPLER; the BibTeX abstract says 8%), and +7% on LIBERO-Long.
  - "RoboMonkey can sample and verify 16 candidate actions in 650 ms (or 1.5 Hz)".
  - Sources: [project page source](https://github.com/robomonkey-vla/robomonkey-vla.github.io) [page source read]; [PMLR](https://proceedings.mlr.press/v305/kwok25a.html) [listing only].
- **CoVer-VLA, "Scaling Verification Can Be More Effective than Scaling Policy Learning for VLA Alignment"** (Kwok, Zhang et al., Stanford/NVIDIA; PREPRINT arXiv 2602.12281, Feb 2026; listed as a Springer ECCV 2026 chapter — [Springer](https://link.springer.com/chapter/10.1007/978-3-032-37016-7_3) [listing only]).
  - README tables: π0 SIMPLER averages go from 41.5 to 65.5 (in-distribution) and from 29.7 to 62.0 (OOD) with CoVer. The headline "22% ... 13%" does not obviously match those tables.
  - The verifier is a "~312MB" checkpoint; its training pipeline is not yet released — [cover-vla README](https://github.com/cover-vla/cover-vla) [README read].
  - The paper reportedly states that RoboMonkey "fails catastrophically on most tasks—often performing even worse than the base π0 model". It attributes this to a verifier trained on OpenVLA-derived preferences, and to step-level verification clashing with action chunks — [search summary of arXiv 2602.12281](https://arxiv.org/abs/2602.12281).
- **DiVeR (Park et al., MSR Asia–Tokyo et al.).** PREPRINT arXiv 2610.04933, early Oct 2026; a NeurIPS 2026 workshop paper per the author page.
  - Verifier-guided best-of-N with a frozen VLA. Verifier training is weighted by "decision criticality", estimated from the spread of sampled actions. It learns from trajectory-level success or failure.
  - Evaluated on LIBERO, RoboCasa and a real Franka FR3, with "negligible verifier inference overhead" — [arXiv](https://arxiv.org/abs/2610.04933), [MSR article](https://www.microsoft.com/en-us/research/articles/diver-decision-critical-verifier-learning-for-vla-test-time-scaling/) [search summary].
- **VERITAS (Zhang & Shah, Princeton).** PREPRINT arXiv 2606.18247, June 2026. A generalist policy proposes candidates; a gradient-free visual verifier scores them at inference; verified successful rollouts then fine-tune the policy offline — [arXiv](https://arxiv.org/abs/2606.18247) [search summary].
  - Also listed: "Inference-time Policy Steering via Vision and Touch" (Yilin Wu, PREPRINT arXiv 2606.14981) — [arXiv](https://arxiv.org/abs/2606.14981) [listing only].
- **FOREWARN, "From Foresight to Forethought" (Wu, Tian, Swamy, Bajcsy).** RSS 2025 per a third-party slide deck (also an ICLR 2025 workshop paper); arXiv 2502.01828.
  - A latent world model predicts outcomes and a VLM (Llama-3.2-11B-Vision) aligned to the latents evaluates them.
  - Stated limitations:
    - It assumes the base policy already contains the correct behaviour; detecting when no sampled plan is suitable is left to future work.
    - Most failures trace to "the world model's imprecise predictions", made worse by limited training data.
    - Large autoregressive models make it slow.
  - Sources: [arXiv PDF via search summary](https://arxiv.org/pdf/2502.01828); code [CMU-IntentLab/Forewarn](https://github.com/CMU-IntentLab/Forewarn).
  - A 2026 paper reports that this VLM-in-the-loop approach takes ~3.7 s per decision — [Describe-Then-Act, arXiv 2603.23149](https://arxiv.org/pdf/2603.23149) [search summary].
- **Inference-Time Policy Steering (ITPS).** Venue not verified.
  - Steers pre-trained ACT or diffusion policies with sketch input, via post-hoc ranking, output perturbation, biased initialisation, guided diffusion and stochastic sampling. Benchmarked on Maze2D. "the ITPS framework assumes the pre-trained policy is given" — [yanweiw/itps README](https://github.com/yanweiw/itps) [README read].
- **World models as policy evaluators:**
  - **WorldGym:** Pearson 0.78 between world-model and real success across 17 tasks, with ~3.3% mean absolute difference — [WorldGym review](https://liner.com/review/worldgym-world-model-as-an-environment-for-policy-evaluation), [project](https://world-model-eval.github.io/abstract) [search summary].
  - **WorldEval** (PREPRINT arXiv 2505.19017): targets policy *ranking* (MMRV, Pearson) rather than exact rates — [arXiv](https://arxiv.org/pdf/2505.19017) [search summary].
  - **Ctrl-World** (PREPRINT arXiv 2510.10125): underestimates execution success and struggles with fine physics such as collisions and rotations — [arXiv](https://arxiv.org/abs/2510.10125) [search summary].
  - **dWorldEval** (PREPRINT, Apr 2026): reports weaker correlations for WorldGym, WorldEval and Ctrl-World on LIBERO — [arXiv](https://arxiv.org/html/2604.22152v1) [search summary].
- **Imperfect verifiers, LLM evidence — Stroebl, Kapoor & Narayanan, "Inference Scaling fLaws"** (PREPRINT arXiv 2411.17501, Nov 2024).
  - Unbounded gains from resampling require a perfect verifier. False positives (unit tests with limited coverage) set an accuracy ceiling that "resampling cannot decrease".
  - On HumanEval and MBPP, single-sample accuracy correlates with false-positive rate, so weaker models cannot resample their way to a strong model's single-sample accuracy.
  - Once false positives are costed, the optimal number of samples can be under 10, sometimes zero.
  - Sources: [arXiv](https://arxiv.org/abs/2411.17501v2), [review](https://www.themoonlight.io/en/review/inference-scaling-flaws-the-limits-of-llm-resampling-with-imperfect-verifiers) [search summary].
  - "Variation in Verification" (ICLR 2026) describes diminishing returns as verifier quality falls — [ICLR 2026 PDF](https://proceedings.iclr.cc/paper_files/paper/2026/file/ea21628f0fc0de542373be4c88343478-Paper-Conference.pdf) [search summary].
- **Training against a proxy verifier, LLM evidence from the MaxRL/TailRL group — "Can Large Reasoning Models Self-Train?" (SRT; Shafayat, Tajwar, Salakhutdinov, Schneider, Zanette; venue not shown on the pages read).**
  - SRT uses majority voting over the model's own samples as the reward.
  - "the self-generated proxy reward initially correlated with correctness can incentivize reward hacking, where confidently incorrect outputs are favored".
  - The authors frame the opportunity as a positive "generation-verification gap", where "generating correct solutions is challenging, but verifying their correctness is relatively easy". This is the LLM form of criterion (b) in Q6.
  - Source: [PDF hosted in Zanette-Labs/MaxRL repo](https://github.com/Zanette-Labs/MaxRL/blob/master/site/static/pdf/srt_paper.pdf) [pp. 1–2 read].
- **Robot RL already patches verifier errors by hand.** HIL-SERL's binary success classifiers needed "additional potential false positive or false negative examples if necessary" — [HIL-SERL paper](https://github.com/hil-serl/hil-serl.github.io/blob/main/static/hil-serl-paper.pdf) [read].
- **Planner exploitation of model or verifier error (control / model-based RL):**
  - **"The Tail and the Judge"** (S. Ramasubramanian, CMU MSR thesis, CMU-RI-TR-26-90, 28 July 2026; not peer-reviewed). A planner that maximises predicted return "will seek out exactly the regions where its world model is most optimistically wrong". The resulting loss is linked to the sharpness of the model's loss landscape, and sharpness-aware training of the world model alone yields gains — [thesis PDF](https://publications.ri.cmu.edu/storage/publications/2026/07/MSR_Thesis__1__20260731134034.pdf), [talk page](https://www.ri.cmu.edu/event/the-tail-and-the-judge-better-policy-gradients-and-world-models-for-reinforcement-learning/) [search summary only; both blocked].
  - **"A Control Theory of Predictability in Latent World Models"** (PREPRINT arXiv 2607.10362, July 2026; ID from search summary, not opened). Single-step validation error is "essentially uncorrelated with control success", whereas fidelity on the planner-reachable distribution tracks it.
  - **"Closing the Train-Test Gap in World Models for Gradient-Based Planning"** (PREPRINT arXiv 2512.09929): planned actions leave the training distribution, so predictions there are inaccurate — [arXiv](https://arxiv.org/abs/2512.09929) [search summary].
  - **"Mitigating Planner Overfitting in Model-Based RL"** (year not shown): "planner overfitting" exploits model errors — [listing](https://paperswithcode.com/paper/mitigating-planner-overfitting-in-model-based) [listing only].
- **Reward models now packaged in LeRobot.** These are candidate verifiers, but all judge a trajectory video *after* execution:
  - **ROBOMETER** (RSS 2026; arXiv 2603.02115): Qwen3-VL-4B backbone; "inference-only" in LeRobot; per-frame progress and success — [robometer.mdx](https://github.com/huggingface/lerobot/blob/main/docs/source/robometer.mdx).
  - **TOPReward** (PREPRINT arXiv 2602.19313): zero-shot token log-probabilities from Qwen3-VL-8B; "~16 GB" weights — [topreward.mdx](https://github.com/huggingface/lerobot/blob/main/docs/source/topreward.mdx).
  - **SARM** (PREPRINT arXiv 2509.25358): CLIP-based stage-and-progress reward used for reward-weighted BC — [sarm.mdx](https://github.com/huggingface/lerobot/blob/main/docs/source/sarm.mdx).
  - All [doc read].

### Inferences
- **A simple ceiling model** (*my derivation, labelled estimate*), in the spirit of Stroebl et al.:
  - Assume proposals succeed i.i.d. with probability p, the verifier has true-positive rate TPR and false-positive rate FPR, and selection picks any accepted candidate.
  - As k→∞, success tends to the verifier's precision: p·TPR / (p·TPR + (1−p)·FPR).
  - Example: p=0.3, TPR=0.9, FPR=0.1 gives ≈0.79, whatever the k.
  - With score-argmax selection, errors correlated with action features (e.g. aggressive pushes where friction is mis-modelled) can make success *fall* as k grows (winner's curse). This is the robotics form of "The Tail and the Judge" claim.
  - All four quantities (p, TPR, FPR, success versus k) are measurable on a real robot by executing *randomly chosen* candidates and recording their verifier scores. That makes a robot "fLaws" measurement feasible without exotic equipment.
- **Robot verifiers differ from LLM unit tests in three ways:**
  1. Pre-execution judges (Q-functions, world models, simulators) are evaluated off their training distribution exactly when selection pressure is high (Train-Test Gap paper; Tail-and-Judge).
  2. Post-hoc video reward models (ROBOMETER, TOPReward, SARM) cannot select before acting without a world model.
  3. The cost of a false positive is physical (a failed or unsafe action), not just wasted compute.
- **Latency is a first-order constraint.** RoboMonkey reports 16 candidates in 650 ms (1.5 Hz) on server hardware, and FOREWARN-style VLM judging about 3.7 s per decision. Both are far from a 10–50 Hz control loop. Lightweight verifiers (Q-functions, physics simulation, small learned scorers) are the only plausible on-budget options for the student's hardware (*inference*).

### Gaps
- **Not found:** any robot study plotting real success versus k against a *measured* verifier false-positive rate (a robot "fLaws" curve). The CoVer finding is an observation, not a calibrated measurement. Search incomplete.
- **Not read in full:** FOREWARN's quantitative results, V-GPS's per-task numbers, WorldGym, Ctrl-World and WorldEval methods, and the Tail-and-Judge thesis (all blocked).
- **ITPS venue and the original ITPS paper's real-robot results** were not verified.

## Q4. What are MaxRL and TailRL, and have pass@k-aware or tail objectives been used for robot policies?

### Takeaway
- **Both papers exist and are recent, from Andrea Zanette's group at CMU.** I read both main texts from author-hosted PDFs. Neither contains any robotics or continuous-control experiment. Both assume perfectly verified (MaxRL) or deterministic (TailRL) rewards, and their gains appear only when success is rare (≈1% down to 0.01% initial success in the maze).
  - **MaxRL** ("Maximum Likelihood Reinforcement Learning", Tajwar et al., ICML 2026 Oral) normalises the group advantage by the mean reward. It thereby optimises a truncated maximum-likelihood objective that is a weighted sum of pass@k terms.
  - **TailRL** ("Tail-Likelihood Reinforcement Learning", Ramasubramanian et al., PREPRINT, Sept 2026) extends this to continuous rewards via an order-statistic advantage whose gradient is a harmonic (1/k) mixture of best-of-k gradients.
- **Closest robotics precedents:**
  - risk-seeking policy gradient used to search for symbolic control policies (Landajuela et al., ICML 2021);
  - learned optimisers or proposals for MPPI (DMPO, real quadrotor);
  - best-of-k-by-Q training (PA-RL).
- **No work found** trains a robot action-proposal distribution with an explicit pass@k or best-of-k objective against a pre-execution verifier and tests it on hardware. This is "no overlap found" with an incomplete search, **not** established novelty.

### Cited Findings
- **MaxRL — what it is.** Tajwar, Zeng, Zhou, Song, Arora, Jiang, Schneider, Salakhutdinov, Feng, Zanette.
  - **Venue:** "ICML 2026 Oral Presentation" and "Best Paper Award at ICLR 2026 SPOT Workshop" per the README; arXiv 2602.02710, Feb 2026 — [tajwarfahim/maxrl README](https://github.com/tajwarfahim/maxrl) [README read]; [ICML 2026 poster page](https://icml.cc/virtual/2026/poster/65332) [listing only].
  - One review labels it ICLR 2026, which conflicts with the README; it was the ICLR *workshop* — [Liner](https://liner.com/review/maximum-likelihood-reinforcement-learning).
- **MaxRL — objective and estimator** (project-page source, read):
  - "Maximum likelihood optimizes an infinite weighted sum of pass@k gradients". The truncated objective at level T is a "compute-indexed family": "T = 1 recovers standard reinforcement learning (pass@1)", and "T → ∞ recovers exact maximum likelihood".
  - The estimator is "an unbiased estimator for the MaxRL gradient of order T = N" (N = rollouts). It normalises "by successful samples K".
  - The advantage "is normalized by the per-task mean reward", rather than left unnormalised (REINFORCE) or divided by the standard deviation (GRPO).
  - Rewards are binary: "$r_i \in \{0,1\}$".
  - Experiments: ImageNet (toy), maze navigation, GSM8K and Qwen3-1.7B/4B math. "requires 2.3× – 19.2× fewer samples than GRPO" to match pass@k; "7.9×–19.2×" test-time-scaling efficiency on Qwen3-4B.
  - No robot or continuous-control experiments; no discussion of noisy verifiers.
  - Source: [Zanette-Labs/MaxRL site source](https://github.com/Zanette-Labs/MaxRL) [page source read].
  - The authors summarise it as "a single line of code (dividing by the mean reward in the advantage computation)", with gains "with perfect verifiers" — [Salakhutdinov on X](https://x.com/rsalakhu/status/2019507844161187916) [search snippet].
  - The code is built on verl (PyTorch). The Qwen3 runs used "4 nodes of 8xH200 GPUs". The repo includes a 17×17 maze — [maxrl README](https://github.com/tajwarfahim/maxrl).
- **MaxRL — verified against the paper.** Source: [MaxRL preprint PDF in the project repo](https://github.com/Zanette-Labs/MaxRL/blob/master/site/static/pdf/MaxRL_Preprint.pdf) [pp. 1–8, 11–13 read].
  - **Abstract framing.** RL is "the method of choice to train models in sampling-based setups with binary outcome feedback, such as navigation, code generation, and mathematical problem solving". The navigation citations are Thrun et al. 2005 and Anderson et al. 2018, but the experiments use text mazes, not robots.
  - **Core identity.** ∇J_ML = Σ_k (1/k) ∇pass@k, so "Reinforcement learning optimizes a first-order approximation of the maximum likelihood objective".
  - **Theorem 1.** The maximum-likelihood gradient is "the average gradient from successful trajectories only".
  - **Estimator.** ĝ_N = (1/K) Σ r_i S_i is "an unbiased estimator for the MaxRL gradient of order T = N".
  - **On-policy Algorithm 1, line 8:** ĝ(x) = (1/(N·r̂(x))) Σ_j (r_j − r̂(x)) S_j if r̂(x) > 0, else 0. In words, "the advantage is normalized by the per-task mean reward, rather than left unnormalized as in RLOO ... or normalized by the reward standard deviation as in GRPO".
  - **Population weights w(p) on pass rate p:** RL = 1; GRPO = 1/√(p(1−p)); MaxRL(T) = (1−(1−p)^T)/p; ML = 1/p. GRPO's weight "inverts for sufficiently large pass rates", which up-weights very easy inputs.
  - **Maze set-up.** "1 million unique 17 × 17 mazes" for training, 256 held out, and a lightweight transformer of "approximately 3M parameters" after brief supervised pretraining.
  - **Stated limitations:** "Our work currently assumes a binary reward setting and does not extend directly to continuous or arbitrarily valued rewards". Future work: "continuous rewards, multi-turn reinforcement learning, and off-policy settings such as PPO-style training". Its headline test-time gains are stated "with a perfect verifier".
- **TailRL — authors and status.** Ramasubramanian, Arora, Tajwar, Zeng (CMU); Wu, Zhou, Xu (Together AI); Feng (UC Berkeley/Impossible); Song, Singh, Salakhutdinov, Bagnell (CMU/Aurora); Schneider, Zanette (CMU). "Preprint, 2026" — [TailRL-website source](https://github.com/Zanette-Labs/TailRL-website) [page source read]. PREPRINT arXiv 2609.02987 (Sept 2026; code repo updated 11 Sep 2026 — [Zanette-Labs org](https://github.com/Zanette-Labs)).
- **TailRL — objective and estimator** (project page "blog.html", read from the GitHub source):
  - "TailRL is the expected log tail-probability over uniformly sampled thresholds", with rewards rescaled to [0,1].
  - $J_{\mathrm{TailRL}} = \sum_k \frac{\text{Best-of-}k-1}{k}$, and "The harmonic weights 1/k are derived from the Maclaurin series of the logarithm. They are not selected or tuned". On binary rewards it reduces to MaxRL.
  - **Advantage:** $\omega(r_{(i)}) = \sum_{k=1}^{i} \frac{r_{(k)}-r_{(k-1)}}{N-k+1}$, $A_i = \omega(r_i)-\bar\omega$. The README adds scaling by N.
  - "a gap only the best rollout clears is worth N times a gap everybody clears"; "no critic, and no new hyperparameter".
  - **Experiments:**
    - ImageNet box localisation: "matches or beats supervised objectives", although PKPO "sits above TailRL at N=16 and N=64" on Best-of-1024 IoU.
    - Text-maze navigation: "From a poor initialization, GRPO and RLOO fail to improve reliably"; PKPO "trails it everywhere".
    - GUI grounding: matches RLOO's best result with "128× fewer" (3B) and "256× fewer" (7B) inference samples.
    - Code runtime optimisation: GRPO/RLOO settle at the "copy the input" reward of 1; gem5 timing means "noise cannot fake a gain".
  - **Robot / control:** "None appear"; there is no limitations section.
  - Source: [TailRL-website blog source](https://github.com/Zanette-Labs/TailRL-website) [page source read].
- **TailRL — verified against the paper.** Source: [TailRL PDF in the project repo](https://github.com/Zanette-Labs/TailRL-website/blob/main/static/TailRL.pdf) [pp. 1–15, all main text, read].
  - **Reward assumption:** "A deterministic reward function provides a scalar feedback for an input rollout pair r(x, z) ∈ [0, 1]", so "all randomness in the reward comes from the policy".
  - **Finite estimator.** Rollout weights follow ω(r_(i)) = ω(r_(i−1)) + (r_(i) − r_(i−1))/(N − i + 1), with centred advantages A_i = ω(r_i) − ω̄. "the centered estimator is unbiased for ∇J^(N−1) rather than ∇J^(N)". In their words, "the rollout budget determines which member of the TailRL family is optimized".
  - **Weighting.** Gradient weight φ′(p) on tail-probability p: RL = 1; TailRL(T) = (1−(1−p)^T)/p; TailRL = 1/p; PKPO = k_opt(1−p)^(k_opt−1).
  - **ImageNet localisation.** N ∈ {16, 64, 256, 1024} training rollouts per input. "even at N = 16, TailRL exceeds both baselines trained at N = 1024". The cosine between the finite-N and population gradients "rises steadily toward exact agreement" as N grows (Fig. 6, N from 2^2 to 2^18).
  - **Maze.** "All methods use the same initial policy and N = 16". Initial shortest-path success ranges "from approximately 1% down to 0.01%". "RLOO, GRPO and TailRL behave similarly when the initial policy produces shortest-path successes frequently, but diverge as the initial success rate falls ... The advantage of TailRL is therefore concentrated where high-reward rollouts are attainable but rare."
  - **GUI grounding.** TailRL matches RLOO's Pass@1024 "with 8 rather than 1024 inference rollouts" (3B) and 4 rollouts (7B). Pass@1 is similar for TailRL and RLOO.
  - **Code optimisation (N = 16; Qwen3-1.7B).** GRPO and RLOO converge on copying the input. TailRL "maintains substantially higher entropy and continues producing risky rewrites. This lowers single-rollout correctness". It reaches a mean Best-of-1024 speedup of 7.7× against 0.98× (GRPO) and 0.96× (RLOO).
  - **Conclusion:** "its gains were largest when high-reward rollouts were rare or when training was drawn toward a common but suboptimal behavior".
  - **No limitations section** in the main text. Related work distinguishes TailRL from risk-sensitive/CVaR RL, which optimises "a tail statistic at a chosen risk level".
  - **Compute:** NCSA DeltaAI and ACCESS allocations (acknowledgements); GPU counts per experiment are in the README only (Q4 bullets above).
- **TailRL — code.** "The RL framework is a fork of verl" (PyTorch); no JAX.
  - The maze is a "17×17" text gridworld with a "3.9M-parameter decoder", swept over seven initial policies whose shortest-path success runs "from 0.83% down to 0.012%". 1 GPU for maze and ImageNet; multiple GPUs for GUI grounding.
  - Source: [Zanette-Labs/TailRL README](https://github.com/Zanette-Labs/TailRL) [README read].
- **The companion thesis "The Tail and the Judge"** (CMU-RI-TR-26-90, 28 July 2026).
  - The tail part says TailRL "aligns training with best-of-many deployment at every sampling budget simultaneously".
  - The judge part concerns world-model error that planners exploit (Q3) — [search summary](https://publications.ri.cmu.edu/storage/publications/2026/07/MSR_Thesis__1__20260731134034.pdf).
- **Related follow-ups seen in results:** "ArgMaxRL: Generalizing MaxRL to Continuous Rewards" — [doubleAI](https://www.doubleai.com/research/argmaxrl-generalizing-maxrl-to-continuous-rewards) [listing only]; "RL2ML: Finite-Rollout Surrogate Objectives from RL to ML" (PREPRINT arXiv 2605.30154) — [arXiv](https://arxiv.org/pdf/2605.30154) [listing only].
- **Pass@k Training** (Chen, Qin, Wu, Ling, Ye, Zhao, Shi; PREPRINT arXiv 2508.10751, 2025).
  - Uses "Pass@k as the reward", with an analytically derived advantage (bootstrap and analytical variants). It claims exploration and exploitation "can mutually enhance each other", and Pass@k improves "while maintaining Pass@1".
  - The only non-language environment is a Maze with a verifier; no robotics — [RUCAIBox/Passk_Training README](https://github.com/RUCAIBox/Passk_Training) [README read].
  - "Pass@K Policy Optimization" (PKPO, Walder & Karkhanis, 2025) was not opened; it appears only as a TailRL baseline (above).
- **Risk-seeking policy gradient** (Petersen et al., "Deep symbolic regression", ICLR 2021).
  - The gradient uses only samples at or above the batch's empirical (1−ε) quantile ("training.epsilon: 0.05").
  - It **has been applied to control**: Landajuela et al., "Discovering symbolic policies with deep reinforcement learning", ICML 2021, configures DSO "to learn a symbolic policy for MountainCarContinuous-v0" — [dso-org README](https://github.com/dso-org/deep-symbolic-optimization) [README read].
- **Learned proposals for sampling-based MPC.** DMPO learns a residual on MPPI's update via RL and beats the best MPC baseline by up to 27% with fewer samples on a real quadrotor — [jisacks/dmpo](https://github.com/jisacks/dmpo) [README read].
- **Training on best-of-k-selected outcomes:**
  - PA-RL selects particles by "max_q_value" — [PolicyAgnosticRL](https://github.com/MaxSobolMark/PolicyAgnosticRL).
  - VERITAS fine-tunes on verifier-approved rollouts — [arXiv 2606.18247](https://arxiv.org/abs/2606.18247).
  - Neither is described as using a pass@k or tail *objective*.

### Inferences
- **Where MaxRL/TailRL genuinely belong in robotics.** They need, all at once:
  - (i) many i.i.d. rollouts from the same context (simulation, or a robot with automatic resets);
  - (ii) a bounded, trustworthy outcome score;
  - (iii) rare success under the initial policy (≤~1%, per the maze sweep);
  - (iv) deployment that genuinely selects among k samples with a verifier that is *the same* or as good as the training reward.
- **Their natural robotics home** is training a *proposal generator* for a simulator-verified planner, as in DSO's symbolic-policy search or MPPI proposals. It is **not** on-robot RL, where (i) fails and RLOO already works from 4% success (RIPT-VLA).
- **Three verified assumptions conflict with physical robots** (*inference from the papers' own text*):
  1. **Deterministic rewards.** TailRL assumes the reward is deterministic, so "all randomness in the reward comes from the policy". MaxRL assumes binary correctness from a perfect verifier. Physical outcomes are stochastic: the same action can succeed or fail. Up-weighting rare high rewards (1/p) then partly rewards luck. With a group of N = 16, the single lucky success in a group gets the largest advantage (MaxRL: N/K − 1 = 15 for K = 1).
  2. **Rollout groups from one context.** The papers use N = 16 (maze, code) and up to 1024 (ImageNet) per input. On hardware, that is 16 identical resets per context.
  3. **Worse single-shot reliability.** TailRL can lower single-sample correctness (code optimisation: "This lowers single-rollout correctness"). A robot that executes once, with no verifier, may become *less* reliable under a tail objective. Any robotics use must report k = 1 performance alongside best-of-k.
- **MaxRL's own future-work list** ("multi-turn" and "off-policy" settings) names exactly what robot RL usually needs, such as off-policy SAC in HIL-SERL. So the published estimators apply only to on-policy, trajectory-level outcome rewards, as in RIPT-VLA- or SimpleVLA-RL-style sim training.
- **Upper-tail selection is already used in large-scale robot RL, though not as a pass@k objective.** RECAP labels the top 30% of advantages "positive" and trains an advantage-conditioned policy — [RLinf recap.rst](https://github.com/RLinf/RLinf/blob/main/docs/source-en/rst_source/examples/embodied/recap.rst). Risk-seeking PG trains only on samples above the ε-quantile (DSO).
  - *Inference:* quantile-based selection of the upper tail already appears in robot RL. What is absent is a training objective explicitly tied to best-of-k *deployment* with a pre-execution verifier.
- **This explains the student's earlier negative result** (TailRL ≈ RLOO; a ten-line controller wins). Shaped checkpoint navigation lacks rare-success structure, and the deployment there was not best-of-k.
- **Tail objectives magnify verifier error.** TailRL's own description says the top rollout's gap is worth up to N× a common gap. With an imperfect judge, the top-ranked rollouts are exactly where false positives concentrate (winner's curse).
  - So tail objectives should *amplify* verifier error relative to RLOO. This is a testable hypothesis (*my inference*), consistent with the Tail-and-Judge claim that planners exploit optimistic model error.
  - The interesting robotics question is therefore not "does TailRL beat RLOO?" but "does training proposals for best-of-k make the system *more* exposed to sim-to-real verifier error?"
- **MCL/WTA connection** (*conceptual, unverified this session*): the winner-takes-all "oracle" loss is supervised best-of-k with a perfect verifier (the ground-truth label). minADE_k in trajectory prediction is the evaluation counterpart. MaxRL/TailRL are the RL-with-verifier analogues.

### Gaps
- **Not verified this session** (search budget exhausted): multiple-choice learning (Guzman-Rivera et al., NeurIPS 2012), stochastic MCL (Lee et al., NeurIPS 2016), WTA/minADE_k in trajectory prediction, "Learning Sampling Distributions for Model Predictive Control" (Sacks & Boots; CoRL 2022 per background knowledge, unverified), and inference-aware fine-tuning for best-of-N (Chow et al.).
- **Not read:** the PKPO paper, and the Tail-and-Judge thesis experiments (unknown whether the judge part uses robot or continuous-control tasks).
- **MaxRL and TailRL main texts were read** from author-hosted PDFs. Their appendices were not read: TailRL Appendix B "evaluated domains" and maze details; MaxRL Appendix E.1 maze details.
- **Possible version differences.** The MaxRL PDF read is the preprint, not the ICML camera-ready.
- **Not run:** a targeted search for "pass@k" or "best-of-N-aware" training of robot action proposals. So "no overlap found" is weak evidence.

## Q5. Bristol lineage: Winfield's "consequence engine" (simulate before acting) and its relation to MPPI and best-of-k

### Takeaway
- **Conceptually this is the lineage the student's question belongs to.** A consequence engine simulates each of a small, enumerated set of candidate actions in an internal robot simulator, scores the predicted outcomes with explicit (safety or ethical) rules, and selects.
  - That is best-of-k with a simulator verifier and hand-enumerated (not learned) proposals.
  - MPPI is the stochastic, softmax-weighted, warm-started version.
  - TailRL would be a way to *train* the proposals.
- **Could not verify.** None of the Winfield papers or any 2024–2026 follow-up could be checked this session (frontiersin.org blocked; search budget exhausted).

### Cited Findings
- None verifiable this session: the Frontiers article fetch failed (host blocked), and no searches remained.

### Inferences
- *Mapping (my analysis):*
  - Consequence engine = deterministic enumeration of k candidates + simulator rollout + rule-based outcome evaluation + argmax.
  - MPPI = k stochastic perturbations of a nominal control sequence + model rollouts + exponentiated-cost weighting (soft best-of-k) + warm-start across time steps.
  - Best-of-k with a learned proposal = a policy replaces the enumeration.
  - All three inherit the same failure: the simulator's error is the verifier's error. The selected action is biased towards situations the internal model misjudges (Q3).
  - A modern, measurable version of the consequence-engine question is therefore: *how does internal-model error cap the benefit of simulating more candidates on a real robot?* That is the surviving candidate in Q7.

### Gaps
- **Unverified (background knowledge only, re-check before citing):**
  - Winfield, Blum & Liu (2014), "Towards an Ethical Robot: Internal Models, Consequences and Ethical Action Selection", TAROS 2014 (LNCS 8717). E-puck experiments: a robot diverts a "human" proxy robot from a hazard by simulating its next actions; in a two-proxy dilemma it reportedly dithered in many trials.
  - Blum, Winfield & Hafner (2018), "Simulation-Based Internal Models for Safer Robots", Frontiers in Robotics and AI 4:74.
  - Vanderelst & Winfield (2018), "An architecture for ethical robots inspired by the simulation theory of cognition", Cognitive Systems Research 48.
  - Not verified: platforms, candidate counts, timing, and whether any 2024–2026 follow-up exists.
- **BRL people:** Alan Winfield (UWE/BRL) is associated with this work per background knowledge; his current BRL status is not verified (see the BRL staff notes). Co-authors Christian Blum and Dieter Vanderelst are not believed to be at BRL now (unverified).

## Q6. The crucial sceptical test — is there a robot problem where (a) good actions are multimodal, (b) a pre-execution verifier or simulator judges better than the policy, and (c) the classical sample-then-verify baseline demonstrably fails at a fixed sample budget?

### Takeaway
- **3pi+:** no. Without exteroception the action choices are rarely multimodal. The action space is 2-D and model rollouts are cheap, so MPPI or a hand-coded controller should match any learned proposal. The student's own result already showed a ten-line controller beating learners.
- **Low-cost arm (SO-101):** (a) and (b) are plausible for contact-rich planar pushing around an obstacle, with a MuJoCo model as verifier. But **(c) has no evidence on this platform**; the only sourced fixed-budget win for learned proposals over MPPI is DMPO on an agile quadrotor.
- **Simulator the student builds:** (a)–(c) can be manufactured at will, which fails filter 3 unless the simulator is the verifier for a real robot.

### Cited Findings
- An SO-101 model exists in MuJoCo Menagerie ("The Robot Studio SO101", `robotstudio_so101`; also `trs_so_arm100` and "Low-Cost Robot Arm"). Menagerie models are not system-identified: a grade "will be applied to each model once a proper system identification toolbox is created", and "the current state of many models is not necessarily as good as it could be". There is no Pololu or small differential-drive model — [MuJoCo Menagerie README](https://github.com/google-deepmind/mujoco_menagerie) [README read].
- DMPO beats "the best MPC algorithm by up to 27% with fewer samples" on a real quadrotor — [jisacks/dmpo](https://github.com/jisacks/dmpo).
- LeRobot documents the cheap proposal policies: ACT at "~2–6GB" training (RTX 3060 listed) and diffusion at "~8–14GB" (batch 8) — [hardware_guide.mdx](https://github.com/huggingface/lerobot/blob/main/docs/source/hardware_guide.mdx).
- TailRL and MaxRL show gains only where initial success is rare (maze initial success 0.83%–0.012%) — [TailRL README](https://github.com/Zanette-Labs/TailRL).

### Inferences
- **(a) Multimodality.**
  - 3pi+: weak. Only bump and line sensing, so detours are only meaningful with a known map; diff-drive speed profiles are unimodal.
  - SO-101: real for pushing an object to a goal around an obstacle (left or right routes, several contact faces). This is the classic setting in which BC policies need multimodal heads (*inference*; I did not re-source the Push-T literature this session).
- **(b) A pre-execution verifier.**
  - 3pi+: an identified dynamics simulator can judge trajectories, but with little to choose between.
  - SO-101: the MuJoCo Menagerie model plus an object initialised from webcam/ArUco pose estimation. Its error (servo tracking, friction, unidentified model) is genuine, not injected. Whether it judges *better than the policy* is an empirical question; the week-1 test measures it.
- **(c) Classical failure at a fixed budget.**
  - Unknown for SO-101 pushing. MPPI with warm-starting is a strong baseline when rollouts are cheap.
  - The defensible framing is therefore **not** "learned proposals beat MPPI". It is: "how does simulator (verifier) error cap the real-world benefit of simulating more candidates — for MPPI, BC proposals and (optionally) tail-trained proposals alike?" That measurement is useful whichever method wins.
- **Plainly:** I could not identify a robot problem on the student's hardware where all three conditions are established. The SO-101 pushing setting is the only one where (a) and (b) are plausible, and (c) is replaced by a measurement question.

### Gaps
- **Not measured:**
  - MuJoCo rollout speed for the SO-101 plus object on the student's CPU/GPU;
  - webcam/ArUco tracking accuracy and rate;
  - SO-101 servo tracking error;
  - SO-101 price (see the low-cost-arm notes).
- **Not searched:** planar-pushing MPC and sampling-based pushing literature (e.g., pusher-slider control), which are the classical baselines for the arm variant.

## Q7. Which candidate questions survive all eight filters, and what is each one's first-week kill test?

### Takeaway
- **One candidate survives, conditionally:** "the robot analogue of Inference Scaling fLaws". It measures how sim-to-real verifier error caps the real-world benefit of simulator-verified best-of-k selection (including MPPI) on a low-cost arm pushing task. A TailRL- or RLOO-trained proposal is an *optional* arm testing whether best-of-k-aware training amplifies verifier exploitation.
- **Everything else fails at least one named filter.** That includes RL post-training of VLAs, TailRL versus RLOO on robots, VLM-verifier best-of-N, world-model verifiers, the 3pi+ variants and retry-aware pass@k.

### Cited Findings
- **Evidence the surviving candidate rests on:**
  - verifier-guided best-of-N works but is fragile for VLAs (RoboMonkey; CoVer-VLA's "worse than the base π0" report; DiVeR; VERITAS);
  - imperfect verifiers cap resampling for LLMs (Stroebl et al.);
  - planners exploit optimistic model error (Tail-and-Judge thesis; Control-Theory-of-Predictability; Train-Test-Gap);
  - learned proposals can beat MPPI on hardware (DMPO);
  - an SO-101 MuJoCo model exists but is unidentified (Menagerie).
  - All are cited in Q3, Q4 and Q6 above.

### Inferences

#### Candidate A (conditional survivor) — "Simulate more, gain less? Measuring verifier-limited best-of-k on a low-cost arm"
- **(a) Research question.** On an SO-101 pushing an object to a goal around an obstacle, how does real-world success scale with the number k of simulator-verified candidates (BC proposals, MPPI samples), and can the plateau or reversal be predicted from a cheap calibration of the simulator's false-positive rate?
- **(b) Why now:**
  - RoboMonkey (arXiv June 2025; CoRL 2025) showed test-time scaling for VLAs.
  - CoVer-VLA (arXiv Feb 2026; ECCV 2026 per listing) reported RoboMonkey's verifier can make π0 worse than no verification.
  - VERITAS (June 2026) and DiVeR (Oct 2026) show the area is live.
  - "The Tail and the Judge" (28 July 2026) and TailRL (Sept 2026) explicitly tie best-of-k training to judges that planners exploit.
  - MuJoCo Menagerie ships an SO-101 model (date not found) that is not system-identified.
- **(c) Nearest work and what remains open.**
  - The strongest overlap is the Tail-and-Judge thesis, judge part: planners exploiting world-model error; sharpness-aware world models. Its experimental domain is unknown, as I could not read it.
  - Then: RoboMonkey (scaling of action error with samples, measured against reference actions rather than real outcomes, as far as I read), CoVer-VLA, V-GPS, FOREWARN (failures traced to world-model error), DMPO (learned proposals beat MPPI) and Stroebl et al. (the LLM ceiling).
  - Open, as far as found: a calibrated, physical-robot measurement of success versus k against verifier FPR, with a physics-simulator verifier. "No overlap found" ≠ novelty (search incomplete).
- **(d) Strongest simple baselines:**
  - (i) k = 1: execute the BC policy's or MPPI's nominal plan;
  - (ii) MPPI with the same simulator and the same total rollout budget;
  - (iii) a hand-coded push primitive with re-planning.
  - Why they might fail here: contact and friction mismatch make *any* simulator-based selection biased, and argmax selection grows the bias with k. Whether (iii) already solves the task is unknown; the kill test checks this.
- **(e) First-week kill test**, in simulation, before buying anything:
  - Build two MuJoCo pushing scenes from the Menagerie SO-101: a "verifier" scene with nominal friction and mass, and a "truth" scene with plausibly different friction, mass and servo gains.
  - Generate k ∈ {1, 2, 4, …, 64} candidates from Gaussian/MPPI and from a scripted multimodal proposal (left/right push families). Record truth success of the verifier-selected candidate, plus the verifier's TPR/FPR on proposals.
  - **Kill if any of:**
    1. Verifier FPR on proposals is near zero across plausible mismatches (no ceiling to study);
    2. k = 1 or the hand-coded primitive already reaches ≥90% truth success (no headroom; *threshold estimated*);
    3. Truth success versus k under mismatch is indistinguishable from the perfect-verifier curve.
  - **Week-2 physical kill:** if webcam/ArUco cannot track the object to within the task tolerance at a usable rate (measure it), stop. Falling back to simulation only would fail filter 1.
- **(f) Hardware and compute:**
  - SO-101 arm: needed (price not sourced here).
  - USB webcam: owned.
  - ArUco markers: printed (*estimated* negligible cost).
  - RTX 3060: enough for ACT (sourced ~2–6 GB) and diffusion at reduced batch (*estimated*; sourced ~8–14 GB at batch 8).
  - MuJoCo rollouts on CPU: speed not found, measure in week 1.
  - No VLA and no GPU cluster needed.
  - Human time for physical trials with manual resets: *estimated* tens of seconds per trial, so ~100 trials per few hours (unmeasured).
  - BRL asks:
    - one or two sessions in a motion-capture volume (or a calibrated overhead camera rig) to validate webcam tracking;
    - access to any BRL arm if an SO-101 is not bought.
- **(g) Outputs:**
  - **Headline figure:** real success versus log k for k = 1, BC + simulator verifier and MPPI, overlaid with the ceiling predicted from measured p, TPR and FPR. Also the "predicted minus realised outcome of the selected candidate" gap as k grows (winner's curse).
  - **Best realistic:** a measured k* beyond which real success plateaus or declines, predicted within a confidence interval by the calibration model. Optionally, evidence that tail-trained proposals widen the gap.
  - **Floor:** simulator verification gives no real gain over k = 1 on this arm, with the measured FPR explaining why. That is a measured boundary, not "it didn't work".
- **(h) Filters:**
  1. **Pass.** Contact, friction and servo physics are the subject.
  2. **Conditional.** Baselines named; whether the primitive or MPPI already saturate is tested in week 1.
  3. **Pass, if** the mismatch comes from the real robot and the unidentified model rather than injected perturbations. The week-1 simulation pair is only a proxy.
  4. **Pass, if** multimodality is confirmed in demonstrations.
  5. **Partial.** Nearest work named; not read in full.
  6. **Pass** (estimated).
  7. **Pass.**
  8. **Pass.**
- **(i) Skills:** MuJoCo modelling and system identification, sampling-based MPC, imitation learning with LeRobot, camera calibration and fiducial tracking, experimental statistics; optionally the student's JAX group-relative RL for the TailRL arm.
- **(j) Bristol/BRL:** the Winfield consequence-engine lineage (simulate before acting) is the natural local framing (unverified, Q5). No other BRL people came up in this scope.
- **Optional TailRL arm.** Train the proposal policy in the verifier scene with RLOO versus TailRL (the student's JAX code). Test on hardware whether TailRL proposals show a larger predicted-minus-real gap.
  - Always report k = 1 success as well. The TailRL paper itself reports lower single-rollout correctness in its code task.
  - Use the papers' group size (N = 16) so the comparison matches the published set-up.
  - Kill: if verifier errors are uncorrelated with the proposals' tail (rank correlation of verifier score error with verifier score ≈ 0 in simulation), the arm has nothing to show.
  - Methods caveat: TailRL assumes deterministic rewards. Train it against the *deterministic* simulator verifier, and treat physical stochasticity as part of what is being measured.

#### Tempting but fails (filter named)
1. **RL post-training (RIPT/SimpleVLA-RL/πRL-style) of SmolVLA or another VLA on an SO-101.** Fails 6: SmolVLA training is ~10–16 GB at batch 8; VLA RL recipes use 3–6 GPUs (RIPT) or 8×A800 80 GB (SimpleVLA-RL). Fails 5: crowded in simulation (RIPT-VLA, SimpleVLA-RL, VLA-RL, πRL).
   - RLinf lists SO101 hardware but states it "does not yet provide a supported real-world task or training workflow for SO101". So the *real SO-101* niche is open, but only for someone with multi-GPU access.
2. **TailRL/MaxRL versus RLOO for robot policy RL** (simulation or real). Fails 4: method-first. Fails 2: no rare-success structure, and hand-coded controllers win (the student's own result). Fails 6 on real robots: same-context groups need identical resets.
   - RIPT-VLA shows RLOO already climbs from 4% to 97+% in simulation.
   - The TailRL paper says its methods "behave similarly when the initial policy produces shortest-path successes frequently" and diverge only from ~1% down to 0.01% initial success.
   - The papers assume deterministic (TailRL) or perfectly verified binary (MaxRL) rewards, which physical outcomes are not.
3. **VLM-verifier best-of-N for a VLA on an SO-101 using LeRobot reward models** (ROBOMETER 4B, TOPReward 8B with ~16 GB weights). Fails 6 (verifier size and latency: RoboMonkey 1.5 Hz on server hardware). Fails the (b) criterion: these models judge recorded video after execution and cannot rank actions before acting without a world model.
4. **Video world model as verifier or evaluator for a low-cost arm** (Ctrl-World/WorldGym-style). Fails 6 (heavy generative models; Jetson-class support is Thor-only in NVIDIA's docs) and 5 (crowded: WorldGym, WorldEval, Ctrl-World, dWorldEval, BWM, GeniWorld in 2025–2026).
5. **3pi+ simulate-before-acting, or MPPI with learned or tail-trained proposals.** Fails 2 and 4: 2-D action, weak multimodality without exteroception, cheap rollouts, and hand-coded controllers already win. It reduces to model-mismatch MPC. It can instead inform the student's open "adaptive motion control under wheel/surface/payload changes" study as a measurement chapter.
6. **Retry-aware (pass@k) policies with physical success sensing** (3pi+ docking via bump sensors; arm insertion). Fails 2: scripted retries and spiral search are strong classical baselines (*background knowledge*). Fails 3: physical attempts are not i.i.d., since a failure changes the state.
7. **Jetson Orin Nano 8 GB VLA deployment benchmark.** Fails 1/4: a systems benchmark with no robotics question. Still, no documented Orin Nano VLA run was found, so measured numbers would be a useful *side chapter* (via LeRobot async inference).
8. **HIL-SERL replication on an SO-101.** Fails 4/5 as a research question: a documented pipeline with no question attached. It is a good skills or warm-up project. A question could emerge from LeRobot's own warning that it "is very sensitive to the intervention strategy", but that sits outside this note's evidence.
9. **Consequence-engine-style safety or ethics layer on the 3pi+.** Fails 2: safety filters and shielding are strong baselines (*background knowledge*). Fails 3: the hazard would have to be manufactured.
10. **DSRL-style noise-space steering with a TailRL objective, standalone.** Fails 4: method-first. It is acceptable only as the optional arm of Candidate A.

### Gaps
- **The surviving candidate's nearest work** (the Tail-and-Judge thesis, judge chapter) was not read. If it already contains a physical-robot calibration of planner exploitation versus k, Candidate A's novelty shrinks to the low-cost-arm replication. Read it before the topic is agreed in early 2027.
- **Not searched:** planar-pushing MPC baselines, Push-T on SO-101 community results, SO-101 price, and BRL equipment. These are covered by other notes or are open.
- **Not run:** the targeted search for "robot proposals trained for best-of-k against a verifier" (search budget exhausted).
