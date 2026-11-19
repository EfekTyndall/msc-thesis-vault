---
date: 2026-10-19
status: accepted
tags: [decision]
---
## Context

How the gripper opening width is measured on the handheld. Source: `msc-thesis/docs/proposal.md` §5 (approved by Supervisor on 19 Oct 2026).

## Options

| Option | Strengths | Weaknesses | Verdict |
| --- | --- | --- | --- |
| ArUco markers on the fingers, used by [UMI](https://arxiv.org/pdf/2402.10329), [ViTaMIn-B](https://arxiv.org/html/2511.05858v1) and [HuMI](https://arxiv.org/html/2602.06643v2) | No electronics; existing detection code (OpenCV, [UMI](https://github.com/real-stanford/universal_manipulation_interface)) | Both fingers must be visible; less precise; fails under occlusion or motion blur; tags appear in training images | Printed-copy fallback only: first choice there, if the camera sees the fingers (was Recommended first) |
| Magnetic encoder (AS5600 module + Raspberry Pi Pico; TacUMI uses an MT6835 per [TouchGuide](https://arxiv.org/pdf/2601.20239)) | Precise; independent of view and lighting; low-cost parts (AS5600 board €8.95) | Small electronics; play in printed gears must be calibrated | Printed-copy fallback only, if markers are not visible or not precise enough (was Recommended) |
| Dex1-1 motor encoder (15-bit, [Unitree](https://unitree.com/Dex1-1)) | No extra parts; the handheld gripper is the real Dex1-1 on the station Jetson, which already reads its state; same sensor as on the robot | Angle-to-width mapping unmeasured; only if the real grippers serve as handhelds | **Chosen** 2026-11-19; was Fallback |

## Decision

The gripper width comes from the Dex1-1's own motor encoder: the station Jetson runs `dex1_1_service`, sends the trigger command and reads the gripper state (`q`) back, which is converted to fingertip width with the calibrated angle-to-width mapping ([[D09 Calibration]], Dex1-1 command → fingertip width). ArUco markers or a magnetic encoder apply only if the printed-copy fallback is resumed ([[D01 Handheld gripper body]]).

## Evidence

Markers need no electronics and are used by [UMI](https://arxiv.org/pdf/2402.10329), [ViTaMIn-B](https://arxiv.org/html/2511.05858v1) and [HuMI](https://arxiv.org/html/2602.06643v2); an encoder is the precise fallback, as in the TacUMI hardware ([TouchGuide](https://arxiv.org/pdf/2601.20239)).

Real gripper (2026-11-19): the real Dex1-1 is the handheld ([[D01 Handheld gripper body]], 2026-11-16) and is driven by the station Jetson ([[D06 Recording and synchronisation]]). `dex1_1_service` publishes the motor state on `rt/dex1/<side>/state` with `q`, `dq` and `tau_est` ([[2026-11-13 dex1_1_service code and Pi 5 fit]]); the encoder is 15-bit absolute ([Unitree](https://unitree.com/Dex1-1)). The same sensor then measures width on the handheld and on the robot, and no tags appear in the training images. The angle-to-width curve is still to be measured on delivery ([[Waiting]], Dex1-1 delivery).

## Revisit if

The motor angle does not map repeatably to fingertip width (backlash, slip of the fingers on the object), or the state stream from `dex1_1_service` cannot be recorded at the needed rate. Printed-copy fallback: marker detection unreliable during grasps.

## History

- 2026-10-19: accepted (proposal approved)
- 2026-11-19: width source changed to the Dex1-1 motor encoder, read by the station Jetson through `dex1_1_service`; markers and magnetic encoder only for the printed-copy fallback (real Dex1-1 is the handheld since 2026-11-16 and sends its own state; Martyn, audit board finding 29)
