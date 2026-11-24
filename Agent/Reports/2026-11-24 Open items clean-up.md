---
date: 2026-11-24
tags: [report, clean-up]
---
# Open items clean-up (proposal, nothing changed yet)

Sources read: [[Waiting]], [[2026-11-24]] `## TODO`, the `## Open` sections of all `Agent/Research` notes. Each item checked against later tests, merged PRs and decision records (D07 (software and data format), D09 (calibration), D10 (handheld mechanism), `PRINT-TEST-QUEUE.md`, `msc-thesis/docs/thesis-plan.md`).

Actions: **Keep** (stays where it is), **Close** (done or no longer needed; ticked with a one-line reason, text kept), **Merge** (two entries for one job become one), **Move** (goes to the place where it can be worked on).

## 1. Close: already done

| Item | Where | Evidence |
| --- | --- | --- |
| Insert boss CAD change (hole depth 6 → 7.0 mm, grow boss) | Research: M3 inserts note | Done in controller-grip v23 (Ø4.1 × 7.0 mm, wall 1.95 mm), [[D10 Handheld mechanism]] History 2026-11-18 |
| Print an insert-hole coupon (3.9 / 4.0 / 4.1 mm) | Research: M3 inserts note | Coupon 4.0 / 4.1 / 4.2 printed and measured, Ø4.1 chosen ([[D10 Handheld mechanism]] 2026-11-18) |
| PETG cycle test of the sleeve (20 open/close cycles) | Research: snap-fit note | Passed 2026-11-17 (`PRINT-TEST-QUEUE.md` §1, v22) |
| Which PETG the lab spools are | Research: snap-fit note | Spectrum PETG, preset 255/70 °C (known since 2026-11-17) |
| Jetson setup step 7d (camera stack) | TODO, Station Jetson | Already replaced 2026-11-19 by the camera steps CAM1–CAM3 |
| "Fusion: ASM real-path v14 left open" | Research: improvement proposals note | Session remark from 2026-11-12, nothing to do |

## 2. Close: no longer needed after a decision

| Item | Where | Decision that made it moot |
| --- | --- | --- |
| Prices for the D01 (handheld gripper body) side-by-side (Dex1-1, MGN9 rails, supply) | Waiting: Company IT access; Research: D01 side-by-side | D01 settled on the real Dex1-1 (2026-11-16); mind map delivered without prices |
| Does the PICO keep its room frame across app restarts | Research: controller pose to robot frame | D07 (software and data format) 2026-11-18: relative actions, no room-to-robot calibration per session |
| ActiveUMI: where the jig-defined origin sits at deployment | Research: UMI variants | Same (D07 relative actions) |
| UMI SLAM appendix not read | Research: UMI variants | No SLAM in this design (PICO tracking, D04 (pose tracking)) |
| MacBook chip (Apple silicon or Intel) | Research: Replay Ladder RL1–RL2 setup | URSim and `ur_rtde` run on fractal-pc; RL1–RL3 (replay in MuJoCo, URSim, real UR10e) passed |
| Private Wi-Fi access point | Waiting: Purchases, only if needed | Not needed: the Mac already works with the PCs, the Jetson and the robots on the local lab network (Martyn, 2026-11-24) |
| Encoder rad → width curve as a calibration | Waiting: Dex1-1 delivery (latency item) | D09 (calibration) 2026-11-23: width from the raw motor angle, caliper only as a sanity check. Keep only "caliper sanity check" |

## 3. Reword: stale supervisor wording (standing rule 2026-11-24: Martyn decides alone; he names what goes to the supervisor)

| Item | Where | Change |
| --- | --- | --- |
| PICO Motion Trackers | Waiting: Later stages | → Move to a new group "Only if time allows" (optional, not mandatory; benefit unknown; most useful for H2 (humanoid) data collection and teleop). Drop the supervisor wording (Martyn, 2026-11-24) |

## 4. Merge: one job listed twice

| Job | Entries now | Proposed single home |
| --- | --- | --- |
| G9 (force limit on contact) tests and choice | Waiting (Dex1-1 delivery); TODO (Software, G9); TODO (Trigger software "must include G9"); Research force-limit note Open 1–6; Research gaps note | Waiting, Dex1-1 delivery: "G9: gauge tests, then choose the limit" (needs the gripper). Research notes point there |
| Gripper control program | TODO Software "Trigger software … against a mock"; TODO Jetson step 8 | One item "Gripper control program (trigger → `rt/dex1/<side>/cmd`) against a mock", parked in Waiting under RL6 (swap to the Dex1-1), see Move. Agreed order (Martyn, 2026-11-24): one trigger program with an exchangeable gripper output; first on the Robotiq 2F-85 in RL4 (Robotiq open/close + pick-and-place, via the team driver), then the Dex1-1 output against a mock just before delivery, then on the real Dex1-1 in RL6 |
| Proportional vs binary trigger test | Waiting (Dex1-1 delivery); part of G2 (trigger-to-gripper mapping) in TODO | Inside the G2 item, in two stages: RL4 (Robotiq pick-and-place) tests binary open/close only, since the team Robotiq driver has nothing else; the proportional vs binary comparison and the G2 choice wait for the Dex1-1 (RL6, swap to the Dex1-1). Updated after the thread (Martyn, 2026-11-24) |
| Deep-research go/no-go for G1–G3 (trigger questions) | TODO G2 item; Research gaps-from-sim note | Only in the G2 item |
| LeRobot dataset version | TODO (data formats); Research data formats; Research GR00T/π0.5 note; `msc-thesis/docs/proposal.md` open question | One TODO item; the notes point to it |
| Dex1-1 acceptance measurements (stroke, version, RS-485 board, back-drive, latency, mass, power, motor IDs) | Two Waiting items + Research D01 side-by-side + integration note | One Waiting item "Delivery check list" |
| Weigh controller, gripper, prints | Waiting (scale); Research D01 side-by-side; Research improvement proposals | Waiting scale item says what to weigh |
| Ask LieSpline-DP authors / watch videos about UR + Dex1-1 mounting | Research integration note; Research LieSpline note | One line in the integration note, "can wait" |
| Station Jetson camera (CAM1–CAM3) and the single D435i | TODO Station Jetson; TODO RL5 note "must share or unmount it" | Under RL5 (D435i + recorder) in the roadmap |

