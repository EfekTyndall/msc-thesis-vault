---
date: 2026-10-19
status: accepted
tags: [decision]
---
## Context
How the gripper opening width is measured on the handheld. Source: `msc-thesis/docs/proposal.md` §5 (approved by Yanmei on 19 Oct 2026).

## Options
| Option | Strengths | Weaknesses | Verdict |
| --- | --- | --- | --- |
| ArUco markers on the fingers, used by [UMI](https://arxiv.org/pdf/2402.10329), [ViTaMIn-B](https://arxiv.org/html/2511.05858v1) and [HuMI](https://arxiv.org/html/2602.06643v2) | No electronics; existing detection code (OpenCV, [UMI](https://github.com/real-stanford/universal_manipulation_interface)) | Both fingers must be visible; less precise; fails under occlusion or motion blur; tags appear in training images | **Recommended first**, if the camera sees the fingers |
| Magnetic encoder (AS5600 module + Raspberry Pi Pico; TacUMI uses an MT6835 per [TouchGuide](https://arxiv.org/pdf/2601.20239)) | Precise; independent of view and lighting; low-cost parts (AS5600 board €8.95) | Small electronics; play in printed gears must be calibrated | **Recommended** if markers are not visible or not precise enough |
| Dex1-1 motor encoder (15-bit, [Unitree](https://unitree.com/Dex1-1)) | No extra parts | Only if the real grippers serve as handhelds | **Fallback** (swap option) |



## Decision
ArUco markers on the printed fingers if the Dex1-1 camera sees both fingers clearly; otherwise a magnetic encoder in the handheld. The check is done as soon as the gripper arrives.

## Evidence
Markers need no electronics and are used by [UMI](https://arxiv.org/pdf/2402.10329), [ViTaMIn-B](https://arxiv.org/html/2511.05858v1) and [HuMI](https://arxiv.org/html/2602.06643v2); an encoder is the precise fallback, as in the TacUMI hardware ([TouchGuide](https://arxiv.org/pdf/2601.20239)).

## Revisit if
Marker detection unreliable during grasps.

## History
- 2026-10-19: accepted (proposal approved)
