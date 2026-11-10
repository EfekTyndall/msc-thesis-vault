---
date: 2026-11-10
tags: [research, gaps, pico]
---
# Research gaps raised by the PICO trigger simulation test

Question: which gaps did the first test (PICO trigger driving a simulated Dex1-1, 2026-11-09) expose that the thesis could measure, control or discuss? Plain-language walk-through of the test: [[2026-11-10 PICO trigger test review]].

**What the test did and did not show.** It ran about 14 s (425 messages at 30 per second, 0 lost) with the trigger only. It had no controller pose, no real gripper, no contact in the simulation and no operator study. Gaps below are therefore about the trigger and the data chain, not about grasping.

**Method and limits.** Sources are the code and paper in the review, the earlier note [[2026-11-02 Research gaps]] (full-section read of the papers in `msc-thesis/thesis/literature.bib`, which holds 27 entries) and the Dex1-1 notes. "Not found" means not found in those sources. No wider web search was run; for G1 to G3 a deep-research pass could look for trigger-based gripper control in teleoperation papers outside the bib. It would take longer, so I ask before running it.

Row labels: **New** = not in the 2026-11-02 note; **Extends** = adds evidence to a gap there (its label in brackets).

| # | Gap | Evidence | Relation to earlier gaps | Planned? | Suggested thesis use |
| --- | --- | --- | --- | --- | --- |
| G1 | **The trigger as a gripper signal is uncharacterised.** Resolution (number of distinct steps), dead zone, linearity and hysteresis (does the value differ going up and down) are not specified | zhao2025 II-B states only "analog … between 0 and 1"; the headset app passes on a decimal from the PICO system (`TrackingData.cs`) with no step size given | Extends B5 (force vs stroke) | Measuring value vs travel is on the TODO; characterising force is not | Method: a short "device characterisation" table (steps, dead zone, linearity, force) measured once |
| G2 | **The trigger-to-gripper mapping is a hidden choice, and toolkits differ.** XRoboToolkit's sample maps 0 to 1 linearly over the full stroke. Unitree's Dex1-1 teleop saturates it (open below about 0.3, closed above about 0.5) and limits the step to 0.18 rad per command. No rationale for either mapping was found in the repos (unverified: comments not exhaustively read) | Code in the review, finding 5 | New. Extends B4 (width mismatch) and A9 (gripper semantics) | No | **Fairness rule:** the VR-teleop baseline and the handheld must use the same mapping, otherwise the gripper channel differs between conditions for reasons unrelated to the data source. Which stack the VR baseline uses (the intern's, per [[2026-11-03 Intern VR teleop]]) is not yet known: ask in that meeting |
| G3 | **End-to-end delay from squeeze to finger is unpublished.** The only measured XRoboToolkit delay is for video (82 to 100.5 ms, Table II); no trigger or pose delay is given | zhao2025 IV-A. chi2024 (abstract) builds latency matching into the policy interface, so a known delay matters for deployment | Extends A5 (residual sync error) and B7 | The Dex1-1 command delay is on the Waiting list; the PICO chain delay is not | Measure the chain delay (squeeze to command at the gripper), and report it next to the sync residual |
| G4 | **Three clocks, and the headset stamps at packing time.** `timeStampNs` is the headset's UTC clock when the message is packed, not when the finger moved; the Linux PC, the recording PC and the camera each have their own clock | `TrackingData.cs`, `Utils.cs` (review, finding 2). banaszczyk2024 found Unity-to-robot timestamps differing by up to 200 ms before correction (per the 2026-11-02 note, B7 row) | Extends B7 (partly resolved: meaning is now known, offset and drift are not) | B7 log-and-drift measurement planned | Measure the headset-to-PC offset and its drift over a session as in B7; state in the thesis that PICO stamps are packing time |
| G5 | **Streaming rate versus recording rate.** The source runs at 90 Hz; Python offers only "latest value" getters, so a poller sees every third value at 30 Hz and may mix two messages | Review, findings 3 and 4; the trigger readout note | New (design gap in D06) | Not decided | Decide the recorder reads the 90 Hz stream (needs custom code; whether the add-on offers a queue is **unverified**) or accepts polled data and states the loss |
| G6 | **Link reliability over a full session is unknown.** 0 of 425 lost covers 14 s. The headset-to-PC leg is Wi-Fi; stalls or loss over 1 to 2 hours are unmeasured. The simulation holds the last value when messages stop, which would hide a stall in a recording and is unsafe on a real gripper | Review section 4 (hold-last-value) | Extends A6 (dropouts, episode rejection) | No | Log gaps in timestamps and receive times in every session; define a timeout behaviour and a rejection rule for episodes with gaps |
| G7 | **Simulation checks arm motion, not the gripper channel.** HiFi-UMI validates data by simulation replay (ai2026 abstract); the 2026-11-02 note records that it leaves the gripper channel out (A9). Our simulation has collisions off, because the finger models overlap | `dex1_1_mujoco.py` header | Extends A7 (feasibility) and A9 | No | Optional: a contact-enabled Dex1-1 model to test that recorded width profiles are executable. Feasibility unverified; the finger geometry needs fixing first |
| G8 | **Squeezing may move the controller.** On the real path the index finger works the trigger; any hand force on the trigger may shift the controller pose, which is the tracked pose. Not tested (the test had no pose). Hypothesis, no source | none | New (hypothesis) | No | Cheap check once the sleeve exists: log pose while squeezing the trigger with the controller held still on a table or the UR10e |

## Where each could go

| Priority | Gaps | Why |
| --- | --- | --- |
| Must | G2 | A confound between the two conditions, cheap to remove by choosing one mapping |
| Should | G1, G3, G4, G6 | Measurements already close to the TODO list; each makes a stated limitation into a number |
| Should (decision) | G5 | Shapes the recorder design in D06 |
| Optional | G7, G8 | Valuable if time allows; G8 only a hypothesis |

Whether any of these enters the thesis plan is for you to decide; I have not changed the plan or any decision note.

## Open

- Run a deep-research pass for G1 to G3 outside the bib? Needs your go. (Moved 2026-11-10 to the TODO in [[2026-11-11]], with the G2 mapping decision.)
- Ask the intern which gripper mapping the VR stack uses (G2). (Moved 2026-11-10 to [[Waiting]], Intern.)
