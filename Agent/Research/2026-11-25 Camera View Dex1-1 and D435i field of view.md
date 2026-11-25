---
date: 2026-11-25
tags: [research, camera, dex1-1, d435i]
---
# Camera View: Dex1-1 and D435i field of view

Topic name: **Camera View**. Question (Martyn, 2026-11-25): how much of the gripper and the task part does each wrist camera see? (1) Dex1-1 camera: check whether Unitree's simulation repos model it, and render its view in MuJoCo. (2) D435i on the UR10e flange (with the Robotiq 2F-85): look at a live frame from the station Raspberry Pi on the Mac.

Sources: Unitree product page, `unitreerobotics/unitree_ros` (BSD-3), `unitreerobotics/unitree_mujoco`, `unitreerobotics/unitree_sim_isaaclab`, Intel RealSense pages. No deep research needed. Status: **checked against Unitree's CAD files; not checked against a real Dex1-1 camera** (grippers not delivered).

## 1. Dex1-1 camera: what the sources give

| Item | Value | Source |
|---|---|---|
| Resolution, frame rate | 1080p, 30/60 fps | [Unitree Dex1-1 page](https://www.unitree.com/Dex1-1/) |
| Field of view | H 118°, V 87°, D 140° | [Unitree Dex1-1 page](https://www.unitree.com/Dex1-1/) |
| Field of view (resellers) | "FOV 160°" (no H/V split) | [RoboStore V2](https://robostore.com/products/unitree-dex1-1-v2-advanced-gripper-rgb-camera), [Scan](https://www.scan.co.uk/products/unitree-dex-1-1-plus-robotic-gripper-right-hand-for-g1-humanoid-2-fingers-1-degree-of-freedom-rgb-ca); conflicts with Unitree, not used |
| Sensor, lens model, distortion | not published | none found |
| Mount pose on the gripper | camera link 68.7 mm forward, 0 mm sideways, 47.5 mm up from the gripper base frame; turned 90° about x, then tilted 30° about y | `dex1_1.urdf`, joint `Camera`, [unitree_ros commit 8927c58](https://github.com/unitreerobotics/unitree_ros/commit/8927c58e133b5e234e78b8581016f9a4867e9e1c) (2026-10-30) |
| Camera shape | `meshes/Camera.STL` (camera block with lens) | same commit |

New since the 2026-10-28 check: Unitree's commit of 2026-10-30 added a `Camera` link and mesh to the Dex1-1 URDF. The older note "no camera link in it" (memory, [[2026-10-28 Handheld design start plan]]) is out of date. Whether the official URDF 261030 in `dex1-1-handheld/hardware/from-official-docs/` (local only) has the same camera link was not checked (Mac offline at the time).

**unitree_mujoco has no Dex1-1.** A search of the repo (commit 1eb6642, 2026-09-07) finds no Dex1-1 files and no `<camera>` element in any robot model; its H2 model has no grippers. Unitree's Dex1 simulation with cameras is `unitree_sim_isaaclab` (G1 + Dex1 tasks, `left/right_gripper_wrist_camera`, 640 × 480, focal 12 mm, aperture 20 mm ≈ 80° horizontal by own calculation; [repo](https://github.com/unitreerobotics/unitree_sim_isaaclab), `tasks/common_config/camera_configs.py`). It needs Isaac Sim, Linux and an NVIDIA GPU, so it does not run on the Mac, and its 80° does not match the 118° product figure. Not used here.

**Which way the lens looks** (from the mesh, see picture `figures/2026-11-25 Dex1-1 camera mount axes.png`): the lens sits on the camera link's +x face. With the URDF angles (read as fixed-axis roll, pitch, yaw, the URDF convention), +x points forward along the fingers and 30° down towards them. So the camera sits on top of the gripper body, just behind the finger base, and looks along the fingers at the tips.

## 2. First rendered view (cloud, MuJoCo 3.x)

![[2026-11-25 Dex1-1 camera view sim.png]]

Setup: Unitree meshes and camera pose from the URDF; gripper pointing straight down; task parts from `msc-thesis/docs/hardware-specs.md` (metal sleeve OD 27.16 mm, 59.70 mm tall; bushing OD 52.77 mm, 60 mm tall, placed 120 mm to the side); table squares 50 mm. Camera: 16:9 image, vertical FOV 87°, which gives 118.7° horizontal in a pinhole model, matching Unitree's H 118°. Panels: (1) open, 60 mm above the grip; (2) open, at grip height; (3) closed on the sleeve; (4) outside view.

What it shows (simulation only):
- Both finger tips are in the lower half of the image in all states; the closed fingers fill the lower-centre third.
- The sleeve top stays in the image centre between the fingers; the bushing 120 mm away is still inside the frame.

Limits:
- MuJoCo draws a pinhole (straight-line) camera. The real lens reaches D 140°, wider than the pinhole's 126° diagonal (own calculation), so the real image shows more at the corners and bends straight lines (fisheye-like). Lens model unknown until the camera arrives.
- Finger opening: joint value 0 = closed (pads touch), 0.041 m per finger = fully open (82 mm), read from the URDF limits and the render.
- Files: `/mnt/project-files/camera-view/` (`scene.xml`, `render.py`, picture). Not yet in a repo.

## 3. Options for running it on the Mac

Assuming MuJoCo already runs on the Mac from RL1 (replay in MuJoCo on the Mac), from [[2026-11-23 RL1 replay in MuJoCo first run]].

| Option | What | Needs | Precedent |
|---|---|---|---|
| A. Picture script | `render.py` saves the four pictures as above | MuJoCo + Pillow in the RL1 environment (Pillow presence unchecked); Unitree meshes downloaded | MuJoCo offscreen `Renderer` (official Python bindings) |
| B. Live viewer | Open `scene.xml` in MuJoCo's viewer, pick camera `dex1_cam`, move the gripper and fingers with the mouse/sliders | Nothing beyond MuJoCo | MuJoCo `simulate` viewer |
| C. Replay with camera | Add the Dex1-1 + camera to the RL1 UR10e scene and record the camera view during a replayed demonstration | Attach to the UR10e flange model; adapter offset from Fusion | UMI-style wrist camera policies (gripper camera as the only view) |

Recommendation: B first (no install, quick look), then C once the adapter offset is fixed, because C shows the view along real recorded motions. Not run on the Mac yet: needs Martyn's go.

## 4. D435i on the flange: live frame on the Mac

Known: the D435i is plugged into the station Raspberry Pi 5 ("arena"); the Pi has `wrist/realsense_streamer.py` (ZMQ/JPEG stream, 848 × 480 at 30 fps per [[2026-11-12 Lab computers and gripper connections]]) and `wrist/fov_check.py` (contents not read) ([[2026-11-12 Raspberry Pi vs lab PC for the Dex1-1]], section "Station Pi inspected").

D435i colour camera: 69° × 42° (H × V), 1920 × 1080, 30 fps ([intelrealsense.com D435i page](https://intelrealsense.com/depth-camera), table headed D435); D400 colour DFOV 77°, ±3° between units ([Intel support article 000030385](https://intel.com/content/www/us/en/support/articles/000030385.html)). So the D435i sees a much narrower picture than the Dex1-1 camera (69° vs 118° across).

| Option | What | Needs | Note |
|---|---|---|---|
| A. One still picture | On the Pi, take one frame with the existing `wrist/` venv (or `fov_check.py` if it does this), copy it to the Mac with `scp`, open in Preview | SSH Mac → Pi; nothing installed | Simplest; enough to see how much of the Robotiq is visible |
| B. Live stream | Start `realsense_streamer.py` on the Pi; a small viewer on the Mac shows the ZMQ/JPEG frames | `pyzmq` + `opencv-python` on the Mac (new dependencies) | Moving view, but needs installs |

Recommendation: A, after a read-only look at `fov_check.py` and `realsense_streamer.py`. Only one program can open the camera at a time, so nothing else may be using it (ask the team if the Pi stream is in use).

## Open

- Martyn's go to (a) look read-only at the Pi's `wrist/` scripts from the Mac, (b) run option B (MuJoCo viewer) on the Mac.
- Check whether the local official URDF 261030 has the same `Camera` joint.
- Real Dex1-1 camera: measure FOV and lens distortion on arrival (RL6 (Dex1-1 swap)); the 160° reseller figure stays unverified.
