---
date: 2026-11-05
tags: [draft, decision, d01]
---
# D01 decision brief: real Dex1-1 or printed copy as the handheld

Companion to the mind map `Agent/Drafts/2026-11-05 D01 mind map.drawio` (visual overview); this note holds the details. Scope: what is known and done as of 2026-11-05, limited to what is needed to choose a path. Nothing has been weighed or tested on hardware; datasheet and CAD values are marked as such. Decision record: [[D01 Handheld gripper body]].

## Summary

**Decision needed:** what the operator holds during handheld data collection, so that handheld and robot camera images match.

1. **Path 1: real Dex1-1.** One of the two ordered grippers is held on the PICO controller; the controller trigger drives it over RS-485 from the recording PC. The other gripper stays on the UR10e.
2. **Path 2: printed copy.** A printed passive copy of the Dex1-1 mechanism, closed by a squeeze trigger. This is the design in the approved proposal (`msc-thesis/docs/proposal.md` §1).

**Recommendation (for discussion):** the real Dex1-1 as the main line; the printed copy frozen at its current state as the documented fallback. **Proposed rule:** real Dex1-1 if the RS-485 check on the delivered gripper passes; otherwise the printed copy. Before delivery, the real path is advanced by a dummy-gripper print and a software pipeline tested against a mock gripper ([[D01 Handheld gripper body]] History 2026-11-05).

**Main reason:** the real gripper gives identical fingers and a full gripper signal (command, achieved width, torque estimate) without designing a mechanism; the copy path kept adding unmodelled parts. **Main cost:** about 410 g more mass and a powered tether on the handheld.

## The two paths

