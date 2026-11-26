---
date: 2026-11-26
tags: [research, rl5, recorder, mcap, team-code]
---
# Team bag recorder: what it records and what RL5 takes from it

Ask (Martyn, 2026-11-26): read `teleop/bag_recorder` in the team's `xr_teleoperate` (commit 27f8736, "Record one MCAP bag per teleop episode on /bag_recorder/cmd start, stop and cancel") and `ur10e.yaml` before RL5 (D435i camera + recorder) work goes on. Read-only on the Mac clone `~/thesis/team/xr_teleoperate` at the pinned 7e6c754; 27f8736 (2026-10-21, author yanmei) and the follow-up ffbfbf7 (same day, adds the wrist camera) are already in that commit, so no fetch was needed. Nothing changed in the team repo. Related: [[2026-11-25 RL5 camera and recorder plan]], [[D07 Software and data format]].

## What it is

| Point | Finding | Source (team code, 7e6c754) |
| --- | --- | --- |
| Program | ROS 2 node in Python (`rclpy`, `rosbag2_py`), run with the system Python of ROS 2 Humble: `/usr/bin/python3 bag_recorder/bag_recorder_node.py bag_recorder/ur10e.yaml` | `bag_recorder_node.py` docstring |
| Status in the team | "Stand-in for the gibbon bag_recorder_node" (a C++ recorder the teleop adapter was first written against); whether the team records its datasets with this stand-in or with the C++ node is not stated | same; `teleop_adapters.py` line 412 |
| File format | One rosbag per episode, `storage_id="mcap"`, messages serialised as CDR (the standard ROS 2 encoding); folder `output_dir/<YYYYmmdd_HHMMSS>/` with the `.mcap` file and rosbag2's `metadata.yaml` | `start()` |
| Timestamps | Each message stored with its **receive time** on the recorder's clock (MCAP log time); the message's own header stamp is kept as published ("header stamps stay as published") | docstring, `on_message()` |
| Episodes | Command topic `/bag_recorder/cmd`, `std_msgs/Int32`: 1 start, 2 stop (keep), 3 cancel (delete). A bag is written in a `.recording` staging folder and moved into place on stop; cancel, Ctrl-C or SIGTERM delete it; leftovers are deleted at the next start | `on_command()`, `finish()` |
| Who sends start/stop | The VR teleop: double-click A = start, double-click B = stop, double-click X = cancel; only while streaming; an e-stop cancels, stopping the stream saves. Sent over DDS with `unitree_sdk2py` (no ROS on the teleop side) | `teleop_bindings.py`, `teleop_fsm.py`, `teleop_adapters.py` `Ros2TopicRecorder` |
| Late topics | Subscribes each listed topic once it has a publisher (checked every 1 s); a topic missing at start is recorded without and logged | `subscribe_new_topics()` |

## Topics in `ur10e.yaml` (output `/media/ssd/bags/ur10e`)

| Topic | Type | Rate | Stamp | Publisher |
| --- | --- | --- | --- | --- |
| `/ur10e/joint_states` | `sensor_msgs/JointState` (measured q, dq) | 125 Hz (control loop) | wall clock at publish, teleop computer | `robot_arm.py` `UR10e_ArmController._tick()` |
| `/ur10e/joint_cmd` | `sensor_msgs/JointState` (servoJ command) | 125 Hz while streaming | same | same |
| `/robotiq_2f85/cmd` | `std_msgs/Int32` (open/close events) | on change | — | teleop end-effector adapter |
| `/robotiq_2f85/joint_states` | `sensor_msgs/JointState` (measured opening) | 30 Hz default (`--rate`) | wall clock, gripper node | `robotiq_2f85_node.py` |
| `/ur10e/wrist_camera/image_raw/compressed` | `sensor_msgs/CompressedImage` (JPEG) | 30 fps | capture time on the Pi clock | `camera_stream/republisher.py` from the Pi stream (port 5555) |

