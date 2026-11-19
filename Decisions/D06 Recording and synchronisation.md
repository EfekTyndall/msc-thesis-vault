---
date: 2026-10-19
status: accepted
tags: [decision]
---
## Context

Where data is recorded and how all streams share one time base. Source: `msc-thesis/docs/proposal.md` §6 (approved by Supervisor on 19 Oct 2026).

## Options

| Option                                                                                                                                           | Strengths                                                                                                                          | Weaknesses                                                                                                                         | Verdict                                                                     |
| ------------------------------------------------------------------------------------------------------------------------------------------------ | ---------------------------------------------------------------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------- | --------------------------------------------------------------------------- |
| Tethered PC, one clock, measured latency                                                                                                         | Light handheld; one clock; unlimited recording headroom; 10 ms alignment target ([ViTaMIn-B](https://arxiv.org/html/2511.05858v1)) | Cable drag and tangling; connector wear                                                                                            | **Recommended** for a fixed workstation                                     |
| Backpack PC with battery ([ActiveUMI](https://arxiv.org/pdf/2510.01607))                                                                         | Same software; untethered from the station                                                                                         | Battery handling; worn weight                                                                                                      | **Fallback** if collection spans several stations or cables are not allowed |
| Raspberry Pi 5 or Jetson per handheld ([Grabette](https://huggingface.co/blog/grabette), [TacUMI](https://arxiv.org/pdf/2601.20239)-style board) | Fully self-contained handheld                                                                                                      | No hardware video encoder; extra weight and a battery on each handheld; three clocks to synchronise; Wi-Fi risk on a factory floor | Last resort                                                                 |
| Hardware trigger ([HiFi-UMI](https://arxiv.org/pdf/2607.25895): < 40 µs; [UMI-3D](https://arxiv.org/pdf/2604.14089): STM32 clock)                | Most precise                                                                                                                       | Needs a triggerable camera; mainly for fast, dynamic tasks                                                                         | Not needed for pick-and-place                                               |

## Decision

No onboard computer as the first option. Each handheld connects by one cable to a single PC, which is the only clock; every sensor's latency is measured and compensated to a residual of about 10 ms or less. Whether the industrial use case is one fixed station or a moving operator is still open, so an onboard or backpack computer remains the fallback.

Station computer (2026-11-16, Path 2 of [[2026-11-13 Handheld computer and gripper connection]]): the single computer is a Jetson at the UR10e station (approved unit 10.36.1.124: Jetson AGX Orin Developer Kit, Ubuntu 22.04, GLIBCXX to 3.4.30, read over SSH 2026-11-16, [[2026-11-16 Station Jetson check]]; not yet moved to the station). It runs `dex1_1_service` (RS-485 adapter plugged in there), the camera stream, the XRoboToolkit service and recording. The Pi 5 + lab PC split (Path 1) is not used for the handheld.

## Evidence

[ViTaMIn-B](https://arxiv.org/html/2511.05858v1) removed its onboard Raspberry Pi in favour of a host PC for "improved ergonomics and reduced weight", one PC gives one clock, and latency compensation raised ViTaMIn-B's final-step success from 20% to 70%.

Station computer: `dex1_1_service` runs a 500 Hz serial loop and brakes when commands stop, so it must run on the computer the RS-485 adapter plugs into, at the station ([dex1_1_service @ ced3216](https://github.com/unitreerobotics/dex1_1_service/tree/ced3216290bb22dae356213993786beeaa7143ab), `main.cpp`); Jetson Orin NX is Unitree's tested platform and the supervisor's VR-teleop stack already runs XRoboToolkit on a Jetson ([[2026-11-13 Handheld computer and gripper connection]]). Approved by the supervisor on 2026-11-16 ([[2026-11-16 Yanmei]]). `unitree_sdk2` and `dex1_1_service` built and started on the approved unit without a gripper on 2026-11-18 ([[2026-11-17 Station Jetson setup and install plan]] §D); the 500 Hz loop with a gripper is untested.

## Revisit if

Collection must span several stations, or cables are not allowed. Station computer: the approved Jetson turns out not to be an Orin (JetPack 4 lacks GLIBCXX 3.4.26), cannot stand at the station, or cannot hold the 500 Hz gripper loop → Path 1 (Pi 5 at the station + lab PC).

## History

- 2026-10-19: accepted (proposal approved)
- 2026-11-16: station computer: one Jetson at the UR10e station runs gripper program, camera, XRoboToolkit and recording (Path 2); supervisor approved a second Jetson (10.36.1.124) for this; the Pi 5 is ordered for the team, not for the handheld (2026-11-16 talk, [[2026-11-16 Yanmei]]; reported by Martyn)
- 2026-11-19: station Jetson facts updated: model checked (AGX Orin, 2026-11-16), `dex1_1_service` built and started without a gripper (2026-11-18); decision unchanged (consistency audit, [[2026-11-19 Three-repo consistency audit]])
