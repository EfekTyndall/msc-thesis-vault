---
date: 2026-11-25
tags: [research, hardware, d01, handheld, modularity]
---
# Handheld Rethink: real gripper vs passive design

Question (Martyn, 2026-11-25): prior work builds passive UMI handhelds with swappable tips; with the Dex1-1 late and only the lab Robotiq 2F-85 at hand, should the handheld stay a real Dex1-1 on the PICO controller (D01 (handheld gripper body), D10 (handheld mechanism)), or become a passive, modular design? Concern: a passive trigger around the PICO grip adds bulk ("very pretty" read as "very bulky/heavy"; assumption, not confirmed).

Assuming the current state from [[D01 Handheld gripper body]] (real Dex1-1 main since 2026-11-16, printed copy frozen as fallback), [[2026-11-05 D01 real vs copy side-by-side]] (≈ 846 g vs ≈ 438 g, modelled, not weighed) and [[2026-11-20 Interim hardware while the Dex1-1 is delayed]] (gate mid-January: 2F-85 for the whole core as last resort).

Method: bib papers and earlier vault notes, plus a wide search with the deep-research skill (three parallel searches, 2026-11-25). All web sources were read through a summarising fetch tool, so section numbers and figures are as reported there; papers marked "not in bib" must be read in the original before citing. Status: **reasoned only** (sources, no measurement); nothing here changes a decision.

## In short

1. Prior work splits into two ways of being modular. **Covers on the robot's fingers** (FastUMI: printed tips on xArm and 2F-85 that look like the handheld's) or **copies of the robot's fingers on one passive handheld body** (HandUMI: five tip sets; RUMI: a copy of the 2F-85 linkage and finger mount). Nobody has validated one handheld on two different robot grippers, and nobody has measured what matched vs unmatched tips do to policy success.
2. Two of the newest designs are very close to this thesis: **HandUMI** uses a **PICO 4 Ultra** controller (via XRoboToolkit) or Quest controllers on a passive body with swappable tips, controller and camera mounts; **RUMI** copies the **Robotiq 2F-85** finger interface, with a D405 in the robot's camera pose and a Quest 3 controller as tracker. Both are from November 2026 and not in bib.
3. Passive designs with a VR controller can be light: YUBI states about 319 g (200 g gripper + 119 g Quest controller) against UMI's 780 g. The bulk comes from the trigger and transmission, not from the controller; HandUMI drives the jaws from finger cradles through a crank instead of a trigger.
4. A passive handheld does **not** remove the dependence on the Dex1-1 for the core study: the robot side needs the Dex1-1 (and its camera) either way. What it removes is the second gripper on the handheld, the 24–64 V supply and RS-485 cable to the hand, and about 400 g.
5. The current fallback is weaker than it looks: the lab has **one** 2F-85, and it sits on the UR10e. If the mid-January gate falls back to the 2F-85, the handheld has no gripper to carry. A passive 2F-85 tip set (RUMI precedent) is the missing piece.

Recommendation (reasoned only): **keep the real Dex1-1 as the main handheld, and replace the frozen Dex1-1 copy fallback with a passive base that takes swappable tip sets, starting with 2F-85 tips** (option D below). Details and the alternatives follow.

## What prior work does