## 5. Move: not workable now, park in Waiting under the step that unblocks it

| Item | From | To |
| --- | --- | --- |
| [S] items check in the PDFs before citing (UMI, DM-UMI, FastUMI, RAPID) | TODO | Waiting "Writing" (before Chapter 2 text) |
| Zotero: add ACT, robomimic, Diffusion Policy, Open X-Embodiment, LeRobot | TODO | Waiting "Writing" |
| LeRobot version; video codec loss in the export | TODO | Waiting "Export" (after RL5 (D435i + recorder), when the exporter is written) |
| Camera CAM1–CAM3 | TODO | Waiting "RL5 (D435i + recorder)" |
| Gripper control program + G9 | TODO | Waiting "Dex1-1 delivery / RL6 (swap to the Dex1-1)" |
| Jetson disk (step 5) and the unused 256 GB SD card: keep "ask the supervisor first" (Martyn names this one, 2026-11-24) | TODO, Jetson step 5 | Waiting "RL5 (D435i + recorder)": first needed when recordings are stored; ask the supervisor then |
| Rough support faces under the clamp collar | TODO Prints | Waiting "Later stages" (next reprint) |
| Optional ufw rule removal on fractal-pc | TODO | Waiting "Later stages" |
| Controller out of view: drift and snap-back | Research controller pose note | Rejection rules evidence log (tests to run) |

After the moves, today's TODO would hold only: RL4 (Robotiq open/close + pick-and-place), G1 (trigger value vs travel) run, G8 (press test), G2 (mapping), task definition, team GPU question.

## 6. Questions for Martyn (yes/no)

1. **24 V supply**: the station supply (Phoenix QUINT, 24 V 10 A) likely covers the Dex1-1 (2026-11-19 label photo), yet Purchases still lists "buy a 24 V supply". Close the colleague question and change the purchase to "only if the station supply's load LEDs or the voltage at the gripper say no"? **Answer: yes (Martyn, 2026-11-24).**
2. **G1 / G8 / G2 now**: these trigger tests use the PICO only (no Dex1-1). Run them as part of RL4, since RL4 is the first step where the trigger drives a gripper? **Answer: yes (Martyn, 2026-11-24)**: G1 (trigger value vs travel) and G8 (does pressing move the controller) run in RL4; G2 (mapping) only partly, binary on the Robotiq, final choice in RL6 (swap to the Dex1-1), see Merge.
3. **Purchases header**: shorten the day-by-day insert log (2026-11-20 … 2026-11-24) to one line "inserts expected 23–27 Nov, not arrived 24 Nov"?

## Keep as is (checked, still valid)

RL4 item; task definition; team GPU question; inserts + sleeve-boss joint; scale; RS-485 adapter; cables + balancer; real-adapter checks; bow flex check; cable clip; RS-485 check; printed-copy fallback group; handover clean-up; rejection rules (parked until after the Replay Ladder); sleeve closure survey (deferred); 20 rollouts per condition; RL5 sync findings Open; rejection evidence log Open; copy-path Research Opens (dormant with the fallback).

Not found stale: `msc-thesis/docs/thesis-plan.md` §6 delivery risk row already names the Replay Ladder (line 99), so the earlier "stale risk row" note is resolved.

## Agreed in the thread (Martyn, 2026-11-24), for the roadmap

- Gripper tests: everything testable now runs on the Robotiq 2F-85 in RL4 (Robotiq open/close + pick-and-place); anything that needs the Dex1-1 waits for delivery (RL6, swap to the Dex1-1). The team Robotiq driver is open/close only (team stack read 2026-11-20); proportional on the Robotiq would need an own command path (2F-85 position/speed/force settings per the Robotiq manual, unverified here).
- Gripper program shape: one shared program (trigger reading, G2 (trigger-to-gripper mapping), G9 (force limit on contact), logging) plus one small "plug" per gripper (set width, set force limit, read width) and one settings file per gripper (max width, speed, force cap, connection). Width in metres, force in newtons. Precedent: Unitree xr_teleoperate selects the hand type by a start option (README, not re-checked); UMI keeps the gripper driver separate.
- Gripping without crushing (G9): cap the squeeze, not the finger travel (Unitree teleop clip 0.18 rad, about 54 N, unverified; UMI firmware force limit 80 N with a separate blocked limit; soft TPU pads). Needed force and damage force: force gauge on the real Dex1-1 (Waiting, G9).
