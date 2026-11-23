---
date: 2026-11-23
tags: [report, replay-ladder, procedure, safety]
---
# RL3 (replay on the real UR10e): procedure

Decided (Martyn, 2026-11-23): RL3 from fractal-pc first; move the replay to the station Jetson later (needs a `ur_rtde` source build, see [[2026-11-23 RL2 replay against URSim first run]]). Code: dex1-1-handheld PR #20 (`ur10e_replay_rtde.py`, with allowed-box check and `--dry-run`).

Checked 2026-11-23 (read-only): fractal-pc reaches the UR10e at 10.36.11.3 (ping 0.2 ms; dashboard 29999 and RTDE 30004 open). Robot software 5.11.7.

## Safety layers

| Layer | Setting |
| --- | --- |
| Small take | Moves of about 10 cm, slow (about 5 s per move), wrist mostly still |
| Allowed box | Script refuses to start if the target or simulated path leaves 0.15 m radius / 0.05 m down / 0.15 m up around the start TCP (`--box`) |
| Joint speed | 0.3 rad/s limit (team teleop value); start pose reached with `moveJ` at 0.3 rad/s |
| Dry runs | MuJoCo view and box check on the Mac, then the same file on URSim |
| People | Martyn at the pendant with the emergency stop in reach; clear table area around and below the start pose |
| Robot | Remote Control mode only while replaying; switch back to Local afterwards; check the team is not using the robot |

take-01 (2026-11-23) is refused by the box check (242 mm horizontal), so RL3 needs a new, smaller take.

## Pass check

Robot moves as in the MuJoCo and URSim replays (Martyn's visual check); logged actual TCP vs target as in RL2; no protective stop. Accuracy against the real world (mm) is not measured in RL3.

## First run (2026-11-23, ≈16:35 CET, Martyn at the pendant)

**RL3 passed (visual check, Martyn): the UR10e replayed the controller movement.** Take-02: desk recording on the station Jetson (`pose_log.py`), 28.2 s, slow moves of about 8–10 cm. Replay from fractal-pc (`ur10e_replay_rtde.py`, PR #20), robot started at `HOME_Q` (moved there by hand first), pendant in Remote Control.

| Check | Result |
| --- | --- |
| Box check | 83 mm horizontal, 32 mm down, 54 mm up (allowed 150 / 50 / 150) |
| MuJoCo prediction (RL1) | tracking RMS 3.1 mm, max 7.7 mm; joint speed max 0.14 rad/s; limit active 0.1 s |
| Robot tracking (target vs actual TCP) | RMS 3.3 mm, max 8.7 mm, 0.0 mm at the end |
| Robot vs MuJoCo TCP, per tick | RMS 1.0 mm, median 0.9 mm, max 2.2 mm |
| Commanded vs actual joints | median 0.0031 rad, max 0.0158 rad |
| Highest actual joint speed | 0.15 rad/s |
| Loop | 3657 ticks in 29.3 s, longest tick 8.4 ms |
| Protective stop | none |
| IK solver gave up | 2 of 3656 ticks (known rare NaN case; held the pose for one tick) |

"Actual TCP" is computed from the robot's joint encoders with the team's kinematic model: it shows that the robot followed the commands, not where the tool was in the room (no external measurement). Logs: fractal-pc `~/rl2-test/take-02-robot.csv`, Mac `~/Desktop/take-02*.csv` (not in Git).

Lessons: the replay printed nothing during the 80 s target computation and Ctrl-C was caught by the team IK, so Martyn's first attempt looked dead; fixed (progress line, Ctrl-C flag). HOME_Q saved by Martyn as a pendant program (own program, team installation unchanged).
