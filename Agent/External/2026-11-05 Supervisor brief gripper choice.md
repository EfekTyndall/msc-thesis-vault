---
date: 2026-11-05
audience: company supervisor
type: decision brief
status: draft
---
# Handheld data collection: real Dex1-1 gripper or printed copy

Status 5 November 2026. Nothing has been weighed or tested on hardware yet; masses are CAD estimates and electrical values come from manufacturer documentation.

## Decision needed

The handheld demonstration device must show the camera the same fingers that the robot uses, so that handheld and robot images match. Two ways to achieve this are compared:

1. **Real gripper.** One of the two ordered Unitree Dex1-1 grippers is mounted on the VR controller and driven electrically from the controller trigger. The second gripper stays on the UR10e.
2. **Printed copy.** A passive, 3D-printed copy of the Dex1-1 mechanism, closed by a squeeze trigger. This is the design in the original proposal.

**Recommendation (for discussion):** the real gripper as the main line, with the printed copy kept at its current design state as a documented fallback. **Proposed rule:** the real gripper is used if a communication and power check on the delivered gripper passes; otherwise the printed copy is built.

**Main argument:** the real gripper gives identical fingers and a complete gripper signal (command, measured position, torque estimate) without designing a mechanism. **Main cost:** about 410 g more mass and a powered cable to the handheld.

## The two options

**Real gripper.** The PICO controller trigger is read on the recording PC, which commands the gripper through Unitree's RS-485-to-USB adapter and open-source gripper service [dex1_1_service]. The trigger can map to the gripper proportionally or as an open/close switch; both mappings are to be tested. The gripper is attached under the controller grip by a printed adapter. A real gripper mounted on a VR controller has been used in ActiveUMI [zeng2025]; one device serving both handheld collection and robot deployment in RAPID [yin2026].

**Printed copy.** Rack-and-pinion mechanism on two MGN9 linear rails, rebuilt from Unitree's official CAD model [Unitree docs]. A multi-finger squeeze trigger in the grip, with about 10 mm travel (design value, untested), drives the fingers through a rigid link at about 4:1, with spring return and an optional lock. Printed copies of the robot gripper are used in UMI [chi2024] and RDT2 [liu2026]; printed linkages in UMI [chi2024] and UMI-3D [wang2026]; a continuous lock in TacUMI [cheng2026].

## Comparison

| Criterion | Real gripper | Printed copy |
| --- | --- | --- |
| Fingers vs robot | Identical (same model, metal jaws) | Same geometry, but printed; friction and stiffness differ from the metal jaws |
| Gripper signal | Commanded position, measured position and torque estimate, logged at 500 Hz by the gripper service [dex1_1_service] | Finger width only, from a fiducial marker in the camera image (offline) or a magnetic angle encoder (online); no command, no force |
| Width in metres | Needs a measured motor-angle-to-width curve; Unitree's own code uses 5.62 rad [dex1_1_service] and 5.4 rad [xr_teleoperate] for the full stroke | Needs a sensor-to-width calibration |
| Grip force | 5–120 N, set by the controller [Unitree] | Operator's finger force; at 4:1, the recommended repetitive grip force of 50–60 N [ILO] allows about 12–15 N at the fingers (computed) |
| Mass incl. controller, without cables (CAD estimate) | ≈ 846 g: gripper 550 g [Unitree] + adapter 64 g + grip sleeve 32 g + controller ≈ 201 g incl. batteries (estimated) | ≈ 438 g, rising: bracket, pivots, spring, pins and lock not yet modelled |
| Reference masses | UMI 780 g [chi2024]; UMI-3D 1120 g, where weight is named as a cause of fatigue [wang2026] | |
| Power and cabling | 24–64 V DC, 3.5 A peak [Unitree] (≈ 84 W at 24 V, computed); camera USB plus one power/RS-485 cable; an extension may be needed (cable length in the box not stated) | Unpowered; camera USB (plus encoder USB) |
| Grip ergonomics | Controller trigger only; grip 56.9 mm deep (CAD) | Grip plus squeeze trigger: 54–64 mm span, above the 45–55 mm optimum [ILO] |
| Robot availability | Second gripper stays on the UR10e | Robot keeps its gripper |
| Parts to buy | 24/48 V supply (≥ 3.5 A), cable extension, threaded inserts | 2 MGN9C carriages, 2 MGN9 rails, spring, inserts; encoder if width is measured online |
| Design state | Adapter and grip sleeve designed; fit checked in CAD | Mechanism, trigger and link designed; gripper bracket open |
| Testable before delivery | Printed dummy gripper (size, adapter fit, tracking-LED visibility); software against a simulated gripper | Whole mechanism, once printed and rails bought |
| Main risk | Delivery date unknown; communication check only possible after delivery | Mechanism and width-calibration effort; printed-finger mismatch |

