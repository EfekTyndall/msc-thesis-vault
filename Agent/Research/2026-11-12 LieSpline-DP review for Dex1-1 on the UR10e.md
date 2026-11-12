---
date: 2026-11-12
tags: [research, dex1-1, ur10e, paper-review]
---
# LieSpline-DP (arXiv 2609.15162) review: Dex1-1 on the UR10e

Question: what does arXiv 2609.15162 report about running a Unitree Dex1-1 on a UR arm (mounting, power, RS-485 link, software, control rates, issues) that transfers to the UR10e?

**Paper:** Xie, Liu, Nie, Liu, Fu, Zhang, "LieSpline-DP: Lie-Group B-Spline Diffusion Policy for Smooth Robot Manipulation", arXiv 2609.15162v2, cs.RO, 2026-09-16, Shanghai Jiao Tong University ([abs](https://arxiv.org/abs/2609.15162), [project page](https://xieerxuan.github.io/LieSpline-DP/), [code, MIT](https://github.com/xieerxuan/LieSpline-DP)). **Not in bib.**

**Method and limits.** The PDF was read as extracted text (no page breaks), so references are by section, figure and table, not page. The project page and the public code repository (commit `6ec4677`, 2026-09-18) were read as well. No wider search was run.

## Short answer

The paper is a policy paper (smooth SE(3) spline actions for diffusion policy), not a hardware paper. The whole hardware description is one sentence: "We evaluate on a UR5e arm equipped with a Unitree Dex1 gripper" (Sec. IV.B). It says "Dex1", not "Dex1-1". Mounting, power, RS-485 link and gripper driver are **not reported** in the paper, the project page or the code. The README states that the hardware-side program is not included ("Developers supply sensor acquisition and hardware execution", `README.md`; `docs/real_robot_deployment.md`, "Connecting your hardware"). It is therefore no help for the flange adapter, power or RS-485 wiring on the UR10e.

## What each item looks like

| Item | What the source says | Source |
| --- | --- | --- |
| Arm / gripper | UR5e + "Unitree Dex1" | Paper Sec. IV.B; project page, Experiments |
| Flange adapter, cable routing | Not reported | Paper, project page, code |
| Gripper power (24–64 V) | Not reported | Same |
| RS-485 / `dex1_1_service` / DDS | Not reported; no Unitree SDK in the dependencies (`robot` extra is only `pyzmq==26.2.0`) | `pyproject.toml` l. 54 |
| Arm driver (RTDE, URScript, ROS) | Not reported; no `ur_rtde` or ROS in the code | `pyproject.toml`, `conda_environment.yaml` |
| Software split | Policy process talks to a separate "Robot Service" over ZeroMQ IPC sockets (`ipc:///tmp/ur5e_robot_service/...`); the service (not published) owns the arm and gripper | `liespline_dp/config/real_world/robot_service_backend.yaml`; `liespline_dp/real_world/robot_service_hardware_backend.py` |
| Service safety checks | Heartbeat 0.2 s, status stale after 1.0 s, max observation age 0.25 s, max camera/state skew 0.08 s; service reports `gripper_target_min/max`, `max_gripper_step`, `max_gripper_speed_s` limits | Same files (l. 27–41 of the backend) |
| Policy / action rate | Waypoints at 15 Hz (sim 20 Hz); 8 actions per chunk, 2-step latency allowance | Paper Sec. IV.A, IV.B; `config/task/real_robot/_bspline_se3.yaml` (`control_frequency_hz: 15.0`) |
| Arm servo rate, gripper rate | Not reported | Paper, code |
| Cameras | Two RGB streams, `wrist_rgb` and `global_rgb`, 240 × 320; model and mount not reported | `_bspline_se3.yaml` `shape_meta` |
| Gripper command | Scalar `gripper_q`, bounds `[0.0, 5.0]` "in consistent gripper units"; training converts targets to **binary open/close intent** with hysteresis 0.25, so intermediate widths are not kept | `_bspline_se3.yaml` (`gripper_bounds`, `gripper_label_construction: binary_intent_causal_zoh`); `docs/real_robot_training.md` |
| Action format | Absolute TCP position + 6D rotation + gripper target (10-D) | `docs/real_robot_deployment.md` |
| Demonstration method | Not reported (only counts: 50 pouring, 50 bucket hooking, 100 cube stacking incl. recovery demos) | Paper Sec. IV.B |
| Reported issues | None about the gripper or arm hardware. Reported failures are policy behaviour: baseline jitter, cube toppling, "hesitated before releasing the final cube" (both methods) | Paper Sec. IV.B |

## Inferences (unverified)

- The `[0.0, 5.0]` gripper bounds fit the Dex1-1 motor angle in radians: `xr_teleoperate` uses 5.4 rad ↔ 90 mm and the service calibrates over 5.62 rad ([[2026-11-03 Dex1-1 technical note]]). If so, they commanded the raw motor `q` from `dex1_1_service`, not a width in metres. The paper does not say this.
- The 0–5 rad range stops short of 5.4–5.62 rad, possibly a soft limit. Unverified.

## What transfers to the thesis

1. **Precedent that a Dex1(-1) runs on a UR e-series arm** with a learned policy (UR5e, Sec. IV.B). The UR5e and UR10e share the e-series controller, but tool flange and payload differ, so the mechanical side does not transfer; that is our own adapter work.
2. **Service split**: a separate hardware process (arm + gripper) and a policy process joined by a socket interface with heartbeat and stale-data checks. This matches the lab plan of the PC running `dex1_1_service` and the robot link (see [[2026-11-12 Raspberry Pi vs lab PC for the Dex1-1]]), and is a citable pattern if the paper enters the bib.
3. **Gripper as binary intent**: their training discards intermediate gripper targets. Relevant to G2 (trigger mapping, [[2026-11-10 Research gaps from the PICO sim test]]): one more precedent for binary gripper labels in policy training.
4. **Data format**: per-step timestamp, measured and target TCP pose, measured and target gripper, two RGB images (`docs/real_robot_training.md`). Close to our export convention (absolute pose + 6D rotation + gripper); they use 15 Hz, we use 30 Hz.

## Open

- Mounting, power, RS-485 and driver details for a UR + Dex1-1 setup are still unsourced. Options: (a) ask the authors (sends a message, needs Martyn's OK); (b) a deep-research pass for other UR + Dex1-1 setups (GitHub issues on `dex1_1_service`, other papers), asked before running; (c) the project-page videos may show the adapter and cables; not watched here (the video host was not reachable from this session).
- Whether "Dex1" in the paper is the Dex1-1 product: likely, unverified.
