---
date: 2026-11-12
tags: [report, dex1-1, compute, explainer]
---
# Lab computers and gripper connections

Plain-language explanation of how the lab pieces connect today (VR teleop with the Robotiq gripper) and how the handheld with the Dex1-1 would connect. Written for reading on 2026-11-13. Visual version: `Agent/Drafts/2026-11-12 Lab Wiring Explained.html` (same content, interactive). Sources and details: [[2026-11-12 Raspberry Pi vs lab PC for the Dex1-1]] and [[2026-11-12 Dex1-1 on the UR10e integration]].

## The short version

- Today a **Raspberry Pi sits at the robot station** only because gripper and camera cables cannot reach the far-away computers. It is a messenger: it forwards gripper messages and camera pictures over the wired network.
- The **Jetson Orin** (far away) is the brain of today's VR teleop: it runs XRoboToolkit, talks to the PICO headset, and controls the gripper through the Pi.
- **fractal-pc** (also far away) can do exactly the same job as the Jetson. For the handheld it is the better choice because it already runs XRoboToolkit with the PICO (tested 2026-11-09) and it does not touch the supervisor's setup.
- The **Dex1-1 cannot be forwarded like the Robotiq.** Its motor needs a message and an answer every 2 ms. So the Pi at the station must **run Unitree's gripper program itself**, and only the slow commands ("open to 40 mm") travel over the network.
- Proposed handheld layout: handheld gripper and camera → **Pi at the station** (runs the Dex1-1 program + camera stream) → wired network → **fractal-pc** (XRoboToolkit, PICO, recording).

## The pieces

| Piece | What it is | Where it stands | Today's job |
| --- | --- | --- | --- |
| UR10e | the robot arm | station | moves during VR teleop |
| Robotiq 2F-85 | today's gripper on the UR10e | on the robot's wrist | opens/closes on command |
| RealSense D435i | today's wrist camera (colour + depth) | on the robot's wrist | gives the policy its view |
| 24 V supply | power unit under the station | station | powers the Robotiq |
| Raspberry Pi 5 ("arena") | small Linux computer, 2 GB memory, Ubuntu 25.10 | station | messenger for gripper and camera |
| Network switch | the box all network cables plug into | station | connects the station to the rest of the lab network |
| Jetson Orin | NVIDIA computer (ARM processor, GPU) | far from the station | runs XRoboToolkit for the supervisor's teleop |
| fractal-pc | lab Linux PC, shared with other students | far from the station | Martyn's XRoboToolkit test (2026-11-09); later training |
| PICO 4 Ultra | VR headset + controllers | with the operator | sends hand pose and trigger over Wi-Fi |
| Dex1-1 (×2, ordered) | Unitree gripper, one motor, RS-485 link | not delivered yet | will be on the handheld and later on the robot |
| MacBook | Martyn's laptop | anywhere on the lab network | editing, CAD, simulation; reaches the PC with `ssh` |

Terms used below:
- **RS-485**: a two-wire cable standard for sending data to motors over a few metres. A small **adapter** turns it into USB for a computer.
- **Relay / forwarding**: the computer passes bytes on without understanding them. On the Pi this is done by a program called `ser2net` ("serial to network").
- **XRoboToolkit**: the open-source program that receives the PICO data on a computer. It has a service part (runs in the background) and a Python part (your own scripts read from it).
- **Clock sync (chrony)**: keeps two computers' clocks within about a millisecond of each other, so time stamps from both can be compared.

## How today's VR teleop works (supervisor's setup)

Checked on the Pi on 2026-11-12 (read-only) and confirmed with the intern.

