---
date: 2026-11-10
tags: [report, pico, xrobotoolkit, sim]
---
# PICO trigger test review, in plain words

On 2026-11-09 the trigger of the PICO controller opened and closed a simulated Dex1-1 gripper on the Mac. This note explains how that worked, what was decided and why, and what is still unknown. File names and source links for checking are collected in the appendix.

**How to read it:** every program, folder or file is tagged with where it lives and who wrote it. The tags are *headset*, *Linux PC* (the lab computer `fractal-pc`) and *Mac*, and the code is either *XRoboToolkit's* (public code written by others, which we only installed or read) or *ours* (your `msc-thesis` repo). Each technical word is explained where it first appears.

**How it was checked:** I read the public source code of XRoboToolkit and Unitree, the XRoboToolkit paper (zhao2025) and the MuJoCo documentation. Nothing was re-run today. Anything not confirmed is marked **unverified**.

## 1. The story in one minute

Squeezing the trigger creates a number between 0 (not pressed) and 1 (fully pressed). That number travels through four devices before the simulated gripper moves. It is like a message passed along a line of people who speak different languages, so each hop needs a translator.

```
 finger ─► controller ─► headset ──Wi-Fi──► Linux PC ──network cable──► Mac
                          XRoboToolkit app   PC Service + our sender    our simulation
```

**Step 1. Controller.** Your finger squeezes the trigger. The controller reports it to the headset over PICO's own wireless link.

