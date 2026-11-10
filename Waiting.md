---
tags: [waiting]
---
# Waiting

Long-running items that cannot be done in a single session, grouped by what unblocks them. Each item lives in one place only: here while it waits, or in a daily note's `## TODO` once it can be worked on. When a blocker clears, move (cut, not copy) its items into that day's TODO. Tests to run on printed parts stay in `msc-thesis/hardware/PRINT-TEST-QUEUE.md`; this note only points to them.

## Supervisor (on leave until early 2027, email only)
- [ ] Decide [[D01 Handheld gripper body]] after she has seen the mind map. Proposed rule: real Dex1-1 if the RS-485 check passes, otherwise the printed copy. Brief and map sent by email 2026-11-06; waiting for her reply. (since 2026-11-03)
- [ ] Claude Code on fractal-pc: give the install go once Yanmei confirms that a company PC may run a cloud AI agent on thesis code and data ([[2026-11-09 Lab workstation Claude Code plan]]; `msc-thesis/docs/setup/SETUP.md` §6 step 4). (since 2026-11-09)

## Dex1-1 delivery (date unknown)
- [ ] RS-485 check: the PC commands the gripper at the needed rate; supply, cable and mass acceptable on a tethered handheld. (since 2026-11-03)
- [ ] Test proportional and binary trigger behaviour. (since 2026-11-04)
- [ ] Measure the finger stroke (official STEP/URDF about 82 mm vs product page 90 mm); check version and accessories (RS-485 board in the box?); check whether the fingers back-drive when unpowered. (since 2026-11-03)
- [ ] Measure trigger-to-finger latency, the rad → width curve, `tau_est` against a force gauge, the cable length, and the undocumented items: mass as delivered, power draw, motor ID of each unit ([[2026-11-03 Dex1-1 technical note]]). (since 2026-11-04)
- [ ] Check whether the UR10e tool connector can supply 24 V at 3.5 A peak for the robot-side gripper, or whether it needs its own supply. (since 2026-11-03)
- [ ] Real path: cable clip for power and RS-485 on real-adapter, once it is known where the cables leave the gripper. (since 2026-11-04)
- [ ] Real-adapter checks on the gripper (rear-mount 4×M3, clamp ring): see `PRINT-TEST-QUEUE.md` §3. (since 2026-11-04)

## Purchases
- [ ] M3 heat-set inserts: CNC Kitchen M3 × 5.7 Standard picked 2026-11-06 ([[2026-11-06 M3 heat-set inserts for the adapter boss]]); once bought, check the 4.0 mm × 7.0 mm boss holes against them, with the hole-size coupon and pull-out check from that note (`PRINT-TEST-QUEUE.md` §3). (since 2026-11-04)

## Company IT access (internal Storefront)
As of 2026-11-10 the Storefront can be browsed but not ordered from.
- [ ] Order M3 heat-set inserts and long M3 cap screws if no colleague in the arena has them (list in the TODO of [[2026-11-11]]). (since 2026-11-10)
- [ ] Prices for the D01 side-by-side (Dex1-1, MGN9C blocks and MGN9 rails, 24/48 V supply, cables), from the company's suppliers; not needed for the mind map ([[2026-11-05 D01 real vs copy side-by-side]]). (since 2026-11-05)

## Intern (no date yet)
- [ ] Hold the meeting, then fill in Notes, Agreed actions and the intern's name in [[2026-11-03 Intern VR teleop]]. (since 2026-11-03)
- [ ] Ask which trigger-to-gripper mapping the VR stack uses (gap G2, [[2026-11-10 Research gaps from the PICO sim test]]). (since 2026-11-10)

## After D01 is settled
- [ ] Update D01, D05 and D10 with the record-decision skill; OK the matching thesis-plan changes.
- [ ] Buy 2 MGN9C blocks and 2 MGN9 rails (75 mm) only if the printed copy stays (on hold since 2026-11-04).
- [ ] Drop the unused Fusion assembly and parts.
- [ ] Revisit PICO Motion Trackers as an optional addition ([[2026-11-06 PICO Motion Tracker uses]]): use one in the UR10e accuracy test next to the controller; forearm trackers for the H2 elbow (only if H2 is the deployment robot) or as an objective effort metric need a thesis-plan change and the supervisor's OK. First bench test: arm trackers and both controllers tracked together through XRoboToolkit. (since 2026-11-06)

## Later stages
- [ ] 20 rollouts per condition is underpowered: keep, or change the protocol ([[2026-11-02 Research gaps]]). Flagged 2026-11-04.