Not recorded: the controller pose, trigger and buttons (the VR input). A VR-teleop bag holds the robot side only: commanded and measured joints, gripper command and width, wrist images. The rates above come from the code defaults, not from a recorded bag (unverified).

## Does this show the supervisor's pipeline uses MCAP?

Yes for the recorder in the team repo: the author is the supervisor, the storage is MCAP, one file per episode (checked, code). Whether the team's existing datasets were recorded this way (or with the C++ "gibbon" node, whose format is not in the repo) is unverified; asking sends a message (Martyn's call).

It also matches the timing idea of D07 (software and data format) and the RL5 plan: receive time and source stamp kept side by side, alignment later. Same two-clock situation as in the RL5 plan: images carry the Pi's capture time, joints the teleop computer's wall time.

## What RL5 can do with it: options

| Option                                               | How                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                      | For                                                                                                                                                           | Against                                                                                                                                                                                                                                                                                                                                                 |
| ---------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ | ------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| A. Run the team node on the station Jetson           | Jetson has ROS 2 Humble ([[2026-11-19 Station Jetson camera install plan]]); our programs publish their streams as topics; the team node writes the bag                                                                                                                                                                                                                                                                                                                                                                  | Same files as the team, no recorder to write                                                                                                                  | Puts ROS 2 into the recording chain (Replay Ladder rule "No ROS 2"); our side would need `unitree_sdk2py` or `rclpy` publishers, and the `republisher.py` path needs `python3-zmq` for the system Python; rosbag2's MCAP plugin on the Jetson unverified (an apt install would need sudo); no channel for the controller pose without new message types |
| **B. Same layout, own writer, no ROS (recommended)** | Our recorder writes MCAP with the **same topic names and ROS 2 message types** (`sensor_msgs/JointState`, `CompressedImage`, `std_msgs/Int32`, CDR), log time = receive time, header stamp = source time, one file per take with start/stop/cancel; plus our own channels for the controller pose, trigger and buttons. Writing ROS 2 messages into MCAP without ROS is what the `mcap-ros2-support` Python package does ([mcap Python docs](https://mcap.dev/docs/python), not in bib; package not checked on PyPI yet) | Keeps "No ROS 2" and D07; one exporter reads team VR bags and our handheld takes (the VR baseline is recorded with the team stack); opens in the same viewers | One more Python package (Martyn's go); message definitions must match the team's exactly (copied from the ROS 2 Humble definitions, checked against a team bag)                                                                                                                                                                                         |
| C. Own JSON channels (current RL5 plan §4)           | As planned                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                               | Simplest                                                                                                                                                      | Two formats to export; the VR baseline bags need a second reader anyway                                                                                                                                                                                                                                                                                 |

Recommendation: B. The RL5 plan's channels change to the team's names and types; the extra package is `mcap-ros2-support` next to `mcap`. Reasoned only; checked when a team bag and one of ours open side by side in the same reader (needs one real team bag, e.g. from the station's `/media/ssd/bags/ur10e`, location unverified on the station Jetson). The start/stop gesture could also follow the team (double-click A / B / X), so handheld and VR operators use the same buttons; G8 (does pressing move the controller) decides whether a double-click on the handheld moves the pose.

## Is "No ROS 2" still needed? (Martyn, 2026-11-26)

**Where the rule came from.** Not a decision note. It is a finding of [[2026-11-20 Replay Ladder RL1-RL2 setup]] (2026-11-20): "ROS 2 is not needed" **to command the robot**, because the UR10e takes commands over RTDE with `ur_rtde`, as UMI does (`ur-rtde` in UMI's `conda_environment.yaml`), and because ROS 2 on macOS is Tier 3 and listed for amd64 only ([REP 2000](https://github.com/ros-infrastructure/rep/blob/master/rep-2000.rst), not in bib), while the Mac is an arm64 M4. [[2026-11-20 Interim hardware while the Dex1-1 is delayed]] repeats it ("ROS 2 is not required (RTDE needs none)"); project memory shortened it to "No ROS 2". The same 2026-11-20 note already listed the team's ROS 2 bag recorder and flagged as open that both conditions need the same export.

**Does it still hold?** For replay and robot control: yes, the reason is unchanged and the chain works (RL1–RL4 and the Jetson Move passed without ROS; the team itself commands the UR10e with `ur_rtde` and sends topics over DDS with `unitree_sdk2py`, also without ROS). For recording: the rule never had a reason there. It is a habit, not a finding, so it should not decide RL5 on its own.

| | ROS 2 for recording (option A) | Own writer, same files (option B) |
| --- | --- | --- |
| Same files as the team's VR bags | yes (same node) | yes, if topic names and message types match (checked on one team bag) |
| Ready-made, tested code | team node + its test (`tests/test_bag_recorder_node.py`) | to write (estimated half a day on top of the RL5 recorder, unverified) |
| Install on the station Jetson | ROS 2 Humble is already in `/opt/ros/humble`, not sourced ([[2026-11-16 Station Jetson check]]); whether rosbag2's MCAP plugin is there is unverified; if not, `apt` needs sudo (Martyn's password). `republisher.py` needs `python3-zmq` for the system Python (team docstring) | `mcap` (already planned) + `mcap-ros2-support`, pip, no sudo |
| Our sources | must publish topics: `rclpy` only runs with the system Python 3.10 that Humble was built for (team docstrings), not in our conda env; controller pose needs a topic (e.g. `geometry_msgs/PoseStamped`, standard) | write directly from our existing Python processes |
| Testing on the Mac | recorder chain only on the Jetson (no ROS 2 on the Mac) | runs on the Mac with fake sources, as `pose_log.py --fake` |
| Sync | no gain: the node stores receive time + header stamp, the same two times as option B; clocks and delays still measured as in the RL5 plan | same |
| Replay Ladder rule | ROS 2 in the recording chain only, replay unchanged | unchanged |

**Recommendation (reasoned only, Martyn decides):** drop "No ROS 2" as a blanket rule and keep it only for robot control and replay. What matters for the thesis is that handheld and VR data end up in identical files, and both options give that. B stays the recommendation because it keeps Mac testing and needs no sudo. A becomes the better choice if the RL5 R1 look (read-only) finds rosbag2's MCAP plugin already installed on the Jetson: then it costs little and reuses the supervisor's tested code. Check for R1: `ls /opt/ros/humble/lib | grep -i mcap` and `dpkg -l | grep rosbag2-storage-mcap`.

**Why the same format matters (Martyn's question, 2026-11-26).** The team's tools may never read the handheld data. The reason is the other direction: the exporter must read the team's VR bags (CDR) if condition C1 (VR baseline) is recorded with this recorder, so one CDR reader would serve both conditions and show they share one export path. That premise is unverified ("confirm with the team: teleop data format", roadmap step 6). If the VR baseline uses another format, JSON channels in MCAP (option C) suffice and `mcap-ros2-support` is not needed. Proposal: build R2 with the message-writing step swappable; choose CDR or JSON once the team confirms its VR data format.

## Open

- ~~Martyn's choice A / B / C~~ Decided 2026-11-26 (Martyn): option B with CDR, recorded in [[D07 Software and data format]]; "No ROS 2" now applies to robot control and replay only.
- ~~R1 check: rosbag2 MCAP plugin on the station Jetson~~ 2026-11-26: present (rosbag2 0.15.17 with `rosbag2-storage-mcap`), so the team node could run there without installs; option B kept (Martyn's choice), the team node stays available as a cross-check.
- Team's VR data format for the baseline (decides CDR vs JSON).
- One real team bag to compare against (where are the team's bags; is the C++ "gibbon" recorder still used).
- `mcap-ros2-support` on PyPI: version, licence, pure Python (check before asking for the install).
