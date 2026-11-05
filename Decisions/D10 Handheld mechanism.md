---
date: 2026-11-01
status: accepted
tags: [decision]
---
## Context

How the passive handheld ([[D01 Handheld gripper body]]) moves its fingers: guide, synchronisation of the two fingers, trigger and return. The proposal (`msc-thesis/docs/proposal.md` §1) says the trigger, return spring and finger linkage start from the open [UMI](https://github.com/real-stanford/universal_manipulation_interface) design and later variants. Design plan: `Agent/Reports/2026-10-28 Handheld design start plan.md`.

## Options

| Option | Strengths | Weaknesses | Verdict |
| --- | --- | --- | --- |
| Copy of the Dex1-1 mechanism: rack-and-pinion on MGN9 miniature rails | Same kinematics and guide as the robot gripper; low play; the pinion shaft carries the AS5600 magnet ([[D05 Gripper width sensing]]) | Rails and blocks must be bought; rails cut to length | **Chosen** |
| Rack-and-pinion on MGN7 rails (as in the open [Actuated UMI](https://github.com/actuated-umi/actuated-umi-gripper)) | Smaller, proven in a UMI-size gripper | Not what the Dex1-1 uses | Rejected |
| Steel rods with linear bushings or printed sleeves | Cheap, easy to source | More play | Rejected |
| Separate pistol grip with trigger, controller mounted elsewhere | Free trigger layout | Two grips, awkward and heavier (Martyn) | Rejected |
| Cap over the controller head (nose) or pommel cup | Short load path | Covers the IR LEDs: 6 around the top rim, 2 at the pommel (Martyn, from the [r/PicoXR video](https://www.reddit.com/r/PicoXR/comments/1fvl1j4/seeing_the_pico_4_ultra_controller_leds/); count unverified) | Rejected |
| Squeeze lever along the controller grip, or thumb lever | Higher force (squeeze lever) | Covers more of the controller, so more LED occlusion risk; thumb lever is weak and sits on buttons | Rejected |
| Printed dovetail slide or UMI-style linkage ([Push/Pull Gripper](https://www.thingiverse.com/thing:2204113)) | No bought parts | Most play and wear; no natural encoder shaft | Rejected |
| Trigger: index-finger lever in front of the controller trigger (~2:1, rod to one carriage) | Small, one finger | Whole squeeze force on one finger; ergonomics and fatigue concern (Martyn) | Superseded 2026-11-05 |
| Trigger: multi-finger squeeze trigger along the sleeve front | Established pistol-grip solution (FastUMI; [[2026-11-02 Handheld trigger survey]]); force spread over several fingers | Still needs about 2:1 motion gain (≈ 41.6 mm per finger side vs 20–25 mm trigger travel, unverified) | **Chosen** 2026-11-05 |
| Transmission A: rigid link from the trigger to one carriage (bellcrank only if the layout needs the 90° turn) | UMI / UMI-3D linkage family; fewest parts; push-pull, no tensioning or creep | Ratio varies over the stroke; pin wear | **Main** 2026-11-05 |
| Transmission B: cable and drum on the pinion shaft | Constant ratio (drum r ≈ 4 mm for 20.8 mm trigger travel, computed); ratio changed by reprinting the drum | No precedent found for a trigger; pull-only; line stretch, tensioner and idler needed; shares the shaft with the AS5600 magnet | **Fallback**, designed in parallel 2026-11-05 |
| Transmission: rack on the trigger meshing a gear (TacUMI wording) | Constant ratio | Trigger moves parallel to the pinion axis, so an extra bevel or gear stage is needed; details unpublished | Rejected |

## Decision

The handheld copies the Dex1-1 mechanism. Each finger carriage carries a rack and a short MGN9 rail; two MGN9C blocks are fixed in the body (blocks fixed, rails moving, as in the Dex1-1). One central pinion (module 0.8, 20 teeth, pitch diameter 16 mm, axis along the approach axis y) synchronises the fingers. Trigger: passive, squeeze to close, spring return to open; hard stops at both stroke limits. Handle: no separate grip; the operator holds the PICO controller, mounted below and behind the gripper like a pistol grip with the gripper as the barrel, and closes the fingers with a multi-finger squeeze trigger along the sleeve front that does not press the controller's own trigger. The trigger drives one carriage through a rigid link (transmission A, main; a bellcrank only if the layout needs it); a cable and drum on the pinion shaft (transmission B) is designed in parallel as the fallback, so both can be compared from the first print. A TacUMI-style continuous lock holds the aperture without sustained squeeze (as an option; ratchets rejected after TacUMI). The controller sits in a printed grip sleeve (two halves) that leaves the faceplate, top rim and pommel uncovered, following community PICO 4 Ultra gun shells ([Printables 1481698](https://www.printables.com/model/1481698-pico-4-ultra-gun-new-glock-shell), CC BY 4.0); the gripper attaches to the sleeve's top front. The controller is rotated +35° in the sleeve, so the grip axis sits about 25° below horizontal with the pommel pointing back (both paths share controller-grip). For the real-Dex1-1 path of [[D01 Handheld gripper body]], the gripper attaches instead through real-adapter: an ActiveUMI-style bow from the gripper's rear to a boss under the sleeve's front half, joined by 2 × M3 screws into heat-set inserts so the sleeve variants stay swappable. Finger geometry: converted from the official Dex1-1 meshes; redrawn as a parametric model once the physical Dex1-1 (the unit for the UR10e) is measured.

## Evidence

Measured from the official Dex1-1 meshes (`msc-thesis/hardware/dex1-1-reference/`, unitree_ros, BSD-3) on 2026-11-01:
- Rack teeth on both finger carriages (`Link1_1`, `Link2_1`) repeat every 2.51 mm, i.e. module 0.8; the racks face each other across z = 0, pitch lines at z = ±8 mm.
- The body mesh contains a pinion with 20 teeth, tip diameter 17.6 mm, root about 14 mm (pitch 16 mm at module 0.8), axis along y at x = z = 0.
- The body mesh contains two fixed blocks of 28.9 × 20.0 × 8.0 mm with ball-groove profile at z = ±17.2 to 25.2 mm. 20 mm matches the MGN9C block width ([reseller table](https://www.lily-bearing.com/products/mgn9c1r115z0cm/), which lists the length as 30 mm); the 28.9 mm length against HIWIN's own table ([MG catalogue](https://s3.amazonaws.com/www.motionusa.com/hiwin/2025/MG%20Series-2025.pdf)) is unverified.
- The official Dex1-1 STEP (Unitree documentation, file dated 2026-07-31; `msc-thesis/hardware/dex1-1-reference/README.md`) confirms the layout: parts named MGN9C blocks, MGN9 rails 75 mm long, a 20-tooth output gear (tip r 8.8 mm, root r 6.91 mm) and two drive racks. Its blocks and rails sit about 2 mm closer to the axis than in the unitree_ros mesh (blocks z 15.09–23.11 vs 17.2–25.2 mm), and its pads are 26.8 mm wide (mesh: 28.4 mm).
- Stroke 89 mm (URDF joint limits) turns the 16 mm pinion by 89 / (π · 16) · 360° ≈ 319°, i.e. less than one turn, so a single-turn AS5600 reads the width directly.
Not yet checked against the physical gripper.

Transmission (2026-11-05, [[2026-11-05 Trigger-to-finger transmission survey]]): no UMI variant describes its transmission in words; the UMI bill of materials (handle, linkage L/R, gear L/R, spring) is the only part-level evidence and points to a printed linkage, which UMI-3D reuses. No trigger-driven cable, Bowden or tendon transmission was found. TacUMI rejected a ratchet lock (quantised widths, cable slip) in favour of a continuous self-locking part (cheng2026, Sec. III-A, IV-B).

## Revisit if

The physical Dex1-1 differs from the meshes (rail size, pinion, module), or the bought rails give too much play or mass. Angle and bow: the grip angle is uncomfortable in the print test, or the bow flexes noticeably under the gripper's weight. Transmission: switch to B if the link path around the sleeve is blocked or the varying ratio feels bad in the print test; drop B if A works.

## History

- 2026-10-28: passive trigger kept (no motor), squeeze to close with spring return (Martyn)
- 2026-11-01: accepted: copy of the Dex1-1 rack-and-pinion on MGN9 rails; finger geometry from the official meshes until the physical gripper is measured (mesh analysis above; Martyn)
- 2026-11-01: handle: the PICO controller is the grip (gun hold, gripper as barrel), with an index-finger lever in front of the controller trigger (Martyn; no separate pistol grip)
- 2026-11-02: controller held in a grip sleeve (rim and pommel LEDs uncovered) instead of a cap on the head (LED positions; community gun shells; Martyn)
- 2026-11-03: evidence updated from the official Dex1-1 STEP (MGN9C/MGN9 confirmed, rails 75 mm; block/rail positions differ by about 2 mm from the unitree_ros mesh). Decision unchanged
- 2026-11-03: finger-replica and mechanism CAD rebuilt from the official STEP parts (Martyn). The STEP's rack stops limit the opening to about 83 mm pad gap, so its stroke is about 41.6 mm per side, matching the official URDF (82 mm) rather than the 90 mm product page (unverified until measured on the gripper)
- 2026-11-04: purchase of 2 MGN9C blocks and 2 MGN9 rails (75 mm) on hold until [[D01 Handheld gripper body]] is settled (only needed if the printed copy stays; Martyn)
- 2026-11-04: controller angle in the sleeve: rotated +35°, grip axis about 25° below horizontal, pommel back (controller-grip v15/v16; Martyn confirmed it in Fusion; comfort unverified until printed)
- 2026-11-04: real-path attachment: ActiveUMI-style bow (real-adapter v4) to a boss under the sleeve front half, 2 × M3 into heat-set inserts (4.0 mm × 6 mm holes); keeps sleeve variants swappable; Fusion check showed aligned holes and no interference (Martyn)
- 2026-11-05: trigger changed from the index-finger lever to a multi-finger squeeze trigger along the sleeve front, not pressing the controller trigger, with a TacUMI-style continuous lock; transmission: rigid link to one carriage (A) as main, cable and drum on the pinion (B) as fallback, both designed for comparison (ergonomics and fatigue of the 2:1 single-finger lever; transmission survey; Martyn)
