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
5. **Most defensible use of a Pi:** a dedicated "gripper box" that runs only `dex1_1_service` and starts it at boot, talking to the PC over Ethernet. This copies Unitree's own split (service on PC2, user code elsewhere over DDS). It costs a second clock to synchronise.

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
| **B. Pi as gripper box** | Pi: only `dex1_1_service` (systemd). PC: everything else, sends commands over wired Ethernet DDS | Gripper independent of the shared PC; starts at boot; Unitree's own PC2 split | Second clock (needs chrony/PTP, residual unmeasured); Ubuntu 24.04 not Unitree's tested OS; one more box, supply and cable; DDS across machines unmeasured | Unitree PC2 (Jetson, not Pi) |
| **C. Pi replaces the PC** | Pi: XRoboToolkit, gripper, RTDE, recording | One small dedicated box | XRoboToolkit ARM build is for Orin / Ubuntu 22.04, Pi needs 24.04 (unverified); no hardware video encoder; training still needs the GPU PC | None found |
| **D. Pi on the handheld** | Pi carried on the handheld or belt | Untethered data link | 24 V power cable stays anyway (or a battery for 3.5 A peak); weight; extra clock; D06 already ranks it last | Grabette (no motor), ViTaMIn-B moved away |

**Reading (not a decision):** A stays the baseline. B is the only Pi option with a precedent and is worth it only if the shared PC is the actual problem. C and D have no precedent here and each conflict with a recorded reason in D06.

## What would need checking before B

1. Build `unitree_sdk2` + `dex1_1_service` on a Pi 5 with Ubuntu 24.04 (unverified).
2. Measure 500 Hz loop timing on the Pi, and command latency PC → Pi → gripper over DDS.
3. Clock sync PC ↔ Pi (chrony) and its residual against the ~10 ms target in D06.
4. Whether the company allows a separate Linux box on the lab network (unknown).

## Open

- What drives the question (Martyn): independence from the shared PC, portability on the handheld, or a setup for the company later. The answer picks between A and B.
