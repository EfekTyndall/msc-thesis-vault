---
date: 2026-11-09
tags: [research, handheld, grip-sleeve, print-test]
---
# Grip sleeve snap-fit closure

Question: why the halves of sleeve variant A do not hold, and how to size a snap-fit that does ([[D10 Handheld mechanism]]; `msc-thesis/hardware/PRINT-TEST-QUEUE.md` §1).

Method: PLA fit print of variant A (Martyn, 2026-11-09, lab Bambu Lab H2, photos in the project thread), then design guides and datasheets read on 2026-11-09 (web search, standard; no deep research). None of the sources is in `msc-thesis/thesis/literature.bib` (all "not in bib"). Barb geometry read from Fusion (controller-grip v19) on 2026-11-09.

## Print result (PLA, variant A)

- The controller fits, including the known risk spot below the grip-button window (+x). The grip-button window is clear.
- The halves do not hold each other. Released, both halves fall off. With a zip tie around the closed halves, the controller slides in from below with little force (Martyn).
- Variant B: the front half failed mid-print (knocked off the plate). Martyn tried the unfinished half and found the hook variant uncomfortable, so B is dropped.

## Sources

| Source | What it gives |
| --- | --- |
| Sculpteo, "How to connect two parts with 3D printed joints and snap fits" ([blog, 2018](https://www.sculpteo.com/blog/2018/04/25/how-to-connect-two-parts-with-3d-printed-joints-and-snap-fits/)) | Qualitative only, no numbers. Joint types: cantilever (the "most common kind"; U, L, separable, annular cantilever, long clips), annular (circular parts only), torsion, screw (round parts only), gutter. Example parts in MJF nylon PA12; "very rigid 3D printable materials are not recommended". |
| Bayer MaterialScience, "Snap-Fit Joints for Plastics – A Design Guide" ([third-party copy](https://fab.cba.mit.edu/classes/S62.12/people/vernelle.noel/Plastic_Snap_fit_design.pdf), undated, injection moulding) | Formulas and strain limits below. Nothing on 3D printing. |
| Hubs, "How to design snap-fit joints for 3D printing" ([knowledge base](https://www.hubs.com/knowledge-base/how-design-snap-fit-joints-3d-printing)) | FDM clearance 0.5 mm; root fillet ≥ 0.5 × base thickness; width ≥ 5 mm; taper the beam towards the tip; avoid cantilevers built along Z (layer bonding); PETG among "tough, ductile polymers", PLA "brittle", "less suitable". No strain values. |
| Ultimaker PETG TDS v1.00 ([PDF](https://um-support-files.ultimaker.com/materials/2.85mm/tds/PETG/Ultimaker-PETG-TDS-v1.00.pdf), printed bars, XY / YZ / Z) | Tensile modulus 1939 / 1874 / 1711 MPa; elongation at yield 5.9 / 6.0 / – %; at break 7.6 / 6.4 / 1.8 %. |
| Ultimaker PLA TDS ([PDF](https://um-support-files.ultimaker.com/materials/2.85mm/tds/PLA/Ultimaker-PLA-TDS-v5.00.pdf), printed bars) | Tensile modulus 3250 / 3292 / 3071 MPa; elongation at yield 3.4 / 3.4 % / "no yield" (Z); at break 7.8 / 4.2 / 2.0 %. |
| Glock shell reference (`msc-thesis/hardware/pico-4-ultra-glock-shell/README.md`) | Joins its two grip halves with separate parts (lower connector plate, side connector), not snap-fits. How these are fixed is not documented. |

Bayer guide, used here:
- Tapered cantilever (thickness h at the root to h/2 at the hook), permissible deflection y = 1.09 · ε · l² / h (recovered from the worked example, p. 16; Table 1 is garbled in the copy). Tapering gives "more than 60 %" more deflection than a constant section (p. 13).
- Deflection force P = (b · h² / 6) · E · ε / l; mating force W = P · (µ + tan α) / (1 − µ tan α), α = lead angle; separation uses the same with the return angle (p. 15).
- Allowable strain: amorphous plastics up to about 70 % of the yield strain (p. 11); for frequent separation use about 60 % of the single-assembly value (Table 2, p. 12). Root radius not below 0.38 mm (p. 8).
- No values for PETG or PLA, and no friction coefficient for either. The ABS value 0.50–0.65 (×1.2 for the same material on both parts) is the closest, assumed here.

## Why A does not hold (measured in CAD)

Section sketches of `sleeve-A barbed tongue` in controller-grip v19 at y = 42 mm, read on the +x seam side (radial positions in x, z ≈ 1.8 mm):

| Feature | Outer edge x (mm) |
| --- | --- |
| Sleeve outer surface | 11.64 |
| Recess in the groove (`recess_out`) | 10.86 |
| Bead on the tongue (`bead_out`) | 10.71 |
| Groove outer wall (`groove_out`) | 10.56 |
| Tongue outer face (`tongue_out`) | 10.42 |
| Tongue inner face (`tongue_in`) | 9.44 |
| Groove inner wall (`groove_in`) | 9.29 |
| Sleeve inner surface | 8.21 |

- The wall is 3.4 mm thick. The tongue is 1.0 mm thick with 0.14–0.15 mm clearance on each side of the groove.
- The bead stands 0.29 mm proud of the tongue. Net of the groove clearance, it overlaps the groove wall by only **0.15 mm**, and the recess gives it 0.15 mm room. So the catch is about 0.15 mm.
- The bead and the recess run as continuous lofts along the whole seam, so the joint is a long, stiff rib, not a free beam.

A 0.15 mm catch is below the dimensional scatter of FDM. Hubs recommends 0.5 mm clearance for FDM fits. The PLA halves therefore close without a felt click and do not hold, as observed. The beam sizing below uses a 1.0 mm catch.

## Allowable strain (computed from the sources above, unverified for printed parts)

| Material (XY bars) | Yield strain | × 0.7 (amorphous, single) | × 0.6 (repeated) |
| --- | --- | --- | --- |
| PETG | 5.9 % | 4.1 % | 2.5 % |
| PLA | 3.4 % | 2.4 % | 1.4 % |

PLA is semi-crystalline to amorphous depending on grade. The 0.7 factor is the guide's amorphous rule applied to both, and is an assumption.

## Draft hook (to check in CAD and print)

Four separate tapered cantilever hooks (two per seam side) on the back half, cut free from the wall by slots, catching a ledge in the front half:

| Parameter | Value | Basis |
| --- | --- | --- |
| Length l | 10 mm | chosen to fit the 58 mm sleeve height |
| Root thickness h, tip | 1.5 mm → 0.75 mm | Bayer taper to h/2 |
| Width b | 6 mm | Hubs ≥ 5 mm |
| Undercut (catch depth) y | 1.0 mm | chosen |
| Strain at full deflection | ε = y · h / (1.09 · l²) = 1.4 % | Bayer formula; ≤ 2.5 % PETG repeated, = 1.4 % PLA repeated |
| Deflection force per hook | P ≈ 6 N (E = 1939 MPa) | Bayer formula, PETG XY |
| Closing force per hook | ≈ 11 N (lead angle 30°, µ = 0.6) | Bayer, factor 1.8 |
| Opening force per hook | ≈ 24 N at a 45° return angle (µ = 0.6) | Bayer, factor 4.0 (computed) |
| Root fillet | 0.75 mm | Hubs 0.5 · h (Bayer minimum 0.38 mm) |
| Print orientation | beam axis in the layer plane (horizontal), never along Z | Hubs |

With four hooks: about 43 N to close and 96 N to open (computed, µ assumed). If opening by hand is too hard, lower the return angle. A 90° return makes the joint permanent (Bayer: separability set by bead size and return angle).

The sleeve was printed upright (layer lines run around the grip in the photos), so the hook beams must run around the circumference, not along the sleeve axis. Otherwise they lie along Z and break at a layer line (Hubs).

## Options

1. **Cantilever hooks as above.** Bayer and Hubs give sizing and print rules, and Sculpteo calls it the most common type. PLA sits at its repeated-use limit, so check fit only in PLA and cycle in PETG.
2. **Separate connector parts** (Glock shell reference). Precedent in the reference shell. Its fixing method is unknown.
3. **One-piece slide-on sleeve.** The PLA zip-tie test shows the closed sleeve slides on. It needs a removable stop under the pommel so the sleeve cannot slide off under the gripper load (reasoning, no precedent checked).
4. **2 × M3 screws across the seam into heat-set inserts.** Same inserts as the adapter boss ([[2026-11-06 M3 heat-set inserts for the adapter boss]]). No snap-fit sizing needed. Not from the snap-fit sources.

## Ring test (chosen 2026-11-09, Martyn: prototypes from what A already has)

The internal cantilever hooks do not fit. The front-half wall at +x is 3.25 mm (y 25) to 2.75–3.0 mm (y 55), measured with 0.25 mm steps, and it curves about 1.25 mm inward over 10 mm. A hook needs about 3.0 mm straight. So A's tongue-and-bead is kept, and the catch is chosen empirically on 10 mm seam rings.

Fusion "sleeve-A seam rings" v1 (`1 Parts`): both halves of `sleeve-A barbed tongue` (controller-grip v19) cut to y 37–47 mm. STLs are in `msc-thesis/hardware/controller-grip/files/ring-A*-{front,back}.stl` (not in Git).

| Ring | Catch | Change from A |
| --- | --- | --- |
| A0 | 0.15 mm | none (control) |
| A1 | 0.30 mm | tongue lengthened 1.0 → 2.0 mm, groove 1.15 → 2.1 mm; old bead and recess removed; new bead on the tongue's outer face near its tip: lead 45°, 0.1 mm flat, return 60° (to the seam axis); recess = bead + 0.15 mm, outer skin 0.6 mm |
| A2 | 0.45 mm | same, outer skin 0.45 mm |
| A3 | 0.45 mm | as A2, return face 80° instead of 60° (added 2026-11-09 after the A0–A2 test; Bayer: separation force set by the return angle, p. 15) |

- The lead and return angles are chosen so the bead fits on a 2 mm tongue. The Bayer annular example uses 45° / 45° (p. 23).
- Checked: section probe of A1 at y 42 (tongue, bead and recess where intended; one body per half).
- Pass: the halves click shut by hand, hold when shaken and released, and open again without cracking.

Result 2026-11-09 (Martyn, PLA, photos in the project thread):

| Ring | Result |
| --- | --- |
| A0 | Does not stay together (as the sleeve) |
| A1 | Locks; splits under slight pull |
| A2 | Firmest; stays closed when shaken; splits under moderate pull; no cracking reported |
| A3 | Pulls apart more easily than A2 (Martyn, 2026-11-09). The steeper return face did not raise the hold, contrary to the Bayer trend. Cause not checked; possibly the printed shape of the near-vertical step (unverified). |

**Choice: A2** (0.45 mm catch, 60° return) goes into the full sleeve.

The bead and tongue print coarser than modelled. The bead (0.30–0.45 mm high, 0.1 mm flat) and the 1.0 mm tongue are close to one extrusion line wide. Printed upright, this profile lies in the layer plane, so line width, not layer height, limits it (reasoning; line width in the slicer profile not checked).

## Full sleeve and hinge-hook test (2026-11-10)

Martyn's PLA prints on the lab Bambu Lab H2S (photos in the project thread); geometry checks in Fusion ("sleeve-A seam rings" v4–v6, controller-grip v20–v21).

| Print | Change | Result |
| --- | --- | --- |
| Sleeve v20 | A2 seam over the full length | Fits with slight play; holds very firmly but is hard to take apart; print failures along the front half's outer groove wall; pointed tip beside the grip-button window pokes the hand |
| Ring H1 | +x seam as a hook: A2 ring with the groove inner wall removed, so the half can swing open about the outer seam edge (Fusion: free from 5° to 25°; A2 jams from 10°) | Hook does not catch; the 0.45 mm bead is too coarse once the tongue can bend |
| Ring H2 | +x knuckle hinge: 0.9 mm quarter-round lip on the front half, J hook on the back (0.8 mm tongue, 0.9 mm hook, 0.2 mm clearance), 6 mm window (Fusion: free from 5° to 90°, catches on a straight pull) | Hook catches the hinge well; strings and a rough step at the flat window and hook ends |
| Ring H2b | H2 with 45° ends | Lip too thin, snapped; hook has too little room to catch |
| Sleeve v21 | v20 plus: pointed slivers beside the window removed, window top corners R 1.5 mm, 1 mm pry notches in the back half (y 54–61) | **Best so far, kept.** Firm lock, slight play, no sharp edges; notches do not help much; removal by sliding the controller out until the halves open |

- Precedent: none found for a hook-one-side, snap-other-side closure in the sources read on 2026-11-10 (web search, standard). Closest is the evive enclosure lid ([Hackaday log](https://hackaday.io/project/13091-evive-a-prototyping-platform-for-makers/log/47471-designing-snap-joints-for-evives-lid), not in bib): "one snap joint from where the user will open and other two passive snaps for locking". Formlabs' snap-fit guide (read through a summary only, unverified) uses a 1.2 mm protrusion.
- Why the hinge failed (reasoning): the hook must swing inward to release, and the controller leaves only about 1 mm of room inside the 3.0–3.4 mm wall. That caps every hinge feature at about two extrusion lines (0.42 mm each), too thin for PLA.
- Feature size: features about one line wide (the 0.45 mm bead, the 0.4–0.5 mm skin over the recess) print rounded or drop out. A2 works anyway because the groove's inner wall pins the tongue. The v20 slicer project used the Classic wall generator; v21 was sliced with Arachne (Martyn; effect not isolated, since v21 also changed the geometry).
- Result: A2 kept on both seams (D10 history 2026-11-10). Later, cosmetic only: shape the edges around the grip-button window.

## Open

- PETG–PETG friction coefficient (none found), so the closing and opening forces carry the ABS value.
- Which PETG the lab spools are. The Ultimaker values stand in for it.
- PETG cycle test of v21 (20 open/close cycles) and repeatability (`msc-thesis/hardware/PRINT-TEST-QUEUE.md` §1).
