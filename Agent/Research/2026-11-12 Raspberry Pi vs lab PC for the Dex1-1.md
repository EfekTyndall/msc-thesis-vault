---
date: 2026-11-12
tags: [research, dex1-1, compute, real-path]
---
# Raspberry Pi vs lab PC for the Dex1-1

Question (Martyn, 2026-11-11): can a Raspberry Pi replace the lab PC (`fractal-pc`, Ubuntu 22.04 x86-64, shared) as the computer that drives the Dex1-1 over RS-485? Follows the open item in [[2026-11-11 Mac-centred lab workflow]]. Builds on [[2026-11-03 Dex1-1 technical note]] and [[D06 Recording and synchronisation]].

Method: source of Unitree `dex1_1_service` ([commit ced3216](https://github.com/unitreerobotics/dex1_1_service/tree/ced3216290bb22dae356213993786beeaa7143ab), 2026-09-16) and `unitree_sdk2` ([commit aaf69cb](https://github.com/unitreerobotics/unitree_sdk2/tree/aaf69cb), 2026-11-09); XRoboToolkit READMEs and PyPI; Ubuntu's Raspberry Pi support page; prior handheld work already in `msc-thesis/docs/research-notes.md` and `docs/proposal.md`. Nothing was run on a Pi. No purchase, no install.

## Summary

1. **The gripper link itself runs on ARM64.** Unitree's own reference host for `dex1_1_service` is an ARM64 board (Jetson Orin NX, "PC2" on its robots), and the repo ships an ARM64 motor library. A Raspberry Pi 5 is the same CPU architecture, but Unitree does not name it; running there is **unverified**.
2. **The operating system is the main snag.** Ubuntu 22.04 (what Unitree and XRoboToolkit build for) is not supported on the Pi 5; Pi 5 support starts at Ubuntu 24.04. So a Pi means a different OS from the tested one.
3. **A Pi does not remove the cable to the handheld.** The gripper needs 24–64 V at up to 3.5 A, so a power cable (or a large battery) goes to the handheld whatever computer drives it.
4. **A Pi cannot replace the whole PC**: no hardware video encoder, and the PC is also the planned training machine (GPU).
5. **Most defensible use of a Pi:** a dedicated gripper computer that runs only `dex1_1_service` and starts it at boot, talking to the PC over Ethernet. This copies Unitree's own split (service on PC2, user code elsewhere over DDS). It costs a second clock to synchronise.

## Facts

| Item | Finding | Source |
| --- | --- | --- |
| `dex1_1_service` host | Install section: "at user development computing unit PC2 (NVIDIA Jetson Orin NX board)" | [README §1](https://github.com/unitreerobotics/dex1_1_service/blob/ced3216290bb22dae356213993786beeaa7143ab/README.md) |
| Motor library | `lib/libUnitreeMotorSDK_Arm64.so` (ELF aarch64) and `libUnitreeMotorSDK_Linux64.so` (x86-64); CMake picks the ARM one on `aarch64` | repo `lib/`, `CMakeLists.txt` (checked here with `file`) |
| Library needs | Highest symbol versions GLIBC_2.17, GLIBCXX_3.4.26, i.e. any current 64-bit Linux | `objdump -T`, checked here |
| Offline packages | `libserialport` .debs for arm64 and amd64, "(Ubuntu 22.04)" | README §1 note |
| `unitree_sdk2` (DDS) | "CPU (aarch64 and x86_64)", prebuild on Ubuntu 20.04; libraries for both in `thirdparty/lib` | [unitree_sdk2 README](https://github.com/unitreerobotics/unitree_sdk2) |
| Autostart | `setup_autostart.sh` installs a systemd service `dex1_gripper.service` | repo |
| DDS interface | `-n` sets the network interface (default `eth0`), so clients may sit on another machine | README §2 |
| Loop | one serial frame every 2 ms (500 Hz) per gripper | [[2026-11-03 Dex1-1 technical note]] |
| Ubuntu on Pi 5 | Pi 5 row: 22.04 blank; 24.04 and 26.04 server + desktop supported | [Ubuntu: Install on Raspberry Pi](https://ubuntu.com/hardware/docs/boards/how-to/ubuntu_supported/raspberry-pi) (updated 2026-07-10) |
| XRoboToolkit PC Service | "supports Windows x86, Linux x86, Linux aarch64"; ARM build instructions use an Orin (`/home/orin_pico/...`); headless .deb only for arm64 (checked 2026-11-09) | [PC-Service README](https://github.com/XR-Robotics/XRoboToolkit-PC-Service) |
| XRoboToolkit Python SDK | 1.1.0 wheels for `manylinux_2_34_aarch64` and `x86_64` | [PyPI](https://pypi.org/project/xrobotoolkit-sdk/1.1.0/) |
| Gripper power | 24–64 V DC, 3.5 A peak | [Unitree Dex1-1](https://unitree.com/Dex1-1) |
| Pi 5 video | no hardware H.264 encoder | [Raspberry Pi white paper](https://pip-assets.raspberrypi.com/categories/685-app-notes-guides-whitepapers/documents/RP-010033-WP-1-H.264%20encoding%20performance%20on%20Raspberry%20Pi%205_series%20computers.pdf) (already in proposal) |

## Prior work on handheld compute

- **ViTaMIn-B** (`li2025`): removed its onboard Raspberry Pi for a host PC, for "improved ergonomics and reduced weight" (already the basis of [[D06 Recording and synchronisation]]).
- **Grabette** (not in bib; [blog](https://huggingface.co/blog/grabette), 2026-07-21): "A Raspberry Pi" (model not stated) on the handheld records cameras and encoders "using a single shared clock", data "saved locally on the Raspberry Pi". No reason, mass or battery life given. Grabette has no motor to power.
- **ActiveUMI** (`zeng2025`), **HuMI** (`nai2026`): onboard or worn computers, built for mobile in-the-wild collection (per `docs/proposal.md`).
- **Unitree robots**: `dex1_1_service` runs on the robot's ARM64 PC2; user programs talk to it over DDS (README). This is the only precedent found for a separate small computer driving the Dex1-1.
- No prior work found that drives a Dex1-1 from a Raspberry Pi.

## Options

| Option | What runs where | For | Against | Precedent |
| --- | --- | --- | --- | --- |
| **A. Lab PC (current)** | PC: XRoboToolkit, `dex1_1_service`, RTDE, recording | One clock ([[D06 Recording and synchronisation]]); x86-64 build shipped by Unitree on the same Ubuntu 22.04 | Shared PC: sudo needs password, other students, services not started at boot | ViTaMIn-B host PC |
| **B. Pi as gripper computer** | Pi: only `dex1_1_service` (systemd). PC: everything else, sends commands over wired Ethernet DDS | Gripper independent of the shared PC; starts at boot; Unitree's own PC2 split | Second clock (needs chrony/PTP, residual unmeasured); Ubuntu 24.04 not Unitree's tested OS; one more box, supply and cable; DDS across machines unmeasured | Unitree PC2 (Jetson, not Pi) |
| **C. Pi replaces the PC** | Pi: XRoboToolkit, gripper, RTDE, recording | One small dedicated box | XRoboToolkit ARM build is for Orin / Ubuntu 22.04, Pi needs 24.04 (unverified); no hardware video encoder; training still needs the GPU PC | None found |
| **D. Pi on the handheld** | Pi carried on the handheld or belt | Untethered data link | 24 V power cable stays anyway (or a battery for 3.5 A peak); weight; extra clock; D06 already ranks it last | Grabette (no motor), ViTaMIn-B moved away |

**Reading (not a decision):** A stays the baseline. B is the only Pi option with a precedent and is worth it only if the shared PC is the actual problem. C and D have no precedent here and each conflict with a recorded reason in D06.

## What would need checking before B

1. Build `unitree_sdk2` + `dex1_1_service` on a Pi 5 with Ubuntu 24.04 (unverified).
2. Measure 500 Hz loop timing on the Pi, and command latency PC → Pi → gripper over DDS.
3. Clock sync PC ↔ Pi (chrony) and its residual against the ~10 ms target in D06.
4. Whether the company allows a separate Linux box on the lab network (unknown).

## Mobile collection without the lab PC (added 2026-11-12)

> **Dropped (Martyn, 2026-11-12):** mobile collection feasibility is not pursued; this and the AGX Orin section below are kept for reference only. The thesis plan already lists "Onboard computer build" as out of scope, so no plan change.

Motivation (Martyn, 2026-11-12): not a requirement; the question is whether handheld collection can be made mobile, not tied to a big PC, because that makes the solution more attractive.

Scope: only **handheld collection** needs to move. It uses the headset, the handheld (controller, Dex1-1, gripper camera) and a computer. VR teleop and deployment stay at the UR10e anyway.

### What the mobile computer must run

| Job | Pi 5 | Jetson Orin NX | Source |
| --- | --- | --- | --- |
| `dex1_1_service` (RS-485) | ARM64 lib fits; OS not Unitree's (Ubuntu 24.04 only on Pi 5); unverified | Unitree's own reference host | `dex1_1_service` README; [Ubuntu Pi page](https://ubuntu.com/hardware/docs/boards/how-to/ubuntu_supported/raspberry-pi) |
| XRoboToolkit PC Service (PICO link) | "Linux aarch64" supported, but built for Orin / Ubuntu 22.04; untested on Pi | ARM build instructions use an Orin | [PC-Service README](https://github.com/XR-Robotics/XRoboToolkit-PC-Service) |
| Video recording (gripper camera) | no hardware H.264 encoder | Orin NX: encoder listed in NVIDIA's spec table (table read with low confidence; unverified). Orin **Nano** has none: "does not have the NVENC engine" | [RPi white paper](https://pip-assets.raspberrypi.com/categories/685-app-notes-guides-whitepapers/documents/RP-010033-WP-1-H.264%20encoding%20performance%20on%20Raspberry%20Pi%205_series%20computers.pdf); [NVIDIA Orin Nano](https://docs.nvidia.com/jetson/archives/r36.3/DeveloperGuide/SD/Multimedia/SoftwareEncodeInOrinNano.html); [NVIDIA Jetson Orin](https://www.nvidia.com/en-us/autonomous-machines/embedded-systems/jetson-orin/) |
| Power draw | not checked | Orin NX "configurable between 10W and 25W" | [NVIDIA Jetson Orin](https://www.nvidia.com/en-us/autonomous-machines/embedded-systems/jetson-orin/) |

With everything on one worn computer there is still **one clock**, so the D06 single-clock argument holds.

### Power

- Dex1-1: 24–64 V, 0.03 A static, 3.5 A peak ([Unitree](https://unitree.com/Dex1-1)). Average draw during collection: unmeasured.
- A common 6S Li-ion/LiPo pack (about 3.0–4.2 V per cell, i.e. 18–25 V) falls below 24 V for most of its discharge, so a 7S or larger pack, or a boost converter to 24 V, would be needed (cell voltages are general Li-ion figures, unverified for a specific pack).
- Rough budget, unverified: Jetson up to 25 W plus the gripper average (unmeasured) plus losses; a 100 Wh pack would last a few hours. To be measured.

### Precedent

- **ActiveUMI** (`zeng2025`): closest match. VR controllers (Quest 3s) with a motorised gripper on the controller, plus "a portable backpack that holds a battery and a PC for self-contained operation". Computer model, battery size and runtime are not stated.
- **HuMI** (`nai2026`): "the entire setup fits into a single backpack"; compute not stated in the part read.
- **Grabette** (not in bib): Raspberry Pi on the handheld, no motor to power.
- **ViTaMIn-B** (`li2025`): went the other way, "removing onboard computing (e.g., Raspberry Pi) and interfacing all sensors directly with the host computer" for ergonomics and weight.
- No published system found that runs a Dex1-1 or XRoboToolkit from a worn computer.

### Answer

Mobile handheld collection looks **possible**, with a **backpack** computer rather than one on the handheld (ActiveUMI pattern; ViTaMIn-B's weight argument). The handheld keeps a short cable (USB + 24 V) to the backpack; the backpack holds the computer, a battery for the gripper, and the PICO's Wi-Fi link (a portable access point or the computer's own hotspot, unverified).

A **Jetson Orin NX** fits better than a Pi 5: it is the platform both Unitree (`dex1_1_service`) and XRoboToolkit build for, and NVIDIA lists a hardware video encoder for it. A Pi 5 would need an untested OS for both and has no hardware encoder; a Pi is only a fit for the gripper-computer option B. An Orin **Nano** lacks the encoder.

This matches the fallback already in [[D06 Recording and synchronisation]] ("onboard or backpack computer"). Note: `msc-thesis/docs/thesis-plan.md` lists "Onboard computer build" as **out of scope**, so building it would need a plan change. It is not a decision.

### What would prove it (no purchase made)

1. Run `dex1_1_service` and the XRoboToolkit service on an ARM64 Linux box (a Jetson, if the lab has one) with the PICO and Dex1-1; check the 500 Hz loop and trigger-to-gripper latency.
2. Measure the gripper's average current during a collection session to size the battery.
3. Record the gripper camera on that box at 30 Hz and check dropped frames.

## Lab Jetson AGX Orin: justification and time-boxed tests (added 2026-11-12)

Fact (Martyn, 2026-11-12): the lab has two **Jetson AGX Orin 64GB Developer Kits**, probably in use; using one needs the supervisor's OK.

### AGX Orin 64GB Developer Kit (NVIDIA)

| Property | Value | Source |
| --- | --- | --- |
| CPU | "12-core Arm Cortex-A78AE v8.2 64-bit", 2.2 GHz (ARM64, same architecture as Unitree's PC2) | [NVIDIA Jetson Orin](https://www.nvidia.com/en-us/autonomous-machines/embedded-systems/jetson-orin/) spec table |
| Video encode | up to "16x 1080p30 (H.265)" (column mapping of the extracted table inferred, verify) | same |
| Power | "configurable between 15W and 60W" | same |
| Size / mass | 110 × 110 × 71.65 mm; mass not stated | same |
| Power input | DC jack 5.5/2.5 mm, centre positive; voltage range not stated | [AGX Orin dev kit layout](https://docs.nvidia.com/jetson/agx-orin-devkit/user-guide/latest/hardware_layout.html) |
| USB | 4 × Type-A (2 × 3.2 Gen 2, 2 × 3.2 Gen 1), 2 × Type-C | same |
| OS | JetPack 7 is "Ubuntu 24.04"; JetPack 6 is Ubuntu 22.04 (general knowledge, not checked against an NVIDIA page; unverified). Version installed on the lab units: unknown | [JetPack](https://developer.nvidia.com/embedded/jetpack) |

### Why this unit and not a Pi 5 or another computer

| Criterion | AGX Orin (lab) | Raspberry Pi 5 | x86 laptop / mini PC with Ubuntu 22.04 |
| --- | --- | --- | --- |
| Cost / availability | already in the lab, no purchase | purchase via Storefront | purchase or borrow (the Mac cannot run XRoboToolkit) |
| Dex1-1 service | ARM64 lib; Unitree's reference host is also an Orin (NX) | ARM64 lib, but Pi 5 needs Ubuntu 24.04, untested by Unitree | x86-64 lib, same as the lab PC |
| XRoboToolkit service | ARM build instructions written on an Orin | untested OS | same .deb as the lab PC |
| Python SDK | wheel `manylinux_2_34_aarch64` needs glibc ≥ 2.34 (Ubuntu 22.04+) | fits | fits |
| Video encoder | yes (NVIDIA table) | no | depends on model |
| Power for battery use | 15–60 W, heaviest of the three | low (not checked) | depends |
| Precedent | Unitree PC2 (Orin NX) for the gripper | Grabette (no motor) | RDT2 onboard Intel PC; UMI-FT laptop tether |

Argument for the supervisor: the AGX Orin is the **only option that needs no purchase** and it is the **platform family both software stacks were built for** (Unitree's gripper service on an Orin NX, XRoboToolkit's ARM build on an Orin), with a hardware video encoder (minor point, see below). A Pi 5 would need a purchase and an OS neither stack was tested on. The AGX is larger and draws more than a final mobile unit needs. If the tests pass on it, the same JetPack software runs on a smaller Orin (NX/Nano) later (unverified for the specific versions). An x86 Ubuntu laptop would be the lowest software risk (same binaries as the lab PC) but must be bought or borrowed.

### How much the video encoder matters (2026-11-12)

Minor. The encoder only compresses camera video while recording. Uncompressed 1080p RGB at 30 Hz is 1920 × 1080 × 3 B × 30 ≈ 187 MB/s ≈ 670 GB/h (computed here), which a worn computer cannot store for long, so some compression is needed. It can come without a hardware encoder: many USB cameras deliver MJPEG frames that can be stored as-is (whether the Dex1-1 camera does: unverified), resolution can be reduced at capture, and the H.264/AV1 MP4 files of the LeRobot layout can be encoded afterwards on the lab PC. Grabette records on a Raspberry Pi, which has no hardware encoder. The encoder is therefore not a reason to prefer the AGX Orin over a Pi; the arguments that hold are no purchase and the Orin being the platform both software stacks were built for.

### Limiting the risk: gates with a time box

Principle: the **core path stays on the lab PC** and is never blocked by this side track. All recording code is written for Linux/Python on the lab PC first; it runs on the Jetson unchanged if the OS matches, so no work is lost if the mobile track is dropped. Each gate has a time box (estimates, unverified); if a gate fails inside its box, stop, record why, and stay on the lab PC. A failed gate is itself a reportable result ("mobile collection on X failed because Y").

| Gate | When | Test | Box | Stop if |
| --- | --- | --- | --- | --- |
| G0 | after supervisor OK, now | Check the unit: JetPack / Ubuntu version (`cat /etc/nv_tegra_release`, `lsb_release -a`), free disk, network, who else uses it | 1 h | Ubuntu older than 22.04 and reflashing not allowed (SDK needs glibc 2.34) |
| G1 | now, no gripper needed | Install XRoboToolkit ARM64 service + SDK; repeat the 2026-11-09 test (PICO trigger → Mac sim) | 1 day | PICO link does not work on the Jetson |
| G2 | after gripper delivery, after it runs on the lab PC | Build `unitree_sdk2` + `dex1_1_service`; trigger drives the real gripper; check the 500 Hz loop | 1 day | Service does not build or loop misses its rate |
| G3 | after the recorder works on the lab PC | 10 min recording: camera, controller pose, gripper at 30 Hz; count dropped frames | 1 day | Drops that the lab PC does not have |
| G4 | last | Measure power draw (Jetson + gripper) on the bench; size a battery; one walk-around demo | ½ day | Battery size not practical |

Total about 3.5 working days, spread over the thesis, with the first two decision points (G0, G1) before the gripper arrives.

Plan impact: `msc-thesis/docs/thesis-plan.md` lists "Onboard computer build" as out of scope. Moving a "mobile collection feasibility (G0–G4)" item to Stretch would be a plan change and needs Martyn's OK.

## Existing UR10e station: a Raspberry Pi already drives an RS-485 gripper (added 2026-11-12)

Facts (Martyn, 2026-11-12, observed at the station): the UR10e carries a Robotiq 2F-85 and an Intel RealSense D435i on its tool flange. The gripper cable gets 24 V from a supply under the station; its RS-485 data lines go to an RS-485-to-USB converter plugged into a **Raspberry Pi**. The D435i is also on the Pi's USB. This setup was used to collect VR-teleop data and train a π0.5 policy.

### Robotiq 2F-85 vs Dex1-1 on the serial link

| | Robotiq 2F-85 | Dex1-1 |
| --- | --- | --- |
| Bus | Modbus RTU over RS-485 ("24V DC supply and Modbus RTU communication over RS-485", manual §1.4) | Unitree motor frames over RS-485, 6 Mbit/s |
| Baud / rate | 115 200 bit/s; "200 Hz is the usual speed" (manual §4.7.1; read from a summary of the PDF, verify) | 500 Hz service loop (`dex1_1_service` source) |
| Software | open register protocol; any Modbus library | closed motor library (x86-64 / ARM64 .so) + `unitree_sdk2` DDS |
| Supply | 24 V, max 1.7 A ([Unchained Robotics](https://unchainedrobotics.de/produkte/end-of-arm-effektor/greifer/finger-greifer/robotiq-2f-85)) | 24–64 V, 3.5 A peak ([Unitree](https://unitree.com/Dex1-1)) |

Manual: [Robotiq 2F-85/2F-140 e-Series manual](https://assets.robotiq.com/website-assets/support_documents/document/2F-85_2F-140_Instruction_Manual_e-Series_PDF_20190206.pdf) (2018 revision).

### What this means

- **In-house precedent for option B.** The station already uses a Pi as a gripper-and-camera node next to the robot, with VR teleop elsewhere. Swapping the Robotiq for the Dex1-1 on the same pattern keeps the station architecture the existing π0.5 pipeline was built on.
- **The Dex1-1 link is heavier than the Robotiq one** (6 Mbit/s vs 115.2 kbit/s, 500 Hz vs ~200 Hz, closed ARM64 library instead of open Modbus). Whether the station Pi runs it depends on its model and OS: the ARM64 library needs a 64-bit OS; the Pi 5 needs Ubuntu 24.04+ or Raspberry Pi OS (both untested by Unitree). Unverified until the Pi is inspected.
- **Comparison fairness.** If the VR-teleop and handheld conditions use the same gripper node, the gripper path is the same in both datasets.

### To find out on the station Pi (read-only)

1. Model and OS: `cat /proc/device-tree/model`, `cat /etc/os-release`, `uname -m` (needs `aarch64`).
2. What runs on it (ROS 2 nodes or custom scripts), how it talks to the teleop PC (network, topics), who maintains it.
3. Whether it may be reused or a second Pi of the same build is preferred (the existing VR-teleop setup must not break).

These overlap the intern questions in `Meetings/2026-11-03 Intern VR teleop.md` (software stack, gripper command, data path).

### Station Pi inspected (Martyn, 2026-11-12, read-only over SSH)

| Item | Result |
| --- | --- |
| Model | Raspberry Pi 5 Model B Rev 1.0, 2 GB RAM (1.9 GiB total, 1.2 GiB available) |
| CPU / OS | `aarch64`; Ubuntu 25.10 desktop (GNOME); 57 GB SD card, 24 % used |
| Serial | `/dev/ttyUSB0` = FTDI `0403:6015` USB-UART bridge (the Robotiq converter, inferred) |
| Camera | RealSense D435i (`8086:0b3a`) on a USB 3 bus |
| Services | `ser2net` ("Serial port to network proxy"), `chrony` (NTP); no ROS in `/opt/ros`; no gripper or camera program running at the time |
| Home folder | `realsense_pi_setup/`, `wrist/` (contents not read) |

Confirmed (`/etc/ser2net.yaml`): TCP port 4001 ↔ `/dev/serial/by-id/usb-FTDI_USB_TO_RS-485_…`, 115200 8N1, `kickolduser: true` (a new client replaces the old one). Camera folders: `realsense_pi_setup/` (`README.md`, `config.env`, `install_pi.sh`, `start_stream.sh`, `stream_camera_sync.py`, `measure_camera_delay.py`, `measure_v4l2_direct.py`, `camera_logs/`) and `wrist/` (Python venv with `realsense_streamer.py`, `stamp_check.py`, `fov_check.py`, `realsense_wb_calibrate.py`); contents not read. The file names suggest the previous setup measured camera delay and timestamp alignment, which is relevant to [[D06 Recording and synchronisation]] (unverified until read).

Reading (inferred, not confirmed with whoever built it): the Pi is a **thin bridge**. `ser2net` exposes the Robotiq serial port over the network, so the gripper logic (Modbus RTU) runs on another computer; the camera is presumably streamed by the scripts in `realsense_pi_setup/` or `wrist/`. Chrony suggests its clock is synced over NTP.

Consequences for the Dex1-1:
- **ser2net does not carry over.** Unitree's closed library opens a local `/dev/tty*` and exchanges a frame every 2 ms at 6 Mbit/s; tunnelling that through the network would put network delay inside the 500 Hz loop (likely to break it; unverified). The Dex1-1 would need `dex1_1_service` running on the Pi itself, with commands over DDS (option B).
- **Architecture OK, OS untested:** `aarch64` matches the ARM64 library; Ubuntu 25.10 is not Unitree's tested OS (22.04). Ubuntu interim releases get 9 months of support, so 25.10 is likely past end of life (general Ubuntu release policy, not checked for this release; unverified). `unattended-upgrades` is active.
- **2 GB RAM** with a desktop session leaves little room; whether it is enough for building and running `unitree_sdk2` is unverified.

### Station Pi camera and clock setup (read 2026-11-12, read-only)

- **Older pipeline (`realsense_pi_setup/`):** D435i colour over V4L2, H.264 encoded on the Pi with libx264 (the hardware path `h264_v4l2m2m` was avoided because of green frames), sent over UDP to 10.36.1.100:5002; frame times (V4L2 buffer time, Pi clock) over UDP :5003 and to CSV. README cites ~43 ms delivery latency (not a sensor offset) and warns frame IDs and video are not guaranteed to match. config.env says 640×480 @ 60 fps, README says 30 fps (mismatch).
- **Newer pipeline (`wrist/`):** pyrealsense2, 848×480 @ 30 fps, JPEG q90 over ZMQ PUB `tcp://*:5555`, each message = [epoch ns, JPEG] using librealsense global-time capture stamp. A `republisher.py` on an **Orin** turns it into a ROS 2 topic (per docstring). `stamp_check.py` measures stamp vs exposure middle; no stored results found.
- **Clock:** chrony syncs to 10.36.1.108 (stratum 10), RMS offset 1.18 ms. Whether .108 is the Orin: unverified.

Relevance: the existing teleop pipeline already uses a Jetson Orin on the robot side and solved camera timestamping (capture-time stamps + chrony), a method the handheld recorder can reuse ([[D06 Recording and synchronisation]]).

## Handheld side: two layouts (added 2026-11-12)

The station Pi serves the robot side. On the handheld side (real path, [[D01 Handheld gripper body]]) the operator holds the PICO controller in the sleeve with a Dex1-1 and its camera. Signals: controller pose and trigger reach the XRoboToolkit service on the PC over Wi-Fi; the Dex1-1 cable carries 24 V and RS-485; the camera has its own USB cable. Only one gripper is driven per phase (2026-11-04).

| Layout | Handheld cables go to | Gripper command path | Clocks | Source / precedent |
| --- | --- | --- | --- | --- |
| H1. Straight to the lab PC | RS-485 adapter and camera on the PC's USB; 24 V from a bench supply | PC: trigger → `dex1_1_service` on the same PC | one (PC) | [[D06 Recording and synchronisation]] first option; ViTaMIn-B (`li2025`) host PC |
| H2. Mirror of the robot station | RS-485 adapter and camera on a Pi (same build as the station Pi) at the collection table; 24 V from a bench supply | PC (XRoboToolkit) → network → Pi → gripper | two (PC, Pi), as on the robot side today | existing station (Martyn, 2026-11-12); Unitree PC2 split |

H1 is the simpler and already-recorded first choice; H2 is worth it only if the station Pi software turns out reusable, so both conditions run the same gripper node. Cable length from handheld to PC or Pi: D06 suggests 3–5 m over a balancer (unmeasured for this lab).

## Station layout confirmed (Martyn, 2026-11-12)

fractal-pc is physically far from the robot station, and handheld collection happens at the station, so layout H1 (handheld straight into fractal-pc) is dropped. The station has a wired network switch. The existing VR-teleop setup (built by the supervisor): the **XRoboToolkit service runs on a Jetson Orin**; the **station Pi** is the I/O node the gripper and camera connect to (ser2net, ZMQ camera stream, chrony).

Consequence: the handheld can mirror this proven split. Jetson = brain (XRoboToolkit, gripper commands, recording); a station computer = gripper and camera I/O. Because the Dex1-1 loop cannot go through ser2net, `dex1_1_service` must run on whichever computer the RS-485 adapter is plugged into: the Jetson (if at the station and reachable by cable; Unitree's reference platform) or a second Pi 5 (untested). Using the same Jetson stack for handheld and VR teleop also keeps both conditions on one pipeline.

### Correction (Martyn, 2026-11-12, 15:44)

Both fractal-pc and the teleop Jetson are far from the station. The only computer at the station is the Pi. So `dex1_1_service` must run on a **station computer that does not exist yet for the Dex1-1**: a second Pi 5 (the existing one stays untouched) or a lab AGX Orin moved to the station. The Jetson keeps XRoboToolkit and sends gripper commands over the wired network (DDS), exactly like the Robotiq setup but with the gripper loop local on the station computer.

Cheapest decisive test (before the gripper arrives): build `unitree_sdk2` + `dex1_1_service` on a Pi 5 with Ubuntu 24.04 and start the service (it should run and report no motor found). Pass → Pi 5 as the station computer. Fail → AGX Orin at the station (Unitree's reference platform). Both are then checked with the gripper (500 Hz loop, commands from the Jetson over the network).

### Martyn's proposed layout and the needed change (2026-11-12, 15:53)

Proposal: handheld Dex1-1 and camera to the station Pi; Dex1-1 powered from the supply under the station; Pi relays to fractal-pc, which runs XRoboToolkit and the PICO. Works for the camera as-is. For the gripper the Pi must **run `dex1_1_service` itself** instead of relaying (ser2net), because the 2 ms request/response loop cannot cross the network (inference from the source, unmeasured); fractal-pc then sends commands at 30–90 Hz over DDS and reads state. Monday questions: (1) existing Pi or a second one (2 GB, Ubuntu 25.10, runs the teleop Robotiq); (2) supply rating: sized for the Robotiq (24 V, max 1.7 A per retailer) vs Dex1-1 24–64 V, 3.5 A peak; (3) handheld on fractal-pc or on the supervisor's Jetson.

## Open

- Where the Jetson stands (at the station?), whether it is one of the two AGX Orin 64GB units, and whether it may be shared (supervisor, Monday).
- Station Pi model, OS and software (above); access to it.
- Company acceptance of a battery pack and a portable access point on the lab network (unknown).
