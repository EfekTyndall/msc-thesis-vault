---
date: 2026-11-25
tags: [report, replay-ladder, jetson, install-plan]
---
# Jetson Move: real-robot replay on the station Jetson (inventory and install plan)

Ask (coordinator for Martyn, 2026-11-25): before RL5 (D435i camera + recorder, recording on the Jetson), move the real-robot replay from fractal-pc to the station Jetson. Agreed split: Mac = development and MuJoCo preview; station Jetson = everything at the robot; fractal-pc later only for training. **Plan only: the Jetson was read over `ssh station-jetson`, nothing installed or changed. Each J step below needs Martyn's go (new dependencies).** Earlier Jetson installs: [[2026-11-17 Station Jetson setup and install plan]]; fractal-pc equivalent: [[2026-11-23 RL2 URSim on fractal-pc install plan]]; RL4 (Robotiq open/close during replay) run: [[2026-11-24 RL4 Robotiq pick-and-place procedure]].

## Short version (plain words)

- The Jetson already has the compiler tools and the Boost library that the robot-connection library (ur_rtde) needs. Everything else comes as ready-made Python packages for the Jetson's processor type. **No sudo (admin password) is needed for any step.**
- Only ur_rtde has no ready-made package for the Jetson; pip compiles it from source on the board (one command, estimated 10–20 min, unverified).
- All new software goes into one folder (`~/miniforge3`, conda env `replay`), removable with one `rm -rf`. Nothing autostarts. Conda chosen over a plain venv on 2026-11-25 (Martyn: "which will make life easier?"): same tool as the Mac, fractal-pc and the team's Jetson, and the same conda-forge Pinocchio build as fractal-pc.
- Recommendation: run the whole replay script on the Jetson, unchanged (option A below). A measurement today shows the joint commands come out identical on two different computer types, so the Jetson should reproduce fractal-pc exactly; the check is a side-by-side replay of take-01.
- Estimated time: about 45 min install (mostly the build), 10 min desk checks, 20 min at the robot.

## 1. Station Jetson inventory (read-only, 2026-11-25 ≈16:25 CET)

