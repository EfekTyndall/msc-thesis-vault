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
