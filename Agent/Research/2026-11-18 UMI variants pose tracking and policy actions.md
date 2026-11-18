---
date: 2026-11-18
tags: [research, umi, tracking, actions, literature]
---
# UMI variants: pose tracking and policy actions

Question (Martyn, 2026-11-18): how does the base UMI track the handheld pose and use it for training, and how do the UMI follow-ups do it, especially those that switched to VR controllers (ActiveUMI)? Follows [[2026-11-17 Controller pose to robot frame]] and [[2026-11-18 GR00T and pi0.5 action format for handheld data]].

Method: arXiv HTML of the bib papers listed below, read 2026-11-18. For UMI, FastUMI, HiFi-UMI, UMI-FT, UMI-3D and ViTaMIn-B only the first ~40k characters were read verbatim; the rest through a model summary, marked (summary). ActiveUMI was read in full. The UMI SLAM appendix was not read.

## Base UMI (chi2024)

1. **Tracking**: GoPro wrist camera + its built-in IMU, inertial-monocular SLAM "based on ORB-SLAM3" (Sec. III-A, HD4). A mapping video is recorded per scene first (summary); all poses are relative to that map (first frame of the mapping video, summary).
2. **Gripper width**: ArUco markers on the fingers, seen by the same camera (HD5).
3. **No robot-base calibration**: "does not require camera-robot-world calibration" (HD1). An absolute-action baseline that did calibrate SLAM and robot to shared markers did worse, blamed on calibration error (Sec. V-A, summary).
4. **Training**: Diffusion Policy (Sec. III-B). Observation = wrist RGB + EE pose + width; proprioception relative to the current pose (summary). Action = "desired relative EE pose and gripper width" (Fig. 5), i.e. a future trajectory expressed relative to the current gripper pose. Latency matching: observations aligned to the slowest stream, actions sent ahead of time (PD1).

## Follow-ups

| Paper | Pose tracking | Width | Frame / robot calibration | Action representation | Model |
| --- | --- | --- | --- | --- | --- |
| FastUMI, zhaxizhuoma2025 | RealSense T265 (visual-inertial, on-board) | ArUco | T265 frame "aligned with the robot base frame" (procedure not described, summary) | ACT: absolute; DP: relative TCP trajectory (summary) | DP, ACT variants |
| UMI-FT, choi2026 | iPhone 15 Pro ARKit, 60 Hz | ArUco | Not stated | 9D poses (xyz + 6D rotation); absolute/relative not stated | Adaptive Compliance Policy (diffusion) |
| UMI-3D, wang2026 | Wrist LiDAR (Livox MID-360) + IMU, LiDAR-inertial odometry | ArUco | First IMU frame; robot calibration not stated | Relative SE(3) to current EE pose (summary) | Diffusion policy |
| HiFi-UMI, ai2026 | Head-mounted stereo-inertial SLAM (offline) + marker cubes on the hands | Encoders (angle) | Hand poses in head-camera frame (summary); robot calibration not stated | Relative pose increments, absolute gripper (summary) | incl. **OpenPI π0.5** |
| DM-UMI, surendran2026 | XVISIO DS80 SLAM camera | Magnetic sensors | Not stated | Relative EE + absolute width | (see paper) |
| TacUMI cheng2026, RDT2 liu2026 | Vive Tracker 3.0 + base stations | — | Fixed tracker → TCP transform; base calibration not stated | — | — |
| **ViTaMIn-B, li2025** | **Meta Quest 3 controller** on the handheld (headset not described) | Tactile/camera (not checked) | Hand-eye: controller mounted on the arm, 10 poses, solve controller → EE (Sec. IV-D); Quest left-handed → right-handed | **Relative**: EE pose change from the previous step | Diffusion Policy |
| **ActiveUMI, zeng2025** | **Meta Quest 3s** headset (worn) + controllers, inside-out | Micro-motor on controller | "absolute coordinates relative to a unified world coordinate system" set by 'B' button or a docking jig (Sec. 3.3); world → robot base not stated | Absolute recording; policy action form not stated; also predicts head pose | **π0**, 50k iterations |

## Pattern

- SLAM / VIO trackers (UMI, FastUMI, UMI-3D, HiFi-UMI, DM-UMI) and VR controllers (ViTaMIn-B, ActiveUMI) all give poses in a tracker-defined world frame, never the robot base directly.
- Most train on **relative** actions (UMI, UMI-3D, HiFi-UMI, DM-UMI, ViTaMIn-B, FastUMI-DP), which removes the need for a world → robot-base calibration.
- Those that keep absolute poses calibrate once per session: FastUMI aligns to the base (method unstated), ViTaMIn-B solves hand-eye (but then trains relative), ActiveUMI resets the origin with a jig.
- Closest to this thesis (VR controller + VLA): ActiveUMI (π0, absolute world frame, jig reset) and HiFi-UMI (π0.5, relative actions).

## Open
- ActiveUMI: how the jig-defined world origin is placed relative to the robot at deployment is not stated.
- UMI SLAM appendix (map building, scale, relocalisation) not read verbatim.
