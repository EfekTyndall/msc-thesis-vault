---
date: 2026-11-02
tags: [research, handheld]
---
# How UMI-style handhelds open and close the gripper

Question: how do published handheld data-collection grippers actuate the fingers, and what does that mean for the Dex1-1 handheld, where the operator holds the PICO controller like a pistol grip ([[D10 Handheld mechanism]])? Concern: squeeze force and operator fatigue over long collection sessions.

Method: hardware sections of the 11 works in `msc-thesis/thesis/literature.bib`, the UMI Hardware Guide, and the hardware figures where the text is silent (marked "figure"). Most papers describe the mechanism in one sentence or not at all.

## Findings

| Work | Actuation | Trigger / how the hand drives the fingers | Other details | Source |
| --- | --- | --- | --- | --- |
| UMI (chi2024) | Passive | "Trigger-activated" parallel jaw; fingers and direction not stated | BOM: 2 × MGN9C 150 mm rails, left/right gears, left/right linkages, compression spring (10 mm OD, 0.7 mm wire, 50 mm free length); 80 mm stroke; 780 g; bulk and weight called less efficient than bare hands (Sec. VIII) | Paper Sec. III-A; [Hardware Guide](https://docs.google.com/document/d/1TPYwV9sNVPAi0ZlAupDMkXZ4CA1hsZx7YDMSmcEy6EU/edit) |
| FastUMI (zhaxizhuoma2025) | Passive (no motor mentioned) | Pistol grip with a long trigger along the front of the grip, pulled by several fingers (figure) | "Parallel-motion gripper"; width from fingertip ArUco markers | Sec. II-C, Fig. 1 |
| VISTA (yang2026) | Uses FastUMI Pro | "Handheld trigger"; width from an internal encoder | About 600 g | Sec. 3.1 |
| ViTaMIn-B (li2025) | Passive | Handles attached to the finger sliders on a guide rail, with a gear synchronising the two sides: the hand moves along the finger direction (figure); the Quest controller sits on a connector at the back, it is not the grip | "Lighter and mechanically simpler, featuring fewer transmission components" than UMI; fatigue reduced by removing the onboard computer | Sec. IV-A, Fig. 4 |
| TacUMI (cheng2026) | Passive | UMI-style rack-and-pinion trigger "indexed to the forefingers"; closure proportional to trigger effort | Continuous self-locking part: thumb engages, index releases; holds the aperture "without sustaining trigger pressure"; centre of mass near the palm | Sec. II-A, III-A, Fig. 2 |
| ActiveUMI (zeng2025) | Micro-motor on a Quest 3S controller | Motor drives open/close; how it is commanded is not stated | Motor and power supply not specified; a backpack holds a battery and a PC (whether it powers the motor is not stated) | Sec. 3.1, Fig. 2 |
| RDT2 (liu2026) | Passive | Non-actuated replica of the robot gripper on a sliding rail with mechanical limit switches; trigger not described | Redesign uses a linkage gripper (Table 1) | Sec. 3–4, App. A.1 |
| UMI-3D (wang2026) | Passive | "Trigger-activated" parallel jaw | 80 mm stroke; 1120 g; weight "can lead to user fatigue during prolonged data collection" | Sec. III-A, VI |
| HiFi-UMI (ai2026) | Passive, no trigger | Glove-like full-palm gripper: thumb against four fingers, the hand closes it directly | Encoders measure the opening; argues triggers give weaker contact feedback | Sec. 3.1.2–3.1.3 |
| UMI-FT (choi2026) | Not described | Not described | Width from ArUco markers | Sec. III-A |
| RAPID (yin2026) | Probably motorised (motor mounting, USB2CAN), not stated for handheld use | Not described | – | Sec. III-A |

## What follows for the Dex1-1 handheld

1. **Motor (ActiveUMI) stays out.** It needs power on the handheld, which D01 excludes; the paper does not say how it is powered or commanded.
2. **Direct drive (ViTaMIn-B, HiFi-UMI) does not fit a pistol hold.** It works because the hand moves along the finger direction. In a pistol hold the fingers pull towards the palm, along the barrel, while the Dex1-1 fingers move sideways, so some transmission is always needed.
3. **The multi-finger trigger is the established pistol-grip solution** (FastUMI, and VISTA through FastUMI Pro). UMI's mechanism (rails, gears, linkages, compression spring) is the same family as the planned Dex1-1 copy (MGN9 rails, rack-and-pinion).
4. **TacUMI's continuous lock addresses sustained effort directly:** the operator locks the aperture while carrying an object instead of holding the trigger. It changes nothing in the recorded width. Not reported elsewhere.
5. **Weight is the fatigue factor the papers name** (UMI-3D at 1120 g, UMI's bulk); none measures trigger force or fatigue.

Recommendation (to be confirmed by Martyn): a multi-finger squeeze trigger along the front of the controller sleeve, driving the pinion through a cable and drum, with a weak return spring, a stroke stop sized to the task parts, and a TacUMI-style lock as an option for long holds. Measure squeeze force and fatigue on the first print (`msc-thesis/hardware/PRINT-TEST-QUEUE.md`).

## Open

Open items reviewed on 2026-11-04: the relevant ones are in the TODO of [[2026-11-04]], the rest were dropped.
