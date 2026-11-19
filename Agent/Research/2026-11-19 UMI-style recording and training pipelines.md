---
date: 2026-11-19
tags: [research, umi, recording, training, calibration]
---
# UMI-style recording and training pipelines

Question (Martyn, 2026-11-19): how do UMI and the UMI-style papers record, synchronise, store, filter, calibrate, export and train, so that the recorder (task 5), calibration steps, export and training setup of [[2026-11-19 Work not blocked by parts]] follow published practice?

Method: arXiv text of the bib papers chi2024 (UMI, plus its [GitHub README](https://github.com/real-stanford/universal_manipulation_interface)), li2025 (ViTaMIn-B), zeng2025 (ActiveUMI), surendran2026 (DM-UMI), zhaxizhuoma2025 (FastUMI), yin2026 (RAPID), zhao2025 (XRoboToolkit) and murooka2026 (RoboManipBaselines), read 2026-11-19. ViTaMIn-B, ActiveUMI and XRoboToolkit were read verbatim. For UMI, DM-UMI, FastUMI, RAPID and RoboManipBaselines only the first ~40 % of the text was verbatim; the rest came from a model summary, marked **[S]** (unverified until checked in the PDF). "–" = not stated in the text read. HiFi-UMI (ai2026) is no longer in `literature.bib` and was not read again.

## In short

- **Where and how to record:** ViTaMIn-B is closest to this thesis (VR controller on the handheld, cable to one host computer). It records every stream with its own timestamp, measures each sensor's delay and moves poses onto the camera frame times. UMI does the same alignment.
- **File format:** nobody shares a format. RAPID uses MCAP with nanosecond timestamps (the choice already in the proposal). FastUMI and RoboManipBaselines use HDF5. UMI keeps raw GoPro MP4 and trains from zarr.
- **Start and stop of a demo:** ViTaMIn-B uses a foot pedal. The others don't say.
- **Rejecting bad demos:** UMI checks whether the robot could reach the poses. FastUMI flags pose jumps and low tracking confidence. No paper reports how many demos were rejected.
- **Training:** 100–345 demos per task are typical. UMI trained on 4–8 data-centre GPUs **[S]**, but its code was tested on a single 24 GB RTX 3090 (README). ActiveUMI fine-tuned π0 for 50k iterations.

## By pipeline step

| Step | UMI (chi2024) | ViTaMIn-B (li2025) | ActiveUMI (zeng2025) | FastUMI (zhaxizhuoma2025) | DM-UMI (surendran2026) | Tools: RAPID, XRoboToolkit, RoboManipBaselines |
| --- | --- | --- | --- | --- | --- | --- |
| Sensors, rates | GoPro (fisheye 155°, IMU in the MP4), ArUco width (HD1–HD5); rates – | GoPro Hero 10; tactile cameras 640×480 at 30 fps (III-A); Quest 3 controller pose at 72 Hz (IV-B); ArUco width (IV-A) | Quest 3 controllers + fisheye wrist cameras; "All sensor and robot data is collected at a frequency of 30Hz" (4.1) | GoPro 1920×1080 at 60 fps; T265 tracker about 200 Hz (III-A) | XVISIO DS80 SLAM camera, magnetic width sensors (III); rates – | XRoboToolkit: tracking JSON at 90 Hz (II-B); its demo dataset at 50 FPS (IV-B) |
| Recording computer | The GoPro itself | Wired "host computer", onboard Pi removed (IV-A, V-D1) | Small computer worn on the back (3.1) | – (ROS nodes) | – | – |
| Raw format | MP4 per demo **[S]** | – | – | HDF5 written by a ROS storage node (III-A) | – | RAPID: MCAP, `timestamp_ns` (App. A, D **[S]**); RoboManipBaselines: HDF5 + MP4 per episode (IV-B) |
| Time alignment | Streams aligned to the slowest (camera); poses and width linearly interpolated at the image timestamps (PD1.1) | Delay measured per sensor: Quest by a sync signal, cameras by a time-varying ArUco marker on a screen; GoPro 0.14 s, tactile 0.08 s, Quest 0.01 s (IV-B, V-D1); poses interpolated onto the frame times; target 10 ms (I); residual – | "synchronous, low-latency" (3.1); method – | One ROS clock for all streams, sub-sampled to 20 Hz, nearest pose per frame (III-A) | – | RAPID: approximate sync window, 25 ms default (III-C); XRoboToolkit: only video delay, 82–100.5 ms (IV-A) |
| Demo start/stop | One video per demo **[S]** | Foot pedal (IV-A) | – | – | – | – |
| Rejection | Kinematic feasibility (HD6); README example: 0 dropped demos | "Valid rate" 1.0 VR vs 0.16 SLAM (Tab. IV), criteria – | – | Low-confidence poses dropped and interpolated; velocity, acceleration and orientation thresholds (III-B) | – | RAPID keeps partial episodes with a per-frame mask (III-C) |
| Calibration | Gripper width range from 5 open/close cycles **[S]**; camera-IMU with OpenImuCameraCalibrator (README); no robot calibration (HD1) | Hand-eye: controller on the arm, 10 poses, solve controller → EE (IV-D); left- to right-handed frame | World origin by button or docking jig (3.3) | Offsets "known"; T265 frame aligned to the base (III-C), method – | – | – |
| Training data | zarr replay buffer (README `07_generate_replay_buffer.py`) | Relative EE actions via the hand-eye transform (IV-D); format – | Absolute world frame (3.3) | HDF5 → zarr **[S]**; absolute TCP, relative TCP or joints (III-C) | Relative EE + absolute width, horizon 24 **[S]** | – |
| Model, demos, compute | Diffusion Policy; 250–305 demos per lab task (V-A, V-B, **[S]**); 4–8 A10G or 8 A100 **[S]**; README: single RTX 3090 24 GB tested | Diffusion Policy (DDIM), 123–345 demos per task (Tab. I); GPU – | π0, 50k iterations, cosine schedule (4.1); GPU – | ACT variants, Diffusion Policy; 10,000 sequences over 22 tasks | Diffusion-based; 100–290 handheld demos per task **[S]** | – |
| Deployment delay | Actions sent ahead by the execution delay (arm 100 ms, gripper 120 ms, Fig. 5); tossing 0.875 with vs 0.575 without matching (Fig. 8) | – | – | – | Commands interpolated to 100 Hz **[S]** | – |

## What this means for the thesis pipeline (options, not decided)

| Item | Current plan | Precedent | Note |
| --- | --- | --- | --- |
| Recording computer | Station Jetson by cable ([[D06 Recording and synchronisation]]) | ViTaMIn-B (wired host, Pi removed) | Matches |
| Raw format | MCAP with all original timestamps (`msc-thesis/docs/proposal.md`) | RAPID (MCAP, ns timestamps); HDF5 in FastUMI and RoboManipBaselines | MCAP has one precedent; HDF5 is the more common choice |
| Alignment | One clock; latency measured and compensated to about 10 ms (D06) | UMI and ViTaMIn-B interpolate poses onto camera frame times; ViTaMIn-B's on-screen ArUco method | Adopt UMI/ViTaMIn-B interpolation at frame times |
| Export rate | 30 Hz (`msc-thesis/CLAUDE.md`) | ActiveUMI 30 Hz with π0; FastUMI 20 Hz; UMI policy 10–20 Hz **[S]** | 30 Hz has a π0-family precedent |
| Demo start/stop | Not decided | Foot pedal (ViTaMIn-B) | A controller button is also possible; no precedent found for it as start/stop |
| Rejection criteria (gap A6) | Not defined | UMI (reachability), FastUMI (pose jumps, low confidence), RAPID (masks) | Write them before collection; G6 timestamp gaps fit FastUMI-style checks |
| Calibration | [[D09 Calibration]]: versioned files, three cadences | ViTaMIn-B hand-eye (controller on the arm, 10 poses); UMI width range from open/close cycles | Width: the Dex1-1 rad → width curve replaces UMI's marker range |
| Actions | Relative EE + absolute width ([[D07 Software and data format]]) | UMI, ViTaMIn-B, DM-UMI **[S]** | Matches |
| Demos per task | Thesis plan: N not fixed | 100–345 per task in UMI, ViTaMIn-B, DM-UMI | Sets the scale of collection time |
| Training GPU | fractal-pc RTX 3080 Ti 12 GB | UMI code tested on a 24 GB RTX 3090; openpi LoRA > 22.5 GB ([[2026-11-19]]) | 12 GB is below every stated training setup |

## Open

- [S] items (UMI appendix, DM-UMI tables, FastUMI V-A to V-C, RAPID appendices): check in the PDFs before citing numbers.
- No paper reports the residual sync error or the share of rejected demos (already gaps A5, A6 in [[2026-11-02 Research gaps]]).
- Demo start/stop method and rejection criteria need a decision; the raw format (MCAP) is in the approved proposal.
