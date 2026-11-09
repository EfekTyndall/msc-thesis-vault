---
date: 2026-11-09
tags: [research, handheld, grip-sleeve, print-test]
---
# Grip sleeve snap-fit closure

Question: why the halves of sleeve variant A do not hold, and how to size a snap-fit that does ([[D10 Handheld mechanism]]; `msc-thesis/hardware/PRINT-TEST-QUEUE.md` §1).

Method: PLA fit print of variant A (Martyn, 2026-11-09, lab Bambu Lab H2, photos in the project thread), then design guides and datasheets read on 2026-11-09 (web search, standard; no deep research). None of the sources is in `msc-thesis/thesis/literature.bib` (all "not in bib"). CAD not measured: Fusion on the Mac did not respond, so the current barb dimensions are not in this note.

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

## Why A does not hold (reasoning, CAD not checked)

A's barb sits on a continuous tongue in a 3.5 mm wall. Nothing in that joint is a free beam, so only the wall itself can deflect, and a stiff, short section allows almost no deflection before it either jams or lets go. This matches the Sculpteo and Bayer descriptions of a snap-fit as a flexible beam that springs back, and the PLA print, where the halves closed but did not hold. To be confirmed against the barb dimensions in Fusion.

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

## Open

- Measure the current barb in Fusion (tongue thickness, barb height, angles) to confirm the explanation above.
- PETG–PETG friction coefficient (none found), so the closing and opening forces carry the ABS value.
- Which PETG the lab spools are. The Ultimaker values stand in for it.
