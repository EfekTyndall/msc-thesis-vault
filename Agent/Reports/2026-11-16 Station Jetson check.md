---
date: 2026-11-16
tags: [report, dex1-1, compute, station]
---
# Station Jetson check

Read-only check of the Jetson approved as the handheld's station computer (Path 2, [[D06 Recording and synchronisation]], [[2026-11-16 Yanmei]]), Jetson 10.36.1.124. Martyn ran one command block over SSH from the Mac on 2026-11-16 at 20:13 CET, logged in with the intern's account; items 3 and 4 also from his own look at the unit. Nothing was installed or changed. Builds on [[2026-11-13 Handheld computer and gripper connection]] and [[2026-11-13 dex1_1_service code and Pi 5 fit]].

## Result

The board fits: AGX Orin, L4T R36, Ubuntu 22.04, aarch64, GLIBCXX up to 3.4.30. This is the same family and OS Unitree names for `dex1_1_service` ("PC2 (NVIDIA Jetson Orin NX board)", offline `.deb` for arm64 22.04; [[2026-11-13 dex1_1_service code and Pi 5 fit]]). Two blockers before use: the unit stands far from the UR10e station, and there is no own account yet.

| # | Item | Measured | Verdict |
|---|------|----------|---------|
| 1 | Model, L4T, OS, arch | `NVIDIA Jetson AGX Orin Developer Kit`; `R36 (release), REVISION: 5.2`; Ubuntu 22.04.5 LTS; kernel 5.15.199-tegra aarch64 | **Pass.** R36 = JetPack 6 series (exact JetPack release for R36.5.2 unverified). Not R32. |
| 2 | C/C++ libraries | GLIBC 2.35; libstdc++ has GLIBCXX up to 3.4.30 | **Pass.** Needs GLIBC ≥ 2.17, GLIBCXX ≥ 3.4.26 (the highest version listed implies the lower ones). |
| 3 | Location | Far from the UR10e station (Martyn) | **Fail for now.** Martyn can unplug and move it. Needs a wired port on the station switch at the new place; whether the address stays the same after the move is unknown. |
| 4 | Free USB ports | `lsusb` shows only the board's internal USB 2.0/3.0 hubs and Bluetooth; no `/dev/ttyUSB*`, `/dev/ttyACM*`, `/dev/video*`; Martyn: nothing plugged in | **Pass.** All external ports free; number of ports not counted (unverified). |
| 5 | Users, load, RAM, disk | One human account (the intern's); 2 SSH sessions, one open since 2026-10-27 from another host, one Martyn's; up 21 days, load 0.08 on 8 cores; RAM 61 GiB (59 GiB available), swap 30 GiB; root on eMMC 57 GB, 24 GB used, 31 GB free | **Pass** for the software. Open: the session since 2026-10-27 means someone else still uses the board; 31 GB free is small for camera recordings (size per episode unmeasured). |
| 6 | Own account and groups | Not created. Intern account has `sudo` and `video`, but **not** `dialout` or `docker` | **Fail (open).** Own account needs `dialout` (RS-485 adapter), `video` (camera), `sudo`; `docker` if containers are used. Who creates it (intern or the board's owner) unknown. |
| 7 | Existing software | ROS 2 Humble in `/opt/ros/humble` (not sourced in the shell, `ros2` not on the path); Docker CE 29.9.2 with compose and buildx; no `/opt/apps` (no XRoboToolkit PC Service, no `roboticsservice` package) | **Info.** XRoboToolkit must be installed; its PC Service lists Linux aarch64 and a headless arm64 `.deb` exists (XR-Robotics/XRoboToolkit-PC-Service README, read 2026-11-09). `nvidia-jetpack` metapackage not listed by `dpkg`. |
| 8 | Time sync | `systemd-timesyncd` active, `System clock synchronized: yes`, NTP active, RTC in UTC, time zone Europe/Berlin; `chrony` inactive | **Pass.** NTP server not checked. Per `msc-thesis/CLAUDE.md` the recording PC's clock is the reference, which this Jetson now is. |

## Open

1. Move the Jetson to the UR10e station and plug it into the station switch; check it is reachable afterwards. Ask first whether the user with the session open since 2026-10-27 still needs it where it stands.
2. Own account with `sudo`, `dialout`, `video` (and `docker` if used); created by the intern or the board's owner.
3. Storage for recordings: 31 GB free on eMMC; measure recording size per episode, then decide (external drive or copy off after each session).
4. Install plan (XRoboToolkit PC Service arm64, `dex1_1_service` dependencies, `unitree_sdk2`) shown to Martyn before anything is installed (new dependencies).