**Real Dex1-1.** Chain: PICO trigger → recording PC → `rt/dex1/<side>/cmd` → Unitree `dex1_1_service` → USB → RS-485-to-USB board → gripper ([[2026-11-03 Dex1-1 technical note]]). The trigger maps to the gripper proportionally or as a binary switch, a software choice; both are to be tested. The gripper attaches to the grip sleeve through real-adapter, an ActiveUMI-style bow (zeng2025) to a boss under the sleeve, joined by 2 × M3 heat-set inserts ([[D10 Handheld mechanism]]). Precedent for a real gripper on a VR controller: ActiveUMI (zeng2025); one device in both roles: RAPID ([arXiv 2602.06653](https://arxiv.org/pdf/2602.06653), as cited in D01).

**Printed copy.** Rack-and-pinion on MGN9 rails, rebuilt from the official Dex1-1 STEP. A multi-finger squeeze trigger, recessed in the sleeve front with about 10 mm travel (unverified), drives one carriage through a rigid link (about 4:1); spring return and an optional continuous lock ([[D10 Handheld mechanism]]). Precedent: printed copies of the robot gripper in UMI (chi2024) and RDT2 (liu2026); printed linkage in UMI and UMI-3D (wang2026); continuous lock instead of a ratchet in TacUMI (cheng2026) ([[2026-11-05 Trigger-to-finger transmission survey]]).

## Comparison

| Criterion | Real Dex1-1 | Printed copy | Source |
| --- | --- | --- | --- |
| Fingers vs robot | Identical (same model, alloy jaws) | Same geometry, printed; grip differently from alloy jaws | D01 Options |
| Gripper signal | Command, achieved position `q` and torque estimate `tau_est` at 500 Hz, recorded for free | Achieved width only: ArUco from the camera (offline) or AS5600 encoder (online); no command, no force | Technical note; D05 |
| Width in metres | Needs a measured rad → width curve (sources disagree: 5.4 vs 5.62 rad full stroke) | Needs a sensor → width mapping (ArUco or AS5600) | Technical note; [[2026-11-02 Research gaps]] B4 |
| Grip force | 5–120 N, set by the controller gains | Operator's finger force; at 4:1, the 50–60 N repetitive grip limit allows about 12–15 N at the carriage (computed) | [Unitree](https://unitree.com/Dex1-1); [[2026-11-05 Trigger travel and grip span]] (ILO, not in bib) |
| Mass without cables (CAD, unweighed) | ≈ 846 g: gripper 550 g + adapter 63.5 g + sleeve 31.8 g + controller ≈ 201 g | ≈ 438 g, rising: bracket, pivots, spring, pins, lock not modelled | [[2026-11-05 D01 real vs copy side-by-side]] |
| For scale | UMI 780 g (chi2024); UMI-3D 1120 g, weight named as a fatigue cause (wang2026) | | [[2026-11-02 Handheld trigger survey]] |
| Power and tether | 24–64 V DC, 3.5 A peak (≈ 84 W at 24 V, computed); camera USB + one power/RS-485 cable; cables listed in the box, length unstated, so a 3–5 m extension may be needed | Unpowered; camera USB (+ encoder USB) | [Unitree](https://unitree.com/Dex1-1); technical note; proposal §6 |
| Grip ergonomics | Controller trigger only; sleeve 56.9 mm deep | Sleeve 56.9 mm deep plus the squeeze blade: about 54–64 mm span vs a 45–55 mm optimum | [[2026-11-05 Trigger travel and grip span]] (Lee et al. 2009, ILO; not in bib) |
| Robot availability | Second gripper stays on the UR10e | Robot keeps its gripper | D01 History 2026-11-04 |
| Parts to buy | Supply (24/48 V, ≥ 3.5 A), cable extension, M3 inserts | 2 MGN9C blocks + 2 MGN9 rails (on hold), spring, inserts; AS5600 if online width | Side-by-side; [[Waiting]] |
| CAD state | real-adapter and controller-grip done; fit checked in Fusion | Mechanism, trigger and link A modelled; gripper bracket empty; frozen | Side-by-side; D10 History |
| Testable before delivery | Dummy-gripper print (size, adapter fit, LED visibility); software against a mock | Whole mechanism, once printed and the rails are bought | D01 History 2026-11-05 |
| Main risk | Delivery date unknown; RS-485 check not yet possible | Mechanism and width-mapping work; printed-finger mismatch | [[Waiting]] |

Prices are left out: company orders go through approved suppliers via the internal Storefront, not yet accessible ([[Waiting]]).

## Same on both paths

- PICO 4 Ultra Enterprise controller as the grip, held like a pistol with the gripper as the barrel, rotated +35° in a printed two-part sleeve that leaves the rim and pommel LEDs uncovered (D10; LED count unverified).
- RGB camera on the gripper (D03), tethered recording PC (D06).
- Gripper size and stroke: the copy is built from the official STEP, so both match the official model (143.3 mm long, about 82–83 mm stroke).
- Robot side: the UR10e needs a flange adapter plate (ISO 9409-1-50-4-M6) and, very likely, an external supply, since its tool connector gives 24 V at 2 A ([UR10e datasheet](https://www.universal-robots.com/manuals/latest/en/datasheets/ur10e/), not in bib) against the gripper's 3.5 A peak. This applies to both paths.

## Open items that could change the choice

- **RS-485 check** on the delivered gripper: command rate, supply, cable, mass on a tethered handheld. Fails → printed copy.
- **Stroke** 82–83 mm (official STEP/URDF) vs 90 mm (product page): measure on the gripper; affects both paths equally.
- **Handheld mass** acceptance: 846 g is above UMI and below UMI-3D; operator fatigue untested.
- **Trigger mapping**, proportional vs binary, and PICO trigger readout via XRoboToolkit (real path).

## Not decided by D01

- Which arm pose is recorded as the action (observed vs desired); DM-UMI found naive mixing of handheld and teleop data worse than handheld alone on all three tasks (surendran2026, p. 5).
- Tracking accuracy (D04) and deployment compliance (D08): the same on both paths.

## After the decision

Update D01, D05 and D10 with the record-decision skill and OK the matching thesis-plan rows; buy the MGN9 parts only if the copy stays; drop the unused Fusion assembly ([[Waiting]], After D01 is settled).
