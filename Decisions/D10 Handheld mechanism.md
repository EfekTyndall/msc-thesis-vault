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
| Printed dovetail slide or UMI-style linkage ([Push/Pull Gripper](https://www.thingiverse.com/thing:2204113)) | No bought parts | Most play and wear; no natural encoder shaft | Rejected |

## Decision
The handheld copies the Dex1-1 mechanism. Each finger carriage carries a rack and a short MGN9 rail; two MGN9C blocks are fixed in the body (blocks fixed, rails moving, as in the Dex1-1). One central pinion (module 0.8, 20 teeth, pitch diameter 16 mm, axis along the approach axis y) synchronises the fingers. Trigger: passive, squeeze to close, spring return to open; hard stops at both stroke limits. Finger geometry: converted from the official Dex1-1 meshes; redrawn as a parametric model once the physical Dex1-1 (the unit for the UR10e) is measured.

## Evidence
Measured from the official Dex1-1 meshes (`msc-thesis/hardware/dex1-1-reference/`, unitree_ros, BSD-3) on 2026-11-01:
- Rack teeth on both finger carriages (`Link1_1`, `Link2_1`) repeat every 2.51 mm, i.e. module 0.8; the racks face each other across z = 0, pitch lines at z = ±8 mm.
- The body mesh contains a pinion with 20 teeth, tip diameter 17.6 mm, root about 14 mm (pitch 16 mm at module 0.8), axis along y at x = z = 0.
- The body mesh contains two fixed blocks of 28.9 × 20.0 × 8.0 mm with ball-groove profile at z = ±17.2 to 25.2 mm. 20 mm matches the MGN9C block width ([reseller table](https://www.lily-bearing.com/products/mgn9c1r115z0cm/), which lists the length as 30 mm); the 28.9 mm length against HIWIN's own table ([MG catalogue](https://s3.amazonaws.com/www.motionusa.com/hiwin/2025/MG%20Series-2025.pdf)) is unverified.
- Stroke 89 mm (URDF joint limits) turns the 16 mm pinion by 89 / (π · 16) · 360° ≈ 319°, i.e. less than one turn, so a single-turn AS5600 reads the width directly.
Not yet checked against the physical gripper.

## Revisit if
The physical Dex1-1 differs from the meshes (rail size, pinion, module), or the bought rails give too much play or mass.

## History
- 2026-10-28: passive trigger kept (no motor), squeeze to close with spring return (Martyn)
- 2026-11-01: accepted: copy of the Dex1-1 rack-and-pinion on MGN9 rails; finger geometry from the official meshes until the physical gripper is measured (mesh analysis above; Martyn)