| Item | Finding | How checked |
| --- | --- | --- |
| Board | Jetson AGX Orin, L4T R36.5.2, kernel 5.15.199-tegra (PREEMPT, not real-time), 8 CPU cores, 61 GiB RAM | `uname -a`, `/etc/nv_tegra_release`, `free` |
| Power mode | Mode 2 `MODE_30W`: CPU capped at 1.73 GHz (hardware max 2.20 GHz in MAXN). Board owner's setting, left as is | `/var/lib/nvpmodel/status`, `cpufreq` |
| Disk | `/` 57 GB, **30 GB free** | `df -h /` |
| Python | System 3.10.12, numpy 1.21.5 (system, not used); `python3-dev`, `python3-venv`, pip present; no conda | `python3 --version`, `dpkg -l` |
| Build tools | cmake 3.22.1, g++ 11.4.0, Boost 1.74 dev (system, thread, program-options; from B4 (dex1_1_service packages) on 2026-11-18), pybind11-dev 2.9.1, Eigen 3.4 | `dpkg -l`, `--version` |
| Existing venv | `~/venvs/xrobotoolkit`: only `xrobotoolkit-sdk` 1.1.0 (headset). Left untouched; the replay gets its own conda env (J2) | `pip list` |
| Not present | ur_rtde, MuJoCo, Pinocchio, CasADi (no apt or pip package), team code `xr_teleoperate`, robot models (`software/sim/models`) | `dpkg -l`, `ls` |
| Our code | `~/dex1-1-handheld` at b055f3f (PR #18); GitHub main is 1f90bd0 (PR #23), so a `git pull` is needed (RL4 code is missing on the Jetson) | `git log`, `git fetch --dry-run` |
| Take | `~/pose-log-20261125-150606/take-01/raw.csv`, same file as on fractal-pc (MD5 `c6f1a2ae…c322c` on both) | `md5sum` |
| Robot | UR10e 10.36.11.3 reachable over `eno1`: ping avg **1.0 ms** (max 1.3 ms, 5 pings). fractal-pc to the robot: avg 0.14 ms. Both well under the 8 ms servoJ tick | `ping` from both |
| Gripper | `arena.local` resolves to 10.36.11.9 (station Pi, Robotiq bridge port 4001) | `getent hosts` |
| Internet | pypi.org and files.pythonhosted.org reachable (HTTP 200/404 headers) | `curl -I` |
| Other users | Only Martyn's own login (from the Mac); load ≈0.9 | `who`, `uptime` |
| Real-time | `ulimit -r` = 0, no RT kernel: ur_rtde then runs at normal priority without warning (same as fractal-pc, whose kernel 6.8.0-generic is not RT either) | `ulimit -r`; ur_rtde 1.6.6 source `rtde_control_interface.cpp` (warns only "even though a realtime kernel is available") |

Side finding (not touched): the Jetson's Wi-Fi now runs an access point `orin-new-hotspot` (10.42.0.1/24); on 2026-11-17 Wi-Fi was off. Probably set up by someone else on the shared board; worth asking before RL5, since the headset path depends on the network.

## 2. What `ur10e_replay_rtde.py` needs (fractal-pc, conda env `ursim-replay`)

| Need | fractal-pc version | Jetson source (aarch64, Python 3.10) |
| --- | --- | --- |
| ur_rtde (`rtde_control`, `rtde_receive`) | 1.6.6 (pip wheel) | **No aarch64 wheel** on PyPI for 1.6.6 (only x86_64, macOS arm64, Windows); source package 15.6 MB, builds with CMake + Boost + Python headers, pybind11 bundled ([PyPI files](https://pypi.org/project/ur-rtde/1.6.6/#files), `pyproject.toml`/`CMakeLists.txt` read 2026-11-25) |
| MuJoCo (imported by `ur10e_replay.py`; runs the RL1-style precompute) | 3.17.0 | wheel `manylinux_2_28_aarch64`, 25.1 MB ([PyPI](https://pypi.org/project/mujoco/3.17.0/#files)) |
| Pinocchio (team IK model, actual-TCP logging) | 4.1.0 (conda-forge) | pip `pin` 4.1.0 wheel `manylinux_2_28_aarch64`, 9.8 MB, plus `cmeel-boost` 1.90 (35.8 MB), `coal`, `eigenpy`, `cmeel-urdfdom` (all aarch64 wheels) ([PyPI pin](https://pypi.org/project/pin/4.1.0/#files)) |
| CasADi (team IK solver, IPOPT) | 3.7.2 | wheel `manylinux2014_aarch64`, 47.3 MB ([PyPI](https://pypi.org/project/casadi/3.7.2/#files)) |
| numpy | 1.24.4 | wheel `aarch64`, 14.0 MB ([PyPI](https://pypi.org/project/numpy/1.24.4/#files)) |
| Team code | `~/team/xr_teleoperate` at 7e6c754 (428 MB with `.git`) | rsync from the Mac clone (same commit) |
| Robot models | Menagerie UR10e + Robotiq 2F-85 via `software/sim/fetch_models.sh` (66 MB) | same script on the Jetson |
| Gripper driver | team `robotiq_2f85_node.py` (numpy only), from the team clone | same |
| Not needed | meshcat, logging_mp, matplotlib (stand-ins in `import_team`) | — |

GLIBC on the Jetson is 2.35 ([[2026-11-17 Station Jetson setup and install plan]]), new enough for the `manylinux_2_28` wheels.

## 3. Where the joint commands are computed: options

Background: the script first computes all joint commands with the team IK inside a MuJoCo simulation of the arm (RL1 (replay in MuJoCo) chain), then streams them at 125 Hz. The precompute is not inside the 8 ms loop, so its speed only decides how long the operator waits before the robot moves.

Measured today on the Mac (M4, conda env `humanoid`, take-01, 3921 ticks):

| Check | Result |
| --- | --- |
| Precompute time | 32.7 s (8.3 ms per tick, IK + MuJoCo step); fractal-pc ≈20 ms per IK solve (2026-11-23, script comment), i.e. ≈1.3 min |
| Same commands on another computer type? | Mac (arm64, MuJoCo 3.11) vs the joint commands fractal-pc (x86_64, MuJoCo 3.17) logged in its real-robot run `rl4-take-01-hyst.csv`: **max difference 2.5×10⁻¹⁴ rad** over all 3921 ticks (median 1.8×10⁻¹⁵). The computation is reproducible across machines |

**Option A: full script on the Jetson (recommended).** Same command as on fractal-pc; the Jetson installs the full list of section 2.
- For: no code change; the take recorded on the Jetson (RL5) is replayed where it lies; matches the agreed split (Jetson = everything at the robot); Unitree's own setup also runs the team IK on a host computer and not on a separate machine ahead of time ([xr_teleoperate README](https://github.com/unitreerobotics/xr_teleoperate), team clone).
- Against: Orin precompute speed unknown (CPU at 1.73 GHz in 30 W mode). Estimate (unverified): 2–4× fractal-pc, i.e. 3–5 min for a 30 s take. Acceptable for the Replay Ladder; measured in J6.

**Option B: precompute on the Mac, stream on the Jetson.** The Mac preview saves the joint commands, start pose and gripper events to a file; the Jetson only streams.
- For: the Jetson needs only ur_rtde, numpy and Pinocchio (no MuJoCo, no CasADi); no waiting at the robot.
- Against: code change (export and load, plus a check that the file belongs to this take and setup file, otherwise a stale file could be streamed); one more copy step Jetson → Mac → Jetson per take; the safety checks (allowed box, floor check) must travel with the file.
- Gains nothing in accuracy: the measurement above shows A and B give the same commands.

Decision rule: A, unless J6 shows a precompute above ≈5 min for take-01 or a package fails to install; then B. Reasoned only (precompute speed on the Orin is not measured yet).

## 4. Install plan (each step needs Martyn's go; none needs sudo)

| Step | What (commands on the Jetson unless noted) | Check | Undo |
| --- | --- | --- | --- |
| J1 (update our code) | `git -C ~/dex1-1-handheld pull` (b055f3f → 1f90bd0) | `git log -1` shows 1f90bd0 | `git checkout b055f3f` |
| J2 (Python packages) | Miniforge into the home folder, no sudo: `wget https://github.com/conda-forge/miniforge/releases/latest/download/Miniforge3-Linux-aarch64.sh` (130 MB), `bash Miniforge3-Linux-aarch64.sh -b -p ~/miniforge3`, then `~/miniforge3/bin/conda config --set auto_activate_base false` and **no** `conda init` (conda is only active when started, so the system `ssh` and JetPack libraries are not disturbed; the team README warns that conda's `LD_LIBRARY_PATH` breaks `ssh`). Env as on fractal-pc: `conda create -n replay -c conda-forge python=3.10 pinocchio=4.1.0 casadi=3.7.2 numpy=1.24.4 libboost-devel cmake`, then `pip install mujoco==3.17.0` (all linux-aarch64 builds exist on conda-forge / PyPI, checked 2026-11-25) | `conda run -n replay python -c "import mujoco, pinocchio, casadi"` prints versions | `rm -rf ~/miniforge3` |
| J3 (ur_rtde build) | In the env: `CMAKE_BUILD_PARALLEL_LEVEL=6 pip install -v ur_rtde==1.6.6` (no aarch64 wheel or conda package, so pip builds the source package). Boost: the env's `libboost-devel` (same Boost 1.90 as Pinocchio's) should be found first; if CMake picks the system Boost 1.74 instead, pass `-DBoost_ROOT=$CONDA_PREFIX` (unverified, checked in the build log). Estimated 10–20 min (unverified) | `python -c "import rtde_control, rtde_receive"`; then **receive only**: `rtde_receive.RTDEReceiveInterface("10.36.11.3").getActualQ()` returns 6 joint angles (reads, never moves the robot) | same as J2 |
| J4 (team code) | From the **Mac**: `rsync -a ~/thesis/team/xr_teleoperate/ station-jetson:team/xr_teleoperate/` (428 MB, commit 7e6c754); on the Jetson add `export XR_TELEOPERATE=~/team/xr_teleoperate` to the run command (the script default is the Mac path) | `git -C ~/team/xr_teleoperate log -1` = 7e6c754 | `rm -rf ~/team/xr_teleoperate` |
| J5 (robot models) | `~/dex1-1-handheld/software/sim/fetch_models.sh` (pinned Menagerie and unitree_mujoco commits, 66 MB, over https) | script prints both commits | `rm -rf ~/dex1-1-handheld/software/sim/models` |
| J6 (speed test, no robot) | `time python ur10e_replay_rtde.py --csv ~/pose-log-20261125-150606/take-01/raw.csv --setup ../config/rl4_robotiq.json --box 0.22 0.05 0.15 --robot 10.36.11.3 --dry-run` | Prints the same RL4 checks as the Mac and fractal-pc (IK error max 3.3 mm, floor check PASSED with the lowest fingertip 303 mm above the table, gripper events close 13.33 / open 15.91 / … s); wall time recorded → decision A or B | — |

Disk use ≈2 GB of 30 GB free (Miniforge env ≈1 GB, team code, models, build). Nothing outside the home folder changes, nothing autostarts.

## 5. Risks

| Risk | Effect | Mitigation |
| --- | --- | --- |
| ur_rtde source build fails on aarch64 (no published aarch64 wheel, so not tested by the release) | No robot connection from the Jetson | Read the build log; build the C++ library with plain CMake as the ur_rtde docs describe; last resort: fractal-pc keeps streaming until fixed |
| aarch64 conda-forge builds differ numerically from x86_64 | Tiny IK differences → commands not bit-identical | Caught by V1 below (Mac arm64 vs fractal-pc x86_64 already agree within 2.5×10⁻¹⁴ rad) |
| Conda on a Jetson | Conda libraries shadowing JetPack's or breaking `ssh` | Base env not auto-activated, no `conda init`; env only used for the replay |
| Slow precompute on the Orin (30 W mode) | Waiting time per take | J6 measures; option B if too slow. Not changing the power mode (owner's setting, needs sudo) |
| Network: 1.0 ms average ping vs 0.14 ms from fractal-pc | Could add jitter to the 8 ms loop | V3 checks the longest tick (fractal-pc: 8.2 ms) |
| Two controllers at once | Robot accepts only one RTDE control client; the Robotiq bridge only one gripper client (connecting cuts off the other) | Never run fractal-pc and the Jetson replay at the same time |
| Shared board | Other users' work | Everything in Martyn's home folder; nothing autostarts; the existing `~/venvs/xrobotoolkit` untouched |

## 6. Verification: same take from fractal-pc and from the Jetson

Reference: fractal-pc real-robot run of take-01 with the trigger hysteresis rule, log `~/rl2-test/rl4-take-01-hyst.csv` (3922 ticks, 31.4 s; tracking target vs actual TCP RMS 6.5 mm, max 21.8 mm, end 0.0 mm; highest joint speed 0.30 rad/s; recomputed from the log today) and its gripper log (6 commands at 13.34 / 15.95 / 18.50 / 21.21 / 24.57 / 25.82 s).

| Step | What | Pass if |
| --- | --- | --- |
| V1 (desk, no robot) | J6 dry run, plus a short comparison of the Jetson's joint commands with the `q_cmd` columns of the fractal-pc log (the same comparison as section 3, run in the scratch folder, not in the repo) | Max difference < 10⁻⁹ rad (Mac vs fractal-pc: 2.5×10⁻¹⁴) |
| V2 (desk, robot on, receive only) | J3 receive check | 6 joint angles that match the pendant |
| V3 (robot) | Same procedure as RL4 (reduced speed, person at the emergency stop, pendant in Remote Control, fractal-pc not connected): `python ur10e_replay_rtde.py --csv ~/pose-log-20261125-150606/take-01/raw.csv --setup ../config/rl4_robotiq.json --box 0.22 0.05 0.15 --robot 10.36.11.3 --log ~/rl4-take-01-jetson.csv`, dry run first | Run completes, no protective stop |
| V4 (compare logs, on the Mac) | Copy `rl4-take-01-jetson.csv` and its `-gripper.csv` to the Mac next to the fractal-pc logs | Commanded joints identical (< 10⁻⁹ rad); tracking RMS and max within ≈1 mm of 6.5 / 21.8 mm (the real arm does not repeat exactly; bound to be confirmed by this run); highest joint speed ≤ 0.30 rad/s; 3922 ticks, longest tick ≤ 10 ms; same 6 gripper commands each within 50 ms (one 20 Hz poll) of the reference |

If V1–V4 pass: the Jetson replaces fractal-pc for real-robot replay, and RL5 (D435i + recorder) can start there.

## 7. How the team's VR teleop Jetson reaches the UR10e (team code, 2026-11-25)

Read from `xr_teleoperate` 7e6c754 (Mac clone); the team Jetson itself was not accessed.

| Point | Finding | Source |
| --- | --- | --- |
| Arm interface | ur_rtde `servoJ` at 125 Hz, lookahead 0.1 s, gain 300; `getActualQ` read back. Same as `ur10e_replay_rtde.py`; no ROS driver | `teleop/robot_control/robot_arm.py`, `UR10e_ArmController` (commit aac0a50) |
| Gripper | `robotiq_2f85_node.py` over the Pi's ser2net bridge, as ours | `teleop/run_ur10e_teleop_real.sh` |
| Environment | conda env `h2_teleop`; Jetson installer uses Miniforge/conda-forge (pinocchio 3.1.0, numpy 1.26.4, casadi), "tested clean on Ubuntu 22.04 / aarch64" | `deploy/install_teleop_env.sh`, `deploy/README.md` |
| ur_rtde | **Not installed by the team installer**; no conda-forge package (anaconda.org search, 2026-11-25) and no aarch64 PyPI wheel, so on an aarch64 Jetson it must have been built from source; how is undocumented (unverified) | as above |

Effect on the plan: the team precedent is conda on the Jetson; J2 now uses Miniforge (changed 2026-11-25, see the short version). Martyn (2026-11-25): own research found no other route, so the team's ur_rtde was most likely built from source; not confirmed with Yanmei (optional question).

## Open

- Martyn's go for J1–J6 (dependencies on the shared Jetson).
- Who set up `orin-new-hotspot` on the Jetson Wi-Fi (side finding).
- Option B only if J6 says so.

Sources: PyPI file lists and the ur_rtde 1.6.6 source package (read 2026-11-25); team code `xr_teleoperate` 7e6c754; measurements on the Mac, fractal-pc and the Jetson as listed. No deep research used.
