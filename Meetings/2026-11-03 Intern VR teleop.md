---
date: 2026-11-03
with: Intern (PICO 4 Ultra VR teleoperation stack)
tags: [meeting]
---
## Agenda

Questions for the intern, grouped so they can be gone through in order. Points marked (D0x) link to the thesis decisions.

**Software stack**
- Which teleop software runs on the PICO 4 Ultra and the PC: XRoboToolkit or something custom? Which versions, and is the code in a repo you can get access to?
- What does the data path look like from headset to PC to robot, and what transport and network does it use (Wi-Fi, USB, ROS 2)?
- Does the stream include controller trigger and button values, not just pose? (This matters for the sync-marker idea.)
- At what rate does controller pose arrive, and what latency or jitter has he measured?

**Tracking and calibration**
- How is the headset/controller frame registered to the robot base? Is there a calibration routine or fixture, and how repeatable is it? (D09)
- Has he seen tracking dropouts, for example from occlusion, lighting, or the lab's distance to the robot? Does he use any guardian or boundary settings?
- Is the Enterprise headset in a special mode (kiosk, developer, streaming)? Are there any company IT or licence restrictions?

**Robot and gripper**
- Which arm and gripper does the teleop drive (the UR10e with the Dex1-1?), and through which interface (RTDE, ROS driver, Unitree SDK)?
- How is the gripper commanded: trigger to width, or open/close only? Is there a width or force command he can share? (This affects mapping handheld width to Dex1-1, D05.)
- Are there safety limits, speed scaling, or workspace limits in place?

**Data recording**
- What gets recorded, in what format (LeRobot, rosbag, HDF5), and at what rates? Which cameras are used, and where are they mounted?
- How are pose, gripper and camera timestamps synchronised? (D06)
- Has any data already been used to train GR00T or π0.5, and what did that pipeline look like?
- Roughly how long does one demo take, and how tiring is a session for the operator?

**Task and alignment**
- Which pick-and-place task and parts has he been using (size, weight)? (This sets your stroke stop and grip force.)
- Could the handheld write the same data format so that both datasets go through one pipeline?
- What's left unfinished or fragile, and who maintains it after his internship?

**Practical**
- Can you get access to the code, docs and hardware, and when is the headset free?
- Who else at Daimler Truck should you talk to?

## Notes

## Agreed actions
