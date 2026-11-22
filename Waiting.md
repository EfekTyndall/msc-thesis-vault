---
tags: [waiting]
---
# Waiting

Long-running items that cannot be done in a single session, grouped by what unblocks them. Each item lives in one place only: here while it waits, or in a daily note's `## TODO` once it can be worked on. When a blocker clears, move (cut, not copy) its items into that day's TODO. Tests to run on printed parts stay in `dex1-1-handheld/hardware/PRINT-TEST-QUEUE.md`; this note only points to them.

## Supervisor (on leave until early 2027, email only)
Talk held 2026-11-16: outcome in [[2026-11-16 Yanmei]] (D01 (handheld gripper body) real Dex1-1, Path 2 station computer, D08 (deployment) confirmed).

## Colleague (station supply)
- [ ] Validate with the colleague responsible for the station whether the 24 V supply under the UR10e station covers the Dex1-1 (24–64 V, about 3.5 A peak; sized for the Robotiq 2F-85). Agreed at the 2026-11-16 talk ([[2026-11-16 Yanmei]]); colleague's name not recorded. (since 2026-11-16)
	- 2026-11-19 (Martyn, photo of the label): Phoenix Contact QUINT POWER, Ord. No. 2904601, output DC 24 V 10 A, adjustable 24–29.5 V, input AC 100–240 V. Rated current is about 2.9 × the Dex1-1 peak (3.5 A), so it fits if the other loads on it stay below about 6.5 A (not measured; the load LEDs >50 % / >75 % / >100 % show the share in use). 24 V sits at the bottom of the Dex1-1 range, so check the voltage at the gripper end of the cable once it is wired. An RS-485-to-USB adapter (label "USB-485-1") in the station cabinet is used by the current gripper (Martyn, 2026-11-19), so the Dex1-1 needs its own.

## Dex1-1 delivery (expected about late Dec 2026 – early Jan 2027, manager 2026-11-20, unconfirmed)
- [ ] RS-485 check: the station Jetson (Path 2, [[D06 Recording and synchronisation]]; board and OS fit, [[2026-11-16 Station Jetson check]]) commands the gripper at the needed rate; supply, cable and mass acceptable on a tethered handheld. (since 2026-11-03)
- [ ] Test proportional and binary trigger behaviour. (since 2026-11-04)
- [ ] Measure the finger stroke (official STEP/URDF about 82 mm vs product page 90 mm); check version and accessories (RS-485 board in the box?); check whether the fingers back-drive when unpowered. (since 2026-11-03)
- [ ] Measure trigger-to-finger latency, the rad → width curve, `tau_est` against a force gauge, the cable length, and the undocumented items: mass as delivered, power draw, motor ID of each unit ([[2026-11-03 Dex1-1 technical note]]). (since 2026-11-04)
- [ ] G9 force-limit tests ([[2026-11-16 Dex1-1 force limiting on contact]], Open 1–4): squeeze force vs `kp · Δ` and `tau` with a force gauge, `tau_est` vs gauge, 60 s stall on a rigid part, holding behaviour when commands stop (BRAKE). (since 2026-11-16)
- [ ] Real path: cable clip for power and RS-485 on real-adapter, once it is known where the cables leave the gripper. (since 2026-11-04)
- [ ] Real-adapter checks on the gripper (rear-mount 4×M3, clamp ring): see `PRINT-TEST-QUEUE.md` §3. (since 2026-11-04)
- [ ] Bow flex check under the real gripper's weight, with screwed joints (needs screws and inserts too); the result decides the adapter material ([[D10 Handheld mechanism]], Revisit if). 2026-11-19: by hand the rod flexes little (Martyn). (since 2026-11-19)

## Purchases
Sorted 2026-11-11 (Martyn): before the gripper arrives vs after. Storefront orders need the manager's approval and a colleague to place them.

