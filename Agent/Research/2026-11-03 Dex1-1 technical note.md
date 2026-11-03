---
date: 2026-11-03
tags: [research, hardware, dex1-1]
---
# Dex1-1 technical note: RS-485 control and use as a handheld

Question: can the real Dex1-1 serve as the handheld body ([[D01 Handheld gripper body]], swap option), driven by the PICO controller trigger from the recording PC? Answers the open questions in `Daily/2026-11-03`.

Method: Unitree product page; reseller listings for box contents; source code of Unitree's `dex1_1_service` ([commit ced3216](https://github.com/unitreerobotics/dex1_1_service/tree/ced3216290bb22dae356213993786beeaa7143ab), 2026-09-16), `unitree_sdk2` headers and the Dex1-1 controller in `xr_teleoperate` ([commit 817fb00](https://github.com/unitreerobotics/xr_teleoperate/tree/817fb00c63cde15e5f24a0f8fa08e1e33ed89d3b), 2026-09-07). Unitree's support page ([Dex1-1 gripper](https://support.unitree.com/home/zh/dex1-1_gripper), section 4.2: wiring) renders only in a browser and was not read. Nothing was tested on hardware.

## Summary

1. **Feasible on paper.** The gripper connects through Unitree's RS-485-to-USB board (included in every version, per resellers) to a Linux PC. Unitree's open-source service turns the serial link into two DDS topics, one for commands and one for state, which a Python or C++ program on the same PC can use. Unitree answered "theoretically feasible" when asked about use with your own PC and arm ([issue #5](https://github.com/unitreerobotics/dex1_1_service/issues/5)).
2. **No separate computer is needed.** The service ships x86-64 and arm64 libraries, so the recording PC can run it. This is inferred from the files in `lib/`; it has not been run on x86-64.
3. **Power: 24–64 V DC (24 or 48 V recommended), 3.5 A peak, 0.03 A static.** This means a bench supply and a power cable to the handheld. No supply is listed in the box.
4. **No force/torque sensor.** The state gives only the motor torque estimate (`tau_est`), alongside position and velocity. Grip force can only be estimated from it after calibration against a force gauge.
5. **The trigger can drive the gripper proportionally or as a binary switch.** Both are software choices. Unitree's own teleop maps the trigger value linearly to position, at a 0.18 rad limit per control cycle.

## Hardware (Unitree product page)

| Property | Value | Source |
| --- | --- | --- |
| Supply voltage | 24–64 V (24/48 V DC recommended) | [Unitree](https://unitree.com/Dex1-1) |
| Current | 0.03 A static, 3.5 A peak | [Unitree](https://unitree.com/Dex1-1) |
| Interface | "High-speed 485", 6 Mbit/s, 1000 Hz control frequency | [Unitree](https://unitree.com/Dex1-1) |
| Encoder | 15-bit absolute | [Unitree](https://unitree.com/Dex1-1) |
| Gripping force | 5–120 N, adjustable | [Unitree](https://unitree.com/Dex1-1) |
| Stroke / resolution | 90 mm / 0.1 mm; 0.24 s full-stroke closing; 50–375 mm/s | [Unitree](https://unitree.com/Dex1-1) |
| Mass / size | 550 g; 143 × 78 × 67 mm | [Unitree](https://unitree.com/Dex1-1) |
| Control modes | angle, angular velocity, torque, stiffness, damping | [Unitree](https://unitree.com/Dex1-1) |
| Camera (Advanced) | 1080p, 30/60 fps, 118° H / 87° V / 140° D | [Unitree](https://unitree.com/Dex1-1) |
| Box contents | gripper, 485-to-USB board, cables; RGB camera and mount (Advanced); no power supply listed | [US Robot Store](https://www.usrobotstore.com/products/dex1-1-gripper-compatible-with-all-unitree-g1-models-and-h2) (reseller) |

Peak electrical power is about 84 W at 24 V (3.5 A × 24 V), computed here and not stated by Unitree.

## Software path (from the source code)

- **Chain:** PC → USB → 485 board → gripper motor (Unitree M4010, `main.cpp`). The service `dex1_1_gripper_server` finds the motor on `/dev/ttyUSB*`, `/dev/ttyCH343USB*` or `/dev/ttyACM*`. Motor ID 0 is published as `right` and ID 1 as `left`.
- **Topics:** `rt/dex1/<side>/cmd` (type `unitree_go::msg::dds_::MotorCmds_`) and `rt/dex1/<side>/state` (`MotorStates_`).
- **Command fields:** `q`, `dq`, `tau`, `kp`, `kd`. The service scales these by the motor gear ratio, so the client works at the output shaft.
- **State fields:** `q`, `dq` and `tau_est` are filled. Temperature and other fields are not filled.
- **Control law:** the motor runs in FOC mode, i.e. position, velocity and torque terms with stiffness `kp` and damping `kd`. Unitree's teleop uses `kp = 5.0`, `kd = 0.05`, `dq = tau = 0`.
- **Loop rate:** the service exchanges one serial frame every 2000 µs, i.e. 500 Hz (`RecurrentThread` interval in microseconds, `unitree_sdk2`). This is lower than the 1000 Hz on the product page.
- **Safety:** if no command arrives within the subscription timeout, the motor switches to BRAKE mode with zero gains.
- **Dependencies:** `unitree_sdk2` (DDS), `libserialport`, `spdlog`, Boost, `yaml-cpp`, `fmt`. The DDS network interface is set with `-n`. Whether it runs on the loopback interface of a single PC is unverified.
- **Calibration:** close the gripper by hand and run `-c`. The calibration uses 0 to 322° (5.62 rad).

**Position scale.** The sources disagree:
- The `xr_teleoperate` comments state 5.4 rad ↔ 90 mm, i.e. 0.6 rad per cm. Another comment in the same file says "0.6 cm/rad · 9 rad = 5.4 cm".
- The service calibrates over 322° (5.62 rad).
- The URDF gives each finger −0.02 to 0.0245 m, i.e. 89 mm in total.

The command-to-width curve must therefore be measured, which the D09 row "Dex1-1 command → fingertip width" already covers.

## Answers to the daily-note questions

| Question | Answer | Status |
| --- | --- | --- |
| How does the RS-485 protocol work? | Unitree motor frames at 6 Mbit/s through the 485-to-USB board. The user does not touch the protocol: `dex1_1_service` exposes DDS topics | From source code |
| Which computing unit? | The recording PC (Linux, x86-64 libraries shipped). No Jetson or onboard board needed | Inferred; not run |
| Power supply? | 24 or 48 V DC, at least 3.5 A peak. Bench supply plus a cable to the handheld | Product page; connector and cable type unverified (support page §4.2) |
| F/T data? | No F/T sensor. Only the motor torque estimate `tau_est` | From source code |
| How does PICO actuation work? | XRoboToolkit gives the trigger as a float (see [[2026-11-03 PICO 4 Ultra trigger readout]]). A program maps it to `q` and publishes it on `rt/dex1/<side>/cmd` | Pattern used in `xr_teleoperate`; not tested |
| Binary or continuous? | Either. Proportional (width follows the trigger) is what Unitree's teleop does. Binary is a threshold on the trigger | Software choice |

## Consequences for the decisions (not yet decided)

- **[[D05 Gripper width sensing]]:** with the real gripper, width comes from `q`, and both the command and the achieved width are recorded. This removes the passive-vs-commanded width mismatch (research gaps B4). The handheld then records the same gripper signal as teleop.
- **[[D10 Handheld mechanism]]:** no rails, pinion, lever or spring are needed. The index finger works the PICO trigger directly. The grip sleeve design carries over, with a bracket to the real gripper body.
- **[[D02 Policy camera]]:** the camera stays on the gripper.
- **New costs:**
  - A power cable and a USB cable to the handheld.
  - 550 g of gripper, plus the controller and sleeve (mass unmeasured).
  - Gripper remount on the UR10e, if one gripper does both roles.
  - Grip force set indirectly through `kp` and the position error.

## Open

- Read the support page §4.2 (wiring, connector, cable) in a browser.
- Confirm with the supplier or the delivered box which version and accessories arrived.
- Check whether the fingers can be back-driven by hand when the motor is unpowered or braked. This matters for safety and for hand-guided corrections; unverified.
- Measure the latency from trigger to finger motion, the command-to-width curve, and `tau_est` against a force gauge.
- Unverified whether the UR10e tool connector can supply 24 V at 3.5 A peak for the robot-side gripper, or whether it needs its own supply there too.
