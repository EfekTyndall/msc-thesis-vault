---
date: 2026-11-25
tags: [report, replay-ladder, rl5, sync, recorder, plan]
---
# RL5 plan: D435i camera, recorder and sync test

Ask (Martyn, 2026-11-25): plan RL5 (D435i camera + recorder, the sync test) of the Replay Ladder, now that the Jetson Move passed ([[2026-11-25 Jetson Move replay install plan]]). **Plan only: nothing installed, nothing run on the Jetson, the Pi or the robot.** Every install below needs Martyn's go. Background and prior work: [[2026-11-23 RL5 sync test findings]] (the methods are taken from there and not repeated); recorder design: [[D06 Recording and synchronisation]], [[D07 Software and data format]].

Assuming: the only D435i stays on the UR10e flange, plugged into the station Pi (`arena.local`), where the team's `realsense_streamer.py` (team `xr_teleoperate` 7e6c754, `teleop/camera_stream/`) sends colour frames on port 5555; from project memory and the team code (read 2026-11-25 on the Mac clone). The station Jetson has no screen (SSH only).

## Short version (plain words)

- **Goal of RL5:** one recording program on the station Jetson that writes camera images, controller pose and trigger, and (during a replay) the robot's own position, all into one raw MCAP file with their original timestamps; plus measured delays for the camera and the controller, so the streams can be lined up afterwards to within about 10 ms (the D06 (recording and synchronisation) target).
- **Camera stays where it is** (on the Pi). The Jetson listens to the team's existing camera stream over the network. Nothing on the Pi or the robot changes; the team's VR setup keeps working.
- **Delays measured with the robot as the clock**, not with a screen: the robot moves in a slow, known wave; the camera sees a marker on the table move, the controller (held by the gripper or taped to the flange) reports its own wave. Shifting each curve until it matches the robot's curve gives each delay. This avoids the screen-lag problem of the screen method and needs no monitor at the Jetson.
- **Installs (Martyn's go):** three small Python packages on the Jetson (`pyzmq`, `mcap`, `opencv-python-headless`), nothing on the Pi. One small code change to the replay log (wall-clock time per tick).
- **Estimated lab time:** about 2 h at the robot (unverified), after about 1 day of code on the Mac.

## 1. What the team camera stream already gives (read from the team code)

| Point | Finding | Source |
| --- | --- | --- |
| Stream | D435i colour 848×480 at 30 fps, JPEG quality 90, ZMQ PUB on port 5555; each message = capture time (int64 ns since epoch) + JPEG | `realsense_streamer.py` |
| Exposure | Auto exposure and auto white balance off; fixed values given on the command line | same |
| Which timestamp | `color.get_timestamp()`, "in librealsense's global time domain: the camera's capture time on the Pi's clock" (code comment) | same |
| What librealsense means by it | Global time = "measured in relation to the camera clock and converted to OS system clock by constantly measure the difference"; the frame timestamp is set "when data readout and transmit commence", not mid-exposure (mid-exposure is the separate `SENSOR_TIMESTAMP`) | [librealsense `rs_frame.h`](https://github.com/IntelRealSense/librealsense/blob/master/include/librealsense2/h/rs_frame.h) (not in bib), read 2026-11-25 |
| Clocks | "the Pi's clock (chrony keeps it on the Orin's)": the Pi follows the **team's** Orin, while the station Jetson uses `htpdate` (project memory). So Pi and station Jetson are two clocks with an unknown offset | `realsense_streamer.py` docstring; [[2026-11-17 Station Jetson setup and install plan]] |
| Consumer | Team `republisher.py` turns the stream into a ROS 2 topic on the Orin; the stream is ZMQ PUB, so a second listener (our recorder) can join without disturbing it | `republisher.py`; ZMQ PUB/SUB fan-out ([ZeroMQ guide](https://zguide.zeromq.org/docs/chapter1/), not in bib) |

Consequence: frame capture times arrive already stamped at the source, which is what UMI and ViTaMIn-B approximate by subtracting a measured camera delay ([[2026-11-23 RL5 sync test findings]] §2). Two things remain: the Pi–Jetson clock offset, and how far "readout start" lies after the middle of the exposure (fixed exposure, so a constant; unverified size).

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

- Martyn's go for the three Jetson packages (R3) and for `mcap` on the Mac (R2).
- Whether a screen can be plugged into the Jetson (only for the S1 cross-check).
- Whether the 2F-85 can hold the handheld sleeve safely for S3 (look at the robot).
- Who set up `orin-new-hotspot` on the Jetson Wi-Fi (from the Jetson Move note).

Sources: [[2026-11-23 RL5 sync test findings]] (UMI chi2024, ViTaMIn-B li2025, FastUMI zhaxizhuoma2025, RAPID yin2026, XRoboToolkit zhao2025); team code `xr_teleoperate` 7e6c754 `teleop/camera_stream/` (read 2026-11-25); librealsense `rs_frame.h` (not in bib). No deep research used.

## Add to Zotero

- Furgale, Rehder, Siegwart, "Unified temporal and spatial calibration for multi-sensor systems", IROS 2013, https://doi.org/10.1109/IROS.2013.6696514 — precedent for estimating a time offset between sensors by matching their motion (basis of S2/S3).
