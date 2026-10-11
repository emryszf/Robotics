# Second pass: review note

Added on 11 October 2026, alongside `Robotics MSc shortlist second pass.md`.

**Where it came from.** The second pass was written in a separate session that had wider web access than the one that produced the first report. This note does not re-check its paper-level claims, because this session's network policy still blocks arXiv, publishers and vendor sites.

## Checked against primary sources on GitHub (10 October 2026)

- RAPTOR's [README](https://raw.githubusercontent.com/rl-tools/raptor/master/README.md) lists reference deployments for PX4, Betaflight, Crazyflie and the M5StampFly.
- RAPTOR's policy takes position, orientation, linear velocity, angular velocity and its previous action as inputs.
- The same README warns that pre-training RAPTOR's 1,000 teacher policies "will take a long time", and lets you start from the released teacher checkpoints instead.
- The Crazyflie firmware includes an INDI controller ([`controller_indi.c`](https://raw.githubusercontent.com/bitcraze/crazyflie-firmware/master/src/modules/src/controller/controller_indi.c)).
- The StampFly [operation manual](https://raw.githubusercontent.com/M5Fly-kanazawa/StampFly2024June/main/README.md) (June 2024) describes Stabilize and Acro modes and an altitude hold that is "in development", with no position hold.

## Review corrections already in the second pass

- N1's question no longer assumes RAPTOR beats classical control with motion capture. It is now a 2 × 2: motion capture or onboard sensing, crossed with RAPTOR or classical control.
- N1 has two classical baselines: gains fixed on the unmodified drone, and gains retuned for the perturbation as an upper bound. The classical position loop is validated before RAPTOR flies.
- There is one perturbation, fixed in advance. Equivalence within a pre-set margin is a floor result, not a kill.
- A borrowed Crazyflie with a Flow deck is the preferred platform, and there is a crash budget.
- RAPTOR is used as released. Any retraining starts from the released teacher checkpoints.
- N2 now asks whether the policy uses onboard sensing or motion-capture state.

## Still open

- **BRL arena size.** The first report's notes give about 15.6 × 11.8 × 5 m with three Vicon systems. The Starling tutorial gives a 10 × 5 × 4 m netted flying volume. This is probably the room versus the netted volume; confirm with BRL.
- **StampFly firmware.** The manual dates from June 2024. Check the current firmware before assuming it still has no position hold.
- **Flight time.** Neither report sources how long one battery lasts or how many flights a session allows. Both numbers set N1's sample size, so measure them in the pilot.
