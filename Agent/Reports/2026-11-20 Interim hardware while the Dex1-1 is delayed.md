---
date: 2026-11-20
tags: [report, decision-brief, planning]
---
# Interim hardware while the Dex1-1 is delayed

Question (Martyn, 2026-11-20): the manager now expects the Dex1-1 grippers and the cameras ordered with them (Dex1-1 RGB camera, D405) in about 40 days to 5–6 weeks, i.e. around 25 Dec 2026 – 1 Jan 2027 (computed from 20 Nov). Which interim path keeps the thesis moving, and what does each cost when the Dex1-1 arrives?

Method: lab hardware and plan from `msc-thesis/docs/thesis-plan.md` (§3–§6), `msc-thesis/docs/hardware-specs.md`, [[D01 Handheld gripper body]], [[D02 Policy camera]], [[D03 Depth sensing]], [[Waiting]], [[2026-11-19 Work not blocked by parts]], [[2026-11-12 Lab computers and gripper connections]]. Hardware facts of prior work read from the arXiv text of the bib papers chi2024 (UMI), zhaxizhuoma2025 (FastUMI), liu2026 (RDT2), yin2026 (RAPID), zeng2025 (ActiveUMI), murooka2026, cheng2026 (TacUMI), li2025 (ViTaMIn-B), ai2026 (HiFi-UMI) and xue2026 (fisheye study, arXiv 2603.02139), plus the Robotiq 2F-85 manual, on 2026-11-20. Items marked **[S]** came from a model summary of the paper's later part and are unverified until checked in the PDF. Nothing was decided; no decision note was changed.

## In short

