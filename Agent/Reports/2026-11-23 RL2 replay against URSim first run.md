---
date: 2026-11-23
tags: [report, replay-ladder, simulation]
---
# RL2 (replay against URSim on fractal-pc): first run

Plan and installs: [[2026-11-23 RL2 URSim on fractal-pc install plan]] (P1–P4 approved and done by Martyn's go, 2026-11-23). RL1 (replay in MuJoCo on the Mac): [[2026-11-23 RL1 replay in MuJoCo first run]]. Code: dex1-1-handheld PR #20 (`software/sim/ur10e_replay_rtde.py`).

## In short

- **RL2 passed.** Martyn's desk take replays on URSim 5.11.11 over RTDE without a protective stop; URSim's tool path matches the MuJoCo replay tick by tick (RMS 1.8 mm, max 9.5 mm) and ends exactly on target.
- URSim runs headless on fractal-pc with ports bound to localhost; pendant screen via an SSH tunnel to the Mac browser. Power-on and brake release work over the dashboard port; **Remote Control mode must be switched on in the pendant** (without it `RTDEControlInterface` times out: "Failed to start RTDE data synchronization").
- The team IK takes about 20 ms per solve on fractal-pc, longer than one 8 ms tick, so the joint targets are computed first (by the RL1 MuJoCo replay) and then streamed at 125 Hz.

## Results (2026-11-23, URSim 5.11.11, ur_rtde 1.6.6, 0.3 rad/s limit)

| | Synthetic pick-and-place (15.1 s) | Desk take-01 (43.2 s) |
| --- | --- | --- |
| MuJoCo tracking RMS / max | 6.5 / 12.0 mm | 22.0 / 83.6 mm |
| URSim tracking RMS / max | 7.8 / 13.7 mm | 22.7 / 89.2 mm |
| URSim end error | 0.0 mm | 0.0 mm |
| URSim vs MuJoCo TCP, per tick | – | RMS 1.8, median 1.1, max 9.5 mm |
| Highest actual joint speed | 0.20 rad/s | 0.32 rad/s |
| Loop | 2012 ticks in 16.1 s, longest 8.2 ms | 5527 ticks in 44.3 s, longest 9.5 ms |
| Safety status after | NORMAL | NORMAL |

Tracking = target TCP (from the recording) vs actual TCP; the large values on take-01 are the 0.3 rad/s limit lagging behind fast hand moves (as in RL1).

## Problems found and fixed

| Problem | Cause | Fix |
| --- | --- | --- |
| RTDE control connection timed out | Remote Control off (dashboard `isInRemoteControl` False) | Martyn enabled it in the pendant |
| First run took 41.6 s for 16.1 s of path | IK (about 20 ms) inside the 8 ms loop | Joint targets computed before streaming |
| IK precomputed with warm start from the previous solution: 1493 of 2010 solves failed | Team IK returns NaN at zero error (CasADi log3); seen once in 2010 in the closed loop | Targets taken from the closed-loop MuJoCo replay instead |
| Loop ran 4× too fast | `RTDEControlInterface` default 500 Hz sets the `waitPeriod` cycle | Constructed with 125 Hz |

## Open

- URSim runs with `--rm`: after a container restart, power-on, brake release and Remote Control must be redone (Remote Control needs the pendant screen). Not checked whether the setting survives.
- URSim keeps running on fractal-pc (shared PC, ports on localhost only); stop with `docker stop ursim` when not in use.
- RL3 (replay on the real UR10e): same script with `--robot 10.36.11.3` from fractal-pc or the Jetson (no `ur_rtde` arm64 wheel); slow takes; reduced speed, small allowed area, person at the emergency stop; Remote Control on the real pendant.
- Team IK speed on fractal-pc (about 20 ms per solve) vs the team's live 125 Hz teleop: how the team loop handles it is not checked.
