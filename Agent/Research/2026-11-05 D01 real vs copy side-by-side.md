---
date: 2026-11-05
tags: [research, hardware, dex1-1, d01]
---
# D01 side-by-side: real Dex1-1 vs printed copy as the handheld

Question: how do the two handheld paths of [[D01 Handheld gripper body]] compare on mass, size, stroke and parts? Input for the D01 mind map (target: end of the 2026-11-06 session).

Method: numbers are taken from the existing notes and repo files, each with its source. Fusion values were read on 2026-11-05 with a read-only script from the "Dex1-1 Handheld" project (`ASM copy-path` v9, `ASM real-path` v11; linked parts copy-mechanism v8, copy-fingers v6, controller-grip v16, real-adapter v6, all at their latest version). Every Fusion body is set to Steel, so printed-part masses are computed here as body volume × 1.27 g/cm³, an assumed PETG density (unverified), at 100 % infill: an upper bound, since real prints are partly hollow. Bought steel parts (MGN9C blocks, MGN9 rails) use Fusion's steel mass. Nothing was weighed or measured on hardware. "Unverified" marks every value without a source or measurement.

Both paths share: the PICO 4 Ultra Enterprise controller as the grip, the grip sleeve (controller-grip, variant A or B), the Dex1-1 RGB camera on the gripper, and the tethered PC setup ([[D06 Recording and synchronisation]]).

## Summary

