---
date: 2026-10-19
status: accepted
tags: [decision]
---
## Context

How handheld episodes are recorded and exported so they match the VR teleop data. Source: `msc-thesis/docs/proposal.md` §7 (approved by Supervisor on 19 Oct 2026).

## Options

| Option | Strengths | Weaknesses | Verdict |
| --- | --- | --- | --- |
| New recorder writing the teleop format | Fits the chosen sensors; one format for both data sources | Built from scratch | **Recommended** (format to be confirmed with the teleop stack) |
| Adapting the [Grabette](https://huggingface.co/blog/grabette) open-source stack | Recorder, processing and LeRobot export already exist | Built around its own cameras and SLAM | **Fallback** / reference implementation |
| Porting the [UMI pipeline](https://github.com/real-stanford/universal_manipulation_interface) | Reference for latency handling and data filtering | Tied to GoPro and SLAM; most stages unused | Not advised |

## Decision

A new recorder that writes episodes directly in the same format as the VR teleop pipeline, using the action representation of [UMI](https://arxiv.org/pdf/2402.10329).

Actions and state (2026-11-18): end-effector actions relative to the current gripper pose plus absolute gripper width, and the policy's pose (state) input also relative (to the current pose or episode start) or left out. Handheld poses are then never needed in the robot base frame, so no per-session room-to-robot-base calibration is needed for training data. The export layer still stores absolute poses (raw tracker frame kept); the controller-to-gripper-tip transform from [[D09 Calibration]] is still applied. Same representation for handheld and VR teleop data.

Training (2026-11-19): π0.5 (openpi) is the first policy; plan 100–300 demonstrations per task and condition; fine-tuning on a GPU with at least 24 GB (fractal-pc's 12 GB RTX 3080 Ti is too small; a larger team PC is being asked for).

## Evidence

One format for handheld and teleop data is what makes the comparison possible, and the [UMI code](https://github.com/real-stanford/universal_manipulation_interface) is tied to the GoPro and SLAM, which this design does not use.

Relative state and actions (2026-11-18): [UMI](https://arxiv.org/pdf/2402.10329) uses relative actions and relative proprioception and needs no camera-robot-world calibration; its absolute-action baseline did worse, blamed on SLAM-to-robot calibration error. [HiFi-UMI](https://arxiv.org/pdf/2607.25895) trains π0.5 on relative pose increments with absolute gripper. The PICO room frame is re-made at each app start or recentre, so absolute handheld poses would need a calibration every session. GR00T N1.7 and openpi convert absolute exports to relative actions themselves but feed the absolute pose as state by default; GR00T N1.7 can zero the state input with its `exclude_state` setting while still converting actions to relative; openpi has no such flag, so relative actions and zero or relative state come from an own input transform (not tried). Details: [[2026-11-17 Controller pose to robot frame]], [[2026-11-18 GR00T and pi0.5 action format for handheld data]], [[2026-11-18 UMI variants pose tracking and policy actions]], [[2026-11-18 Relative vs absolute actions decision draft]].

Training (2026-11-19): π0.5 is the thesis plan's default policy family (`msc-thesis/docs/thesis-plan.md` §2); ActiveUMI fine-tuned the related π0 on VR-controller handheld data for 50k iterations (zeng2025, 4.1). Demonstrations per task in UMI-style work: UMI 250–305 (chi2024, V-A, V-B; partly from a summary, unverified), ViTaMIn-B 123–345 (li2025, Tab. I), DM-UMI 100–290 handheld (surendran2026, summary, unverified). GPU: openpi README requires > 22.5 GB for LoRA fine-tuning and > 70 GB for full fine-tuning ([openpi](https://github.com/Physical-Intelligence/openpi)); UMI's code was tested on a single 24 GB RTX 3090 ([UMI README](https://github.com/real-stanford/universal_manipulation_interface)). Details: [[2026-11-19 UMI-style recording and training pipelines]].

## Revisit if

GR00T N1.7 or openpi cannot train with relative or no end-effector state → transform handheld poses into the robot base with a per-session room-to-base calibration (option A of the draft).
Training: no GPU with ≥ 24 GB becomes available, or the pilot run shows far more or fewer demonstrations are needed.

## History

- 2026-10-19: accepted (proposal approved)
- 2026-11-18: actions and state made relative (relative EE actions + absolute width; relative or no pose state), so no per-session room-to-base calibration (Martyn; PICO room frame changes every session, UMI calibration-free and better than its absolute baseline)
- 2026-11-19: training: π0.5 first, 100–300 demonstrations per task, GPU ≥ 24 GB (Martyn; UMI-style survey, openpi README)
