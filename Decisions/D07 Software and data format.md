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

Raw format and export (2026-11-23): every stream is recorded raw in MCAP with its original nanosecond timestamps; alignment is done offline from the raw files; training data is exported to LeRobot. No HDF5 step in between.

Training (2026-11-19): π0.5 (openpi) is the first policy; plan 100–300 demonstrations per task and condition; fine-tuning on a GPU with at least 24 GB (fractal-pc's 12 GB RTX 3080 Ti is too small; a larger team PC is being asked for).

Framework, SwapKit (2026-11-24): one thin framework with three modes (teleoperation, recording, replay), real and simulated, where robot, gripper, camera and input device are swappable plugs chosen in a settings file (option A of [[2026-11-24 SwapKit modular framework options]]). Built inside the Replay Ladder, no separate phase: RL4 (Robotiq pick-and-place) builds the gripper plug, RL5 (camera + recorder) the camera plug and the recorder, RL6 (swap to the Dex1-1) is the first real plug swap. The team stack stays the UR10e (and later H2) driver as an outside dependency at a pinned commit. Built and proven on the UR10e; the H2 stays a stretch goal. No LeRobot dependency (option B rejected for now).

## Evidence

One format for handheld and teleop data is what makes the comparison possible, and the [UMI code](https://github.com/real-stanford/universal_manipulation_interface) is tied to the GoPro and SLAM, which this design does not use.

Relative state and actions (2026-11-18): [UMI](https://arxiv.org/pdf/2402.10329) uses relative actions and relative proprioception and needs no camera-robot-world calibration; its absolute-action baseline did worse, blamed on SLAM-to-robot calibration error. [HiFi-UMI](https://arxiv.org/pdf/2607.25895) trains π0.5 on relative pose increments with absolute gripper. The PICO room frame is re-made at each app start or recentre, so absolute handheld poses would need a calibration every session. GR00T N1.7 and openpi convert absolute exports to relative actions themselves but feed the absolute pose as state by default; GR00T N1.7 can zero the state input with its `exclude_state` setting while still converting actions to relative; openpi has no such flag, so relative actions and zero or relative state come from an own input transform (not tried). Details: [[2026-11-17 Controller pose to robot frame]], [[2026-11-18 GR00T and pi0.5 action format for handheld data]], [[2026-11-18 UMI variants pose tracking and policy actions]], [[2026-11-18 Relative vs absolute actions decision draft]].

Training (2026-11-19): π0.5 is the thesis plan's default policy family (`msc-thesis/docs/thesis-plan.md` §2); ActiveUMI fine-tuned the related π0 on VR-controller handheld data for 50k iterations (zeng2025, 4.1). Demonstrations per task in UMI-style work: UMI 250–305 (chi2024, V-A, V-B; partly from a summary, unverified), ViTaMIn-B 123–345 (li2025, Tab. I), DM-UMI 100–290 handheld (surendran2026, summary, unverified). GPU: openpi README requires > 22.5 GB for LoRA fine-tuning and > 70 GB for full fine-tuning ([openpi](https://github.com/Physical-Intelligence/openpi)); UMI's code was tested on a single 24 GB RTX 3090 ([UMI README](https://github.com/real-stanford/universal_manipulation_interface)). Details: [[2026-11-19 UMI-style recording and training pipelines]].

Raw format (2026-11-23): MCAP stores many channels per file with uint64 ns `log_time` and `publish_time` per message ([MCAP spec](https://mcap.dev/spec)), so streams at different rates keep their own timing; HDF5 needs an own layout for this, and FastUMI resamples to 20 Hz at record time, losing the raw timing (zhaxizhuoma2025, III-A). MCAP is append-only "to reduce the risk of data corruption during unclean shutdowns" ([mcap.dev](https://mcap.dev/)); for HDF5 no guarantee for an unclosed file is documented, crash corruption is reported ([HDF forum 3481](https://forum.hdfgroup.org/t/file-state-after-flush-and-crash/3481)), and SWMR cannot append variable-length data ([HDF Group SWMR intro](https://support.hdfgroup.org/documentation/hdf5-docs/advanced_topics/intro_SWMR.html)). Precedent: RAPID records MCAP with ns timestamps (yin2026); HDF5 is more common (FastUMI, RoboManipBaselines murooka2026) but stores already-aligned data. Raw storage lets alignment be redone when latencies are re-measured (UMI PD1.1 and ViTaMIn-B IV-B align offline by measured delays; [[2026-11-23 RL5 sync test findings]]). openpi and GR00T read LeRobot datasets, so an HDF5 intermediate adds nothing. Already in `msc-thesis/docs/proposal.md` (raw MCAP). Details: [[2026-11-19 UMI-style recording and training pipelines]] (MCAP vs HDF5).

Framework (2026-11-24): the split into one interface per device kind, with real and simulated robots behind the same interface, follows RoboManipBaselines (murooka2026, IV-A: new robot = inherit one class, override a few methods) and LeRobot (`Robot`, `Teleoperator` classes; `lerobot-teleoperate`, `lerobot-record`, `lerobot-replay`; [LeRobot docs](https://huggingface.co/docs/lerobot/il_robots), not in bib). XRoboToolkit (zhao2025) is already the input layer. Unitree `xr_teleoperate` supports the H2 and the Dex1-1 since v1.6 (2026-07-29, [README](https://github.com/unitreerobotics/xr_teleoperate)); whether the team's pinned fork includes it is unverified. RAPID (yin2026) and DM-UMI (surendran2026) are the closest precedents for handheld and robot data in one system. Building inside the Replay Ladder costs no extra lab time and fixes the plug interfaces before the Dex1-1 arrives. Details: [[2026-11-24 SwapKit modular framework options]].

## Revisit if

Raw format: the MCAP writer cannot keep up on the station Jetson, or a training framework needs a format LeRobot export cannot give.
GR00T N1.7 or openpi cannot train with relative or no end-effector state → transform handheld poses into the robot base with a per-session room-to-base calibration (option A of the draft).
Training: no GPU with ≥ 24 GB becomes available, or the pilot run shows far more or fewer demonstrations are needed.

Framework: building the plugs delays the mid-January replay gate, or the H2 becomes reachable early enough to test portability (then option C, measured portability test, in the March buffer).

## History

- 2026-10-19: accepted (proposal approved)
- 2026-11-18: actions and state made relative (relative EE actions + absolute width; relative or no pose state), so no per-session room-to-base calibration (Martyn; PICO room frame changes every session, UMI calibration-free and better than its absolute baseline)
- 2026-11-19: training: π0.5 first, 100–300 demonstrations per task, GPU ≥ 24 GB (Martyn; UMI-style survey, openpi README)
- 2026-11-23: raw recording in MCAP (all streams, original ns timestamps), offline alignment, LeRobot export, no HDF5 step (Martyn; multi-rate timing, append-only crash safety, alignment can be redone; RAPID precedent)
- 2026-11-24: framework SwapKit, option A: thin plug framework (robot, gripper, camera, input; modes teleoperation, recording, replay) built inside the Replay Ladder, team stack as outside dependency, H2 stays stretch (Martyn; least effort, fixes interfaces before the Dex1-1 arrives, RL6 becomes the first swap; RoboManipBaselines and LeRobot precedents)
