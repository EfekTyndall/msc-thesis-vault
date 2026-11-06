---
date: 2026-11-05
audience: company supervisor
type: decision brief
status: draft
---
# Handheld data collection: Real Dex1-1 Gripper or Printed Copy

> Nothing has been weighed or tested on hardware yet; masses are CAD estimates and electrical values come from manufacturer documentation.

## Decision needed

The handheld demonstration device must show the camera the same fingers that the robot uses, so that handheld and robot images match. Two ways to achieve this are compared:

1. **Real gripper.** One of the two ordered Unitree Dex1-1 grippers is mounted on the VR controller and driven electrically from the controller trigger.
2. **Printed copy.** A passive, 3D-printed copy of the Dex1-1 finger mechanism, closed by hand. This is the concept in the original proposal.

**Recommendation (for discussion):** the real gripper as the main approach, with the printed copy kept as a fallback at concept level.

**Proposed rule:** the real gripper is used if a communication and power check on the delivered gripper passes; otherwise the printed copy is built.

**Main argument:** the real gripper gives identical fingers and a complete gripper signal (command, measured position, torque estimate) without designing a mechanism. 

**Main cost:** about 410 g more mass and a powered cable to the handheld.

## The two options

**Real gripper.** The PICO controller trigger is read on the recording PC, which commands the gripper through Unitree's RS-485-to-USB adapter and open-source gripper service ([Unitree dex1_1_service](https://github.com/unitreerobotics/dex1_1_service)). The trigger can map to the gripper proportionally or as an open/close switch; both mappings are to be tested. The gripper is attached under the controller grip by a printed adapter. A real gripper mounted on a VR controller has been used in ActiveUMI ([Zeng et al., 2025](https://arxiv.org/abs/2510.01607)); one device serving both handheld collection and robot deployment in RAPID ([Yin et al., 2026](https://arxiv.org/abs/2602.06653)).

**Printed copy.** The copy reuses the finger mechanism of the Dex1-1, a rack and pinion, rebuilt from Unitree's official CAD model ([Unitree developer docs](https://support.unitree.com/home/en/dex1-1_gripper/about_dex1_1)). A pinion is a small gear; a rack is a straight bar with teeth. Each finger is fixed to its own rack, and both racks mesh with one central pinion from opposite sides (Figure 1). When the pinion turns, the two racks move by the same distance in opposite directions, so both fingers close or open together and stay centred. In the robot gripper the motor turns the pinion. In the printed copy there is no motor: a trigger in the grip pushes one finger, and the pinion makes the other finger follow by the same amount. Each rack runs on a miniature linear rail, as in the original. Printed copies of the robot gripper are used in UMI ([Chi et al., 2024](https://arxiv.org/abs/2402.10329)) and RDT2 ([Liu et al., 2026](https://arxiv.org/abs/2602.03310)).

![Rack-and-pinion principle](figures/2026-11-06%20rack%20and%20pinion%20principle.png)

*Figure 1. Rack-and-pinion principle of the Dex1-1 finger mechanism, which the printed copy reuses (schematic, not to scale). Left: open. Right: the pinion turns clockwise, rack 1 moves right and rack 2 moves left, so the fingers close symmetrically. Layout after Unitree's official CAD model ([developer docs](https://support.unitree.com/home/en/dex1-1_gripper/about_dex1_1)).*

## Design state in CAD

![Real gripper option, side view](figures/2026-11-05%20real%20gripper%20option%20side.png)

