---
date: 2026-11-17
tags: [report, compute, jetson, software, install-plan]
---
# Station Jetson: setup results and install plan

Station computer for the handheld (Path 2, [[D06 Recording and synchronisation]]): Jetson AGX Orin, L4T R36.5.2, Ubuntu 22.04.5 aarch64. Follows [[2026-11-16 Station Jetson check]]. Steps 1–5 of the checklist in [[2026-11-17]] were run by Martyn on 2026-11-17 with Claude guiding; outputs pasted into the project thread or read over `ssh station-jetson` (read-only). Part B is the install plan for step 6: **nothing in Part B is installed until Martyn's go.**

## A. Done on 2026-11-17

| # | Item | Result | How checked |
| --- | --- | --- | --- |
| 1 | Other users | The only other session (since 2026-10-27, from 10.36.1.100) was the intern's own `saad` login, idle 17 days. Only system processes running. Intern OK'd moving and working on the board. | `who`, `w`, `ps aux` |
| 2 | Move | Clean `sudo shutdown -h now`, moved to the UR10e station, wired to the station switch. | — |
| 3 | Network | Address unchanged: **10.36.1.124** on wired port **`eno1`** (MAC 48:b0:2d:e6:4b:58), router 10.36.1.1 (same as the Mac). **The wired port is `eno1`, not `eth0`**: `dex1_1_service` must be started with `--network eno1`. | `ip -br addr`, `ip route`, `ping` from the Mac |
| 3 | University Wi-Fi | After the reboot the Wi-Fi had joined `uni-stuttgart-open` (DNS 129.69.252.252). Plain http then hit `guest-internet.tik.uni-stuttgart.de` and `apt update` failed ("Clearsigned file isn't valid"). Over the cable, http and `apt update` work. Fix: `nmcli connection modify uni-stuttgart-open connection.autoconnect no` (intern does not need it). Only `Wired connection 1` autoconnects now; Wi-Fi is down. | `curl --interface eno1`, `apt update`, `nmcli` |
| 3 | Clock | NTP is blocked for the whole lab network: `pool.ntp.org` gets no answer from the Jetson or the Mac (`sntp`), and the router 10.36.1.1 serves no NTP. The Jetson's RTC reads 1970 (not kept while off), so after each power-up it resumed from its last shutdown time (≈3.6 min behind after the move). Fix: **`htpdate`** installed (Ubuntu package 1.2.6, Martyn's go): reads the HTTP `Date` header of web servers; with the Ubuntu default `-D -s` it steps the clock once at start-up and afterwards only slews it (`adjtime`), so no jumps during recordings (source: `htpdate.c` `setclock()` and main loop). Result: Jetson within ≈2 s of the Mac (the gap between the two `date` calls). `timedatectl` still says "synchronized: no"; that flag belongs to `systemd-timesyncd`, not htpdate. | `date` on both, `systemctl is-active htpdate` |
| 4 | Own account | User **`martyn`** (uid 1001) with groups `sudo`, `dialout`, `video` (plus Jetson defaults `audio`, `render`, `i2c`, `gpio`, …). Key login from the Mac: `ssh station-jetson` (key `~/.ssh/id_ed25519_dtag_station-jetson`, alias in `~/.ssh/config`). `sudo` asks for the password. | `id`, `ssh station-jetson hostname` |
| 5 | Disk | `/` and `/home` on the 64 GB eMMC (`mmcblk0p1`): 57 GB, **31 GB free**. A **256 GB SD card** (`mmcblk1`, ext4, label `sdcard`, model SN256) is inserted but not mounted; `/media/saad` exists, so it may be the intern's (ask before use). No NVMe drive. | `df -h`, `lsblk` |
| — | Firewall | `ufw` inactive: unlike fractal-pc, no port rule is needed for the headset (TCP 63901). | `systemctl is-active ufw` |
| — | Present already | git 2.34, cmake 3.22, g++ 11.2, build-essential, python3 3.10.12 with `python3-venv` and `pip`, libyaml-cpp-dev, libeigen3-dev, libboost-dev 1.74, libfmt-dev, libspdlog-dev, libjpeg-turbo8 **2.1.2**, ROS 2 (`/opt/ros`), Docker. No conda. GitHub SSH (port 22) reachable. | `dpkg-query`, `ssh -T git@github.com` |
| — | Missing | `libboost-all-dev`, `libserialport-dev`; nothing in `/opt/unitree_robotics` or `/opt/apps/roboticsservice`. | `dpkg-query`, `ls /opt` |

Not done on purpose: the 544 pending Ubuntu updates (`apt upgrade`). The board is shared and runs NVIDIA's JetPack packages; a full upgrade is not needed for Path 2 and is left to the board's owner.