**Before the gripper arrives**
2026-11-20 (Martyn): screws and inserts expected next week; work that needs them starts in the week of 23 Nov at the earliest. 2026-11-22 (Martyn): screws arrived, collected on Monday 2026-11-23; inserts expected about 23–27 Nov.
- [ ] M3 heat-set inserts: ruthex set GE-KASTEN-001 (100 × RX-M3x5.7, same dims as the CNC Kitchen M3 × 5.7 picked 2026-11-06, [[2026-11-06 M3 heat-set inserts for the adapter boss]]); ordered 2026-11-11 via the Storefront, delivery 1 day per the Storefront, but not arrived by 2026-11-13, date unknown. On arrival: insert fit and pull-out check on the hole-size coupon (4.0 / 4.1 / 4.2 mm; holes measured 2026-11-17, Ø4.1 chosen for the boss) (`PRINT-TEST-QUEUE.md` §3); iron 10–15 °C above the PETG print temperature, max 300 °C; first insert on the coupon. (since 2026-11-04)
- [ ] M3 screw assortment on Amazon (Martyn orders; A2-70 DIN 912, M3 × 6–40 incl. 20 × M3 × 25 and 20 × M3 × 16, 220 nuts): arrived, Martyn collects it on Monday 2026-11-23 (Martyn, 2026-11-22); was postponed 2026-11-12 until the real-adapter redesign was final. 2026-11-16: real-adapter v10+ uses 4 × M3 × 35 for the rear mount instead of M3 × 25 (`PRINT-TEST-QUEUE.md` §3). 2026-11-18: real-adapter v14 clamp needs 2 × M3 × 16 + 2 nuts; pad joint 2 × M3 × 10 (all within the M3 × 6–40 range). (since 2026-11-11)
- [ ] Screwed adapter joints, once screws and inserts are here: 2 × M3 × 10 into inserts at the sleeve boss and 2 × M3 × 16 + nuts on the v14 clamp ring; until then sleeve–rod and rod–base are superglued ([[2026-11-18 Plate 2c fit and sleeve slip diagnosis]]). (since 2026-11-18)
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
- [ ] Prices for the D01 (handheld gripper body) side-by-side (Dex1-1, MGN9C blocks and MGN9 rails, 24/48 V supply, cables), from the company's suppliers; not needed for the mind map ([[2026-11-05 D01 real vs copy side-by-side]]). (since 2026-11-05)

## Printed-copy fallback (only if the real path fails)
D01 (handheld gripper body) settled 2026-11-16 on the real Dex1-1; these items wait on the copy being resumed (RS-485 check fails or delivery slips).
- [ ] Buy 2 MGN9C blocks and 2 MGN9 rails (75 mm) only if the printed copy stays (on hold since 2026-11-04).
- [ ] Copy path, only if resumed: print tests in `dex1-1-handheld/hardware/PRINT-TEST-QUEUE.md` §2 (lever force over the stroke, friction and lock, ArUco / AS5600 width error against calipers). Copy path frozen since 2026-11-05. (since 2026-11-04)

## Thesis end (Daimler handover)
- [ ] Clean `dex1-1-handheld` before it is shared with the company: checklist and check commands in [[Handover]]. (since 2026-11-19)

## Later stages
- [ ] Grip sleeve closure survey: how open controller grips and gun shells close their halves, as options beyond the tongue-and-groove snap (screws into inserts, connector parts, one-piece slide-on; [[2026-11-09 Grip sleeve snap-fit closure]] Options). Deferred by Martyn 2026-11-20; prototype is the plate 4 v23 front + plate 1 v22 back + tape ([[D10 Handheld mechanism]]). Start at the latest if that sleeve stops snapping or slips in use. (since 2026-11-20)
- [ ] 20 rollouts per condition is underpowered: keep, or change the protocol ([[2026-11-02 Research gaps]]). Flagged 2026-11-04.
- [ ] Revisit PICO Motion Trackers as an optional addition ([[2026-11-06 PICO Motion Tracker uses]]): use one in the UR10e accuracy test next to the controller; forearm trackers for the H2 elbow (H2 confirmed as second deployment robot 2026-11-16, [[D08 Deployment]]) or as an objective effort metric need a thesis-plan change and the supervisor's OK. First bench test: arm trackers and both controllers tracked together through XRoboToolkit. (since 2026-11-06)
