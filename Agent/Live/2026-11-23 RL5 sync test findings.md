---
date: 2026-11-23
tags: [research, sync, latency, replay-ladder]
---
# RL5 sync test: findings and things to consider

Collected for RL5 (D435i + recorder: images, pose and trigger with timestamps, raw MCAP) of the Replay Ladder ([[2026-11-20 Interim hardware while the Dex1-1 is delayed]]). Martyn, 2026-11-23: go through the Replay Ladder in order from RL1; the sync test is done when RL5 is reached, not pulled forward. Gripper-side measurements follow in RL6 (Dex1-1 arrives: swap gripper and camera, repeat RL4–RL5).

Sources: arXiv HTML of chi2024 (UMI, Sec. III-B and Appendix A read verbatim 2026-11-23), li2025 (ViTaMIn-B, IV-B and V-D1 verbatim), zhaxizhuoma2025 (FastUMI, III-A and III-B verbatim), yin2026 (RAPID, III-C verbatim), zhao2025 (XRoboToolkit, II-B, II-D, IV-A), the [XRoboToolkit-PC-Service README](https://github.com/XR-Robotics/XRoboToolkit-PC-Service) and the [UMI README](https://github.com/real-stanford/universal_manipulation_interface). Background: [[2026-11-19 UMI-style recording and training pipelines]].

## 1. Why delay compensation matters

- Each stream arrives late by its own delay. Pairing an image with the pose received at the same moment pairs it with a pose taken later than the image.
- ViTaMIn-B Tab. III (weight placement): step 1/2/3 success 0.5/0.3/0.2 without time alignment, 0.9/0.8/0.7 with it.
- UMI Fig. 8 (tossing, deployment side): 0.875 with latency matching vs 0.575 without.
- Worked example (numbers from ViTaMIn-B V-D1: camera 0.14 s, Quest 0.01 s; poses invented for illustration): a frame received at 10.000 s was captured at 9.860 s. Poses received at 9.864 s and 9.878 s were taken at 9.854 s (x = 100 mm) and 9.868 s (x = 107 mm). Interpolated pose at 9.860 s: x = 100 + (6/14) × 7 = 103 mm. Without compensation the frame gets the pose taken about 0.13 s later; at 0.5 m/s that is about 65 mm off.

## 2. Alignment methods in prior work (candidates to compare)

| Method | Source | How | Keeps raw timing |
| --- | --- | --- | --- |
| Delay subtraction + interpolation | UMI PD1.1 (III-B); ViTaMIn-B IV-B | Measure each stream's delay; shift timestamps back; down-sample camera (UMI 10–20 Hz); linearly interpolate pose and width at each image capture time. UMI aligns to "the stream with the highest latency (usually the camera)" | Yes (raw kept) |
| Nearest neighbour on one clock | FastUMI III-A | One ROS clock; sub-sample to 20 Hz (every third 60 fps frame), pair each frame with "the temporally nearest T265 pose"; stores only the pairs (HDF5) | No |
| Approximate sync window | RAPID III-C | Align within a window, "default 25 ms, similar to ROS ApproximateSync"; missing modalities zero-filled | No |

Plan: record once raw (MCAP), align the same recording all three ways offline, compare the residual error (section 5).

## 3. Measuring the delays

### Camera delay (screen clock)
- ViTaMIn-B IV-B: a time-varying ArUco marker on a screen with known display time; each frame's receive time recorded; delay = receive time − display time, per frame.
- UMI App. A.0-A1: same idea with a QR code showing the display time; l_camera = t_recv − t_display − l_display, "subtracting the known latency of display refresh l_display" (value and source not given).
- Steps: (1) the recording computer shows a marker that changes every screen refresh and logs which ID was shown when; (2) the D435i films the screen; (3) per frame, note receive time and decoded ID; (4) delay = receive time − display time of that ID; collect a few hundred frames, keep mean and spread.
- **The screen must be driven by the recording computer** (one clock). Otherwise the clock offset between two computers enters the result.

### Display refresh and lag
- A 60 Hz screen updates every 16.7 ms, so the shown time is uncertain by up to one refresh, plus the panel's own lag. That is above D06's 10 ms target (D06: recording and synchronisation).
- UMI subtracts a "known" l_display (value, method not stated); ViTaMIn-B makes no display correction.
- XRoboToolkit IV-A avoids the screen: an LED panel cycling at 100 Hz, filmed next to the VR display (10 frames per condition, mean and STD).
- Options (none chosen): high-refresh monitor; LED panel as in XRoboToolkit; measure the screen's own lag once (method unverified).

### Controller pose delay
- ViTaMIn-B IV-B: "exchanging synchronization signals between the Quest and the computer", delay = t_rec − t_actual (0.01 s wired, V-D1). Covers transport, not the tracking pipeline (own reading, unverified).
- UMI A.0-A2 (robot proprioception): robot timestamps where available, else "1/2 of ICMP round-trip time".
- XRoboToolkit: no pose latency or end-to-end latency reported in the paper. The PC-Service README gives each pose message a `timeStampNs`, "Timestamp when the data was obtained, Unix (nanoseconds)". Which clock and how it is synced to the PC: not stated. If that clock matches the recorder's, pose delay = receive time − `timeStampNs` (unverified).
- Already logged: `software/xr/trigger_log.py` (dex1-1-handheld) writes the local `time.time_ns()` and the XRoboToolkit timestamp per sample; `trigger_udp.py` sends `xr_ts_ns` and `send_ts_ns`. So existing G1 (trigger value vs travel) and press-test logs give receive − `timeStampNs` directly, but it means a delay only if the clocks are synced (check).
- Motion method (own proposal; adapts UMI A.0-A3 cross-correlation of sinusoidal gripper commands vs measured width; no direct precedent for pose): ArUco marker on the controller, swing it left–right in front of the D435i; marker position from the camera (times corrected by the known camera delay) vs controller position from the pose log; the time shift that best overlaps the two curves (cross-correlation) = pose delay. Example: leftmost point in the camera at 5.42 s − 0.12 s = 5.30 s true; in the pose log at receive time 5.32 s → pose delay 0.02 s. Covers tracking plus transport.

## 4. Other delays and clocks

| Item | Applies to | Method in prior work | When |
| --- | --- | --- | --- |
| Gripper width (Dex1-1 motor state over RS-485) | Handheld and VR | UMI A.0-A3: cross-correlation of sinusoidal width command vs measured width | RL6 (needs the Dex1-1) |
| Trigger | Handheld and VR | Travels in the same XRoboToolkit JSON as the pose, so likely the same delay (unverified) | RL5 |
| Clock offset between computers (headset, station Jetson, Mac) | Any multi-computer setup | Not described in XRoboToolkit; avoid by recording on one computer (D06) | RL5 |
| Robot state (proprioception) | VR baseline, replay, policy runs | UMI A.0-A2: robot timestamps or ½ ping | RL3 onward |
| Robot execution | Replay, policy runs | UMI A.0-A4 (teleoperated trajectories); actions sent ahead, arm 100 ms, gripper 120 ms (Fig. 5) | RL3 onward |

## 5. Things to decide or measure at RL5

- How often to measure: not stated in UMI or ViTaMIn-B. Proposal (no precedent): once per device and connection; again whenever the chain changes (camera, resolution or frame rate, USB port, driver, recording computer); short screen check at the start of each collection day to catch drift. Store mean and spread.
- Store delays as a versioned file per device (D09: calibration cadences); raw MCAP lets old recordings be re-aligned when delays are re-measured.
- Residual error metric: marker position in the image vs controller pose after alignment, per method in section 2. No paper reports residual sync error (gap A5 in [[2026-11-02 Research gaps]]), so reporting it is a contribution.
- Display choice (section 3, display refresh).
- Clock question: is `timeStampNs` on the headset clock, and is it synced to the Jetson?

## 6. Moving to the final hardware (RL6)

- Carries over: method, recorder, alignment code, residual metric, delay-file format.
- Does not carry over: the numbers. Delays are per device and connection (ViTaMIn-B reports GoPro 0.14 s, tactile 0.08 s, Quest 0.01 s separately). Re-run the screen test for the Dex1-1 camera; add the gripper width cross-correlation (UMI A.0-A3); update the delay files; re-align the pilot recordings.

## Prerequisites (from the Replay Ladder and station Jetson plans)

- D435i moved to the station Jetson: CAM1 (plug in and check), CAM2 (test frames), both without installs; CAM3 (recorder venv: numpy, opencv-python-headless, mcap) needs Martyn's go ([[2026-11-19 Station Jetson camera install plan]]).
- A display driven by the station Jetson (or the LED-panel alternative).
- XRoboToolkit PC Service running on the Jetson (`runService.sh`, no autostart).

## Open

- Whether `timeStampNs` is synced to the recorder clock: unverified.
- Display lag value for the chosen screen: unknown.
- Whether the trigger has the same delay as the pose: unverified.
