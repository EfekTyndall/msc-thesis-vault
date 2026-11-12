---
date: 2026-11-12
tags: [research, handheld, real-path, design-review]
---
# Real-path handheld: improvement proposals (design review)

Question (Martyn, [[2026-11-12]]): how can the real-path handheld be improved without scrapping the current design, on four axes: design for the lab printer and filaments, ergonomics, structure and rigidity, and weight against rigidity? This note is also the first stage of a capability test of the model doing the review.

Test approach: stage 1 (this note) is a paper review with ranked proposals, each with expected gain, cost, source and a check on the dummy gripper. Stage 2, after Martyn's go, models one or two chosen proposals in Fusion as new versions. Stage 1 can be scored on: facts correct, printable on the H2S with the lab filaments, and new information.

Method: geometry read from Fusion on 2026-11-12 with read-only scripts (`ASM real-path` v14: real-adapter v7, controller-grip v21, dex1-1-ref v2); nothing was changed or saved. Print history from `dex1-1-handheld/hardware/PRINT-TEST-QUEUE.md`. Prior work from the vault surveys ([[2026-11-02 Handheld trigger survey]], [[2026-11-05 D01 real vs copy side-by-side]]). Printer, filament and ergonomics pages read on 2026-11-12 (web search, standard; no deep research); none is in `msc-thesis/thesis/literature.bib` (all "not in bib"). Stiffness values are hand beam calculations (script logic in the section "How the stiffness estimate was made"), **not FEA and not measured**. Printed-part masses use body volume × 1.27 g/cm³ at 100 % infill (upper bound).

## Main findings

1. **The bow is the weak point, by a wide margin.** Estimated droop of the gripper under its own weight is about 8 mm with a fully solid bow and about 15 mm as a 2-wall print, with a first wobble frequency of about 4–6 Hz. The pose is tracked at the controller, so any flex between controller and gripper is an unrecorded pose error that changes with orientation.
2. **Weight cannot be improved much; the lever arm can.** Gripper and controller are 89 % of the mass. All printed parts together are about 92 g. The load on the wrist comes from the combined centre of mass sitting about 134 mm in front of the grip.
3. **The same material, arranged as a larger hollow section, is about 5 times stiffer.** A 24 × 18 mm box with 4 walls uses slightly less plastic per length than the current solid 14 × 10 mm bar.

## Baseline (Fusion, 2026-11-12)

Frame: project frame, +y approach, +z camera side, origin at the Dex1-1 base.

| Item | Value | Source |
| --- | --- | --- |
| real-adapter v7 volume | 48.59 cm³ → 61.7 g | Fusion |
| Sleeve A v21 volume (front 12.50 + back 11.43) | 23.93 cm³ → 30.4 g | Fusion |
| Bow section | 14 mm wide (x) × 10 mm deep (in the bow plane), solid | Fusion faces |
| Bow centreline length, pad to rear plate | about 195 mm, 6 straight segments, R 5 inside fillets | Fusion faces |
| Rear plate | 36 × 55 × 5 mm, 4 × Ø3.4 with Ø6.5 × 3 counterbore (2.0 mm under the heads); the bow joins only its lowest 8.5 mm | Fusion faces |
| Clamp ring | bore Ø49.2, 4 mm wall, 12 mm wide (y 18–30), 1 × M3 at the top split; tied to the plate by a 10 × 7 mm beam | Fusion faces |
| Pad on the sleeve boss | about 14 × 16 mm, 2 × Ø3.4, hole spacing 8.0 mm | Fusion faces |
| Grip axis against gripper axis | 25° | Fusion occurrence transform |
| Grip centre (mean of the two sleeve-half centroids) | y ≈ −149, z ≈ −75 mm | Fusion |
| Bow mass share | bow ≈ 28 cm³ of the adapter's 48.6 cm³ (ring ≈ 8, plate ≈ 10, tie beam ≈ 2) | computed from the face dimensions |

