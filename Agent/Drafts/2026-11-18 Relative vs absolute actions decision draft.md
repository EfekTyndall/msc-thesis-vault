---
date: 2026-11-18
tags: [draft, decision, actions, calibration]
---
# Relative vs absolute actions: decision draft

Draft for Martyn, not a recorded decision. If accepted, it is recorded as an update to [[D07 Software and data format]] and [[D09 Calibration]] with the record-decision skill.

Background notes: [[2026-11-17 Controller pose to robot frame]], [[2026-11-18 GR00T and pi0.5 action format for handheld data]], [[2026-11-18 UMI variants pose tracking and policy actions]].

## What is already decided

- D07 / proposal §7 (`msc-thesis/docs/proposal.md` L171–173): actions follow UMI, "end-effector poses relative to the current pose plus absolute gripper width"; export stays **absolute** (position + 6D rotation + width, 30 Hz) and each framework derives relative actions itself.
- D09 / proposal §9: controller → gripper-tip transform by hand-eye on the UR10e (ViTaMIn-B method, 10–20 poses, `cv2.calibrateRobotWorldHandEye`), once per build.

## Found inconsistency

`msc-thesis/docs/thesis-plan.md` L29 lists "relative actions" as a **stretch goal**, while D07 and the proposal make relative actions the core representation. One of the two needs changing (thesis plan changes need Martyn's OK).

## Open points not covered by D07 / D09

1. **Room → robot base link per session.** The PICO room frame is re-made at every app start or recentre ([[2026-11-17 Controller pose to robot frame]]). D09 lists no per-session step for it.
2. **State (proprioception) input.** GR00T N1.7 and openpi feed the current *absolute* pose to the model even when actions are relative ([[2026-11-18 GR00T and pi0.5 action format for handheld data]]). Handheld state in the room frame does not match robot state in the base frame.

## Options

| Option | What it means | Calibration needed | Sources | Verdict |
| --- | --- | --- | --- | --- |
| A. Relative actions + absolute state in robot base | Keep D07 export; transform handheld poses to the base frame before export | Controller → tip (once per build) **and** room → base every session | li2025 hand-eye (solves both at once); zeng2025 docking jig; FastUMI aligns T265 to base (zhaxizhuoma2025, method unstated) | Works with GR00T / openpi defaults; adds a per-session step and its error |
| B. Relative actions + relative or no state | Export state relative to the episode start or current pose, as UMI does | Controller → tip only | chi2024 (relative proprioception, calibration-free); HiFi-UMI ai2026 (relative actions with π0.5; state handling not checked) | No per-session calibration; framework support for relative/no state not checked |
| C. Absolute actions in robot base | Train on absolute targets | Controller → tip and room → base every session | zeng2025 (absolute, π0; base link not stated); chi2024 absolute baseline did worse (calibration error) | Not recommended: against GR00T default and UMI result |

Common to all: the D04/D09 accuracy test (controller vs UR10e TCP) needs the room → base link once per test anyway, and the same hand-eye run gives it.

## Recommendation (draft)

**B if GR00T / openpi accept relative or no state cleanly, otherwise A.** Next check: whether the GR00T N1.7 modality config and openpi transforms allow relative or no EEF state (not checked yet). For the comparison, the VR teleop data use the same representation.

## To decide (Martyn)
1. Fix the thesis-plan / D07 inconsistency (relative actions core or stretch).
2. A or B (after the framework check).
