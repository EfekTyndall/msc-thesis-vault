---
date: 2026-11-24
topic: SwapKit (modular framework: teleoperation, recording, replay with swappable robot, gripper, camera)
status: options for Martyn, no decision yet
---
# SwapKit: one framework, three modes, swappable hardware

**SwapKit** is the working name for the framework Martyn asked about on 2026-11-24: one program with three modes (Teleoperation, Recording, Replay), where the robot, gripper, camera and input device are swappable parts. Built and proven on the UR10e first; the Unitree H2 later.

## Short answer

- **Feasible: yes.** About half of it already runs (replay in simulation and on the real UR10e, the team's VR teleop). Every piece has a published precedent (table below).
- **Thesis strength: yes, as the means, not as the headline.** It guarantees that handheld and VR data go through the same recorder, the same export and the same replay, which is what the fairness rules in the thesis plan (section 3) demand. "A modular robot framework" on its own is not new: RoboManipBaselines, XRoboToolkit, RAPID and LeRobot already claim it.
- **Main risk: time, not technique.** The mid-January gate (replay with the Dex1-1 succeeds) does not move. SwapKit must be built as part of the Replay Ladder, not next to it.

## Terms used here

| Term | Meaning |
| --- | --- |
| Mode | What the program does: drive the robot live (Teleoperation), save an episode (Recording), play a saved episode back (Replay) |
| Plug | A small program for one device that "speaks" a fixed set of commands, e.g. a gripper plug knows "set width", "set force limit", "read width". Swapping hardware = choosing a different plug in a settings file |
| Interface | The fixed list of commands every plug of one kind must offer. The modes only talk to the interface, never to a device directly |
| Episode | One recorded demonstration, start to stop |

## The picture

```mermaid
flowchart LR
  subgraph IN[Input source plug]
    VR[PICO controller live<br/>VR teleop]
    HH[Handheld<br/>PICO + Dex1-1, no robot]
    EP[Saved episode]
  end
  subgraph MODES[Modes]
    T[Teleoperation]
    R[Recording]
    P[Replay]
  end
  subgraph OUT[Hardware plugs]
    RB[Robot plug<br/>UR10e real / URSim / MuJoCo<br/>later H2]
    GR[Gripper plug<br/>Robotiq 2F-85 now<br/>Dex1-1 later]
    CA[Camera plug<br/>D435i now<br/>Dex1-1 camera later]
  end
  VR --> T --> RB
  VR --> R
  HH --> R
  EP --> P --> RB
  T --- GR
  P --- GR
  R --- CA
  R --> MC[(raw MCAP)] --> LR[(LeRobot export)]
```

Read it as: pick an input, pick a mode, pick the plugs in a settings file. The recorder and the export are the same for every combination.

What each mode needs:

| Mode | Input | Robot | Gripper | Camera | Writes |
| --- | --- | --- | --- | --- | --- |
| Teleoperation | VR controller live | real or simulated | set width | optional (operator view) | nothing |
| Recording, VR | VR controller live | real or simulated | set + read width | read images | raw MCAP |
| Recording, handheld | handheld | none | read width (Dex1-1 on the handheld) | read images | raw MCAP |
| Replay | saved episode | real or simulated | set width | optional (check) | log of what the robot did |

Note: with the real Dex1-1 on the handheld (D01 (handheld gripper body)), the **same gripper plug** reads the width on the handheld and sets it on the robot. That is a concrete benefit of the plug design for this thesis.

## What already exists (2026-11-24)

| Piece | Status | Where |
| --- | --- | --- |
| Replay in MuJoCo (RL1) | runs | dex1-1-handheld `software/sim/ur10e_replay.py` |
| Replay to URSim and real UR10e (RL2, RL3) | runs, passed 2026-11-23 | `software/sim/ur10e_replay_rtde.py` |
| VR teleop of the UR10e | runs, team stack | team fork of `xr_teleoperate` (pinned 7e6c754, not ours) |
| UR10e IK, HOME_Q, speed limit | reused from team stack, not copied | same |
| Pose / trigger logging | runs (CSV) | `software/xr/pose_log.py`, `trigger_log.py` |
| H2 and UR10e models for MuJoCo | load and run; H2 cannot stand yet (needs balance controller) | `software/sim/load_models.py` |
| Gripper plug design | agreed, not built | vault `Agent/Reports/2026-11-24 Open items clean-up.md` |
| Recorder (raw MCAP) and LeRobot export | decided (D07 (data format)), not built | RL5 (camera + recorder test) |

So today the pieces are separate scripts. SwapKit mainly means: put them behind fixed interfaces instead of writing a new script per robot.

## How prior work splits it

| Framework | Split into | Modes | Robots shown | Fit for this thesis | Bib |
| --- | --- | --- | --- | --- | --- |
| [RoboManipBaselines](https://arxiv.org/html/2509.17057v2) | environment (robot + sensors), dataset, policy; one Gym-style interface for real and sim | teleop + collection, train, rollout; no replay mode | sim: UR5e, xArm 7, Franka, Kinova, ALOHA, Unitree G1 and more (MuJoCo); real: UR5e, xArm 7 | Closest precedent for "same code in sim and real". New robot = inherit one class, override a few methods (sec. IV-A). Input: keyboard, 3D mouse, GELLO; no VR, no handheld | murooka2026 |
| [XRoboToolkit](https://arxiv.org/html/2508.00097) | XR app on headset, PC service, Python SDK, robot-side IK | teleop, recording on the headset; no replay | real: UR5 (dual), ARX R5, Galaxea R1-Lite; sim: MuJoCo only | Already the input layer here (PICO). Robot side has no plug system (sec. II-A) | zhao2025 |
| [RAPID](https://rapid-kit.github.io/) | modular hardware + software that knows which sensors are plugged in | handheld collection and robot deployment in one system | not read (abstract only) | Closest to "plug and play gripper and camera" and to handheld + robot in one stack. Measures modularity as setup time (two orders of magnitude faster, abstract) | yin2026 |
| [UMI](https://umi-gripper.github.io) | handheld collection, policy interface | collect, train, deploy | several arms | Its recorded data is robot-independent (end-effector pose + gripper width), "deployable across multiple robot platforms" (abstract). Same action format as here (D07) | chi2024 |
| DM-UMI | one device used both as handheld and as teleop input | handheld and teleop collection | one arm | Only precedent found for handheld + teleop data from one system; also shows naive mixing can hurt (vault `Literature/@surendran2026`) | surendran2026 |
| [LeRobot](https://huggingface.co/docs/lerobot/il_robots) | `Robot`, `Teleoperator`, camera classes; settings class per device | `lerobot-teleoperate`, `lerobot-record`, `lerobot-replay` (sends recorded actions back to the robot) | many low-cost arms | Same three modes as Martyn's list, as ready commands. Handheld (no robot) is not a built-in case (unverified) | not in bib |
| [Unitree xr_teleoperate](https://github.com/unitreerobotics/xr_teleoperate) | arm IK, hand/gripper control, image service, recorder | teleop, `--record`, `--sim` (Isaac Lab); no replay command | G1, H1, H1_2, **H2** (since v1.6, 2026-07-29), R1; end-effectors include **Dex1-1** (`--ee dex1`) | The team fork adds the UR10e and holds `assets/h2/H2.urdf`, so the team stack already switches robots by a setting. Whether pin 7e6c754 includes upstream H2 control: unverified | not in bib |

What the precedents agree on:
1. One fixed interface per device kind; a new device = one small class + a settings file (RoboManipBaselines, LeRobot).
2. Real and simulated robots behind the same interface (RoboManipBaselines).
3. Robot-independent episode format: end-effector pose + gripper width (UMI, already D07).
4. No precedent found for one framework that does VR teleop, handheld recording and replay, across sim and real, with swappable grippers. DM-UMI and RAPID come closest. This was checked only against the bib and the sources above, not with a wide search.

## Options

| | Option | What it means | Source | Thesis impact | Time |
| --- | --- | --- | --- | --- | --- |
| A | **Thin SwapKit inside the Replay Ladder** (recommended) | Own small interfaces (robot, gripper, camera, input) in dex1-1-handheld. Existing scripts move behind them during RL4–RL5. Team stack stays the UR10e and H2 driver (outside dependency). Built plugs: UR10e (MuJoCo, URSim, real), Robotiq, D435i; later Dex1-1. H2 plug only as stretch | RoboManipBaselines pattern; UMI data format | Methods chapter gets one "system" section; no new research question; fairness rules get a concrete guarantee | Mostly inside RL4–RL6; extra effort unverified, estimated days, not weeks |
| B | **Build on LeRobot** | Write UR10e, Robotiq, PICO as LeRobot `Robot` / `Teleoperator` plugins; use its record and replay commands | LeRobot docs | Same as A; less own code, but a new dependency (needs Martyn's OK), and handheld recording without a robot must be fitted in | Learning cost; unverified whether its loop fits the 125 Hz RTDE and the MCAP raw format of D07 |
| C | **SwapKit as a second contribution** | A + a measured portability test: same episode replayed on two grippers (Robotiq and Dex1-1) and, if time, on the H2; count code and time to add each plug (RAPID measures setup time) | RAPID, RoboManipBaselines | Adds a small result chapter section; needs the plan's scope table changed | +about 1 week in March (buffer month), unverified |
| D | **H2 in the core** | Prove modularity on the H2 before submission as a must | xr_teleoperate H2 support | Moves H2 from stretch to core; H2 needs standing balance control, not set up | High; collides with the mid-January gate and data collection in January |

Why A first: it costs least, fixes the interfaces before the Dex1-1 arrives (RL6 (swap to the Dex1-1) becomes the first real plug swap, at no extra lab time), and keeps C open for March. D is not recommended: the plan's rule "if replay does not succeed by mid-January, all stretch goals are dropped" would still apply, and the H2 balance controller is unsolved.

## What changes in the thesis (proposal, needs Martyn's OK)

- **Research question:** unchanged. SwapKit is the tool that answers it fairly.
- **Contributions:** add "a single recording, export and replay pipeline for handheld and VR data, with swappable robot, gripper and camera" next to the error budget (plan section 4).
- **Scope table (plan section 2):** core keeps UR10e only. Option C would add "portability check: one episode on two grippers" to core or stretch.
- **Schedule (plan section 5):** no new phase. RL4 (Robotiq pick-and-place) builds the gripper plug, RL5 (camera + recorder) builds the camera plug and recorder, RL6 (swap to the Dex1-1) is the first swap test. VR teleop in MuJoCo for the UR10e is a new small piece (live version of RL1).
- **Thesis plan and CLAUDE.md:** not changed by this note. Changing them is Martyn's call.

## Open / unverified

- Whether team pin 7e6c754 contains upstream H2 arm control (check on the Mac: `git log` of `~/thesis/team/xr_teleoperate`, read-only).
- How the H2 arms are driven while the robot stands (balance controller); H2 MuJoCo model loads but cannot stand.
- LeRobot fit (option B): its camera class and simulation interface were not read.
- Effort estimates in the options table are guesses, not measured.

## Decision to record (when Martyn chooses)

Suggested entry for the `record-decision` skill: "SwapKit: thin modular framework built inside the Replay Ladder (option A); H2 stays stretch". Not recorded yet.
