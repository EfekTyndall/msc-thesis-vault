---
tags: [waiting]
---
# Waiting

Long-running items that cannot be done in a single session, grouped by what unblocks them. Each item lives in one place only: here while it waits, or in a daily note's `## TODO` once it can be worked on. When a blocker clears, move (cut, not copy) its items into that day's TODO. Tests to run on printed parts stay in `dex1-1-handheld/hardware/PRINT-TEST-QUEUE.md`; this note only points to them.

## Supervisor (on leave until early 2027, email only)
Talk held 2026-11-16: outcome in [[2026-11-16 Yanmei]] (D01 real Dex1-1, Path 2 station computer, D08 confirmed).
- [ ] How her VR-teleop stack commands the UR10e (RTDE / ROS 2 driver): not asked at the 2026-11-16 talk; not needed until VR teleop is set up on the handheld Jetson. (since 2026-11-16)

## Colleague (station supply)
- [ ] Validate with the colleague responsible for the station whether the 24 V supply under the UR10e station covers the Dex1-1 (24–64 V, about 3.5 A peak; sized for the Robotiq 2F-85). Agreed at the 2026-11-16 talk ([[2026-11-16 Yanmei]]); colleague's name not recorded. (since 2026-11-16)

## Dex1-1 delivery (date unknown)
- [ ] RS-485 check: the station Jetson (Path 2, [[D06 Recording and synchronisation]]; board and OS fit, [[2026-11-16 Station Jetson check]]) commands the gripper at the needed rate; supply, cable and mass acceptable on a tethered handheld. (since 2026-11-03)
- [ ] Test proportional and binary trigger behaviour. (since 2026-11-04)
- [ ] Measure the finger stroke (official STEP/URDF about 82 mm vs product page 90 mm); check version and accessories (RS-485 board in the box?); check whether the fingers back-drive when unpowered. (since 2026-11-03)
- [ ] Measure trigger-to-finger latency, the rad → width curve, `tau_est` against a force gauge, the cable length, and the undocumented items: mass as delivered, power draw, motor ID of each unit ([[2026-11-03 Dex1-1 technical note]]). (since 2026-11-04)
- [ ] G9 force-limit tests ([[2026-11-16 Dex1-1 force limiting on contact]], Open 1–4): squeeze force vs `kp · Δ` and `tau` with a force gauge, `tau_est` vs gauge, 60 s stall on a rigid part, holding behaviour when commands stop (BRAKE). (since 2026-11-16)
- [ ] Real path: cable clip for power and RS-485 on real-adapter, once it is known where the cables leave the gripper. (since 2026-11-04)
- [ ] Real-adapter checks on the gripper (rear-mount 4×M3, clamp ring): see `PRINT-TEST-QUEUE.md` §3. (since 2026-11-04)

## Purchases
Sorted 2026-11-11 (Martyn): before the gripper arrives vs after. Storefront orders need the manager's approval and a colleague to place them.

