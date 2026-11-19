---
date: 2026-11-19
tags: [report, jetson, camera, install-plan]
---
# Station Jetson: camera install plan (D435i stand-in)

Goal: read a camera on the station Jetson (Path 2, [[D06 Recording and synchronisation]]) so that the first recorder can save images with the controller pose and trigger on one clock. The team's RealSense D435i moves from the team Pi to the Jetson as a stand-in for the Dex1-1 camera (Martyn, 2026-11-19). Only its colour stream is used: the design is RGB only ([[D03 Depth sensing]]), and UMI and ViTaMIn-B read their cameras directly on the recording device or host ([[2026-11-19 UMI-style recording and training pipelines]]).

## What the Jetson has now (read-only check over SSH, 2026-11-19)

- L4T R36.5.2, kernel 5.15.199-tegra; 30 GB free on eMMC; user `martyn` is in the `video` group.
- Present: v4l-utils 1.22.1 (`v4l2-ctl`), GStreamer 1.20 with plugins base, good and ugly (`gst-launch-1.0`), OpenCV C++ libraries (NVIDIA 4.8.0 and Ubuntu 4.5.4), libjpeg-turbo 2.1.2, ROS 2 Humble.
- Missing: OpenCV for Python (`import cv2` fails), librealsense, ffmpeg, numpy in the venv. No camera plugged in (no `/dev/video*`).

## Steps

| Step | What | Installs | Check | Undo |
| --- | --- | --- | --- | --- |
| C1 | Plug the D435i into a USB 3 port (blue) of the Jetson | nothing | `lsusb` lists Intel 8086:0b3a; `v4l2-ctl --list-devices` shows its video nodes; `--list-formats-ext` lists a colour format (YUYV expected, unverified on this kernel) | unplug |
| C2 | Grab frames with the tools already there | nothing | `v4l2-ctl -d /dev/videoN --stream-mmap --stream-count=300` reports the frame rate; one frame saved to a JPEG with `gst-launch-1.0` and copied to the Mac | delete the test files |
| C3 | Python environment for the recorder: new venv `~/venvs/recorder` with `numpy`, `opencv-python-headless` and `mcap` from PyPI (https works on the lab network) | 3 Python packages, in the venv only, no sudo | `python -c "import cv2, mcap"`; 10 s of colour frames read with OpenCV, kernel frame timestamps logged | `rm -rf ~/venvs/recorder` |

**Not now:**
- **librealsense / pyrealsense2.** Only needed for depth or the D435i's IMU, neither of which the design uses; on Jetson it needs a source build or NVIDIA's repo.
- **teleimager.** Unitree's image streamer. It sends video to another computer (useful for the VR-teleop view, task 7), not for recording on the Jetson itself. It also needs libjpeg-turbo ≥ 3.0 while the Jetson has 2.1.2 ([[2026-11-13 dex1_1_service code and Pi 5 fit]]).

## Risks

- The D435i's colour stream on the Tegra kernel without RealSense kernel patches is **unverified**. If C1 shows no colour node, fall back to any plain USB webcam for the recorder tests (thesis plan §6) and keep the D435i on the Pi.
- Whether the Dex1-1 camera is a plain UVC camera stays unverified until delivery. A V4L2/OpenCV reader covers it if it is.

## Approval

C1 and C2 install nothing. C3 adds three Python packages (new dependencies), so it waits on Martyn's go.
