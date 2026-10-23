---
date: 2026-10-19
status: accepted
tags: [decision]
---
## Context
How the transforms, latencies and width mappings the data depends on are calibrated and versioned. Source: `msc-thesis/docs/proposal.md` §9 (approved by Yanmei on 19 Oct 2026).

## Options
| Calibration | Method | Cadence |
| --- | --- | --- |
| Camera intrinsics (handheld and robot cameras) | Fisheye model ([Kannala-Brandt via OpenCV](https://docs.opencv.org/4.x/db/d58/group__calib3d__fisheye.html) or [Kalibr](https://github.com/ethz-asl/kalibr/wiki/supported-models)); target < 0.5 px. Used for geometry checks only: the policy receives raw images, as in [UMI](https://arxiv.org/pdf/2402.10329) | Once per camera unit |
| Controller-to-gripper-tip transform | Printed mount on the real Dex1-1 on the UR10e; 10–20 poses with large rotations; [`cv2.calibrateRobotWorldHandEye`](https://docs.opencv.org/4.x/d9/d0c/group__calib3d.html) (template: [UMI script](https://github.com/real-stanford/universal_manipulation_interface/blob/main/scripts/calibrate_robot_world_hand_eye.py)); compare with CAD | Once per build; 5-pose re-check after a reprint |
| Encoder angle → fingertip width | \~10 caliper-measured gaps; lookup table or polynomial; per unit ([UMI](https://github.com/real-stanford/universal_manipulation_interface/blob/main/scripts/calibrate_gripper_range.py) recalibrates its range per session) | Once per build; zero/open check every session |
| Dex1-1 command → fingertip width | Command sweep on the real gripper, measured gaps, inverted at deployment ([Dex1-1](https://unitree.com/Dex1-1): 90 mm stroke, 0.1 mm resolution) | Once per robot gripper |
| Latency table | Time-coded marker on a monitor (camera), headset timestamps plus clock-offset estimate (PICO), cross-correlation or LED (encoder); see section 6 | Once per build; clock-offset check every session |
| Visual match of printed copy vs real gripper | Images at equal widths; finger-mask overlap and camera pose offset | Once per build; after finger or mount reprint |
| Tracking accuracy and jitter | Controller pose vs UR10e TCP pose after hand-eye; RMSE and static jitter (method of [arXiv 2412.06116](https://arxiv.org/html/2412.06116v1)) | Once per build; short static log every session |



## Decision
Treat calibration as its own subsystem that produces versioned files, with three cadences: once per build, after any reprint, and every session. Each dataset records which calibration version it used.

## Evidence
[ViTaMIn-B](https://arxiv.org/html/2511.05858v1) is the only variant that describes its calibration procedure (hand-eye with about 10 poses on the robot), and it is reproducible here because the real Dex1-1 sits on the UR10e.

## Revisit if
–

## History
- 2026-10-19: accepted (proposal approved)
