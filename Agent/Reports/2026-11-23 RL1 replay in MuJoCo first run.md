---
date: 2026-11-23
tags: [report, replay-ladder, simulation]
---
# RL1 (replay in MuJoCo on the Mac): first run

Ask (Martyn, 2026-11-21): start RL1 as far as possible without lab hardware, and get a Unitree H2 model loading in MuJoCo "just to make it ready". Context: [[2026-11-20 Replay Ladder RL1-RL2 setup]], [[2026-11-20 Interim hardware while the Dex1-1 is delayed]].

Code: dex1-1-handheld branch `claude/project-thread-bh74ma`, `software/sim/` (`fetch_models.sh`, `load_models.py`, `ur10e_replay.py`, `README.md`). Team code: `~/thesis/team/xr_teleoperate` at `7e6c754`, imported, not copied.

## In short

- **Both models load** on the Mac: MuJoCo Menagerie UR10e and Unitree's own H2 model, both BSD-3-Clause, downloaded at pinned commits into `software/sim/models/` (not in Git).
- **No new dependency was needed.** The existing conda env `humanoid` already has MuJoCo 3.11.0, Pinocchio 4.1.0 and CasADi 3.7.2. Three team imports the UR10e IK does not use (meshcat, logging_mp, matplotlib) are replaced by empty stand-ins when missing.
- **The team UR10e IK drives the Menagerie UR10e unchanged.** Joint names, zero positions, joint limits and base frame match; the Menagerie flange is 16.55 mm short (corrected when measuring).
- **Synthetic pick-and-place replays correctly:** directions and 1:1 scale come out as intended (table below), and the simulated tool ends 0.8 mm from the target.
- **Replay input format = `trigger_log.py` `raw.csv`**, so the first lab recording plugs in without conversion.

## Checks (measured 2026-11-23)

| Check | Result |
| --- | --- |
| UR10e loads, holds home keyframe 2 s | Largest joint change 0.007 rad (position servos, no gravity compensation) |
| H2 loads and steps 2 s | 32 joints (1 floating base + 31), 31 torque motors, 71.3 kg, finite; falls without a controller (expected) |
| H2 joint names vs team `assets/h2/H2.urdf` | Identical sets (31 joints) |
| UR10e: team URDF (`base`→`tool0`, Pinocchio) vs Menagerie (`attachment_site`, MuJoCo), 200 random poses | Rotation identical; position 16.5 mm mean, 17.4 mm max; after adding 16.55 mm along the tool axis (UR10e d6 = 0.11655 m vs Menagerie 0.100 m): 0.7 mm mean, 1.25 mm max (rounded link lengths) |
| Gravity sag of the Menagerie servos at the team `HOME_Q` | Shoulder 0.48°, elbow 0.37°, about 10 mm at the tool; removed by MuJoCo gravity compensation in the replay (the real UR controller compensates gravity) |
| Synthetic pick-and-place, 15 s, path 750 mm, peak 0.19 m/s | IK error RMS 0.6 mm; sim tracking RMS 6.5 mm, max 13.5 mm (servo lag while moving), 0.8 mm at rest at the end |
| Team IK solver failures | 1 of 2010 solves (NaN gradient in CasADi); the IK holds the pose for that tick, giving a 10.6 mm error and a 0.45 rad/s catch-up |
| Same path at scale 0.8 | IK max 1.0 mm; sim tracking max 9.8 mm; highest joint speed 0.15 rad/s |

Operator direction → simulated TCP movement in the UR base frame (scale 1:1):

| Leg (operator) | Base x (mm) | Base y (mm) | Base z (mm) |
| --- | --- | --- | --- |
| Down 100 mm | 0 | 0 | −99 |
| Forward (toward the robot) 150 mm | 0 | +150 | +1 |
| To the operator's left 200 mm (wrist turns 20°) | −200 | 0 | 0 |
| Up 100 mm | 0 | 0 | +99 |

So with the team's `BASE_YAW = π/2`, operator forward = UR base +y and operator left = base −x. Whether base +y really points away from the operator at the station is not checked (station layout, pendant).

## How the recording is mapped

As the team's VR teleop: XRoboToolkit pose taken as OpenXR basis (team default, no extra change of basis), turned into robot convention (x forward, y left, z up); only motion since the first sample is used, first pose → team `HOME_Q`; `BASE_YAW` for the half-turn between operator and robot; TCP 0.174 m along `tool0`. Details and column table: `dex1-1-handheld/software/sim/README.md`.

## First real take (2026-11-23, ≈13:30 CET, Martyn)

Desk recording with `pose_log.py` on the station Jetson (button start/stop), replayed on the Mac with `ur10e_replay.py --view`. 43.2 s, 3377 samples (78 Hz mean; one 0.78 s gap at 4.4 s, similar to the 0.66 s gap of 18 Nov, cause unknown). Hand path range in the base frame 230 × 232 × 177 mm, up to 81° rotation.

| Check | Result |
| --- | --- |
| IK error | RMS 0.8 mm, max 12.9 mm; 0 solver failures in 5525 solves |
| Sim tracking | RMS 6.8 mm, max 66.1 mm, 0.8 mm at rest at the end; > 20 mm for 0.4 s in total |
| Max 66 mm (t = 18.9 s) | Fast hand move, about 9 cm in 0.1 s (raw samples smooth, not a tracking jump); servo lag |
| Joint speed max 4.28 rad/s (t = 40.1 s, wrist 1) | Fast wrist twist, about 20° in 0.1 s (raw samples smooth) |
| Joint speed > 0.3 rad/s (team real-robot limit) | 11.3 s of 44.2 s |
| Directions | First move → base +y (forward), later move → +z (up), as intended; Martyn watched the replay: the simulated movement matches the recorded movement (2026-11-23) |

**RL1 passed (visual check, Martyn, 2026-11-23).** The headset-axis assumptions (SDK pose in the OpenXR basis; forward = direction faced at recentre) hold qualitatively; not measured.

Consequence for RL3 (replay on the real UR10e): slow takes, or a 0.3 rad/s joint speed limit in the replay (proposed, not added).

## Open

- RL1 pass check on a **real** recording: record small slow moves with `trigger_log.py` at the station (procedure in the README), replay, check directions. Two unverified assumptions it tests: the SDK pose is in the OpenXR basis (team code says "VERIFY w/ A/B test"), and the PICO room forward = direction faced at recentre.
- Station check: does UR base +y point away from the operator (table side)?
- Start pose: the team `HOME_Q` (tool pointing down) is fine for the controller-only test (Martyn, 2026-11-23). Testing with the mounted gripper (tool pointing forward) needs a second home pose and matching orientation mapping.
- First real recording can be made at a desk, not at the robot: simple moves only, logged on a computer running the XRoboToolkit PC Service (Martyn, 2026-11-23).
- Controller-to-gripper-tip offset (D09 (calibration)) not applied; RL1 replays the controller origin.
- The simulated arm has no joint speed limit; the team's real-robot path limits joints to 0.3 rad/s, so fast recordings will lag on the robot (RL2 (replay against URSim) / RL3 (replay on the real UR10e)).
- Team IK: rare CasADi NaN failure (1 in 2010 here, likely the rotation-error log at zero rotation; cause not proven). Reported, not fixed (team code).
- H2: standing needs Unitree's simulator bridge (`unitree_sdk2_python`, CycloneDDS) and a balance controller: new dependencies, not installed.
- Raw MCAP recorder (D07 (software and data format), 2026-11-23): the replay gets an MCAP reader when that recorder exists.
- dex1-1-handheld PR not opened yet (waiting for Martyn's OK to commit and push).
