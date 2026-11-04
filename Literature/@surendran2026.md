---
citekey: surendran2026
title: "Bridging Handheld and Teleoperated Supervision for Contact-Rich Manipulation via State-Gated Experts"
year: 2026
tags: [literature]
---
# Bridging Handheld and Teleoperated Supervision for Contact-Rich Manipulation via State-Gated Experts
Vidullan Surendran, Neehar Peri, David Watkins · [Zotero](zotero://select/library/items/Z7HIKJZL)

---

## Key facts (with page)

Naive mixing was worse than handheld alone on all tree tasks (Page 5):
- **NIST Pulley Routing:** 44.0 → 0.0 %
- **Pipe Insertion:** 13.3 → 6.7 %, 
- **Spring-Loaded Battery Evaluation:** 10.0 → 0.0 %

---

## Relevance to thesis


### Contact-Sensitive Phases: Where Handheld devices fails

Handheld devices record _observed_ actions (where the end-effector actually went), while a robot controller is driven by _desired_ actions (the reference it is commanded to track). In free-space, **tolerant** phases the two coincide, so observed handheld labels are valid supervision. In **contact-sensitive** phases they diverge: the desired position drives into the contact surface while the observed position is held above it. Tracking the observed trajectory at the stiffness needed to follow it then produces large, unsafe contact forces. This is why handheld-only labels fail precisely where contact matters — and why targeted teleoperation, which captures desired actions directly, is uniquely valid there.



---

## Q&A

### Question: What is tolerant phase and contact sensitive phase?

In the context of hybrid robotic data collection (Surendran et al., 2026), **tolerant phases** and **contact-sensitive phases**categorize task segments based on how accurately handheld data represents the true control intent needed by a robot:

#### 1. Tolerant Phase

- **Definition**: Task segments involving unconstrained, free-space motion with minimal or no physical contact with the environment.
- **Action Validity**: In tolerant phases, the **observed action** (the physically realized path recorded by a handheld device like UMI) closely matches the **desired action** (the target trajectory commanded to a robot controller).
- **Supervision**: Handheld data provides fully valid, highly scalable supervision for these phases without requiring a robot in the loop.

#### 2. Contact-Sensitive Phase

- **Definition**: Task segments requiring tight geometric alignment, compliance, or interaction against persistent external forces (such as inserting a pipe, seating a spring-loaded battery, or maintaining tension on an O-ring).
- **Action Mismatch (\(\Delta\))**: Due to surface contact, controller dynamics, and finite stiffness, the desired trajectory drives into or below the contact surface, creating a persistent tracking deviation (\(\Delta\)) relative to the observed trajectory.
- **Why Observed Supervision Fails**:
    - If a policy trained only on observed actions executes under **high stiffness**, proportional control converts tracking error into excessively large contact forces (\(F \propto K(x_d - x)\)).
    - If executed under **low stiffness**, positioning errors cause jamming or misalignment.
- **Supervision**: Because observed actions are an invalid proxy for desired intent during contact, these phases require **desired action supervision**—typically gathered through targeted on-robot teleoperation.

### Question: Are there any specific reasons on the chosen hardwares on this paper?

In **Surendran et al. (2026)**, the hardware selections for the **Dual-Mode UMI (DM-UMI)** device and execution platform were specifically chosen to enable **online, real-time teleoperation** alongside scalable handheld data collection:

#### 1. Dual-Mode UMI (DM-UMI) Hardware Modifications

Standard UMI relies on offline monocular SLAM for 6-DoF tracking and offline ArUco tag detection for gripper width estimation. While this works for offline handheld dataset processing, teleoperation requires real-time feedback to command the robot. To enable dual-mode operation, the authors modified the sensing stack:

- **XVISIO DS80 SLAM Camera**: Chosen over standard monocular action cameras (like GoPros) because it provides **online 6-DoF pose estimates** in real time, allowing live streaming of end-effector pose targets to the robot during teleoperation. Furthermore, it provides high relative tracking precision (1.1 mm relative translation error and 0.2° relative rotation error).
- **Magnetic Position Sensors**: Replaced offline visual ArUco tag tracking with **magnetic position sensors** to continuously measure and stream gripper opening/closing width online.

#### 2. Robot Platform & Compliant Control

- **Franka FR3 7-DoF Arm**: Serves as the real-robot execution platform for targeted teleoperation and policy evaluation.
- **Cartesian Impedance Controller**: The robot is operated under a Cartesian impedance controller with torque smoothing. Compliance is critical for contact-rich tasks (such as pipe insertion, battery placement, and pulley routing) because rigid position controllers produce excessively large contact forces when small tracking deviations occur during contact.

---

## Annotations

