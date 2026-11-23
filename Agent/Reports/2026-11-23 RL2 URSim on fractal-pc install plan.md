---
date: 2026-11-23
tags: [report, replay-ladder, install-plan]
---
# RL2 (replay against URSim on fractal-pc): check and install plan

Ask (Martyn, 2026-11-23): next step after RL1 (replay in MuJoCo on the Mac, passed: [[2026-11-23 RL1 replay in MuJoCo first run]]). Read-only check of fractal-pc approved by Martyn 2026-11-23; nothing installed yet. Background: [[2026-11-20 Replay Ladder RL1-RL2 setup]].

## Read-only check of fractal-pc (2026-11-23, `ssh fractal-pc`)

| Item | Finding |
| --- | --- |
| Machine | Ubuntu 22.04.5, x86_64, 24 cores, 31 GB RAM; `/` 105 GB free of 543 GB (80 % used) |
| Docker | 29.9.3, service active; account `martyn` is in the `docker` group (no sudo needed). Existing images from other work (mlflow, isaac_ros, foundationpose, …); no URSim image, no running containers |
| Conda | `~/anaconda3`, 14 envs; `xrobotoolkit` has the XRoboToolkit SDK. No env has `ur_rtde`, Pinocchio, CasADi or MuJoCo; system Python 3.10.12 has none either |
| ROS 2 | Humble in `/opt/ros` (not needed: RTDE needs no ROS) |
| `~/ur10e_ws` | One ROS 2 Python package `ur10e_control` (Oct 2024, not a git repo): `move_robot.py` publishing joint trajectories over `rclpy`, generated package description. Not related to the team's RTDE stack; not needed for RL2 |
| Code | `~/dex1-1-handheld` present; no `xr_teleoperate` clone |
| Network | `enp3s0` 10.36.1.102/16; `docker0` down |

## Plan (each step needs Martyn's go)

| Step | What | Size / source | Undo |
| --- | --- | --- | --- |
| P1 URSim image | `docker pull universalrobots/ursim_e-series:<version>`; `<version>` = the real UR10e's PolyScope version if known (pendant: Settings → About), else 5.26 (newest) | 0.97 GB compressed, amd64 only ([Docker Hub](https://hub.docker.com/r/universalrobots/ursim_e-series), tag API 2026-11-23) | `docker rmi universalrobots/ursim_e-series:<version>` |
| P2 Run URSim | `docker run --rm -d --name ursim -e ROBOT_MODEL=UR10 -p 127.0.0.1:29999-30004:29999-30004 -p 127.0.0.1:6080:6080 universalrobots/ursim_e-series:<version>`. Ports bound to localhost only, so nothing is exposed on the lab network (shared PC). Pendant screen from the Mac: `ssh -L 6080:localhost:6080 fractal-pc`, then `http://localhost:6080/vnc.html` | Ports from the [UR Client Library URSim guide](https://github.com/UniversalRobots/Universal_Robots_Client_Library/blob/master/doc/setup/ursim_docker.rst) (6080 VNC; robot interfaces 29999–30004; exact list to confirm when starting) | `docker stop ursim` (container removed by `--rm`) |
| P3 Conda env | `conda create -n ursim-replay python=3.10 -c conda-forge pinocchio casadi numpy`, then `pip install ur_rtde==1.6.6 mujoco` | ur_rtde 1.6.6, MIT, wheel `cp310 manylinux x86_64` ([PyPI](https://pypi.org/project/ur-rtde/)); Pinocchio and CasADi as on the Mac `humanoid` env; MuJoCo only because the replay script imports it | `conda env remove -n ursim-replay` |
| P4 Team code | Copy the Mac's `~/thesis/team/xr_teleoperate` (commit `7e6c754`) to `~/team/xr_teleoperate` on fractal-pc with `rsync` (read-only use, team machine) | ~ size of the clone | `rm -rf ~/team/xr_teleoperate` |
| P5 Replay backend (code, PR) | `ur10e_replay.py --robot <ip>`: same IK, mapping and 0.3 rad/s limit; sends each 125 Hz step with `servoJ` (as the team's `UR10e_ArmController`), reads `getActualQ`/`getActualTCPPose`, saves the robot's actual path to a CSV for viewing on the Mac | dex1-1-handheld PR | revert PR |

Unverified until P2: whether URSim needs Remote Control mode switched on in the pendant before `ur_rtde` may send `servoJ` (the real robot does: team `run_ur10e_teleop_real.sh`), and whether power-on and brake release can be done over the dashboard port (29999) instead of the pendant screen.

## Pass check for RL2

Martyn's desk take (`take-01.csv`) replays unchanged against URSim: no protective stop, joint speed ≤ 0.3 rad/s, and URSim's actual TCP path matches the MuJoCo replay of the same take (difference target: a few mm; MuJoCo model rounding ≤ 1.3 mm).

## Open

- Real UR10e PolyScope version (sets the URSim tag).
- Go for P1–P4 (installs on a shared PC); P5 is code via PR.
