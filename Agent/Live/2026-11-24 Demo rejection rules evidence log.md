---
date: 2026-11-24
tags: [research, rejection, tracking, replay-ladder]
---
# Demo rejection rules: evidence log

Topic: rules for throwing away bad demonstrations, the same for handheld and VR (gap A6 (dropouts and episode rejection), [[2026-11-02 Research gaps]]).

Approach (Martyn, 2026-11-24): prior work is the starting reference; it gets adjusted to our setup based on our own tests. Every test already done or planned that bears on rejection is logged here with its result. The decision is made later, after the Replay Ladder; nothing is decided in this note.

## 1. Prior work (starting reference)

Read in the arXiv HTML versions on 2026-11-24.

| Check | Source | What they do | Numbers published |
| --- | --- | --- | --- |
| Tracking confidence | FastUMI (zhaxizhuoma2025, III-B) | T265 gives Failed / Low / Medium / High per pose. Before recording, the room is accepted only if ≥ 95 % of sample poses are High. During recording, low-confidence poses are removed and interpolated from neighbours; sequences hit by drift are discarded or re-recorded | 95 % High (room check only) |
| Pose jumps | FastUMI (zhaxizhuoma2025, III-B) | "user-defined thresholds on velocity, acceleration, and relative orientation identify abrupt transitions" | None |
| Episode filter list | HiFi-UMI (ai2026, Sec. 5.5) | Episode removed on any of: SLAM tracking failure, missing camera frames, timestamp discontinuity, severe hand-pose outliers, action spikes above a physical threshold, incomplete task execution, inconsistent gripper-state reconstruction | None |
| Robot reach (simulation replay) | HiFi-UMI (ai2026, Sec. 3.3.3) | Every trajectory replayed in simulation on the target robot; kinematically or dynamically infeasible ones discarded | 98 % pass replay; ≈ 96 % of raw captures valid overall |
| Robot reach (kinematic filter) | UMI (chi2024, HD6 and Sec. VIII) | With known robot base and kinematics, recorded end-effector poses are filtered for kinematic and dynamic feasibility before training | None |
| Keep and mask instead of drop | RAPID (yin2026, III-C) | Partial episodes kept with a per-frame mask ([[2026-11-19 UMI-style recording and training pipelines]]) | – |

No paper reports its rejected share per reason, and none publishes pose-jump thresholds.

## 2. Our setup vs prior work

| Check | Applies here? | Note |
| --- | --- | --- |
| Tracking confidence | No direct equivalent | The PICO / XRoboToolkit controller stream has no tracked flag; out-of-view poses are IMU-inferred, so drift or snap-back looks like normal data ([[2026-11-17 Controller pose to robot frame]]) |
| Timestamp gaps | Yes | Pose log has local receive time and XRoboToolkit `timeStampNs` per sample |
| Pose jumps | Yes | Must separate real fast hand motion from glitches (see 3) |
| Robot reach | Yes | RL1 (replay in MuJoCo on the Mac) is a simulation replay as in HiFi-UMI |
| Missing camera frames | From RL5 (D435i + recorder) on | No camera recorded yet |
| Gripper state | From RL6 (Dex1-1 swap) on | Width from the Dex1-1 motor angle, D09 (calibration) |

## 3. Results from tests already done

| Date | Test | Result relevant to rejection | Source |
| --- | --- | --- | --- |
| 2026-11-18 | Trigger test (XRoboToolkit stream) | One 0.66 s gap, cause unknown | [[2026-11-23 RL1 replay in MuJoCo first run]] |
| 2026-11-23 | Desk take-01, 43.2 s, 3377 samples (78 Hz mean) | One 0.78 s gap at 4.4 s, cause unknown | same |
| 2026-11-23 | RL1 replay of take-01 | Largest tracking error 66 mm at 18.9 s from a real fast move (≈ 9 cm in 0.1 s, ≈ 0.9 m/s) and a ≈ 20° wrist twist in 0.1 s; raw samples smooth, not a tracking jump. A jump threshold below these values would reject real motion | same |
| 2026-11-23 | RL1 replay of take-01 | IK error RMS 0.8 mm, max 12.9 mm; 0 solver failures in 5525 solves; joint speed above 0.3 rad/s for 11.3 s of 44.2 s | same |
| 2026-11-23 | RL3 box check | take-01 refused (242 mm horizontal, allowed 150 mm) | [[2026-11-23 RL3 real UR10e replay procedure]] |
| 2026-11-23 | RL3 replay of take-02 (28.2 s) on the real UR10e | IK solver gave up 2 of 3656 ticks (held pose one tick); no protective stop | same |
| 2026-11-25 | RL4 simulation of handheld take-01 (`pose-log-20261125-150606`, 30.4 s) | Floor check passed (lowest fingertip 303 mm above the table); 201 mm horizontal, refused by the 150 mm RL3 radius until widened to 0.22 m; IK max 3.3 mm, 0 give-ups in 3921 solves | [[2026-11-24 RL4 Robotiq pick-and-place procedure]] |
| 2026-11-25 | RL4 on the real UR10e, same take | Simulation predicted the robot: tracking RMS 5.7 mm in sim vs 6.5 mm real (max 18.6 vs 21.8 mm); gripper commands within 40 ms. Supports using the simulation replay as the robot-free feasibility check (as HiFi-UMI, ai2026) | same, section 13 |
| 2026-11-25 | Trigger in take-01 | A resting trigger dipped 0.033 below the 0.9 threshold and toggled the gripper; fixed by hysteresis (open below 0.8), not a rejection reason | same, section 13 |

## 4. Tests to run later (none scheduled)

- Scan of all existing pose logs for gaps and speed distribution (no hardware).
- One 30–60 min recording to count gaps over a session (G6 (link reliability over a session), [[2026-11-10 Research gaps from the PICO sim test]]).
- Deliberate occlusion (controller behind body or hand) to see whether drift or snap-back is visible in the pose stream.
- From RL5 (D435i + recorder): missing camera frames; from RL6 (Dex1-1 swap): gripper-width consistency.

## Open

- Cause of the 0.66 s and 0.78 s gaps: unknown.
- Thresholds: none chosen; decision deferred until after the Replay Ladder (Martyn, 2026-11-24), then recorded with the record-decision skill.
