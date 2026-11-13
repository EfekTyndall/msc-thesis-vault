---
date: 2026-11-13
tags: [research, dex1-1, compute, software]
---
# dex1_1_service code and Pi 5 fit

Questions (Martyn, 2026-11-13): (1) Does `dex1_1_service` also handle the Dex1-1 camera, and is there Unitree software for that camera? (2) Can a Raspberry Pi run `dex1_1_service`, given the stated need for a 64-bit OS and Ubuntu 22.04 or 24.04? Follows [[2026-11-12 Raspberry Pi vs lab PC for the Dex1-1]] and [[2026-11-12 Lab computers and gripper connections]].

Method: source read of [`dex1_1_service` @ ced3216](https://github.com/unitreerobotics/dex1_1_service/tree/ced3216290bb22dae356213993786beeaa7143ab) (2026-09-16), [`unitree_sdk2` @ aaf69cb](https://github.com/unitreerobotics/unitree_sdk2/tree/aaf69cb) (2026-11-09) and [`teleimager` @ 266b393](https://github.com/unitreerobotics/teleimager/tree/266b393) (2026-11-13); prebuilt libraries inspected with `file`/`objdump`; Ubuntu and Raspberry Pi download pages. Nothing was built or run on a Pi.

## Answers

1. **`dex1_1_service` is gripper only, no camera.** Its source has no camera, video or image code. Unitree's camera program is a separate one, **teleimager**.
2. **A Pi 5 can very likely run `dex1_1_service` (untested).** All prebuilt parts exist for aarch64 and need only an old C library. Ubuntu 24.04 LTS (and 26.04 LTS) officially supports the Pi 5; **22.04 does not**. The station Pi already runs Ubuntu 25.10 aarch64.

## What `dex1_1_service` does (code)

- `main.cpp` (240 lines): scans `/dev/ttyUSB*`, `/dev/ttyCH343USB*` and `/dev/ttyACM*` (comment: "support unitree H2"), probes motor IDs 0 and 1 on each port, and starts one control thread per motor found.
- Control thread: `RecurrentThread(2000, …)`, i.e. one serial send-and-receive every 2000 µs (500 Hz) per motor via `serial_->sendRecv(&cmd_, &state_)`.
- If no command arrived recently (`sub_->isTimeout()`), the motor is put in **BRAKE** mode with zero gains. Otherwise FOC mode with `kp`, `kd`, `q`, `dq`, `tau` from the command, scaled by the gear ratio.
- DDS topics (via `unitree_sdk2`, Cyclone DDS): `rt/dex1/right/cmd` + `/state` for motor ID 0, `rt/dex1/left/cmd` + `/state` for ID 1. State carries `q`, `dq`, `tau_est`. Network interface flag `-n` (default `eth0`).
- `-c` runs a calibration: close the gripper by hand, press `s`; sets the zero at 322° motor angle.
- `setup_autostart.sh` installs a systemd service (`dex1_gripper.service`, user `unitree`, restart on failure).
- Motor protocol: closed-source `lib/libUnitreeMotorSDK_Arm64.so` / `_Linux64.so` (M4010 motor). Serial settings are inside it, so the 6 Mbit/s rate cannot be read from the open code.

## Can a Pi 5 run it?

| Requirement | Finding | Source |
| --- | --- | --- |
| CPU architecture | Motor SDK ships `libUnitreeMotorSDK_Arm64.so` ("ELF 64-bit … ARM aarch64"); CMake picks it when the host is `aarch64` | `CMakeLists.txt`, `file lib/*.so` |
| C/C++ library versions | Motor SDK needs GLIBC ≥ 2.17 and GLIBCXX ≥ 3.4.26; Cyclone DDS libs the same | `objdump -T` on the `.so` files |
| `unitree_sdk2` | Prebuilt `lib/aarch64/libunitree_sdk2.a` and aarch64 DDS libs; built on "Ubuntu 20.04 LTS", "gcc version 9.4.0" | [unitree_sdk2 README](https://github.com/unitreerobotics/unitree_sdk2) |
| Other packages | `libserialport`, `spdlog`, `boost`, `yaml-cpp`, `fmt` from apt; offline `.deb` given for arm64 22.04 | `dex1_1_service` README |
| Unitree's tested platform | "PC2 (NVIDIA Jetson Orin NX board)" | `dex1_1_service` README |
| Ubuntu on Pi 5 | 22.04 LTS: no Pi 5 entry. 24.04 LTS: Server + Desktop supported and certified (Desktop needs ≥ 4 GB RAM). 26.04 LTS: supported and certified | [Ubuntu Raspberry Pi support matrix](https://ubuntu.com/hardware/docs/boards/how-to/ubuntu_supported/raspberry-pi/), [download page](https://ubuntu.com/download/raspberry-pi) |
| Raspberry Pi OS | 64-bit for Pi 5, Debian 13 (trixie); legacy Debian 12 (bookworm) | [raspberrypi.com OS page](https://www.raspberrypi.com/software/operating-systems/) |

Reading: every newer Ubuntu (22.04 has GLIBC 2.35, 24.04 has 2.39; standard Ubuntu versions, not checked here) satisfies GLIBC ≥ 2.17 and GLIBCXX ≥ 3.4.26, so the "22.04 or 24.04" requirement reflects what Unitree tested, not a hard limit of the binaries. On a Pi 5 the practical choice is **Ubuntu 24.04 LTS (64-bit)**. Raspberry Pi OS 64-bit would probably also work but is further from what Unitree tested (unverified).

Open risks (not tested):
- Building `unitree_sdk2` / linking the gcc-9 static library with a newer gcc (13 on 24.04): usually fine, untested.
- 500 Hz timing on a Pi 5 under load (camera streaming at the same time): measure on the gripper.
- Serial driver for Unitree's RS-485-to-USB board on the Pi kernel: chip unknown until the box is opened (`ttyUSB` / `ttyCH343USB` / `ttyACM` all supported by the code).
- Existing station Pi: 2 GB RAM, Ubuntu 25.10 (non-LTS, not on Unitree's list), serves the supervisor's Robotiq. A second Pi 5 with 24.04 LTS is cleaner (Monday question).

## Camera: teleimager

- Unitree's image server for [xr_teleoperate](https://github.com/unitreerobotics/xr_teleoperate): captures UVC, V4L2, GStreamer and RealSense cameras and publishes over **ZeroMQ** (LAN) or **WebRTC** (browser). Server runs on "the robot/device the cameras are plugged into".
- JPEG via PyTurboJPEG (needs libjpeg-turbo ≥ 3.0); Python 3.10–3.12; `Operating System :: POSIX :: Linux`; conda instructions mention aarch64/Jetson. Not tested on a Pi by Unitree (no mention).
- Same pattern as the station Pi's own ZMQ JPEG wrist stream ([[2026-11-12 Lab computers and gripper connections]]).
- It does **not** mention the Dex1-1 camera. Whether that camera is a plain UVC camera (then teleimager's `--uvc` scan would list it) is unverified until the gripper arrives.

## Consequence for the station layout

The station computer for the handheld runs two independent programs: `dex1_1_service` (gripper, 500 Hz local loop, DDS to fractal-pc) and a camera streamer (teleimager, or the supervisor's existing ZMQ script). Proposed first test, before delivery: on a Pi 5 with Ubuntu 24.04 LTS, build `unitree_sdk2` + `dex1_1_service` and start it; expected output "No ttyUSB serial ports found." Install teleimager and stream any USB webcam to fractal-pc.
