---
date: 2026-11-13
audience: Company Supervisor
type: Decision Brief
status: draft
---
# Handheld data collection: computer and gripper connection

> Nothing in this brief has been tested on hardware. The current setup in panel (a) is a working picture to be confirmed in the meeting on 16 November 2026. Gripper values come from the manufacturer; software facts come from Unitree's public source code.

## Decision needed

Which computer drives the real Dex1-1 gripper and its camera during handheld data collection at the UR10e station: a Raspberry Pi 5 at the station together with the lab PC (Path 1), or one Jetson Orin that runs everything (Path 2).

**Proposal (for discussion):** Path 1 first, because it leaves the existing VR-teleoperation setup untouched; Path 2 as the fallback if the Pi 5 build test fails, or as the first choice if a Jetson Orin can be placed at the station.

![Figure 1](figures/2026-11-13%20handheld%20connection%20paths.png)

*Figure 1. (a) Current VR teleoperation at the UR10e station as understood on 13 November 2026, to be confirmed. (b) Path 1: Raspberry Pi 5 at the station runs the gripper service and the camera stream; the lab PC runs XRoboToolkit and recording. (c) Path 2: one Jetson Orin at the station runs everything. Diagram drawn for this brief; nothing built.*

## Why the gripper program must run at the station

The Robotiq gripper uses short request–answer messages at 115.2 kbit/s ([Robotiq 2F-85 manual](https://assets.robotiq.com/website-assets/support_documents/document/2F-85_2F-140_Instruction_Manual_e-Series_PDF_20190206.pdf)), so the station Pi can forward its bytes to a distant computer. The Dex1-1 cannot be forwarded this way. Unitree's gripper service exchanges one command and one answer with the motor every 2 ms (500 Hz) and switches the motor to brake mode when commands stop arriving ([Unitree dex1_1_service](https://github.com/unitreerobotics/dex1_1_service), source code). The service must therefore run on the computer the RS-485-to-USB adapter is plugged into. Other programs talk to it over the network with slow position commands (30–90 per second) and read back the gripper state. Both paths follow this rule. The handheld is used at the station, and the gripper and camera cables only reach a few metres.

## The two paths

| | Path 1: Pi 5 + lab PC | Path 2: one Jetson Orin |
| --- | --- | --- |
| Station computer | Raspberry Pi 5, Ubuntu 24.04 LTS: gripper service + camera stream | Jetson Orin: gripper service, camera, XRoboToolkit, recording |
| XRoboToolkit host ([Zhao et al., 2025](https://arxiv.org/abs/2508.00097)) | Lab PC (PICO trigger to gripper already demonstrated on it in simulation, 9 November 2026) | The Jetson |
| Tested by Unitree | No (Pi 5 not documented) | Yes, on Jetson Orin NX ([dex1_1_service README](https://github.com/unitreerobotics/dex1_1_service)) |
| Effect on the existing setup | None; same split as today's Pi + Jetson | Jetson must stand at the station; shared with VR teleoperation |
| Comparison handheld vs VR teleoperation | Two stacks, same recording format | One stack for both conditions |
| Later mobile handheld | Harder (two computers) | Simpler: one computer could be carried with a battery, as in ActiveUMI ([Zeng et al., 2025](https://arxiv.org/abs/2510.01607)); outside the current scope |
| Cost | One additional Pi 5 | None if an Orin is available |

**Camera.** The gripper service contains no camera code. Unitree's separate image server streams USB cameras as JPEG over the network ([Unitree teleimager](https://github.com/unitreerobotics/teleimager)); whether the Dex1-1 camera works with it is unverified until delivery.

## Operating-system requirements of the gripper service

From the source code and its prebuilt libraries ([dex1_1_service](https://github.com/unitreerobotics/dex1_1_service), [unitree_sdk2](https://github.com/unitreerobotics/unitree_sdk2)):

- 64-bit Linux on ARM (aarch64) or x86-64; Unitree ships the closed motor library for both.
- The prebuilt libraries need C library version 2.17 or newer and the C++ runtime of gcc 9 or newer. The SDK was prebuilt on Ubuntu 20.04.

- Raspberry Pi 5: Ubuntu 22.04 is not available for it; Ubuntu 24.04 LTS is supported ([Ubuntu Raspberry Pi support](https://ubuntu.com/hardware/docs/boards/how-to/ubuntu_supported/raspberry-pi/)). Its library versions meet the requirement, but the build and the 500 Hz loop on a Pi 5 are **untested**.
- Not usable: the original Jetson Nano, which stays on Ubuntu 18.04 with an older C++ runtime.

## Common to both paths

- Gripper power: 24–64 V DC, 3.5 A peak ([Unitree Dex1-1](https://unitree.com/Dex1-1)). The rating of the existing station supply is not yet known; the Robotiq needs 24 V ([Robotiq 2F-85 manual](https://assets.robotiq.com/website-assets/support_documents/document/2F-85_2F-140_Instruction_Manual_e-Series_PDF_20190206.pdf)).
- Picture time stamps synchronised with the recording computer's clock, as in the current setup.

## Questions for the meeting

1. Is panel (a) correct? Does XRoboToolkit and the teleoperation stack run on the Jetson, and how does it command the UR10e?
2. Which Jetson is it, where does it stand, and may it be shared or moved to the station?
3. Path 1: may the gripper service run on the existing station Pi (2 GB memory, Ubuntu 25.10, already serving the Robotiq and the RealSense), or should a second Pi 5 be set up next to it? A second Pi 5 can be purchased.
4. What are the voltage and current rating of the station supply?
5. Preference between Path 1 and Path 2.

## Next step after the meeting

Build the gripper service on the chosen computer and start it without the gripper (expected result: "no serial port found"); then stream a USB camera to the recording computer. With the gripper: check the 500 Hz loop and commands over the network.

## References

**Papers**

- Zhao et al., 2025. XRoboToolkit: A Cross-Platform Framework for Robot Teleoperation. https://arxiv.org/abs/2508.00097
- Zeng et al., 2025. ActiveUMI: Robotic Manipulation with Active Perception from Robot-Free Human Demonstrations. https://arxiv.org/abs/2510.01607

**Documentation / code**

- Unitree dex1_1_service: https://github.com/unitreerobotics/dex1_1_service
- Unitree unitree_sdk2: https://github.com/unitreerobotics/unitree_sdk2

- Unitree teleimager: https://github.com/unitreerobotics/teleimager
- Unitree Dex1-1 product page: https://unitree.com/Dex1-1

- Ubuntu on Raspberry Pi, supported boards: https://ubuntu.com/hardware/docs/boards/how-to/ubuntu_supported/raspberry-pi/
- Robotiq 2F-85/2F-140 instruction manual (e-Series): https://assets.robotiq.com/website-assets/support_documents/document/2F-85_2F-140_Instruction_Manual_e-Series_PDF_20190206.pdf
