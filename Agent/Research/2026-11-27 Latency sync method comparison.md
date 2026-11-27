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
- Best method precedent: Morehead et al. 2023 (rail + 240 fps camera + LED, 21-42 ms on Vive/Rift/Index, about 4 ms uncertainty), no code.
- Related: Hu et al. 2025 (UDP clock offset, code Duke-I3T-Lab/XR_Tracking_Evaluation, headset only); Magic Leap 2 controller on UR10e vs OptiTrack (arXiv 2509.05391, accuracy only).

## Robotiq gripper facts
No latency number from any source. Manual: 200 Hz usual rate, 5 ms between commands. Must be measured.

## Not checked
UMI paper appendix A; ser2net timing; whether bib papers kuhlmanndecanaviri2023 and kulozik2024 state delays (kulozik: about 10 ms estimated, not measured).

## Add to Zotero (not in bib)
Morehead 2023 (10.3758/s13428-022-01983-5); ExtremControl (arXiv 2602.11321); Friston and Steed 2014; VRLate (arXiv 1809.06320); Hu 2025 (arXiv 2508.08642); Magic Leap 2 study (arXiv 2509.05391); Robotiq 2F-85 manual; XRoboToolkit paper (arXiv 2508.00097).
