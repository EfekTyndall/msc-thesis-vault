---
date: 2026-11-19
tags: [report, planning]
---
# Work not blocked by the screws, inserts or the Dex1-1

Question (Martyn, 2026-11-19): which thesis tasks can go ahead while the M3 screws, the heat-set inserts and the real Dex1-1 are still missing?

Method: read `msc-thesis/docs/thesis-plan.md` (§4 error budget, §5 schedule, §8 open items), [[Waiting]], [[2026-11-19]] (TODO carries all open items forward), [[D02 Policy camera]], [[D06 Recording and synchronisation]], [[D07 Software and data format]], [[D09 Calibration]], [[D10 Handheld mechanism]], [[2026-11-02 Research gaps]], [[2026-11-10 Research gaps from the PICO sim test]] (G1–G9), [[2026-11-18 GR00T and pi0.5 action format for handheld data]], [[2026-11-16 Yanmei]], [[2026-11-03 Intern VR teleop]] and the Chapters hubs. Each item was checked against later results so that nothing already answered is listed. Sizes are rough estimates (S < ½ day, M 1–2 days, L ≥ 3 days, spread over time), not measured.

## Unblocked tasks, by priority

| # | Task | Why it is not blocked | Size | Source |
| --- | --- | --- | --- | --- |
| 1 | **G1 trigger measurement + G2 mapping decision** (linear, saturated or open/close; same for handheld and VR baseline) | Trigger streams to the PC with `software/xr/trigger_udp.py`; the supervisor left the mapping to Martyn. Only the proportional-vs-binary test on the real gripper waits | S–M | Daily TODO; G1, G2; [[2026-11-16 Yanmei]] |
| 2 | **Station Jetson camera stack** (step 7d: teleimager, any USB webcam streamed to the Mac) | Thesis plan §6 fallback: "build the recorder with any USB camera". Whether the Dex1-1 camera is plain UVC stays unverified until delivery. New install, so needs Martyn's go | M | Daily TODO (Jetson step 7d); [[2026-11-13 dex1_1_service code and Pi 5 fit]] |
| 3 | **Recorder skeleton on the station Jetson**: PICO pose + trigger, webcam, a mock gripper state → raw MCAP with all original timestamps. Includes G5 (read the 90 Hz stream, not 30 Hz polling), G4 (headset-to-Jetson clock offset and drift) and G6 (log gaps, timeout rule) | November gate in thesis plan §5: "Handheld records camera, pose and width together"; width can be a mock until delivery. Only the real width stream (D05) waits. Adds dependencies (e.g. MCAP library), so needs Martyn's go | M–L | thesis-plan §5; [[D06 Recording and synchronisation]]; G4–G6 |
| 4 | **Task definition and evaluation protocol**: sleeve-into-bushing task, part presentation, start-position grid, success criterion and tolerance, episode rejection criteria (gap A6) | Parts already measured (`msc-thesis/docs/hardware-specs.md`, task parts). The 20-rollouts power question (gap C1, [[Waiting]] Later stages) changes the thesis plan, so Martyn decides | M | thesis-plan §3, §8 ("Choose and specify the task"); gaps A6, C1 |
| 5 | **Related-work outline, then thesis chapters 2 and 3** | Thesis plan §5: related work in November; all 10 chapter hubs are "not started" and the LaTeX chapters hold only headings (102 words). The real-path handheld design (sleeve, adapter) is frozen enough for a chapter 3 draft | L (ongoing) | thesis-plan §5 rules, §8; Chapters hubs |
| 6 | **Messages with long lead times** (each sends a message, so each needs Martyn's go): GPU access for fine-tuning and UR10e robot time; supplier follow-up on the Dex1-1 camera interface and connector; 24 V station supply check with the colleague; a date for the intern meeting | Thesis plan §8 open items; replies take days, so asking early saves time later | S each | thesis-plan §8; [[Waiting]] (Colleague, Intern) |
| 7 | **Sync and latency method** (time-coded marker on a monitor, controller tap) and **G3** chain delay up to the Jetson | Thesis plan §5: December starts with calibration, the gate's critical path. Webcam + PICO + Jetson are enough to build and dry-run the method; the squeeze-to-finger part waits on the gripper | M | thesis-plan §4, §5; G3 |
| 8 | **G8 check**: log the controller pose while squeezing the trigger with the handheld held still | The prototype (v22 + tape, glued adapter, dummy gripper) now exists; no gripper or screws needed. Hypothesis only, no source | S | G8 |
| 9 | **Training stack**: confirm the policy family (thesis plan scope: π0.5 default; the supervisor said this is Martyn's to work out), install GR00T N1.7 and openpi on a team GPU PC, smoke-test fine-tuning on a public LeRobot dataset; answer the open points (openpi LeRobot version vs GR00T v2; π0.5 with EE-pose actions) | Needs only GPU access (item 6), not hardware. Checks the D07 Revisit-if (training with relative or no EE state) before any data exists | M–L | [[2026-11-18 GR00T and pi0.5 action format for handheld data]] Open; [[D07 Software and data format]]; thesis-plan §8 |
| 10 | **LeRobot export** from the recorder's MCAP: relative EE actions + absolute width, 30 Hz, LeRobot v2 | Can run on recorder test files from item 3 with mock width | M | `msc-thesis/CLAUDE.md` conventions; D07 |
| 11 | **PICO controller accuracy test prep**: printed mount of the controller (in its sleeve) on the UR10e flange, replay script, robot pose as ground truth | Thesis plan §4 measures the controller, not the gripper; needs robot time (item 6) and a mount design (Fusion work, Martyn's go) | M + robot time | thesis-plan §4; gaps A3, B1 |
| 12 | **VR-teleop baseline arm control**: XRoboToolkit → UR10e without the Dex1-1; ask how the supervisor's stack commands the UR10e | Arm control does not need the gripper. The supervisor's answer is in [[Waiting]] ("not needed until VR teleop is set up"), so it can be asked now by email | M + robot time | [[Waiting]] (Supervisor); [[2026-11-03 Intern VR teleop]] |
| 13 | **D09 calibration procedure written down** (controller-to-gripper-tip transform from the CAD chain as a prior; re-seat check after opening the sleeve, gap B2) | Procedure and CAD prior need no parts; the measurement itself waits on the gripper | S–M | [[D09 Calibration]]; gaps B2, B3 |
| 14 | **Small chores**: ~~printer firmware update at day end, then flow calibration pass 2~~ (dropped 2026-11-19, Martyn); optional ufw rule removal on fractal-pc; update `PRINT-TEST-QUEUE.md` §1 and §3 with today's comfort, finger-room and tracking results (dex1-1-handheld PR) | No parts needed | S | Daily TODO |

## Recommended order

1. ~~Today at day end: printer firmware update (14).~~ Dropped 2026-11-19 (update failed, internet error; Martyn).
2. Give one go for the long-lead messages (6), so replies arrive while the rest runs.
3. G1 measurement and the G2 mapping decision (1): the mapping feeds the recorder and the gripper program.
4. Camera stack (2) → recorder skeleton (3) with G4–G6, plus the quick G8 check (8): this is the November gate.
5. Task definition and protocol (4), including the rollout-count decision.
6. Sync method (7), calibration procedure (13) and the controller accuracy mount (11): December's critical path.
7. Training stack (9) and export (10), once GPU access is confirmed; VR baseline arm control (12) once robot time is booked.
8. Writing (5) runs alongside: related-work outline first, then chapter 3.

## Still blocked (for reference)

Screws and inserts: screwed adapter joints, insert fit and pull-out, v23 sleeve print, bow flex check (now also needs the gripper). Dex1-1: RS-485 check at 500 Hz, proportional vs binary trigger on the real fingers, stroke and rad → width curve, G9 force-limit tests, cable clip, weight test, camera UVC check. Digital scale: mass and centre of mass.

## Open

- None of the tasks above has been started; Martyn picks which to start.

## Revised list (2026-11-19, 15:58, after Martyn's answers)

Martyn's answers: GPU approval exists, but fractal-pc (RTX 3080 Ti, 12 GB) is below the openpi and GR00T README minimums for fine-tuning, so he asks for a larger team PC; the UR10e is always free; no supplier contact (the gripper was ordered through the neighbouring company's CTO, asked in person); intern meeting dropped; the task definition (sleeve into bushing) is wrong and is redone from scratch; the D435i moves from the team Pi to the station Jetson for camera tests; recorder, training and export follow UMI and UMI-style papers; controller accuracy test on the robot (old 11): no; thesis writing: low priority; sync test (old 7): after the Dex1-1 camera arrives.

| Order | Task | Who | Size |
| --- | --- | --- | --- |
| 1 | G1 trigger run (dex1-1-handheld PR #13 logger, caliper steps), then G2 mapping decision | Martyn runs, Claude analyses | S–M |
| 2 | G8: does squeezing move the controller (pose logged, handheld still on the table) | Claude adds pose logging, Martyn runs | S |
| 3 | Survey: how UMI and UMI-style papers record, synchronise, store, calibrate, export and train (bib papers + public code) | Claude | M |
| 4 | Camera software install plan for the station Jetson with the D435i (libjpeg-turbo ≥ 3.0 vs 2.1.2; RealSense on aarch64), then camera test | Claude plans, Martyn OKs | M |
| 5 | Recorder skeleton: D435i + controller pose + trigger + stand-in width, one clock, raw MCAP | Claude | M–L |
| 6 | Task definition from scratch | Martyn describes, Claude writes it up | M |
| 7 | VR-teleop baseline: XRoboToolkit → UR10e arm control | Claude + Martyn at the robot | M |
| 8 | Calibration procedure written down (old 13), from the survey | Claude | S–M |
| 9 | Chores: ~~firmware (running)~~ dropped 2026-11-19 (update failed, internet error; Martyn), optional ufw rule removal on fractal-pc; print queue update done in PR #13 | Martyn | S |
| Later | LeRobot export and training setup (after the survey and a larger GPU); sync and latency test (after the Dex1-1 camera); thesis writing (low priority) | | |
