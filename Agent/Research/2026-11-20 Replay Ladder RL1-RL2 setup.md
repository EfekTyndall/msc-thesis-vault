---
date: 2026-11-20
tags: [research, replay-ladder, simulation]
---
# Replay Ladder RL1–RL2: simulation setup

Question (Martyn, 2026-11-20): for the Replay Ladder ([[2026-11-20 Interim hardware while the Dex1-1 is delayed]]), where do RL1 (MuJoCo) and RL2 (URSim) run, and is ROS 2 or Linux required?

Method: official sources read on 2026-11-20: MuJoCo Menagerie README and `universal_robots_ur10e/README.md` (GitHub), Universal Robots Client Library doc "Setup URSim with Docker" (GitHub source of the docs.ros.org page), Docker Hub `universalrobots/ursim_e-series` (description and tag API), REP 2000 (ROS 2 platforms, GitHub source), PyPI `ur-rtde` 1.6.5 metadata, UMI `README.md` and `conda_environment.yaml` (GitHub). Nothing installed or run.

## In short

- **ROS 2 is not needed.** The UR10e can be commanded over RTDE, Universal Robots' network interface, with the `ur_rtde` Python library (MIT, SDU Robotics). UMI drives its UR5/UR5e this way (`ur-rtde==1.5.6` in `conda_environment.yaml`; README: "switch control mode to remote").
- **RL1 on the Mac works:** MuJoCo Menagerie has a UR10e model (BSD-3-Clause, "Requires MuJoCo 2.3.3 or later"); the Mac already has MuJoCo 3.12 (`mujoco-lab` env).
- **RL2 belongs on fractal-pc:** the official URSim Docker image is published for **linux/amd64 only** (tags 5.22–5.26, all `amd64`). fractal-pc is Ubuntu 22.04 x86_64; the Jetson (arm64) and an Apple-silicon Mac do not match (Mac chip unverified).
- **`ur_rtde` wheels exist only for Linux x86 and Windows** (PyPI 1.6.5: `manylinux …x86_64`, `…i686`, `win_amd64`; no macOS, no aarch64). On fractal-pc it installs with pip; on the Mac or the Jetson it would need a source build.

## Findings

| Item | Finding | Source |
| --- | --- | --- |
| UR10e in MuJoCo | `universal_robots_ur10e`, 6 DoF, BSD-3-Clause; derived from the ROS-Industrial URDF; position-controlled actuators whose damping/armature "have not been carefully tuned"; home keyframe; `scene.xml` | [Menagerie README](https://github.com/google-deepmind/mujoco_menagerie), [UR10e README](https://github.com/google-deepmind/mujoco_menagerie/tree/main/universal_robots_ur10e) |
| URSim | "acts almost identically to a real robot connected over the network"; Docker image `universalrobots/ursim_e-series`; model via `-e ROBOT_MODEL=UR10` (in the e-Series image UR10 means the UR10e; inferred); PolyScope in a browser via VNC port 6080; robot interface ports published by `start_ursim.sh` | [UR Client Library: Setup URSim with Docker](https://github.com/UniversalRobots/Universal_Robots_Client_Library/blob/master/doc/setup/ursim_docker.rst); [Docker Hub](https://hub.docker.com/r/universalrobots/ursim_e-series) |
| URSim architectures | Every listed tag (5.22–5.26, updated 2026-10-28) is `linux/amd64` | Docker Hub tag API, read 2026-11-20 |
| ROS 2 on macOS | Humble and Jazzy: macOS is **Tier 3** ("community reports indicate that the release is functional"; no tests by the developers), listed for amd64 only | [REP 2000](https://github.com/ros-infrastructure/rep/blob/master/rep-2000.rst), Humble and Jazzy tables |
| `ur_rtde` | Version 1.6.5, MIT licence; wheels for CPython 3.7–3.12 on Linux x86_64/i686 and Windows | [PyPI](https://pypi.org/project/ur-rtde/) |
| Precedent | UMI uses `ur-rtde` for its UR5/UR5e, tested on Ubuntu 22.04 | [UMI repo](https://github.com/real-stanford/universal_manipulation_interface) |

## What this means for the Replay Ladder (proposal, nothing installed)

| Phase | Machine | Software | New dependency (needs Martyn's go) |
| --- | --- | --- | --- |
| RL1 | MacBook | MuJoCo 3.12 (present) + Menagerie UR10e model + replay script; the recorded controller path is turned into joint targets by an inverse-kinematics step (tool not chosen; no source checked yet) | Menagerie model files (download only); an IK step |
| RL2 | fractal-pc | Docker + `universalrobots/ursim_e-series` with `ROBOT_MODEL=UR10`; replay script with `ur_rtde` | Docker on fractal-pc (installed or not: unchecked); the URSim image; `ur_rtde` in a conda env |
| RL3 | fractal-pc or the station Jetson | Same script, address of the real UR10e; robot in remote control mode | – on fractal-pc; source build of `ur_rtde` on the Jetson (arm64) |

RL1 also checks the fixed half-turn between operator and robot (operator faces the robot from the table side) and the motion scale before any robot software is involved.

## Open

- How the team's VR-teleop stack commands the UR10e (RTDE, ROS 2 driver or other); Martyn shares the stack when he has access. If it already uses a method that works, copy it instead of adding `ur_rtde`.
- fractal-pc: Docker installed and usable by Martyn's account? `ur10e_ws` contents? (read-only check, needs Martyn's OK; shared PC)
- MacBook chip (Apple silicon or Intel): only matters if URSim or `ur_rtde` is ever wanted on the Mac.
- IK for RL1: pick a method with a source before writing the script.
- The station Jetson is the planned recording computer (Path 2), but `ur_rtde` has no arm64 wheel: decide later whether RL3+ commands the robot from the Jetson (source build) or fractal-pc.