**Before the gripper arrives**
- [ ] M3 heat-set inserts: ruthex set GE-KASTEN-001 (100 × RX-M3x5.7, same dims as the CNC Kitchen M3 × 5.7 picked 2026-11-06, [[2026-11-06 M3 heat-set inserts for the adapter boss]]); ordered 2026-11-11 via the Storefront, delivery 1 day per the Storefront, but not arrived by 2026-11-13, date unknown. On arrival: insert fit and pull-out check on the hole-size coupon (4.0 / 4.1 / 4.2 mm; holes measured 2026-11-17, Ø4.1 chosen for the boss) (`PRINT-TEST-QUEUE.md` §3); iron 10–15 °C above the PETG print temperature, max 300 °C; first insert on the coupon. (since 2026-11-04)
- [ ] M3 screw assortment on Amazon (Martyn orders; A2-70 DIN 912, M3 × 6–40 incl. 20 × M3 × 25 and 20 × M3 × 16, 220 nuts): postponed by Martyn 2026-11-12 until the real-adapter redesign (v9, Fable 5.2 test) is final, since it may change the screws. 2026-11-16: real-adapter v10+ uses 4 × M3 × 35 for the rear mount instead of M3 × 25 (`PRINT-TEST-QUEUE.md` §3). 2026-11-18: real-adapter v14 clamp needs 2 × M3 × 16 + 2 nuts; pad joint 2 × M3 × 10 (all within the M3 × 6–40 range). (since 2026-11-11)
- [ ] Screwed adapter joints, once screws and inserts are here: 2 × M3 × 10 into inserts at the sleeve boss and 2 × M3 × 16 + nuts on the v14 clamp ring; until then sleeve–rod and rod–base are superglued ([[2026-11-18 Plate 2c fit and sleeve slip diagnosis]]). (since 2026-11-18)
- [ ] Next sleeve print, once inserts and screws are here: v23 front + v22 back with the plate 1 settings (Generic PETG, flow 0.95) but no brim (Martyn, 2026-11-19; bed-hold checks in `PRINT-TEST-QUEUE.md` §1) plus one tape layer in the back half; then fit inserts in the Ø4.1 boss holes (`PRINT-TEST-QUEUE.md` §1, [[D10 Handheld mechanism]]). Fusion controller-grip v24 still has the "v24 back inner offset" feature active (`back_inner_offset` = 0.3 mm): use the v22 back-half STL, or set the parameter to 0 or suppress the feature before any new back-half export. Until then the prototype uses the plate 1 v22 sleeve with tape. (since 2026-11-19)
- [ ] Digital scale: Martyn borrows one from a friend; tells Claude when it is there. (since 2026-11-11)
- Ballast for the dummy: spare lab screws and nuts, nothing to buy.

**After the gripper arrives**
- [ ] RS-485-to-USB adapter: check the gripper box first; else Storefront (about 1 day to 1 week). (since 2026-11-11)
- [ ] 24 V supply (24–64 V, about 3.5 A peak) and cables: Storefront or arena. (since 2026-11-11)
- [ ] 3–5 m sleeved cable and an overhead cable balancer: not in the lab; length depends on where the cables leave the gripper. (since 2026-11-11)

**Only if needed**
- [ ] Private Wi-Fi access point: only if measurements show the shared lab network is too slow (it worked end to end on 2026-11-09); IT rules question for Yanmei first (not on the 2026-11-16 agenda, Martyn 2026-11-13). (since 2026-11-11)
- Vive Tracker 3.0 + base stations: fallback only, not a concern for now (Martyn, 2026-11-11).

## Company IT access (internal Storefront)
As of 2026-11-10 the Storefront can be browsed but not ordered from; orders go through the manager's approval and a colleague (2026-11-11).
- [ ] Prices for the D01 side-by-side (Dex1-1, MGN9C blocks and MGN9 rails, 24/48 V supply, cables), from the company's suppliers; not needed for the mind map ([[2026-11-05 D01 real vs copy side-by-side]]). (since 2026-11-05)

## Intern (no date yet)
- [ ] Hold the meeting, then fill in Notes, Agreed actions and the intern's name in [[2026-11-03 Intern VR teleop]]. (since 2026-11-03)

## Printed-copy fallback (only if the real path fails)
D01 settled 2026-11-16 on the real Dex1-1; these items wait on the copy being resumed (RS-485 check fails or delivery slips).
- [ ] Buy 2 MGN9C blocks and 2 MGN9 rails (75 mm) only if the printed copy stays (on hold since 2026-11-04).
- [ ] Copy path, only if resumed: print tests in `dex1-1-handheld/hardware/PRINT-TEST-QUEUE.md` §2 (lever force over the stroke, friction and lock, ArUco / AS5600 width error against calipers). Copy path frozen since 2026-11-05. (since 2026-11-04)

## Thesis end (Daimler handover)
- [ ] Clean `dex1-1-handheld` before it is shared with the company: checklist and check commands in [[Handover]]. (since 2026-11-19)

## Later stages
- [ ] 20 rollouts per condition is underpowered: keep, or change the protocol ([[2026-11-02 Research gaps]]). Flagged 2026-11-04.
- [ ] Revisit PICO Motion Trackers as an optional addition ([[2026-11-06 PICO Motion Tracker uses]]): use one in the UR10e accuracy test next to the controller; forearm trackers for the H2 elbow (H2 confirmed as second deployment robot 2026-11-16, [[D08 Deployment]]) or as an objective effort metric need a thesis-plan change and the supervisor's OK. First bench test: arm trackers and both controllers tracked together through XRoboToolkit. (since 2026-11-06)
