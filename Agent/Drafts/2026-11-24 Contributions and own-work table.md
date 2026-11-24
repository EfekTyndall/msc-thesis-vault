---
date: 2026-11-24
topic: Thesis contributions list and built / adapted / reused table (draft for Martyn)
status: draft, living; Martyn decides the final wording
---
# Contributions and own-work table

Purpose: make Martyn's own work visible to both examiners (ABPO §22(1) "independently", §23(1) written grade justification) and feed the AI register ([[2026-11-24 AI use register]]). The introduction gets the numbered contributions; the methods chapter gets the table.

## Draft contributions (introduction)

Each contribution names the result that backs it. Status as of 2026-11-24.

| # | Contribution | Backed by | Status | Why it counts |
| --- | --- | --- | --- | --- |
| 1 | Controlled comparison of handheld and VR-teleoperation data for one policy (π0.5 default) on an industrial pick-and-place task: equal data, equal operator time, handheld plus about 10% teleop (conditions C1–C4 (VR baseline, handheld equal data, handheld equal time, handheld + 10% teleop)) | Success rates with Wilson intervals and Fisher's tests, 80 rollouts (thesis plan §3) | planned, Feb 2027 | DM-UMI (surendran2026) found naive mixing worse than handheld alone; a matched comparison is still open, so any outcome is a result |
| 2 | Error budget of a VR-controller-based handheld: controller pose error at the gripper tip and sync residual, measured on the UR10e | Error-budget measurements (thesis plan §4) | planned, Dec 2026 – Jan 2027 | Thesis plan §4: no UMI-style system reports these together |
| 3 | Operator workload and collection efficiency of handheld vs VR (NASA-TLX, demonstrations per hour, unusable share) | Metrics table (thesis plan §3) | planned, Jan 2027 | Research gaps note (2026-11-02), summary 2: no handheld paper measures workload |
| 4 | A handheld that carries the real Dex1-1 on a PICO 4 Ultra controller grip, so handheld and robot share the same gripper | CAD, print tests, mass and cost; replay with the Dex1-1 by mid-January (gate) | prototype printed; Dex1-1 not delivered | Design is Martyn's (sleeve, adapter, trigger mapping); no precedent found for this combination (unverified beyond the bib) |
| 5 | One recording, export and replay pipeline for both data sources with swappable robot, gripper and camera (working name SwapKit) | Same episode replayed on two grippers (Robotiq 2F-85 and Dex1-1); Replay Ladder RL1–RL3 (MuJoCo, URSim, real UR10e) passed | partly built | Supports the fairness rules; framed as the means, not the main claim (options note [[2026-11-24 SwapKit modular framework options]]) |

## Built / adapted / reused (methods chapter)

- **Built:** designed and made in this thesis.
- **Adapted:** existing work changed for this thesis (what was changed is named).
- **Reused:** used as is, cited.
- **AI-assisted:** Claude wrote or co-wrote it (details in the AI register). Martyn's part is in "Own work".

### Hardware

| Part | Category | Source | Own work | AI-assisted |
| --- | --- | --- | --- | --- |
| Grip sleeve around the PICO controller (PETG, two halves) | built | own CAD, Fusion | design, 20+ versions, print tests | CAD edits partly via the Fusion connector |
| Adapter rod + two-piece clamp ring to the Dex1-1 | built | own CAD | design, fit tests on the printed dummy | partly |
| PICO 4 Ultra Enterprise controller and headset | reused | PICO | — | no |
| Controller shape reference | reused | Thingiverse 7350019 scan, CC BY | used as reference geometry | no |
| Gun-shell reference for grip and trigger | reused (reference only) | Printables 1481698, CC BY 4.0 | not printed | no |
| Dex1-1 gripper | reused | Unitree | — | no |
| Dex1-1 dummy and shell for fit tests | adapted | Unitree STEP, simplified | shell generation, printing | script: yes |
| Robotiq 2F-85, D435i, UR10e | reused | lab | — | no |

### Software

| Part | Category | Source | Own work | AI-assisted |
| --- | --- | --- | --- | --- |
| XR input (controller pose, trigger) | reused | XRoboToolkit (zhao2025) | — | no |
| UR10e IK, arm controller, HOME_Q, base yaw | reused (outside dependency, pinned) | team fork of Unitree `xr_teleoperate` | — | no |
| Dex1-1 driver | reused | Unitree `dex1_1_service`, `unitree_sdk2` | build on the station Jetson | no |
| UR10e and H2 simulation models | reused | MuJoCo Menagerie (BSD-3), unitree_mujoco (BSD-3) | model check against the team URDF | check script: yes |
| Robot connection | reused | ur_rtde, URSim | — | no |
| Pose and trigger logging | built | dex1-1-handheld `software/xr/` | requirements, lab runs | yes |
| Replay in MuJoCo, URSim and on the real UR10e (Replay Ladder RL1–RL3) | built | dex1-1-handheld `software/sim/` | requirements, safety limits, all lab runs | yes |
| Recorder (raw MCAP) and LeRobot export | built (planned) | format choice D07 (data format) | decision, tests | expected |
| Trigger-to-gripper mapping and force cap | built (planned) | gaps G2 (mapping) and G9 (force limit) | design decisions, tests | expected |
| Controller-to-tip offset | adapted | D09 (calibration): fixed offset from the CAD assembly | CAD measurement | partly |
| Policy (π0.5 or GR00T) and training code | reused | model authors | training runs, settings | no |

### Method

| Part | Category | Source | Own work |
| --- | --- | --- | --- |
| Hypotheses H1–H3 and conditions C1–C4 | built | motivated by UMI (chi2024), HiFi-UMI (ai2026), ActiveUMI (zeng2025) | design of the study |
| Pose error measurement on the robot | adapted | Banaszczyk et al. (banaszczyk2024) | applied to the controller on the UR10e flange |
| Sync test | adapted | UMI (chi2024), ViTaMIn-B (li2025) | Replay Ladder RL5 (camera + recorder) test |
| Workload questionnaire | reused | NASA-TLX | — |
| Statistics (Wilson intervals, Fisher's test) | reused | standard methods | — |

## Open

- Contribution 4 "no precedent" is checked only against the bib; a wider search needs Martyn's OK.
- Contribution 5 depends on the SwapKit option Martyn chooses.
- Hardware AI-assisted entries ("partly") need Martyn's own estimate of what he did in Fusion himself.
