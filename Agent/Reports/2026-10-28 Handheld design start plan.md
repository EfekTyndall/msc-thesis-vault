# Handheld design: where to start

Starting plan for the handheld (printed Dex1-1 copy with trigger, camera, PICO controller and width sensing). No CAD yet. Sources: [[D01 Handheld gripper body]] to [[D06 Recording and synchronisation]], [[D09 Calibration]], `msc-thesis/docs/proposal.md` §1–6, `msc-thesis/docs/thesis-plan.md` §5–6. Papers are cited by `literature.bib` key.

## 1. What is already fixed

| Subsystem | Fixed by | What it means for the design |
| --- | --- | --- |
| Body | [[D01 Handheld gripper body]] | 3D-printed passive copy; no motor, no power on the handheld. Swapping a real Dex1-1 in is the fallback only. |
| Camera | [[D02 Policy camera]] | The Dex1-1's own RGB camera and mount are moved from the robot gripper; no mirrors. |
| Depth | [[D03 Depth sensing]] | D405 mounted on the Dex1-1 bracket and recorded, not a policy input. |
| Tracking | [[D04 Pose tracking]] | One PICO controller rigidly fixed to the handheld, facing the headset. |
| Width | [[D05 Gripper width sensing]] | ArUco tags on the fingers if the camera sees both; otherwise an AS5600 magnet on a rotating shaft + Raspberry Pi Pico. |
| Recording | [[D06 Recording and synchronisation]] | No onboard computer; one cable bundle per handheld to the PC. |
| Calibration | [[D09 Calibration]] | Controller-to-gripper transform solved by hand-eye with the controller on the real Dex1-1 on the UR10e. |

## 2. The one principle that orders the work

Only what the camera sees must match the robot gripper: finger shape and the camera pose relative to the fingers (chi2024; liu2026 uses a non-actuated replica with the robot gripper's exact geometry). Everything behind the camera (handle, trigger, controller mount, cables) is free. So the design starts at the fingertips and the camera mount and works backwards.

## 3. Requirements

| # | Requirement | Source | Check |
| --- | --- | --- | --- |
| R1 | Finger geometry and camera pose relative to the fingers identical to the robot Dex1-1 | D01, D02, chi2024 | Image overlay at equal widths (D09) |
| R2 | Same opening range as the Dex1-1 (90 mm stroke, 80 mm jaws per [Unitree](https://unitree.com/Dex1-1)), symmetric parallel jaw motion, hard stops at both limits (liu2026 matches the robot's range with limit switches) | Unitree, liu2026 | Caliper at stops |
| R3 | Camera and mount removable and re-mountable in the same pose on robot and handheld | D02 | Repeated remount + overlay or extrinsic check |
| R4 | Rigid, repeatable PICO controller mount; LEDs not covered by the operator's hand; controller facing the headset | D04 | Static jitter log; occlusion test |
| R5 | Space for both width options in v1: tag faces visible to the camera, and a shaft site for the AS5600 magnet | D05 | Finger-visibility check on delivery |
| R6 | D405 on the Dex1-1 bracket | D03 | Fit check |
| R7 | One cable exit with strain relief | D06 | Pull test (method unverified) |
| R8 | Mass: no target set. References: TacUMI handheld 540 g (cheng2026 / zhang2026), Dex1-1 550 g ([Unitree](https://unitree.com/Dex1-1)), UMI-3D at 1120 g reported as causing fatigue (wang2026) | – | Weigh v1 |
| R9 | Printable in-house (PLA/PETG body, TPU fingertips, per proposal BOM) | proposal | Print |

## 4. Subsystems and interfaces

One reference frame for everything: the Dex1-1 TCP (centre between the fingertips), with the camera mount as the main datum. Interfaces between subsystems:

```
trigger ──linkage──> finger carriages ──> printed Dex1-1 jaws (+ ArUco tags)
   │                     │
 return spring       encoder shaft (option)
body/handle ── camera mount datum ── Dex1-1 RGB camera, D405 bracket
   └── controller adapter ── PICO controller
   └── cable exit
```

1. **Finger module:** printed jaws copied from the Dex1-1, on a guide that gives parallel, symmetric motion. The real Dex1-1's internal mechanism is not documented in the sources so far (unverified); the replica only has to reproduce the jaw motion, not the mechanism.
2. **Actuation (trigger, linkage, spring):** start from the open UMI design. The UMI repo links a [Hardware Guide](https://docs.google.com/document/d/1TPYwV9sNVPAi0ZlAupDMkXZ4CA1hsZx7YDMSmcEy6EU/edit?usp=sharing) and credits its mechanism to the [Push/Pull Gripper](https://www.thingiverse.com/thing:2204113) ([UMI repo](https://github.com/real-stanford/universal_manipulation_interface)). A rack-and-pinion with one central gear gives symmetric motion and a natural shaft for the AS5600 magnet (D05); whether it is better than a linkage is untested here.
3. **Camera interface:** a copy of the Dex1-1 camera-mount interface, from the Unitree STEP file (requested, open) or measured on delivery.
4. **Controller adapter:** one printed adapter for the PICO controller, designed once and used both on the handheld and on the real Dex1-1 for the hand-eye calibration (D09). LED placement is undocumented (proposal §4), so orientation is found by test.
5. **Width sensing provisions:** tag faces and an encoder pocket (R5).
6. **Handle and cable exit:** last, since nothing the camera sees depends on it.

## 5. Order of work

| Step | Work | Needs the gripper? | Done when |
| --- | --- | --- | --- |
| 1 | Collect references: UMI Hardware Guide and Push/Pull Gripper, CAD of later variants where published (open item in the thesis plan), [G1 + Dex1-1 meshes](https://huggingface.co/k-valentin/unitree-g1-mujoco) for rough geometry; check the PICO model (Settings > About) | No | Folder of references; model known |
| 2 | CAD skeleton: TCP frame, camera-mount datum, finger travel envelope (0–90 mm), from the meshes; replaced by measured values later | No | Skeleton in `msc-thesis/hardware/` |
| 3 | Controller adapter v0 and a bench tracking test while holding it as intended (occlusion by the hand, facing the headset) | No | Static jitter and dropouts logged with XRoboToolkit |
| 4 | Mechanism v0: jaws + guide + trigger + spring, no camera | No | Full stroke, returns to open, play measured with calipers |
| 5 | On delivery: measure the Dex1-1, confirm camera interface (USB/UVC), finger-visibility check (decides tags vs encoder, D05), update skeleton | Yes | Measured dimensions; D05 branch chosen |
| 6 | Integrate v1: camera mount, D405, controller, tags or encoder, cable exit | Yes | Thesis-plan November gate: "Replica records camera, pose and width together" |

Steps 1–4 can start now; steps 3 and 4 are independent and can run in parallel. Hardware freeze is end of December (thesis plan §5), so v1 should leave December for calibration.

## 6. Open questions for Martyn

1. "Adjusted Dex1-1 gripper": does this mean the printed copy (D01), or modifying a real Dex1-1 (the fallback)? This plan assumes the printed copy and no motor.
2. Gripper delivery date, and whether the Flagship version (with the RGB camera) is confirmed. Without it, the camera interface (step 5) is blocked.
3. Which CAD tool will be used (e.g. Onshape, Fusion, SolidWorks)? It decides what goes into `msc-thesis/hardware/` besides STEP.
4. Trigger behaviour: squeeze closes and a spring opens (assumed here as UMI-like; unverified), or the reverse?
5. Is there a weight target, or is "weigh v1 and compare with R8" enough?
6. Are the two ordered Dex1-1s left and right versions with different geometry? Unverified; affects which unit the replica copies.
