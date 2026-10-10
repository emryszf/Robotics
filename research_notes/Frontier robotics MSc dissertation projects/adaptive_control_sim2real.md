# Adaptive motion control and sim-to-real on small differential-drive robots (Pololu 3pi+ 32U4), judged against classical adaptive control — research notes as of 10 Oct 2026

**Access caveat (read first).** In this session WebFetch failed DNS for every host, and the egress proxy refused pololu.com, forum.pololu.com, arxiv.org, science.org, IEEE Xplore, openreview.net, roboticsproceedings.org, bristol.ac.uk, semanticscholar, st.com, docs.espressif.com and all docs sites (HTTP 403). Only `raw.githubusercontent.com` and `pypi.org` could be reached. The web-search budget, shared by all agents, ran out after about ten of my searches. So:
- Pololu facts come from search-engine extracts of Pololu pages, checked against Pololu's own Arduino library source and the Bristol RSS course code on GitHub.
- Literature claims rest on abstracts or snippets plus the official code repositories (primary sources).
- Anything I know only from memory is flagged **"prior knowledge, unverified this session"** and kept out of Cited Findings.
- Numbers I computed are labelled **computed** (from sourced data) or **estimated** (from first principles), with the reasoning given.

---

## Q1. Platform facts: what the student's robot is and what the Bristol RSS setup achieves

### Takeaway
The Standard Edition 3pi+ 32U4 has 30:1 (29.86:1) MP 6 V micro-metal gearmotors. They run from a **regulated 8 V motor rail** through DRV8838 drivers, so battery sag is largely engineered out. The encoders give 358.3 counts per wheel revolution. Pololu quotes a top speed of 1.5 m/s and a mass of about 100 g without batteries. The sensors are an LSM6DS33 + LIS3MDL IMU, five RC-timed IR line sensors and two reflectance-based bump sensors. Bristol's own measured data show that moving from table to carpet cuts wheel speed at a fixed PWM by only 3–7% at moderate speed, but by 20–50% near the ±12–14 PWM dead-band. The 2026 RSS platform closes its controller loop on the M5Stamp C3 at 100 Hz over 400 kHz I2C.