1. The operator wears the PICO. The headset sends hand pose and trigger over **Wi-Fi** to the **Jetson**, where XRoboToolkit receives them.
2. The Jetson decides what the gripper should do and sends a **Modbus** message (Robotiq's simple message format) over the **wired network** to the Pi, on network port 4001.
3. The **Pi forwards** those bytes unchanged through the RS-485 adapter to the Robotiq, and the answer back the same way. The Pi does not decide anything.
4. The **camera** plugs into the Pi by USB. A Python script on the Pi takes 30 pictures per second (848 × 480), compresses each to JPEG, stamps it with the moment it was taken, and sends it to the Jetson. The Jetson turns it into a ROS 2 stream (ROS 2 = the common robot-software message system) for the teleop and π0.5 pipeline.
5. The Pi keeps its clock in step with another lab computer (10.36.1.108, probably the Jetson) to about 1 ms, so picture time stamps match the Jetson's time.
6. Power: the Robotiq gets 24 V from the supply under the station; only its data goes through the Pi.

Not yet known: how the Jetson moves the UR10e arm itself (most likely over the network with UR's RTDE interface; unverified).

## Why the Robotiq can be forwarded but the Dex1-1 cannot

Think of it as **letters versus a phone call**.

- **Robotiq: letters.** The computer writes "close", the gripper writes back "OK". This happens up to about 200 times a second at 115 kbit/s (Robotiq manual), and a few milliseconds of network travel do not matter. So the Pi can just carry the letters to a far computer.
- **Dex1-1: a phone call with no pauses.** Unitree's gripper program (`dex1_1_service`) talks to the motor **every 2 ms** (500 times a second, at 6 Mbit/s) and waits for an answer each time. If the Pi only forwarded these, each exchange would go to the far computer and back over the network, and any delay or hiccup would land inside that 2 ms window. The gripper also brakes when commands stop arriving. In addition, Unitree's program expects the adapter on the same computer.

Result: the **Unitree program must run on the computer the Dex1-1 adapter is plugged into**. That computer then offers a simple network interface: the far computer sends "go to this opening" 30–90 times a second and reads back the gripper position. This is how Unitree's own robots do it: the gripper program runs on a small computer inside the robot, and other programs talk to it over the network.

These timing statements come from reading Unitree's code; nothing has been measured in this lab.

## Proposed layout for the handheld

```
HANDHELD (operator at the station)        STATION                              FAR AWAY
 PICO controller ──── Wi-Fi ───────────────────────────────────────────────▶ fractal-pc
 Dex1-1 ── RS-485 ──▶ adapter ──USB──▶ Raspberry Pi                            • XRoboToolkit + PICO
                                       • runs Unitree gripper program ◀─net──▶ • sends gripper commands
 Dex1-1 camera ──────────USB─────────▶ • camera stream ─────────────net──────▶ • records everything
 24 V supply (under station) ─▶ Dex1-1                 └─ switch ─┘
```

Who does what:
- **Handheld**: PICO controller in the sleeve (gives hand pose and trigger), Dex1-1 (gripper), Dex1-1's own camera.
- **Pi at the station**: runs Unitree's gripper program and the camera stream. Fast gripper loop stays here.
- **fractal-pc**: XRoboToolkit with the PICO, turns trigger into gripper commands, records pose, gripper position and pictures with time stamps.
- **Network**: wired switch at the station. Only slow traffic crosses it.

Cables from the handheld only need to reach the station (a few metres), not the far computers.

## Why fractal-pc and not the Jetson

Both stand far from the station and both can run XRoboToolkit, so **technically there is no difference** for the handheld. Reasons for fractal-pc:
- XRoboToolkit with the PICO already works on it (2026-11-09).
- It is set up for Martyn's workflow: own account, repository copy, VS Code from the Mac.
- It leaves the supervisor's working teleop untouched.

The only argument for the Jetson: both data sources (handheld and VR teleop) would run through one pipeline. Not required, because both can be recorded in the same format.

## Ideas that were looked at and dropped

| Idea | Why dropped |
| --- | --- |
| Pi replaces fractal-pc completely | Pi 5 has no hardware video encoder; training needs the PC's GPU; XRoboToolkit not tested on Pi |
| Carried (mobile) computer, e.g. Jetson in a backpack | Possible on paper (ActiveUMI does it), but out of scope for the thesis (dropped by Martyn 2026-11-12) |
| Handheld plugged straight into fractal-pc | fractal-pc is too far from the station |
| Forwarding the Dex1-1 through `ser2net` like the Robotiq | 2 ms motor loop cannot cross the network (see above) |
| Dex1-1 through the UR10e's wrist connector | Wrist connector gives 12/24 V at 2 A and RS-485 up to 5 Mbit/s; the Dex1-1 needs 24–64 V, 3.5 A peak and 6 Mbit/s ([[2026-11-12 Dex1-1 on the UR10e integration]]) |

## Later: the Dex1-1 on the robot

When a Dex1-1 replaces the Robotiq on the UR10e, the same pattern applies: its adapter plugs into a computer at the station (the Pi), which runs the Unitree program; power comes from a separate supply, not from the robot's wrist. Only one Dex1-1 is driven at a time (handheld collection, VR teleop, or deployment), so one station computer can serve whichever is in use.

## Questions for the supervisor (Monday)

1. **Which Pi?** May the Unitree gripper program go on the existing station Pi, or should a second Pi be set up next to it? The existing one has 2 GB memory and Ubuntu 25.10, neither tested by Unitree, and it serves her Robotiq teleop.
2. **Power:** what are the voltage and current of the supply under the station? The Dex1-1 needs 24–64 V and up to 3.5 A peak; the Robotiq needs 24 V and at most 1.7 A (retailer listing).
3. **Computer:** is it fine to run the handheld on fractal-pc instead of her Jetson?
4. Optional: how does the Jetson move the UR10e arm (RTDE, ROS 2 driver)?

## What still needs testing (nothing tested yet)

1. Build Unitree's gripper program on a Pi 5 and start it. It should run and report "no motor found". Possible before the gripper arrives.
2. With the gripper: check the 500 Hz loop on the Pi, and commands from fractal-pc over the network.
3. Picture time stamps from the Pi against fractal-pc's clock (chrony), as the supervisor's setup already does.
