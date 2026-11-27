---
date: 2026-11-25
tags: [report, replay-ladder, rl5, sync, recorder, plan]
---
# RL5 plan: D435i camera, recorder and sync test

> **2026-11-27:** the sync parts of this plan (S1 screen clock, S2 robot motion, S3 controller on the flange, R5 sync run, the board wave, `delay_xcorr.py`) are replaced by the sync method locked in [[D06 Recording and synchronisation]]. R1–R4 results stay valid.

Ask (Martyn, 2026-11-25): plan RL5 (D435i camera + recorder, the sync test) of the Replay Ladder, now that the Jetson Move passed ([[2026-11-25 Jetson Move replay install plan]]). **Plan only: nothing installed, nothing run on the Jetson, the Pi or the robot.** Every install below needs Martyn's go. Background and prior work: [[2026-11-23 RL5 sync test findings]] (the methods are taken from there and not repeated); recorder design: [[D06 Recording and synchronisation]], [[D07 Software and data format]].

Assuming: the only D435i stays on the UR10e flange, plugged into the station Pi (`arena.local`), where the team's `realsense_streamer.py` (team `xr_teleoperate` 7e6c754, `teleop/camera_stream/`) sends colour frames on port 5555; from project memory and the team code (read 2026-11-25 on the Mac clone). The station Jetson has no screen (SSH only).

## Short version (plain words)

