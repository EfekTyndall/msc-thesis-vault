---
date: 2026-11-27
tags: [report, g8, recorder, rl5, pico]
---
# G8 (press test): result and start/stop buttons for the RL5 recorder

Ask (Martyn, 2026-11-27): run G8 (does pressing a PICO controller button move the controller pose enough to corrupt recorded data) and choose the recorder's start/stop button. Gap definition: [[2026-11-10 Research gaps from the PICO sim test]] (G8, hypothesis, no source). Part of the RL5 plan ([[2026-11-25 RL5 camera and recorder plan]]).

## Plain version

Worked example: Martyn holds the right controller in the gun hold and presses A ten times. After each press the controller position is compared with where it was 0.3 s before. A press moves it about 1 mm (largest 2.7 mm) and turns it about 0.7 degrees. Normal hand movement over the same 0.5 s is of the same size (1.2 mm typical, measured at quiet moments in the same run; the baseline has outliers, see Limits).

Why it was run: the tracked pose is what a policy learns from; a press that jerks the controller would put a fake movement into the data.

What it means: the movement comes from the hand and fingers reaching for the button, not from the button (Martyn's observation), and no test can separate the two. A start/stop press happens before and after the usable part of a take, so the first and last 2 s of each take are cut and the movement never reaches the training data. The trigger is different: it is used during the task, so its coupling is part of the real operator motion in every demo.

**Choice (Martyn, 2026-11-27): follow the team's teleoperation stack.** Double-click A starts a take, double-click B stops it, double-click X cancels it (team `xr_teleoperate`, `teleop/teleop_bindings.py` lines 66 to 68, read on the station Jetson 2026-11-27). Handheld and VR operators then use the same buttons. X is on the left controller; the handheld uses the right one, so cancel needs a right-hand button (open). Cut the first and last 2 s of each take (2 s is a proposal, not a measured value). Earlier in the day the choice was a single press of A; the double-click replaces it. The measurement above (A, B about 0.6 to 1.0 mm per press) supports using A and B: the press movement is small and in the cut part of the take.

## Measured (hand run, controller held still in the gun hold)

Data: `msc-thesis/data/g8-press-test/hand-raw.csv` (Mac only, not in Git). Analysis: position change in mm (max within 0.5 s after the press vs the median of the 0.3 s before); angle likewise.

| Control | Presses | Position, median / max (mm) | Angle, median / max (deg) |
|---|---|---|---|
| A | 11 | 1.01 / 2.70 | 0.71 / 1.13 |
| B | 11 | 0.59 / 1.43 | 0.33 / 0.72 |
| Side grip | 10 | 0.32 / 0.67 | 0.26 / 0.40 |
| Joystick click | 10 | 1.04 / 2.08 | 1.38 / 1.66 |
| No press (quiet moments) | 168 | 1.23 / outliers | 0.30 / outliers |

Raw excerpt, one A press (A column goes 0 to 1 at 27.97 s): position x, y, z in mm goes from (40.86, -276.75, -179.35) at 27.860 s to (40.80, -277.22, -179.76) at 28.028 s.

![[2026-11-27 G8 hand run press plot.png]]

## Not measured (open)

- **Trigger:** the value stayed at 0.0 for the whole hand run (cause unknown: not squeezed, or not reported). No trigger result.
- **Menu button:** never appears in the SDK data. No result.
- **Table run** (controller resting on the table): the headset connection dropped after 80 s ("device missing"); only 4 A and 1 B presses usable (A 0.42 mm, B 2.3 mm). Not a valid table result.
- Limits: noise baseline has large outliers from a 41 s data gap and the first samples; the press response is one hand, one session, 10 presses per button.
- Data rate was 52 to 71 rows/s, not the 90 Hz stream rate (reason not checked).

Files on the Jetson (`~/press-table`, `~/press-hand`) are copied to the Mac with matching SHA-256 checksums.

## Follow-ups

- Record this choice as a decision (D06 recording and synchronisation) via `record-decision`; I did not edit `Decisions/`.
- G1 (trigger value vs travel) run will also show whether the trigger is reported at all.
