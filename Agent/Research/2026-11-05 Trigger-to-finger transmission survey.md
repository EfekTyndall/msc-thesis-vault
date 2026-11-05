---
date: 2026-11-05
tags: [research, handheld]
---
# Trigger-to-finger transmission in UMI-style handhelds

Question: how do UMI variants transmit the trigger motion to the fingers (mechanism, parts, extra hardware, ratio, reliability), and which transmission suits the copy path of [[D01 Handheld gripper body]]? Fixed inputs (Martyn, 2026-11-05): multi-finger squeeze trigger, optional TacUMI-style lock, trigger does not press the controller trigger. Builds on [[2026-11-02 Handheld trigger survey]], which covers the actuation type (passive, motor, glove); this note covers the transmission only.

Method: paper text (arXiv HTML), project pages, the UMI Hardware Guide (plain-text export) and open-hardware READMEs, read on 2026-11-05. Figures, videos and CAD were not viewable, so anything that only a figure shows is missing here.

**Main finding:** no paper describes its transmission in words. The only part-level evidence is the UMI bill of materials; everything else is one phrase per paper.

## Findings

| Work | Transmission | Sync of the two fingers | Return | Extra hardware | Ratio | Reliability notes | Source |
| --- | --- | --- | --- | --- | --- | --- | --- |
| UMI (chi2024) | Linkage. Printed train: handle, linkage L/R, gear L/R, finger holder L/R; 2 × MGN9C 150 mm rails; 1 compression spring (10 mm OD, 0.7 mm wire, 50 mm free length). How they connect is not stated; reading from the part names: trigger pulls two links to the finger holders, two gears sync (unverified) | Gear L/R (inferred from names) | Compression spring (position and direction not stated) | None (width from ArUco) | Not stated; 80 mm stroke | Super Lube in BOM; weight and bulk named as the efficiency limit (Sec. VIII); no wear data | Paper Sec. III-A; Hardware Guide BOM and print list; mechanism credited to the [Push/Pull Gripper](https://www.thingiverse.com/thing:2204113) (page did not load) |
| UMI-3D (wang2026) | Same as UMI: "completely interchangeable with the original UMI" | As UMI | As UMI | LiDAR, camera | Not stated; 80 mm stroke | Weight (1120 g) causes fatigue (Sec. VI) | Sec. III-A |
| TacUMI (cheng2026) | "UMI-style rack-and-pinion trigger", closure "proportional to trigger effort" | Not stated | Not stated for the final design | Continuous self-locking part: "thumb-engaged with index release"; lock element not named | Not stated | Ratchet lock rejected: discontinuous widths, cable slipped out. No lock: trigger force contaminates F/T data. Tension-spring (normally closed) variant rejected: cannot open wide enough | Sec. III-A2, III-A4, IV-B, Fig. 6; [project page](https://tac-umi.github.io/TacUMI/) (3D-print zips released, contents not checked) |
| FastUMI (zhaxizhuoma2025) | Not stated ("parallel-motion gripper") | Not stated | Not stated | None (width from ArUco) | Not stated | Not stated | Sec. II-C, Fig. 1 |
| DM-UMI (surendran2026) | Not stated | Not stated | Not stated | "Magnetic position sensors to measure gripper width" | Not stated | Not stated | Sec. III |
| ActiveUMI (zeng2025) | Micro-motor on the controller; coupling not stated | Not stated | Motor | Motor, backpack battery and PC; command input not stated | Not stated | Not stated | Sec. 3.1 |
| ViTaMIn-B (li2025) | Direct: handles on the finger sliders | Gear (figure, per the 2026-11-02 survey) | Not stated | None (width from ArUco, 80 mm max) | 1:1 (direct) | "Fewer transmission components, which improves durability" (Sec. IV-A) | Sec. IV-A |
| RDT2 (liu2026) | Direct: fingers set by hand on a sliding rail with mechanical limit switches (App. A.1); Table 1 says "linkage gripper" | Not stated | Not stated | Limit switches | 1:1 (direct) | CNC nylon 66 + glass fibre for "higher stiffness … consistency" (Table 1) | App. A.1, Table 1 |
| HiFi-UMI (ai2026) | Direct: glove, thumb against four fingers | n/a | Not stated | Encoders (opening-angle error < 0.1°) | 1:1 (direct) | Argues triggers give weaker contact feedback | Sec. 3.1.2–3.1.3, Table 2 |
| Actuated UMI (Helmut et al. 2024, [repo](https://github.com/actuated-umi/actuated-umi-gripper), not in bib) | Motor + flat belt (5 × 1 mm, 300 mm) over a drive pulley; 2 constant-force springs on idlers (tensioning, inferred); 2 × MGN7C 80 mm | Belt (inferred) | Not stated | Dynamixel XL430, U2D2, power hub | Not stated | Not stated | README BOM |

Not found in any source: a trigger-driven cable, Bowden or tendon transmission. The belt in Actuated UMI is motor-driven. A wider search (DexUMI, ExUMI, Dobb-E) was not done.

## What follows for the copy path

Both concepts from the discussion are compared below against the evidence. Counts are for the transmission only (trigger, return spring and the existing rack-and-pinion excluded); they are estimates, not CAD.

| Rank | Transmission | Parts added | Precedent | Reliability | Ratio |
| --- | --- | --- | --- | --- | --- |
| 1 | **Rigid link to one carriage** (option A); rack-and-pinion syncs the other finger. Add a bellcrank only if the squeeze direction is not in the plane of carriage travel | 1 link + 2 pins; bellcrank: +1 part, +1 pivot | UMI and UMI-3D (same parts): printed linkage with an open BOM | Push-pull, no tensioning, no creep; wear at the pins (UMI uses grease) | Set by link angles or bellcrank arms (e.g. 10 : 20 mm arms give 2 : 1, computed); varies over the stroke |
| 2 | **Cable and drum on the pinion** (option B) | Line, drum, ≥1 idler for the 90° turn (pinion axis lies along the barrel), tension screw | None found for a trigger | Pull-only, so the spring alone opens; line stretch and creep need a tensioner; drum competes with the AS5600 magnet for the shaft | Constant: drum radius 4 mm for 20.8 mm trigger travel (computed from 41.6 mm per side and 8 mm pinion radius) |
| 3 | Rack on the trigger meshing a gear (TacUMI wording) | Rack + extra gear stage for 2 : 1 | TacUMI (details not published) | Gear mesh needs accurate alignment in print | Constant; trigger travels along the barrel, parallel to the pinion axis, so a direct mesh is not possible without a bevel or extra stage (from D10 geometry) |

Excluded: motor (ActiveUMI, Actuated UMI) because D01 keeps the handheld passive; direct handles (ViTaMIn-B, RDT2, HiFi-UMI) because they do not fit the pistol hold ([[2026-11-02 Handheld trigger survey]], point 2).

**Lock:** TacUMI shows that a ratchet fails (quantised widths, slip) and a continuous lock works, but not how it is built. A continuous lock fits a rigid link more easily than a cable: a friction plate that tilts and grips the link, as in a bar clamp, released by the index finger (unverified for this use; untested). For a cable, a cam cleat would do the same (unverified).

**Recommendation (for Martyn to decide):** option A, a rigid link from the trigger to one carriage, with a bellcrank only if the Fusion layout needs the 90° turn. It is the transmission family the UMI line actually uses, it adds the fewest parts, and it has no tensioning or creep. This revises the cable-and-drum recommendation in [[2026-11-02 Handheld trigger survey]], which had no precedent behind it. Keep option B as the fallback if the link path around the controller sleeve is blocked or the varying ratio feels bad in the print test.

## Open
- Check in Fusion (controller-grip v16, +35°) whether the squeeze direction lies in the plane of carriage travel; this decides whether option A needs a bellcrank.
- Mechanism details that only figures and videos show: UMI [assembly video](https://youtu.be/x3ko0v_xwpg) and [Onshape CAD](https://cad.onshape.com/documents/a8a98201374f5cc9c6538790), TacUMI Fig. 2 lock insets and 3D-print zips. Martyn can view these; they were not readable here.
- Trigger travel 20–25 mm is unverified; measure it on the first print.