- **Goal of RL5:** one recording program on the station Jetson that writes camera images, controller pose and trigger, and (during a replay) the robot's own position, all into one raw MCAP file with their original timestamps; plus measured delays for the camera and the controller, so the streams can be lined up afterwards to within about 10 ms (the D06 (recording and synchronisation) target).
- **Camera stays where it is** (on the Pi). The Jetson listens to the team's existing camera stream over the network. Nothing on the Pi or the robot changes; the team's VR setup keeps working.
- **Delays measured with the robot as the clock**, not with a screen: the robot moves in a slow, known wave; the camera sees a marker on the table move, the controller (held by the gripper or taped to the flange) reports its own wave. Shifting each curve until it matches the robot's curve gives each delay. This avoids the screen-lag problem of the screen method and needs no monitor at the Jetson.
- **Installs (Martyn's go):** three small Python packages on the Jetson (`pyzmq`, `mcap`, `opencv-python-headless`), nothing on the Pi. One small code change to the replay log (wall-clock time per tick).
- **Estimated lab time:** about 2 h at the robot (unverified), after about 1 day of code on the Mac.

## 1. What the team camera stream already gives (read from the team code)

| Point                         | Finding                                                                                                                                                                                                                                                                  | Source                                                                                                                                                   |
| ----------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ | -------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Stream                        | D435i colour 848×480 at 30 fps, JPEG quality 90, ZMQ PUB on port 5555; each message = capture time (int64 ns since epoch) + JPEG                                                                                                                                         | `realsense_streamer.py`                                                                                                                                  |
| Exposure                      | Auto exposure and auto white balance off; fixed values given on the command line                                                                                                                                                                                         | same                                                                                                                                                     |
| Which timestamp               | `color.get_timestamp()`, "in librealsense's global time domain: the camera's capture time on the Pi's clock" (code comment)                                                                                                                                              | same                                                                                                                                                     |
| What librealsense means by it | Global time = "measured in relation to the camera clock and converted to OS system clock by constantly measure the difference"; the frame timestamp is set "when data readout and transmit commence", not mid-exposure (mid-exposure is the separate `SENSOR_TIMESTAMP`) | [librealsense `rs_frame.h`](https://github.com/IntelRealSense/librealsense/blob/master/include/librealsense2/h/rs_frame.h) (not in bib), read 2026-11-25 |
| Clocks                        | "the Pi's clock (chrony keeps it on the Orin's)": the Pi follows the **team's** Orin, while the station Jetson uses `htpdate` (project memory). So Pi and station Jetson are two clocks with an unknown offset                                                           | `realsense_streamer.py` docstring; [[2026-11-17 Station Jetson setup and install plan]]                                                                  |
| Consumer                      | Team `republisher.py` turns the stream into a ROS 2 topic on the Orin; the stream is ZMQ PUB, so a second listener (our recorder) can join without disturbing it                                                                                                         | `republisher.py`; ZMQ PUB/SUB fan-out ([ZeroMQ guide](https://zguide.zeromq.org/docs/chapter1/), not in bib)                                             |

Consequence: frame capture times arrive already stamped at the source, which is what UMI and ViTaMIn-B approximate by subtracting a measured camera delay ([[2026-11-23 RL5 sync test findings]] §2). Two things remain: the Pi–Jetson clock offset, and how far "readout start" lies after the middle of the exposure (fixed exposure, so a constant; unverified size).

**R1 (read-only look), 2026-11-26, station Jetson over SSH from the Mac:** clock synchronised (`timedatectl`: yes, NTP service active); `htpdate` runs as a service since 2026-11-24 against `www.pool.ntp.org`, `www.ntp.br`, `www.wikipedia.org` (HTTP time, so its accuracy is unverified at the ms level); disk 27 GB free of 57 GB; ROS 2 Humble has rosbag2 0.15.17 **with the MCAP storage plugin** (`ros-humble-rosbag2-storage-mcap`, `librosbag2_storage_mcap.so`), `rosbag2_py` in `/opt/ros/humble/local/lib/python3.10/dist-packages/`. Station Pi: not reached, `ssh arena@arena.local` refused (no key from the Mac), so stream, chrony and the Pi–Jetson offset are still open. Mac: `mcap` 1.6.0 and `mcap-ros2-support` 0.6.0 installed in `mujoco-lab`; the recorder's fake test and `delay_xcorr.py --selftest` pass there too.

**R1, station Pi (2026-11-26, `ssh station-pi`, read only):** chrony follows one source, 10.36.1.108 (stratum 10), within about 2 ms RMS; but the Pi clock is **about 16.3 s behind the station Jetson** (five `date` rounds from the Mac in both orders: −16.03 s and −16.57 s, about ±0.27 s ssh time each way; the Jetson is within about 0.1 s of the Mac). So 10.36.1.108 (probably the team's Orin, which the streamer docstring names as the Pi's time source; unverified) is itself about 16 s off; the team's own setup stays consistent because its Pi and Orin share that clock. The camera streamer was not running (nothing on port 5555). `~/wrist/stamp_check.py` exists: it measures frame stamp minus mid-exposure (150 frames, fixed exposure 166, white balance 4600).

Consequence: Pi capture stamps cannot be compared with Jetson times directly. Options: (1) use the Jetson receive time minus a measured, constant camera delay, keeping the Pi stamp only to check network jitter (the spread of receive − stamp shows jitter even with a constant offset); this is the one-clock approach of D06 (recording and synchronisation) and ViTaMIn-B; (2) correct the Pi stamp by an offset tracked from receive − stamp over the take; (3) put both computers on one time source (changes chrony on a team machine or the time on the shared Jetson; sudo; not recommended). Recommended: (1), with (2) only if R4 shows receive-time jitter above about 3 ms. Reasoned only; R4 measures the jitter.

**Order agreed (Martyn, 2026-11-26):** (1) RL5 with the D435i on the station Pi, aligned by the Jetson's receive time minus the measured delay; the Pi's stamp kept only for jitter. (2) Then the Jetson camera path with the plain-webcam reader (V4L2), so it carries over to the Dex1-1 camera; the D435i may be moved to the Jetson for this (team intern agreed to sharing camera, robot and gripper), and **is plugged back into the station Pi before Martyn leaves the lab**. (3) The Robotiq stays on the Pi. Ideas not taken: station Jetson following the team Orin's clock (small gain, sudo, depends on a team machine); Robotiq on the Jetson (no clock problem, not reused for the Dex1-1).

**R4 (60 s camera-only recording), 2026-11-26, passed.** Camera on the station Pi (team streamer, exposure 166, white balance 4600, gains 1 1 1, since no team values were found), recorder code 6f02f87 on the station Jetson (`~/venvs/xrobotoolkit` + pyzmq 27.3.0, mcap 1.6.0, mcap-ros2-support 0.6.0), take `~/rl5-r4/take-01.mcap` (now Mac `msc-thesis/data/rl5/2026-11-26-r4-pi/`). 1800 frames in 60.006 s (29.98 fps), no dropped frames; frame interval by camera stamp 33.30–33.37 ms, by Jetson arrival 30.66–36.18 ms. Receive − Pi stamp: median 16 442.18 ms, spread 7.10 ms (p5–p95 4.3 ms, SD 1.38 ms). The offset **drifts**: +2.5 ms between the first and last 10 s (about 50 ppm, one take, not repeated), about 30 ms over 10 min. Size 195.6 MB per minute (3.3 MB/s, mean JPEG 108.6 kB), about 11.7 GB per hour, so 27 GB free on the Jetson hold about 2 h of camera.

Consequence for the camera time (reasoned only, checked at R5): the camera's own stamps are very regular (±0.04 ms), the arrival times jitter by a few ms. Best of both: per take, fit a straight line to receive − stamp (its lower edge = offset + drift + smallest network delay), then frame time = Pi stamp + fitted line. That keeps the camera's regular timing on the Jetson clock and removes both the 16 s offset and the drift. The constant part of the camera delay (exposure, readout, smallest network time) is still measured in R5 against the robot motion. Using the arrival time alone stays the fallback (SD 1.4 ms, also within the 10 ms target). Least-squares line on take-01 (all 1800 frames): receive − stamp = 16 440.47 ms + 0.0649 ms/s × t, i.e. drift **about 65 ppm** (replaces the rough 50 ppm from first vs last 10 s); residuals −2.4 to +3.3 ms. A line through the middle includes the mean network delay (about 1–2 ms) in the offset; it is absorbed into the constant camera delay measured in R5; a lower-edge fit would reduce it (unverified).

**Camera on the station Jetson (Martyn, 2026-11-26: "Let's just have the camera in the Jetson").** This replaces the Pi path as the main route (step (2) of the agreed order, taken now). Reason: on the Pi, the camera needs a clock fix (the 16 s offset plus about 65 ppm drift); on the Jetson, the camera is on the same clock as the controller and the robot, and the same reader serves the Dex1-1 camera later. The D435i goes back to the station Pi before Martyn leaves the lab.

**R4b (60 s camera-only recording, D435i on the Jetson), 2026-11-26.** Read as a plain webcam (V4L2, kernel driver `uvcvideo`); no RealSense SDK, no sudo, no udev rule. `lsusb -t` shows 5000M (USB 3). The colour node is `/dev/video4` (YUYV 640×480 and 848×480 at 30 fps); `/dev/video0` is depth, `/dev/video2` infrared. Recorder e6e5278 in `~/rl5-recorder`, opencv-python-headless 5.1.0 added to `~/venvs/xrobotoolkit`, take `~/rl5-r4-jetson/take-01.mcap`. Results:
- 1700 frames; frame interval by kernel stamp, median 33.36 ms.
- Receive − kernel stamp: median 32.53 ms (first 300 frames 32.55, last 300 frames 32.50), so **no offset and no drift**.
- Normal frames 31.8–33.2 ms. The constant of about 32.5 ms is roughly one frame; it is probably the moment the driver stamps the frame plus OpenCV's read, but that is unverified, and R5 measures the true delay.
- 161 MB per minute (mean JPEG 94.7 kB, quality 90); colour image checked (frame 900).
- **One stall:** at 25.9 s the reader stopped for 2.5 s. Four queued frames arrived late, each with its correct stamp, and about 74 frames were lost.
- Cause of the stall unknown. Candidates (none checked): an eMMC write, Python garbage collection, other load. `dmesg` needs root.
- Exposure and white balance were the camera defaults (not set).

Fix in dex1-1-handheld PR #24 (7f2d504, desk-tested with `--fake` only): the file is written by its own thread from a queue, and the recorder prints the longest single write and any camera gap over 200 ms. **Next check:** repeat the 60 s take three times. If the gaps still occur with short writes, the cause is not the disk.

**Retest, 2026-11-26 (7f2d504, three 60 s takes, `~/rl5-r4-jetson-b/`, now Mac `msc-thesis/data/rl5/2026-11-26-retest/`):**

| take | frames | fps | longest write | receive − source, median / max / SD |
| --- | --- | --- | --- | --- |
| 01 | 1779 | 29.82 | 32.8 ms | 32.50 / 36.22 / 1.38 ms |
| 02 | 1786 | 29.82 | 28.7 ms | 32.23 / 34.04 / 1.36 ms |
| 03 | 1784 | 29.83 | 30.0 ms | 32.39 / 34.63 / 1.37 ms |

- No gap over 45 ms in the middle of any take. During the takes the Jetson's CPU was 84 % idle and I/O wait was 0.08 % (vmstat).
- The stall is likely caused by the disk write inside the camera loop. This is not proven: one event in 1 min against none in 3 min. **A 10 min take is still to do.**
- Each take started with about 0.6 s of unusable frames while the camera opened: 3 old buffered frames without a kernel timestamp, then about 0.4 s of nothing.
- Chart of all four takes, drawn from the measured summary numbers: ![[2026-11-26 RL5 Jetson camera holes.png]]
- Fix e957524, desk-tested only: `--camera-only` waits 1 s before the take, and frames without a kernel timestamp are dropped. With the headset, the camera is already open before the double-click (not checked).

## 2. Where the camera is read: options

| Option | How | For | Against | Precedent |
| --- | --- | --- | --- | --- |
| **A. Listen to the Pi stream (recommended)** | Recorder on the station Jetson subscribes to `arena.local:5555`; stores the capture stamp (Pi clock) and its own receive time (Jetson clock) per frame | No hardware change; nothing on the Pi; team setup untouched; per-frame capture time already there | Two clocks (Pi, Jetson): offset and drift must be measured each session; network adds jitter to the receive time (not to the capture stamp) | Team stack (Pi streamer + Orin listener); RAPID records several machines into MCAP (yin2026, III-C); D06 prefers one clock |
| B. Move the D435i USB cable to the Jetson | CAM1–CAM3 of [[2026-11-19 Station Jetson camera install plan]] | One clock, as D06 wants | Unplugs team hardware the VR setup uses; cable route flange → Jetson unknown; D435i colour over plain UVC on the Tegra kernel unverified; librealsense not on the Jetson | UMI, ViTaMIn-B read the camera on the recording host (chi2024, li2025) |
| C. Run our recorder on the Pi | Recorder next to the camera | One clock for the camera | Pose (XRoboToolkit) and robot run on the Jetson, so the clock problem moves, not disappears; installs on a team machine | none found |

Decision rule: A, unless the clock check (step R3) shows offset jitter above about 5 ms within a session, which would eat half of the 10 ms target; then B. Reasoned only (offset not measured yet).

## 3. Measuring the delays: options

The findings note lists the screen method (UMI App. A, ViTaMIn-B IV-B) and the controller swing. New since then: the robot now replays from the Jetson, so the robot's own measured position, read on the Jetson clock, can serve as the reference.

| Option | How | For | Against | Precedent |
| --- | --- | --- | --- | --- |
| S1. Screen clock | Screen shows a changing marker with known display time; camera films it; delay = capture stamp − display time | Standard in UMI-style work | Screen must hang on the Jetson (no screen there now); 60 Hz refresh gives up to 16.7 ms uncertainty plus unknown panel lag, above the 10 ms target | UMI App. A.0-A1 (chi2024); ViTaMIn-B IV-B (li2025) |
| **S2. Robot motion as reference (recommended)** | Robot at HOME_FACE moves slowly along one axis in a sine (e.g. ±40 mm, 0.5 Hz, within the RL3 speed limits); flange camera looks at an ArUco marker fixed on the table; marker position in the image vs the robot's measured TCP position (RTDE, Jetson clock); the time shift with the best match (cross-correlation) = camera delay incl. Pi–Jetson offset | No display lag; uses only what is already at the robot; the same run also gives the pose delay (S3); sub-frame resolution from interpolated curves | Robot state delay itself (about ½ network round trip, ≈0.5 ms from the 1.0 ms ping) enters the result; needs the replay log to carry wall-clock time (code change, section 5) | UMI App. A.0-A3 uses the same cross-correlation of a sine command vs measured response for the gripper (chi2024); camera–robot motion matching is the idea behind joint temporal calibration in Kalibr ([Furgale et al. 2013](https://doi.org/10.1109/IROS.2013.6696514), not in bib). For the camera on a robot arm: no direct precedent in the bib |
| S3. Controller pose delay | During the S2 run, the PICO controller is fixed to the flange (held by the 2F-85 in the handheld sleeve, or taped to the camera adapter); controller position vs robot TCP, cross-correlated | Same run, same reference; no hand shake; covers tracking + transport | Controller must be fixed so it cannot fall (gripper force on the sleeve, unverified); headset must see the controller from the operator place | UMI A.0-A3 method; ViTaMIn-B measured Quest transport only (li2025, IV-B); hand swing in front of the camera in [[2026-11-23 RL5 sync test findings]] §3 is the fallback |

Recommendation: S2 + S3 in one run, S1 only as a cross-check if a screen can be plugged into the Jetson. Reasoned only; first check in the desk step R4.

## 4. Recorder (SwapKit camera plug + recorder)

**Update 2026-11-26:** the team already records VR-teleop episodes as ROS 2 bags in MCAP; the channel names below should follow the team's topics and message types so one exporter reads both ([[2026-11-26 Team bag recorder and RL5 format]], option B with CDR chosen 2026-11-26, [[D07 Software and data format]]).

One Python program on the station Jetson, written against fixed interfaces so the Dex1-1 camera is a plug swap in RL6 (SwapKit option A, [[D07 Software and data format]]).

| Channel (MCAP) | Content | `publish_time` | `log_time` |
| --- | --- | --- | --- |
| `/camera/wrist` | JPEG bytes as received (no re-encode) | capture stamp from the stream (Pi clock) | Jetson receive time |
| `/xr/right` | pose (7 values, unconverted), trigger, buttons | `xr_ts_ns` (XRoboToolkit `timeStampNs`) | Jetson `time.time_ns()` |
| `/robot/state` (only during a replay) | actual TCP pose and joints, RTDE robot timestamp | robot timestamp | Jetson time |
| `/meta` | setup file, calibration version (D09 (calibration)), delay-file version, code commit | — | start of take |

- Start/stop: A button, as `pose_log.py` (G8 (does pressing move the controller) press test still open; the A button is the working choice until then).
- Robot state: written by the replay script into its own MCAP (or CSV with wall-clock time) on the same Jetson clock; merged offline. Keeps the 125 Hz robot loop free of camera work (reasoned only).
- Process split: the recorder runs in the existing `~/venvs/xrobotoolkit` (the only place `xrobotoolkit_sdk` is installed); the replay keeps the conda env `replay`.
- Alignment: offline on the Mac, all three methods of the findings note §2 (delay + interpolation, nearest neighbour, 25 ms window) on the same raw take; residual error from the S2/S3 run (gap A5 (no residual sync error reported), [[2026-11-02 Research gaps]]).
- Images are sync-test data only: the flange D435i sees just the 2F-85 fingertips ([[2026-11-25 Camera View Dex1-1 and D435i field of view]]).

## 5. Steps

| Step | Where | What | Installs | Check (pass if) |
| --- | --- | --- | --- | --- |
| R1 (read-only look) | Jetson, Pi over SSH | Is the streamer running? Port 5555 open? Pi: `chronyc tracking` (whom it follows); Jetson: `timedatectl`, htpdate status; disk free | none | Facts written into this note §1 |
| R2 (desk code) | Mac | Recorder (camera plug for the ZMQ stream, XR source, MCAP writer) with `--fake` sources as in `pose_log.py`; wall-clock column + RTDE timestamp in the replay log; analysis script for S2/S3 (cross-correlation) tested on simulated data with a known shift | Mac only: `mcap` into the existing conda env (Martyn's go); `pyzmq` is already in `mujoco-lab` | Simulated 40 ms shift recovered within 1 ms |
| R3 (installs + clock check) | Jetson | `pip install pyzmq mcap opencv-python-headless` into `~/venvs/xrobotoolkit`; 10 min clock check: Pi–Jetson offset by a small round-trip script (no install on the Pi beyond what is there; method: NTP-style 4-timestamp exchange, unverified on this setup) | **3 packages, Martyn's go** | Offset spread within a session < 5 ms → option A stays |
| R4 (camera only, no robot motion) | Jetson + flange camera | Record 60 s with the robot still; frame rate, dropped frames, capture-to-receive time; MCAP opens on the Mac | none | 30 fps ± 1, no gaps > 2 frames |
| R5 (sync run) | Robot, Remote mode, slow, person at the stop | S2 + S3: marker on the table, controller fixed to the flange, sine motion program run by the replay script (dry run in MuJoCo first, as RL4) | none | Camera delay and pose delay each with spread < 5 ms over 3 runs |
| R6 (handheld take) | At the table | Record a handheld take (camera + controller) and a replay of it with `/robot/state`; align offline; delays stored as a versioned delay file | none | Residual error after alignment ≤ 10 ms (D06 target), reported per method |

RL5 passes when R6 passes. Then step 3 (export to LeRobot) can start on R6 data.

## 6. Risks

| Risk | Mitigation |
| --- | --- |
| Team uses the camera or the robot at the same time | Agree slots; our recorder only listens (no effect on the team stream) |
| Pi clock follows the team Orin, which may be off or drifting | R3 measures; option B if too noisy |
| Readout-start vs mid-exposure offset unknown | Fixed exposure makes it constant; S2 measures it as part of the total camera delay |
| Controller slips out of the gripper during S3 | Tape plus gripper; slow motion; fallback: hand swing (findings note §3) |
| Jetson disk (27 GB free) | R4 gives MB per minute; JPEG 848×480 at 30 fps estimated ≈1–2 MB/s (unverified) |

## Open

- ~~Martyn's go for the three Jetson packages (R3) and for `mcap` on the Mac (R2).~~ Resolved 2026-11-26: installed on both (see R4, R4b).
- 10 min camera-only take on the Jetson (Retest above): moved to daily [[2026-11-27]].
- Delay method, proposed 2026-11-26, not yet chosen by Martyn: a desk board wave (controller taped to a printed marker, waved at the camera, `delay_xcorr.py`, same method as the team's `diag/analyze_diag.py`) instead of the R5 (sync run) sine motion; plus a table marker at the next ordinary replay for the camera-to-robot delay. Moved to daily [[2026-11-27]].
- Whether a screen can be plugged into the Jetson (only for the S1 cross-check). In [[Waiting]] (RL5); moot if the board wave is chosen.
- Whether the 2F-85 can hold the handheld sleeve safely for S3 (look at the robot).
- Who set up `orin-new-hotspot` on the Jetson Wi-Fi (from the Jetson Move note).

Sources: [[2026-11-23 RL5 sync test findings]] (UMI chi2024, ViTaMIn-B li2025, FastUMI zhaxizhuoma2025, RAPID yin2026, XRoboToolkit zhao2025); team code `xr_teleoperate` 7e6c754 `teleop/camera_stream/` (read 2026-11-25); librealsense `rs_frame.h` (not in bib). No deep research used.

## Add to Zotero

- Furgale, Rehder, Siegwart, "Unified temporal and spatial calibration for multi-sensor systems", IROS 2013, https://doi.org/10.1109/IROS.2013.6696514 — precedent for estimating a time offset between sensors by matching their motion (basis of S2/S3).
