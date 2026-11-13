---
date: 2026-11-13
tags: [report, dex1-1, compute, station]
---
# Handheld computer and gripper connection

Documentation of where the handheld's computer and gripper connection stand on 2026-11-13, before the talk with Yanmei on 2026-11-16 ([[2026-11-16 Yanmei]], block A). Builds on [[2026-11-12 Lab computers and gripper connections]], [[2026-11-12 Raspberry Pi vs lab PC for the Dex1-1]] and [[2026-11-13 dex1_1_service code and Pi 5 fit]]. Nothing here is tested on hardware.

![[2026-11-13 handheld connection paths.png]]

Figure: (a) current VR teleop at the UR10e station (working picture, to be confirmed), (b) Path 1, (c) Path 2. Editable source (draw.io): `Agent/Reports/figures/2026-11-13 handheld connection paths.drawio`; PDF export: `Agent/Reports/figures/2026-11-13 handheld connection paths.pdf`.

## State of the question

Open: which computer drives the real Dex1-1 and its camera during handheld collection at the UR10e station. Two paths proposed by Martyn (2026-11-13); choice after the Monday talk.

## Fixed constraint (both paths)

- `dex1_1_service` exchanges one command and one answer with the motor every 2 ms (500 Hz, `RecurrentThread(2000, …)`) and brakes the motor when DDS commands stop (`isTimeout()` → BRAKE). Source: [dex1_1_service @ ced3216](https://github.com/unitreerobotics/dex1_1_service/tree/ced3216290bb22dae356213993786beeaa7143ab), `main.cpp`.
- So it must run on the computer the RS-485-to-USB adapter is plugged into; only slow commands (30–90 Hz, design value, unmeasured) and state cross the network (DDS topics `rt/dex1/<side>/cmd|state`). The Robotiq can be relayed with ser2net because it is plain Modbus at 115.2 kbit/s.
- The handheld is used at the station; fractal-pc and the teleop Jetson stand far from it. The adapter computer therefore stands at the station.
- `dex1_1_service` has no camera code. Camera: Unitree [teleimager](https://github.com/unitreerobotics/teleimager) (UVC/V4L2/RealSense → ZMQ/WebRTC) or the supervisor's existing ZMQ JPEG script. Dex1-1 camera as plain UVC: unverified.

## Path 1: Raspberry Pi 5 at the station + fractal-pc

- Pi 5 (Ubuntu 24.04 LTS): `dex1_1_service` + camera stream.
- fractal-pc: XRoboToolkit service + PICO (works since 2026-11-09), trigger → gripper command, recording.
- Pros: same split as the supervisor's Pi + Jetson setup; leaves her Jetson untouched; cheap.
- Cons: Pi 5 not documented by Unitree, build and 500 Hz loop untested; two computers; mobile version harder.
- Existing station Pi (2 GB, Ubuntu 25.10, runs Robotiq relay + D435i stream, load unmeasured) is a poor fit; manager agreed a second Pi 5 may be bought, order after Monday ([[Waiting]]).

## Path 2: one Jetson Orin

- Jetson Orin runs `dex1_1_service`, camera, XRoboToolkit and recording.
- Pros: Unitree's tested platform (Orin NX, README); if XRoboToolkit and the teleop stack already run on it, handheld and VR teleop share one stack (easier comparison); one computer could later go into a backpack (ActiveUMI, `zeng2025`; mobile collection is out of scope, dropped 2026-11-12).
- Cons: must stand at the station; shared with the supervisor's teleop; which unit and whether it may be used is unknown.
- Fallback for Path 1 if the Pi 5 test fails.

## OS requirements of `dex1_1_service`

From [[2026-11-13 dex1_1_service code and Pi 5 fit]]:
- 64-bit Linux, aarch64 or x86-64 (prebuilt closed motor library for both).
- GLIBC ≥ 2.17, GLIBCXX ≥ 3.4.26 (gcc 9 runtime); `unitree_sdk2` prebuilt on Ubuntu 20.04, gcc 9.4.
- "Ubuntu 22.04 or 24.04" is not stated by Unitree; its tested platform is Jetson Orin NX, offline packages for 22.04.
- Pi 5: Ubuntu 22.04 not available, 24.04 LTS supported (Canonical); not documented by Unitree → build test first.
- Not usable: Jetson Nano (JetPack 4, Ubuntu 18.04, GLIBCXX 3.4.25), incl. the borrowed reComputer J1020 v2.

## Open (Monday, [[2026-11-16 Yanmei]] A1–A3)

1. Confirm the current setup (panel a), incl. whether XRoboToolkit + teleop stack run on her Jetson and how it commands the UR10e.
2. Which Jetson, where it stands, whether it may be shared or moved to the station.
3. Existing station Pi or a second Pi 5.
4. Station supply rating vs Dex1-1 24–64 V, 3.5 A peak.
5. Her preference between the paths.

## Next steps after the choice

1. Build `unitree_sdk2` + `dex1_1_service` on the chosen computer and start it without the gripper (expected: "No ttyUSB serial ports found.").
2. Stream a USB camera to the recording computer (teleimager).
3. With the gripper: 500 Hz loop, commands over the network, camera time stamps (chrony).

## Log

- 2026-11-12: station inspected (Pi 5 relay for Robotiq + D435i); proposal Pi at station + fractal-pc.
- 2026-11-13: `dex1_1_service` code read; Pi 5 OS support checked; borrowed Jetson identified as a Nano (not usable); Martyn defined Path 1 / Path 2 with Jetson as fallback; agenda block A rewritten; diagram made.