- **Recommended: option B, a "pipeline rig".** Mount the team D435i on the existing handheld prototype, use the lab's Robotiq 2F-85 on the UR10e for replay, and build recorder → export → open-loop replay now. All of it carries over to the Dex1-1 unchanged except the gripper driver and the camera. Data from it are **pilot data only**.
- **Decision gate:** if the Dex1-1 has not arrived and passed the RS-485 check by **mid-January 2027** (the thesis plan's existing replay deadline), put option D (2F-85 for the whole core) to the supervisor.
- **Not recommended now:** a motorised Dex1-1 copy (E). It needs a new mechanism, motor and driver, which would take about as long as the delivery itself (estimate, unverified), and the robot side still needs a gripper.
- **D02 and D03 flags:** using the D435i as the *policy* camera would change D02 (Dex1-1 RGB camera). The D435i's colour view is narrow (69.4° × 42.5° × 77° ±3° per a quoted spec line, unverified; Dex1-1: 118° horizontal per D02). For FastUMI, first-person D435i teleop scored 0 % where the GoPro fisheye scored 80–100 % (Table V **[S]**). D03 (RGB only) stays as long as the D435i depth stream is not recorded or used.

## What the lab has now

| Item | Status | Source |
| --- | --- | --- |
| UR10e | Always free (Martyn, 2026-11-19) | [[2026-11-19 Work not blocked by parts]] |
| Robotiq 2F-85 on the UR10e (current VR-teleop gripper) | 85 mm opening (manual p.11); Modbus RTU over RS-485 (p.15), 115.2 kbit/s **[S]**; URCap for e-Series (p.29); relayed by the station Pi via `ser2net` | `msc-thesis/docs/hardware-specs.md`; [Robotiq manual](https://assets.robotiq.com/website-assets/support_documents/document/2F-85_2F-140_Instruction_Manual_e-Series_PDF_20190206.pdf) |
| RealSense D435i (UR10e wrist camera, VR teleop) | One unit known; planned to move to the station Jetson for camera tests ([[2026-11-19 Station Jetson camera install plan]]). A second unit is not known | hardware-specs; Martyn 2026-11-19 |
| VR-teleop stack | XRoboToolkit → UR10e + 2F-85 + D435i already works for the team | [[2026-11-12 Lab computers and gripper connections]] |
| Handheld prototype | PICO sleeve (v23/v22 + tape), real-adapter, printed dummy Dex1-1 shell | [[D10 Handheld mechanism]] |
| Station Jetson | Path 2 computer; D435i plan C1–C3 | [[D06 Recording and synchronisation]] |
| Printed Dex1-1 copy | Frozen fallback (link A modelled, bracket not started) | [[D01 Handheld gripper body]] |

## What prior work says about switching grippers and cameras

| Finding | Source |
| --- | --- |
| UMI uses "the same soft fingers" on the handheld and the deployed robot grippers (WSG-50 **[S]**), and the same GoPro position relative to them | chi2024, HD1; App. F.1 **[S]** |
| FastUMI fits fingertips "identical to those on the handheld device" to robot grippers, with adapters built for "the xArm gripper and Robotiq 2f-85" | zhaxizhuoma2025, Sec. II-C |
| RDT2: "As long as the same model of camera and gripper are installed", the policy transfers zero-shot; the handheld is a non-actuated replica of the robot gripper | liu2026, Fig. 1; App. A.1 **[S]** |
| ActiveUMI mounts "an identical copy" of the robot gripper on a Quest 3s controller, driven by a micro-motor | zeng2025, Sec. 3.1 |
| Changing the lens collapses a π0.5 policy (wide lens 0.0025 without augmentation) | xue2026, Tab. S10 **[S]** |
| D435i first-person view: 0 % vs GoPro fisheye 80–100 % (teleop, two tasks) | zhaxizhuoma2025, Table V **[S]** |
| Camera-only model trained on handheld data dropped from 0.76 to 0.23 on robot data ("substantial domain gap") | cheng2026, Tables I–II |
| Open motorised UMI gripper: Dynamixel XL430, MGN7C rails, belt (MIT licence) | [actuated-umi-gripper](https://github.com/actuated-umi/actuated-umi-gripper) (not in bib) |

Consequence for every option: **data from two different gripper/camera sets cannot be mixed in one condition** (RDT2, fisheye study). Whatever hardware the C1–C4 data are recorded with must be the same for teleop and handheld, and stay the same from the first to the last demonstration (thesis plan §3, fairness rules).

## Options

| | A. Wait (current plan) | **B. Pipeline rig (recommended)** | C. Interim training-grade rig | D. 2F-85 for the whole core | E. Motorised Dex1-1 copy |
| --- | --- | --- | --- | --- | --- |
| What it is | Software-only work from [[2026-11-19 Work not blocked by parts]]; no robot tests with a gripper | D435i on the existing prototype; trigger value as stand-in width; replay on the UR10e with the 2F-85; VR teleop on the existing 2F-85 stack | B plus FastUMI-style matched fingers: printed fingertips on the 2F-85 and a passive printed parallel handheld (UMI open hardware) with the same fingertips; one camera moved between robot and handheld | C, kept to the end; Dex1-1 only for the H2 stretch | Printed Dex1-1 copy (frozen fallback) plus a small motor, as handheld and possibly as robot gripper |
| Precedent | thesis plan §6 ("build the recorder with any USB camera") | thesis plan §6; recorder/replay steps are gripper-agnostic (UMI replays poses + width) | FastUMI (2F-85 fingertip adapter); UMI (same fingers on both) | FastUMI, UMI | ActiveUMI (micro-motor copy); actuated-umi (Dynamixel). No precedent found for a motorised copy *replacing* the robot's commercial gripper in a study |
| Unblocks now | Recorder skeleton, export, training stack on public data | Recorder with real images, sync/latency method, D09 offset procedure, **open-loop replay on the robot (Dec gate)**, VR baseline arm + gripper control, task protocol dry-run | B + a pilot policy trained end-to-end on own data | Everything, including real C1–C4 data from January | Handheld trigger → motor loop; robot side only if a second unit is built |
| Rework when the Dex1-1 arrives | None (work still to do) | Gripper driver (2F-85 Modbus → `dex1_1_service`), camera (D435i → Dex1-1 RGB), CAD offset file; pilot data discarded | As B, plus the fingertips and passive handheld are dropped; all pilot data discarded | None for the core; Dex1-1 work moves to the stretch | Copy dropped or kept as a second handheld; motor work discarded |
| Risk to the handheld vs VR comparison | None directly; schedule risk is the threat | Low: pilot data never enter C1–C4 | Low if pilot only; medium if anyone is tempted to mix pilot data in | **High for D01/D02:** changes the approved design (proposal, supervisor's 16 Nov choice); the D435i's narrow view weakens both conditions unless a wide camera replaces it (D02 fallback) | Medium: a printed motorised copy grips differently from the real alloy jaws (D01 weakness of the copy) |
| Schedule impact (estimate, unverified) | Dec gate (replay succeeds) cannot be met; collection slips from Jan to about Feb | Dec gate met on the 2F-85; after delivery about 1–2 weeks to swap driver and camera and repeat the replay; collection late Jan | +1–2 weeks of CAD and printing before pilot data | No slip; but supervisor approval (on leave, email only) and new fingertip/handheld CAD needed | Several weeks of new mechanism, motor and driver work, overlapping the delivery date; the copy path "kept adding problems" (5 Nov) |
| Changes a decision? | No | No (D435i only as a test camera, as already planned) | No if pilot only; D02 if the D435i becomes the policy camera | **Yes: D01, D02** (and D10 handheld body) | D01 (copy instead of real) |

## Recommendation

**Option B now, with a gate in mid-January.**

1. Move the D435i to the station Jetson as already planned (C1–C3) and mount it on the existing prototype with a printed bracket. This is only a test camera, so D02 does not change. Check first whether the teleop team needs the D435i on the robot in the meantime.
2. Build recorder → MCAP → LeRobot export with real images, the trigger value as stand-in width, and the D09 CAD offset.
3. Replay recorded handheld trajectories open-loop on the UR10e with the 2F-85 (Modbus via the existing Pi relay). This meets the thesis plan's December gate on interim hardware.
4. Bring up the VR baseline (XRoboToolkit → UR10e + 2F-85) and dry-run the task protocol.
5. When the Dex1-1 arrives: RS-485 check, swap the driver and camera, repeat the replay, then collect C1–C4 on the final hardware only.
6. **Gate:** if the Dex1-1 has not passed the RS-485 check by mid-January 2027 (thesis plan §5 rule "if replay does not succeed by mid-January"), propose option D to the supervisor with this brief, including a wide USB camera on both sides (D02 fallback) instead of the D435i.

Why B over C: C's matched fingers and passive handheld only make the *pilot* data more realistic and are thrown away when the Dex1-1 arrives. B already gives everything the December gate needs. Why not E: it rebuilds the frozen copy path with a motor added, and it would be ready at about the same time as the real gripper.

## Open

- Is there a second D435i, or does the teleop team need the one on the UR10e during these weeks? (asked 2026-11-20, unanswered)
- D435i colour field of view not read in Intel's datasheet (Intel pages refused automated fetch); 69.4° × 42.5° × 77° ±3° is a user's quote on the RealSense forum ([post](https://support.realsenseai.com/hc/en-us/community/posts/360049401533-D435i-RGB-Sensor-FOV)), unverified.
- 2F-85 mass and grip-force table not checked verbatim (manual spec section **[S]**).
- The new delivery estimate comes from the manager, without a confirmed date.
- Option D needs the supervisor (on leave, email only) and would change D01, D02 and the thesis plan; any such change goes through record-decision.
