---
date: 2026-11-04
tags: [report, d01]
---
# Open research items and DM-UMI hardware lessons for the D01 mind map

Input for the [[D01 Handheld gripper body]] mind map (real Dex1-1 vs printed copy as the handheld). The mind map itself is not started.

Method: (1) every "Open" item and every unresolved table row in the four files of `Agent/Research/` (state of vault `main`, 2026-11-04), sorted by whether it separates the two D01 paths; (2) [[@surendran2026]] read and checked against the paper's arXiv HTML ([2606.26603v2](https://arxiv.org/html/2606.26603v2)). Key check: `surendran2026` is in Martyn's local `msc-thesis/thesis/literature.bib` (Zotero export 2026-11-04 11:05), but not yet in the GitHub copy (last changed 2026-10-23); it needs a commit and push before LaTeX can cite it.

## 1. Open items in Agent/Research

"Separates paths" = the answer differs between real and copy, so it belongs in the mind map. "Before mind map" = it can be answered now, at the desk, and would change the mind map.

### Worth resolving before the mind map

| # | Item | Source | Why before the mind map | How to close |
| --- | --- | --- | --- | --- |
| R1 | Handheld mass and size, real vs copy (real: 550 g gripper + controller + sleeve; copy: unknown) | Research gaps B11, Dex1-1 note "New costs" | Mass is the only fatigue factor the papers name; it is the clearest physical difference between the paths. This is the side-by-side comparison already planned (weight, size, stroke, cost/parts) | Fusion mass properties of the copy parts + Unitree 550 g + controller mass (scale). Waits for Martyn's go |
| R2 | Can one PC and one RS-485-to-USB adapter drive both grippers (robot + handheld)? | Dex1-1 note, Wiring: "more than one gripper per adapter not stated" | Decides whether the real path needs one or two adapters/services, and so its parts list | Desk check of `dex1_1_service` (it already publishes motor ID 0 as `right`, ID 1 as `left`, which suggests two motors per service; unverified) |
| R3 | Width mismatch: passive copy records achieved width, Dex1-1 executes a command | Research gaps B4 (Must), A9 | The main data-quality argument for the real path: it records command and achieved width (`q`), like teleop. The copy needs a width mapping at export | Already analysed; state it in the mind map as the copy path's extra task. The Dex1-1 behaviour at a blocked target still needs the gripper |
| R4 | Camera per path. Both grippers are Flagship (RGB camera each), so two cameras exist | Dex1-1 note "Version"; D02; research gaps B8 | D01 and D02 assume one camera moved between robot and handheld. With two cameras, the copy could keep the second camera permanently, removing the remount (B8) but adding a camera-to-camera difference (intrinsics, mount). Same question on the real path | Martyn to confirm the plan; reasoning here, not checked |
| R5 | Correct the DM-UMI mixing claim | Research gaps Summary 3, C2 | The note says naive mixing was worse than handheld alone on "two of three tasks". The arXiv HTML gives worse on all three (44.0 → 0.0%, 13.3 → 6.7%, 10.0 → 0.0%); the abstract says the same without numbers. Affects H3 framing, not D01 | Confirm in the PDF tables, then fix the note and project memory |

### Separates the paths but needs hardware (list in the mind map as "to be measured")

| Item | Path | Source | Blocked on |
| --- | --- | --- | --- |
| RS-485 check: command rate, latency trigger → finger, command-to-width curve, `tau_est` vs force gauge | Real | Dex1-1 note, Open | Delivery |
| Finger stroke (official CAD ~82 mm vs product page 90 mm) | Both (copy copies it) | Dex1-1 note; memory | Delivery |
| Back-drive by hand when unpowered or braked | Real | Dex1-1 note, Open | Delivery (a desk check of the support page could give a first answer) |
| Cable length and tether to the handheld (power + USB) | Real | Dex1-1 note, Wiring | Delivery |
| Adapter and accessories in the box | Real | Dex1-1 note, Open | Delivery |
| Trigger value vs travel (linearity, dead zone, resolution) | Real (trigger is the gripper command) | Trigger readout, Open | PICO + XRoboToolkit setup |
| Squeeze force over the stroke, friction, lock | Copy | Research gaps B5; trigger survey Open | First print |
| Width sensing error (ArUco or AS5600, backlash) | Copy | Research gaps B6 | First print |
| LED occlusion by gripper and hand (geometry differs per path) | Both, differently | Research gaps B1 | Print + headset |

### Not needed for the mind map

