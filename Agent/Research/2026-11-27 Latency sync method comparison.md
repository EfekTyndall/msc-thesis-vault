---
type: research
date: 2026-11-27
---
# Latency sync method comparison (fresh start, 2026-11-27)

Replaces the scrapped sync explainers (now in `Agent/Archive/`). Deep research was used for the controller part (three research passes, web + public code read, nothing run).

## Plain version
The question: once each stream's delay is known, how do the streams get lined up? Prior work does it in one of three ways. Recommended: measure each delay once, keep it as a number in the calibration file, and at export move every stream back by its number so all match the camera frame times (UMI, DexUMI, ViTaMIn-B all do this). The open part is how to measure the PICO controller delay: no prior work or code does that.

## Options
| Option | Who does it | How it works | Source |
|---|---|---|---|
| 1. Measure once, store numbers, shift at export | UMI, DexUMI, ViTaMIn-B | Camera, gripper, robot each get a delay in a config; at export each stream is moved back by its delay and read at the camera frame times | UMI umi.yaml l.3-5, sampler.py l.131-151; DexUMI data_buffer.py l.151-161; ViTaMIn-B task_config.yaml l.55-57 |
| 2. No measurement, pair by timestamp | FastUMI | Nearest sample by timestamp; correct only if all streams have the same delay | FastUMI data_collection.py l.235, 255 |
| 3. Estimate from each recording's own motion | ExtremControl (VR teleop, 54 to 64 ms end to end) | Optical flow of human and robot hand, offset that best aligns them | arXiv 2602.11321; no code found |

## Plug and play check (Martyn, 2026-11-27: Dex1-1 gripper and other camera must swap in)
- Option 1: yes. One delay number per device in the calibration file; the alignment step only reads the numbers and never knows the device. A new device needs only its own measurement. Same shape as UMI's config (one entry each for camera, gripper, robot).
- Option 2: yes, but no numbers at all, so a swapped device with a different delay is silently wrong.
- Option 3: no. Needs a matching signal pair per device type (human hand vs robot hand), so the alignment step becomes device-specific.

## Controller (PICO) facts, checked in XRoboToolkit code
- Pose has no capture time. Only `timeStampNs` on the headset clock, set when the packet is built (TrackingData.cs l.155-156). Poses are predicted to display time (l.314-338). So the delay and the clock offset must be measured, not read.
- No VR teleop repo found that estimates or compensates controller delay. No paper measures PICO controller delay.
- Best method precedent: Warburton et al. 2023 (rail + 240 fps camera + LED, 21-42 ms on Vive/Rift/Index, about 4 ms uncertainty), no code.
- Related: Hu et al. 2025 (UDP clock offset, code Duke-I3T-Lab/XR_Tracking_Evaluation, headset only); Magic Leap 2 controller on UR10e vs OptiTrack (arXiv 2509.05391, accuracy only).

## PICO pass 2 (2026-11-27, deep research on Opus: PICO docs, forums and GitHub issues, papers)
- Result: no measured PICO controller pose latency anywhere: not in PICO specs or SDK docs, not in XRoboToolkit or xr_teleoperate issues, not on Reddit or the PICO forum, not in papers.
- PICO SDK: `GetControllerPredictPosition(controller, predictTime)` takes ms; 0 means no prediction (native header PxrInput.h). Pose struct has `poseTimeStampNs` ("time to obtain the data"), clock not documented. Prediction model closed source. No PICO latency test tool.
- Closest PICO numbers are end to end, operator to robot, so they include the robot: HuMiT (arXiv 2610.29903, PICO 4 Ultra) 65 ms median, 200 fps camera + optical flow, about 20 ms packet-to-action from internal stamps. XRoboToolkit (zhao2025) reports video latency only (82 ms ZED Mini to PICO 4 Ultra).
- Warburton et al. 2023: delay differs at movement start (21-42 ms) and in steady motion (2-13 ms) because of prediction, so one fixed number may not fit; for PICO unverified.
- Closest precedent for a controller held by a robot: Banaszczyk 2024 (UR5e replay, motion peaks matched to remove a 0-200 ms clock offset; accuracy only).

## Robotiq gripper facts
No latency number from any source. Manual: 200 Hz usual rate, 5 ms between commands. Must be measured.

## Not checked
UMI paper appendix A; ser2net timing; whether bib papers kuhlmanndecanaviri2023 and kulozik2024 state delays (kulozik: about 10 ms estimated, not measured).

## Add to Zotero (not in bib)
Warburton et al. 2023 (10.3758/s13428-022-01983-5); ExtremControl (arXiv 2602.11321); Friston and Steed 2014; VRLate (arXiv 1809.06320); Hu 2025 (arXiv 2508.08642); Magic Leap 2 study (arXiv 2509.05391); Robotiq 2F-85 manual. (XRoboToolkit paper is already in the bib as zhao2025.)

## Relative to PICO (Martyn's question, 2026-11-27)
Question: can everything be lined up to the PICO pose so the PICO delay never has to be known? Yes for the camera: rigidly mounted camera and controller turn at the same rate, so matching their rotation speed gives the camera-vs-PICO offset directly, without either absolute delay and without the clock offset.
- Precedent with code (read, not run): ethz-asl/hand_eye_calibration (Furrer et al. 2017), `python/hand_eye_calibration/time_alignment.py`: both pose streams resampled (l.124-180), rotation speed per sample (l.103-121), clip + smooth (l.23-44), cross-correlate and take the peak (l.47-60). Whole samples only. Kalibr (Furgale et al. 2013) adds a fine continuous step (IccSensors.py l.217-274, 326-351); ExtremControl's `cross_correlation` adds a parabola step (math_utils.py l.713-750). Banaszczyk 2024 code (head_trajectory, compare_traj_peaks.py l.73-119): peak matching on one axis.
- Camera needs its own rotation: from a fixed marker board or visual odometry. Works for any camera (plug and play).
- Gripper: PICO does not see the gripper width; no motion-based precedent for gripper vs camera or PICO. Gripper keeps its own measured number (UMI gripper test), placed on PICO time.
- Accuracy for us unverified: published sub-ms figures use 200 Hz IMUs; a 30 Hz camera gives about 33 ms steps before a fine step. Needs turning motion. Controller prediction may change the offset between movement start and steady motion (Warburton 2023), unverified for PICO.

| | Each device on its own | Relative to PICO |
|---|---|---|
| PICO delay needed | yes, no precedent to measure it | no |
| Headset clock offset | must be handled separately | absorbed |
| Camera | screen test (UMI) | rotation matching (hand_eye_calibration code) |
| Gripper | own number (UMI test) | own number (same) |
| New camera / Dex1-1 | new number each | rerun matching / new gripper number |
