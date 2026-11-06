---
tags: [waiting]
---
# Waiting

Long-running items that cannot be done in a single session, grouped by what unblocks them. Each item lives in one place only: here while it waits, or in a daily note's `## TODO` once it can be worked on. When a blocker clears, move (cut, not copy) its items into that day's TODO. Tests to run on printed parts stay in `msc-thesis/hardware/PRINT-TEST-QUEUE.md`; this note only points to them.

## Supervisor (on leave until early 2027, email only)
- [ ] Decide [[D01 Handheld gripper body]] after she has seen the mind map. Proposed rule: real Dex1-1 if the RS-485 check passes, otherwise the printed copy. Brief and map sent by email 2026-11-06; waiting for her reply. (since 2026-11-03)

## Dex1-1 delivery (date unknown)
- [ ] RS-485 check: the PC commands the gripper at the needed rate; supply, cable and mass acceptable on a tethered handheld. (since 2026-11-03)
- [ ] Test proportional and binary trigger behaviour. (since 2026-11-04)
- [ ] Measure the finger stroke (official STEP/URDF about 82 mm vs product page 90 mm); check version and accessories (RS-485 board in the box?); check whether the fingers back-drive when unpowered. (since 2026-11-03)
- [ ] Measure trigger-to-finger latency, the rad → width curve, `tau_est` against a force gauge, the cable length, and the undocumented items: mass as delivered, power draw, motor ID of each unit ([[2026-11-03 Dex1-1 technical note]]). (since 2026-11-04)
- [ ] Check whether the UR10e tool connector can supply 24 V at 3.5 A peak for the robot-side gripper, or whether it needs its own supply. (since 2026-11-03)
- [ ] Real path: cable clip for power and RS-485 on real-adapter, once it is known where the cables leave the gripper. (since 2026-11-04)
- [ ] Real-adapter checks on the gripper (rear-mount 4×M3, clamp ring): see `PRINT-TEST-QUEUE.md` §3. (since 2026-11-04)

## PICO + XRoboToolkit setup
- [ ] Check that the PC service delivers the Enterprise controller inputs, then measure trigger value vs travel (linearity, dead zone, resolution). Needed for the real path, where the trigger is the gripper command. (since 2026-11-03)

## Printer access
- [ ] Print and test everything in `msc-thesis/hardware/PRINT-TEST-QUEUE.md`: sleeve variants A and B (PETG), fit near the screw boss. Copy-path items (lever force, friction and lock, ArUco / AS5600 width error) only if the copy path is resumed (frozen since 2026-11-05). (since 2026-11-02)
- [ ] Print the dummy gripper from the official STEP (size in hand, adapter fit, LED check); main line since 2026-11-05; preparing the print file is a daily TODO. (since 2026-11-04)

## Purchases
- [ ] M3 heat-set inserts: once bought, check the 4.0 mm × 6 mm boss holes against them (`PRINT-TEST-QUEUE.md` §3). (since 2026-11-04)

## Company IT access (internal Storefront)
- [ ] Prices for the D01 side-by-side (Dex1-1, MGN9C blocks and MGN9 rails, 24/48 V supply, cables), from the company's suppliers; not needed for the mind map ([[2026-11-05 D01 real vs copy side-by-side]]). (since 2026-11-05)

## Intern (no date yet)
- [ ] Hold the meeting, then fill in Notes, Agreed actions and the intern's name in [[2026-11-03 Intern VR teleop]]. (since 2026-11-03)

## After D01 is settled
- [ ] Update D01, D05 and D10 with the record-decision skill; OK the matching thesis-plan changes.
- [ ] Buy 2 MGN9C blocks and 2 MGN9 rails (75 mm) only if the printed copy stays (on hold since 2026-11-04).
- [ ] Drop the unused Fusion assembly and parts.

## Later stages
- [ ] 20 rollouts per condition is underpowered: keep, or change the protocol ([[2026-11-02 Research gaps]]). Flagged 2026-11-04.
