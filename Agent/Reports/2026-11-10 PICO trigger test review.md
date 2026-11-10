---
date: 2026-11-10
tags: [report, pico, xrobotoolkit, sim]
---
# PICO trigger test review, in plain words

On 2026-11-09 the trigger of the PICO controller opened and closed a simulated Dex1-1 gripper on the Mac. This note explains how that worked, what was decided and why, and what is still unknown. Technical details sit in the appendix at the end, so the main text needs no coding knowledge.

Checked by reading the public source code of XRoboToolkit and Unitree, the XRoboToolkit paper (zhao2025) and the MuJoCo documentation. Nothing was re-run today. Anything not confirmed is marked **unverified**.

## 1. The story in one minute

Squeezing the trigger creates a number between 0 (not pressed) and 1 (fully pressed). That number has to travel through four devices before the simulated gripper moves. It is like a message passed along a line of people, where each person speaks a different language and so needs a translator:

1. **Controller.** Your finger squeezes the trigger.
2. **Headset.** It runs a small app called XRoboToolkit that reads the trigger and sends it over Wi-Fi.
3. **Lab Linux PC (`fractal-pc`).** A program called the *PC Service* receives it. A short script of ours (`trigger_udp.py`) asks the service for the number 30 times per second and forwards it.
4. **Your Mac.** Our simulation program receives the number and closes the simulated fingers by that amount. Squeezing halfway closes them halfway.

Why does the Linux PC sit in the middle? XRoboToolkit's receiving software only exists for Windows and Linux, not for Mac. So the Linux PC catches the number, and the Mac only does the drawing.

## 2. Who owns what

| Piece | What it is | Runs on | Whose code |
| --- | --- | --- | --- |
| XRoboToolkit app | The headset app that sends the trigger | Headset | XRoboToolkit (public) |
| PC Service | The program that catches the headset's data | Linux PC | XRoboToolkit (public) |
| `xrobotoolkit-sdk` | A Python add-on that lets a Python script read the PC Service's data | Linux PC | XRoboToolkit (public) |
| `trigger_udp.py` | Reads the number and forwards it to the Mac | Linux PC | Ours, in `msc-thesis/software/xr/` |
| `dex1_1_mujoco.py` | The simulation: Dex1-1 model, receiving the number, moving fingers | Mac | Ours, in `msc-thesis/software/sim/` |

So everything XRoboToolkit-related is someone else's public code that we installed or only read. Only the two last rows are ours. Nothing in this review changed any code.

## 3. Words you asked about

- **Repository / "repo":** a folder of code on GitHub. Three are involved here: `msc-thesis` (yours), `msc-thesis-vault` (your notes), and XRoboToolkit's public ones, which belong to the XRoboToolkit authors.
- **`Assets/`:** a folder inside XRoboToolkit's headset-app repository on GitHub (`XRoboToolkit-Unity-Client`). It is the source code of the headset app. It is not in your repo and not on the Linux PC. I read it online to find out what the app does. The app itself, already installed on your headset, was built from it. (Whether the installed version equals the public code is **unverified**.)
- **Service:** a program that keeps running in the background and waits for others to talk to it. The PC Service waits for the headset.
- **SDK:** a toolbox. Here, `xrobotoolkit-sdk` is the toolbox our script uses to ask the PC Service for the trigger.
- **gRPC:** a standard way for two programs to call each other. Here the PC Service and our script run on the same Linux PC and use it to talk. Think of it as an internal phone line inside the PC. It never leaves the PC.
- **Pybind:** glue code. The PC Service is written in C++, which Python cannot talk to directly. Pybind makes a Python-friendly front door, which is what `xrobotoolkit-sdk` is.
- **Mutex:** a lock. If one part of a program writes a number while another reads it, the reader could see half of the old and half of the new. A mutex makes them take turns, like a bathroom door lock. It is only an inner detail of XRoboToolkit; you never touch it.
- **Port:** a numbered door on a computer. Each program listens at its own door, and the firewall decides which doors stay open (63901 for the headset; 5005 for our Mac link).
- **TCP and UDP:** two ways of sending data. TCP is like registered mail: it arrives complete and in order. UDP is like postcards: it is fast, and a lost one is not resent. We use UDP to the Mac, because a stale trigger value is not worth resending.
- **Timestamp:** a clock reading attached to a message.
- **Firewall (ufw):** the lab PC's door guard. On 2026-11-09 we told it to let the headset in at door 63901.
- **MuJoCo:** a program that simulates physical objects. Here it draws and moves the Dex1-1.

## 4. What the review found