1. **Mass:** the real path carries the 550 g gripper ([Unitree](https://unitree.com/Dex1-1)) plus real-adapter; the copy path carries printed parts plus 2 MGN9C blocks and 2 MGN9 rails. Modelled so far, without controller and cables: about 645 g for the real path and about 228 g for the copy path (sleeve A in both), i.e. about 420 g more on the real path. The copy figure misses the lever, rod, spring, bracket and screws, which are not modelled yet.
2. **Size and stroke are the same by construction:** the copy is rebuilt from the official STEP solids, so both have the 143.3 mm length and about 82–83 mm stroke of the official model. The 90 mm on the product page is unverified until the real gripper is measured.
3. **Parts:** the real path needs no bought mechanical parts beyond the ordered grippers, but adds a 24–64 V supply (≥ 3.5 A peak), a power cable and the RS-485 cable to the handheld. The copy path needs rails, blocks, a return spring, a lever and, for an online width signal, an AS5600 encoder.
4. **Cost:** no prices are in the notes for the Dex1-1, the MGN9 parts, the supply or cables (all "to be quoted"); only the AS5600 module is priced (€8.95). A cost comparison in euros is therefore open.

## Side-by-side

| Property | Real Dex1-1 path | Printed-copy path | Source |
| --- | --- | --- | --- |
| Gripper body mass | 550 g (product page); 0.571 kg sum of official URDF link masses | 174.6 g: steel 108.6 g (2 MGN9C blocks 25.8 g each, 2 MGN9 75 mm rails 28.5 g each) + printed 66.0 g (housing 12.67, 2 fingers 23.04, 2 pads 6.22, 2 racks with stops 7.22, pinion 2.81 cm³; 51.96 cm³ in total). Not modelled: lever, rod, spring, screws | [Unitree](https://unitree.com/Dex1-1); `msc-thesis/hardware/dex1-1-reference/README.md`; Fusion |
| Camera | Included in the gripper (Flagship) | Moved from the robot gripper: 21.5 g (`Camera` link in the official URDF; unverified) | `dex1-1-reference/README.md` |
| Attachment to sleeve | real-adapter v6 (bow + clamp ring): 50.00 cm³ → 63.5 g | grip-to-gripper-bracket v6 is still empty (0 cm³) | Fusion |
| Sleeve (shared) | Variant A 25.01 cm³ → 31.8 g; variant B 18.46 cm³ → 23.4 g | same | Fusion (controller-grip v16) |
| Controller (shared) | Unverified (not weighed) | same | – |
| Handheld total without cable (sleeve A) | ≈ 645 g + controller | ≈ 228 g + controller + unmodelled lever, rod, spring, bracket | Sum of rows above |
| Length / envelope of the gripper | 143 × 78 × 67 mm (product page); 0.1433 m in the official URDF | Same outer geometry (copy-fingers from the official STEP); copy-mechanism 84.0 × 27.9 × 52.2 mm, copy-fingers 84.0 × 80.4 × 29.0 mm (Fusion bounding boxes at q = 0) | [Unitree](https://unitree.com/Dex1-1); `dex1-1-reference/README.md`; Fusion |
| Stroke | Official URDF 0–0.041 m per side (82 mm); STEP rack stops ≈ 41.6 mm per side (83.2 mm pad gap); product page 90 mm (unverified which is right) | Same geometry: copy-mechanism moves without collision for q from −16.5 to 24.9 mm per side (≈ 41.4 mm per side) | `dex1-1-reference/README.md`; [[D10 Handheld mechanism]] History 2026-11-03 |
| Actuation | PICO trigger → PC → `rt/dex1/<side>/cmd`; proportional or binary (software choice) | Passive index-finger lever in front of the controller trigger, ~2:1, rod to one carriage, spring return | [[2026-11-03 Dex1-1 technical note]]; [[D10 Handheld mechanism]] |
| Width signal | Command and achieved `q` at 500 Hz, plus `tau_est`; rad → width curve still to be measured | Achieved width only: ArUco from the camera (offline) or AS5600 encoder (online); needs a width mapping | Technical note; `msc-thesis/docs/proposal.md` §5 |
| Grip force | 5–120 N, set through `kp` and position error | Operator's finger force through the lever (unmeasured) | [Unitree](https://unitree.com/Dex1-1); technical note |
| Power on the handheld | 24–64 V DC, 0.03 A static, 3.5 A peak (≈ 84 W at 24 V, computed) | None (encoder option: powered over its USB link to the PC) | [Unitree](https://unitree.com/Dex1-1); proposal §5 |
| Cables to the handheld | Camera USB + power + RS-485 (one XT30 2+2 cable from the gripper; length unverified) | Camera USB (+ encoder USB if used) | Technical note, Wiring |
| Robot availability | Second ordered gripper stays on the UR10e (two grippers ordered) | Robot keeps its gripper; camera moves between robot and handheld per session | [[D01 Handheld gripper body]] History 2026-11-04; D01 Options |
| Fingers vs robot | Identical (same model) | Printed fingers, grip differently from the alloy jaws | D01 Options |
| Delivery risk | Needs a delivered gripper and a passed RS-485 check | Testable now as soon as a printer is available | [[Waiting]] |

## Parts and cost

| Item | Real path | Copy path | Cost | Source |
| --- | --- | --- | --- | --- |
| Dex1-1 gripper | 1 (already ordered, Flagship with RGB camera) | 0 extra (camera borrowed from the robot gripper per D01) | Not stated in the notes | proposal BOM; [[2026-11-03 Dex1-1 technical note]] |
| 485-to-USB board, gripper cable | In the box (reseller) | – | Included | [US Robot Store](https://www.usrobotstore.com/products/dex1-1-gripper-compatible-with-all-unitree-g1-models-and-h2) |
| DC supply 24 or 48 V, ≥ 3.5 A | 1 (not in the box) | – | To be quoted | Technical note |
| Power + RS-485 cable extension to the handheld (3–5 m) | 1 | – | To be quoted | Technical note; proposal §6 (3–5 m) |
| MGN9C blocks + MGN9 rails 75 mm | – | 2 + 2 (on hold until D01) | To be quoted | [[D10 Handheld mechanism]]; [[Waiting]] |
| Return spring, lever, rod | – | 1 set (printed / small parts) | Unverified | D10 |
| AS5600 module + Raspberry Pi Pico | – | 1 set, only if an online width signal is needed | €8.95 (module) + Pico to be quoted | proposal BOM |
| M3 heat-set inserts + M3 screws | 2 + 2 (bow joint), plus 4 × M3 rear mount and 1 clamp screw | Bracket screws (bracket not modelled yet) | To be quoted | `msc-thesis/hardware/PRINT-TEST-QUEUE.md` §3 |
| Printed parts (PETG) | Sleeve + real-adapter | Sleeve + housing, carriages, pinion, fingers, lever | Filament, in-house | PRINT-TEST-QUEUE |
| Cable clip | 1 (waits on cable exit) | – | Printed | [[Waiting]] |

## Open

- Weigh the controller, the delivered gripper and the printed parts; Fusion masses use an assumed PETG density at 100 % infill (upper bound); set the real material and infill once the slicer settings are known.
- Prices for the Dex1-1, MGN9 parts, supply and cables are not in the notes.
- Stroke: measure on the delivered gripper (82–83 mm vs 90 mm).