*Figure 2. Real gripper option, side view (CAD, 5 November 2026). The gripper (blue, Unitree's official CAD model from the [developer docs](https://support.unitree.com/home/en/dex1-1_gripper/about_dex1_1)) is attached by a printed adapter to a two-part sleeve on the PICO controller. Controller shape: 3D scan "Pico 4 Ultra controller scan" by alonym, [Thingiverse 7350019](https://www.thingiverse.com/thing:7350019), CC BY.*

![Real gripper option, isometric view](figures/2026-11-05%20real%20gripper%20option%20iso.png)

*Figure 3. Real gripper option, isometric view (same model as Figure 2).*

The printed copy is described at concept level only (Figure 1); its detailed design is not shown.

## Comparison

| Criterion | Real gripper | Printed copy |
| --- | --- | --- |
| Fingers vs robot | Identical (same model, metal jaws) | Same geometry, but printed; friction and stiffness differ from the metal jaws |
| Gripper signal | Commanded position, measured position and torque estimate, logged at 500 Hz by the gripper service ([Unitree dex1_1_service](https://github.com/unitreerobotics/dex1_1_service)) | Finger width only, from a fiducial marker in the camera image (offline) or a magnetic angle encoder (online); no command, no force |
| Width in metres | The gripper reports its finger position as a motor angle in radians, not as a finger gap in metres. The angle-to-width curve must be measured on the delivered gripper, because Unitree's own code disagrees on the motor angle for the full stroke: 5.62 rad in the gripper service ([Unitree dex1_1_service](https://github.com/unitreerobotics/dex1_1_service)) and 5.4 rad in the teleoperation code ([Unitree xr_teleoperate](https://github.com/unitreerobotics/xr_teleoperate)) | No built-in measurement: an added sensor (a marker on the fingers seen by the camera, or an angle sensor on the pinion) gives a raw reading that must be related to the true finger gap by measuring known gaps |
| Grip force | 5–120 N ([Unitree product page](https://unitree.com/Dex1-1)). Not commanded as a force: the recording PC sends a target finger position and a stiffness, and the gripper's built-in motor controller pushes harder the further an object keeps the fingers from that target ([Unitree dex1_1_service](https://github.com/unitreerobotics/dex1_1_service)) | Operator's hand force through trigger and link. For repetitive gripping, about 50–60 N is recommended for women and 90–100 N for men ([ILO Encyclopaedia, hand tools](https://iloencyclopaedia.org/part-iv-66769/ergonomics-52353/item/634)); at the design link ratio of about 4:1 the lower figure allows about 12–15 N at the fingers (computed) |
| Mass incl. controller, without cables (CAD estimate) | ≈ 846 g: gripper 550 g ([Unitree product page](https://unitree.com/Dex1-1)) + adapter 64 g + grip sleeve 32 g + controller ≈ 201 g (155 g per a [retailer listing](https://vr-expert.com/accessories/vr-accessories/pico-4-ultra-controller-l/) plus batteries, estimated) | ≈ 438 g, rising: bracket, pivots, spring, pins and lock not yet modelled |
| Reference masses | UMI 780 g ([Chi et al., 2024](https://arxiv.org/abs/2402.10329)); UMI-3D 1120 g, where weight is named as a cause of fatigue ([Wang, 2026](https://arxiv.org/abs/2604.14089)) | |
| Power and cabling | 24–64 V DC, 3.5 A peak ([Unitree product page](https://unitree.com/Dex1-1)) (≈ 84 W at 24 V, computed); camera USB plus one power/RS-485 cable; an extension may be needed (cable length in the box not stated) | Unpowered; camera USB (plus encoder USB) |
| Grip ergonomics | Index finger on the controller's own trigger; the hand closes around the sleeve, which measures 56.9 mm from the back of the grip to the front (CAD estimate) | The hand closes around sleeve plus squeeze trigger: about 54–64 mm (CAD estimate), above the 45–55 mm span at which grip strength and comfort are highest ([ILO Encyclopaedia, hand tools](https://iloencyclopaedia.org/part-iv-66769/ergonomics-52353/item/634)) |
| Robot availability | One of the two ordered grippers becomes the handheld; one gripper is enough to validate the concept (trigger, command, gripper signal, power, communication). Grippers needed on the robot depend on the deployment robot, not yet decided | Both ordered grippers remain available for the robot |
| Parts to buy | 24/48 V supply (≥ 3.5 A), cable extension, threaded inserts | 2 MGN9C carriages, 2 MGN9 rails, spring, inserts; encoder if width is measured online |
| Design state | Adapter and grip sleeve designed; fit checked in CAD | Concept worked out; design not final |
| Testable before delivery | Printed dummy gripper (size, adapter fit, tracking-LED visibility); software against a simulated gripper | Whole mechanism, once printed and rails bought |
| Main risk | Delivery date unknown; communication check only possible after delivery | Mechanism and width-calibration effort; printed-finger mismatch |

Costs are not yet included.

## Common to both options

- PICO 4 Ultra Enterprise controller as the handle, held like a pistol with the gripper as the barrel. A printed two-part sleeve holds the controller tilted by 35° relative to the gripper, so that the grip slants back like a pistol grip (CAD; comfort untested), and leaves the tracking LEDs uncovered.
- RGB camera on the gripper; recording PC connected by cable.
- Gripper size and stroke follow the official CAD model: 143.3 mm long, about 82–83 mm stroke ([Unitree developer docs](https://support.unitree.com/home/en/dex1-1_gripper/about_dex1_1)).
- Deployment robot: not yet decided between the UR10e and the Unitree H2 (possibly with a gripper on each hand); trade-offs not yet weighed. Known for the UR10e: it needs a flange adapter plate (ISO 9409-1-50-4-M6) and very likely an external supply, since its tool connector provides 24 V at 2 A ([UR10e datasheet](https://www.universal-robots.com/manuals/latest/en/datasheets/ur10e/)) against the gripper's 3.5 A peak.

## Open points that could change the choice

- **Concept check on one delivered gripper:** trigger reading, commands and command rate, gripper signal, power supply, communication, cable and handling with the tether. If it fails, the printed copy is built. Data collection and deployment are planned after it passes.
- **Stroke:** about 82–83 mm in the official CAD model versus 90 mm on the product page ([Unitree developer docs](https://support.unitree.com/home/en/dex1-1_gripper/about_dex1_1); [Unitree product page](https://unitree.com/Dex1-1)); to be measured on the gripper. Affects both options equally.
- **Handheld mass:** 846 g lies between UMI and UMI-3D; operator fatigue is untested.
- **Trigger mapping:** proportional versus open/close, and reading the PICO trigger through XRoboToolkit ([Zhao et al., 2025](https://arxiv.org/abs/2508.00097)).

## Outside the scope of this choice

- Which arm pose is recorded as the action (observed or commanded). DM-UMI reports that naive mixing of handheld and teleoperation data performed worse than handheld data alone on all three tasks ([Surendran et al., 2026, p. 5](https://arxiv.org/abs/2606.26603)).
- Tracking accuracy and deployment compliance are the same for both options.

## References

**Papers**

- C. Chi et al., "Universal Manipulation Interface: In-The-Wild Robot Teaching Without In-The-Wild Robots," 2024, [arXiv:2402.10329](https://arxiv.org/abs/2402.10329).
- S. Liu et al., "RDT2: Exploring the Scaling Limit of UMI Data Towards Zero-Shot Cross-Embodiment Generalization," 2026, [arXiv:2602.03310](https://arxiv.org/abs/2602.03310).
- V. Surendran, N. Peri, D. Watkins, "Bridging Handheld and Teleoperated Supervision for Contact-Rich Manipulation via State-Gated Experts," 2026, [arXiv:2606.26603](https://arxiv.org/abs/2606.26603).
- Z. Wang, "UMI-3D: Extending Universal Manipulation Interface from Vision-Limited to 3D Spatial Perception," 2026, [arXiv:2604.14089](https://arxiv.org/abs/2604.14089).
- Z. Yin et al., "RAPID: Reconfigurable, Adaptive Platform for Iterative Design," 2026, [arXiv:2602.06653](https://arxiv.org/abs/2602.06653).
- Q. Zeng et al., "ActiveUMI: Robotic Manipulation with Active Perception from Robot-Free Human Demonstrations," 2025, [arXiv:2510.01607](https://arxiv.org/abs/2510.01607).
- Z. Zhao et al., "XRoboToolkit: A Cross-Platform Framework for Robot Teleoperation," 2025, [arXiv:2508.00097](https://arxiv.org/abs/2508.00097).

**Manufacturer documentation, open-source code, guidelines, models**

- [Unitree Robotics, Dex1-1 product page](https://unitree.com/Dex1-1)
- [Unitree Robotics, Dex1-1 developer documentation and CAD model](https://support.unitree.com/home/en/dex1-1_gripper/about_dex1_1)
- [Unitree Robotics, dex1_1_service](https://github.com/unitreerobotics/dex1_1_service)
- [Unitree Robotics, xr_teleoperate](https://github.com/unitreerobotics/xr_teleoperate)
- [Universal Robots, UR10e datasheet](https://www.universal-robots.com/manuals/latest/en/datasheets/ur10e/)
- [ILO Encyclopaedia of Occupational Health and Safety, Ergonomics: hand tools](https://iloencyclopaedia.org/part-iv-66769/ergonomics-52353/item/634)
- ["Pico 4 Ultra controller scan" by alonym, Thingiverse 7350019, CC BY](https://www.thingiverse.com/thing:7350019)

[^1]: 