1. **The doors are confirmed by the source code.** The headset knocks at door **63901** of the Linux PC, which is the one we had to open in the firewall. The internal phone line inside the PC uses door **60061**. Both match what we measured on 2026-11-09. **Door 42952 is probably not needed:** the PC Service shouts its own address into the network every 5 seconds, so that the headset can find it, and 42952 is most likely just the random door that shouting leaves from. So the firewall rule for 42952 can probably go (**unverified**: not tested).
2. **What the time stamp means.** Each message from the headset carries a clock reading. The headset app takes it from the headset's own clock, at the moment it packs the message, not when the finger actually moved. Two consequences. First, the headset clock and the Linux PC clock can differ, and we do not know by how much until we measure (this answers half of research gap B7). Second, the project rule "all times on the recording PC clock" will need a conversion.
3. **Speed.** The headset sends 90 updates per second (paper, section II-B). Our script only asks 30 times per second, which is plenty for the simulation. A real recorder should take all 90.
4. **Two ways to turn the trigger into finger motion.** Ours is the simplest: squeezing 0 → 1 closes the fingers 0 → fully. XRoboToolkit's own example does the same. Unitree's own teleoperation program (code on GitHub) does something different for the Dex1-1: below about 0.3 the gripper stays fully open, above about 0.5 it is fully closed, and only in between does it follow the finger. I derived this from their code (**not tested**). It matters later for your "proportional versus on/off trigger" test, because it is a design choice with a precedent on both sides.
5. **Nobody published trigger delay.** The paper only measures the delay of the *video*, around 80 to 100 ms. How long it takes from squeeze to finger motion is not documented anywhere I could find. It has to be measured (**open**).

## 5. Your question: can we find out the trigger's resolution?

"Resolution" means how many different values the trigger can report between 0 and 1: 10 steps, 256, or smooth?

Nobody documents it; the app just passes along a decimal number from the PICO system. Yes, we can measure it ourselves:

1. Squeeze the trigger very slowly from 0 to 1, over about 20 seconds, while our script logs every value (it already prints them).
2. Count how many *different* values appear. If there are around 255 distinct steps, the resolution is about 8 bit. If there are only a handful, it is coarse.
3. From the same log you also get the dead zone (the travel at the start where nothing registers), and whether the value rises evenly with finger travel.

This is already on your TODO as "Measure trigger value vs travel". It needs about 15 minutes, the headset and the Linux PC, and no new code. Do you want me to prepare a short step-by-step for it?

## 6. Decisions of 2026-11-09 and why

| Decision | Why | Is the reason backed up? |
| --- | --- | --- |
| Use the Linux PC as the middle station | No Mac version of the PC Service or SDK exists | Yes: XRoboToolkit's own READMEs say so |
| Install the SDK in its own isolated Python environment, with a fixed version | So nothing else on the PC breaks, and the result can be repeated | Yes: the version matches the public code |
| Check the installer's checksum | To be sure the file is the genuine one | Yes: it matched the release |
| Open TCP door 63901 for the lab network only | The headset stayed on "connecting" until then | Yes: source code confirms the door |
| Also open UDP door 42952 | Tried while troubleshooting | No: probably not needed (see 4.1) |
| Switch on "Tracking → Controller" and "Send" in the headset app | The trigger read 0.000 until then | Yes: XRoboToolkit's README describes both switches |
| Start the PC Service by hand each session, and stop it at the end | It does not start by itself; every reconnect left an unused connection behind | Partly: the README gives the start command and says nothing about autostart; the leftover connections were seen, not explained |
| Send values to the Mac by UDP, port 5005, 30 times per second | Simple; 30 matches the project's training data rate | Own choice; no source needed |
| Keep the last value if messages stop | So the simulation does not jump | Own choice; fine for a simulation. For the **real** gripper a safe behaviour on signal loss is still undecided |
| Build the simulation from Unitree's official model (URDF 261030), 0.041 m per finger | It is the official geometry | Yes, but the real stroke (82 or 90 mm) is still to be measured on the real gripper |
| Move only one finger directly, and make the other follow | The same rule XRoboToolkit's sample uses for parallel grippers | Yes: its `teleop_details.md` says so |
| Fingers' stiffness and damping values | They give a calm motion | Picked by us, not Dex1-1 data; the code says so |
| Collisions off | The finger models overlap each other and would jam | Stated in the code. The simulation shows motion only, not gripping |

## 7. Review of our two scripts and the setup guide

- `trigger_udp.py` does its job. One limit: it reads the number and the clock reading in two separate steps, so they could belong to two different messages. Harmless for the simulation; matters for recording.
- `dex1_1_mujoco.py` does its job. Small things: it accepts packets from anyone on the network without checking who sent them (harmless in a simulation, worth remembering before the same idea drives the real gripper), and a restarted sender could confuse its lost-packet counter.
- `docs/setup/pico-trigger-sim.md` is correct, except for two sentences: "ports not documented" should say "not in the docs, but set in the source", and the guess about door 42952 can now be firmer. I did not change the guide.

## 8. Open items

