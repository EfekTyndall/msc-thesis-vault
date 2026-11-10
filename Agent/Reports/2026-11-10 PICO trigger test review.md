---
date: 2026-11-10
tags: [report, pico, xrobotoolkit, sim]
---
# PICO trigger test review (XRoboToolkit → MuJoCo Dex1-1)

This note reviews the 2026-11-09 test, in which the PICO 4 Ultra Enterprise right trigger drove a simulated Dex1-1 on the Mac. It covers how the chain works, each decision taken and its reason, the setup guide, and the code. Every claim is tied to a source or a measurement, or is marked unverified.

**Scope:** `msc-thesis/software/xr/trigger_udp.py`, `software/sim/dex1_1_mujoco.py`, `software/sim/run_dex1_1.sh` and `docs/setup/pico-trigger-sim.md` (msc-thesis PR #16), plus [[2026-11-09]] Done / Measured.

**Method:** code reading against the upstream source at these commits:
- [XRoboToolkit-PC-Service](https://github.com/XR-Robotics/XRoboToolkit-PC-Service/tree/85bac4dbc1fd5cef42c74a160d9c30aa3491f122) `85bac4d`
- [XRoboToolkit-PC-Service-Pybind](https://github.com/XR-Robotics/XRoboToolkit-PC-Service-Pybind/tree/8941e1dbbf4313fe83846f82984eaa8c992690bf) `8941e1d` (= SDK 1.1.0)
- [XRoboToolkit-Unity-Client](https://github.com/XR-Robotics/XRoboToolkit-Unity-Client/tree/cdc53166b0bf412efae71046c6a225eb5091605f) `cdc5316`
- [XRoboToolkit-Teleop-Sample-Python](https://github.com/XR-Robotics/XRoboToolkit-Teleop-Sample-Python/tree/79e5cb8a56e3455515ce1b476e993c764ec58739) `79e5cb8`
- Unitree [xr_teleoperate](https://github.com/unitreerobotics/xr_teleoperate/tree/817fb00c63cde15e5f24a0f8fa08e1e33ed89d3b) `817fb00` and [televuer](https://github.com/unitreerobotics/televuer/tree/766de45e74373ae0ea66321d942ce538385655a5) `766de45`
- The XRoboToolkit paper (zhao2025, read in its [HTML version](https://arxiv.org/html/2508.00097))

**Caveats:** the Unity client is a public repo whose head may not match the installed app 1.1.1, and the PC Service source may not match the installed `.deb` 1.0.0. Nothing was re-run today.

## 1. How it works (verified chain)

| Hop | What happens | Source |
| --- | --- | --- |
| Controller → headset app | The Unity app reads `CommonUsages.trigger` (a float) and puts it in the controller JSON as `"trigger"`. Range: "analog controls with values between 0 and 1" (zhao2025, Sec. II-B). Resolution: not documented. | Unity client `Assets/Scripts/TrackingData.cs` (`GetControllerJsonData`) |
| Headset → PC Service | All tracking data goes "within a single JSON object at 90 Hz" (zhao2025, Sec. II-B). The app connects over TCP to port **63901**. | `Assets/Scripts/Network/TcpHandler.cs` (`TCP_PORT = 63901`); PC Service `Manage_global.h` (`DEFAULT_TCP_PORT 63901`), listening on all interfaces (`tcpserverworker.cpp`) |
| PC Service → Python SDK | The service is a gRPC server on **127.0.0.1:60061** by default (`setting.ini` keys `Service/listenAddr`, `listenPort`). The SDK registers a callback through `PXREAInit` and keeps the latest value per field, each under its own mutex. | `PXREAGRPCServer.cpp`; Pybind `bindings/py_bindings.cpp` (`RightTrigger = right["trigger"]`) |
| SDK → `trigger_udp.py` → Mac | The script polls `get_right_trigger()` and `get_time_stamp_ns()` at 30 Hz and sends one JSON UDP packet per poll to the Mac on port 5005. | msc-thesis `software/xr/trigger_udp.py` |
| Mac sim | Takes the newest packet, sets ctrl = trigger and steps MuJoCo; finger target = 0.041 m × (1 − trigger). | `software/sim/dex1_1_mujoco.py` |

Measured on 2026-11-09 ([[2026-11-09]]):
- About 63 kB/s headset → TCP 63901 and about 67 kB/s on 60061.
- 30 packets/s with 0 lost of 425.
- Pad gap 80.0 / 39.0 / −2.0 mm at trigger 0 / 0.5 / 1.

The source confirms all three ports, so the guide's line "the ports are not documented by XRoboToolkit" should now read "not in the docs, but set in the source".

## 2. New findings from the source

1. **UDP 42952 is a discovery broadcast, not data (inferred from code).** Every 5 s (`BROADCAST_MEADIA_SERVER_INTERVAL 5000`) the PC Service sends its own IP from an unbound `QUdpSocket` to `<own IP with the last octet set to 255>:29888` (`tcpserverworker.cpp`, `onBraodCastTCPServer`). The headset app listens on UDP 29888 to fill the IP prompt (`UIUdpReceiver.cs`). An unbound socket gets an ephemeral source port, so 42952 is most likely that port and changes per run (unverified).
   - The ufw rule for UDP 42952 is therefore not needed. Removing it is still the optional TODO.
   - The code always replaces only the last octet. If the lab subnet is really /16 (the ufw rules assume 10.36.0.0/16; the netmask was not checked), then 10.36.1.255 is not the broadcast address. That could explain the 1920 bytes stuck in the send queue (unverified). It is harmless, because the IP is entered by hand.
2. **`xr_ts_ns` meaning, which resolves research gap B7 in part (from code).** The headset app sets `timeStampNs = Utils.GetCurrentTimestamp()` when it packs the JSON. That function is the headset's UTC wall clock (`DateTime.UtcNow`), converted to ns.
   - It is pack time, not sensor sample time: the line using `sensorState.poseTimeStampNs` is commented out (`TrackingData.cs`).
   - It is on the headset clock, not the PC clock. A clock offset between the headset and the recording PC is therefore unknown until measured (D06 rule: all timestamps on the recording PC clock).
3. **Rate:** the headset sends at 90 Hz (zhao2025) and `trigger_udp.py` samples the latest value at 30 Hz. That is enough for the 30 Hz export, but it drops two of three samples and adds up to 33 ms of sampling delay. A recorder should use the 90 Hz callback stream instead (as already noted in [[2026-11-03 PICO 4 Ultra trigger readout]]).
4. **Unitree's own Dex1-1 controller mapping is not linear over 0–1 (inferred from code, not tested).**
   - `televuer` turns the trigger t into `10 − 10 t` (`tv_wrapper.py`).
   - `Dex1_1_Gripper_Controller` then interpolates that value over `[5, 7]` → `[0, 5.4]` rad, the constants for the hand-tracking pinch distance (`robot_hand_unitree.py`).
   - In controller mode the gripper is therefore fully open for t ≤ 0.3 and fully closed (0 rad) for t ≥ 0.5. Only 0.3–0.5 is proportional.
   - It also limits each command step to ±0.18 rad around the measured state (`DELTA_GRIPPER_CMD`, with the comment "5.4 rad ↔ 9 cm, 0.6 rad ↔ 1 cm") in a 200 Hz loop.
   - Our sim maps 0–1 linearly over the full stroke. This is the pattern of XRoboToolkit's own `calc_parallel_gripper_position` (`parallel_gripper_utils.py`), so both precedents exist. The choice matters for the "proportional vs binary trigger" test ([[Waiting]]).
5. **Paper latency is video only.** zhao2025 Table II reports video latency, e.g. PICO 4 Ultra → PICO 4 Ultra 100.5 ± 3.1 ms. No controller or trigger latency is published, so trigger-to-finger latency stays a measurement item.

## 3. Decisions taken on 2026-11-09 and their grounding

| Decision | Reason given | Source / status |
| --- | --- | --- |
| Two computers: stream ends on Linux PC, viewer on Mac | No macOS build of PC Service or SDK; PC is SSH-only | PC Service README (Windows, Linux x86, Linux aarch64); Pybind README and its CLAUDE.md ("macOS is unsupported (no vendor binaries exist)"). Grounded. |
| SDK in its own conda env, `xrobotoolkit-sdk==1.1.0` pinned | Isolation; reproducibility | PyPI version = Pybind `pyproject.toml` 1.1.0. Grounded. |
| `.deb` checked by SHA-256 | Supply-chain check | Checksum matched the release ([[2026-11-09]]). Grounded. |
| ufw: TCP 63901 from 10.36.0.0/16 | Headset stuck "connecting" until opened | Port from source (§1). Needed and grounded; subnet mask unchecked. |
| ufw: UDP 42952 | Added during troubleshooting | Not needed (§2.1). Removal is the optional TODO. |
| Headset settings: Tracking → Controller, Data & Control → Send | Trigger stayed 0.000 without them | Unity client README table ("Toggle On/Off to parse VR controller's 6 DoF pose and button status"; "Send … sync above selected poses"). Grounded. |
| Service started by hand, restarted per session, stopped after | Does not autostart; reconnects left stale TCP connections | Unity client README run step ("Run `/opt/apps/roboticsservice/runService.sh`"). No autostart is documented. Stale connections were observed, not explained (unverified). |
| UDP + JSON, port 5005, 30 Hz | Simple; 30 Hz is the export rate | 30 Hz from msc-thesis CLAUDE.md (training export). Port 5005 and JSON are a free choice with no precedent needed. |
| Hold last value when packets stop | The sim must not jump | Own choice. Acceptable for a sim. For the real gripper, a timeout or safe state is undecided (no source checked). |
| Sim from the official URDF 261030, stroke 0.041 m per finger | URDF is the geometry reference | Project memory: service URDF outdated; use official STEP/URDF (vault PR #15). Real stroke still to measure (82 vs 90 mm). |
| One driven joint + equality constraint for finger 2 | Same as XRoboToolkit's parallel-gripper rule | `teleop_details.md`: "only 1 of the joints should be actuated, the others should be controlled by additional equality constraints". Grounded. |
| Position actuator via affine bias, stiffness 2000 N/m, damping ≈ critical for 0.1 kg, joint damping 2 N s/m, armature 0.01 | Calm response | Form matches MuJoCo's `position` actuator (gain kp, bias 0, −kp, −kv; [MuJoCo XML reference](https://mujoco.readthedocs.io/en/stable/XMLreference.html#actuator-position)), with ctrl inverted. Checked: joint stiffness = kp / 0.041² = 2000 N/m. Values are picked, not Dex1-1 data, as the code says. |
| Collisions off, gravity off | Finger meshes include the racks, which cross the centre line | Stated in the code. The sim shows motion only, no grasping. Fine for the stated purpose. |
| `mjpython` for `--udp` on macOS | `launch_passive` needs it on macOS | [MuJoCo Python docs, passive viewer](https://mujoco.readthedocs.io/en/stable/python.html#passive-viewer). Grounded. |

## 4. Code and guide review

`trigger_udp.py`
- Correct for its purpose. The trigger and the timestamp are read by two separate calls under different mutexes, so a packet can pair a trigger with the timestamp of another message. This does not matter for the sim, but it matters for recording ([[2026-11-03 PICO 4 Ultra trigger readout]]).
- `send_ts_ns` is the Linux PC clock and the Mac does not log its receive time, so no latency can be computed from the packets yet.
- `xrt.close()` in `finally` maps to `deinit` (`PXREADeinit`) in the bindings (`py_bindings.cpp`, `XRT_FUNCTION("close", deinit, …)`). Correct.

`dex1_1_mujoco.py`
- Packet loss counting ignores out-of-order packets and a sender restart (seq back to 0). Minor; a LAN with 0 loss was measured.
- The UDP port accepts packets from any host on the LAN, with no check. This is harmless for a sim, but worth keeping in mind before the same pattern drives the real gripper.

`run_dex1_1.sh` finds the conda env and swaps in `mjpython` for `--udp` on macOS. No issues found.

`pico-trigger-sim.md`: accurate against the source, except for two points (neither changed here):
- "the ports are not documented by XRoboToolkit" should read "not in the docs, but set in the source" (§1).
- UDP 42952 "probably an outgoing discovery or broadcast attempt" is now backed by the source: a broadcast to UDP 29888 every 5 s (§2.1).

## Open
- Measure the trigger value against travel (resolution, dead zone, linearity), which is already a TODO. The Unity `CommonUsages.trigger` resolution is undocumented.
- Measure the headset-to-PC clock offset and its drift (research gaps B7, D06), now that `timeStampNs` is known to be headset UTC at pack time.
- Choose the trigger mapping for the real gripper: full-range linear (XRoboToolkit sample, our sim) or Unitree's saturated 0.3–0.5 band. This belongs in the proportional vs binary test.
- Optional: remove the UDP 42952 ufw rule; check the lab netmask (`ip -4 addr show enp3s0`).
- If wanted: update the two guide lines in §4 (msc-thesis PR).
