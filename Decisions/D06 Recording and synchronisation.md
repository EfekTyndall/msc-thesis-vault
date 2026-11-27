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

Station computer (2026-11-16, Path 2 of [[2026-11-13 Handheld computer and gripper connection]]): the single computer is a Jetson at the UR10e station (approved unit 10.36.1.124: Jetson AGX Orin Developer Kit, Ubuntu 22.04, GLIBCXX to 3.4.30, read over SSH 2026-11-16, [[2026-11-16 Station Jetson check]]; moved to the station 2026-11-17 with its own account, XRoboToolkit PC Service and SDK installed 2026-11-17, [[2026-11-17 Station Jetson setup and install plan]]). It runs `dex1_1_service` (RS-485 adapter plugged in there), the camera stream, the XRoboToolkit service and recording. The Pi 5 + lab PC split (Path 1) is not used for the handheld.

Sync method (2026-11-27, Martyn): each device's delay is measured once and stored as one number per device in the versioned calibration file; at export each stream is shifted by its number onto the camera frame times. The camera is matched relative to the PICO controller pose: the rigid handheld (camera bolted to the controller) is turned in front of a fixed marker board and the two rotation-speed curves are slid until they match, so the PICO's own delay and the headset clock offset never need to be known and no screen test is needed. The gripper gets its own number from UMI's gripper sine test (Robotiq 2F-85 possible now, Dex1-1 when it arrives). For the test the D435i is mounted on the handheld adapter rod with the team's D435i UR10e adapter (STEP) adapted to the rod. Strictly modular: one entry per device, the alignment step knows no device, so the Dex1-1 and another camera plug in by measuring their own number. The robot / VR teleop delay is a separate topic, not part of this decision.

## Evidence

[ViTaMIn-B](https://arxiv.org/html/2511.05858v1) removed its onboard Raspberry Pi in favour of a host PC for "improved ergonomics and reduced weight", one PC gives one clock, and latency compensation raised ViTaMIn-B's final-step success from 20% to 70%.

Station computer: `dex1_1_service` runs a 500 Hz serial loop and brakes when commands stop, so it must run on the computer the RS-485 adapter plugs into, at the station ([dex1_1_service @ ced3216](https://github.com/unitreerobotics/dex1_1_service/tree/ced3216290bb22dae356213993786beeaa7143ab), `main.cpp`); Jetson Orin NX is Unitree's tested platform and the supervisor's VR-teleop stack already runs XRoboToolkit on a Jetson ([[2026-11-13 Handheld computer and gripper connection]]). Approved by the supervisor on 2026-11-16 ([[2026-11-16 Yanmei]]). `unitree_sdk2` and `dex1_1_service` built and started on the approved unit without a gripper on 2026-11-18 ([[2026-11-17 Station Jetson setup and install plan]] §D); the 500 Hz loop with a gripper is untested.

Sync method (all code read, not run; details and line numbers in [[2026-11-27 Latency sync method comparison]]): measure once, store per device, shift at export is what [UMI](https://github.com/real-stanford/universal_manipulation_interface) (`umi.yaml` l.3-5, `sampler.py` l.131-151), DexUMI (`data_buffer.py` l.151-161) and ViTaMIn-B (`task_config.yaml` l.55-57) do. Camera vs pose by rotation matching: [ethz-asl/hand_eye_calibration](https://github.com/ethz-asl/hand_eye_calibration) `time_alignment.py` l.23-60, 103-180 (Furrer et al. 2017, not in bib); finer step in [Kalibr](https://github.com/ethz-asl/kalibr) (Furgale et al. 2013, not in bib). No paper, doc or forum reports a measured PICO controller pose delay (deep research 2026-11-27). Gripper: UMI `calibrate_gripper_latency.py`. Unverified: accuracy with a 30 Hz camera (about 33 ms steps before a finer step); effect of PICO pose prediction (Warburton et al. 2023, not in bib).

Rejected sync options: pairing by timestamp without measuring (FastUMI; wrong when devices have different delays); measuring the camera on its own (needs the PICO delay, which has no precedent); estimating end to end from each recording's motion (ExtremControl, device-specific, no code).

## Revisit if

Collection must span several stations, or cables are not allowed. Station computer: the station Jetson cannot hold the 500 Hz gripper loop → Path 1 (Pi 5 at the station + lab PC). (The earlier conditions are settled: it is an AGX Orin, checked 2026-11-16, and it stands at the station since 2026-11-17.)

Sync method: rotation matching cannot reach about 10 ms with the 30 Hz camera even with a finer step, or the PICO offset changes noticeably between movement start and steady motion.

## History

- 2026-10-19: accepted (proposal approved)
- 2026-11-16: station computer: one Jetson at the UR10e station runs gripper program, camera, XRoboToolkit and recording (Path 2); supervisor approved a second Jetson (10.36.1.124) for this; the Pi 5 is ordered for the team, not for the handheld (2026-11-16 talk, [[2026-11-16 Yanmei]]; reported by Martyn)
- 2026-11-19: station Jetson facts updated: model checked (AGX Orin, 2026-11-16), `dex1_1_service` built and started without a gripper (2026-11-18); decision unchanged (consistency audit, [[2026-11-19 Three-repo consistency audit]])
- 2026-11-22: Decision text: Jetson moved to the station (2026-11-17, own account, XRoboToolkit installed); Revisit if: settled conditions (not an Orin, cannot stand at the station) removed; decision unchanged (weekly consistency audit 2026-11-22, Martyn's board choice)
- 2026-11-27: sync method added: one delay number per device, shift at export to camera frame times (UMI, DexUMI, ViTaMIn-B); camera matched relative to the PICO pose by rotation at a fixed marker board (hand_eye_calibration, Kalibr); gripper by UMI sine test; D435i on the adapter rod via the adapted team adapter; robot/VR teleop delay separate (Martyn locked it after the prior-work comparison, [[2026-11-27 Latency sync method comparison]])
