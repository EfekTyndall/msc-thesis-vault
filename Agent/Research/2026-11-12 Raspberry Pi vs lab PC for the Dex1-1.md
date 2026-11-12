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

## Mobile collection without the lab PC (added 2026-11-12)

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

A **Jetson Orin NX** fits better than a Pi 5: it is the platform both Unitree (`dex1_1_service`) and XRoboToolkit build for, and NVIDIA lists a hardware video encoder for it. A Pi 5 would need an untested OS for both and has no hardware encoder; a Pi is only a fit for the gripper-box option B. An Orin **Nano** lacks the encoder.

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

Argument for the supervisor: the AGX Orin is the **only option that needs no purchase** and it is the **platform family both software stacks were built for** (Unitree's gripper service on an Orin NX, XRoboToolkit's ARM build on an Orin), with a hardware video encoder. A Pi 5 would need a purchase and an OS neither stack was tested on. The AGX is larger and draws more than a final mobile unit needs. If the tests pass on it, the same JetPack software runs on a smaller Orin (NX/Nano) later (unverified for the specific versions). An x86 Ubuntu laptop would be the lowest software risk (same binaries as the lab PC) but must be bought or borrowed.

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

## Open

- Supervisor OK to use one lab AGX Orin; whether the units are in use and may be reflashed.
- Company acceptance of a battery pack and a portable access point on the lab network (unknown).
