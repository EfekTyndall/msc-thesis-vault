---
date: 2026-10-19
status: accepted
tags: [decision]
---
## Context
How the transforms, latencies and width mappings the data depends on are calibrated and versioned. Source: `msc-thesis/docs/proposal.md` §9 (approved by Supervisor on 19 Oct 2026).

## Options

| Calibration | Method | Cadence |
| --- | --- | --- |
| Camera intrinsics (handheld and robot cameras) | Fisheye model ([Kannala-Brandt via OpenCV](https://docs.opencv.org/4.x/db/d58/group__calib3d__fisheye.html) or [Kalibr](https://github.com/ethz-asl/kalibr/wiki/supported-models)); target < 0.5 px. Used for geometry checks only: the policy receives raw images, as in [UMI](https://arxiv.org/pdf/2402.10329) | Once per camera unit |
| Controller-to-gripper-tip transform | **Since 2026-11-23:** fixed offset read from the Fusion CAD assembly (controller mount → gripper tip), stored as a versioned file. Was: printed mount on the real Dex1-1 on the UR10e; 10–20 poses with large rotations; [`cv2.calibrateRobotWorldHandEye`](https://docs.opencv.org/4.x/d9/d0c/group__calib3d.html) (template: [UMI script](https://github.com/real-stanford/universal_manipulation_interface/blob/main/scripts/calibrate_robot_world_hand_eye.py)) | Once per build; re-read after a CAD change or reprint |
| Encoder angle → fingertip width | **Since 2026-11-23:** width from the raw Dex1-1 motor angle (same gripper on handheld and robot); caliper only as a sanity check at delivery. Was: \~10 caliper-measured gaps; lookup table or polynomial; per unit ([UMI](https://github.com/real-stanford/universal_manipulation_interface/blob/main/scripts/calibrate_gripper_range.py) recalibrates its range per session) | Sanity check once per unit at delivery |
| Dex1-1 command → fingertip width | Command sweep on the real gripper, measured gaps, inverted at deployment ([Dex1-1](https://unitree.com/Dex1-1): 90 mm stroke, 0.1 mm resolution) | Once per robot gripper |
| Latency table | Time-coded marker on a monitor (camera), headset timestamps plus clock-offset estimate (PICO), cross-correlation or LED (encoder); see section 6 | Once per build; clock-offset check every session |
| Visual match of printed copy vs real gripper (only if the printed-copy fallback of [[D01 Handheld gripper body]] is resumed) | Images at equal widths; finger-mask overlap and camera pose offset | Once per build; after finger or mount reprint |
| Tracking accuracy and jitter | Controller pose vs UR10e TCP pose after hand-eye; RMSE and static jitter (method of [arXiv 2412.06116](https://arxiv.org/html/2412.06116v1)) | Once per build; short static log every session |

## Decision

Treat calibration as its own subsystem that produces versioned files, with three cadences: once per build, after any reprint, and every session. Each dataset records which calibration version it used.

Controller-to-tip and width (2026-11-23): no robot hand-eye calibration. The controller → gripper-tip transform is a fixed offset read from the Fusion CAD assembly and stored as a versioned file. Gripper width comes from the raw Dex1-1 motor angle; the same gripper type is on the handheld and the robot, so handheld and robot widths share one scale. A caliper check at delivery is a sanity check only.

## Evidence

[ViTaMIn-B](https://arxiv.org/html/2511.05858v1) is the only variant that describes its calibration procedure (hand-eye with about 10 poses on the robot), and it is reproducible here because the real Dex1-1 sits on the UR10e.

Fixed offset (2026-11-23): several variants only state that the tracker-to-gripper transform is fixed by the rigid mount, e.g. [TacUMI](https://arxiv.org/pdf/2601.14550) (cheng2026: fixed tracker → TCP transform) and [RDT2](https://arxiv.org/pdf/2602.03310), and the [UMI code](https://github.com/real-stanford/universal_manipulation_interface) hard-codes measured offsets (`msc-thesis/docs/proposal.md` §9; [[2026-11-18 UMI variants pose tracking and policy actions]]). With relative actions ([[D07 Software and data format]]) a constant translation error largely cancels; rotation errors leak into rotating motions (proposal §9), so the CAD rotation of the controller in the sleeve matters most. Width: the handheld carries the real Dex1-1 ([[D01 Handheld gripper body]], 2026-11-16), so recorded and commanded widths are in the same motor units without a mapping.

## Revisit if

The PICO's tracked origin inside the controller cannot be located (not published; unverified), or a replay on the UR10e (Replay Ladder RL3–RL4) shows a tip offset error large enough to miss the grasp → hand-eye on the robot as in ViTaMIn-B.

## History

- 2026-10-19: accepted (proposal approved)
- 2026-11-19: visual-match calibration applies only if the printed-copy fallback is resumed (real Dex1-1 chosen in [[D01 Handheld gripper body]], 2026-11-16); thesis-plan §3–4 aligned in msc-thesis PR #38 (consistency audit, [[2026-11-19 Three-repo consistency audit]])
- 2026-11-23: controller-to-tip transform from the Fusion CAD assembly instead of hand-eye on the robot; width from the raw Dex1-1 motor angle, caliper only as sanity check (Martyn rejected robot hand-eye 2026-11-19; fixed offsets as in TacUMI, RDT2 and the UMI code; same gripper on handheld and robot)
