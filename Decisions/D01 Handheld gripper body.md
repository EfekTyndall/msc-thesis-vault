---
date: 2026-10-19
status: accepted
tags: [decision]
---
## Context

What the operator holds: how the handheld reproduces the Dex1-1 so that handheld and robot camera images match. Source: `msc-thesis/docs/proposal.md` §1 (approved by Supervisor on 19 Oct 2026).

## Options

| Option                                                                                    | Strengths                                                                                                          | Weaknesses                                                                                                                                                                      | Verdict                                                                                    |
| ----------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------ | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------ |
| 3D-printed copy                                                                           | Robot stays available; light; cheap; no motor power on the handheld                                                | Camera moved between robot and handheld per session; printed fingers grip differently from alloy jaws; mechanism must be designed                                               | **Fallback** since 2026-11-16 (frozen since 2026-11-05); was Recommended                   |
| Swapping the two real Dex1-1s                                                             | Exact match (same unit); only 2 grippers; [RAPID](https://arxiv.org/pdf/2602.06653) uses one device for both roles | Robot unavailable during collection; remounting must be repeatable; handheld needs 24–64 V ([Unitree](https://unitree.com/Dex1-1)), RS-485 and a trigger-driven motor; no spare | **Chosen** 2026-11-16 (supervisor); was Fallback                                           |
| Two extra Dex1-1s as handhelds ([ActiveUMI](https://arxiv.org/pdf/2510.01607)-style copy) | Exact match; robot stays available                                                                                 | 550 g ([Unitree](https://unitree.com/Dex1-1)) plus motor power on each handheld; doubles gripper cost and lead time                                                             | Only if the printed copy proves too inaccurate                                             |

## Decision

The real Dex1-1 is the handheld gripper body (driven over RS-485 by the station computer, trigger as command), mounted on the PICO controller grip sleeve through the real-adapter ([[D10 Handheld mechanism]]). The 3D-printed passive copy stays the documented fallback for the worst case, frozen at its 2026-11-05 state; its use is considered unlikely (supervisor and Martyn, 2026-11-16).

## Evidence

[UMI](https://arxiv.org/pdf/2402.10329) and [RDT2](https://arxiv.org/pdf/2602.03310) both use a printed copy of the robot gripper; only the fingers and the camera position must match what the camera sees, and the robot keeps its gripper during collection.

Real gripper (2026-11-16): the supervisor accepted the real-Dex1-1 path at the 2026-11-16 talk ([[2026-11-16 Yanmei]]), after the brief and mind map of 2026-11-06 ([[2026-11-05 D01 real vs copy side-by-side]]). The same unit as on the robot gives an exact finger and camera match, as in [RAPID](https://arxiv.org/pdf/2602.06653) and the ActiveUMI-style use of real grippers ([ActiveUMI](https://arxiv.org/pdf/2510.01607)). Costs from the side-by-side: about 846 g incl. controller vs about 438 g for the copy (older adapter estimate), a powered cable, and an RS-485 link; not yet checked on the delivered gripper.

## Revisit if

The RS-485 check on the delivered Dex1-1 fails, the delivery slips too far, or the handheld proves too heavy in use: then resume the printed copy (fallback).

## History

- 2026-10-19: accepted (proposal approved)
- 2026-10-28: the replica copies the Dex1-1 that sits on the UR10e; passive trigger confirmed, no motor (Martyn)
- 2026-11-01: mechanism details moved to [[D10 Handheld mechanism]]
- 2026-11-04: still open in practice, decision kept until the supervisor decides: the real Dex1-1 as handheld is worked on in parallel with the printed copy; the printed copy is the test rig until delivery, or the full fallback; two grippers are ordered (one pair = two grippers); delivery date unknown, so the RS-485 check waits (Martyn; supervisor on leave until early 2027, email only)
- 2026-11-05: work order changed, decision itself kept until the supervisor decides: the real Dex1-1 path becomes the main line before delivery (dummy gripper printed from the official STEP for fit, balance and LED check; trigger software against a mock gripper); the printed copy is frozen at its current state (link A modelled, bracket not started, see [[D10 Handheld mechanism]]) as the documented fallback if delivery slips or the RS-485 check fails (Martyn; the copy path kept adding open parts)
- 2026-11-16: decided: real Dex1-1 as the handheld, printed copy as fallback (unlikely to be used); delivery date still unknown (supervisor at the 2026-11-16 talk, [[2026-11-16 Yanmei]]; reported by Martyn)
