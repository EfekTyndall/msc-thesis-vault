---
tags: [waiting]
---
# Waiting

Long-running items that cannot be done in a single session, grouped by what unblocks them. Each item lives in one place only: here while it waits, or in a daily note's `## TODO` once it can be worked on. When a blocker clears, move (cut, not copy) its items into that day's TODO. Tests to run on printed parts stay in `dex1-1-handheld/hardware/PRINT-TEST-QUEUE.md`; this note only points to them.

## Supervisor (on leave until early 2027, email only)
- [ ] Decide [[D01 Handheld gripper body]] after she has seen the mind map. Proposed rule: real Dex1-1 if the RS-485 check passes, otherwise the printed copy. Brief and map sent by email 2026-11-06; waiting for her reply. (since 2026-11-03)
- [ ] Claude Code on fractal-pc: give the install go once Yanmei confirms that a company PC may run a cloud AI agent on thesis code and data ([[2026-11-09 Lab workstation Claude Code plan]]; `msc-thesis/docs/setup/SETUP.md` §6 step 4). (since 2026-11-09)

- [ ] Station computer for the Dex1-1 ([[2026-11-12 Raspberry Pi vs lab PC for the Dex1-1]]): fractal-pc and the teleop Jetson are both far from the station (Martyn, 2026-11-12), so `dex1_1_service` must run on a computer at the station. Ask at the Monday talk: may a second Pi 5 be set up at the station next to the existing one (build test needed, Ubuntu 24.04+), or may one of the two AGX Orin 64GB units be placed there? Is the teleop Jetson one of those two? (since 2026-11-12)

## Dex1-1 delivery (date unknown)
- [ ] RS-485 check: the station computer (Jetson or Pi, see Supervisor) commands the gripper at the needed rate; supply, cable and mass acceptable on a tethered handheld. (since 2026-11-03)
- [ ] Test proportional and binary trigger behaviour. (since 2026-11-04)
- [ ] Measure the finger stroke (official STEP/URDF about 82 mm vs product page 90 mm); check version and accessories (RS-485 board in the box?); check whether the fingers back-drive when unpowered. (since 2026-11-03)
- [ ] Measure trigger-to-finger latency, the rad → width curve, `tau_est` against a force gauge, the cable length, and the undocumented items: mass as delivered, power draw, motor ID of each unit ([[2026-11-03 Dex1-1 technical note]]). (since 2026-11-04)
- [ ] Real path: cable clip for power and RS-485 on real-adapter, once it is known where the cables leave the gripper. (since 2026-11-04)
- [ ] Real-adapter checks on the gripper (rear-mount 4×M3, clamp ring): see `PRINT-TEST-QUEUE.md` §3. (since 2026-11-04)

## Purchases
Sorted 2026-11-11 (Martyn): before the gripper arrives vs after. Storefront orders need the manager's approval and a colleague to place them.

**Before the gripper arrives**
- [ ] M3 heat-set inserts: ruthex set GE-KASTEN-001 (100 × RX-M3x5.7, same dims as the CNC Kitchen M3 × 5.7 picked 2026-11-06, [[2026-11-06 M3 heat-set inserts for the adapter boss]]); ordered 2026-11-11 via the Storefront, delivery 1 day per the Storefront (expected about 2026-11-12); the coupon and pull-out check are in the TODO of [[2026-11-12]]. (since 2026-11-04)
- [ ] Digital scale: Martyn borrows one from a friend; tells Claude when it is there. (since 2026-11-11)
- Ballast for the dummy: spare lab screws and nuts, nothing to buy.

**After the gripper arrives**
- [ ] RS-485-to-USB adapter: check the gripper box first; else Storefront (about 1 day to 1 week). (since 2026-11-11)
- [ ] 24 V supply (24–64 V, about 3.5 A peak) and cables: Storefront or arena. (since 2026-11-11)
- [ ] 3–5 m sleeved cable and an overhead cable balancer: not in the lab; length depends on where the cables leave the gripper. (since 2026-11-11)

**Only if needed**
- [ ] Private Wi-Fi access point: only if measurements show the shared lab network is too slow (it worked end to end on 2026-11-09); IT rules question for Yanmei first. (since 2026-11-11)
- Vive Tracker 3.0 + base stations: fallback only, not a concern for now (Martyn, 2026-11-11).

## Company IT access (internal Storefront)
As of 2026-11-10 the Storefront can be browsed but not ordered from; orders go through the manager's approval and a colleague (2026-11-11).
- [ ] Prices for the D01 side-by-side (Dex1-1, MGN9C blocks and MGN9 rails, 24/48 V supply, cables), from the company's suppliers; not needed for the mind map ([[2026-11-05 D01 real vs copy side-by-side]]). (since 2026-11-05)

## Intern (no date yet)
- [ ] Hold the meeting, then fill in Notes, Agreed actions and the intern's name in [[2026-11-03 Intern VR teleop]]. (since 2026-11-03)
- [ ] Ask which trigger-to-gripper mapping the VR stack uses (gap G2, [[2026-11-10 Research gaps from the PICO sim test]]). (since 2026-11-10)

## After D01 is settled
- [ ] Update D01, D05 and D10 with the record-decision skill; OK the matching thesis-plan changes.
- [ ] Buy 2 MGN9C blocks and 2 MGN9 rails (75 mm) only if the printed copy stays (on hold since 2026-11-04).
- [ ] Drop the unused Fusion assembly and parts.
- [ ] Copy path, only if resumed: print tests in `dex1-1-handheld/hardware/PRINT-TEST-QUEUE.md` §2 (lever force over the stroke, friction and lock, ArUco / AS5600 width error against calipers). Copy path frozen since 2026-11-05. (since 2026-11-04)
- [ ] Revisit PICO Motion Trackers as an optional addition ([[2026-11-06 PICO Motion Tracker uses]]): use one in the UR10e accuracy test next to the controller; forearm trackers for the H2 elbow (only if H2 is the deployment robot) or as an objective effort metric need a thesis-plan change and the supervisor's OK. First bench test: arm trackers and both controllers tracked together through XRoboToolkit. (since 2026-11-06)

## Later stages
- [ ] 20 rollouts per condition is underpowered: keep, or change the protocol ([[2026-11-02 Research gaps]]). Flagged 2026-11-04.