| Device | Handheld | Robot side | How tips match | Grippers | Mass | Camera | Tracking | Source |
| --- | --- | --- | --- | --- | --- | --- | --- | --- |
| UMI | Passive trigger, 80 mm stroke | Real WSG-50 with the same printed soft fingers | Same fingers on both | 1 | 780 g | GoPro 155° fisheye, same pose | GoPro SLAM (ORB-SLAM3) | chi2024, Sec. III-A, HD2–HD4 |
| FastUMI | Passive parallel gripper | Real robot grippers with plug-in fingertip covers | Covers look like the handheld tips outside, fit each gripper inside; 5 variants, xArm and 2F-85 named; ~1 cm finger-length change of non-parallel jaws compensated in software | Claimed several; tested on xArm 6 only | Not stated | GoPro, bottom of image aligned with fingertip bottom | RealSense T265 | zhaxizhuoma2025, Sec. II-C, IV-E, VI |
| RDT2 | Unpowered copy of the robot gripper | Same gripper model on Franka and UR5e | "Same model of camera and gripper" | 1 model, 2 arms | Not stated | Same camera model | 4 × Vive Tracker 3.0 | liu2026, Fig. 1, App. A.1 |
| ActiveUMI | Copy of the robot gripper on a Quest 3S controller, micro-motor | Real ARX gripper kept | Whole-gripper copy | 1 | Not stated | Fisheye on each controller | Quest 3S | zeng2025, Sec. 3.1 |
| RAPID | Same motorised base in the hand and on the robot, adapter swapped | Same unit | Fingers swap in about 5 s | Reconfigurable | Not stated | Not stated | Not stated | yin2026, Table I |
| DM-UMI | Handheld, width from magnetic sensors | – | Not stated | – | Not stated | – | XVISIO DS80 SLAM: 25.9 mm / 6.3° absolute, 1.1 mm / 0.2° relative error | surendran2026, Table I |
| ViTaMIn-B | Passive, Quest 3 controller fixed at the back | – | – | – | "Lighter" than UMI | – | Quest 3: valid demos 1.0 vs 0.16, policy 0.7 vs 0.1 against SLAM | li2025, Table IV |
| YUBI | Passive finger-driven | Same YUBI gripper mounted on UR, Franka, ELEY | Same gripper | 1 gripper, 3 arms | ≈ 319 g incl. Quest controller | Wrist camera | Quest 3S | [arXiv 2606.10244](https://arxiv.org/abs/2606.10244), Sec. 3.1 (not in bib) |
| HandUMI | Passive, hand-worn; finger cradles drive parallel jaws through a crank | Robot keeps its own gripper | Handheld tips copy PiPER, ARX X5, Dream Gripper, WidowX AI, UMI finger; "new grippers require new capture tips" | 5 tip sets; only PiPER tested on a robot | Not stated (parts USD 207 per pair) | Fisheye, swappable mount | **PICO 4 Ultra** (XRoboToolkit) or Quest 3/3S | [arXiv 2611.22146](https://arxiv.org/abs/2611.22146), Sec. 2–4 (not in bib) |
| RUMI | Passive linkage copied from the official **2F-85** CAD, encoder | Real 2F-85 on a Franka FR3 | Finger mount copied, so original Robotiq fingers and sensor tips move between both | 1 (2F-85) | Not stated | D405 "in the same pose used on the target robot" | Quest 3 controller | [arXiv 2611.15802](https://arxiv.org/abs/2611.15802), Sec. III-B (not in bib) |
| UMI-3D | Passive | Printed UMI copy | Same UMI fingers | 1 | 1120 g; weight causes fatigue | Fisheye | LiDAR-inertial | wang2026, Sec. III-A, VI |

Bulk and fatigue: UMI calls its handheld bulkier and slower than bare hands (chi2024, Sec. VIII); UMI-3D names its 1120 g as a fatigue limit (wang2026, Sec. VI). No paper reports the mass or fatigue of a handheld that carries a motorised robot gripper, and none compares passive and actuated handhelds (wide search, not exhaustive).

## What keeps the embodiment gap small

- **Matched setups stay near source level**: UMI 20/20 on the cup task with the same fingers and camera pose (chi2024, Sec. V-A); HiFi-UMI with identical gripper and cameras: π0.5 77.5 % on handheld data vs 74.4 % on teleop data (ai2026, Sec. 6.2).
- **The camera matters most**: a lens change drops π0.5 from 1.00 to 0.50 (150°) or 0.0025 (220°) (xue2026, Tab. S10); a different gripper in view drops Diffusion Policy from 60–90 % to 0–20 % without correction ([Mirage, arXiv 2402.19249](https://arxiv.org/abs/2402.19249), Table IV, not in bib). Inpainting recovers much of it but costs GPU time and needs calibration ([DexUMI, arXiv 2505.21864](https://arxiv.org/abs/2505.21864), Table 1, not in bib). Every handheld above avoids inpainting by putting the same camera in the same pose relative to the fingertips.
- **Finger shape**: no policy-level ablation of matched vs unmatched tips exists; HandUMI states the "tip-matching gain" is unmeasured (Sec. 4). The only numbers are from humans: UMI-style stiff parallel fingers opened 65.8 % of bandages vs 100 % with angled fingers ([arXiv 2603.17189](https://arxiv.org/abs/2603.17189), not in bib).
- **Compliance**: LEGATO argues a passive handheld against an actively driven robot gripper is itself a mismatch (grip force set by the hand vs by the motor) ([arXiv 2411.03682](https://arxiv.org/abs/2411.03682), not in bib).

For this thesis: the Dex1-1 camera sits at 68.7 / 0 / 47.5 mm, tilted 30° down, in Unitree's URDF ([[2026-11-25 Camera View Dex1-1 and D435i field of view]]). The real Dex1-1 path gets this pose for free. A passive copy needs the same camera model in the same pose; whether the Dex1-1 camera can be bought separately or moved between grippers is not checked.

## Options

| | A. Keep real Dex1-1 (current) | B. Passive Dex1-1 copy (resume frozen copy path) | C. Passive base, swappable tips, as main | **D. A as main + C-style base as fallback (recommended)** |
| --- | --- | --- | --- | --- |
| What it is | Second Dex1-1 on the PICO sleeve via the real-adapter; PICO trigger commands the motor | Printed Dex1-1 copy (rack-and-pinion on MGN9 rails) with passive trigger; Dex1-1 camera on it | One passive body around the PICO; tip sets for 2F-85 (RUMI-style) and Dex1-1; camera mount per tip set | A unchanged; the frozen copy fallback is replaced by a passive base with a 2F-85 tip set first, Dex1-1 tips only if needed |
| Precedent | RAPID (same unit both sides), ActiveUMI (actuated copy) | UMI, RDT2 (unpowered copy of the robot gripper) | HandUMI (PICO 4 Ultra, 5 tip sets), RUMI (2F-85 copy), FastUMI | RUMI for the 2F-85 tip set; RAPID / ActiveUMI for the main |
| Bulk and mass | ≈ 846 g modelled; no trigger mechanism (PICO trigger is the trigger), but 550 g gripper + power/RS-485 cable | ≈ 438 g modelled + unmodelled bracket, spring, lock; trigger mechanism in front of the grip | Similar to B; YUBI shows ≈ 319 g is possible; HandUMI avoids a trigger with finger cradles | Main as A; fallback as C |
| Gripper dependence | Needs both Dex1-1 units and a passed RS-485 check | Robot still needs a Dex1-1; handheld does not | 2F-85 tips usable now; robot keeps the 2F-85 untouched (unlike FastUMI covers) | As A; fallback usable without any Dex1-1 |
| Timeline before mid-Jan (estimate, unverified) | No new CAD; wait for delivery late Dec / early Jan | Copy path "kept adding problems" (5 Nov): several weeks of CAD and prints | New mechanism plus two tip sets: likely the most CAD work; competes with RL4–RL6 (the Replay Ladder phases) | Only the 2F-85 tip set before the gate; Dex1-1 tips later or never |
| PICO tracking | Kept (proven in the prototype use test 2026-11-19) | Kept; trigger and link must leave the rim and pommel LEDs free | Kept; HandUMI is a direct precedent with PICO 4 Ultra | Kept |
| Camera view match | Exact (same gripper, same built-in camera) | Needs a second Dex1-1 camera or a moved one; pose from the URDF | Per tip set: D435i in the 2F-85 robot pose (only one D435i, on the flange: moved per session, unless a second is bought); Dex1-1 camera as B | Main exact; fallback moves the D435i |
| Thesis claim | "Handheld" is an actuated twin: smallest gap, but less like the UMI family; no haptic grip feel | Classic UMI-style passive handheld; gap from printed vs alloy jaws and hand-set grip force | Adds a modular-handheld angle (no precedent for validating two grippers); widens scope | Core claim as now; the fallback keeps the study alive on the 2F-85 if the gate fails |
| Changes a decision? | No | D01 (copy becomes main) | D01, D10 (new mechanism) | D01 fallback line only |

## Recommendation

**D**, reasoned only. Why:

1. The Dex1-1 is needed on the robot whatever the handheld is, so going passive (B or C) does not remove the delivery risk for the core data; it only trades it for new mechanism work, which the copy path showed is slow (D01 History, 2026-11-05).
2. The real path already has the smallest embodiment gap (same gripper, same camera pose, same grip force from the motor), which the matched-setup results above favour, and it needs no trigger mechanism, so it is not the bulky one in Martyn's sense; its cost is the weight (≈ 846 g, similar to UMI's 780 g, unweighed).
3. The real weakness is the fallback: with one 2F-85 on the robot, the mid-January last resort has no handheld. A passive base with 2F-85 tips (RUMI precedent, same Robotiq CAD already in Fusion "0 Reference") fixes that, and the same base can later take Dex1-1 tips (HandUMI precedent) if the Dex1-1 handheld proves too heavy.
4. Why not C as main: it is the most new CAD before the gate, and no prior work has shown one handheld validated on two robot grippers, so the modular claim would be new work, not a reuse.

Before this changes CAD or procedure: weigh the delivered Dex1-1 on the prototype (the 846 g is modelled), and check whether a second D435i or a separate Dex1-1 camera can be ordered (both decide how good D's fallback can be). Martyn decides; record any change with the record-decision skill.

Note: Martyn rejected option C (matched 2F-85 fingertips) of [[2026-11-20 Interim hardware while the Dex1-1 is delayed]] on 2026-11-20. That option put covers on the robot's fingers and would be thrown away at delivery; D puts the copy on the handheld only (robot 2F-85 untouched) and is kept as the fallback.

## Open

Placed on the roadmap and in Waiting on 2026-11-25 (all ⚪ can wait): fallback yes/no, camera availability, weighing at delivery, Zotero and positioning ([[2026-11-24 Red Line roadmap]], [[Waiting]]). Nothing changes this week; RL4 (Robotiq open/close) continues.

- HandUMI and RUMI (November 2026) were read through a summary only; read both in the original, especially RUMI's 2F-85 linkage copy and HandUMI's PICO mount and finger-cradle drive.
- FastUMI names only two of its five tip variants; fastumi.com could not be fetched.
- No source found on the Dex1-1 on any handheld, or on PICO-controller LED occlusion handling on handhelds.
- RDT2 is inconsistent about its gripper type (Table 1 "linkage" vs App. A.1 parallel-jaw replica).

## Martyn's rationale (own words, 2026-11-25, draft for the discussion section)

> Why I chose the approach is because of simplicity. The idea of exploring the world of handheld data collection comes from the team at Daimler Truck, so more or less they are trying to discover if these kind of data collection can be implemented in the industry. The team already know which robot and gripper they want to work with and what kind of use case they want to test it. That's why using the real gripper and PICO headset comes to mind, because of their simplicity, which is not adding to much hardware as much as possible. But then during research and looking into other recent works, something comes to mind: Modularity. Modularity is what makes todays robotic very attractive and they move into that direction. Toolkit such as XRoboToolkit is an example. I think, handheld data collection devices needs to be modular as well, at least on the gripper side, because currently tracking with VR controller is probably still the best option so far. This problem we also encounter because of the delayed gripper. That's why I am having a doubt, mainly because my approach isn't modular enough. But that is the trade-off on keeping things simple.

Kept verbatim as Martyn's own input (own-work record); to be rewritten in third person for the thesis.

## Add to Zotero

| Title | Link | Why |
| --- | --- | --- |
| HandUMI | https://arxiv.org/abs/2611.22146 | Passive handheld with swappable tips for 5 robot grippers; PICO 4 Ultra tracking |
| RUMI | https://arxiv.org/abs/2611.15802 | Passive copy of the Robotiq 2F-85 with swappable finger modules, Quest 3 tracking |
| YUBI | https://arxiv.org/abs/2606.10244 | Light passive handheld with a Quest controller (≈ 319 g); one gripper on 3 arms |
| Robot-Matched Soft Fingers for UMI Data Collection | https://arxiv.org/abs/2611.03779 | Robot gripper built to share the handheld's tips and camera pose |
| Influence of Gripper Design on Human Demonstration Quality | https://arxiv.org/abs/2603.17189 | Only numbers on finger shape vs demonstration quality |
| LEGATO | https://arxiv.org/abs/2411.03682 | One tool shared by human and robot; passive vs active grip mismatch |
| Mirage | https://arxiv.org/abs/2402.19249 | Success drop when the gripper in view changes; cross-painting fix |
| DexUMI | https://arxiv.org/abs/2505.21864 | Inpainting vs masking numbers for the handheld-to-robot view gap |