### Cited Findings
**Motor and drivetrain**
- 30:1 MP 6 V micro metal gearmotor (29.86:1). At 6 V: no-load ≈720 rpm and 0.07 A; stall current 0.67 A; stall torque 0.33 kg·cm; maximum output power 0.57 W; maximum-efficiency point ≈510 rpm, 0.21 A, 0.080 kg·cm. Pololu labels the stall figures "theoretical" (sourced; search-engine extract, page not opened). — [Pololu 2364 specs](https://pololu.com/product/2364/specs); [Pololu 2378 specs](https://www.pololu.com/product/2378/specs)
- **Conflicting older listing:** a third-party listing returned by the same search gives 730 rpm, 40 mA free-run, 0.6 kg·cm (8 oz·in) and 0.7 A stall for the same MP unit (sourced, exact retailer page not identified). — [search set incl. SparkFun 8911](https://sparkfun.com/products/8911); [Core Electronics](https://core-electronics.com.au/30-1-micro-metal-gearmotor-mp.html). I could not confirm which Pololu item number (extended-shaft variant) is fitted to the 3pi+.
- **Top speed:** Pololu's user's-guide table lists the Standard Edition (30:1 MP 6 V) at 1.5 m/s, "a good combination of speed and controllability" (sourced; extract). — [3pi+ 32U4 User's Guide](https://www.pololu.com/docs/0J83/all); [product 4975](https://www.pololu.com/product/4975/cat60)
- **Encoders:** 12 counts per motor-shaft revolution (both edges of both channels), so 29.86 × 12 ≈ 358.3 counts per wheel revolution (sourced; extract). — [User's Guide 5.4](https://www.pololu.com/docs/0J83/5.4). The 2026 RSS I2C protocol also states "Encoder resolution remains fixed at the documented 358.3 counts/revolution". — [i2c_datatypes.h](https://raw.githubusercontent.com/paulodowd/RoboticsScienceAndSystems/main/StampC3_Template/i2c_datatypes.h)
- Encoders are decoded in pin-change and external interrupts on the ATmega32U4, using XOR'd channels. — [Pololu3piPlus32U4Encoders.cpp](https://raw.githubusercontent.com/pololu/pololu-3pi-plus-32u4-arduino-library/master/src/Pololu3piPlus32U4Encoders.cpp)
- **Motor PWM:** Timer1 runs phase-correct at "16MHz / 1 (prescaler) / 2 (phase-correct) / 400 (top) = 20kHz". Speed commands are clamped to ±400, which also confirms the 16 MHz clock. — [Pololu3piPlus32U4Motors.cpp](https://raw.githubusercontent.com/pololu/pololu-3pi-plus-32u4-arduino-library/master/src/Pololu3piPlus32U4Motors.cpp)
- **Motor drivers:** one DRV8838 per motor (sourced). — [User's Guide 5.7](https://www.pololu.com/docs/0J83/5.7); [RSS L1 Motors labsheet](https://github.com/paulodowd/SEMTM0042_43/blob/main/Labsheets/Core/L1_Motors_SEMTM0042_43.ipynb)
- **Wheel diameter and wheelbase:** Pololu's figures were *not found*. The RSS 2026 odometry template defaults to radius 16 mm, separation 90 mm and 358.3 CPR, explicitly labelled "Nominal starting values, not measured calibration results". — [OdometryModel.h](https://raw.githubusercontent.com/paulodowd/RoboticsScienceAndSystems/main/StampC3_Template/OdometryModel.h). Students calibrate `wheel_radius` from a 100 mm drive and `wheel_sep` from a 90° hand rotation. — [RSS L3 Motion Control labsheet](https://github.com/paulodowd/SEMTM0042_43/blob/main/Labsheets/Core/L3_MotionControl_SEMTM0042_43.ipynb)

**Mass and size**
- 100 g assembled, without batteries; 97 × 96 × 36 mm; 9.7 cm diameter (sourced; extract). — [Pololu 4975 specs](https://www.pololu.com/product/4975/specs); [RobotShop listing](https://www.robotshop.com/products/pololu-3pi-32u4-oled-robot-assembled-standard-edition-301-mp-motors)

**Power: does battery sag change motor behaviour?**
- **Power path:** four AAA cells (NiMH ≈4.8 V, alkaline ≈6 V nominal). VBAT passes through reverse protection and a pushbutton power switch to give VSW. VSW feeds the motor regulator, whose output VM powers the DRV8838s.
- **Regulator cut-out:** the motor regulator is "designed to cut out at a higher voltage than the 5 V logic regulator", so large transients cannot reset the MCU.
- **Battery monitoring:** VSW is readable through a divider on analog pin 1 (PF6) (sourced; extract). — [User's Guide 5.7 Power](https://www.pololu.com/docs/0J83/5.7)
- **Speed independence:** the product page says the motors run from a constant 8 V independent of battery charge, so speed does not vary with battery voltage. The guide's wording is that battery voltage "does not typically have a major impact" (sourced; extract). — [product 3737](https://www.pololu.com/product/3737)
- **Corroborated by the course:** RSS labsheet: "VM: is the regulated voltage supplied to the motors at 8v". — [RSS SL0 Measuring Battery Voltage](https://github.com/paulodowd/SEMTM0042_43/blob/main/Labsheets/Supp/SL0_MeasuringBatteryVoltage.ipynb)
- The 3pi+ 2040 guide (a different board) says battery condition "can still have an impact on motor performance in some situations". — [3pi+ 2040 guide 6.7](https://www.pololu.com/docs/0J86/6.7). The 32U4 regulator's current rating was *not found*.

**Sensors**
- **IMU:** ST LSM6DS33 (accelerometer + gyro) and ST LIS3MDL magnetometer. The library only detects the `LSM6DS33_LIS3MDL` type. — [Pololu library README](https://raw.githubusercontent.com/pololu/pololu-3pi-plus-32u4-arduino-library/master/README.md); [Pololu3piPlus32U4IMU.cpp](https://raw.githubusercontent.com/pololu/pololu-3pi-plus-32u4-arduino-library/master/src/Pololu3piPlus32U4IMU.cpp); [User's Guide 5.6](https://www.pololu.com/docs/0J83/5.6)
- The newer 3pi+ 2040 uses an LSM6DSO. — [3pi+ 2040 guide 6.6](https://www.pololu.com/docs/0J86/6.6). I found no source saying the 32U4 board switched IMU, so check the chip marking.
- **IMU as configured by RSS 2026:** the StampC3 reads the IMU directly over the shared I2C bus, set to "104 Hz using +/-2 g … +/-245 degrees/s". Sensitivities are 0.061 mg/count and 8.75 mdps/count; the LIS3MDL runs at ±4 gauss. — [RobotIMU.h](https://raw.githubusercontent.com/paulodowd/RoboticsScienceAndSystems/main/StampC3_Template/RobotIMU.h)
- The RSS labsheet warns: "the IMU is known to have a relatively high degree of noise … you are likely to get a better estimate of position from your encoders". — [RSS SL3 Inertial Sensors](https://github.com/paulodowd/SEMTM0042_43/blob/main/Labsheets/Supp/SL3_InertialSensors.ipynb)
- **Line sensors:** five downward IR reflectance sensors (emitter + phototransistor), read by QTR-style RC decay timing. — [User's Guide 5.5](https://www.pololu.com/docs/0J83/5.5). The default RC timeout is 4000 µs, with 1000–3000 µs recommended. — [LineSensors.h](https://raw.githubusercontent.com/pololu/pololu-3pi-plus-32u4-arduino-library/master/src/Pololu3piPlus32U4LineSensors.h). Sensor spacing, lateral span and height range were *not found*.
- **Bump sensors:** two reflectance sensors that detect deflection of the bumper flaps. They share the EMIT pin with the line sensors (high = line emitters on, low = bump emitters on). The pressed threshold is baseline × (100 + 50)/100, from a 50% default margin. — [User's Guide 5.5](https://www.pololu.com/docs/0J83/5.5); [BumpSensors.h](https://raw.githubusercontent.com/pololu/pololu-3pi-plus-32u4-arduino-library/master/src/Pololu3piPlus32U4BumpSensors.h)

**The 2026 Bristol RSS platform: the student's 3pi+ plus M5Stamp C3 (repo created 16 Sep 2026)**
- **Bus roles:** the StampC3 is I2C master at `Wire.setClock(400000)`. The 3pi+ runs "robot middleware" as an I2C slave at 0x08. — [Robot.cpp](https://raw.githubusercontent.com/paulodowd/RoboticsScienceAndSystems/main/StampC3_Template/Robot.cpp); [i2c_datatypes.h](https://raw.githubusercontent.com/paulodowd/RoboticsScienceAndSystems/main/StampC3_Template/i2c_datatypes.h)
- **Middleware registers:**
  - surface snapshot (5 readings plus a µs timestamp)
  - encoder snapshot (2 × int32)
  - pose estimate computed on the 32U4
  - motor command (2 × int16 PWM)
  - helper motions (distance, rotation; fixed internal PWM, safety timeout)
  - a 2 s self-test
  - motion geometry

  There is **no wheel-speed setpoint register**, so by default wheel-speed feedback is closed across I2C. — same file.
- **Loop rates:** `UPDATE_INTERVAL_MS = 10`, described as the "Fastest interval between controller and I2C acquisition cycles" (100 Hz). Telemetry goes every 50 ms over a WiFi access point and TCP. The supplied baseline is a nominal line follower with `BASE_PWM = 45`. — [Controller.h](https://raw.githubusercontent.com/paulodowd/RoboticsScienceAndSystems/main/StampC3_Template/Controller.h); [StampC3_Template.ino](https://raw.githubusercontent.com/paulodowd/RoboticsScienceAndSystems/main/StampC3_Template/StampC3_Template.ino)

**What RSS students normally do and achieve (SEMTM0042/43 labsheets)**
- **Labsheet content:** PWM and dead-band, encoder speed estimation in 10 ms windows, PID wheel-speed control, nested PID (a heading PID feeding two wheel-speed PIDs), hand calibration of kinematic odometry, line following, and a "Foraging Challenge".
- **Example Assessment 2 projects:** box pushing, leader–follower, 2D localisation. — [SEMTM0042_43 README](https://raw.githubusercontent.com/paulodowd/SEMTM0042_43/main/README.md); [L3 labsheet](https://github.com/paulodowd/SEMTM0042_43/blob/main/Labsheets/Core/L3_MotionControl_SEMTM0042_43.ipynb)
- **Measured dead-band:** "a large gap … across PWM values -12 to +14" on a smooth table without USB, with speed taken from encoders at 100 Hz (10 ms). Left and right wheels differ on both table and carpet: "one wheel is always rotating slightly faster … This trend also seems to get worse at higher PWM values". The variance of speed grows with PWM. — [RSS L1 Motors labsheet](https://github.com/paulodowd/SEMTM0042_43/blob/main/Labsheets/Core/L1_Motors_SEMTM0042_43.ipynb)
- **Surface data I analysed** (computed from Bristol's [surfaces.csv](https://raw.githubusercontent.com/paulodowd/SEMTM0042_43/refs/heads/main/ExampleData/surfaces.csv): 10 samples per PWM, wheel and surface, 10 ms windows; PWM in labsheet units, the Arduino `analogWrite` 0–255 scale, which I infer from the labsheet's use of `analogWrite`):

  | PWM | carpet/table speed ratio (mean of both wheels) |
  |---|---|
  | 20 | 0.52 |
  | 24 | 0.63 |
  | 28–32 | 0.81–0.82 |
  | 36–44 | 0.88–0.91 |
  | 48–116 | 0.93–0.97 |

  Other figures from the same file:
  - **Table speed at PWM 116:** 2.43 counts/ms ≈ 0.68 m/s, assuming the nominal 32 mm wheel.
  - **Left/right asymmetry:** 2–4% at PWM 100–116.
  - **Per-sample standard deviation:** 0.03–0.12 counts/ms, which is about one count of quantisation at 10 ms.
- **Asymmetric dead-band** (computed from [deadband.csv](https://raw.githubusercontent.com/paulodowd/SEMTM0042_43/refs/heads/main/ExampleData/deadband.csv)): the left wheel starts moving at about PWM 14, the right at about PWM 6–12.

**Ground truth available in the Bristol ecosystem**
- Paul O'Dowd's ArUco tracking server ("SwarmTracker" WiFi) sends each robot its pose packet (x, y, θ, sequence, quality) "every 100ms (10hz)". It uses 4×4, 45 mm markers and logs over TCP. — [ProcessingTrackingVisualiser README](https://raw.githubusercontent.com/paulodowd/ProcessingTrackingVisualiser/main/README.md)

### Inferences
- **Speed quantisation (estimated, from the sourced CPR and the nominal 32 mm wheel):**
  - One count is π × 32 mm / 358.3 = 0.281 mm, which is **28 mm/s per count in a 10 ms window**.
  - At 1.5 m/s that is ≈53 counts per 10 ms (≈2% resolution). At 0.1 m/s it is ≈28%.
  - So **low-speed velocity estimation is quantisation-limited**, which matters exactly where surface effects are largest (near the dead-band). Period (time-between-edges) measurement on the 32U4 is the classical fix.
- **Top-speed consistency check (estimated):**
  - Scaling the 6 V free speed linearly to 8 V gives ≈960 rpm, or ≈1.61 m/s with a 32 mm wheel. This is consistent with Pololu's 1.5 m/s.
  - Extrapolating the RSS table data linearly to PWM 255 gives ≈1.57 m/s.
- **Achievable dynamics (estimated, first-principles; to be replaced by week-one measurements):**
  - **Stall torque at 8 V:** ≈0.44 kg·cm ≈ 0.043 N·m per motor, i.e. ≈2.7 N per wheel and ≈5.4 N total.
  - **Total mass:** ≈0.15 kg (100 g sourced, plus ~45–50 g for 4×AAA cells, plus the Stamp; the battery mass is my estimate).
  - **Launches and braking:** motor-limited launch acceleration is ≈36 m/s². Friction-limited acceleration is μg × (fraction of weight on the drive wheels) ≈3–10 m/s² for μ ≈ 0.5–1. So **full-PWM launches and braking are probably traction-limited over most of the speed range**. Motor torque (∝ 1 − v/v₀) drops below ~7 m/s² only above ≈1.3 m/s.
  - **Turns:** the minimum radius before lateral slip, R = v²/(μg), is ≈0.15 m at 1 m/s and ≈0.33 m at 1.5 m/s for μ ≈ 0.7.
  - **Caveat:** at 8 V each motor's stall current is ≈0.9 A (≈1.8 A for both). If the 8 V regulator cannot supply that, torque is capped and the regulator may cut out. That would bring back a "battery/regulator" effect, but only in hard transients.
- **Battery state is not a credible perturbation** on this robot under normal operation, because the motor rail is regulated. This removes one leg of the student's "adaptive control" idea.
- **Loop structure (estimated):**
  - With the RSS default (PWM over I2C at ≤100 Hz, 10 ms speed windows), the speed loop carries roughly 10–15 ms of effective delay.
  - Moving the wheel-speed PI loop onto the 32U4 at 0.5–1 kHz (interrupt encoders, 20 kHz hardware PWM) and sending setpoints over I2C is easy and removes most of that delay.
  - An 8-byte encoder read or a 14-byte surface read takes ≈100–140 bit-times, ≈0.25–0.35 ms at 400 kHz before software overhead. So I2C bandwidth is not the 100 Hz bottleneck; line-sensor RC timeouts (1–4 ms) and middleware scheduling are more likely limits.

### Gaps
- **Pololu primary pages:** the product pages, user's guide, schematic (`0J1790`) and datasheets could not be opened. Not found:
  - wheel diameter and wheel spacing (also absent from the [3pi+ 2040 repo README](https://raw.githubusercontent.com/pololu/pololu-3pi-2040-robot/master/README.md), which covers the same chassis; a GitHub code search scoped to that repo for "wheel" returned no hits)
  - line-sensor spacing and range
  - motor-regulator current rating
  - motor R, Kt and rotor inertia
  - gearbox efficiency
- **RSS middleware firmware:** the 32U4-side source and its internal loop rate were not found in the public repo; a scoped code search for an I2C receive handler returned nothing.
- **ESP32-C3 WiFi latency and jitter:** not found.

---

## Q2. When does a well-tuned classical wheel-speed/heading controller degrade at this scale, and is there evidence that learned adaptation beats it?

### Takeaway
No published result was found (the search was incomplete) showing learned adaptation beating tuned classical or adaptive control on a robot of this class (sub-kilogram, differential drive). At the 3pi+'s normal operating points the plant changes are small or engineered out:
- **surface:** 3–7% change in PWM→speed gain at moderate speed;
- **payload:** linear-in-parameter inertia and drag;
- **battery:** regulated out;
- **wheel diameter:** an odometry-calibration problem, not a control problem.

Degradation should appear only at the traction limit, at torque–speed saturation, in the low-speed dead-band/quantisation zone, and when loops are closed over slow links. The best-documented "learned beats classical adaptive" result is on quadrotors (Neural-Fly). Its evaluation included L1, INDI and a no-learning adaptive ablation, which is the evaluation template the student would have to match.

### Cited Findings
- **Surface effect on this robot** is large near the dead-band and small at moderate PWM (carpet/table 0.52 at PWM 20 versus 0.93–0.97 at PWM 48–116). — computed from [RSS surfaces.csv](https://raw.githubusercontent.com/paulodowd/SEMTM0042_43/refs/heads/main/ExampleData/surfaces.csv)
- **Neural-Fly** (O'Connell, Shi, et al.; *Science Robotics*, 4 May 2022):
  - Compared methods: "nonlinear baseline control", "incremental nonlinear dynamics inversion control" (INDI), "L1 adaptive control", "Neural-Fly-Constant, our adaptive controller without any learning", Neural-Fly-Transfer and Neural-Fly.
  - Wind conditions ran from 0 to 12.1 m/s.
  - Offline learning is "Domain Adversarially Invariant Meta Learning (DAIML)".

  Read: README in full; paper not accessed; quantitative gains not verified. — [neural-fly README](https://raw.githubusercontent.com/aerorobotics/neural-fly/main/README.md); [paper](https://www.science.org/doi/abs/10.1126/scirobotics.abm6597)
- **DATT** (PREPRINT per its README; arXiv 2310.09053): "we introduce our architecture DATT and compare it with PID and MPPI as baselines". Adaptation can use L1 or RMA-style modules (config names `datt_adaptive_L1_config` and `datt_adaptive_RMA_config`). Read: README in full. — [DATT README](https://raw.githubusercontent.com/KevinHuang8/DATT/main/README.md)
- **AnyCar:** "outperforms specialist models by up to 54%" in real-world experiments. The baselines are learned specialist models, not classical adaptive controllers. Read: abstract only. — [arXiv 2409.15783](https://arxiv.org/abs/2409.15783)
- **RMA for wheeled robots:** a search for wheeled or ground-vehicle RMA follow-ups returned only legged, biped and manipulator variants. This means "no overlap found", not established novelty. — search set incl. [A-RMA, IROS 2022](https://hybrid-robotics.berkeley.edu/publications/IROS2022_AdaptingRMA_Biped.pdf); "Rapid Embodiment Adaptation for Quadrupedal Locomotion", PREPRINT Aug 2026, [arXiv 2608.01506](https://arxiv.org/html/2608.01506)

### Inferences
- **Perturbation → what changes in the plant → classical remedy → expected size.** Sizes are sourced where marked, otherwise estimated.

  | Perturbation | What changes in the plant | Classical remedy | Expected size |
  |---|---|---|---|
  | Battery | almost nothing (8 V regulated) | none needed | regulated out (sourced) |
  | Surface | PWM→speed gain and dead-band; rolling resistance | integral action, dead-band feedforward, RLS on gain/friction | 3–7% gain change at mid-speed; up to 50% near the dead-band (sourced data) |
  | Payload mass | wheel-load inertia; rolling resistance | integral action; adaptive acceleration feedforward (RLS on mass, linear-in-parameter) | modest (estimated) |
  | Payload offset | yaw torque ∝ forward acceleration × lateral offset; uneven wheel loading changes traction | gyro heading loop plus RLS feedforward on acceleration | modest (estimated) |
  | Wheel diameter or wear | distance and odometry scale only; encoder-based speed loops are unaffected | UMBmark-style calibration, gyro fusion, online RLS of wheel radii | an odometry problem, not a control problem |

  Every row is a linear-in-parameter, slowly varying uncertainty, which is textbook territory for MRAC, L1 or RLS. A learned adaptation module has no structural advantage there.
- **Identifiability is the sceptical core.** The friction limit μ does not appear in the no-slip dynamics, so it is unobservable from encoder or IMU history until slip begins. Neither RLS/MRAC nor an RMA-style history encoder can anticipate a low-μ patch; both can only react once slip shows up as an IMU-versus-encoder mismatch. The only anticipatory channel would be exteroceptive, such as the downward IR sensors. This caps the upside of "learned adaptation to surface changes".
- **Where classical control should measurably degrade (estimated):**
  1. Aggressive launches, braking and tight fast turns at the traction limit. Encoder-only speed loops keep tracking *wheel* speed while the body lags, so odometry corrupts silently.
  2. Near top speed, where available torque tends to zero; this is a saturation and feasibility problem.
  3. Low speeds in the dead-band, where quantisation and static friction dominate.
  4. Loops closed over I2C at 100 Hz or over WiFi (delay).

  Each has a classical counterpart: traction control with IMU-aided slip detection; MPC or explicit saturation handling; friction feedforward with period-based speed measurement; and on-board loops.
- **"By how much" learned beats classical on a comparable platform:** not found.

### Gaps
- I could not search for classical adaptive control applied to differential-drive or small wheeled robots (MRAC, L1, disturbance observers, RLS + MPC). Prior-knowledge pointers to verify before citing (unverified this session):
  - Fukao, Nakagawa & Adachi, IEEE T-RA 2000 (adaptive tracking of nonholonomic robots)
  - Borenstein & Feng, IEEE T-RA 1996 (UMBmark odometry calibration)
  - Ostafew, Schoellig & Barfoot, IJRR 2016 (learning-based NMPC path tracking on a ground robot)
  - Hanover et al., RA-L 2022 (adaptive NMPC for quadrotor payloads)
  - Nagabandi et al., ICLR 2019 (meta-RL adaptation on a legged millirobot, including payload and terrain)
- Neural-Fly's quantitative margins over L1 and INDI, and any ground-robot analogue: not verified.

---

## Q3. Strongest overlapping learned-adaptation work, read at methods level

### Takeaway
The closest methods are:
- in-context or history-based adaptation for agile wheeled cars (AnyCar, onboard mini-PC plus MPPI);
- real-to-sim alignment by learned delta actions (ASAP, humanoid, mocap);
- RMA's latent-extrinsics adaptation (legged);
- learned basis plus classical adaptive law (Neural-Fly, quadrotor).

None targets a microcontroller-class differential-drive robot. None of the wheeled work, as far as the abstracts show, benchmarks against classical adaptive or traction control. Song et al. (2023) attribute RL's racing advantage to its task-level objective, not better optimisation.

### Cited Findings
- **RMA** (Kumar, Fu, Pathak, Malik; RSS 2021; peer-reviewed).
  - Method: a base policy conditioned on a latent extrinsics vector encoding sim parameters; an adaptation module estimates that vector online from proprioceptive history.
  - A-RMA (IROS 2022) found "the extrinsics estimator can be imperfect" when scaling to bipeds and fine-tunes the base policy with the imperfect estimator.
  - Read: secondary description in the A-RMA paper (via search extract); RMA itself not accessed. — [A-RMA IROS 2022 PDF](https://hybrid-robotics.berkeley.edu/publications/IROS2022_AdaptingRMA_Biped.pdf); [RMA arXiv 2107.04034](https://arxiv.org/pdf/2107.04034)
- **AnyCar** (Xiao, Xue, Tao, Kalaria, Dolan, Shi). arXiv Sep 2024; the README cites it only as arXiv. An IEEE Xplore PDF is indexed, so it was probably published at an IEEE conference in 2025 (likely ICRA 2025), but the venue is **unverified**; treat it as PREPRINT until checked.
  - **Model and controller:** a transformer dynamics model trained in JAX, with "real-world finetuning" and sampling-based MPC (MPPI). The baselines in the repo are a dynamic bicycle model, pure pursuit and PID.
  - **Training data:** Assetto Corsa Gym, Isaac Sim, MuJoCo and a numeric dynamic-bicycle-model simulator.
  - **Platforms:** F1/10 and a custom F1/16 (Traxxas 1/16 chassis, VESC 6 EDU with IMU, 3800 kV BLDC motor, 2S LiPo, onboard mini PC with no GPU, optional Hokuyo LiDAR).
  - **Claims:** "outperforms specialist models by up to 54%" and adapts "even in the wild and under large state estimation errors".
  - **Limitations:** the limitations section was not accessed. One secondary review notes that generalist models "often lack the precision required for high-speed maneuvers".
  - Read: abstract, the repo READMEs (main, hardware, data collection, dynamics) and a secondary review. — [arXiv](https://arxiv.org/abs/2409.15783); [IEEE PDF](https://ieeexplore.ieee.org/iel8/11127273/11127223/11128396.pdf); [README](https://raw.githubusercontent.com/LeCAR-Lab/anycar/main/README.md); [hardware README](https://raw.githubusercontent.com/LeCAR-Lab/anycar/main/hardware/README.md); [car_collect README](https://raw.githubusercontent.com/LeCAR-Lab/anycar/main/car_collect/README.md); [car_dynamics README](https://raw.githubusercontent.com/LeCAR-Lab/anycar/main/car_dynamics/README.md); [Moonlight review](https://www.themoonlight.io/en/review/anycar-to-anywhere-learning-universal-dynamics-model-for-agile-and-adaptive-mobility)
- **Neural-Fly** (*Science Robotics* 2022; peer-reviewed). DAIML offline meta-learning of a wind-invariant representation, combined with online adaptation. Baselines: L1, INDI, a nonlinear baseline, and the no-learning ablation NF-Constant. Read: README only. No ground-robot analogue was found (incomplete search). — [README](https://raw.githubusercontent.com/aerorobotics/neural-fly/main/README.md)
- **ASAP** (He et al.; RSS 2025; peer-reviewed).
  - **Method:** a "delta action model learns corrective actions from the mismatch between replayed real-world trajectories and their simulated counterparts". It is used to fine-tune the policy in simulation and is not used at deployment.
  - **Limitations reported by secondary summaries:** dependence on motion capture; delta-model training is data-hungry; repeated motor overheating and hardware failures (two Unitree G1s reportedly broke) made a full 23-DoF delta model infeasible, so a 4-DoF ankle model was trained instead.
  - **Future work (secondary summary):** generalising the delta model across limbs and sensors; learning from fewer real samples.
  - Read: README in part and secondary summaries; the primary limitations text was not read. — [ASAP README](https://raw.githubusercontent.com/LeCAR-Lab/ASAP/main/README.md); [RSS 2025 page](https://roboticsconference.org/program/papers/66/); [alphaxiv overview](https://www.alphaxiv.org/overview/2502.01143); [swiftscholar summary](https://www.swiftscholar.net/paper/6960b7f24ed9e1885047eb39); [emergentmind summary](https://www.emergentmind.com/papers/2502.01143)
- **Song, Romero, Müller, Koltun, Scaramuzza** (*Science Robotics* 8(82), Sep 2023; peer-reviewed).
  - Finding: RL outperforms optimal control for drone racing, and "the fundamental advantage of RL over OC is not that it optimizes its objective better but that it optimizes a better objective".
  - OC's "decomposition limits the range of behaviors that can be expressed by the controller". RL "can leverage domain randomization to cope with model uncertainty".
  - Results: peak acceleration above 12 g and peak speed of 108 km/h.
  - Read: abstract only. — [arXiv 2310.10943](https://arxiv.org/abs/2310.10943); [Science Robotics](https://www.science.org/doi/10.1126/scirobotics.adg1462)
- **Recent adaptation PREPRINTS** (abstract-level):
  - Rapid Embodiment Adaptation for quadrupeds (Aug 2026): a cross-embodiment base policy plus an adaptation module that infers embodiment parameters. — [arXiv 2608.01506](https://arxiv.org/html/2608.01506)
  - World models as reference trajectories for RMA (Walker2D in simulation). — [arXiv 2505.15589](https://arxiv.org/pdf/2505.15589)

### Inferences
- **Methods-level gaps relevant to the 3pi+:**
  1. Every overlapping system has orders of magnitude more compute or sensing than the 3pi+ (onboard PC plus MPPI for AnyCar; mocap for ASAP; flight computers for Neural-Fly). The open question is whether *any* of the learned adaptation survives a 100 Hz loop on an ESP32-C3.
  2. Only Neural-Fly benchmarks against classical adaptive control and a no-learning adaptive ablation. Any 3pi+ study must do the same, or it cannot claim learning helped.
  3. ASAP's main cost (hardware damage, data scarcity) mostly disappears on a 3pi+ (estimated). That makes "parametric identification versus learned delta action" cheap to test properly at small scale.
- **Song et al.'s critique** targets planning/tracking decompositions. Contouring MPC already optimises a progress (task-level) objective, so for a 2-input planar robot the RL-objective advantage should be smaller (inference). Any racing comparison must use an MPCC-type baseline, not trajectory tracking.

### Gaps
- Not accessed:
  - Kabzan et al. (RA-L 2019, GP-MPC). Prior knowledge, unverified: it ran on a full-size Formula Student car, not a small car. The small-car GP-MPC reference is Hewing, Liniger & Zeilinger (ECC 2018, 1:43 cars); MPCC on 1:43 cars is Liniger et al. (OCAM 2015).
  - F1TENTH RL sim-to-real 2024–2026.
  - Neural-Fly ground analogues.
  - Primary limitations and future-work sections of AnyCar, ASAP and RMA.
- AnyCar's peer-reviewed venue is unconfirmed.

---

## Q4. Is the 3pi+ ever genuinely dynamics- or traction-limited, and what limits time-optimal line following?

### Takeaway
On first-principles estimates the Standard 3pi+ is **traction-limited** for full-PWM launches and braking over most of its speed range and in fast tight turns, and **motor-torque-limited** only near top speed. These estimates need week-one confirmation. Line-following speed is more plausibly limited by **sensing geometry** (five sensors, no preview) and tuning than by dynamics: Pololu's own PID example caps the Standard Edition at 50% speed. No credible RL-versus-PID/MPC line-follower literature could be checked.

### Cited Findings
- **Pololu's LineFollowerPID example:**
  - "A maxSpeed of 400 would let the motors go at top speed, but the value of 200 here imposes a speed limit of 50%."
  - "making the 3pi+ go faster on a line following course might involve more than just increasing this number; you will often have to adjust the PID constants too".
  - Standard Edition settings: maxSpeed 200, P = 1/4, D = 1, which "seems to work well at low speeds for all of our 3pi+ editions". Hyper Edition: maxSpeed 100. Turtle Edition: 400.

  — [LineFollowerPID.ino](https://raw.githubusercontent.com/pololu/pololu-3pi-plus-32u4-arduino-library/master/examples/LineFollowerPID/LineFollowerPID.ino)
- **Line-sensor read time** is bounded by the RC timeout (4000 µs default, 1000–3000 µs recommended). — [LineSensors.h](https://raw.githubusercontent.com/pololu/pololu-3pi-plus-32u4-arduino-library/master/src/Pololu3piPlus32U4LineSensors.h)
- **Surface dependence is concentrated near the dead-band** (Q1 table). — [RSS data](https://raw.githubusercontent.com/paulodowd/SEMTM0042_43/refs/heads/main/ExampleData/surfaces.csv)

### Inferences
- **Limits (estimated, see Q1 arithmetic):**
  - Launch and braking: traction (μg ≈ 3–10 m/s² against ≈36 m/s² motor-limited).
  - High-speed cornering: lateral friction (R_min ≈ 0.15–0.33 m at 1–1.5 m/s).
  - Top speed: torque–speed saturation.
  - Line following: the sensor array's lateral span and lack of preview. At 1.5 m/s the robot travels 15 mm per 10 ms control cycle, so on curves the line can leave the array before the controller reacts (estimated; sensor span not found).
- **Classical baselines for time-optimal line following:** PD with curvature-based speed scheduling, plus iterative learning control or lap memory on repeated tracks. An RL policy cannot create preview information that the sensors do not provide, so the problem is likely sensing-limited (inference).
- **A waypoint "race" with external tracking** (RSS ArUco at 10 Hz, or a webcam) is latency-dominated: 100 ms at 1.5 m/s is 15 cm of travel. The gap would be created by the tracking system, not by the robot's dynamics.

### Gaps
- Not found:
  - line-following competition top speeds and failure modes
  - any peer-reviewed RL-versus-tuned-PID/MPC comparison for small line followers or micro-robots (the search budget ran out before this was searched)
  - measured μ for typical lab surfaces against 3pi+ tyres
  - the fraction of weight carried by the drive wheels versus the casters

---

## Q5. GPU-parallel and differentiable simulators: what they model that matters for a 150 g differential-drive robot

### Takeaway
The material change in October 2025 to October 2026 is in **MuJoCo**:
- **3.5.0 (12 Feb 2026):** MuJoCo Warp was officially released, alongside a System Identification toolbox and actuator/sensor delays.
- **3.7.0 (14 Apr 2026):** a `dcmotor` actuator. It accepts a datasheet-style `nominal = voltage, stall torque, no-load speed` and offers optional inductance, cogging, thermal effects, LuGre friction and torque/current limits.
- **3.12.0 (20 Aug 2026):** a voltage-input and controller redesign.

Together these directly cover what the student's kinematic simulator missed. Caveats:
- MJX-JAX, the differentiable path, does not list the dcmotor dynamics type; MJX-Warp supports it but is not differentiable.
- No differential-drive or Pololu model exists in MuJoCo Menagerie.
- Isaac Lab offers a torque–speed-saturation DC motor and randomised delays.
- No simulator throughput figure could be verified.

For a planar two-wheel robot, an extended JAX planar model is the simplest adequate simulator.

### Cited Findings
- **MuJoCo changelog:**
  - 3.5.0 (12 Feb 2026): "MuJoCo Warp is now officially released"; "Added a new System Identification toolbox (Python)"; "Actuators and sensors now support arbitrary delays via history buffers, and sensor values can be computed at intervals larger than the simulation timestep".
  - 3.7.0 (14 Apr 2026): "Added the dcmotor actuator for modeling DC motors. Supports optional electrical dynamics (inductance), cogging torque, thermal resistance variation, and LuGre friction".
  - 3.12.0 (20 Aug 2026): the dcmotor controller was redesigned. The `input` signature is any subset of `[pos, vel, ff, voltage]`, and the "drive voltage compensates back-EMF as in a current-controlled driver". A `pid` actuator subsuming the mujoco.pid plugin was also added.
  - 3.14.0 (22 Sep 2026): a multi-input actuator API.

  — [changelog.rst](https://raw.githubusercontent.com/google-deepmind/mujoco/main/doc/changelog.rst). The latest release is mujoco 3.15.0 (5 Oct 2026). — [PyPI mujoco](https://pypi.org/pypi/mujoco/json)
- **dcmotor parameters:**
  - `resistance` and `motorconst` (Kt, Ke).
  - `nominal` = voltage, stall_torque, no_load_speed, from which "The compiler derives K = voltage / no_load_speed and R = K · voltage / stall_torque".
  - `inductance` (L or time constant).
  - `saturation` = torque, current, current_rate.
  - `cogging` and `lugre` (stiffness, damping, coulomb, static, stribeck).
  - `controller` = kp, ki, kd, slewmax, Imax, Vmax.
  - The docs note dcmotor "does not conform to the affine gain / bias structure … except for the stateless case".

  — [XMLreference.rst](https://raw.githubusercontent.com/google-deepmind/mujoco/main/doc/XMLreference.rst)
- **sysid toolbox:**
  - Method: nonlinear least squares with box constraints, "Gauss-Newton with finite-difference Jacobians", with all perturbation rollouts in "a single batched call to `mujoco.rollout`, parallelized across threads".
  - What it identifies: contact sliding friction, joint damping, body mass, mass plus centre of mass, full 10-D inertia (Rucker & Wensing pseudo-inertia) and actuator P/D gains.
  - Measurement parameters: "sensor delays, gains, and biases … functionally complete but their API is not yet final".

  — [sysid README](https://raw.githubusercontent.com/google-deepmind/mujoco/main/python/mujoco/sysid/README.md)
- **MJX feature parity:**
  - MJX-JAX actuator dynamics: `NONE, INTEGRATOR, FILTER, FILTEREXACT, MUSCLE`; gains `FIXED, AFFINE, MUSCLE`; bias `NONE, AFFINE, MUSCLE`.
  - MJX-Warp: "All".
  - Differentiability is "mostly supported" in MJX-JAX and "not currently available in MJX-Warp".
  - In MJX-JAX, cylinders "only collide with other primitives". Contact options include condim 1/3/4/6 and pyramidal or elliptic cones.

  — [mjx.rst](https://raw.githubusercontent.com/google-deepmind/mujoco/main/doc/mjx.rst)
- **MuJoCo Warp:** "maintained by Google DeepMind and NVIDIA as part of the Newton project"; "requires an NVIDIA GPU for fast simulation but supports CPU for development". On PyPI since 8 Jan 2026. — [MJWarp README](https://raw.githubusercontent.com/google-deepmind/mujoco_warp/main/README.md); [PyPI](https://pypi.org/pypi/mujoco-warp/json)
- **MuJoCo Playground:** environments are dm_control classic control, quadruped and biped locomotion, and manipulation. It trains with MJX-JAX or MJWarp. No wheeled environment is listed. `playground` 0.2.0 was released 16 Mar 2026. — [README](https://raw.githubusercontent.com/google-deepmind/mujoco_playground/main/README.md); [PyPI](https://pypi.org/pypi/playground/json)
- **MuJoCo Menagerie:** the only "Mobile Bases" entry is "Robot soccer kit omnidirectional"; the mobile manipulators are Stretch, TidyBot, TIAGo and Google Robot. There is no differential-drive or Pololu model. — [Menagerie README](https://raw.githubusercontent.com/google-deepmind/mujoco_menagerie/main/README.md)
- **Isaac Lab:**
  - `DCMotor` "implements a saturation model defined by a linear four quadrant DC motor torque-speed curve", parameterised by `saturation_effort`, `effort_limit` and `velocity_limit`. I found no voltage input or electrical model in `actuator_pd.py`.
  - `DelayedPDActuator` samples a per-environment delay between `min_delay` and `max_delay`.
  - The docs warn: "policies trained with implicit actuators may not transfer to the exact same robot model when using explicit actuators".
  - `isaaclab` 2.3.2.post1 was released 11 Feb 2026.

  — [actuator_pd.py](https://raw.githubusercontent.com/isaac-sim/IsaacLab/main/source/isaaclab/isaaclab/actuators/actuator_pd.py); [actuators.rst](https://raw.githubusercontent.com/isaac-sim/IsaacLab/main/docs/source/overview/core-concepts/actuators.rst); [PyPI](https://pypi.org/pypi/isaaclab/json)
- **Genesis:**
  - `genesis-world` 1.4.3 was released 30 Sep 2026; the first release was 18 Dec 2024. — [PyPI](https://pypi.org/pypi/genesis-world/json)
  - The current README describes multi-physics (rigid, FEM, MPM, PBD/SPH, IPC) and has IMU, domain-randomisation and contact-force examples. A grep found **no throughput claim** in the README as fetched on 10 Oct 2026. — [Genesis README](https://raw.githubusercontent.com/Genesis-Embodied-AI/Genesis/main/README.md)
  - A benchmark repo under a Genesis author's account (zhouxian) only points to a Notion report, which I could not access. — [genesis-speed-benchmark README](https://raw.githubusercontent.com/zhouxian/genesis-speed-benchmark/main/README.md)
- **Brax** 0.14.2 was released 15 Mar 2026. — [PyPI](https://pypi.org/pypi/brax/json)

### Inferences
- **What matters for this robot**, and who models it:

  | Effect | Who models it |
  |---|---|
  | Motor torque–speed and back-EMF, with voltage input | MuJoCo dcmotor (voltage input); Isaac Lab (saturation only) |
  | Dead-band and static friction | MuJoCo LuGre/Coulomb |
  | Loop delay | MuJoCo ≥3.5; Isaac Lab DelayedPD |
  | Wheel–ground friction and slip | MuJoCo soft-contact friction cones; realism of tyre slip unverified, and no slip-curve tyre model found |
  | Caster drag; encoder quantisation; IMU noise | none natively found; the student adds them in observation wrappers |

- **Bridging MJX-JAX's gap:** the stateless DC motor (no inductance) is affine in control and velocity: τ = (K/R)·V − (K²/R)·ω. So it can be written as a `general` actuator with affine gain and bias inside MJX-JAX, keeping differentiability (inferred from the docs' "stateless case" remark).
- **The strongest simple simulator is not a GPU physics engine.** It is the student's own JAX planar model extended with:
  - a DC-motor model (back-EMF, saturation)
  - dead-band and friction
  - a friction-circle slip model
  - loop delay
  - encoder quantisation

  This is fully differentiable and vmap-able on the RTX 3060; its throughput was not measured but is estimated to be far above need. GPU engines add value only if 3D effects (load transfer onto casters, pitch under braking, payload height) matter, and week-one data should decide that. Choosing Isaac Lab or Genesis for this robot would be "method chosen for being new".
- **Credibility of published speed claims:** could not be assessed this session; no figures are reported.

### Gaps
- Not found: independent re-benchmarks of Genesis, MJWarp nightly benchmark figures, MuJoCo Playground paper figures, and the Isaac Lab paper.
- No differential-drive or Pololu models found anywhere (search incomplete).
- Not checked: whether MJWarp supports the 2026 dcmotor and delay features in practice. The parity table says "All" but gives no explicit dcmotor test.

---

## Q6. Learned controllers on microcontrollers (ATmega32U4 / ESP32-C3 class) versus off-board over WiFi

### Takeaway
There is one hard, sourced figure. RLtools benchmarks a [64, 64] MLP at **6.6–8.7 ms per inference on an ESP32-C3** (115–150 Hz), against 0.28–0.33 ms on an Xtensa ESP32 with DSP or optimised kernels. On the student's M5Stamp C3, a modest float network would use most of a 10 ms control period. No ATmega32U4 figures and no WiFi latency or jitter figures were found.

### Cited Findings
- **RLtools inference benchmark**, "two-layer [64, 64] fully-connected neural network", generic / DSP library / RLtools optimised. Read: README and table. — [RLtools README](https://raw.githubusercontent.com/rl-tools/rl-tools/master/README.md); [table image](https://raw.githubusercontent.com/rl-tools/media/master/microcontroller_inference.png); JMLR 25 (2024), [paper](http://jmlr.org/papers/v25/24-0248.html), not read

  | Platform | Generic | DSP library | RLtools optimised |
  |---|---|---|---|
  | ESP32-C3 (RISC-V) | 8716 µs (115 Hz) | 6950 µs (144 Hz) | 6645 µs (150 Hz) |
  | ESP32 (Xtensa) | 4282 µs | 279 µs | 333 µs |
  | Crazyflie | 743 µs | 478 µs | 293 µs |
  | Pixhawk 6C | 133 µs | 93 µs | 53 µs |
  | Teensy 4.1 | 64 µs | 45 µs | 41 µs |

- RLtools lists ESP32 as an inference target and Teensy for training and inference. It showcases "Learning to Fly in Seconds" (arXiv 2311.13081; venue not verified this session). — [RLtools README](https://raw.githubusercontent.com/rl-tools/rl-tools/master/README.md)
- TinyMPC describes itself as "Model-Predictive Control on Resource-Constrained Microcontrollers" (venue and timing figures not accessed). — [TinyMPC README](https://raw.githubusercontent.com/TinyMPC/TinyMPC/main/README.md)
- Paul O'Dowd's "Fixed16": "A 16 bit fixed point representation for Arduino as an alternative to float (32 bits)", created Dec 2025. — [Fixed16 README](https://raw.githubusercontent.com/paulodowd/Fixed16/main/README.md)
- **RSS timing:** the controller cycle is 10 ms over I2C, telemetry 50 ms over WiFi TCP, and tracking-server pose 100 ms (10 Hz). — [Controller.h](https://raw.githubusercontent.com/paulodowd/RoboticsScienceAndSystems/main/StampC3_Template/Controller.h); [tracker README](https://raw.githubusercontent.com/paulodowd/ProcessingTrackingVisualiser/main/README.md)

### Inferences
- **On-board policies on the C3 must be tiny.** At 100 Hz, a float [64, 64] policy leaves only ≈1–3 ms of a 10 ms cycle for I2C, estimation and safety logic. Policies need to be about 16–32 hidden units, or fixed-point/int8 (estimated by MAC scaling). The ATmega32U4 (16 MHz 8-bit AVR without an FPU) is suitable only for trivial fixed-point maps (estimated; no figure found).
- **A cheap hardware change would help.** Replacing the Stamp C3 with an ESP32-S3-class board (Xtensa, single-precision FPU, not in the RLtools table) would very likely give about an order-of-magnitude more on-board network throughput (estimated; price not found).
- **Off-board learned control over WiFi** (desktop or Jetson) is plausible for ≤20–50 Hz high-level commands with timestamping and latency compensation. It is unsuitable for wheel-speed loops (estimated; no sourced jitter figures).
- **The defensible architecture is hierarchical:**
  - the 32U4 runs PI wheel loops at ≥500 Hz;
  - the C3 runs heading, trajectory and adaptation at 100 Hz;
  - off-board compute is used only for training and logging.

### Gaps
- Not found: measured ESP32-C3 WiFi (UDP/TCP/ESP-NOW) latency and jitter, any learned-controller inference on an ATmega32U4, and TinyMPC timings on an ESP32-C3.

---

## Q7. Which candidate questions survive all eight filters, and what kills them in week one?

### Takeaway
One candidate survives cleanly: **C1**, a real-to-sim fidelity study that turns the earlier kinematic-simulator failure into a measured question. The student's open idea, **adaptive motion control under wheel, surface or payload changes, fails filters 2 and 3 in its general form**:
- battery is regulated out;
- surface changes alter the no-slip plant by 3–7% at moderate speed;
- payload effects are linear-in-parameter;
- wheel-diameter change is odometry calibration.

It survives only as a narrow **traction-limit** study (C2), and only if a week-one slip measurement shows real adaptation headroom. Run it as the second half of C1, not on its own.

### Inferences

#### C1 — "Minimum sufficient simulator fidelity for zero-shot sim-to-real on a 150 g differential-drive robot" (recommended; survives)

**(a) Research question.** Which model components are necessary for controllers designed or trained in simulation to transfer zero-shot to the 3pi+, as a function of manoeuvre aggressiveness? The ladder of components:
- acceleration limits
- DC-motor torque–speed/back-EMF
- dead-band and static friction
- loop delay
- encoder quantisation
- wheel–ground slip
- a learned residual or delta action

**(b) Why now.**
- MuJoCo's 2026 releases: the sysid toolbox and delays (12 Feb 2026); dcmotor with datasheet `nominal` parameters (14 Apr 2026); the voltage-input redesign (20 Aug 2026). — [changelog](https://raw.githubusercontent.com/google-deepmind/mujoco/main/doc/changelog.rst)
- ASAP (RSS 2025) made learned delta-action alignment a standard comparator.
- Most concretely, the student's own result: a kinematic simulator without acceleration limits led a ten-line hand controller to beat every learner. The motivating gap is documented, not manufactured.

**(c) Nearest work and what remains open.**
- **ASAP:** humanoid; learned delta action; mocap; data-hungry; a 4-DoF ankle model forced by hardware failures. Open: on a cheap, robust robot with abundant data, does a learned residual capture anything that a physically parameterised (dcmotor + friction + delay + slip) model identified by sysid misses?
- **AnyCar:** multi-simulator data plus real fine-tuning on 1/10–1/16 cars with an onboard PC. Open: a systematic fidelity ablation on microcontroller-scale differential-drive robots.
- **Prior knowledge (unverified this session):** Tan et al., RSS 2018 (Minitaur) reported that actuator and latency modelling were essential for transfer; Hwangbo et al., *Science Robotics* 2019 (ActuatorNet).
- For 3pi+-class wheeled fidelity ablations: **no overlap found, search incomplete**.

**(d) Strongest simple baseline and why it might fail.** The baseline is a calibrated kinematic model with a first-order lag plus speed and acceleration limits fitted from step tests (the existing simulator plus about three parameters). Why it should fail on this platform:
1. Back-EMF makes the acceleration limit fall with speed, so a constant limit is wrong.
2. Launches are probably traction-limited (estimated ≈36 m/s² motor-limited against μg ≈ 3–10 m/s²).
3. The dead-band is surface-dependent, 20–50% near PWM 20–28 (sourced data).
4. There is 10–15 ms of loop delay with the RSS I2C architecture (estimated).

How to show it: hold out aggressive manoeuvres (bang-bang 0.5–1 m moves, maximum-rate turns, figure-8 at rising speed) and compare predicted against measured trajectories (webcam ArUco) and accelerations (IMU).

**(e) First-week test and the result that would kill it.**
- **Day 1–2:** wheels-off-ground PWM steps (25/50/75/100%), reversals and chirps, logged at ~1 kHz on the 32U4 (USB serial). Fit first-order + dead-band + delay, and the dcmotor `nominal` model.
- **Day 3–4:** on 3–4 surfaces, full-PWM launches and stops. Compare IMU x-acceleration with encoder-derived acceleration. Run constant-radius turns at rising speed, comparing the gyro z-rate with the encoder differential yaw rate.
- **Day 5:** replay the old hand-coded controller and one kinematic-sim policy in simulation and on the robot.
- **Kill if:**
  - the first-order + delay model predicts held-out wheel speed within ≈1 count per 10 ms, **and**
  - no slip is detectable at full PWM or at maximum-speed turns on any practical surface, **and**
  - real trajectories match the calibrated kinematic + lag simulator within camera error.

  Then calibration closes the gap and C1 shrinks to a chapter. Fix the thresholds before running.

**(f) Hardware and compute.**
- **Owned:** the 3pi+ and Stamp C3; USB webcam plus printed ArUco markers. Webcam frame rate and latency are estimated at 30 fps and 50–150 ms, so sync offline with an LED or bump event.
- **Cheap extras:** surfaces, a low-μ sheet and payload weights. Prices not found; estimated well under £50 of the £200.
- **Compute:** desktop CPU for sysid (sourced: thread-parallel batched rollouts); RTX 3060 for JAX/MJWarp training (sourced: MJWarp needs an NVIDIA GPU). No cluster needed (estimated).
- **Ask BRL for:**
  1. sessions on Paul O'Dowd's ArUco tracker (10 Hz, sourced) or 2–3 × 2-hour mocap sessions with a marker kit for a 97 mm robot;
  2. a one-week loan of 5–10 RSS 3pi+ robots to measure unit-to-unit spread.

**(g) Headline figure and outcomes.**
- **Headline:** a "fidelity ladder" plot of sim-to-real error (RMS position error over a standard manoeuvre set, and transfer gap in completion time) against model component and aggressiveness. Plot it for a hand controller, an MPC and an RL policy trained in each simulator.
- **Best realistic:** an aggressiveness threshold (acceleration and speed) above which back-EMF, slip and delay modelling becomes necessary, plus RL trained in the identified simulator transferring where kinematic-simulator RL fails.
- **Floor:** a validated, parameterised 3pi+ simulator with uncertainty bounds, and a quantified "first-order + delay suffices up to X" result. Both are still useful.

**(h) Filters.**

| Filter | Verdict | Reason |
|---|---|---|
| 1. Robotics first | pass | the robot's physics is the subject |
| 2. Baseline named | pass | a named baseline with physical failure mechanisms and a test |
| 3. Gap is real | pass, conditional on week one | the gap is physical and documented by the earlier failure; risk that it is small |
| 4. Method justified | pass | system ID is the natural method; the learned residual is only an ablation |
| 5. Nearest work read | partial | ASAP and AnyCar are named; primary texts not read; search incomplete |
| 6. Feasible | pass | owned hardware plus webcam |
| 7. Week-one test | pass | kill test defined |
| 8. Useful if learning loses | pass | a measured fidelity boundary is the product |

**(i) Skills it builds.** System identification and DC-motor modelling; AVR and ESP32 firmware and logging; time-synchronisation and sensor fusion; MuJoCo and JAX simulation; experimental design; sim-to-real RL. These map onto GNC, autonomy, embedded and applied-ML roles.

**(j) People.** Paul O'Dowd (University of Bristol). Evidence: the RSS unit's teaching repos, his 3pi+ IR-communication and I2C expansion boards ("Swarm-B2", "Swarm-Bs") and the ArUco tracker. — [GitHub repos list](https://github.com/paulodowd/SEMTM0042_43). No other BRL staff were verified in this strand.

#### C2 — "Learned versus classical adaptation at the traction limit" (the student's idea reframed; conditional, second-tier)

**(a) Research question.** At the traction limit of a 150 g differential-drive robot, does a history-based learned adaptive controller (RMA-style, trained in the C1 simulator with μ, payload and wheel radius randomised) cut minimum-time point-to-point time or error, across unknown surfaces, surface transitions and payloads? It is compared against classical traction control (IMU-aided slip detection plus acceleration limiting plus an RLS μ update) and a no-learning adaptive ablation.

**(b) Why now.** C1's 2026-era identified simulator makes randomised training credible. AnyCar (2024/25) showed in-context adaptation for agile wheeled robots. Neural-Fly supplies the evaluation template (L1, INDI, a no-learning adaptive ablation).

**(c) Nearest work.** AnyCar uses in-context learned dynamics plus MPPI on onboard-PC cars, against specialist learned baselines. Open: microcontroller-scale differential-drive robots, onboard-only sensing, and a comparison with classical adaptive and traction control. Neural-Fly uses a learned basis plus an adaptive law on quadrotors. RMA is legged.

**(d) Baseline and why it might fail.**
- **Baseline:** PI wheel loops on the 32U4 at ≥500 Hz, gyro heading, a per-surface acceleration limit, a slip detector (encoder versus IMU acceleration; gyro versus encoder yaw rate) and RLS μ on detected slip.
- **Why it might fail:**
  - μ is unobservable before slip, so a single a_max is either conservative or slips;
  - RLS needs slip excitation;
  - the IMU is noisy (sourced RSS warning).
- **The learned method's only structural edge** is faster or richer slip-onset detection from history. That claim must be measured, not assumed (this is why filter 4 is weak).

**(e) First-week test and the result that would kill it.** Run the classical stack on 3–4 surfaces with 0, +50 and +100 g payloads (centred and offset). Do full-PWM launches and 1 m stop-on-target moves, measuring slip, completion time and stopping error. **Kill if** either:
- there is no detectable slip at full PWM on any realistic surface (for example, because the regulator caps torque); or
- a single conservative a_max is within run-to-run noise of per-surface-tuned settings, i.e. there is no adaptation headroom.

**(f) Hardware and compute.** As C1, plus low-μ sheets and weights. An on-board policy on the C3 must be tiny: a [64, 64] MLP takes 6.6–8.7 ms on an ESP32-C3 (sourced), so use 16–32 units or fixed-point, or move to an ESP32-S3 (estimated).

**(g) Headline figure and outcomes.**
- **Headline:** completion time and stopping error across surface × payload, for learned, classical adaptive and fixed-conservative controllers; plus slip-detection latency.
- **Best realistic:** a tie at steady state, with the learned controller ahead only on abrupt surface transitions (estimated).
- **Floor:** an adaptation-headroom map showing classical suffices, which is still a useful measured boundary.

**(h) Filters.**

| Filter | Verdict | Reason |
|---|---|---|
| 1. Robotics first | pass | the robot's traction physics is the subject |
| 2. Baseline named | pass | strong named classical baseline |
| 3. Gap is real | **conditional** | fails unless week one shows slip and headroom |
| 4. Method justified | weak | the identifiability argument limits learning to faster detection |
| 5. Nearest work read | partial | AnyCar, Neural-Fly, RMA named; primary texts not read |
| 6. Feasible | pass | only with a tiny on-board policy |
| 7. Week-one test | pass | kill test defined |
| 8. Useful if learning loses | pass | a headroom map is the product |

**(i) Skills.** Traction control, estimation, RL with domain randomisation, embedded deployment.

**(j) People.** Paul O'Dowd, as above.

#### Explicit verdict on "adaptive motion control under wheel, surface or payload changes" (general form): fails filters 2 and 3
**Reasons:**
- motor voltage is regulated at 8 V ([product 3737](https://www.pololu.com/product/3737); [RSS SL0](https://github.com/paulodowd/SEMTM0042_43/blob/main/Labsheets/Supp/SL0_MeasuringBatteryVoltage.ipynb));
- table→carpet changes moderate-speed wheel speed by 3–7% ([RSS data](https://raw.githubusercontent.com/paulodowd/SEMTM0042_43/refs/heads/main/ExampleData/surfaces.csv)), which integral action absorbs;
- payload and COM effects are linear-in-parameter (RLS/MRAC);
- wheel-diameter change is odometry calibration.

**Week-one measurement that would show classical control already suffices:**
1. Implement per-wheel PI + feedforward with dead-band compensation on the 32U4, plus a gyro heading loop.
2. Measure on table, carpet, paper and low-μ sheet, crossed with 0, +50 and +100 g payload (centred and with a 30 mm lateral offset), crossed with stock and taped (diameter-changed) wheel:
   - step-response settling time
   - wheel-speed RMSE
   - heading drift over 2 m
   - figure-8 tracking error (webcam)
3. If the worst-case degradation against nominal is within run-to-run spread (or below a pre-registered threshold, e.g. 20%), classical suffices and the idea is dead, except at the traction limit (C2).

#### Tempting but fails (with the failing filter)
1. **Battery-state adaptation:** fails filter 3 (regulated 8 V motor rail).
2. **Learned wheel-speed control for surfaces and payloads at normal speeds:** fails filters 2 and 3 (3–7% plant change; PI absorbs it).
3. **Learned wheel-diameter or odometry adaptation:** fails filter 2. UMBmark-style calibration, gyro fusion and online RLS are strong baselines (prior knowledge); this overlaps the state-estimation strand.
4. **RL time-optimal line following:** fails filters 2 and 3 (Pololu PD at 50%; sensing and preview limit; classical speed scheduling plus ILC/lap memory). No RL-versus-PID literature was verified.
5. **Waypoint racing with webcam or ArUco tracking:** fails filter 3. A 10–30 Hz external pose makes the problem latency-dominated, a gap created by the instrumentation. With on-board odometry it reduces to C1/C2.
6. **Off-board WiFi learned control versus on-board control:** fails filters 2 and 4 (on-board PI wins at wheel level; no sourced WiFi figures).
7. **"Tiny policy on an MCU" as the research question:** fails filters 1 and 4 (an embedded benchmark; RLtools already reports MCU inference).
8. **Differentiable-simulation policy optimisation for the 3pi+:** fails filter 4 (method-driven; MPC on the same model is the baseline). Gradient-based system ID can still serve as a tool inside C1.
9. **A GPU-parallel simulator (Isaac Lab, Genesis, MJWarp) as the contribution:** fails filters 4 and 6 in spirit (a planar 2-wheel robot needs only a vmap'd JAX model).
10. **TailRL/MaxRL risk-sensitive objectives for robust sim-to-real:** fails filters 4 and 5. Prior knowledge: EPOpt-style CVaR ensembles (ICLR 2017) cover this, and the earlier tests showed small objective effects. Revisit only if C1 shows heavy-tailed transfer failures.
11. **Predicting friction from the downward IR reflectance:** likely fails filters 2 and 3 (a lookup table beats learning with few surfaces; reflectance is not causally tied to μ). Kill test: correlation across ≥8 surfaces.
12. **A fleet meta-learned dynamics model across many 3pi+ robots:** fails filter 2 as a learning question (a 1–2 minute automated per-robot ID probably wins). Measuring unit-to-unit variance is still a good C1 add-on if robots can be borrowed.

### Gaps
**Search incompleteness.** The shared web-search budget ran out before I could search for:
- classical adaptive control on differential-drive robots
- F1TENTH RL sim-to-real 2024–2026
- RL-versus-PID line followers
- Duckietown sim-to-real results
- ESP32 WiFi latency
- independent Genesis benchmarks
- other BRL staff

"No overlap found" claims above are therefore weak.

**Source access log** (reading depth):
- **Read in full or in the relevant parts (primary, GitHub raw or PyPI):**
  - Pololu 3pi+ Arduino library (README, Motors, IMU, Line/Bump sensors, Encoders, LineFollowerPID, RotationResist)
  - RSS SEMTM0042_43 README and labsheets L1, L3, SL0, SL1, SL3, SL7 (relevant sections), plus the example CSVs (fully analysed)
  - RSS 2026 StampC3 template headers
  - ProcessingTrackingVisualiser README
  - AnyCar READMEs (4); ASAP README (partial); Neural-Fly README; DATT README
  - RLtools README and inference table; TinyMPC README
  - MuJoCo changelog (2026 entries), XMLreference dcmotor section, MJX parity table, sysid README, Menagerie README
  - MuJoCo Playground and MJWarp READMEs
  - Isaac Lab actuator source and docs
  - Genesis README; genesis-speed-benchmark README
  - PyPI metadata
- **Abstract or search extract only:** Pololu product, spec and user's-guide pages; AnyCar abstract; Song et al. 2023 abstract; RMA (via A-RMA description); ASAP secondary summaries (alphaxiv, swiftscholar, emergentmind); arXiv 2608.01506; arXiv 2505.15589.
- **Could not access:**
  - all pololu.com pages, PDFs and the schematic
  - arxiv.org and every paper's full text
  - science.org, IEEE Xplore, openreview, the RSS proceedings site
  - the AnyCar and ASAP project pages (github.io)
  - Genesis' Notion benchmark report
  - bristol.ac.uk
  - CMU MSR theses by Tairan He (2025) and Tony Tao (Dec 2025), which may contain fuller limitations sections for ASAP and AnyCar
