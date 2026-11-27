---
date: 2026-11-27
tags: [agent, report]
---
# Sources to add to Zotero

Collected 2026-11-27 from the "Add to Zotero" and "not in bib" lines of the vault notes. Checked against `msc-thesis/thesis/literature.bib` on `main` (fb1d2a9): HandUMI, RUMI, YUBI, Robot-Matched Soft Fingers, gripper-design study, LEGATO, Mirage, DexUMI, DM-UMI, LieSpline-DP and Banaszczyk are already in the bib and are not listed. Martyn adds these in Zotero; `literature.bib` is never edited by hand.

## Papers: sync method (D06, recording and synchronisation) — add first
| Title | Link | Why | Note |
|---|---|---|---|
| Warburton et al. 2023, VR controller latency | doi 10.3758/s13428-022-01983-5 | Delay differs at movement start (21-42 ms) and in steady motion (2-13 ms) | [[2026-11-27 Latency sync method comparison]] |
| ExtremControl | arXiv 2602.11321 | Cross-correlation with parabola step for time offset | same |
| HuMiT | arXiv 2610.29903 | Closest PICO 4 Ultra end-to-end latency number (65 ms median) | same |
| Furrer et al. 2017, hand_eye_calibration (code ethz-asl/hand_eye_calibration) | not looked up | Time alignment by rotation speed; basis of the camera-to-PICO match | same |
| Furgale, Rehder, Siegwart 2013, Kalibr: unified temporal and spatial calibration | https://doi.org/10.1109/IROS.2013.6696514 | Time offset by matching motion; fine continuous step | [[2026-11-25 RL5 camera and recorder plan]] |
| Friston and Steed 2014 | not looked up | VR latency measurement | [[2026-11-27 Latency sync method comparison]] |
| VRLate | arXiv 1809.06320 | VR latency measurement tool | same |
| Hu 2025 | arXiv 2508.08642 | Latency study | same |
| Magic Leap 2 study | arXiv 2509.05391 | Controller latency study | same |

## Papers: operator effort and teleop (PICO Motion Tracker note, only if used)
| Title | Link | Why | Note |
|---|---|---|---|
| Gholami et al. 2021 | arXiv 2105.09809 | Objective ergonomics of teleop interfaces plus NASA-TLX | [[2026-11-06 PICO Motion Tracker uses]] |
| Moyen et al. 2025 | arXiv 2509.03222 | RULA score of the upper arm in whole-body teleop | same |
| Teleopit | arXiv 2608.01834 | Headset plus ankle trackers | same |

## Papers: data formats and policies (only if the thesis text cites them)
ACT (Zhao et al. 2023), robomimic, Diffusion Policy (Chi 2023; the bib has 3D Diffusion Policy, not this), Open X-Embodiment, LeRobot. Source: [[2026-11-23 Robot data formats]]. Also Lee et al. 2009 (grip span) and McAtamney and Corlett 1993 (RULA): cited in notes, not checked.

## Code repositories and manuals (cite as software or online sources)
UMI (real-stanford/universal_manipulation_interface), DexUMI, ViTaMIn-B, FastUMI repositories (named in [[D06 Recording and synchronisation]]); ethz-asl/hand_eye_calibration and ethz-asl/kalibr; Robotiq 2F-85 manual; librealsense `rs_frame.h`; actuated-umi-gripper (https://github.com/actuated-umi/actuated-umi-gripper); HandUMI hardware and software repos (murobotics-ai). Check which of these the thesis text will actually cite.
