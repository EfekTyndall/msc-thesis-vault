---
date: 2026-11-06
tags: [research, tracking]
---
# PICO Motion Tracker: possible uses in the project

Question (Martyn, 2026-11-06): PICO Motion Trackers are available in the lab. How could they help the project as an addition to the PICO 4 Ultra Enterprise controller? Should they go on the controller or on the operator's arms? If no prior work exists, is this a useful thing for the thesis to check?

Method: `msc-thesis/thesis/literature.bib`, the official PICO product page, the XRoboToolkit paper, and a few targeted web searches (standard search, not a deep-research pass), read on 2026-11-06. Sources that are not in the bib are marked "not in bib". Number of trackers in the lab: not recorded.

**Main finding:** published work puts PICO trackers on the operator's body (ankles, waist, wrists, elbows), never on a handheld tool. Elbow trackers are used to resolve arm redundancy in teleoperation (zhao2025). No paper was found that measures PICO tracker accuracy, or that uses body trackers to compare operator motion between handheld and VR-teleop collection.

## Device facts

| Fact | Value | Source |
| --- | --- | --- |
| Mass | About 27 g, without strap or pad | [PICO product page](https://www.picoxr.com/global/products/pico-motion-tracker) (not in bib) |
| Sensing | 1 IMU + 12 IR sensors, 6-DoF, up to 200 Hz; tracked by the headset | PICO product page; [UploadVR](https://www.uploadvr.com/pico-motion-trackers-object-tracking/) (not in bib) |
| Stated accuracy | Full-body mode only: about 5 cm and 6° average deviation, about 20 ms latency. No figure for object mode | PICO product page, footnote 6 |
| Modes | Full-body or object mode; "object tracking and leg tracking can't be used together" | UploadVR; zhao2025 Sec. II-D ("None, Full-Body, or Object") |
| Trackers per mode | Not stated. NVIDIA IsaacTeleop supports 2, 3 or 5 trackers in body tracking (ankles, waist, wrists or thighs) | [IsaacTeleop body tracking](https://nvidia.github.io/IsaacTeleop/main/device/body_tracking.html) (not in bib) |
| Data in XRoboToolkit | Pose, velocity, acceleration and serial number in the 90 Hz JSON stream; object mode "directly returns motion tracker poses"; not available on Meta Quest | zhao2025 Sec. II-C, II-D, Table I |
| Mounting | Strap and pad; screw thread not stated | PICO product page |

## Prior use

| Work | Placement | Purpose | Source |
| --- | --- | --- | --- |
| XRoboToolkit (zhao2025) | Operator's elbows | Position-only IK constraints "for resolving null-space redundancies" of 7-DoF arms; demo on a Unitree G1 upper body | Sec. II-C.1, III-C |
| GR00T-WholeBodyControl | Feet | Whole-body humanoid teleop; loose clothing can make tracking "fail unpredictably" | [Tutorial](https://nvlabs.github.io/GR00T-WholeBodyControl/tutorials/vr_wholebody_teleop.html) (not in bib) |
| Teleopit (2026) | Ankles | Skeleton "reconstructed from the headset and ankle-tracker signals" | [arXiv 2608.01834](https://arxiv.org/pdf/2608.01834) (not in bib) |
| IsaacTeleop | Ankles, waist, wrists or thighs | 24-joint body pose; joints marked invalid during fast motion or occlusion | IsaacTeleop docs |
| HuMI (nai2026) | Grippers, waist, feet (Vive Ultimate Tracker, not PICO) | Whole-body humanoid data; chosen because headset tracking degrades under occlusion | Sec. II-A |
| Gholami et al. 2021 | Xsens IMU suit, whole body | Objective physical ergonomics of teleop interfaces (posture comfort, range-of-motion comfort, joint usage) plus NASA-TLX | [arXiv 2105.09809](https://arxiv.org/abs/2105.09809) (not in bib) |
| Moyen et al. 2025 | OptiTrack upper-body suit | RULA score of the upper arm in whole-body teleop, alongside NASA-TLX | [arXiv 2509.03222](https://arxiv.org/abs/2509.03222) (not in bib) |

UMI-style handhelds in the bib (chi2024, cheng2026, li2025, zeng2025, liu2026) put one pose sensor on the handheld and none on the operator's body.

## Placement options

**A. On the controller or handheld.** No precedent was found for a second tracker beside the controller on a UMI-style handheld. Both devices are tracked by the same headset (UploadVR), so they share the same occlusion and lighting weaknesses, and the stated 5 cm and 6° is coarser than the controller is likely to be (controller accuracy is still unmeasured, see [[D04 Pose tracking]]). Use: one-off accuracy comparison on the UR10e, which the proposal already plans as "test only" (`msc-thesis/docs/proposal.md`, cost table). It adds little value during collection.

**B. On the operator's arms during VR teleop (robot control).** Precedent: zhao2025 (elbow trackers, 7-DoF arms). The UR10e has 6 joints, so there is no elbow redundancy to resolve and this does not apply to the core platform. The H2 has 7 revolute joints per arm (shoulder pitch, roll, yaw; elbow; wrist roll, pitch, yaw) in Unitree's official URDF ([unitree_ros `robots/h2_description/H2-A.urdf`](https://github.com/unitreerobotics/unitree_ros/blob/41dec58c7a3ccb3050243ef16ac2ca74b2b0f9ca/robots/h2_description/H2-A.urdf); the [H2 developer guide](https://support.unitree.com/home/en/H2_developer) could not be read automatically), so it has elbow redundancy and this applies if the H2 is the deployment robot (still undecided).

**B2. On the operator's forearms during robot-free handheld collection (H2 only).** Martyn's idea (2026-11-06): the XRoboToolkit setup (headset, controllers, forearm trackers), plus the gripper on the controller, used without a robot. The handheld pose comes from the controller as planned in [[D04 Pose tracking]]. The forearm trackers add an elbow position per frame, which could serve as an elbow target for the redundant H2 arm. Precedent: zhao2025 uses the elbow signal live, through its IK with the robot in the loop. [HuMI](https://arxiv.org/html/2602.06643v2) (nai2026) records body parts besides the grippers (waist, feet) in robot-free collection for a humanoid, but not elbows. No work was found that records elbow trackers in robot-free handheld collection. Open points:
- Retargeting: the human elbow does not map directly onto the robot elbow. XRoboToolkit treats it as a position-only IK constraint (zhao2025 Sec. II-C.1); how it is used offline, or by a policy, is not described.
- Export: the training export is gripper pose plus width (`msc-thesis/CLAUDE.md`), so the elbow stream would be an extra recorded field. Whether GR00T or π0.5 use it, or only the H2 IK at deployment, is open.
- Reachability: with no robot in the loop, feasibility is checked afterwards. UMI (chi2024) filters by robot kinematics; HiFi-UMI replays every trajectory in simulation and drops about 2 % (`msc-thesis/docs/proposal.md`). The XRoboToolkit simulation could take this replay role (untested).
- Not needed for the UR10e (6 joints).

**C. On the operator's arms in both conditions (effort measurement).** The research question includes operator effort, currently measured with NASA-TLX and a usability questionnaire only (`msc-thesis/docs/thesis-plan.md`). Objective posture metrics such as RULA or joint-usage comfort have been used to compare teleop interfaces (Gholami 2021, Moyen 2025), but with Xsens or OptiTrack, not PICO trackers, and never for handheld vs VR teleop. The same trackers could be worn on the upper arm or wrist in both conditions, giving one objective arm-motion measure next to NASA-TLX. No precedent was found for this pairing. Open points:
- Whether 6° average error is good enough for posture bands is unverified. RULA scores the upper arm in bands of 20° and more (McAtamney & Corlett 1993, not in bib, not checked here).
- Arm-only placement is not a standard body-tracking setup. IsaacTeleop's setups all include ankle trackers, so arm-only use would probably need object mode (unverified).
- In handheld collection the headset must track the controller and the trackers at once. Whether object mode runs alongside controller tracking is unverified.
- It adds wearables and setup time, against the supervisor's aim of minimal extra hardware (Daily/2026-11-03). That aim concerns the handheld itself; arm straps do not touch the handheld.

## What follows

- **For D01 and D10:** no change. Each option applies equally to the real and the copy path.
- **For D04:** option A stays the planned "test only" check, now without a purchase.
- **Thesis addition:** option C fits the existing operator-effort question and has no direct precedent, so it is a candidate addition. Adding it changes the thesis plan's metrics, so it needs Martyn's and the supervisor's approval. Not urgent before D01.

## Open

- Object-mode accuracy and tracker count: not published by PICO. A wider (deep-research) search for independent accuracy studies has not been run. One candidate, an IEEE Sensors Journal 2026 article on [HAL](https://hal-lirmm.ccsd.cnrs.fr/PEPR_O2R/hal-05496408v1), could not be read (access blocked), so whether it covers PICO trackers is unknown.
- Number of trackers in the lab.
- Whether controller tracking and object-mode trackers run together on the PICO 4 Ultra Enterprise with XRoboToolkit (bench test).