Costs are not yet included.

## Common to both options

- PICO 4 Ultra Enterprise controller as the handle, held like a pistol with the gripper as the barrel, rotated by 35° in a printed two-part sleeve that leaves the tracking LEDs uncovered.
- RGB camera on the gripper; recording PC connected by cable.
- Gripper size and stroke follow the official CAD model: 143.3 mm long, about 82–83 mm stroke [Unitree docs].
- Robot side: the UR10e needs a flange adapter plate (ISO 9409-1-50-4-M6) and very likely an external supply, since its tool connector provides 24 V at 2 A [UR10e] against the gripper's 3.5 A peak.

## Open points that could change the choice

- **Communication and power check** on the delivered gripper: command rate, supply, cable, handling with the tether. If it fails, the printed copy is built.
- **Stroke:** about 82–83 mm in the official CAD model versus 90 mm on the product page [Unitree docs; Unitree]; to be measured on the gripper. Affects both options equally.
- **Handheld mass:** 846 g lies between UMI and UMI-3D; operator fatigue is untested.
- **Trigger mapping:** proportional versus open/close, and reading the PICO trigger through XRoboToolkit [zhao2025].

## Outside the scope of this choice

- Which arm pose is recorded as the action (observed or commanded). DM-UMI reports that naive mixing of handheld and teleoperation data performed worse than handheld data alone on all three tasks [surendran2026, p. 5].
- Tracking accuracy and deployment compliance are the same for both options.

## References

**Papers**

- [cheng2026] T. Cheng et al., "TacUMI: A Multi-Modal Universal Manipulation Interface for Contact-Rich Tasks," 2026.
- [chi2024] C. Chi et al., "Universal Manipulation Interface: In-The-Wild Robot Teaching Without In-The-Wild Robots," 2024.
- [liu2026] S. Liu et al., "RDT2: Exploring the Scaling Limit of UMI Data Towards Zero-Shot Cross-Embodiment Generalization," 2026.
- [surendran2026] V. Surendran, N. Peri, D. Watkins, "Bridging Handheld and Teleoperated Supervision for Contact-Rich Manipulation via State-Gated Experts," arXiv:2606.26603, 2026.
- [wang2026] Z. Wang, "UMI-3D: Extending Universal Manipulation Interface from Vision-Limited to 3D Spatial Perception," 2026.
- [yin2026] Z. Yin et al., "RAPID: Reconfigurable, Adaptive Platform for Iterative Design," arXiv:2602.06653, 2026.
- [zeng2025] Q. Zeng et al., "ActiveUMI: Robotic Manipulation with Active Perception from Robot-Free Human Demonstrations," 2025.
- [zhao2025] Z. Zhao et al., "XRoboToolkit: A Cross-Platform Framework for Robot Teleoperation," 2025.

**Manufacturer documentation, open-source code, guidelines**

- [Unitree] Unitree Robotics, Dex1-1 product page, https://unitree.com/Dex1-1
- [Unitree docs] Unitree Robotics, Dex1-1 developer documentation and CAD model, https://support.unitree.com/home/en/dex1-1_gripper/about_dex1_1
- [dex1_1_service] Unitree Robotics, dex1_1_service, https://github.com/unitreerobotics/dex1_1_service
- [xr_teleoperate] Unitree Robotics, xr_teleoperate, https://github.com/unitreerobotics/xr_teleoperate
- [UR10e] Universal Robots, UR10e datasheet, https://www.universal-robots.com/manuals/latest/en/datasheets/ur10e/
- [ILO] ILO Encyclopaedia of Occupational Health and Safety, Ergonomics: hand tools, https://iloencyclopaedia.org/part-iv-66769/ergonomics-52353/item/634
