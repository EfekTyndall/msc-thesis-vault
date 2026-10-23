---
date: 2026-10-19
status: accepted
tags: [decision]
---
## Context
What the operator holds: how the handheld reproduces the Dex1-1 so that handheld and robot camera images match. Source: `msc-thesis/docs/proposal.md` §1 (approved by Yanmei on 19 Oct 2026).

## Options
| Option | Strengths | Weaknesses | Verdict |
| --- | --- | --- | --- |
| 3D-printed copy | Robot stays available; light; cheap; no motor power on the handheld | Camera moved between robot and handheld per session; printed fingers grip differently from alloy jaws; mechanism must be designed | **Recommended**, reusing the camera and mount supplied with the gripper |
| Swapping the two real Dex1-1s | Exact match (same unit); only 2 grippers; [RAPID](https://arxiv.org/pdf/2602.06653) uses one device for both roles | Robot unavailable during collection; remounting must be repeatable; handheld needs 24–64 V ([Unitree](https://unitree.com/Dex1-1)), RS-485 and a trigger-driven motor; no spare | **Fallback** if the camera cannot be remounted repeatably or the replica proves inaccurate |
| Two extra Dex1-1s as handhelds ([ActiveUMI](https://arxiv.org/pdf/2510.01607)-style copy) | Exact match; robot stays available | 550 g ([Unitree](https://unitree.com/Dex1-1)) plus motor power on each handheld; doubles gripper cost and lead time | Only if the printed copy proves too inaccurate |



## Decision
A 3D-printed passive copy of the Dex1-1, matching only what the camera sees.

## Evidence
[UMI](https://arxiv.org/pdf/2402.10329) and [RDT2](https://arxiv.org/pdf/2602.03310) both use a printed copy of the robot gripper; only the fingers and the camera position must match what the camera sees, and the robot keeps its gripper during collection.

## Revisit if
Printed copy proves too inaccurate.

## History
- 2026-10-19: accepted (proposal approved)
