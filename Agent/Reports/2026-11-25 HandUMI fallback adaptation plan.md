---
date: 2026-11-25
tags: [report, plan, hardware, handumi, fallback, d01]
---
# HandUMI fallback adaptation plan (Dex1-1 tips)

Topic: **Handheld Rethink**. Martyn (2026-11-25): keep the real Dex1-1 as the main handheld (D01 (handheld gripper body) unchanged) and build an adapted HandUMI with printed Dex1-1 tips **in parallel**, as the fallback if the Dex1-1 is late, never delivered or too heavy on the handheld; it also allows tests without the gripper. Background and options: [[2026-11-25 Handheld Rethink real gripper vs passive design]].

Assuming the HandUMI repositories as read on 2026-11-25: hardware [murobotics-ai/handumi-hw](https://github.com/murobotics-ai/handumi-hw), software [murobotics-ai/handumi-sw](https://github.com/murobotics-ai/handumi-sw), both Apache-2.0 (licence field and README badge). The HandUMI paper ([arXiv 2611.22146](https://arxiv.org/abs/2611.22146), Martyn adds it to the bib) was read only through a summary; arXiv refused direct fetches. Status: **plan, reasoned only**; nothing ordered, printed or decided. Orders, new software dependencies and any decision change wait for Martyn's go.

## What HandUMI is (from the repository)

| Item | Fact | Source |
| --- | --- | --- |
| Body | Hand-worn: thumb link and index/middle-finger link slide on two Ø4 mm guide shafts in LM4UU bushings; a crank plate on the servo shaft couples both links through two connecting links (46 mm), so a pinch closes both jaws symmetrically | Assembly guide steps 6–8 (`hardware/HandUMI_Assembly_Guide.pdf`) |
| Width sensing | Feetech STS3215 servo used as a 12-bit encoder on the crank; Waveshare bus-servo USB adapter; 5 V supply | `bom/README.md`; main README |
| Tracking | PICO 4 Ultra (through XRoboToolkit) or Quest 3/3S; a printed controller support per hand, separate STL for PICO (`left_controller_support_pico.stl`) and Meta | main README; `STL/left_handumi/` |
| Camera | SVPRO IMX335 5 MP wide-angle UVC, on an inclined support above the fingers; optional ZED Mini top view | main README; BOM; guide steps 10–11 |
| Tips | Detachable, 4 × M2 × 12 per finger link; STEP sources for every tip (PiPER, ARX X5, Dream Gripper, WidowX AI, UMI, plus Galaxea R1 Lite, I2RT YAM, OpenArm, MakerMods in the repo) as templates | guide step 7; `STEP/gripper_tips/` |
| Editable CAD | STEP for the whole left and right body (`STEP/left_handumi/`, `STEP/right_handumi/`) | repository listing |
| Fit | Finger cradles drawn from a 3D scan of one operator's hand; "the same workflow can be repeated to fit another operator" | main README, Hand-Fit Design |
| Cost | About USD 104 per unit in parts (pair USD 207.11), headset extra | BOM |
| Software | `handumi` CLI: calibration, QA, replay, retargeting; PICO via XRoboToolkit installed by default; LeRobot-compatible captures | handumi-sw README |
| Opening range | 0–66 mm, mapped linearly to the PiPER stroke (paper, Sec. 2, summary only) | arXiv 2611.22146 |

Computed here from the left-hand STL files (2026-11-25): printed volume ≈ 197 cm³ without the Meta support and foam pad, ≈ 245 g of PLA at 100 % infill (upper bound; the BOM says about 200 g of filament per unit). With the STS3215, camera and PICO controller (≈ 201 g with batteries, [[2026-11-05 D01 real vs copy side-by-side]]) the unit is roughly 500 g, against ≈ 846 g for the real-Dex1-1 handheld; **estimate, nothing weighed**.

## Fit to this thesis

| Need | HandUMI as is | Adaptation |
| --- | --- | --- |
| Dex1-1 is a parallel-jaw gripper (rack and pinion) | Built for parallel-jaw grippers | Good fit; the 2F-85 (linkage gripper) is not, so no 2F-85 tip |
| Dex1-1 opening ≈ 82 mm (URDF 0–0.041 m per side; product page 90 mm, unverified) | 0–66 mm (paper, summary) | Map the HandUMI width linearly onto the Dex1-1 range, as HandUMI does for PiPER; or lengthen the shafts and crank. Check the real travel in Fusion from the STEP first |
| PICO tracking via XRoboToolkit, LeRobot export (D04 (pose tracking), D07 (software and data format)) | Same stack | No change; the own recorder (MCAP, D07) reads the controller as today |
| Gripper width signal | Servo encoder over USB | New: a small "plug" in the shared gripper program reads the encoder; needs a Feetech driver library (new dependency, Martyn's go) |
| D09 (calibration): controller-to-tip offset from CAD | Different geometry | New offset from the adapted HandUMI assembly in Fusion |
| D10 (handheld mechanism): controller as the grip | Hand-worn, controller on the wrist | Only for the fallback; D10 stays for the main path |
| Camera view as on the robot | Own fisheye | See camera options below |

## Dex1-1 tip design

1. Start from a HandUMI tip STEP (PiPER is the closest: parallel jaws, one part per side) to get the 4 × M2 mounting interface on the finger link.
2. Replace the jaw by the Dex1-1 finger and pad shape from the official Dex1-1 STEP (local, `dex1-1-handheld/hardware/from-official-docs/`, must not be shared) or the unitree_ros meshes (BSD-3), so that the part the camera sees has the same shape and size.
3. Place the tips so that fingertip positions relative to the camera match the robot: Dex1-1 camera at 68.7 / 0 / 47.5 mm, tilted 30° down, in the Dex1-1 URDF ([[2026-11-25 Camera View Dex1-1 and D435i field of view]]). Check with a MuJoCo render of both views (Camera View scene) before printing.
4. Colour: same as the real fingers; check on the delivered unit or the product images (unverified); pads in TPU if the real pads are rubber (unverified).
5. Fusion: new part "handumi-tip-dex1" in "1 Parts", HandUMI STEP in "0 Reference" (one part per file, shared frame).

## Camera options

| Option | What | Matches the robot view? | Gripper dependence | Delivery | Precedent |
| --- | --- | --- | --- | --- | --- |
| A. Dex1-1 camera on both | Built-in camera on the robot; a second Dex1-1 camera on the handheld | Exact | Needs delivery; whether the camera can be bought or removed separately is unknown | With the gripper | RDT2 ("same model of camera and gripper") |
| **B. Two identical wide USB cameras (recommended for the fallback)** | One on the robot flange next to the gripper, one on HandUMI, same pose relative to the tips | Yes, by construction, for any gripper | None | AliExpress listing (HandUMI's IMX335, ≈ USD 51) takes shipping time; an EU-stock wide-angle UVC model via the Storefront may be faster (not checked) | UMI (same GoPro on both), FastUMI, HandUMI |
| C. D435i | Existing camera | Narrow view; only one unit | None | Now | Weak: lens changes collapse π0.5 (xue2026, Tab. S10) |

B changes D02 (policy camera) if the fallback becomes the study setup; record with the record-decision skill then. It also makes the camera independent of the gripper: the same camera pose works with the 2F-85 now and the Dex1-1 later.

## Parts (one unit, from the HandUMI BOM)

Printed body and tips (lab PLA/PETG; TPU for pads), 4 × MR63ZZ bearings, 4 × LM4UU bushings, Ø4 mm stainless rod (cut to 135 mm), M2/M3 screws and nuts (lab stock likely), hook-and-loop straps, 1 × Feetech STS3215, 1 × Waveshare Bus Servo Adapter (A), 5 V barrel supply, USB-A to USB-C cable, camera per option B (×2: robot and handheld). Order through the Storefront (manager's approval); not ordered.

## Timeline (estimate, unverified)

| When | Step | Fits with |
| --- | --- | --- |
| Now – RL4 done | Read the paper; import HandUMI STEP into Fusion; measure real jaw travel; choose camera; Martyn approves the parts list | RL4 (Robotiq open/close) stays the current step |
| Week of 30 Nov | Order parts; print body in PLA; draw the Dex1-1 tips; refit finger cradles to Martyn's hand if needed | RL5 (D435i + recorder) |
| Week of 7 Dec | Assemble; encoder plug and width calibration; camera mount at the matched pose; MuJoCo view check | RL5 |
| Week of 14 Dec | Test recordings with PICO tracking; replay on the UR10e with the 2F-85 (width mapped) as a pipeline test | RL5 / before delivery |
| Delivery (late Dec – early Jan) | Weigh the real handheld; replay HandUMI recordings on the real Dex1-1 (RL6 (swap to the Dex1-1)) | RL6 |
| Gate (mid-Jan) | Choose the study handheld: real Dex1-1 or HandUMI with Dex1-1 tips | Gate |

Risk: parts delivery time is the main unknown; the copy path showed CAD work can grow (D01 History, 2026-11-05).

## What it does to the thesis

- If the real Dex1-1 is used: HandUMI stays a tested fallback and supports the discussion on hardware modularity (Martyn's rationale in the Handheld Rethink note).
- If the fallback is used: the handheld is "HandUMI, adapted with Dex1-1 tips and a matched camera", a clear build-on-prior-work story; the Dex1-1 tip and an industrial task are new. Deployment on the real Dex1-1 with printed copy tips follows HandUMI, FastUMI and RUMI; the effect of tip matching is unmeasured in all of them (HandUMI, Sec. 4, summary).
- Credit: Apache-2.0 requires keeping the licence and notices for reused files; list HandUMI as "adapted" in [[2026-11-24 Contributions and own-work table]] once built.

## Open

- Real jaw travel of HandUMI (paper says 0–66 mm, summary only): measure in Fusion from the STEP.
- Dex1-1 finger colour and pad material: check on delivery or product images.
- Camera model for option B: two identical units, delivery time via the Storefront.
- Feetech driver library on the station Jetson: new dependency, Martyn's go.
- Whether the Dex1-1 camera can be bought or moved separately (option A).