**Step 2. Headset.** The headset runs an app called *XRoboToolkit* (XRoboToolkit's code, already installed on the headset). It reads the trigger and sends it over Wi-Fi to the Linux PC.
- *Where does that app's code live?* The app's source code is a public project on GitHub called `XRoboToolkit-Unity-Client`. "Unity" is the game engine the app is built with. Inside that project, the folder `Assets/` holds the app's own code. So `Assets/` is **not** in your `msc-thesis` repo, **not** on the Linux PC and **not** on your Mac. I read it online, only to find out what the installed app does. Whether the installed app 1.1.1 matches the public code exactly is **unverified**.
- The app sends the data to a numbered "door" of the Linux PC, called a *port*. Every program on a computer listens at its own port number, and the PC's firewall (a door guard, here `ufw`) decides which ports stay open. The headset uses port 63901. The messages travel by *TCP*, a way of sending data that is checked and resent if lost, like registered mail.

**Step 3. Linux PC.** Two programs work here.
- The *PC Service* (XRoboToolkit's code, installed on 2026-11-09 from a `.deb` file, which is an installer package for Ubuntu; it lives in `/opt/apps/roboticsservice/` on the Linux PC). A *service* is a program that runs in the background and waits for others to talk to it. This one waits for the headset and keeps the newest value of every control. It does not start by itself, so it is started by hand each session.
- Our sender `trigger_udp.py` (ours). It cannot read the PC Service directly, so it uses a Python add-on called `xrobotoolkit-sdk` (XRoboToolkit's code; installed on the Linux PC in a separate Python environment called `xrobotoolkit`). An *SDK* is a toolbox that lets your own script use someone else's program. This one is made with *Pybind*, a tool that builds a Python front door onto the PC Service's code, which is written in C++, a language Python cannot talk to directly. The add-on then asks the PC Service through *gRPC*, a standard way for two programs to call each other. Here that happens inside the Linux PC, over an internal "phone line" at port 60061 that never leaves the machine. Our sender asks for the trigger 30 times per second and sends each value to the Mac as a small message by *UDP* to port 5005. UDP is like a postcard: fast, but a lost one is not resent. That is fine here, because the next value comes 33 ms later.

**Step 4. Mac.** Our simulation `dex1_1_mujoco.py` (ours, in `msc-thesis/software/sim/`) receives the number. It moves the simulated fingers by that fraction of their travel, using a physics program called *MuJoCo*. Squeezing halfway closes the fingers halfway.

**Why is the Linux PC in the middle?** XRoboToolkit's PC Service and add-on exist only for Windows and Linux, not for Mac. So the Linux PC catches the number, and the Mac only draws the gripper.

### Where `trigger_udp.py` lives: two copies

The file exists twice, and you assumed right that one is a copy. It is the other way round: the original is the one on the **Mac**.
- **Mac:** `~/msc-thesis/software/xr/trigger_udp.py`. This is the master copy. It is in your repo, so Git and GitHub track it.
- **Linux PC:** `~/xr_trigger/trigger_udp.py`. On 2026-11-09 it was copied there once with the command `scp` (copy a file between computers over a secure connection), as step 4 of `docs/setup/pico-trigger-sim.md` says. The Linux PC has no clone of your repo (the guide says "until `software/xr/` is on `main` on the PC"), so Git does not update this copy.
- **What follows:** change the file only on the Mac, then copy it again. If someone edits the Linux copy, the Mac one will not know. Whether the two are identical today is **unverified**. This command prints one checksum per copy (a fingerprint of the file; equal fingerprints mean equal files):
  ```
  shasum -a 256 ~/msc-thesis/software/xr/trigger_udp.py; ssh fractal-pc sha256sum ~/xr_trigger/trigger_udp.py
  ```

## 2. What the review found

1. **The ports are confirmed by the source code.** The headset app connects to port **63901** (the firewall rule we had to add). The internal phone line is port **60061**. Both match what was measured on 2026-11-09. Port **42952** is probably not needed: the PC Service shouts its own address into the network every 5 seconds, so the headset can offer it in its IP prompt. That shout leaves from a randomly chosen port, and 42952 is most likely that one (**unverified**, not tested). The firewall rule for 42952 can therefore probably go.
2. **What the time stamp means.** Every message from the headset carries a *timestamp*, a clock reading. The headset app takes it from the headset's own clock at the moment it packs the message, not when the finger moved. Two consequences. The headset clock and the Linux PC clock can differ, and we do not know by how much until we measure. And the project rule "all times on the recording PC clock" will need a conversion. This answers half of the open question B7 in the research gaps.
3. **Speed.** The headset sends 90 updates per second (paper, section II-B). Our sender asks only 30 times per second, which is enough for the simulation. A real recorder should capture all 90.
4. **Possible mix-ups inside the add-on.** The add-on keeps the newest value of each control behind a *mutex*, a lock that makes two parts of a program take turns (like a bathroom door lock), so nobody reads a number while it is being written. The trigger and the timestamp have separate locks, so our sender can pair a trigger with the timestamp of a neighbouring message. Harmless for the simulation, relevant for recording.
5. **Two ways to turn the trigger into finger motion.** Ours is the simplest: trigger 0 to 1 moves the fingers from open to fully closed. XRoboToolkit's own sample does the same. Unitree's own teleoperation program (public code) does something else for the Dex1-1: below about 0.3 the gripper stays fully open, above about 0.5 it is fully closed, and only in between does it follow the finger (derived from their code, **not tested**). This matters for your later test of a proportional versus an on/off trigger, and for fairness between handheld and VR teleop (see the new research note below).
6. **Nobody published the trigger delay.** The paper measures only the delay of the *video* (82 to 100.5 ms). The delay from squeeze to finger motion is not documented anywhere I found. It has to be measured.

## 3. Can we find out the trigger's resolution?

*Resolution* means how many different values the trigger can report between 0 and 1: 10 steps, 256 steps, or smooth. Nobody documents it. The app simply passes on a decimal number from the PICO system. We can measure it ourselves:

1. Squeeze the trigger very slowly from 0 to 1 over about 20 seconds while `trigger_udp.py` logs every value (it already prints one per second; a full log needs a small change, which I have not made).
2. Count how many *different* values appear. About 255 distinct steps means roughly 8 bit; only a handful means a coarse trigger.
3. The same log shows the dead zone (the start of the travel where nothing registers) and whether the value rises evenly with finger travel.

This is already on your TODO as "Measure trigger value vs travel".

## 4. Decisions of 2026-11-09 and why

| Decision | Why | Is the reason backed up? |
| --- | --- | --- |
| Use the Linux PC as the middle station | No Mac version of the PC Service or add-on exists | Yes: XRoboToolkit's own READMEs say so |
| Install the add-on in its own isolated Python environment, with a fixed version (1.1.0) | So nothing else on the PC breaks and the result can be repeated | Yes: the version matches the public code |
| Check the installer's checksum | To be sure the file is the genuine release | Yes: it matched |
| Open TCP port 63901 for the lab network only | The headset stayed on "connecting" until then | Yes: the source confirms the port |
| Also open UDP port 42952 | Tried while troubleshooting | No: probably not needed (finding 1) |
| Switch on "Tracking → Controller" and "Data & Control → Send" in the headset app | The trigger read 0.000 until then | Yes: XRoboToolkit's README describes both switches |
| Start the PC Service by hand each session and stop it at the end | It does not start by itself; every reconnect left an unused connection behind | Partly: the README gives the start command and says nothing about autostart; the leftover connections were seen, not explained |
| Send values to the Mac by UDP, port 5005, 30 times per second | Simple; 30 matches the project's training data rate | Own choice; no source needed |
| Hold the last value if messages stop | So the simulation does not jump | Own choice. Fine for a simulation. For the real gripper, the behaviour on signal loss is undecided |
| Build the simulation from Unitree's official model (URDF 261030), 0.041 m per finger | It is the official geometry | Yes, but the real stroke (82 or 90 mm) is still to be measured on the real gripper |
| Drive one finger and make the other follow | The rule XRoboToolkit's sample gives for parallel grippers | Yes: its `teleop_details.md` says so |
| Finger stiffness and damping values | They give a calm motion | Picked by us, not Dex1-1 data; the code says so |
| Collisions off | The finger models overlap each other and would jam | Stated in the code. The simulation shows motion only, not gripping |

## 5. Review of our two scripts and the setup guide

- `trigger_udp.py` does its job. It reads the trigger and the timestamp in two separate steps (finding 4). Its "clean up" call at the end maps to the add-on's real shutdown function.
- `dex1_1_mujoco.py` does its job. It accepts messages from anyone on the network without checking the sender (harmless in a simulation, worth remembering before the same idea drives the real gripper). A restarted sender could confuse its lost-message counter.
- `docs/setup/pico-trigger-sim.md` is correct except for two sentences: "ports not documented" should say "not in the docs, but set in the source", and the guess about port 42952 can be firmer. I did not change the guide.

## 6. Open items

- Measure resolution, dead zone and linearity of the trigger (section 3).
- Measure the clock offset between headset and PC, and the delay from squeeze to finger.
- Decide how the real gripper maps the trigger (full range, or Unitree's 0.3 to 0.5 band) and what it does when the signal is lost.
- Optional: remove the firewall rule for port 42952; check the lab network mask with `ip -4 addr show enp3s0`.
- Optional: fix the two sentences in the guide (small msc-thesis PR).
- Research gaps found from this test: see [[2026-11-10 Research gaps from the PICO sim test]].

## Appendix: sources for checking

Source commits read:
- [XRoboToolkit-PC-Service](https://github.com/XR-Robotics/XRoboToolkit-PC-Service/tree/85bac4dbc1fd5cef42c74a160d9c30aa3491f122) `85bac4d`
- [XRoboToolkit-PC-Service-Pybind](https://github.com/XR-Robotics/XRoboToolkit-PC-Service-Pybind/tree/8941e1dbbf4313fe83846f82984eaa8c992690bf) `8941e1d` (= `xrobotoolkit-sdk` 1.1.0)
- [XRoboToolkit-Unity-Client](https://github.com/XR-Robotics/XRoboToolkit-Unity-Client/tree/cdc53166b0bf412efae71046c6a225eb5091605f) `cdc5316` (the headset app)
- [XRoboToolkit-Teleop-Sample-Python](https://github.com/XR-Robotics/XRoboToolkit-Teleop-Sample-Python/tree/79e5cb8a56e3455515ce1b476e993c764ec58739) `79e5cb8`
- Unitree [xr_teleoperate](https://github.com/unitreerobotics/xr_teleoperate/tree/817fb00c63cde15e5f24a0f8fa08e1e33ed89d3b) `817fb00` and [televuer](https://github.com/unitreerobotics/televuer/tree/766de45e74373ae0ea66321d942ce538385655a5) `766de45`
- Paper: [XRoboToolkit (zhao2025)](https://arxiv.org/html/2508.00097)

| Claim | Where in the source |
| --- | --- |
| Trigger is a decimal read from the headset system and sent as `"trigger"` | Headset app `Assets/Scripts/TrackingData.cs`, function `GetControllerJsonData` (`CommonUsages.trigger`) |
| "analog controls with values between 0 and 1"; "single JSON object at 90 Hz" | zhao2025, section II-B |
| Headset connects to TCP 63901 | Headset app `Assets/Scripts/Network/TcpHandler.cs` (`TCP_PORT = 63901`); PC Service `Manage_global.h` (`DEFAULT_TCP_PORT 63901`); listens on all addresses (`tcpserverworker.cpp`) |
| PC Service gRPC on 127.0.0.1:60061 | PC Service `PXREAGRPCServer.cpp` (`Service/listenAddr`, `Service/listenPort` in `setting.ini`) |
| Add-on keeps the latest value per field, each under a mutex | Pybind `bindings/py_bindings.cpp` (`RightTrigger = right["trigger"]`, `rightMutex`, `timestampMutex`) |
| `xrt.close()` maps to `PXREADeinit` | `py_bindings.cpp`, `XRT_FUNCTION("close", deinit, …)` |
| Discovery shout every 5 s to `<ip>.255:29888`; headset listens there | PC Service `tcpserverworker.cpp` (`onBraodCastTCPServer`, `BROADCAST_MEADIA_SERVER_INTERVAL 5000`); headset app `UIUdpReceiver.cs` (`UdpListenPort = 29888`). That 42952 is its sending port: unverified |
| `timeStampNs` = headset UTC clock at packing | `TrackingData.cs` (`Utils.GetCurrentTimestamp()`); `Utils.cs` (`DateTime.UtcNow`); the sensor-time line is commented out |
| No macOS support | PC Service README (Windows, Linux x86, Linux aarch64); Pybind README and `CLAUDE.md` |
| Run command, headset switches | Headset app README (`runService.sh`; Tracking and Data & Control table) |
| Only one finger joint driven, the other by equality constraint | Teleop sample `teleop_details.md` |
| Linear 0 to 1 mapping | Teleop sample `utils/parallel_gripper_utils.py` (`calc_parallel_gripper_position`) |
| Unitree mapping: `10 − 10 t`, then interpolation over `[5, 7]` to `[0, 5.4]` rad, ±0.18 rad step limit, 200 Hz | `televuer` `tv_wrapper.py`; `xr_teleoperate` `teleop/robot_control/robot_hand_unitree.py` (`Dex1_1_Gripper_Controller`); inferred from code, not run |
| Video delay 82 to 100.5 ms; no trigger delay published | zhao2025, section IV-A, Table II |
| Position actuator form; passive viewer needs `mjpython` on macOS | MuJoCo docs: [position actuator](https://mujoco.readthedocs.io/en/stable/XMLreference.html#actuator-position), [passive viewer](https://mujoco.readthedocs.io/en/stable/python.html#passive-viewer) |
| Copy of `trigger_udp.py` to the Linux PC | `docs/setup/pico-trigger-sim.md` section 2 (Linux PC), step 4 |

The simulation's stiffness is 2000 N/m per joint (its gain is that value times 0.041 squared), checked by reading the code.