Mass and lever arms (assumption: gripper centre of mass at y = 45 ± 15 mm, z ≈ 5 mm, **unverified**; controller centre of mass assumed at the grip centre, unverified):

| Part | Mass (g) | Share | y of centre of mass (mm) |
| --- | --- | --- | --- |
| Dex1-1 ([Unitree](https://unitree.com/Dex1-1)) | 550 | 65 % | 45 (assumed) |
| Controller with batteries ([[2026-11-05 D01 real vs copy side-by-side]]) | ≈ 201 | 24 % | −149 (assumed) |
| real-adapter v7 | 61.7 | 7 % | −46 |
| Sleeve A v21 | 30.4 | 4 % | −149 |
| Total | ≈ 843 | | combined: y ≈ −15, z ≈ −21 |

- Combined centre of mass: about 134 mm in front of the grip centre and about 54 mm above it; it lies just behind the gripper's rear flange.
- Static moment about the grip centre with the gripper axis horizontal: about 1.1 N·m (0.55 kg × 9.81 × 0.194 m for the gripper, plus 0.06 N·m for the adapter). With the gripper pointing straight down the lever is the 80 mm height offset instead: about 0.43 N·m.

## What the sources say

| Topic | Statement | Source |
| --- | --- | --- |
| Weight | 1.4 kg or less for one-handed tools; 0.4 kg for precision tools; heavier tools "should be supported with a counterbalance, tool holder, articulating arm, or straps" | [CCOHS, Hand Tool Ergonomics: Tool Design](https://www.ccohs.ca/oshanswers/ergonomics/handtools/tooldesign.html) |
| Weight | "Frequently used tools that weigh more than 0.5 kg (1 pound) should be counter-balanced." | [CCOHS, Powered Hand Tools: Ergonomics](https://www.ccohs.ca/oshanswers/safety_haz/power_tools/ergo.html) |
| Balance | "the centre of gravity be aligned with the centre of the gripping hand"; "front-heavy" drills "should be avoided" | CCOHS Tool Design |
| Handle | 30–50 mm diameter for a power grip, 40 mm best; length not less than 100 mm | CCOHS Tool Design |
| Handle angle | Bent or pistol handles where the force is in line with the forearm (typically horizontal); straight handles where it is perpendicular (typically vertical); avoid wrist flexion, extension or deviation | CCOHS Tool Design |
| Trigger | Frequent index-finger triggering is a risk; a longer trigger for two or three fingers reduces it; minimise trigger force | CCOHS Tool Design |
| Surface | Non-slip, compressible handle materials; avoid glossy handles; cover sharp edges | CCOHS Tool Design |
| Prior handhelds | UMI 780 g, UMI-3D 1120 g and names weight as a fatigue cause; TacUMI places the centre of mass near the palm and adds a lock so the trigger need not be held | [[2026-11-02 Handheld trigger survey]] (chi2024, wang2026, cheng2026) |
| Printer | H2S: 340 × 320 × 340 mm, single hardened-steel nozzle to 350 °C, chamber heated to 65 °C, carbon- and glass-fibre filaments listed | Reseller and press pages: [Core Electronics](https://core-electronics.com.au/bambu-lab-h2s-3d-printer-machine-only.html), [3D Printing Industry](https://3dprintingindustry.com/news/bambu-lab-launches-the-new-h2s-technical-specifications-and-pricing-243603/); Bambu's own page could not be read |
| PETG | Flexural modulus 2000 MPa, flexural strength 64 MPa, HDT 70 °C (High Speed); flexural strength 72.3 MPa flat against 38.8 MPa upright at 100 % infill (Premium) | [Spectrum PET-G Premium High Speed TDS](https://spectrumfilaments.com/wp-content/uploads/2024/11/en_tds_spectrum_petg_premium_high_speed.pdf), [Spectrum Premium PET-G TDS](https://download.spectrumfilaments.com/TDS/Spectrum_Premium_PETG_TDS_EN.pdf). That the lab spools (SKU 81546 and others, [[2026-11-05]]) are Spectrum is inferred from the names and barcode prefix, **unverified** |
| HT-PLA | Tg 59.8 °C, HDT (1.8 MPa) 58.6 °C as printed; modulus values between 2.4 and 3.3 GPa are listed, but the rows could not be assigned from the PDF text (**unverified**) | [Polymaker HT-PLA TDS V1.3](https://polymaker.com/wp-content/uploads/lana-downloads/TDS_Polymaker_HT-PLA_V1.3_2025-12-10_EN.pdf) |
| TPU | Bambu TPU for AMS is Shore 68D, Young's modulus 1190 MPa flat: a hard TPU, not a soft cushion | [Bambu TPU for AMS TDS](https://cdn.shopify.com/s/files/1/1339/4265/files/Bambu_TPU_for_AMS_Technical_Data_Sheet.pdf) |
| Walls vs infill | Hook test at 20 % infill: 2 perimeters about 20 kg, 4 perimeters 33 kg. Second test: perimeters add strength for about 0.25 m of filament per kg, infill for 0.64 m per kg | [CNC Kitchen](https://www.cnckitchen.com/blog/the-effect-of-extrusion-width-on-strength-and-quality-of-3d-prints), [Raspberry Pi magazine](https://magazine.raspberrypi.com/articles/experiments-with-strong-3d-prints) |
| Inserts | Ruthex M3 inserts pulled out of 100 % PLA at 181 kg on average; in torque the screw head sheared first | [CNC Kitchen](https://www.cnckitchen.com/blog/threaded-inserts-for-3d-prints-cheap-vs-expensive) |

Not found: no UMI-family paper in the vault surveys states a stiffness target or a measured flex between tracker and gripper. The targets below are therefore proposals without precedent.

## Proposals

Ranked by expected effect. "Keeps" lists what stays untouched, since the current design is to be kept.

### P1. Replace the solid 14 × 10 bar by a larger hollow section (rigidity, weight)

- **Finding:** bending stiffness grows with the cube of the section depth, and the bow bends about its 10 mm side. Estimates for the gripper's own weight (E = 2000 MPa, sleeve joint assumed rigid, so real values are worse):

| Bow section (width × depth, mm) | Plastic per 195 mm | Droop at 1 g | Sideways per 1 g | First wobble |
| --- | --- | --- | --- | --- |
| 14 × 10 solid (v7 at 100 % infill) | 35 g | 7.6 mm | 8.2 mm | ≈ 5.5 Hz |
| 14 × 10, 2 walls (0.84 mm), infill ignored | 9 g | 15.5 mm | 19 mm | ≈ 4 Hz |
| 18 × 18 box, 4 walls (1.68 mm) | 27 g | 1.8 mm | 3.4 mm | ≈ 8–12 Hz |
| 24 × 18 box, 4 walls | 32 g | 1.4 mm | 1.9 mm | ≈ 11–13 Hz |
| 24 × 22 box, 6 walls (2.52 mm) | 51 g | 0.7 mm | 1.1 mm | ≈ 15–19 Hz |
| 28 × 24 box, 6 walls | 59 g | 0.5 mm | 0.7 mm | ≈ 19–23 Hz |

- **Proposal:** model the bow as a solid 24 × 18 to 24 × 22 mm section and let the slicer make the box (4 to 6 walls, 10–15 % infill). Grow the section on the side away from the fingers and in width, so the finger room (23 mm at the trigger, 26–32 mm at the grip) is unchanged. Round the two edges facing the fingers (R 2–3 mm; CCOHS on sharp edges).
- **Why walls, not infill:** the CNC Kitchen and Raspberry Pi magazine tests above.
- **Cost:** about 0 to +20 g; a wider bow in front of the hand.
- **Keeps:** bow route, pad, boss, inserts, rear plate pattern, clamp ring, sleeve.
- **Check:** on the ballasted dummy, clamp the sleeve and measure the fingertip height with and without the 550 g (ruler or calipers against the table); film a pluck in phone slow motion and count the wobble frequency. Proposed target without precedent: droop under 1 mm and first wobble above 15 Hz.
- **No precedent** for a stiffness number; the route itself follows the ActiveUMI-style bow already chosen in [[D10 Handheld mechanism]].

### P2. Shorten the lever arm (ergonomics, rigidity)

- **Finding:** between the bow's riser and the rear plate there is an 18 mm horizontal piece, and the plate sits about 40 mm in front of the controller head (screenshot scale, ± 3 mm).
- **Proposal A (small):** set the rear plate directly on the riser, moving the gripper back by 15–18 mm. Estimated effect: wrist moment −8 %, droop about −20 % (computed from the same beam model, since droop scales with the square of the arm).
- **Proposal B (larger, needs a test):** move the gripper further back and up over the controller head. This follows CCOHS (centre of gravity at the hand) and TacUMI (centre of mass near the palm), but it may hide the rim LEDs from the headset and change what the gripper camera sees. Only after the tracking test in `PRINT-TEST-QUEUE.md` §4.
- **Cost:** A changes only real-adapter. B needs a dummy print and the tracking test.

### P3. Stiffen and de-risk the joint to the sleeve (rigidity)

- **Finding:** the whole 1.1 N·m passes through a 14 × 16 mm pad with two M3 screws 8 mm apart, into a boss on the sleeve's front half. Estimated tension in the outer insert: about 80 N static, about 250 N at 3 g (computed). The ruthex pull-out of 181 kg in PLA gives a strength margin, so strength is not the concern. Stiffness is: a short pad rotates easily, the front half is held to the back half only by the snap seam, and the v21 print "slides down and twists a little" on the controller (`PRINT-TEST-QUEUE.md` §1). Each of these adds to the flex of P1 and none is in the estimate.
- **Proposals, in order of effort:**
  1. Longer pad and boss with the screws 16–20 mm apart, if the ring and little finger still have room (check on the PETG print). Rotation stiffness of the joint grows roughly with the square of the spacing (computed, idealised).
  2. A printed step or key between pad and boss so the screws only clamp and the printed shapes carry the moment.
  3. Remove the play between sleeve and controller. Three shallow crush ribs inside the sleeve are common practice (no source read, **unverified**); the lab TPU is too hard (68D) to act as a cushion.
  4. Print the bow and the sleeve front half as one part. The reason for the bolted joint was to keep sleeve variants swappable ([[D10 Handheld mechanism]], 2026-11-04), and variant B was dropped on 2026-11-09. Drawback: the sleeve prints upright and the bow prints best on its side, so one of them gets the wrong layer direction. Not recommended before 1–3 are tried.
- **Check:** same droop test as P1, once with the sleeve clamped and once with the controller clamped; the difference is the joint and sleeve share.

### P4. Rear plate details (structure)

- **Finding:** the bow joins the 55 mm tall plate only along its lowest 8.5 mm, and 2.0 mm of plastic sits under the four screw heads.
- **Proposal:** one central rib (x = 0, at most 3 mm wide, between the screw columns at x = ±5 mm) or two ribs outside them (|x| > 9 mm) from the riser up the back of the plate, leaving tool access to the screw heads, and 3 mm under the heads if the M3 × 25 thread engagement (6 mm now) allows it.
- **Cost:** a few grams. **Check:** no visible gap opening at the plate foot under the ballast.

### P5. Print orientation and settings (printer, filament)

- **Bow on its side.** The bow is planar, so lying on its side every layer runs along the load path. The PETG data sheet gives about half the flexural strength across layers (38.8 against 72.3 MPa). The clamp ring then stands upright: its bore needs support and its top and bottom see hoop load across layers. If the ring cracks in the dummy test, the fix is a thicker ring wall, not a new orientation.
- **4 walls on both parts.** The saved sleeve slice uses 2 walls and 15 % infill (`sleeve-A2.3mf`, PLA fit print, 2026-11-10). In a 3.5 mm wall, 4 walls make the wall solid perimeter. Arachne stays on (v21 finding).
- **Material stays PETG for the sleeve** (the snap needs toughness; `PRINT-TEST-QUEUE.md`). For the bow, HT-PLA may be 20–60 % stiffer (unverified, see table) but softens near 59 °C and the geometry change of P1 gives 5 to 10 times. Recommendation: geometry first, PETG throughout.
- **Fibre-filled filament (not in the lab):** the H2S is listed for carbon- and glass-fibre grades with its stock hardened nozzle. A PETG-CF bow is an option only if P1 in plain PETG is still too soft; it needs a purchase and a data sheet check first.
- **Build volume** (340 × 320 × 340 mm) fits the whole adapter (about 191 × 57 × 168 mm) in any orientation, so no splitting is needed.

### P6. Hang point for the balancer and cable route (ergonomics)

- **Finding:** at 843 g the handheld is under the 1.4 kg one-hand limit but above the 0.5 kg at which CCOHS asks for a counterbalance on frequently used tools. A balancer is already on the purchase list ([[Waiting]]).
- **Proposal:** add a hang eye on top of the rear plate, above the combined centre of mass (y ≈ −15 mm). The clamp-ring tab (y 18–30) is 33–45 mm in front of it, so hanging there tips the gripper nose up. Route the camera and power cables back along the bow to the pommel end, so their weight and pull act at the hand and not at the gripper.
- **Check:** hung from the eye with cables attached, the gripper axis stays level within a few degrees. Re-derive the eye position once the gripper and controller are weighed.

### P7. Hand interface (ergonomics)

- **Handle size is fine:** the sleeve is about 42 mm across, inside the 30–50 mm range.
- **Grip angle:** at 25° the handle is closer to a straight handle than to a pistol grip. CCOHS recommends straight handles for vertical force and pistol handles for horizontal force, so the right angle depends on whether the task is mostly top-down picking or forward reaching. The angle can be changed in the bow alone (pad angle), with the sleeve untouched. Proposal: decide after holding the ballasted dummy at the real table height, not before.
- **Trigger hold:** on the real path the controller's index trigger is the gripper command, so holding an object means holding the trigger. CCOHS flags repeated index-finger triggering, and TacUMI adds a lock for this. Proposal: a software hold on the controller's grip button, to be settled with the trigger mapping decision (G2). No hardware change.
- **Surface:** PETG prints glossy and CCOHS advises against glossy handles. A textured outer wall from the slicer ("fuzzy skin", no source read, **unverified**) could be tried on a seam ring first.
- **Rim edges:** blend the sleeve's upper and lower rims, since the sleeve is about 65 mm long and the palm overlaps its ends (CCOHS asks for handles of 100 mm or more to avoid pressure points).

### P8. Weight (weight against rigidity)

- Printed parts are 92 g of 843 g. Removing a third of the printed mass saves about 30 g, or 3.5 %.
- The exchange rate is strongly in favour of rigidity: P1 gains a factor of 5 for about 0 g and a factor of 10 for about +20 g.
- Recommendation: do not spend effort on gram saving. Spend it on the lever arm (P2), the balancer (P6) and stiffness (P1, P3).

## Suggested order

| Step | What | Needs |
| --- | --- | --- |
| 1 | Print the dummy and the current v7 adapter as they are; run the droop and pluck test. This confirms or refutes the estimate at no design cost. | PETG, ballast, phone |
| 2 | P1 + P2A + P4 in one new real-adapter version | Martyn's go for Fusion |
| 3 | P3 items 1–3 on the sleeve front half | PETG v21 result first |
| 4 | P6 hang eye, once the scale is there | Scale |
| 5 | P7 angle and P2B only after the tracking test | Headset session |

## How the stiffness estimate was made

- Bow centreline as six straight segments in the y-z plane: (−153, −115), (−148, −127), (−128, −124), (−100, −110), (−72, −100), (−42, −68), (−35, −30), (−17, −29.5) mm, from the Fusion faces.
- Fixed at the pad; plate, ring and gripper rigid; load 0.55 kg at (45, 5) mm.
- Vertical droop: δ = W / (E·I) · ∫ (45 − y)² ds. Sideways: bending about the in-plane axis plus torsion, with G = E / 2.76 and thin-wall torsion constants for the boxes.
- E = 2000 MPa (PETG data sheet, flexural). Printed parts are anisotropic and not fully dense, so this is an optimistic modulus.
- Left out: sleeve joint, sleeve on controller, plate and ring flex, the adapter's own mass, creep. All of them make the real part softer.
- Wobble frequency: single mass (0.55 kg) on the computed spring.

## Open

- The estimate is a hand calculation. Step 1 above, or a Fusion static simulation in stage 2, would verify it.
- Gripper centre of mass and controller mass are assumed; weigh and balance both when the scale and the gripper arrive.
- Official Bambu H2S specification page and the HT-PLA modulus rows were not readable; both are marked unverified above.
- A wider search (deep research) could look for stiffness or flex figures in handheld data-collection devices and for wrist-moment limits; not run, waits for Martyn's go.
- Fusion: `ASM real-path` v14 was opened for reading and left open, unmodified.

## Stage 2: real-adapter v8 (Fusion, 2026-11-12)

Modelled after Martyn's go; saved as a new Fusion version, v7 remains in the version history. Controller-grip and the assemblies were not changed (`ASM real-path` was only refreshed in memory for the check and not saved).

| Change | Detail | Check |
| --- | --- | --- |
| Bow section (P1) | 24 mm wide (x ±12), 22 mm deep. The finger-side faces are the v7 faces, so the finger room is unchanged; all added depth is on the outside | Fusion face planes reused for the profile |
| Heel block | The bow root behind and below the pad is now 24 mm wide and runs back to y = −170 | No contact with the sleeve (see below) |
| Pad screw access (new finding) | In v7 neither pad screw could be inserted: the rear Ø3.4 hole was blind after 13 mm, and the front screw had 9 mm of free space below the pad for a 13 mm long screw. v8 has two Ø6.5 tunnels along the screw axes from the pad underside through the bow | Point test along both axes: 5 mm of pad, then free to the outside |
| Plate onto riser (P2A) | Not done as proposed. The thicker riser now reaches the rear plate by itself (1.5 mm short of it at the plate foot), so the 18 mm horizontal piece is absorbed. Moving the gripper back further would mean moving the controller in controller-grip, which was out of scope | – |
| Plate ribs (P4) | Two ribs, 3.5 mm thick at \|x\| = 8.5–12, from the riser top up the plate to z = 8; they leave the screw heads at x = ±5 accessible | Fusion |
| Under-head thickness (P4) | Left at 2.0 mm. Raising it to 3 mm costs 1 mm of the 6 mm thread engagement of the M3 × 25 screws, and the load per screw is a few newtons (computed) | – |
| Finger-side edges | R 2.5 mm on the edges facing the fingers | Fillet feature healthy |

Checks run:
- Timeline: 93 features, none unhealthy. One solid body.
- Interference of the adapter against all visible bodies in `ASM real-path` (gripper STEP parts, sleeve A halves): none.
- Minimum distance to the sleeve front half: 0.30 mm (the designed pad-to-boss gap); to the back half 29.2 mm.
- Volume 126.7 cm³ (v7: 48.6 cm³). This is the solid volume; printed with 6 walls and 15 % infill the bow is roughly half full, so the printed adapter is estimated at about 90 g against about 62 g for v7 (**unverified** until sliced). The handheld total rises to about 870 g.
- Stiffness estimate for this section (same beam model): droop about 0.7 mm and sideways about 1.1 mm per 1 g, first wobble about 15–19 Hz, against about 8 mm and 5.5 Hz for a solid v7. Not FEA, not measured.

Not done: STL export and print-queue text (dex1-1-handheld, by pull request), P3 (sleeve joint), P6 (hang eye).

Images: project files `cad-snapshots/2026-11-12-asm-real-path-adapter-v8-side.png`, `…-iso.png`, `2026-11-12-real-adapter-v8.png`.

## Stage 2b: real-adapter v9, clean rebuild (Fusion, 2026-11-12)

Martyn asked for a tidy part (look, ergonomics, fewer sharp edges, rigidity) after noticing that pad and sleeve boss did not line up. The timeline was rebuilt from scratch: 30 features instead of 93, one side-profile sketch drives the bow. Saved as v9; v8 and v7 stay in the version history. All interfaces to the gripper and the sleeve are unchanged, so nothing else has to change.

| Change | Detail |
| --- | --- |
| Joint to the sleeve | The pad is now the top of a 24 mm wide pedestal that is flush with the 16 mm sleeve boss front and back (v8: 14 mm pad, 4 mm overhang in front, 13 mm heel behind). Screw positions, 5 mm pad thickness and the 0.3 mm gap are unchanged |
| Bow shape | One continuous 24 mm wide band. Bends rounded with R 30 on the finger side and R 52 outside instead of kinks; one kink of the v7 finger-side line removed (about 2 mm more finger room there) |
| Edges | Finger-side long edges R 3, outside long edges R 2, pedestal corners R 6 to R 12, rear plate corners R 4 |
| Riser to plate | Solid 24 mm wide gusset up to z = 8 replaces the two thin ribs of v8 |
| Screw access | Pad screws: two Ø6.5 tunnels straight through to the underside. Lower rear-mount screws: two Ø6.5 key tunnels through the gusset. Upper rear-mount screws are in the open |

Kept as in v8: rear plate 36 × 55 × 5 with four Ø3.4 holes and Ø6.5 × 3 head bores, tie beam, clamp ring with split tabs and Ø3.4 clamp hole.

Checks run:
- Timeline: 30 features, none unhealthy. One solid body, 129.7 cm³ (v8: 126.7 cm³), so the printed mass estimate stays at about 90 g (**unverified** until sliced).
- Point test along all six screw axes (2 pad, 4 rear mount): free from the screw seat to the outside.
- `ASM real-path` (refreshed in memory, not saved): no interference with the gripper STEP bodies or sleeve A; 0.30 mm to the sleeve front half (designed gap); 0.0 mm to the cable flange block (plate seat); 0.1 mm ring to motor barrel (as in v7).
- Stiffness: section is the same 24 × 22 mm class as v8, so the v8 hand estimate carries over (not recomputed, not FEA, not measured).

Sleeve (controller-grip v21): left unchanged. The boss already ends 1 mm from the pommel-side sleeve rim and the little finger sits directly in front of it, so the screw spacing cannot grow in either direction without touching what the v21 tests validated (A2 catch, grip size, LED coverage).

Known leftovers:
- The pedestal (24 mm, centred on x = 0) is wider than the sleeve boss (14 mm, centred on x = −2.4), so a step remains across the width. Removing it means widening the boss in controller-grip.
- The wall between the front pad-screw tunnel and the pedestal front face is 0.75 mm (two extrusion lines).
- Printing on its side puts one set of the R 2 / R 3 edge rounds on the bed; expect a slightly rough first layers there.

Images: project files `cad-snapshots/2026-11-12-asm-real-path-adapter-v9-side.png`, `…-v9-underside.png`, `2026-11-12-real-adapter-v9.png`.
