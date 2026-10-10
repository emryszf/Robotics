# State estimation and active localisation from cheap sensors on a Pololu 3pi+ 32U4: learned and calibrated odometry, active localisation, magnetic localisation

Research date: Saturday 10 October 2026.

**Access and read-depth note (applies to the whole file).**
- **Blocked sources.** This session's egress policy blocked arXiv, IEEE Xplore, HAL, PMC/MDPI, OpenAlex, Semantic Scholar, pololu.com and st.com. WebFetch also failed DNS resolution. As a result, **no paper was read in full or at methods-section level directly**. Three vendor documents are the exception: the ST LSM6DS33 and LIS3MDL datasheets and a saved copy of Pololu's 3pi+ 2040 User's Guide, all read directly from copies committed to third-party GitHub repos.
- **Read-depth labels.** Findings come from four kinds of evidence, each marked:
  - **[SX]**: a search-engine extract of an abstract or page. This is abstract-level, so treat its details as provisional.
  - **[GH-full]**: a primary GitHub file (README or source code) read in full via raw.githubusercontent.com.
  - **[GH-copy]**: a vendor document (datasheet or user's guide) read directly from a copy committed to a third-party GitHub repo. The content is the vendor's, but the copy was not checked against the vendor's site.
  - **"estimated" / "inference"**: my own reasoning.
- **Search budget.** The shared web-search budget (200 per turn across all agents) ran out partway through. The planned searches I could not run are listed under Gaps.
- **Labels.** PREPRINT means arXiv-only as far as this session could verify.

## 1. How large is dead-reckoning error on a 3pi+-class robot after classical calibration, which error sources dominate, and what are the 3pi+ 32U4's IMU and magnetometer parts and datasheet figures?

### Takeaway
- **Sensor parts.** The 3pi+ 32U4 carries an ST **LSM6DS33** (accelerometer and gyro) plus an ST **LIS3MDL** magnetometer, not an LSM6DSO. Pololu's own library recognises only this pair. The LSM6DSO is on the RP2040-based 3pi+ 2040, which shares the chassis.
- **Datasheet figures (read directly).**
  - LSM6DS33: gyro noise 7 mdps/√Hz; typical zero-rate level ±10 dps; zero-rate drift ±0.05 dps/°C; accelerometer noise 90 µg/√Hz.
  - LIS3MDL: RMS noise 3.2/3.2/4.1 mgauss, specified at ±12 gauss in ultra-high-performance mode; typical zero-gauss offset ±1 gauss (100 µT).
- **No published 3pi+ figures.** No sourced post-calibration dead-reckoning error exists for the 3pi+, e-puck or Romi class of robot, so the student must measure it.
- **Likely dominant error (estimated).** Heading drift from residual gyro bias and temperature, plus effective track-width error in turns, should dominate. Gyro white noise and encoder quantisation should not.

### Cited Findings
- **IMU parts (Pololu docs).** The 3pi+ 32U4's inertial sensors are an ST LSM6DS33 (3-axis accelerometer + 3-axis gyro) and an ST LIS3MDL 3-axis magnetometer. They sit on the default I²C lines behind 3.3 V↔5 V level shifters, and they are the same ICs as Pololu's MinIMU-9 v5. Search found no 3pi+ 32U4 variant with an LSM6DSO — [Pololu 3pi+ 32U4 User's Guide §5.6 [SX]](https://www.pololu.com/docs/0J83/5.6)
- **IMU parts (Pololu library source).** Pololu's official 3pi+ 32U4 Arduino library (latest release 1.1.3, dated 2022-09-06) detects only `IMUType::LSM6DS33_LIS3MDL`:
  - It checks WHO_AM_I = 0x69 at LSM6DS33 address 0b1101011 (0x6B), and WHO_AM_I = 0x3D at LIS3MDL address 0b0011110 (0x1E).
  - There is no LSM6DSO code path.
  - Sources: [Pololu3piPlus32U4IMU.cpp [GH-full]](https://github.com/pololu/pololu-3pi-plus-32u4-arduino-library/blob/master/src/Pololu3piPlus32U4IMU.cpp); [Pololu3piPlus32U4IMU_declaration.h [GH-full]](https://github.com/pololu/pololu-3pi-plus-32u4-arduino-library/blob/master/src/Pololu3piPlus32U4IMU_declaration.h); [library README / version history [GH-full]](https://github.com/pololu/pololu-3pi-plus-32u4-arduino-library/blob/master/README.md)
- **Library default sensor settings** — [Pololu3piPlus32U4IMU.cpp [GH-full]](https://github.com/pololu/pololu-3pi-plus-32u4-arduino-library/blob/master/src/Pololu3piPlus32U4IMU.cpp):
  - Accelerometer: 52 Hz, ±2 g.
  - Gyro: 208 Hz, ±245 dps.
  - Magnetometer: ultra-high-performance mode on X, Y and Z; 10 Hz output data rate; ±4 gauss; continuous conversion.
  - `configureForTurnSensing()` switches the gyro to 833 Hz, ±2000 dps.
- **Pololu's magnetometer warning.** "The magnetometer on the 3pi+ 32U4 can be affected by magnetic fields from the 3pi+ itself." Named sources are magnets in the motors and encoders, electrical currents through the board, and hard-iron distortion from metal — [User's Guide §5.6 [SX]](https://www.pololu.com/docs/0J83/5.6)
- **Encoder resolution.** 12 counts per motor-shaft revolution, counting both edges of both channels. The 30:1 motors are more precisely 29.86:1, so 29.86 × 12 ≈ 358.3 counts per wheel revolution — [User's Guide §5.4 [SX]](https://www.pololu.com/docs/0J83/5.4)
- **Encoder counter width.** Counts are signed 16-bit and wrap at +32767 / −32768 — [Pololu3piPlus32U4Encoders.h [GH-full]](https://github.com/pololu/pololu-3pi-plus-32u4-arduino-library/blob/master/src/Pololu3piPlus32U4Encoders.h)
- **Line sensors.** Five downward reflectance sensors; `readLineBlack()` returns 0–4000 from left to right — [Pololu3piPlus32U4LineSensors.h [GH-full]](https://github.com/pololu/pololu-3pi-plus-32u4-arduino-library/blob/master/src/Pololu3piPlus32U4LineSensors.h)
- **Bump sensors.** Two front bump sensors (left and right), each triggered by a baseline-plus-margin threshold — [Pololu3piPlus32U4BumpSensors.h [GH-full]](https://github.com/pololu/pololu-3pi-plus-32u4-arduino-library/blob/master/src/Pololu3piPlus32U4BumpSensors.h)
- **LSM6DS33 datasheet, Table 3 "Mechanical characteristics"** (DocID027423 Rev 4, October 2015; at Vdd 1.8 V, 25 °C; typical values, "not guaranteed"; offsets are "values after soldering") — [LSM6DS33 datasheet, copy committed in Stanford's cs240lx repo [GH-copy]](https://github.com/dddrrreee/cs240lx-20spr/blob/master/labs/8-i2c-accel/docs/LSM6DS33.pdf):
  - **Gyro:**
    - rate noise density Rn = 7 mdps/√Hz;
    - typical zero-rate level G_TyOff = ±10 dps;
    - zero-rate change versus temperature G_OffDr = ±0.05 dps/°C;
    - sensitivity change versus temperature G_SoDr = ±1.5 % (−40 to +85 °C, relative to 25 °C);
    - sensitivity 8.75 mdps/LSB at ±245 dps and 70 mdps/LSB at ±2000 dps.
  - **Accelerometer:**
    - noise density An = 90 µg/√Hz (FS ±2 g, ODR 104 Hz);
    - zero-g offset LA_TyOff = ±40 mg, changing by ±0.5 mg/°C;
    - sensitivity change ±1 %;
    - sensitivity 0.061 mg/LSB at ±2 g.
  - **Not in Table 3:** a bias-instability (Allan) figure, and a room-temperature gyro sensitivity tolerance.
  - The same Rn and An figures appear in [Mouser's LSM6DS3/LSM6DS33 release [SX]](https://www.mouser.com/en/publicrelations_st_lsm6ds3_3d_accelerometer_gyroscope_2015final/).
- **The LSM6DSO is on the 3pi+ 2040, not the 32U4.** Pololu's 3pi+ 2040 User's Guide: "The first chip, an ST LSM6DSO, combines a 3-axis accelerometer and 3-axis gyro … The second chip is an ST LIS3MDL 3-axis magnetometer" — [saved copy of Pololu's 3pi+ 2040 User's Guide in a third-party repo [GH-copy]](https://github.com/FREECPP/Bachelor-Thesis/blob/main/Skripte/00_Weitergabe_Micromouse/doku/Pololu_docs.htm)
- **Shared 3pi+ chassis, from the same guide** — [saved 3pi+ 2040 User's Guide [GH-copy]](https://github.com/FREECPP/Bachelor-Thesis/blob/main/Skripte/00_Weitergabe_Micromouse/doku/Pololu_docs.htm):
  - **Parts:** the 3pi+ Chassis Kit includes "two 32×7 mm Pololu wheel hubs and silicone tires", a 4×AAA battery holder and a ball caster. The robot is 9.7 cm in diameter.
  - **Evidence the chassis is shared:** the guide says the 3pi+ 32U4 OLED assembly instructions apply to the 2040 because "the physical differences do not affect the assembly process".
  - **3pi+ 2040 top speeds by edition:**
    - Standard (30:1 MP 6V): 1.5 m/s;
    - Turtle (75:1 LP): 0.4 m/s;
    - Hyper (15:1 HPCB): about 4 m/s.

    These are 2040 figures; the 32U4 Standard Edition's top speed was not retrieved.
  - **Battery sag:** a dropping battery voltage can make the robot "accelerate more slowly or even be unable to reach as high of a top speed".
- **LIS3MDL datasheet, Table 3** (DocID024204 Rev 6, May 2017; at Vdd 2.5 V, 25 °C; typical values "not guaranteed"; factory calibrated at 2.5 V) — [LIS3MDL datasheet, copy committed in a third-party repo [GH-copy]](https://github.com/Arunvishnu123/iot-project/blob/master/docs/lis3mdl.pdf):
  - **Noise:** RMS noise is 3.2 mgauss (X), 3.2 mgauss (Y) and 4.1 mgauss (Z), specified at **FS ±12 gauss in ultra-high-performance mode**. Noise at ±4 gauss, the Pololu library default, is not given.
  - **Sensitivity:** 6842 LSB/gauss at ±4 gauss.
  - **Offset:** typical zero-gauss level ±1 gauss at FS ±4 gauss.
  - **Other:** non-linearity ±0.12 %FS. The zero-gauss offset "starts to degrade" after exposure to a 50 gauss disturbance field.
  - **Output data rate:** up to 80 Hz normally. With FAST_ODR it reaches 155 Hz in ultra-high-performance mode and 560 Hz in medium-performance mode.
- **This resolves an earlier conflict between secondary sources.**
  - An ST moderator (2023) quoted 3.2/4.1 mgauss without a condition — [ST Community [SX]](https://community.st.com/t5/mems-sensors/lis3mdl-noise-and-maximum-accuracy/m-p/595789).
  - element14 gave "@FS ±12 gauss" — [element14 [SX]](https://community.element14.com/products/manufacturers/st/w/documents/23712/stmicroelectronics-lis3mdl-3-axis-standalone-digital-magnetometer-complete-range-of-sensors-delivering-ultimate-design-flexibility). The datasheet confirms element14's condition.
  - Separately, a 2019 user measured LIS3MDL noise at about 2× an LSM303D per axis, and ST staff suggested the LIS2MDL (3 mgauss RMS) — [ST Community 2019 [SX]](https://community.st.com/t5/mems-sensors/the-rms-noise-of-lis3mdl-and-lsm303d/m-p/401563).
- **LIS3MDL lifecycle.** ST's product page lists the LIS3MDL as obsolete or NRND (extracts disagree on which) and points to the LIS2MDL — [ST LIS3MDL page [SX]](https://st.com/en/mems-and-sensors/lis3mdl.html)
- **UMBmark method.** Borenstein & Feng, IEEE Trans. Robotics & Automation 12(6):869–880, 1996. The robot drives a square clockwise and counter-clockwise, which separates the two dominant systematic differential-drive errors — [paper copy [SX]](https://cs.au.dk/~ocaprani/legolab/DigitalControl.dir/NXT/Lesson10.dir/paper58.pdf):
  - Unequal wheel diameters, which curve the path.
  - Effective wheelbase error, which causes rotation errors.
- **UMBmark's claimed gain.** "One order of magnitude" improvement is a secondary claim from an MDPI article, not confirmed in the original — [MDPI Applied Sciences [SX]](https://www.mdpi.com/2076-3417/12/5/2606)
- **UMBmark limitations.**
  - It is relatively difficult to run, very sensitive to non-systematic errors, and needs a relatively large test space — [UTP review (Spanish) [SX]](https://ojs2.utp.edu.co/index.php/revistaciencia/article/download/4035/2291/2871).
  - Approximation errors in the calibration equations, and coupling between the two error sources, limit accuracy — [SAGE IJARS [SX]](https://journals.sagepub.com/doi/pdf/10.5772/45712).
- **Calibration survey.** Sousa, Petry & Moreira (2020) review odometry calibration methods since UMBmark, including fifteen for differential drive — [INESC TEC repository [SX]](https://repositorio.inesctec.pt/handle/123456789/13699)

### Inferences
- **Gyro white noise is negligible (estimated from the sourced Rn).** 7 mdps/√Hz gives an angle random walk of 0.007 °/√s, or about 0.42 °/√h. That is about 0.05° of heading after 60 s and 0.17° after 10 min, assuming high-performance mode and white noise only.
- **Residual bias and temperature drift likely dominate (estimated, from the DS33's own Table 3).**
  - Uncalibrated, the typical ±10 dps zero-rate level would integrate to about 600° of heading per minute, so at-rest bias calibration is mandatory.
  - With ±0.05 dps/°C, a 5 °C board warm-up after an at-rest bias calibration leaves about 0.25 °/s of bias, which is about 15° of heading per minute if uncompensated.
  - Gyro scale error (±1.5 % sensitivity change over the full temperature range; room-temperature tolerance not given) matters mainly in spins: 1 % on a 360° turn is 3.6°.
  - Lateral error is roughly d·δθ. For example, 1° of heading error over 1 m gives about 17 mm.
  - So slowly varying bias, not sensor noise, sets the error budget.
- **The platform gives a free classical fix (inference).** The 3pi+ knows from its encoders when it is stationary, so it can re-estimate gyro bias at every stop (a zero-angular-rate update). This is a strong classical baseline against any learned gyro-correction method.
- **Encoder quantisation is negligible (estimated).** With the 32 mm wheel sourced above, one count is about 0.28 mm (π × 32 mm / 358.3). That is small compared with systematic errors.
- **Counter wrap (estimated).** The 16-bit counter wraps after about 91 wheel revolutions, roughly 9.2 m with 32 mm wheels, so firmware must log count deltas.
- **Magnetometer offset needs calibration first (inference).** The LIS3MDL's typical ±1 gauss (100 µT) zero-gauss offset, before adding the robot's own hard-iron field, is as large as or larger than the geomagnetic field. (The geomagnetic magnitude is general knowledge, not sourced in this session.) Offset and hard-iron calibration therefore come before any magnetometer heading or map use.
- **Silicone tyres (inference).** The sourced silicone tyres plus the 9.7 cm body suggest a small effective track width with a tyre contact patch that is not negligible relative to it. That makes encoder-derived heading in pivots especially sensitive to scrub; measure it rather than assume.
- **Heading during turns (inference).** Encoder heading depends on the effective track width, which is uncertain for a short-wheelbase robot with rubber tyres scrubbing during pivots. The classical split is gyro for heading and encoders for distance; Borenstein & Feng's "gyrodometry" (ICRA 1996) was not verified in this session.
- **Likely error ranking on a smooth table (inference, needs measurement):**
  1. Heading drift: gyro bias versus temperature, or track-width and turn scrub if heading comes from encoders.
  2. Wheel-diameter mismatch, which UMBmark can calibrate out.
  3. Slip during hard acceleration or on dusty or glossy surfaces.
  4. Bumps, which are mostly irrelevant on a tabletop.
- **Implication for learned methods (inference).** Because datasheet noise is not the bottleneck, a learned method has value only on **non-stationary** effects that classical calibration cannot capture: temperature, surface or payload changes, and slip events.
- **Ground truth (feasibility).** Every learned or calibrated odometry topic needs ground truth. The student's USB webcam, mounted overhead with printed fiducials, is the realistic substitute for motion capture. Its accuracy was not sourced and must be measured against a ruler grid in week 1.

### Gaps
- **No 3pi+-class dead-reckoning figures.** I found no sourced post-calibration dead-reckoning error (mm, or % of distance) for 3pi+, e-puck, Romi or similar robots. The searches returned only method papers. The student's own UMBmark run will be the primary number.
- **LSM6DS33 gaps.** The datasheet gives no bias-instability (Allan) figure and no room-temperature gyro sensitivity tolerance. Measure both: Allan variance from a long static log, and scale factor from N spins over a radial tape line.
- **LIS3MDL gaps.** Noise at the library's ±4 gauss setting, and the zero-gauss offset's temperature drift, are not in Table 3.
- **3pi+ geometry and speed.** Track width (wheel separation) was not found. Wheel size comes from the 3pi+ 2040 guide's shared-chassis kit list. The 32U4 Standard Edition's top speed was not retrieved; only the 2040's 1.5 m/s was.
- **Board revision.** The student should read WHO_AM_I on their own board; 0x69 means LSM6DS33. The LSM6DSO's WHO_AM_I value was not verified here.
- **Gyrodometry.** Borenstein & Feng's ICRA 1996 gyrodometry paper was not retrieved because the search budget ran out.

## 2. What do the strongest learned wheel/inertial odometry and learned noise/covariance methods do at methods level, and what do they leave open for low-cost, low-speed ground robots?

### Takeaway
- **Where the strong results come from.** The best learned methods were shown on cars (KITTI), drones, pedestrians or headsets, or in simulation. Most use better sensors (for example a fibre-optic gyro) or solve IMU-only problems.
- **No small-robot evidence.** None was found on a small educational differential-drive robot with a consumer MEMS IMU and wheel encoders at low speed.
- **What transfers.** The useful idea is learned, input-dependent noise covariances inside a physically structured filter (AI-IMU, TLIO, WING). End-to-end learned odometry does not transfer: the 3pi+ already measures velocity with its encoders.

### Cited Findings
- **Brossard & Bonnabel, ICRA 2019 (Montreal)** — [HAL hal-01874593 [SX]](https://hal.archives-ouvertes.fr/hal-01874593):
  - Gaussian processes are trained on the residuals between a nominal model and ground truth.
  - An EKF propagates with wheel speeds plus the gyro, and updates with the IMU.
  - Sensors: two wheel encoders, a **fibre-optic gyro**, and an IMU.
  - The learned, corrected models beat the originals.
- **AI-IMU Dead-Reckoning, IEEE Trans. Intelligent Vehicles, 2020 (IEEE doc 9035481)** — [ai-imu-dr README [GH-full]](https://github.com/mbrossar/ai-imu-dr); [arXiv 1904.06064 [SX]](https://arxiv.org/pdf/1904.06064):
  - The filter uses zero lateral and vertical velocity as pseudo-measurements.
  - A deep "noise parameter adapter" maps raw IMU signals directly to those measurements' covariance matrices, "without requiring knowledge of any state estimate nor any other quantity".
  - It reports 1.10 % translational error on KITTI using only an IMU.
  - Methods-level limitation, quoted from the README: the released parameters "are trained on sequences 00, 01, 04-11, so only sequence 02 is a test sequence".
- **Gyro denoising, IEEE RA-L 2020 (received 24 Feb 2020, accepted 8 Jun 2020; IEEE doc 9119813)** — [denoise-imu-gyro README [GH-full]](https://github.com/mbrossar/denoise-imu-gyro); [arXiv 2002.10718 [SX]](https://arxiv.org/pdf/2002.10718):
  - A dilated-convolution network, with no recurrent layers, computes gyro corrections from past IMU samples. It is trained on ground truth with a loss on orientation increments, then the corrected signal is integrated open-loop.
  - On EuRoC and TUM-VI it "beat[s] top-ranked visual-inertial odometry systems" on attitude.
  - Training "roughly takes 5 minutes per dataset with a decent GPU".
- **TLIO** — [arXiv 2007.01867 [SX]](https://arxiv.org/pdf/2007.01867); [TLIO README [GH-full]](https://github.com/CathIAS/TLIO). The RA-L 2020 venue is as given in the brief; neither the extract nor the README states it.
  - A network regresses 3-D displacement plus its uncertainty. A stochastic-cloning EKF fuses it to estimate pose, velocity and biases.
  - The authors report statistically consistent network uncertainty.
  - Constant-covariance ablations fail in some cases.
  - Training data are pedestrian headset recordings.
  - The README warns the code "requires the user to generate its own dataset and retrain".
- **RoNIN, ICRA 2020 (IEEE doc 9196860).** Smartphone pedestrian inertial navigation. From the README: "due to security concerns we were unable to publish 50% of our dataset" — [RoNIN README [GH-full]](https://github.com/Sachini/ronin)
- **AirIMU, "Learning Uncertainty Propagation for Inertial Odometry" (arXiv 2310.04874; README gives no venue, so PREPRINT as far as verified).** Evaluated on EuRoC, TUM-VI, KITTI and SubT-MRS — [AirIMU README [GH-full]](https://github.com/haleqiu/AirIMU)
- **AirIO, IEEE RA-L (accepted May 2025; arXiv 2501.15659)** — [Air-IO README [GH-full]](https://github.com/Air-IO/Air-IO); [arXiv 2501.15659 [SX]](https://arxiv.org/html/2501.15659v1):
  - Argues that converting UAV IMU data to a global frame "undermines the observability" of kinematics.
  - A body-frame representation that keeps gravity improves ATE by 73.9 % on Blackbird.
  - Gains average 66.7 % across three datasets, plus 23.8 % more from encoding attitude explicitly.
  - Its EKF needs AirIMU preintegration and uncertainty, and the README notes a "conventional mode that uses ground truth orientation as input".
  - The AirIMU README states a different headline, "up to 86.6% performance boost over SOTA" — [AirIMU README [GH-full]](https://github.com/haleqiu/AirIMU)
- **Tartan IMU, CVPR 2025 (pp. 22520–22529)** — [CVF open access [SX]](https://openaccess.thecvf.com/content/CVPR2025/html/Zhao_Tartan_IMU_A_Light_Foundation_Model_for_Inertial_Positioning_in_CVPR_2025_paper.html):
  - Argues that learned IMU odometry is dataset-specific and overfits.
  - Pretrained on more than 100 h of multi-platform data, with a 36 % ATE improvement over specialised models.
  - LoRA adaptation needs 1.1 M trainable parameters; online test-time adaptation runs at 200 FPS.
  - The Hugging Face card links the model to an IROS 2026 challenge (unverified) — [Hugging Face [SX]](https://huggingface.co/Tartan-IMU/TartanIMU)
- **KalmanNet family.**
  - The original repo links arXiv 2107.10043, "Neural Network Aided Kalman Filtering for Partially Known Dynamics". The code covers simulated linear and Lorenz-attractor cases — [KalmanNet_TSP README [GH-full]](https://github.com/KalmanNet/KalmanNet_TSP).
  - Uncertainty extension (ICASSP 2022): KalmanNet's learned gain yields an error covariance. It is KF-like when dynamics are known and more accurate under evolution-model mismatch — [arXiv 2110.04738 [SX]](https://arxiv.org/pdf/2110.04738).
  - Bayesian KalmanNet (IEEE TSP 2025 per the extract): Monte-Carlo sampling of J KalmanNet realisations (concrete dropout), with a sample covariance. The authors note "there is no general guarantee" that each estimator is a realisation of an unbiased estimator — [arXiv 2309.03058 [SX]](https://arxiv.org/pdf/2309.03058).
  - Mortada et al. (PREPRINT, April 2026): predict the Cholesky factor of the noise-dependent covariance term, with a NEES plus log-determinant loss to stop the covariance blowing up — [arXiv 2604.02441 [SX]](https://arxiv.org/pdf/2604.02441).
- **WING (PREPRINT, July 2024)** — [arXiv 2407.10101 [SX]](https://arxiv.org/pdf/2407.10101):
  - Neural networks correct IMU and wheel-encoder measurements "while considering their uncertainty".
  - A sliding-window filter uses the time-varying predicted uncertainty, with ground-manifold constraints.
  - It is the closest learned wheel-inertial method with learned covariances.
- **Adjacent wheel-odometry learning.**
  - Online attention-based wheel-odometry correction (Navone et al., PREPRINT 2023) — [arXiv 2303.11725 [SX]](https://arxiv.org/abs/2303.11725v1).
  - An online-learned kinematic model inside a LiDAR-IMU-wheel factor graph for skid-steer robots (Okawara et al., 2024) — [arXiv 2407.08907 [SX]](https://arxiv.org/abs/2407.08907v1).
  - SlipOdometryNet: a 1-D CNN from encoders and commanded velocities, with no IMU, for skid-steer robots (Research Square preprint, date unconfirmed) — [Research Square [SX]](https://www.researchsquare.com/article/rs-9959755).
  - An LSTM on encoder plus IMU data for slippery ground (KIST, 2023) — [KIST [SX]](https://pubs.kist.re.kr/handle/201004/113109).
- **Adaptive Kalman-Informed Transformer (Cohen & Klein, Engineering Applications of AI, 2025).** Learns the EKF process-noise covariance online. In an AUV case study, the authors report more than 49.5 % better than an EKF and 35.4 % better than a model-based adaptive EKF — [arXiv 2401.09987 [SX]](https://www.arxiv.org/abs/2401.09987)

### Inferences
- **The mechanism that transfers (inference).** AI-IMU learns the covariance of pseudo-measurements that come from motion constraints. On a differential-drive robot the analogues are no-lateral-slip and encoder-consistent forward speed. A learned adapter that inflates these covariances when slip is likely keeps the physics in the filter and lets learning set only the trust level. That is a structural, not fashionable, justification (filter 4).
- **The IMU-only line is a manufactured gap here (inference).** RoNIN, TLIO, AirIO and Tartan IMU solve IMU-only problems. The 3pi+ has a direct velocity sensor (encoders) and stops often, so transplanting them fails filter 3.
- **KalmanNet's premise does not hold (inference).** KalmanNet targets partially known dynamics. Unicycle kinematics are known; what is unknown is non-stationary noise from slip, surface and temperature.
- **What remains open for cheap, slow ground robots (inference from extracts; must be checked in full text):**
  - No evidence was found that learned covariances stay calibrated under surface or payload shift. TLIO's consistency claim appears to be in-distribution only.
  - Evaluations report trajectory error (ATE/RTE), not decision-relevant calibration.
  - The baselines named in the extracts are fixed-covariance filters. None mentioned adaptive classical filters, such as innovation-based adaptive EKF, χ² gating, or encoder-versus-gyro yaw-rate disagreement slip detectors. That gap is what a 3pi+ study should test.
  - The 3pi+ can produce labelled data cheaply with overhead-camera ground truth, which suits small-data learned-covariance work.
- **Compute is not a constraint (estimated).** Brossard's gyro denoiser trains in about 5 min per dataset on a GPU (sourced). 3pi+-scale models will be far smaller than anything that strains a 12 GB RTX 3060.

### Gaps
- **No limitations sections read.** None of these papers' limitations or future-work sections could be read directly; the only direct quotes come from READMEs.
- **Venues not re-verified.** TLIO (RA-L 2020) and KalmanNet (IEEE TSP 2022) were not re-verified, and AirIMU's venue is unknown.
- **No 2025–2026 small-robot slip work found.** No paper learning slip detection plus covariances for wheel-inertial fusion on small robots turned up; the search was incomplete because the budget ran out.
- **Baselines unchecked.** Whether AI-IMU or WING compare against adaptive classical filters is unknown.

## 3. Calibrated uncertainty: classical consistency (NEES/NIS), conformal prediction and adaptive conformal inference in state estimation and navigation, and whether calibrated learned odometry uncertainty has driven a decision on real hardware

### Takeaway
- **What exists.** Conformal methods for robot pose and odometry exist: conformal visual odometry (2023), CLAPS (December 2025, with a real MBot) and adaptive geodesic CP (2026). ACI has also been combined with particle filtering (2024).
- **What was not found.** No work used calibrated learned or conformal odometry uncertainty to **trigger re-localisation on real hardware**.
- **The structural opening.** On a robot that re-localises at known lines, each re-localisation reveals the realised error. That gives a sparse, delayed, partial label stream whose timing depends on the robot's own decisions. This is exactly where online recalibration is needed, and where ACI-style guarantees are untested.

### Cited Findings
- **Conformal visual odometry (Stutts et al., 2023).** Produces prediction intervals across all degrees of freedom — [arXiv 2303.02207 [SX]](https://www.arxiv.org/abs/2303.02207). The multimodal follow-up (Parente et al.) handles disjoint uncertainty regions under ambiguity or occlusion — [arXiv 2309.11018 [SX]](https://export.arxiv.org/pdf/2309.11018)
- **CLAPS (Marques, Ghaffari & Berenson; PREPRINT, December 2025)** — [arXiv 2512.10294 [SX]](https://arxiv.org/html/2512.10294v1):
  - Conformal prediction sets on Lie groups (SE(2)) for the robot's configuration after an action.
  - Distribution-free coverage, with no assumed error distribution.
  - Experiments on a simulated JetBot and a real **MBot**, a small differential-drive robot.
  - It calibrates dynamics prediction regions; it does not trigger re-localisation.
- **Adaptive geodesic CP for egocentric camera pose (PREPRINT, May 2026).** Motivated by the claim that "visual odometry methods with heuristic confidence scores or learned covariances lack formal coverage guarantees" — [arXiv 2605.00233 [SX]](https://arxiv.org/pdf/2605.00233)
- **ACI by particle filtering under hidden Markov models (Su, Zhou & Luo; PREPRINT, November 2024)** — [arXiv 2411.01558 [SX]](https://arxiv.org/pdf/2411.01558):
  - Conformal inference for hidden states with no true-label calibration set, using weighted particles and an online-adapted miscoverage level.
  - In a target-localisation simulation, coverage was 0.897 against a 0.9 target, versus 0.877 for the non-adaptive version.
- **Probe sets for nonlinear Kalman filtering (PREPRINT, September 2026)** — [arXiv 2609.14271 [SX]](https://arxiv.org/pdf/2609.14271):
  - Conventional nonlinear filters give overconfident covariances when the local model varies across the uncertainty region.
  - Proposes covariance recalibration, evaluated on battery estimation and terrain-referenced navigation.
  - Runtime is 2.7–4.0× the conventional filter's.
- **Lindemann et al. survey, "Formal Verification and Control with Conformal Prediction: Practical Safety Guarantees for Autonomous Systems" (PREPRINT; v1 August 2024, v3 August 2025)** — [arXiv 2409.00536 [SX]](https://arxiv.org/pdf/2409.00536). A related CPS-VO tutorial by Lindemann and Deshmukh promises public slides and code — [CPS-VO [SX]](https://cps-vo.org/node/98826)
- **Time-aware motion planning with adaptive CP (Liang et al., L4DC 2026, PMLR v331).** Tightens safety margins in higher-uncertainty regions; experiments are numerical only — [PMLR [SX]](https://proceedings.mlr.press/v331/liang26a.html)
- **Conformal sound-source localisation (ICASSP 2025).** Motivated by robot audition; it has no hardware re-localisation trigger — [arXiv 2409.11804 [SX]](https://arxiv.org/html/2409.11804v2)
- **Classical consistency tests used for learned covariances.**
  - NEES used as a training loss (Mortada et al.) — [arXiv 2604.02441 [SX]](https://arxiv.org/pdf/2604.02441).
  - TLIO's claim of statistically consistent network uncertainty — [arXiv 2007.01867 [SX]](https://arxiv.org/pdf/2007.01867).

### Inferences
- **NEES and NIS are both available on a 3pi+ (inference).** NEES needs ground truth, which the overhead camera can supply offline. NIS needs only the filter's own innovation at each line crossing (a 1-D observation), so it can run online on the robot with no camera.
- **Standard conformal guarantees break here (inference).** Split-conformal guarantees assume exchangeability. Odometry errors are temporally correlated and drift with surface and temperature, so exchangeability fails.
- **ACI's feedback is the hard part (inference).** Gibbs & Candès-style ACI gives long-run coverage under shift, but it needs feedback. On the 3pi+, feedback arrives only at re-localisation events, and those carry three complications:
  1. **Sparse:** labels arrive only when the robot re-localises.
  2. **Decision-dependent:** the policy chooses when to look, which creates a selection effect.
  3. **Partial:** a line crossing reveals only the error component perpendicular to the line.

  No overlap was found for this setting. That is "no overlap found", not established novelty.
- **Miscalibration can masquerade as a bad decision rule (inference).** If the EKF is overconfident during slip, trace(P) under-reports the error. The robot then re-localises too late, misses the line's capture region, or fails the task. This is diagnosable: if empirical coverage at re-localisation events matches the nominal level, the rule is at fault; if it does not, the calibration is.

### Gaps
- **ACI primary source.** Gibbs & Candès (NeurIPS 2021) was not retrieved because the search budget ran out.
- **NEES/NIS references.** Classical sources (e.g., Bar-Shalom et al.) were not retrieved.
- **No hardware re-localisation trigger found.** No hardware paper using calibrated odometry uncertainty to trigger re-localisation turned up. The search was incomplete. CLAPS is the nearest hardware use, but it calibrates action-outcome sets.
- **CLAPS details unread.** What CLAPS learns, which MBot sensors it uses, and how it measures coverage were not read.

## 4. Active localisation (classical, belief-space, learned, and 2024–2026 "when to re-localise"): what is the strongest simple baseline, when does it fail, could failure come from miscalibration, and what physical re-localisation actions exist on a 3pi+?

### Takeaway
- **The idea is already framed, in simulation.** The nearest work to the student's idea already treats "when to spend effort on localising" as a chance-constrained POMDP or RL problem: "When to Localize? A POMDP Approach" (Williams, Torshizi & Tokekar; IEEE SSRR 2024) and RiskRL (Shek et al.; arXiv 2411.02788, v2 2025). Both work in simulation (RiskRL's code is grid worlds) with known models.
- **What is open.** Two things: (a) real re-localisation actions on hardware, which cost time and can fail; and (b) whether the belief that drives the decision is calibrated.
- **The baseline to beat.** A threshold on predicted uncertainty. Where uncertainty grows roughly monotonically between resets, it is plausibly near-optimal *if its input is calibrated* (inference, untested).

### Cited Findings
- **Fox, Burgard & Thrun, "Active Markov Localization for Mobile Robots", Robotics and Autonomous Systems 25(3):195–207, November 1998.** Gives principled criteria for where to move and where to point sensors. It is robust to noisy sensors and approximate world models, and was tested in a structured office — [CMU RI [SX]](https://www.ri.cmu.edu/publications/active-markov-localization-for-mobile-robots). The 1997 precursor is "Active Mobile Robot Localization by Entropy Minimization" — [CMU RI [SX]](https://www.ri.cmu.edu/publications/active-mobile-robot-localization-by-entropy-minimization)
- **Active Neural Localization (Chaplot, Parisotto & Salakhutdinov; ICLR 2018).** A fully differentiable network trained end-to-end with RL to localise accurately and efficiently. Real-robot evaluation was not confirmed by the extract — [ICLR 2018 [SX]](https://iclr.cc/virtual/2018/poster/319)
- **Deep Active Localization (Gottipati, Seo, …, Paull; IEEE RA-L 2019)** — [arXiv 1903.01669 [SX]](https://arxiv.org/abs/1903.01669?context=cs):
  - CNN perception plus a deep-RL planning module.
  - Trained in simulation and transferred to real hardware "with zero refinement".
  - Notes that ANL trains deep RL on the outputs of a fixed measurement-likelihood model.
- **Placed et al., "A Survey on Active SLAM", IEEE T-RO 39(3):1686–1705, 2023 (DOI 10.1109/TRO.2023.3248510)** — [arXiv 2207.00254 [SX]](https://arxiv.org/pdf/2207.00254); [Zaguan record [SX]](https://zaguan.unizar.es/record/126293):
  - Gives a unified formulation and a modular identify–select–execute pipeline, and covers belief-space planning, deep RL and multi-robot coordination.
  - Open problems, citing Cadena et al.: when to stop active SLAM, how to predict the effects of future actions, and the lack of optimality guarantees.
  - Calls for reproducible research.
- **"When to Localize? A POMDP Approach" (Williams, Torshizi & Tokekar; IEEE SSRR 2024; arXiv 2411.08281)** — [arXiv 2411.08281 [SX]](https://arxiv.org/abs/2411.08281); [RAAS Lab PDF [SX]](http://raaslab.org/pubs/williams2024when.pdf):
  - **Goal:** minimise localisation actions without exceeding a failure probability.
  - **Method:** a constrained POMDP solved with CC-POMCP, which "simulates failure probabilities to decide if a robot moves to its goal or localizes".
  - **Evaluation:** numerical experiments against multiple baselines.
  - **Conclusion:** cost-constrained planners offer "a precise mechanism for setting performance thresholds like failure probabilities … reducing arbitrary reward tuning".
  - **Stated limitations:** "high compute time", and the need for "well-defined transition and observation models". One version reports more than 20 min of inference for a 55-waypoint path.
- **RiskRL, "When to Localize? A Risk-Constrained Reinforcement Learning Approach" (Shek, Torshizi, Williams & Tokekar; arXiv 2411.02788)** — [arXiv 2411.02788 [SX]](https://arxiv.org/pdf/2411.02788); [UMD CS [SX]](https://www.cs.umd.edu/content/when-localize-risk-constrained-reinforcement-learning-approach); [when-to-localize-riskrl README [GH-full]](https://github.com/raaslab/when-to-localize-riskrl):
  - **Problem:** bi-criteria. Minimise localisation actions while keeping the probability of failure (collision, or not reaching the goal) bounded.
  - **Method:** a particle filter plus twin recurrent Soft Actor-Critic with risk constraints.
  - **Results changed between versions:** v1 reports at least 13 % better than baseline; v2 reports at least 26 % higher success rate in unseen test environments.
  - **Code:** the public code is a Gymnasium package of grid worlds, e.g. `W2LGridTunnel-v0`.
  - **Venue:** an extract mentions a July 2025 conference listing, but the venue is unverified.
- **GUIDE / G-SAC (arXiv 2410.15178).** Conditions navigation policies on task-specific, location-varying uncertainty requirements, tested on autonomous surface vehicles. It argues that fixed, environment-wide uncertainty thresholds are impractical when localisation is costly — [arXiv 2410.15178 [SX]](https://arxiv.org/html/2410.15178v3)
- **Localizability-aware deep-RL navigation (IET Cyber-Systems and Robotics; press coverage August 2025).** Uses RGB-D input and ORB-SLAM2 feedback, and argues against fixed penalty thresholds — [Nanowerk [SX]](https://www.nanowerk.com/news2/robotics/newsid=67508.php)
- **3pi+ re-localisation sensors.** Five downward reflectance sensors with a line position of 0–4000, and two front bump sensors — [LineSensors.h [GH-full]](https://github.com/pololu/pololu-3pi-plus-32u4-arduino-library/blob/master/src/Pololu3piPlus32U4LineSensors.h); [BumpSensors.h [GH-full]](https://github.com/pololu/pololu-3pi-plus-32u4-arduino-library/blob/master/src/Pololu3piPlus32U4BumpSensors.h)

### Inferences
- **Strongest simple baselines (inference):**
  1. A threshold on predicted position uncertainty — trace(P), or σ perpendicular to the nearest line — tuned on training runs.
  2. Periodic or distance-based re-localisation.
  3. A one-step expected-cost rule: re-localise when P(fail | continue) × cost_fail > cost_detour.

  Uncertainty grows roughly monotonically and resets at re-localisation, so the problem resembles optimal stopping or renewal. There, threshold-type policies are often near-optimal. This is an inference; no theorem was verified here. It is the student's own recurring failure mode — a ten-line rule beating learners — in a new setting.
- **When a threshold rule should fail on a 3pi+ (inference):**
  - **Miscalibrated σ:** the filter is overconfident during slip, surface transitions or temperature-driven gyro drift.
  - **Location-dependent re-localisation cost:** when the nearest line is far away, opportunistic re-localisation while passing close to a line beats a fixed threshold.
  - **Location-dependent failure risk:** narrow gaps or table edges, which is GUIDE's argument.
  - **Fallible actions:** if the error exceeds the line's capture region, the robot misses the line or associates with the wrong one. The threshold must stay below the capture limit.
  - **Partial fixes:** a line gives only a 1-D fix.

  The first of these is a calibration failure, not a decision-rule failure, and it is testable (see Candidate A).
- **Physical re-localisation actions on a 3pi+ (inference from the sensor set).** Each costs time or distance and can fail:
  - **Drive to a known tape line.** Gives a 1-D position fix, plus heading from the timing difference between the two outer line sensors crossing.
  - **Follow a line to a junction or marker.** Gives a 2-D fix.
  - **Square up against a wall** until both bump sensors are pressed. Gives a 1-D fix plus heading.
  - **Stop for a zero-rate gyro-bias update.** Improves heading only and costs time.
  - **Read magnetometer heading,** if self-interference is compensated (§5).
- **Infrastructure assumption (inference).** Sparse floor lines are a legitimate stand-in for real sparse-infrastructure robots (AGVs with floor tape, cleaning robots that bump walls). The overhead webcam must be used **only** for evaluation; otherwise relocalisation becomes pointless.
- **Simulation must be refitted (inference).** The student's existing kinematic simulator has no acceleration limits, slip or drift. Any simulator for this topic needs a noise model fitted to logged 3pi+ runs, with policies validated on the hardware. RiskRL's grid worlds do not model these physics.

### Gaps
- **Belief-space classics.** Prentice & Roy (belief roadmap) and Platt et al. (belief-space planning with maximum-likelihood observations) were not retrieved because the search budget ran out.
- **When-to-Localize internals unread.** Full methods (state and action spaces, the localisation-action model, the baseline list) of both When-to-Localize papers were not read. The POMDP paper's experiments are "numerical" per its abstract, and the RiskRL code indicates grid worlds. No hardware validation was found.
- **RiskRL venue.** The v2 venue is unverified.
- **Tiny-robot active localisation.** No 2024–2026 active-localisation work was found for tiny ground robots that re-localise with line or bump sensors. The search was incomplete.

## 5. Indoor magnetic-field localisation, learned compensation of a robot's own magnetic interference, and defence-relevant magnetic anomaly navigation (2024–2026)

### Takeaway
- **Magnetic SLAM is mature elsewhere.** It works for pedestrians, handhelds and model ships (Solin et al., T-RO 2018; Viset et al., Sensors 2022; Kok & Solin, 2024).
- **Self-interference compensation is already done.** Learned compensation of a robot's own magnetic interference already exists: Christensen et al. (IROS 2017) and Lee et al. (IEEE TIE 2020). Aircraft MagNav uses Tolles–Lawson plus neural networks.
- **For a 3pi+, the deciding unknowns are empirical.** How strong and repeatable are floor anomalies at about 1 cm height, compared with the robot's own motor and encoder-magnet fields? A physics-structured (Tolles–Lawson-like) regression is a strong classical baseline that may make learning pointless.
- **Defence relevance is real but at a different scale.** The 2025–2026 activity (Q-CTRL, April 2025; Royal Navy–Sussex, October 2025; the QinetiQ-led ESA NAVISP project from July 2026) concerns aircraft and ships with quantum magnetometers.

### Cited Findings
- **Haverinen & Kemppainen.** An ICRA 2009 (Kobe) paper, "A global self-localization technique utilizing local anomalies of the ambient magnetic field", is confirmed via a reference list. The Robotics and Autonomous Systems 2009 journal paper could not be confirmed — [Crossref reference list [SX]](https://api.crossref.org/works/10.3390%2FS151229799)
- **Solin, Kok, Wahlström, Schön & Särkkä, IEEE T-RO 34(4):1112–1127, 2018** — [arXiv 1509.04634 [SX]](https://arxiv.org/abs/1509.04634); [Aalto accepted manuscript [SX]](https://acris.aalto.fi/ws/portalfiles/portal/30807293/SCI_Solin_Kok_Modeling_and_Interpolation_article.pdf):
  - A GP prior on a latent scalar potential, so the field is curl-free and consistent with Maxwell's equations.
  - Hilbert-space reduced-rank inference, with sequential updating.
  - Applications include mapping with a **Raspberry Pi-powered robot** and with a smartphone.
- **Viset, Helmons & Kok, Sensors 22(8):2833, 7 April 2022** — [MDPI [SX]](https://www.mdpi.com/1424-8220/22/8/2833):
  - A reduced-rank GP map with Laplace basis functions gives analytic field gradients, which allow an EKF instead of a Rao-Blackwellised particle filter.
  - Per-step cost drops from O(Np·Nm²) to O(Nm²).
  - Tested on a foot-mounted sensor dataset and on a model ship in an indoor pool.
  - Drift compensation was comparable to the particle-filter version.
- **Kok & Solin (PREPRINT, 2024).** Online 1-D magnetic-field SLAM with loop-closure detection and an extended Kalman smoother, using a handheld iPad indoors. It is restricted to 1-D paths because GP maps are computationally expensive — [arXiv 2409.01091 [SX]](https://arxiv.org/pdf/2409.01091)
- **Distributed multi-agent magnetic-field norm SLAM with GPs (PREPRINT, 2023)** — [arXiv 2310.19400 [SX]](https://arxiv.org/pdf/2310.19400)
- **MERL (Berntorp & Menner).** A particle filter for GP state-space models that exploits curl-free fields; simulation only — [MERL TR2023-075 [SX]](https://shadow.merl.com/publications/docs/TR2023-075.pdf)
- **Iterative-EKF plus GP magnetic SLAM variant (ICARCV 2024)** — [KFUPM [SX]](https://pure.kfupm.edu.sa/en/publications/an-improved-magnetic-field-slam-algorithm-based-on-iterative-exte/)
- **Cleaning-robot precedent.** Encoder odometry from a cleaning robot plus magnetic SLAM gave drift-free estimates. This is reported as prior work in Viset et al.'s 2021 paper per the extract; the original was not retrieved — [Viset et al. 2021 (NTNU-hosted) [SX]](https://tomgra.folk.ntnu.no/papers/VisetGK2021.pdf)
- **Monte Carlo localisation from local magnetic anomalies for 2-D indoor self-localisation (2013)** — [XJTLU [SX]](https://scholar.xjtlu.edu.cn:443/en/publications/a-monte-carlo-localization-algorithm-for-2-d-indoor-self-localiza)
- **Reinforced-concrete buildings.** Their fields are often inhomogeneous and usable for localisation (CVUT thesis "Indoor Robot Localization Using Magnetometer Data") — [CVUT [SX]](https://dspace.cvut.cz/handle/10467/103405)
- **Christensen, Krell & Kirchner (DFKI), IROS 2017, "Learning Magnetic Field Distortion Compensation for Robotic Systems".** Neural networks and support-vector regression model magnetometer distortions caused by changing currents, postures and configurations. The aim is better heading estimation, SLAM and magnetic localisation — [DFKI [SX]](https://robotik.dfki-bremen.de/en/research/publications/9087)
- **Lee et al., IEEE Trans. Industrial Electronics 2020.** Compensates on-board-current magnetic disturbance with a gyro and an EKF on a real UAV, reducing yaw error by about two-thirds versus the existing scheme — [DOI 10.1109/TIE.2019.2950841 via Unpaywall [SX]](https://unpaywall.org/10.1109%2FTIE.2019.2950841)
- **Goll & Borisov, IMEKO TC17 2018.** A magnetic-field reconstruction stand for debugging mobile-robot magnetometer navigation, including compensation of power-supply interference — [IMEKO [SX]](https://www.imeko.info/publications/tc17-2018/IMEKO-TC17-2018-005.pdf)
- **MagNav challenge (Gnadt et al.; PREPRINT, July 2020; MIT, MIT Lincoln Laboratory and AFIT).** Separate the aircraft's field from the Earth's anomaly field. "Traditionally … the Tolles-Lawson model. However, this is impractical for operational aircraft." It calls for neural networks combined with navigation physics — [arXiv 2007.12158 [SX]](https://arxiv.org/abs/2007.12158v1); [AFIT Scholar [SX]](https://scholar.afit.edu/facpub/912)
- **MagNav.jl** — [MagNav.jl README [GH-full]](https://github.com/MIT-AI-Accelerator/MagNav.jl):
  - Covers flight data import and simulation, mapping, aeromagnetic compensation and navigation.
  - "This specific project has concluded. Only minimal changes will be made moving forward".
  - Data use is under a USAF Data Sharing Agreement limited to non-profit research and education, with a one-year term.
- **Reservoir computing and time-delayed neural networks combined with Tolles–Lawson on the challenge data (Physical Review Applied 19, 034030)** — [APS accepted manuscript [SX]](https://link.aps.org/accepted/10.1103/PhysRevApplied.19.034030)
- **Hager et al. (Airbus with NTNU co-authors; PREPRINT, March 2026)** — [arXiv 2603.08265 [SX]](https://arxiv.org/pdf/2603.08265):
  - A Kalman filter with Tolles–Lawson coefficients, plus a neural network for **residual nonlinearities only**.
  - Bounds inertial drift on MagNav data using a magnetometer-only feature set.
  - Accuracy is comparable to offline-trained state of the art.
- **Q-CTRL Ironstone Opal (announced around 14 April 2025; figures are company-sourced)** — [The Quantum Insider [SX]](https://thequantuminsider.com/2025/04/14/q-ctrl-overcomes-gps-denial-with-quantum-sensing-reports-quantum-advantage/); [Critical Comms [SX]](https://www.criticalcomms.com.au/content/transport/article/quantum-assured-navigation-overcomes-gps-denial-986099726):
  - Up to 50× lower positioning error than a high-end conventional alternative. Reports describe the baseline inconsistently, variously as velocity-aided INS.
  - Airborne trials at 19,000 ft.
  - More than 6× better than INS in ground-vehicle tests.
  - Magnetic maps taken from public databases; a quantum magnetometer paired with classical inertial hardware.
  - 99.97 % uptime in flight.
  - The accompanying arXiv 2504.08167 reports accuracy better than a strategic-grade INS in airborne and ground trials — [Emergent Mind summary [SX]](https://www.emergentmind.com/papers/2504.08167).
  - A third-party LinkedIn claim of better than 0.3 nmi over 95 % of a flight, and DO-160 qualification, is **unverified**.
- **Royal Navy and University of Sussex.** Optically pumped magnetometer trials in open water from HMNB Portsmouth, on a Hydrographic Exploitation Group vessel, announced **15 October 2025** — [Royal Navy [SX]](https://www.royalnavy.mod.uk/news/2025/october/15/20251015-rn-university-of-sussex-quantum-trials); [GPS World [SX]](https://www.gpsworld.com/royal-navy-trials-quantum-navigation-systems-with-university-of-sussex/)
- **ESA NAVISP project led by QinetiQ.** "Quantum magnetometer and algorithms for integrated map matching and map building". Starts **30 July 2026** and runs 19 months, with field trials to quantify accuracy, drift mitigation and map repeatability — [ESA NAVISP [SX]](https://navisp.esa.int/project/details/458/show)
- **University of Strathclyde (2026).** Its UMQT team is trialling positioning against crustal geomagnetic anomalies with ultra-stable quantum magnetic sensors; preprint arXiv 2606.24612 — [Strathclyde news [SX]](https://www.strath.ac.uk/science/physics/news/2026/pioneeringgps-freepositioning/)
- **Secondary source, verify before use.** An August 2026 trade report says the Royal Navy tested quantum inertial navigation in the Arctic aboard MV Anvil Point — [ShipUniverse [SX]](https://www.shipuniverse.com/?p=16717)

### Inferences
- **Likely self-interference on a 3pi+ (estimated physics, must be measured).** Three terms, each with an obvious classical model:
  1. **Permanent fields** from motor magnets, encoder magnets and the battery. These are fixed to the body (hard iron), so classical ellipsoid calibration by spinning in place handles them.
  2. **Motor and board currents.** The field is proportional to current (Biot–Savart), so it is linear in PWM duty or a current proxy.
  3. **Rotating encoder magnets on the motor shafts.** They produce a periodic field at shaft frequency. The shaft angle is observable from encoder counts (to within 30°), so a Fourier regression in shaft angle can model it.

  **Aliasing risk (estimated arithmetic from sourced figures).** With 32 mm wheels and a 29.86:1 gearbox:
  - at 0.1 m/s the wheel turns about 1 rev/s and the motor shaft about 30 rev/s;
  - at the 2040 Standard Edition's 1.5 m/s top speed, the shaft would turn about 445 rev/s.

  The library samples the magnetometer at 10 Hz, and the LIS3MDL's fastest ultra-high-performance rate is 155 Hz (560 Hz in medium-performance mode). So shaft-rate interference aliases at almost every useful speed and cannot be low-pass filtered. It must be modelled synchronously with the encoder (by shaft angle), or the robot must stay slow during magnetic sensing. Which harmonics appear depends on the encoder disc's pole count, which was not verified here.

  Together these terms form a Tolles–Lawson analogue (permanent, induced and current-driven terms), which is the classical baseline. Learning earns a place only on residual nonlinearities such as battery sag or magnet temperature. That mirrors Hager et al. (2026), where the network handles residuals only.
- **Sensor noise is not the limit (estimated).** LIS3MDL noise is about 0.32 µT RMS (3.2 mgauss, with 1 mgauss = 0.1 µT, specified at ±12 gauss), while its typical zero-gauss offset is ±1 gauss (100 µT). The limits are offset and self-interference calibration, then map repeatability.
- **Novelty (inference).** Learned self-interference compensation on robots is not new (Christensen et al., 2017). What was not found is the micro-robot case: millimetre-to-centimetre distances, aliased encoder-magnet interference, and floor-level maps at about 1 cm. That is "no overlap found", not established novelty.
- **Map-repeatability threats (estimated).** Moved furniture or equipment, lab electronics, the table's material, and changes in height or tilt. Near the floor, gradients are larger, which means more information but also more sensitivity.
- **Defence relevance (inference).** The method chain (compensation, then map matching, then filtering) mirrors UK defence magnetic-navigation work. The scale and sensors differ, though, so a 3pi+ study is a teaching analogue, not a contribution to MagNav proper.

### Gaps
- **Floor-anomaly strength.** No sourced microtesla magnitudes or height dependence for indoor floor anomalies; the search returned only qualitative sources. The student must measure this.
- **3pi+ self-interference.** No sourced measurement of its magnitude.
- **Unverified papers.** Haverinen & Kemppainen (RAS 2009) and Kok & Solin (FUSION 2018) were not verified.
- **MagNav results.** The MagNav challenge scores and tables were not read.
- **Q-CTRL figures.** These are company-sourced; arXiv 2504.08167 was not read.
- **Random-forest compensation.** A Hoang & Pietrosanto random-forest yaw-compensation paper appeared in the results, but its URL could not be identified from the extract.
- **2024–2026 ground robots.** No ground-robot (wheeled) deployment of magnetic SLAM from 2024–2026 was found. The searches returned pedestrian, handheld and ship platforms.

## 6. Which candidate dissertation questions survive all eight filters, and what is each one's first-week kill test? (Verdict on the student's open "active localisation" idea)

### Takeaway
- **Candidate A (conditional pass on all eight filters).** One candidate survives: a hardware study separating **calibration** from **decision** in "when to re-localise" on the 3pi+. It needs two conditions: (1) the student reads RiskRL and the POMDP paper in full, and (2) week-1 data show calibrated dead reckoning exceeds the tolerance of a pre-registered task.
- **Candidate B (conditional).** A magnetic self-interference and map-repeatability study survives as a GNC estimation project only if week-1 measurements are favourable. Its learning component probably fails filter 4.
- **The student's open idea.** It survives only if reframed from "learn a when-to-localise policy" — which RiskRL already does in simulation — to "what limits the decision on real hardware: calibration or the rule?"

### Cited Findings
- **Nearest overlapping work for Candidate A.**
  - RiskRL: particle filter plus recurrent SAC with chance constraints; at least 26 % higher success rate in unseen environments (v2); grid-world code — [arXiv 2411.02788 [SX]](https://arxiv.org/pdf/2411.02788); [README [GH-full]](https://github.com/raaslab/when-to-localize-riskrl).
  - CC-POMDP predecessor, which needs "well-defined transition and observation models" and has "high compute time" — [arXiv 2411.08281 [SX]](https://arxiv.org/abs/2411.08281).
- **Nearest hardware calibration work.** CLAPS, conformal SE(2) sets on a real MBot, December 2025 — [arXiv 2512.10294 [SX]](https://arxiv.org/html/2512.10294v1)
- **Evidence that learned or filter covariances lack guarantees or are overconfident.**
  - [arXiv 2605.00233 [SX]](https://arxiv.org/pdf/2605.00233) (May 2026).
  - [arXiv 2609.14271 [SX]](https://arxiv.org/pdf/2609.14271) (September 2026).
- **Nearest work for Candidate B.**
  - Learned robot magnetic-distortion compensation — [Christensen et al., IROS 2017 [SX]](https://robotik.dfki-bremen.de/en/research/publications/9087).
  - Tolles–Lawson with a neural-network residual — [Hager et al., 2026 [SX]](https://arxiv.org/pdf/2603.08265).
  - Pololu's warning about the 3pi+'s own fields — [User's Guide §5.6 [SX]](https://www.pololu.com/docs/0J83/5.6).
  - 1-D magnetic SLAM — [Kok & Solin, 2024 [SX]](https://arxiv.org/pdf/2409.01091).

### Inferences

#### Candidate A: calibration versus decision in "when to re-localise" on a 3pi+ (CONDITIONAL PASS)
- **(a) Research question.** On a 3pi+ doing pre-registered waypoint tasks in an arena with sparse tape lines and walls, how much of the gap between a tuned covariance-threshold re-localisation policy and a hindsight-oracle policy is closed by:
  - (i) **recalibrating the odometry uncertainty**: adaptive covariance, or conformal/ACI updated at re-localisation events; versus
  - (ii) **a richer decision rule**: one-step expected cost, chance-constrained POMCP, or a RiskRL-style recurrent SAC?
- **(b) Why now.**
  - The CC-POMDP paper (IEEE SSRR 2024; arXiv November 2024) and RiskRL (arXiv 2411.02788, v2 2025) defined the chance-constrained "when to localise" problem. They stayed in simulation with known models and flagged compute and model requirements as limitations.
  - CLAPS (December 2025) showed conformal SE(2) sets on a real small differential-drive robot.
  - Preprints from May 2026 (adaptive geodesic CP) and September 2026 (probe-set recalibration) stress that learned or filter covariances are uncalibrated or overconfident.
- **(c) Nearest work and what remains open.**
  - **RiskRL.** Particle-filter belief, recurrent SAC and a chance constraint; Gymnasium grid worlds.
  - **What it leaves open:**
    - real re-localisation actions that take time and can fail (missed line, wrong line);
    - 1-D partial fixes;
    - the calibration of the belief that drives decisions;
    - a hardware comparison against tuned-threshold and expected-cost rules.
  - **CLAPS** calibrates action-outcome sets on an MBot but does not use them for re-localisation decisions.
  - **Overlap status:** "no overlap found" for the hardware decomposition. This is not established novelty, because neither paper was read beyond its abstract or README.
- **(d) Strongest simple baseline, and why it might fail here.**
  - **Baseline:** a trace(P) threshold tuned on training runs, plus the one-step expected-cost rule.
  - **Why it might fail:** the EKF's σ is likely wrong exactly when it matters — slip during acceleration or turns, surface transitions, temperature-driven gyro bias — and detour costs vary with position.
  - **How to show it:**
    - empirical-versus-nominal coverage of the predicted error at line crossings;
    - NIS statistics;
    - a regret decomposition across a factorial design of {fixed-Q EKF, adaptive-Q, learned-covariance adapter, ACI-recalibrated} × {threshold, expected-cost, POMCP/RL}.
- **(e) First-week test and kill condition.**
  - **Day 1–2.** Read WHO_AM_I. Build the overhead webcam plus fiducial ground truth and measure its accuracy against a ruler grid.
  - **Day 3–4.** Run UMBmark (clockwise and counter-clockwise squares at the largest size the arena allows). Calibrate gyro bias at rest, and gyro scale factor via N spins over a radial tape line.
  - **Day 5.** Run the pre-registered task course open-loop about 10 times on each of the lab's real surfaces, logging encoders, gyro and PWM. Compute the final-error distribution against the task tolerance τ, and the EKF's NEES.
  - **Kill if** the 95th-percentile calibrated dead-reckoning error over the full course is below τ on every surface. Re-localisation is then unnecessary at tabletop scale; report the critical length L* and move on. Do **not** lengthen the course or tighten τ to rescue the idea, because that would fail filter 3.
  - **Soft kill if** NEES is already consistent across surfaces. The calibration arm then has nothing to fix, and a threshold policy is likely near-optimal.
- **(f) Hardware and compute.**
  - **Owned (per the brief):** the 3pi+ and the M5Stamp C3 telemetry firmware for logging; the USB webcam for ground truth.
  - **Webcam accuracy:** not measured. It is estimated at sub-centimetre over about 1 m² with fiducials and must be measured.
  - **Consumables:** tape and printed fiducials (prices not found; estimated a few pounds); arena boards (estimated under £30).
  - **Compute (estimated):** the RTX 3060 is ample for an EKF, small networks and POMCP over a low-dimensional belief.
  - **Ask BRL for:**
    - 2–4 hours of motion-capture time to validate the webcam ground truth (availability unconfirmed);
    - any 3pi+ odometry logs from past Robotics Science and Systems cohorts;
    - a second 3pi+ to check unit-to-unit variation (optional).
- **(g) Headline figure, best realistic outcome and floor.**
  - **Headline:** "Of the regret (time plus failures) between a tuned threshold and the hindsight oracle, X % is removed by calibration and Y % by a better decision rule." Plus empirical coverage at 90 % nominal for each calibration method across surfaces.
  - **Best realistic:** a clean decomposition in which a calibrated threshold matches POMCP or RL within noise. This would be an informative negative result for learned policies.
  - **Floor:** dead reckoning is within tolerance for tabletop tasks. The result is then L*(τ, surface) plus a characterisation of 3pi+ odometry calibration and coverage — a measured boundary.
  - **Variance and regime map:** vary line density as an independent variable and report the regime map, so that line placement is not chosen to manufacture the gap.
- **(h) Filter-by-filter verdict.**
  1. **Pass.** The robot's sensing and decisions are the subject.
  2. **Conditional.** The baseline is named, but its failure mechanism must be shown with week 1–3 data.
  3. **Conditional.** The gap is real only if a pre-registered task, with τ taken from a stated task such as ±1–2 cm docking, exceeds dead-reckoning tolerance. Surfaces must be the lab's real ones, and the camera is for evaluation only.
  4. **Pass.** ACI is justified by sparse, delayed, policy-dependent labels; POMCP by the chance constraint; RL is one arm, not the thesis.
  5. **Partial.** RiskRL, the POMDP paper and CLAPS were read only at abstract or README level, and the student must read them in full.
  6. **Pass.** Owned hardware suffices. The simulator must be a noise model fitted to logs, not the existing kinematic simulator.
  7. **Pass.**
  8. **Pass.** The factorial decomposition is informative whichever arm wins.
- **(i) Skills built.** EKF design; UMBmark and gyro calibration; NEES/NIS consistency testing; conformal and ACI methods; POMDP and constrained RL; embedded logging; pre-registered experimental design. These map to GNC, autonomy and applied-ML roles.
- **(j) Bristol/BRL people.** None verified in this session, because the search budget was exhausted before the BRL search. Practical lead: the teaching team of the Robotics Science and Systems unit that issues the 3pi+. Defer to the BRL-staff researcher (task #7).

#### Candidate B: characterise and compensate a 3pi+'s self-generated magnetic interference, and test floor-level map repeatability (CONDITIONAL; GNC/system-identification rather than learning)
- **(a) Research question.** Can a 3pi+'s own magnetic interference — hard iron, motor current, and rotating encoder magnets aliased at low output data rate — be compensated well enough to give:
  - (i) usable magnetometer heading at speed, and
  - (ii) repeatable floor-level magnetic maps for re-localisation at about 1 cm height?

  The compensation is a Tolles–Lawson-style physics regression, with or without a learned residual.
- **(b) Why now.**
  - Hager et al. (March 2026): Tolles–Lawson plus a neural-network residual inside a Kalman filter.
  - Kok & Solin (2024): 1-D magnetic SLAM.
  - Active UK and allied magnetic-navigation work: Q-CTRL (April 2025), Royal Navy–Sussex (15 October 2025), QinetiQ's NAVISP project (from 30 July 2026), Strathclyde (2026).
- **(c) Nearest work and what remains open.**
  - Christensen et al. (IROS 2017) compensate robot currents and postures with neural networks and SVR. This is strong conceptual overlap.
  - Lee et al. (TIE 2020) compensate UAV currents, cutting yaw error by about two-thirds.
  - MagNav covers aircraft.
  - Open: a micro-robot with encoder magnets at centimetre range, aliasing at 10 Hz output rate, and repeatability of floor-level maps.
- **(d) Strongest simple baseline, and why it might fail.**
  - **Baseline:** ellipsoid hard- and soft-iron calibration, plus linear terms in PWM or current, plus Fourier terms in shaft angle.
  - **Why it might fail:** current or temperature nonlinearity; near-field magnets poorly described by low-order terms; aliasing.
  - **How to show it:** held-out speeds and battery voltages.
- **(e) First-week test and kill condition.**
  - **Lifted-wheel PWM sweep.** Log the magnetometer at its maximum output rate (155 Hz in ultra-high-performance mode, or 560 Hz in medium-performance mode, per the datasheet), plus encoders and PWM. Fit the baseline and record the residual σ_self.
  - **Floor-grid survey.** Survey about a 10 cm grid over about 1 m² with motors off, on two different days. Record the spatial σ_map and the day-to-day correlation r.
  - **Kill if** any of these holds (thresholds estimated):
    - σ_map is below about 3 × σ_self;
    - r is below about 0.8;
    - the anomaly correlation length is shorter than typical dead-reckoning error.
  - **Kill the learning arm if** the physics regression already leaves residuals near sensor noise (about 0.32 µT).
- **(f) Hardware and compute.**
  - Owned: the 3pi+ and the webcam.
  - Optional: an external reference magnetometer on a non-magnetic mast (price not found).
  - Compute: trivial (estimated).
- **(g) Headline figure, best realistic outcome and floor.**
  - **Headline:** residual self-interference in µT versus wheel speed, plus map repeatability r and re-localisation success rate at about 1 cm height.
  - **Best realistic:** magnetometer heading within a few degrees at speed, and tape-free re-localisation.
  - **Floor:** a quantified "micro-robot magnetometer unusable above speed X" boundary.
- **(h) Filter-by-filter verdict.**
  1. **Pass.**
  2. **Pass.**
  3. **Pass.** Pololu itself documents the self-interference.
  4. **At risk for learning.** It passes only if framed as system identification with learning on the residual.
  5. **Partial.** Christensen et al. 2017 was not read beyond the abstract.
  6. **Pass.**
  7. **Pass.**
  8. **Pass.**
- **(i) Skills built.** Magnetometer calibration, system identification, GP map-based localisation and sensor fusion. These are directly relevant to UK defence GNC and magnetic-navigation roles.
- **(j) Bristol/BRL people.** None verified.

#### Tempting but fails (filter named)
- **IMU-only learned inertial odometry on the 3pi+ (RoNIN, TLIO, AirIO or Tartan IMU style, including LoRA fine-tuning of Tartan IMU).** Fails filters **2 and 3**. The encoders already measure velocity, so the IMU-only problem is manufactured on this platform.
- **Learned gyro denoising (Brossard, RA-L 2020) on the LSM6DS33.** Fails filter **2**. Zero-rate updates at every encoder-detected stop, plus temperature-bias calibration, should leave little to learn. Making learning matter would need artificially long runs without stops, which also fails filter **3**.
- **KalmanNet-style learned Kalman gain for wheel–gyro fusion.** Fails filter **4**. The dynamics are known, so the "partially known dynamics" motivation does not apply; the gap is in non-stationary noise, which covariance adaptation addresses more directly.
- **End-to-end RL "when to re-localise" policy as the headline method.** Fails filter **4** (method chosen for being new; RiskRL already does this in simulation) and is at risk on filter **2** (a tuned threshold is likely near-optimal for monotone uncertainty growth). Keep it only as one arm of Candidate A.
- **Standalone learned slip detection.** Fails filter **3** unless realistic slip occurs in the intended task, and filter **2** because encoder–gyro yaw-rate disagreement plus χ² innovation gating is a strong classical detector. Keep it only as Candidate A's covariance-adapter arm.
- **Cooperative multi-robot re-localisation with several 3pi+ robots.** Fails filter **6**. It needs extra robots (BRL availability unconfirmed) and an inter-robot relative-sensing modality the 3pi+ lacks.
- **Magnetometer or localisation spoofing and resilience.** Already rejected by the student, and this research found no new evidence to revive it.
- **Quantum or optically pumped magnetometer anomaly navigation as a defence analogue.** Fails filter **6**. Such sensors are far beyond the £200 budget (estimated; no prices found).
- **1-D magnetic route re-localisation (Kok & Solin style) as a standalone project.** At risk on filter **3**. On a lab bench, tape markers are a trivial and better alternative unless "no infrastructure" is a genuine requirement. It survives only as an add-on to Candidate B if B's week-1 test passes.

#### Overall verdict and ranking (by evidence)
1. **Candidate A (conditional).** It turns the student's open idea into a question whose answer is useful whichever way it falls. The week-1 dead-reckoning test is decisive. It also tests the student's own prior lesson that simple rules beat learners.
2. **Candidate B (conditional).** A solid GNC estimation project only if week-1 field measurements are favourable. Learning is unlikely to be justified beyond the residual.
3. **Everything else** fails at least one filter.
- **Shared prerequisite.** Both candidates need the same week-1 infrastructure: webcam ground truth, calibrated dead reckoning and a logging pipeline. Running that first de-risks both, and the student's ESP32-C3 encoder-telemetry firmware is a head start.

### Gaps
- **Search and access limits (affect every verdict above).**
  - arXiv, IEEE, HAL, PMC/MDPI, OpenAlex, Semantic Scholar, pololu.com and st.com were blocked. WebFetch failed DNS resolution. The shared web-search budget ran out.
  - Not run because of the budget:
    - BRL and Bristol staff connected to localisation;
    - Prentice & Roy and Platt et al. (belief space);
    - Gibbs & Candès (ACI);
    - gyrodometry;
    - floor-anomaly magnitudes;
    - the CLAPS methods;
    - 2025–2026 wheel-inertial slip-learning papers;
    - Christensen et al. 2017 methods.
- **Must-read in full before the topic is agreed.** These would upgrade filter 5 from partial to pass:
  - RiskRL (arXiv 2411.02788 v2) and the CC-POMDP paper (arXiv 2411.08281);
  - CLAPS (arXiv 2512.10294);
  - Christensen, Krell & Kirchner (IROS 2017);
  - AI-IMU (IEEE T-IV 2020) and WING (arXiv 2407.10101);
  - Kok & Solin (arXiv 2409.01091).
  - The datasheets have now been read via GitHub-hosted copies, but the student should confirm against ST's current revisions.
- **Unverified items.**
  - RiskRL's venue.
  - TLIO's and KalmanNet's venues; AirIMU's venue.
  - Haverinen & Kemppainen RAS 2009.
  - Kok & Solin FUSION 2018.
  - LSM6DS33 bias instability, which the datasheet does not give.
  - The 3pi+ track width, and the 32U4 Standard Edition's top speed.
  - Webcam ground-truth accuracy.

#### Source register (read depth)

| Source | Venue / status (as verified here) | Read depth |
|---|---|---|
| Pololu 3pi+ 32U4 library: IMU.cpp, IMU_declaration.h, README, LineSensors.h, BumpSensors.h, Encoders.h | Vendor code, v1.1.3 (2022-09-06) | Read in full [GH-full] |
| Pololu 3pi+ 32U4 User's Guide §5.4, §5.6 | Vendor documentation | Search extract only [SX]; site blocked |
| Pololu 3pi+ 2040 User's Guide (saved copy in a third-party GitHub repo) | Vendor documentation | IMU, encoder, chassis and edition sections read directly [GH-copy] |
| LSM6DS33 datasheet, DocID027423 Rev 4 (copy in Stanford's cs240lx repo) | ST datasheet, 2015 | Table 3 read in full [GH-copy] |
| LIS3MDL datasheet, DocID024204 Rev 6 (copy in a third-party repo) | ST datasheet, 2017 | Table 3 and output-data-rate tables read in full [GH-copy] |
| LSM6DS3 datasheet (Digi-Key); Mouser LSM6DS33 copy and release | ST datasheets / press | Search extract only [SX]; superseded by the direct reads above |
| LIS3MDL: ST Community 2019 and 2023, element14, ST product page | Forum / vendor | Search extract only [SX] |
| Borenstein & Feng UMBmark | IEEE T-RA 1996 | Search extract only [SX] |
| Sousa et al. odometry calibration review | 2020 review (venue per INESC record) | Search extract only [SX] |
| Brossard & Bonnabel | ICRA 2019 | Search extract only [SX] |
| AI-IMU dead reckoning | IEEE T-IV 2020 | README read in full [GH-full]; paper [SX] |
| Gyro denoising | IEEE RA-L 2020 | README read in full [GH-full]; paper [SX] |
| TLIO | RA-L 2020 (per brief; not re-verified) | README read in full [GH-full]; paper [SX] |
| RoNIN | ICRA 2020 | README read in full [GH-full] |
| AirIMU | PREPRINT (arXiv 2310.04874) | README read in full [GH-full] |
| AirIO | IEEE RA-L 2025 | README read in full [GH-full]; paper [SX] |
| Tartan IMU | CVPR 2025 | Search extract only [SX] |
| KalmanNet | arXiv 2107.10043 (TSP 2022 per brief; not re-verified) | README read in full [GH-full] |
| KalmanNet uncertainty | ICASSP 2022 | Search extract only [SX] |
| Bayesian KalmanNet | IEEE TSP 2025 per extract | Search extract only [SX] |
| Mortada et al. | PREPRINT 2026 | Search extract only [SX] |
| WING | PREPRINT 2024 | Search extract only [SX] |
| Navone et al.; Okawara et al.; SlipOdometryNet; KIST LSTM | Preprints / 2023–2024 | Search extract only [SX] |
| A-KIT (Cohen & Klein) | Engineering Applications of AI 2025 | Search extract only [SX] |
| Conformal VO (Stutts et al.); Parente et al. | 2023 | Search extract only [SX] |
| CLAPS | PREPRINT, December 2025 | Search extract only [SX] |
| Adaptive geodesic CP | PREPRINT, May 2026 | Search extract only [SX] |
| ACI via particle filtering | PREPRINT, November 2024 | Search extract only [SX] |
| Probe sets for nonlinear KF | PREPRINT, September 2026 | Search extract only [SX] |
| Lindemann et al. survey | PREPRINT, v3 August 2025 | Search extract only [SX] |
| Liang et al. | L4DC 2026 | Search extract only [SX] |
| Conformal sound-source localisation | ICASSP 2025 | Search extract only [SX] |
| Fox, Burgard & Thrun | Robotics and Autonomous Systems 1998 | Search extract only [SX] |
| Active Neural Localization | ICLR 2018 | Search extract only [SX] |
| Deep Active Localization | RA-L 2019 | Search extract only [SX] |
| Placed et al. active SLAM survey | IEEE T-RO 2023 | Search extract only [SX] |
| When to Localize? POMDP | IEEE SSRR 2024 | Search extract only [SX] |
| RiskRL | arXiv; venue unverified | Paper [SX]; code README read in full [GH-full] |
| GUIDE | arXiv 2410.15178 | Search extract only [SX] |
| Localizability-aware DRL | IET CSR, press August 2025 | Search extract only [SX] |
| Solin et al. | IEEE T-RO 2018 | Search extract only [SX] |
| Viset et al. | Sensors 2022 | Search extract only [SX] |
| Kok & Solin | PREPRINT 2024 | Search extract only [SX] |
| Distributed multi-agent norm SLAM | PREPRINT 2023 | Search extract only [SX] |
| MERL (Berntorp & Menner) | MERL technical report | Search extract only [SX] |
| Iterative EKF + GP magnetic SLAM | ICARCV 2024 | Search extract only [SX] |
| Christensen et al. | IROS 2017 | Search extract only [SX] |
| Lee et al. | IEEE TIE 2020 | Search extract only [SX] |
| Goll & Borisov | IMEKO TC17 2018 | Search extract only [SX] |
| MagNav challenge (Gnadt et al.) | PREPRINT 2020 | Search extract only [SX] |
| MagNav.jl | Software | README read in full [GH-full] |
| Reservoir computing on MagNav data | Physical Review Applied 19, 034030 | Search extract only [SX] |
| Hager et al. | PREPRINT, March 2026 | Search extract only [SX] |
| Q-CTRL coverage (The Quantum Insider, Critical Comms, Emergent Mind) | Press / company-sourced | Search extract only [SX] |
| Royal Navy release; GPS World | Official / trade press, 2025 | Search extract only [SX] |
| ESA NAVISP project page | Official, 2026 | Search extract only [SX] |
| Strathclyde news | University news, 2026 | Search extract only [SX] |
| ShipUniverse | Trade press, 2026 (secondary) | Search extract only [SX] |
| Haverinen & Kemppainen | ICRA 2009 (via reference list); RAS 2009 not found | Not accessed beyond the citation |
| Gibbs & Candès ACI; Prentice & Roy; Platt et al.; Kok & Solin FUSION 2018; gyrodometry | — | Not accessed (search budget exhausted) |