- Measure resolution, dead zone and linearity of the trigger (section 5).
- Measure the clock offset between headset and PC, and the delay from squeeze to finger.
- Decide how the real gripper maps the trigger (full range, or Unitree's 0.3 to 0.5 band) and what it does when the signal is lost. This belongs to the proportional versus on/off test.
- Optional: remove the firewall rule for door 42952; check the lab network mask with `ip -4 addr show enp3s0`.
- Optional: fix the two sentences in the guide (small msc-thesis PR).

## Appendix: technical details and sources

Source commits read:
- [XRoboToolkit-PC-Service](https://github.com/XR-Robotics/XRoboToolkit-PC-Service/tree/85bac4dbc1fd5cef42c74a160d9c30aa3491f122) `85bac4d`
- [XRoboToolkit-PC-Service-Pybind](https://github.com/XR-Robotics/XRoboToolkit-PC-Service-Pybind/tree/8941e1dbbf4313fe83846f82984eaa8c992690bf) `8941e1d` (= `xrobotoolkit-sdk` 1.1.0)
- [XRoboToolkit-Unity-Client](https://github.com/XR-Robotics/XRoboToolkit-Unity-Client/tree/cdc53166b0bf412efae71046c6a225eb5091605f) `cdc5316` (the headset app)
- [XRoboToolkit-Teleop-Sample-Python](https://github.com/XR-Robotics/XRoboToolkit-Teleop-Sample-Python/tree/79e5cb8a56e3455515ce1b476e993c764ec58739) `79e5cb8`
- Unitree [xr_teleoperate](https://github.com/unitreerobotics/xr_teleoperate/tree/817fb00c63cde15e5f24a0f8fa08e1e33ed89d3b) `817fb00` and [televuer](https://github.com/unitreerobotics/televuer/tree/766de45e74373ae0ea66321d942ce538385655a5) `766de45`
- Paper: [XRoboToolkit (zhao2025)](https://arxiv.org/html/2508.00097)

| Claim | Where in the source |
| --- | --- |
| Trigger is a decimal read from the headset system and sent as `"trigger"` | Unity client `Assets/Scripts/TrackingData.cs`, function `GetControllerJsonData` (`CommonUsages.trigger`) |
| "analog controls with values between 0 and 1"; "single JSON object at 90 Hz" | zhao2025, section II-B |
| Headset connects to TCP 63901 | Unity client `Assets/Scripts/Network/TcpHandler.cs` (`TCP_PORT = 63901`); PC Service `Manage_global.h` (`DEFAULT_TCP_PORT 63901`); listens on all addresses (`tcpserverworker.cpp`) |
| PC Service gRPC on 127.0.0.1:60061 | `PXREAGRPCServer.cpp` (`Service/listenAddr`, `Service/listenPort` in `setting.ini`) |
| SDK keeps the latest value per field, each under a mutex | Pybind `bindings/py_bindings.cpp` (`RightTrigger = right["trigger"]`, `rightMutex`) |
| `xrt.close()` maps to `PXREADeinit` | `py_bindings.cpp`, `XRT_FUNCTION("close", deinit, …)` |
| Discovery broadcast every 5 s to `<ip>.255:29888`; headset listens there | `tcpserverworker.cpp` (`onBraodCastTCPServer`, `BROADCAST_MEADIA_SERVER_INTERVAL 5000`); `UIUdpReceiver.cs` (`UdpListenPort = 29888`). That 42952 is its sending port: unverified |
| `timeStampNs` = headset UTC clock at packing | `TrackingData.cs` (`Utils.GetCurrentTimestamp()`); `Utils.cs` (`DateTime.UtcNow`); the sensor-time line is commented out |
| No macOS support | PC Service README (Windows, Linux x86, Linux aarch64); Pybind README and `CLAUDE.md` |
| Run command, switches | Unity client README (`runService.sh`; Tracking and Data & Control table) |
| Only one finger joint driven, other by equality constraint | Teleop sample `teleop_details.md` |
| Linear 0 to 1 mapping | Teleop sample `utils/parallel_gripper_utils.py` (`calc_parallel_gripper_position`) |
| Unitree mapping: `10 − 10 t`, then interpolation over `[5, 7]` to `[0, 5.4]` rad, ±0.18 rad step limit, 200 Hz | `televuer` `tv_wrapper.py`; `xr_teleoperate` `teleop/robot_control/robot_hand_unitree.py` (`Dex1_1_Gripper_Controller`); inferred from code, not run |
| Video delay 82 to 100.5 ms; no trigger delay published | zhao2025, section IV-A, Table II |
| Position actuator form and passive viewer on macOS (`mjpython`) | MuJoCo docs: [position actuator](https://mujoco.readthedocs.io/en/stable/XMLreference.html#actuator-position), [passive viewer](https://mujoco.readthedocs.io/en/stable/python.html#passive-viewer) |

The sim's stiffness is 2000 N/m per joint (the gain is that value times 0.041 squared), checked by reading the code.
