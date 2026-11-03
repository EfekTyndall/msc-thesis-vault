---
date: 2026-11-03
tags: [research, hardware, pico]
---
# PICO 4 Ultra Enterprise: trigger readout via XRoboToolkit

Question: how the PICO controller trigger reaches the recording PC, so that it can drive the real Dex1-1 ([[2026-11-03 Dex1-1 technical note]]) or be recorded as the gripper signal. Scope is limited to the controller inputs. Headset facts are in [[D04 Pose tracking]].

Method: source code of [XRoboToolkit-PC-Service-Pybind](https://github.com/XR-Robotics/XRoboToolkit-PC-Service-Pybind/tree/8941e1dbbf4313fe83846f82984eaa8c992690bf) (commit 8941e1d, 2026-10-14) and [XRoboToolkit-Teleop-Sample-Python](https://github.com/XR-Robotics/XRoboToolkit-Teleop-Sample-Python/tree/79e5cb8a56e3455515ce1b476e993c764ec58739) (commit 79e5cb8, 2025-12-31). Not tested on the headset.

## Findings

- **Inputs per controller:**
  - Pose `[x, y, z, qx, qy, qz, qw]`
  - `trigger` and `grip` as floats
  - Menu, primary, secondary and axis-click buttons
  - Axis x/y

  All arrive in one JSON message from the PC service (`py_bindings.cpp`, callback `OnPXREAClientCallback`).
- **Python API:** `xrt.get_right_trigger()`, `xrt.get_left_trigger()`, `get_*_grip()`, `get_*_controller_pose()`, `get_time_stamp_ns()` (package `xrobotoolkit-sdk`, Linux x86-64/aarch64 or Windows, Python ≥ 3.10).
- **Range:** the trigger is analog, 0.0 when released and 1.0 when fully pressed. This is inferred from the teleop sample, which raises an error outside [0, 1] and maps the value linearly between the open and closed gripper position (`calc_parallel_gripper_position`, also used for its UR gripper config). The resolution is not documented.
- **Timestamp:** one `timeStampNs` per message, so trigger and pose share a timestamp. Its meaning (sample or send time, and on which clock) is still undocumented, as noted in D04 and research gaps B7.
- **Latest-value API:** each getter returns the last received value under its own lock. A recorder that polls the getters can mix two messages, or miss messages. For recording, the message should be read once per callback, or pose, trigger and timestamp read together. This is inferred from the code, not measured.
- **Unitree's own teleop** (`xr_teleoperate`, which uses WebXR through `televuer`, not XRoboToolkit) reads the same trigger as 0–1 and rescales it to 10 → 0. It then maps it linearly to the Dex1-1 position. This confirms that proportional trigger control of the Dex1-1 is an existing pattern.

## Consequences

- With the real Dex1-1 as the handheld, the index finger works the controller trigger directly, and the trigger value becomes the gripper command. No lever or encoder is needed.
- With the printed copy, the lever in front of the trigger ([[D10 Handheld mechanism]]) presses the controller trigger as it closes the fingers. The trigger value could then serve as a coarse width signal. This depends on the lever geometry and is unverified.

## Open

- Measure the trigger value against trigger travel (linearity, dead zone, resolution) on the PICO 4 Ultra Enterprise controller.
- Measure the latency from trigger press to arrival at the PC, together with the pose latency (D06).
- Whether the PC service also delivers controller inputs from the Enterprise firmware without changes is unverified (D04: Enterprise and consumer controllers are reported as the same hardware).