## B. Install plan (step 6, for Martyn's go)

Rules for all items: own account `martyn`; source code in `~/src/`; nothing autostarts; each item can be removed again (line "Undo"). `apt` shows the download size and asks before installing; answer `n` if it is unexpectedly large. Order = what is needed first for the trigger test, then the gripper.

### B1. XRoboToolkit PC Service (headset → Jetson)

- What: release v1.0.0 has two arm64 packages: `XRoboToolkit-PC-Service_1.0.0.0_arm64.deb` (54.2 MB) and `XRoboToolkit-PC-Service-headless_1.0.0.0_arm64.deb` (38.5 MB) ([release v1.0.0](https://github.com/XR-Robotics/XRoboToolkit-PC-Service/releases/tag/v1.0.0), listed 2026-11-17). This closes the open question from [[2026-11-16 Station Jetson check]]: a prebuilt arm64 package exists.
- Choice: **headless**, since the Jetson is used over SSH only. Whether the headless package contains `runService.sh` like the amd64 one is **unverified**; checked with `dpkg -c` before installing. Fallback: the GUI package.
- Commands:
  ```
  mkdir -p ~/Downloads && cd ~/Downloads
  wget https://github.com/XR-Robotics/XRoboToolkit-PC-Service/releases/download/v1.0.0/XRoboToolkit-PC-Service-headless_1.0.0.0_arm64.deb
  sha256sum XRoboToolkit-PC-Service-headless_1.0.0.0_arm64.deb   # record the hash (no published hash)
  dpkg -c XRoboToolkit-PC-Service-headless_1.0.0.0_arm64.deb | grep -E 'runService|RoboticsServiceProcess'
  sudo apt install ./XRoboToolkit-PC-Service-headless_1.0.0.0_arm64.deb
  ```
- Firewall: none needed (ufw inactive).
- Check: start the service, headset connects to `10.36.1.124`, status WORKING (as `msc-thesis/docs/setup/pico-trigger-sim.md` §3).
- Undo: `sudo apt remove <package name from dpkg -c / apt>`.

### B2. XRoboToolkit Python SDK (Jetson reads the trigger)

- What: `xrobotoolkit-sdk==1.1.0`, which has a `manylinux_2_34_aarch64` wheel for Python ≥ 3.10 ([PyPI files](https://pypi.org/project/xrobotoolkit-sdk/1.1.0/#files), checked 2026-11-16); GLIBC 2.35 ≥ 2.34. In a Python virtual environment (no conda on the Jetson; `python3-venv` is present), so nothing is added to the system Python.
- Commands:
  ```
  python3 -m venv ~/venvs/xrobotoolkit
  ~/venvs/xrobotoolkit/bin/pip install xrobotoolkit-sdk==1.1.0
  ```
- Check: `trigger_udp.py --host 10.36.1.125` with the headset; the Mac sim follows the trigger (same test as fractal-pc on 2026-11-09).
- Undo: `rm -rf ~/venvs/xrobotoolkit`.

### B3. `dex1-1-handheld` clone (our code on the Jetson)

- What: read-only GitHub **deploy key** for this board, as on fractal-pc (`pico-trigger-sim.md` §2 step 4). A new key per machine, not a copy of fractal-pc's private key.
- Commands (Jetson): `ssh-keygen -t ed25519 -f ~/.ssh/dex1_1_handheld_deploy -N ""`, then `cat ~/.ssh/dex1_1_handheld_deploy.pub`. **Martyn** adds it on GitHub (repo `dex1-1-handheld` → Settings → Deploy keys, read-only). Then an ssh alias `github-dex1` and `git clone git@github-dex1:EfekTyndall/dex1-1-handheld.git ~/dex1-1-handheld`.
- Check: `git -C ~/dex1-1-handheld pull`.
- Undo: delete the deploy key on GitHub and `rm -rf ~/dex1-1-handheld`.

### B4. `dex1_1_service` (gripper driver; start test without a gripper)

- What: the two missing apt packages from its README, then a build in `~/src` ([dex1_1_service README](https://github.com/unitreerobotics/dex1_1_service)). The README's tested platform is a Jetson Orin NX ("PC2").
- Commands:
  ```
  sudo apt install libserialport-dev libboost-all-dev
  mkdir -p ~/src && cd ~/src
  git clone https://github.com/unitreerobotics/dex1_1_service
  cd dex1_1_service && mkdir build && cd build && cmake .. && make -j6
  sudo ./dex1_1_gripper_server --network eno1
  ```
- Expected without a gripper: "No ttyUSB serial ports found." ([[2026-11-13 dex1_1_service code and Pi 5 fit]]). Stop with Ctrl+C.
- `libboost-all-dev` pulls all Boost development packages (size shown by apt; unverified). Leaner alternative: run `cmake ..` first and install only what it reports missing. The README way is proposed because it is what Unitree documents.
- Not now: `setup_autostart.sh` (it installs a systemd service for user `unitree` and defaults to `eth0`); only after the gripper works, adapted to `martyn` and `eno1`.
- Undo: `rm -rf ~/src/dex1_1_service`; `sudo apt remove libserialport-dev libboost-all-dev` (then `sudo apt autoremove`).

### B5. `unitree_sdk2` (needed for our gripper control program, item 8)

- What: Unitree's DDS SDK; prebuilt aarch64 libraries, build deps already present (cmake, g++, libyaml-cpp-dev, libeigen3-dev, libfmt-dev; `libboost-all-dev` from B4) ([unitree_sdk2 README](https://github.com/unitreerobotics/unitree_sdk2)).
- Install into its own folder `/opt/unitree_robotics` (the README's second option), so it does not mix into `/usr/local` on the shared board and is removed by deleting one folder:
  ```
  cd ~/src && git clone https://github.com/unitreerobotics/unitree_sdk2
  cd unitree_sdk2 && mkdir build && cd build
  cmake .. -DCMAKE_INSTALL_PREFIX=/opt/unitree_robotics && sudo make install
  ```
- Check: build of the SDK examples finishes; later our program finds it via `CMAKE_PREFIX_PATH=/opt/unitree_robotics`.
- Undo: `sudo rm -rf /opt/unitree_robotics ~/src/unitree_sdk2`.
- Can wait until the gripper control program is written (it must include a force limit, [[2026-11-16 Dex1-1 force limiting on contact]]).

### Deferred (not in this go)

- **Camera stack (teleimager):** needs libjpeg-turbo ≥ 3.0 (Jetson has 2.1.2), i.e. a source build or another packaging route, and it is unknown whether the Dex1-1 camera is plain UVC. Plan it when the gripper arrives.
- **Recording storage:** measure the size of one recorded episode first; then decide between the 31 GB eMMC, the SD card (owner first; SD write speed to be measured) and an external SSD.
- **ROS 2 / Docker:** present, not used by Path 2.

## C. Installed after the go (2026-11-17, Martyn: "go B1 B2")

| Item | Result | How checked |
| --- | --- | --- |
| B1 PC Service | `XRoboToolkit-PC-Service-headless_1.0.0.0_arm64.deb` (38,477,318 bytes, matches the release asset; sha256 `532c605dfa1a02b05b7c285b856c91771c78623cded30ef5b16ea371de49ed5f`, no published hash). Package `roboticsservice` 1.0.0.0, `Depends` empty, installs to `/opt/apps/roboticsservice` (incl. `runService.sh`, own Qt 6.6.2 libs); maintainer scripts only refresh icon/menu caches. Files came with numeric owner uid 1000 (= `saad` here); fixed with `sudo chown -R root:root /opt/apps/roboticsservice`. Headless package: `RobotDemoQt`/`RobotDataRecorder` lack Qt6 Gui/Qml (not needed). Started as on fractal-pc (`nohup setsid bash runService.sh > /tmp/xrsvc.log`), no autostart; listens TCP *:63901 and 127.0.0.1:60061. Stop: `pkill -f RoboticsServiceProcess`. | `dpkg -I/-c`, `ldd`, `ss -ltnup`, log |
| B2 Python SDK | venv `~/venvs/xrobotoolkit`, `xrobotoolkit-sdk` 1.1.0 (aarch64 wheel, no extra dependencies). | import test |
| Headset → Jetson | Headset app connected to 10.36.1.124, status WORKING (headset at 10.36.1.128). SDK read both triggers as analog 0–1 values (e.g. 0.212, 0.502, 0.596) and the right controller position live, 2 Hz print over 20 s. For ≈6.5 s after `init()` all values and the timestamp read 0 until "device found": a recording script must wait for a non-zero timestamp. | 20 s SDK loop while Martyn squeezed the triggers |

## D. Installed after the go (2026-11-18, Martyn: "Go with B3 and B4", then "Go B5")

| Item | Result | How checked |
| --- | --- | --- |
| B3 clone | New key `~/.ssh/dex1_1_handheld_deploy` (ed25519, `SHA256:vnEPzGYhij/PhKYA+xF644oCs9xU7SvJF/273gi8L+o`), added by Martyn as read-only deploy key `station-jetson`. Alias `github-dex1` in `~/.ssh/config`; GitHub host key matches the published ED25519 fingerprint `SHA256:+DiY3wvvV6TuJJhbpZisF/zLDA0zPMSvHdkr4UvCOqU`. Clone `~/dex1-1-handheld` at 1ac22d3. | `git pull` ok; `git push --dry-run` refused |
| B4 packages | `libserialport-dev` 0.1.1-4, `libboost-all-dev` 1.74.0.3ubuntu7 (installed by Martyn with sudo). apt dry run beforehand: 89 new packages, 17.4 MB download, ≈146 MiB installed, nothing upgraded or removed. | `apt-get install -s`, `dpkg-query` |
| B4 needs B5 | **Plan error:** `dex1_1_service` does not build without `unitree_sdk2` (`unitree/idl/go2/MotorCmds_.hpp: No such file`; its CMakeLists links `unitree_sdk2 ddsc ddscxx`, README troubleshooting says install the SDK first). B5 was therefore installed too (Martyn's go). | `make` log |
| B5 `unitree_sdk2` | `~/src/unitree_sdk2` at aaf69cb (2026-11-09), built with `-DCMAKE_INSTALL_PREFIX=/opt/unitree_robotics`; `sudo make install` run by Martyn. `/opt/unitree_robotics`: `include/` (incl. `ddscxx`), `lib/` (`libunitree_sdk2.a`, `libddsc.so`, `libddscxx.so`), owner root. 45 MB. | `ls`, `du` |
| B4 build | `~/src/dex1_1_service` at ced3216 (2026-09-16), source unchanged. Its CMakeLists hard-codes `/usr/local/include/ddscxx`, so the SDK paths are passed as flags: `cmake .. -DCMAKE_CXX_FLAGS="-I/opt/unitree_robotics/include -I/opt/unitree_robotics/include/ddscxx" -DCMAKE_EXE_LINKER_FLAGS="-L/opt/unitree_robotics/lib -Wl,-rpath,/opt/unitree_robotics/lib:/home/martyn/src/dex1_1_service/lib"`. Binaries in `~/src/dex1_1_service/bin/` (`dex1_1_gripper_server`, two test programs). | `make` rc 0, `ldd` (all libraries resolved) |
| B4 check | `./dex1_1_gripper_server --network eno1` as user `martyn` (no sudo; `martyn` is in `dialout`): `[warning] No ttyUSB serial ports found.`, then exits by itself (rc 0). Whether it needs sudo with the gripper attached is unverified. No systemd unit, no autostart. | run with 8 s timeout, `pgrep`, `systemctl list-unit-files` |
| Disk | `/` free before 32,090,128,384 B, after 31,399,714,816 B (−0.69 GB). Sizes: `~/src/unitree_sdk2` 440 MB (incl. build), `~/src/dex1_1_service` 22 MB, `~/dex1-1-handheld` 5.8 MB, `/opt/unitree_robotics` 45 MB; rest apt packages. | `df -B1 /`, `du -sh` |

Undo B5: `sudo rm -rf /opt/unitree_robotics`; `rm -rf ~/src/unitree_sdk2`.

**Trigger → Mac sim from the Jetson (2026-11-18, ≈16:00 CET, run by Martyn):** PC Service started on the Jetson (`runService.sh`, no autostart), headset app connected to 10.36.1.124, sender `~/venvs/xrobotoolkit/bin/python -u ~/dex1-1-handheld/software/xr/trigger_udp.py --host 10.36.1.125` from the clone (5ddc8d3), Mac sim `run_dex1_1.sh --udp 5005`. Result from both terminal outputs: sender reached seq ≈1560 (≈52 s at 30 Hz); the Mac counted **1558 received, 3 lost (0.19 %)**; one gap of 0.66 s around packet 459 (2 of the 3 losses). Trigger values covered 0.000 to 1.000 (e.g. 0.161, 0.541, 0.788, 1.000) and reached the sim (its printed `trigger` follows the sender). The viewer fingers followed the trigger (Martyn, observed). Cause of the gap and losses unknown (Wi-Fi headset link or LAN; not measured). Same path as on fractal-pc 2026-11-09 (0 lost of 425), now with the Jetson as PC. Service stopped afterwards.

## Open

- Owner and contents of the 256 GB SD card (ask the intern). Superseded 2026-11-17 by [[2026-11-17 Station Jetson SD card check]] (empty, left unused).
- B3–B5 wait on Martyn's go; moved to [[2026-11-18]] TODO (2026-11-17). XRoboToolkit service stopped at day end 2026-11-17. Done 2026-11-18 (§D).
- `trigger_udp.py` from the clone → Mac sim test on the Jetson: done 2026-11-18 (§D).
- fractal-pc clock is ≈1 h 44 min fast with NTP inactive (`timedatectl`, 2026-11-17); not ours to fix, worth telling its admin.
