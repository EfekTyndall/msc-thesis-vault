---
date: 2026-11-24
tags: [report, replay-ladder, procedure, safety, robotiq]
---
# RL4 (Robotiq pick-and-place during replay): procedure

RL4 adds the gripper to the replay: the recorded trigger opens and closes the Robotiq 2F-85 while the UR10e replays the controller movement, first in the air, then as one pick-and-place of a light, soft object. Scope agreed 2026-11-24: binary open/close only; G2 (trigger-to-gripper mapping) is decided in RL6 (swap to the Dex1-1). The Robotiq plug of SwapKit option A ([[D07 Software and data format]]) is built here. Earlier steps: [[2026-11-23 RL3 real UR10e replay procedure]].

**Change 2026-11-24 (Martyn): RL4 uses the bare PICO controller, without the handheld prototype.** The "fingertip" is then a chosen point on the controller, and its offset from the tracked point is read in Fusion (step 0.1, section 1b).

Read on the pendant (Martyn, 2026-11-24): TCP (X, Y, Z) = 0, 0, 174 mm, rotation 0, 0, 0 (matches the team IK value 0.174 m); payload 1.770 kg, centre of gravity (9, 26, 40) mm.

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

## 1b. Where the controller's tracked point is (checked 2026-11-24)

The PICO developer docs ([input mapping](https://developer.picoxr.com/document/unity/input-mapping/), Martyn's link) give buttons only, no coordinate system or origin. The answer is in the PICO Unity SDK code ([PICO-Unity-Integration-SDK](https://github.com/Pico-Developer/PICO-Unity-Integration-SDK), default branch, read 2026-11-24):

| Finding | Evidence |
| --- | --- |
| The XRoboToolkit headset app sends the PICO SDK's own controller pose | Unity client `TrackingData.cs`: `PXR_Input.GetControllerPredictPosition/Rotation` (commit `cdc5316`) |
| The SDK ships a 3D model of the PICO 4 Ultra controller | `Assets/Resources/Controller/PICO 4U/Animation/mesh/o_com_PICO4U_right_01.fbx` (≈ 750 kB) |
| That model is drawn exactly at the tracked pose: no shift, only a 180° turn about the vertical axis | `PICO 4U R.prefab` (position 0, 0, 0; rotation y = 180°); `PXR_ControllerLoader.cs` places it with no offset |

Consequence: the origin of that model file is the tracked point. Lined up with our controller scan in Fusion, it gives the tracked point inside the controller, and the tip offset follows from CAD (D09 (calibration)). Assumption (unverified): PICO's own model is placed at the same pose the app sends; the RL4b pick checks it. Licence: the SDK is proprietary PICO material ("reproduction … strictly forbidden" without permission), so the model is used locally for measuring only, never committed to Git or shared.

Done 2026-11-24: model downloaded (SDK commit `c93a59b`) to `~/thesis/local/pico-4u-official/` (outside all repos) and converted to `pico4u_right_official_mm.stl` (mm; origin = tracked point; 2 472 points, 4 731 triangles; FBX in cm). Size check against our scan (main extents): official 137.2 / 68.5 / 61.6 mm, scan 138.2 / 69.3 / 57.6 mm. Automatic shape matching (ICP) did **not** settle the position: best fits leave 3 mm median gaps and two near-equal fits put the tracked point about 10 mm apart (whole-model fit, both ways; fixed below).

Refined line-up (2026-11-24, replaces the first by-eye fit A, which was rotated wrongly): every point of our scan fitted onto PICO's model surface (point-to-surface ICP, 200 000 surface samples, scan → model so the parts missing from the scan do not matter). Result: scan points lie on the PICO model within **0.22 mm median, 0.45 mm for 95 %**; eight restarts from shifted and turned starting poses all end at the same pose (spread 0.0 mm). Checked by eye in Fusion (unsaved "Untitled" document): thumbstick, buttons and handle coincide. **Tracked point:** inside the controller, 5.5 mm below the nearest surface, 56 mm below the end of the button head and 75 mm above the bottom of the handle (138 mm long); in the scan frame (3.78, −1.59, 4.76) mm. Transform files (local only): `~/thesis/local/pico-4u-official/refined_official_to_scan.npz`, model in scan frame `fit_refined_official_on_scan.stl`.

Handheld assembly check (2026-11-24, `ASM real-path v18`, read only, nothing changed): the scan mesh inside `controller-grip v22` → `controller-scan` uses exactly the scan file's coordinates, and the grip sleeve parts share its placement. Because the PICO model fits the scan to 0.22 mm, **no part has to be moved or reshaped**; only the tracked point has to be added. Tracked point in the assembly frame: (−13.2, −112.8, −51.0) mm. Dex1-1 finger pads (`手指防滑垫` 1 and 2, as placed in the assembly): x ±22.3…26.1, y 95.3…144.8, z ±13.4 mm; midpoint between the pad centres (0, 120.0, 0) mm, so tracked point → pad centre (13.2, 232.8, 51.0) mm, length 239 mm (assembly axes). Pose axes (derived from code 2026-11-24, not measured): the XRoboToolkit app sends the PICO runtime's own pose (`Pxr_GetControllerTrackingState`, no Unity conversion in `PXR_Plugin.cs`; the app never calls `SetControllerOriginOffset`). Unity draws the model with: FBX import (x mirrored, `bakeAxisConversion: 0`), node turn −90° about x, prefab turn 180° about y, and Unity = runtime pose with z mirrored. Chaining these gives, for a model point g: pose-frame coordinates = (g_x, g_z, −g_y), a proper rotation (determinant +1). So the pose's −z axis runs along the controller's length toward the button head. **Offset tracked point → Dex1-1 pad centre, in the pose frame: (38.0, 37.4, −232.7) mm** (model frame (38.0, 232.7, 37.4) mm). The chain of axis turns is read from code, not measured; the first handheld replay checks it.

Fusion (2026-11-24, Martyn's OK): construction point and sketch "PICO tracked point" added to component `controller-scan` in `controller-grip` (opened as v24), at (3.777, −1.585, 4.759) mm in the scan's coordinates. Not saved yet when added; `ASM real-path v18` still references `controller-grip v22`, so the assembly shows the point only after its reference is updated.

PICO model in the CAD (Martyn, 2026-11-24: "add it anyway"): mesh "PICO 4 Ultra controller R (PICO official model, local use only, not for sharing)" added to `controller-scan` in `controller-grip` (after v25), placed on the scan with the refined fit. Licence consequence: the Fusion project must not be passed on while it contains this mesh. Scan vs PICO model where the sleeve sits (scan coordinates y 12…70 mm): median 0.18 mm, 95 % within 0.38 mm, largest 1.04 mm, no size difference (signed median −0.05 mm); 3.9 % of the PICO surface there has no scan point within 1 mm. Recommendation: no redraw of the sleeve sketches (differences far below the gain a redraw could bring; the printed sleeve already fits); new parts are drawn on the PICO model. **Agreed (Martyn, 2026-11-24):** PICO model is the controller shown in the assembly, new parts are drawn on it, existing sleeve sketches stay on the scan. Saved as `controller-grip v26`.

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
| 0.1 | Tip offset for the bare controller: tracked point (from PICO's controller model lined up with our scan in Fusion, section 1b) → a chosen, marked point on the controller, read in Fusion and saved as a versioned file | Claude in Fusion, Martyn confirms the marked point | No separate test (pivot test dropped, Martyn, 2026-11-24). Reason: a point error e shifts the robot tip by about e × wrist-turn angle (5 mm × 30° ≈ 3 mm; × 90° ≈ 8 mm), small against the 85 mm Robotiq opening. The RL4b pick is the check; a miss of more than about 1 cm triggers D09 (calibration) "Revisit if". |
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

**Table layout** (agreed 2026-11-20, [[2026-11-20 Interim hardware while the Dex1-1 is delayed]]): tape outlines for the pick spot and the place spot, about 10 cm apart, both within reach from the table side and of the robot. The controller tip must start at the same point as the robot's fingertip at HOME_Q, which is in the air. Proposal: a **start stand**, a box of the height from 0.3 with a cross on top, placed on a taped outline under the robot's fingertip at HOME_Q; the controller tip starts on the cross; the stand is removed before each replay (same idea as ActiveUMI's docking jig, zeng2025 Sec. 3.3).

**Object:** light and soft, about 5–6 cm tall and narrower than 7 cm (for example a foam block or sponge), so a wrong height or a hard close cannot damage the gripper or table.

**Allowed box:** the pick needs the tip to go down almost to the table, more than the RL3 limit of 5 cm. Proposal: allowed depth = fingertip height above the table at HOME_Q (0.3) minus 3 cm, so the tip stays at least 3 cm above the table; set with `--box 0.15 <depth> 0.15`. Sideways and up limits unchanged.

1. Recentre the headset while facing the robot squarely (the replay assumes the operator faced the robot at recentre, [[2026-11-23 RL1 replay in MuJoCo first run]]).
2. Object on the pick outline. Controller tip on the start-stand cross; press A to start; move to the object, squeeze, lift about 5 cm, move to the place outline, lower, release, lift away; press A to stop.
3. Mac dry run (box check, gripper events). Remove the start stand; object back on the pick outline; robot at HOME_Q.
4. Replay on the robot as in step 3, with the new `--box` depth.
5. Repeat the replay three times with the same take, putting the object back on the pick outline each time.

**Pass (proposal):** in each of the three replays the robot lifts the object and leaves it inside the place outline. Record per run: picked yes/no, placed yes/no, distance of the object centre from the place-outline centre (mm, ruler). If the tip misses the object by more than about 1 cm, D09 (calibration) "Revisit if" applies: check the tip offset in Fusion before anything else.

## 4. Open

- Trigger threshold 0.9 (team rule) is kept unless G1 shows a full squeeze does not reach it.
- Start stand, object size, allowed depth and the three-run pass rule are proposals; Martyn decides.
- The Robotiq position step in mm and the grip force at setting 100 are not checked in the manual (unverified); not needed for open/close.
- Tip offset file format and version naming follow D09 (calibration); defined in the PR.

## 5. Start pose facing the operator, HOME_FACE (2026-11-24, simulation only)

Decision path (2026-11-24): first proposal "gripper points toward the robot base, same direction as the handheld" (HOME_FWD) withdrawn after Martyn's objection: the UR10e should work in front of itself, gripper facing the operator. This is valid under UMI-style relative actions (chi2024; [[D07 Software and data format]]): the recorded motion is applied relative to the robot's gripper, so what has to match is the **layout relative to the gripper**, not the same spot in the room. The robot's work is the operator's demonstration turned by 180° about the vertical axis (a turn, not a mirror image: the operator's left becomes the robot's left). This is also how the team's VR teleop maps motion (`BASE_YAW`), so the robot works in the same area for the VR baseline and the handheld replay. The RL3 observation (push toward the robot → robot moves toward the operator) is consistent with this and is kept.

Computed with the team's UR10e model (Pinocchio, TCP 0.174 m), fingertip at the HOME_Q fingertip point (−0.174, 0.692, 0.240) m:

| | Base | Shoulder | Elbow | Wrist 1 | Wrist 2 | Wrist 3 |
| --- | --- | --- | --- | --- | --- | --- |
| HOME_Q (team, tool down) | −90.0° | −84.1° | 111.2° | −117.1° | −90.0° | 0.0° |
| HOME_FACE (proposal) | −90.0° | −87.3° | 136.7° | −49.3° | 90.0° | 180.0° or 90.0° |

- Gripper level, pointing away from the base toward the operator. Wrist 3 sets the jaw direction (left–right wanted); which of the two values gives that depends on how the Robotiq sits on the flange: check by eye.
- Wrist stays up (wrist 1 at 0.36 m above the base plane, as at HOME_Q); the other solution (wrist below, 0.12 m) was rejected because it would come close to the table.
- Reach: all 27 points of a 20 cm cube around the start solved; largest joint change 26°.
- Layout for RL4b: one start stand under the fingertip. Recording: pick outline 10 cm from the start toward the robot, place outline beside it. Replay: the same outlines turned by 180° about the start point (pick outline 10 cm from the start toward the operator). Tape both sets in different colours.
- Code: the recorded motion is applied in the gripper frame from HOME_FACE (no change of the team code; start pose from the settings file).
- Pendant: Local mode, Move → joint values above at low speed, check jaws, save as its own program "HOME_FACE".

## 6. Fingertip reference on both sides (2026-11-24)

Plan (Martyn, 2026-11-24): record with the full handheld (sleeve + adapter + printed Dex1-1 dummy); the robot keeps the real Robotiq; the replay matches the two fingertips in software (no new adapter).

| Side | Fingertip reference | Value | Status |
| --- | --- | --- | --- |
| Handheld | Midpoint between the Dex1-1 dummy pad centres | Tracked point → pads (38.0, 37.4, −232.7) mm in the pose frame (section 1b) | CAD; axis directions from code |
| Robot | Midpoint between the Robotiq pad centres | From the flange: 16.9 mm coupling + 130.2 mm (open) = **147.1 mm**; 160.7 mm when closed (pads move 13.6 mm forward on closing) | Robotiq STEP files and General manual 2021-06-23, p. 64–65, 68, 78 (via the Robotiq clamp thread) |
| Robot, pendant TCP | 174.0 mm from the flange (incl. coupling) | Lies 26.9 mm beyond the open pad centre | Manual p. 78; pendant read by Martyn |

The Dex1-1 dummy closes in a straight line (no forward motion); the 2F-85 pads move forward while closing, so a fixed offset matches at one width only. Proposal: use the open pad centre (147.1 mm) as the robot reference, because the robot positions the fingers before closing; the up to 13.6 mm forward shift during closing is taken up by the soft block. (Reasoned only.)

Measured on the robot (Martyn, 2026-11-24): Robotiq jaws close left–right at the current mounting; flange → fingertip ends ≈ 170 mm (ruler; gripper state and exact start face not recorded). Manual: fingertip ends at 166.2 mm open / 179.7 mm closed from the flange (149.3 / 162.8 mm overall height + 16.9 mm coupling). The 170 mm lies inside that range, so the pendant TCP (174 mm) marks the fingertip ends, not the pad middle; the pad-middle reference of 147.1 mm (open) stays.

**Correction (Martyn, 2026-11-24):** a camera adapter (D435i mount) sits between the flange and the Robotiq. The manual-based 147.1 mm (open pad middle) and the 166–180 mm finger-end range leave it out and are **not valid** for this robot. Whether the pendant TCP (174 mm) includes the adapter is unknown. To measure at the robot: flange face → middle of the finger pads, gripper fully open (this single number is the robot fingertip reference); also adapter thickness and whether the Robotiq coupling is still fitted.

## 7. Measured at HOME_FACE (Martyn, 2026-11-25, ruler, "maybe not accurate")

| Quantity | Measured | Cross-check |
| --- | --- | --- |
| Flange face → middle of finger pads, open | 150 mm | Manual 147.1 mm + camera adapter 7.25 mm = 154.4 mm (if the Robotiq coupling is fitted). Manual finger ends open 166.2 + 7.25 = 173.5 mm ≈ pendant TCP 174 mm, so the pendant TCP is very likely the open finger ends including the adapter. |
| Middle of finger pads above the table at HOME_FACE | 345 mm | Puts the table about 0.107 m below the UR base plane (simulation). The team's code comment "TCP ~0.25 m above the table" at HOME_Q does not match; the measured value is used. |
| Camera adapter thickness | 7.25 mm | — |

Proposed robot fingertip reference: 154 mm from the flange (manual + adapter; agrees with the pendant TCP; ruler 150 mm within about 4 mm). Reasoned from two sources, not a precise measurement.

Consequence for RL4b (simulation, reasoned): from 345 mm the pick needs about 31 cm of downward travel. With the gripper level, its body and the wrist sit at the height of the pads, so at pad height 3 cm they would reach the table: the grasp needs the gripper tilted down, as a hand would do it. Lower start option computed: pads 150 mm above the table, same place and orientation, joints −90 / −62.3 / 140.6 / −78.2 / 90 / 180 deg (wrist joints and flange 150 mm above the table at the start; ±10 cm cube reachable).

Station facts (Martyn, 2026-11-25, with photos): the UR10e is mounted on a carriage of a linear axis; the work surface (slotted aluminium profile table) lies about 113 mm below the base mounting face (ruler). This agrees with the simulation estimate from the HOME_FACE measurements (0.107 m below the base plane, difference 6 mm). The carriage position must be the same for recording and replay (mark it). Jaw direction at HOME_FACE (wrist 3 = 180°): horizontal, left–right (Martyn, at the robot, 2026-11-25; an earlier reading of the photos as up–down was wrong). Wrist 3 stays 180°.

**Start pose chosen (Martyn, 2026-11-25): option A**, lower start: joints −90 / −62.3 / 140.6 / −78.2 / 90 / 180 deg; pads about 156 mm above the table (with the 113 mm table measurement). Move from HOME_FACE is a straight lowering; lowest axis point along the way 156 mm above the table (simulation; gripper body, camera and wrist housings add their radius, about 4–6 cm, not modelled). Replaces HOME_FACE as the RL4 start; to be saved on the pendant as its own program (name proposed: "HOME_FACE_LOW").

**Changed (Martyn, 2026-11-25): HOME_FACE stays the RL4 start** (−90 / −87.3 / 136.7 / −49.3 / 90 / 180 deg; pads about 345–350 mm above the table); option A dropped. The table height (113 mm below the base mounting face) is taken into account instead: the allowed box gets a floor limit from the table (pads at least about 30 mm above it) rather than the RL3 5 cm depth, so a pick from the table means up to about 32 cm downward travel; the handheld start stand is about 35 cm tall.

## 8. RL4 scope changed (Martyn, 2026-11-25)

RL4 no longer picks anything. New idea: a handheld episode (movement + trigger) is replayed first in simulation to see what the robot would do and whether it is safe, then on the real robot, moving slowly, with the Robotiq opening and closing at the recorded trigger moments.

Proposed pass checks (Claude, for Martyn's OK):

| Stage | Pass when |
| --- | --- |
| RL4-sim (MuJoCo on the Mac) | The recorded take plays on the UR10e model with a gripper that opens/closes at the trigger moments; all safety checks pass (floor limit from the table, sideways box, 0.3 rad/s); Martyn judges it looks right and safe |
| RL4-real (UR10e, from HOME_FACE) | The robot follows as in RL3 (no protective stop, tracking logged); the Robotiq opens and closes in the same order and at about the same moments as in the simulation (log: command time and measured width) |

Pick-and-place moves out of RL4; proposed home: RL6 (swap to the Dex1-1), where the mid-January gate needs successful replays anyway. Sections 3 (step 4, RL4b) and the 3-of-3 pass rule are superseded by this section.

## 9. Code (2026-11-25)

dex1-1-handheld PR #22 (`claude/rl4-gripper-replay`): gripper program and Robotiq plug (`software/gripper/`), settings `software/config/rl4_robotiq.json`, `ur10e_replay.py --setup` (fingertip-relative mapping, HOME_FACE start, Menagerie Robotiq in the simulation, floor check), `ur10e_replay_rtde.py --setup` (gripper thread and log). Checked on the Mac: synthetic path and take-02 pass the floor check; directions turned, not mirrored (operator forward → robot +148 mm toward the operator); gripper plug against a fake Modbus server. Not yet run on the real gripper or robot. Steps per take: `software/sim/README.md`, section RL4.

Open (2026-11-25): the simulation applies the trigger rule directly to the MuJoCo gripper; it does not go through the gripper program's timed runner and a plug. D07 (software and data format) puts real and simulated devices behind the same interface (RoboManipBaselines, LeRobot), so a simulation plug is to be added. Not needed for the first RL4 run (Martyn, 2026-11-25: "not yet"); latest with the Dex1-1 plug before RL6.

## 10. Gripper test alone: passed (Martyn, 2026-11-25, ≈14:59 CET)

`software/gripper/robotiq_2f85.py --cycles 5` from fractal-pc and from the Mac (both reach arena.local:4001): 5 of 5 close/open cycles, no error. Opening read back 13.3 mm closed (team "closed" position 215/255) and 84.0 mm open, identical in every cycle and from both computers. The mm values come from the plug's linear mapping of Robotiq's position register over 85 mm, not from a measurement of the real gap.

## 11. First RL4 take in simulation (2026-11-25)

Take `pose-log-20261125-150606/take-01` (station Jetson, handheld with Dex1-1 dummy, Martyn): 30.4 s, 2187 samples, largest gap 0.02 s, trigger up to 1.000. Simulation with `--setup` (Mac): IK error max 3.3 mm; joint speed max 0.25 rad/s (limit active 0.7 s); floor check PASSED (lowest robot fingertip 303 mm above the table); fingertip span x 106 / y 200 / z 109 mm. Gripper events: close 13.3–15.9 s, 18.5–21.0 s, 24.5–25.8 s, plus a 0.1 s open/close flicker at 21.0 s while the trigger hovered at 0.9 (single threshold, no hysteresis; relevant to G1 (trigger value vs travel) and G2 (trigger-to-gripper mapping)). Real-robot dry run refuses: horizontal 201 mm > 150 mm allowed radius (RL3 value); decision needed: re-record smaller or widen the radius.

Decision (Martyn, 2026-11-25): option B, allowed radius widened to 0.22 m for take-01 (`--box 0.22 0.05 0.15`; 'down' replaced by the floor check). Dry run with 0.22 m passes (201 mm horizontal). Take copied to fractal-pc `~/rl2-test/pose-log-20261125-150606/`.

## 12. First RL4 run on the real UR10e (Martyn, 2026-11-25, ≈15:24 CET)

Take-01 from fractal-pc, start HOME_FACE, radius 0.22 m, floor check passed (dry run first). Models on fractal-pc fetched into `~/dex1-1-handheld/software/sim/models` with `fetch_models.sh` the same day.

| Check | Result |
| --- | --- |
| Run completed | Yes, 30.4 s take, no early stop reported |
| Robot tracking (target vs actual TCP, encoders) | RMS 6.5 mm, max 22.3 mm, 0.1 mm at the end (RL3: RMS 3.3, max 8.7 mm; this take moved more and faster) |
| Highest actual joint speed | 0.29 rad/s (limit 0.3) |
| Loop | 3922 ticks in 31.5 s, longest tick 8.2 ms |
| Gripper commands | 8 of 8 sent in the simulated order, at 13.35 / 15.94 / 18.54 / 20.97 / 21.03 / 21.15 / 24.60 / 25.79 s (simulation: 13.3 / 15.9 / 18.5 / 21.0 / 21.0 / 21.1 / 24.5 / 25.8 s) |
| Command → first finger movement | 56–63 ms (width read at 20 Hz, so resolution about 50 ms); flicker commands at 21.03 / 21.15 s while the fingers were still moving |

Logs: fractal-pc `~/rl2-test/rl4-take-01-robot.csv`, `~/rl2-test/rl4-take-01-robot-gripper.csv` (not in Git). Pass checks of section 8 met by the numbers. Martyn's visual check (2026-11-25): "everything looks as expected". **RL4 passed 2026-11-25.**

## 13. Follow-ups: trigger hysteresis and simulation gripper plug (2026-11-25)

Code on dex1-1-handheld branch `claude/project-thread-sx49ne` (pull request pending; not yet on the robot).

**Trigger hysteresis.** At 21.0 s of take-01 the trigger fell from 1.000 to 0.867, rose back to 0.933 and then went to 0. With one threshold at 0.9 that gave open / close / open within 0.16 s (section 12). New rule: close at trigger ≥ 0.9 (`trigger_close_at`, kept equal to the team's VR teleop for G2 (same trigger-to-gripper mapping in both conditions)), open again only below 0.8 (`trigger_open_below`); in between the last command is kept. Both values are in `software/config/rl4_robotiq.json`; setting them equal restores the old rule.

| Basis for 0.8 | Source |
| --- | --- |
| Observed dip below 0.9 during the hover: 0.033 (lowest 0.867); a 0.1 band leaves about 3× margin | take-01 `raw.csv` (measured) |
| Cost: the three releases of take-01 open 12, 224 (the hover) and 24 ms later than with one threshold; at the hover the final open is 21.18 s instead of the last flicker command at 21.11 s | take-01, computed with `trigger_events` |
| Team VR teleop: one threshold 0.9, no hysteresis | `controller_events.py` (`_AnalogTracker`) |
| LieSpline-DP (not in bib) turns gripper targets into binary open/close intent with hysteresis 0.25 on a −1..1 scale (switches above +0.25, below −0.25), i.e. a band of 25 % of the range, used for training labels, not teleop | [`bspline_se3_util.py` `_binary_intent_from_target`](https://github.com/xieerxuan/LieSpline-DP/blob/main/liespline_dp/common/bspline_se3_util.py), `_bspline_se3.yaml` `gripper_intent_hysteresis_q: 0.25` |

No teleop paper or toolkit with a published trigger hysteresis value was found (team code, XRoboToolkit and Unitree mappings per G2); 0.8 rests on the measurement above and is to be revisited with G1 (trigger value vs travel). Side finding for G1: all 2187 trigger values of take-01 are multiples of 1/255 (127 distinct values), i.e. the trigger arrives with 8-bit steps.

**Simulation gripper plug.** The Mac simulation (`ur10e_replay.py --setup`) no longer applies the trigger rule itself: it runs the same gripper program as the real robot (`GripperRunner`, 20 Hz poll, command log) on the simulated clock, with a simulation plug `sim_robotiq_2f85` (MuJoCo Menagerie 2F-85, same width mapping as the real plug). The setup file names both plugs (`plug` for the robot, `sim_plug` for the simulation), the SwapKit idea of real and simulated gripper behind one interface (D07 (software and data format)).

Checks on the Mac (take-01, conda env `humanoid`):

| Check | Result |
| --- | --- |
| Old rule (0.9 / 0.9) through the new simulation path: command times vs real robot log (section 12) | Sim 13.38 / 15.90 / 18.54 / 21.00 / 21.06 / 21.11 / 24.58 / 25.82 s; real 13.35 / 15.94 / 18.54 / 20.97 / 21.03 / 21.15 / 24.60 / 25.79 s. Same 8 commands in the same order, each within 50 ms (one 20 Hz poll) |
| New rule (0.9 / 0.8) | 6 commands: close 13.38, open 15.96, close 18.54, open 21.22, close 24.58, open 25.82 s; flicker gone |
| Arm path unchanged | IK error max 3.3 mm, joint speed max 0.25 rad/s, floor check PASSED (lowest fingertip 303 mm above the table), as in section 11 |
| `ur10e_replay_rtde.py --dry-run` (real-robot script, now picks the plug from the setup file) | Passes, 201 mm horizontal within 220 mm |
| Synthetic test path with `--setup` | Floor check PASSED, close 2.30 s / open 12.88 s |

Also fixed: the "command → finger movement" printout only counts movement before the next command. Re-read of the real log with it: 56–57 ms for the six normal commands; the flicker commands show 0 ms or none because the fingers were already moving.

Open: the new rule has not run on the real robot; one re-run of take-01 on the UR10e would confirm 6 commands there.