| Item | Source | Reason |
| --- | --- | --- |
| 20 rollouts underpowered (C1) | Research gaps | Parked for later stages (Martyn, 2026-11-04); same for both paths |
| Sleeve re-seat repeatability (B2), calibration fixture (B3) | Research gaps | Both paths use the sleeve and need a robot-side bracket with the same interface; not a discriminator |
| Tracking accuracy, sync, dropouts, rejection criteria (A3, A5, A6, B7) | Research gaps | Same for both paths |
| Kinematic filtering, efficiency accounting, operators (A7, C3–C5) | Research gaps | Experiment design, not hardware |
| PICO Enterprise firmware delivers controller inputs; trigger-to-PC latency | Trigger readout | Needed for the VR teleop baseline either way |
| UR10e tool connector 24 V / 3.5 A | Dex1-1 note | The robot carries a Dex1-1 in both paths |
| UMI trigger geometry; no published trigger force data | Trigger survey | Copy-path detail for D10, after D01 |
| Unchecked bib papers; non-bib works read only via HTML | Research gaps, Open | Literature work; DM-UMI is now in Zotero and can be re-read there |

## 2. Hardware lessons from Surendran et al. 2026 (DM-UMI / BRIDGE)

Facts from the arXiv HTML (sections as given there; page numbers not extracted). Implications are reasoning for this thesis, not results of the paper.

| Fact (surendran2026) | Implication for the handheld and D01 |
| --- | --- |
| A handheld records observed actions; a controller needs desired actions. They match in free-space ("tolerant") phases and diverge in contact-sensitive ones, where tracking observed actions needs high stiffness and gives large forces, F ∝ K(x_d − x) (Sec. I, III, Fig. 1) | Holds for the arm pose in both D01 paths: neither path records a desired arm pose. For the **gripper** it separates the paths: the real Dex1-1 records the command (desired) and `q` (achieved); the passive copy records only achieved width, which stops at the object (research gaps B4). The real path removes the problem for the gripper channel only |
| DM-UMI replaced UMI's offline sensing (monocular SLAM, ArUco width) with online sensors, an XVISIO DS80 SLAM camera and magnetic position sensors for width, "since teleoperation requires online estimates" (Sec. III) | Width sensing that works live lets the same handheld also teleoperate the robot. Real path: online by design (`q` at 500 Hz). Copy path: ArUco is offline-only; the AS5600 fallback in D05 would be the online option |
| Same device used for handheld collection and teleop of the robot (Sec. III, IV); whether the operator stands away from the robot is not stated | Pattern for the real path: PICO-tracked real Dex1-1 is close to a dual-mode device. Not part of the thesis design; noted as an option only |
| DS80 accuracy over 1000 demos vs Vicon: mean absolute error 25.9 mm / 6.3°, mean relative error 1.1 mm / 0.2° (Sec. IV, Table I). The literature note quotes only the relative figures | Absolute pose error can be an order of magnitude above relative error. Report both for the PICO controller (A3); same for both D01 paths |
| Robot: Franka FR3, 1 kHz Cartesian impedance controller, commands at 100 Hz; HIGH 1600 N/m / 60 Nm/rad, LOW 800 N/m / 30 Nm/rad (Sec. III, Table VIII) | The method relies on a compliant arm. For the UR10e and H2, controller compliance at deployment is a D08 question, not D01. For the gripper, the Dex1-1 `kp` plays the stiffness role; only the real path records which `kp` was used during collection |
| Authors: F/T-instrumented UMI-style devices "may allow us to identify" contact phases directly (Sec. V) | Dex1-1 has no F/T sensor, but its `tau_est` gives a grip-contact signal during handheld collection on the real path only; the copy gives none. Usefulness unverified |
| Not reported: handheld mass, gripper model, finger design, whether handheld and robot grippers match (Sec. III) | The paper gives no evidence for real vs printed copy. It cannot be cited for D01 itself |
| Full-teleop policy on a second robot dropped to 10%; BRIDGE with 50 support demos reached 30% (Sec. IV) | Policies are sensitive to the embodiment; relevant to UR10e → H2 transfer, not D01 |
| Pulley teleop caused frequent robot safety-limit faults during collection (Sec. IV) | A teleop cost worth logging in the efficiency metric (C3); not D01 |

### Notes on the literature note

- The Q&A says DS80 was chosen "over standard monocular action cameras (like GoPros)". The HTML does not mention GoPro; the stated reason is online estimation. Suggest rewording or adding a page reference once checked in the PDF.
- "Key facts (with page)" is still empty; the facts above could go there once page numbers are taken from the PDF.

## 3. Suggested mind map entries from this review

- **Gripper action semantics:** real = desired + achieved width (+ `tau_est`); copy = achieved width only, needs a width mapping.
- **Online width signal:** real yes; copy only with an encoder.
- **Mass and tether:** real 550 g + cables (measured value pending); copy mass pending (R1).
- **Cameras:** two Flagship cameras available; remount need depends on the camera plan (R4).
- **Not decided by D01:** arm observed-vs-desired gap, tracking accuracy, deployment compliance.

## Open

- Page numbers: everything in section 2 is from the arXiv HTML via a summarising fetch; re-check numbers in the PDF before citing.
- `literature.bib` with `surendran2026` is not pushed to GitHub yet.
- R1 (side-by-side) waits for Martyn's go; R2 and R5 are desk checks that can be done on request; R4 needs Martyn's input.
