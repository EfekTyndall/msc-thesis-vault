---
date: 2026-11-24
tags: [report, replay-ladder, procedure, safety, robotiq]
---
# RL4 (Robotiq pick-and-place during replay): procedure

RL4 adds the gripper to the replay: the recorded trigger opens and closes the Robotiq 2F-85 while the UR10e replays the handheld movement, first in the air, then as one pick-and-place of a light, soft object. Scope agreed 2026-11-24: binary open/close only; G2 (trigger-to-gripper mapping) is decided in RL6 (swap to the Dex1-1). The Robotiq plug of SwapKit option A ([[D07 Software and data format]]) is built here. Earlier steps: [[2026-11-23 RL3 real UR10e replay procedure]].

## 1. The team's Robotiq driver (checked 2026-11-24)

Read in the team repo at the pinned commit `7e6c754` (`~/thesis/team/xr_teleoperate`, same commit on fractal-pc), file `teleop/robot_control/robotiq_2f85_node.py` (team, 2026-09-28/29).

| Question | Answer | Evidence |
| --- | --- | --- |
| Open/close only? | **The node yes, the driver class no.** The node sends only open (0) or "closed" (215 of 255, stops just before the pads meet). The class inside it, `RobotiqModbusGripper`, takes any position 0.0 (open) … 1.0 (closed) and reads the measured position back. Speed 255 and force 100 (of 255) are fixed in the file. | `set_position()`, `position()`, `CLOSED = 215/255`, `SPEED, FORCE = 255, 100` |
| How it connects | **Not through the UR tool connector or a URCap.** Modbus RTU (Robotiq's serial message format) sent as bytes over the wired network to the station Pi (`arena.local`, 10.36.11.9), port 4001. The Pi's `ser2net` passes them to a USB-to-RS-485 adapter and the gripper. Gripper ID 10. Power: 24 V supply under the station. | Node docstring and `connect()`; Pi config `/etc/ser2net.yaml` (115200 baud, `kickolduser: true`), read 2026-11-12 ([[2026-11-12 Lab computers and gripper connections]]). The first version (2026-09-28) used a URCap socket; replaced the next day. |
| Which host runs it | Any computer on the lab network; it is a client of the Pi. In the supervisor's teleop it is started by `teleop/run_ur10e_teleop_real.sh` on the teleop computer (the Jetson, assumed in the 12 Nov note). | Launch script |
| Can fractal-pc use it? | **Yes, without installing anything.** fractal-pc reaches `arena.local` (ping 0.2 ms) and the class imports in the existing conda env `ursim-replay` (only numpy needed). Port 4001 was not opened (see the warning below). | Checked 2026-11-24 over SSH |
| Trigger rule in the team teleop | Right trigger ≥ 0.9 closes, back below 0.9 opens; one threshold, no hysteresis. | `teleop_bindings.py` (`analog_push("R_trigger")`), `controller_events.py` (`trigger_threshold = 0.9`) |

**Warning, one user at a time:** `kickolduser: true` means a new connection to port 4001 throws out the old one. If the supervisor's teleop is running, the RL4 program would cut her gripper off, and the other way round. Check that nobody uses the robot before each session (as in RL3).

**Activation stroke:** if the gripper is not yet activated (after a power cycle), connecting makes it close fully and reopen once. Keep hands and objects out of the fingers when the program starts.

## 2. What gets built (dex1-1-handheld PR, before the lab session)

No new dependencies. The team class is imported from the team repo (outside dependency at `7e6c754`, no copied code).

| Part | What it does |
| --- | --- |
| Gripper plug, Robotiq | Three functions as agreed for SwapKit: set width (m), set force limit, read width (m). Width in metres converted to the 0…255 position scale with a straight line over the 85 mm stroke ([Robotiq 2F-85](https://robotiq.com/products/adaptive-grippers), not in bib); the true width per step is unverified and not needed for open/close. Set force limit: fixed at the team value (100 of 255) in RL4, because the team class has no force setting. |
| Settings file per gripper | `robotiq_2f85.json`: Pi address and port, gripper ID, stroke 0.085 m, closed position 215/255, force 100. A `dex1_1.json` (≈ 0.082 m) follows in RL6. JSON, so no new library. |
| Shared gripper program (first part) | Turns the recorded trigger into open/close with the team rule (≥ 0.9 = close), so handheld and VR baseline use the same rule (G2 fairness rule, [[2026-11-10 Research gaps from the PICO sim test]]). Logs command time and measured width. |
| Replay change | `ur10e_replay_rtde.py --gripper robotiq_2f85`: gripper commands run in their own thread, so the 125 Hz arm loop is never held up by a network reply. MuJoCo replay (RL1) prints the open/close times, so they can be checked on the Mac first. |
| Tip offset | `--tip-offset <file>`: the robot follows the movement of the handheld fingertip instead of the controller centre (D09 (calibration)). Needed because the robot's tool point is already at the Robotiq fingertip (team IK: 0.174 m along the flange axis, "copy from PolyScope"), while the recording is the controller's centre; any wrist turn would otherwise move the tip wrongly. |

## 3. Procedure

Martyn can only reach the table in front of the robot; every step below is done from there or from the Mac.

### Step 0. Inputs (before the lab day)

| # | Item | Who | How it is checked |
| --- | --- | --- | --- |
| 0.1 | D09 (calibration) tip offset: controller tracked point → dummy Dex1-1 fingertip midpoint, from the Fusion assembly, saved as a versioned file | Claude reads Fusion (the Fusion connection failed on 2026-11-24, retry), Martyn confirms | The PICO's tracked point inside the controller is not published (unverified, [[D09 Calibration]] "Revisit if"). Optional check without CAD: pivot test, fingertip held on one point while the handheld is tilted around it; the point that stays still gives the offset ([Yaniv 2015](https://doi.org/10.1117/12.2081348), not in bib). |
| 0.2 | Robot tool point: read PolyScope Installation > TCP; must match 0.174 m | Martyn at the pendant | Number written in this note |
| 0.3 | Height of the fingertip above the table at HOME_Q (team comment says ≈ 0.25 m) | Martyn, ruler | Number written in this note; sets the allowed box in step 4 |
| 0.4 | PR merged (plug, settings file, replay change), dry run on the Mac with a synthetic take | Claude, Martyn merges | MuJoCo replay prints the gripper events at the right times |

### Step 1. G1 (trigger value vs travel) and G8 (does pressing move the controller)

As written in the Daily TODO of [[2026-11-24]] (station Jetson, `trigger_log.py`, caliper steps up and down; then the press test on the table and in the hand). G1 also checks that the trigger reliably reaches 0.9; if a full squeeze stays below 0.9, the threshold is lowered before step 3. No robot needed.

### Step 2. Gripper alone (robot does not move)

1. Check nobody uses the robot or the supervisor's teleop; pendant can stay in Local.
2. Fingers clear. From the Mac: `ssh fractal-pc`, env `ursim-replay`, run the plug's test (open, close, five times; read width each time).
3. **Pass:** the gripper follows each command; read width ≈ 85 mm open and ≈ 0 when closed empty (closed position 215/255 stops just short of the pads); no fault message.

### Step 3. RL4a: replay with the gripper, in the air

1. Record a small take on the station Jetson (`pose_log.py`, A button start/stop): slow moves of about 8–10 cm like RL3 take-02, with two or three full trigger squeezes and releases.
2. On the Mac: MuJoCo dry run with `--tip-offset`; box check passes; gripper events printed.
3. On the robot (RL3 safety layers unchanged: 0.3 rad/s, allowed box, Martyn at the emergency stop, Remote Control only while replaying): `ur10e_replay_rtde.py --gripper robotiq_2f85 --tip-offset …`.
4. **Pass:** the gripper closes and opens once per squeeze, in the same order as recorded; arm tracking as in RL3 (no protective stop); the log shows command times and when the measured width started to change.

### Step 4. RL4b: one pick-and-place

**Table layout** (agreed 2026-11-20, [[2026-11-20 Interim hardware while the Dex1-1 is delayed]]): tape outlines for the pick spot and the place spot, about 10 cm apart, both within reach from the table side and of the robot. The handheld must start at the same point as the robot's fingertip at HOME_Q, which is in the air. Proposal: a **start stand**, a box of the height from 0.3 with a cross on top, placed on a taped outline under the robot's fingertip at HOME_Q; the handheld's fingertip starts on the cross; the stand is removed before each replay (same idea as ActiveUMI's docking jig, zeng2025 Sec. 3.3).

**Object:** light and soft, about 5–6 cm tall and narrower than 7 cm (for example a foam block or sponge), so a wrong height or a hard close cannot damage the gripper or table.

**Allowed box:** the pick needs the tip to go down almost to the table, more than the RL3 limit of 5 cm. Proposal: allowed depth = fingertip height above the table at HOME_Q (0.3) minus 3 cm, so the tip stays at least 3 cm above the table; set with `--box 0.15 <depth> 0.15`. Sideways and up limits unchanged.

1. Recentre the headset while facing the robot squarely (the replay assumes the operator faced the robot at recentre, [[2026-11-23 RL1 replay in MuJoCo first run]]).
2. Object on the pick outline. Handheld fingertip on the start-stand cross; press A to start; move to the object, squeeze, lift about 5 cm, move to the place outline, lower, release, lift away; press A to stop.
3. Mac dry run (box check, gripper events). Remove the start stand; object back on the pick outline; robot at HOME_Q.
4. Replay on the robot as in step 3, with the new `--box` depth.
5. Repeat the replay three times with the same take, putting the object back on the pick outline each time.

**Pass (proposal):** in each of the three replays the robot lifts the object and leaves it inside the place outline. Record per run: picked yes/no, placed yes/no, distance of the object centre from the place-outline centre (mm, ruler). If the tip misses the object by more than about 1 cm, D09 (calibration) "Revisit if" applies: check the tip offset (pivot test) before anything else.

## 4. Open

- Trigger threshold 0.9 (team rule) is kept unless G1 shows a full squeeze does not reach it.
- Start stand, object size, allowed depth and the three-run pass rule are proposals; Martyn decides.
- The Robotiq position step in mm and the grip force at setting 100 are not checked in the manual (unverified); not needed for open/close.
- Tip offset file format and version naming follow D09 (calibration); defined in the PR.

## Add to Zotero

- Z. Yaniv, "Which pivot calibration?", Proc. SPIE 9415, Medical Imaging 2015, [doi:10.1117/12.2081348](https://doi.org/10.1117/12.2081348): standard method to find a tracked tool's tip from rotations about a fixed point; optional check of the D09 CAD offset.
