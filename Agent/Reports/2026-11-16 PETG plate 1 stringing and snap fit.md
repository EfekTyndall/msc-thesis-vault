---
date: 2026-11-16
tags: [report, handheld, grip-sleeve, print-test, petg]
---
# PETG plate 1: stringing and sleeve snap fit

Plate 1 (sleeve A v22 front/back, dummy rear section, insert coupon 4.0/4.1/4.2) was printed on the lab Bambu Lab H2S in Spectrum PETG, Generic PETG filament profile, 255 °C nozzle, 70 °C textured PEI, Arachne, 4 walls, avoid crossing walls on, 8 mm brim, tree supports (`dex1-1-handheld/hardware/controller-grip/print-files/plate1-sleeveA-v22-PETG.3mf`; baseline `dex1-1-handheld/hardware/SLICER-SETTINGS.md`). Martyn reports heavy stringing and that the halves do not snap together (photo in the project thread). This note gives the likely causes with sources, fixes, and how each fix is checked. Nothing in Fusion or the slicer was changed.

## Observed (photo, Martyn 2026-11-16)

- Fine strings across the whole plate: between the two halves, over the inside of the back half, and on the dummy rear section.
- Both halves sit on a wide brim; the dummy rear section is on tree supports.
- Failure mode of the snap (won't engage vs. engages but won't hold) not known yet.

## Sources (read 2026-11-16, standard web search; none in `literature.bib`, all "not in bib")

| Source | Relevant facts |
| --- | --- |
| Spectrum, PET-G Premium TDS ([PDF](https://download.spectrumfilaments.com/TDS/Spectrum_Premium_PETG_TDS_EN.pdf), prepared 2019-05-08) | Nozzle 230–255 °C, bed 60–80 °C, fan up to 100 %, speed 40–120 mm/s. Test bars printed at 240 °C. No drying or shrinkage data. |
| Spectrum, PET-G Premium High Speed TDS ([PDF](https://spectrumfilaments.com/wp-content/uploads/2024/11/en_tds_spectrum_petg_premium_high_speed.pdf), prepared 2024-10-05) | Nozzle 240–265 °C (standard speed), bed 60–80 °C, fan 75–100 %. Drying if wet: at least 4 h at 60 °C. Elongation at yield 6 %, tensile modulus 2990 MPa (table mapping inferred from the PDF text). |
| Bambu Lab wiki, drying filament ([page](https://wiki.bambulab.com/en/filament-acc/filament/dry-filament)) | PETG: 60–70 °C for 8 h in a forced-air oven, or 75–85 °C bed for 12 h on X1/P1. Moisture causes "stringing, lack of material, holes, rough surfaces, and reduced strength". At about 55 % RH, dried filament degrades within about 2–12 h. |
| [[2026-11-09 Grip sleeve snap-fit closure]] | A2 seam: 2 mm tongue, 0.45 mm catch, 45° lead, 60° return, tuned on PLA rings. Tongue/groove clearance in v19 was 0.14–0.15 mm per side. Ultimaker TDS: printed PETG E = 1939 MPa, PLA E = 3250 MPa (XY). Features about one line wide print coarse. |
| Spool label (Martyn, 2026-11-16, slicing thread) | Standard speed 230–270 °C, high speed 250–270 °C, bed 60–80 °C. These ranges match neither TDS exactly, so the grade is uncertain. |
| `SLICER-SETTINGS.md`, plate-1 slice | Travel 1000 mm/s, elephant foot compensation 0.15 mm, outer wall 200 mm/s. Plate 1 had avoid crossing walls on (v21 process). Filament profile (retraction, flow ratio, fan) not captured. |

The Bambu stringing page and PETG HF page could not be read (fetch refused); no Bambu statement on PETG retraction or fan was found.

## Stringing: likely causes

1. **Moisture** (most likely if the spool was not dried). Bambu names stringing as the first moisture symptom; PETG is dried at 60–70 °C (Bambu) or ≥ 4 h at 60 °C (Spectrum HS). Whether the lab spool was dried or how long it was open is unknown.
2. **Nozzle temperature.** 255 °C lies in the upper half of the label's standard-speed range (230–270 °C) and at the low end of its high-speed range (250–270 °C). Plate 1 is not a high-speed print in Spectrum's sense (outer wall 200 mm/s), so the standard range applies (assumption). Spectrum printed its Premium test bars at 240 °C. Higher temperature lowers viscosity and favours oozing (general FDM reasoning, no source read).
3. **Retraction / flow from the Generic PETG profile.** Not captured, so not assessable. Generic profiles are not tuned to this filament (assumption).

## Snap fit: likely causes

| Cause | Mechanism | Fits "won't engage" | Fits "won't hold" |
| --- | --- | --- | --- |
| Strings and blobs in the groove | Strings across the 2.1 mm groove or on the tongue block the 0.15 mm-class clearances | yes | – |
| Over-extrusion (Generic profile, flow not calibrated) | Wider lines thicken the tongue and narrow the groove; clearance is only about 0.15 mm per side | yes | – |
| Elephant foot at the seam base | First layers on a 70 °C bed spread out; compensation is 0.15 mm, set for the baseline (not tuned for PETG) | yes, at the pommel end | – |
| Lower stiffness of PETG | Bayer: deflection force P ∝ E at the same geometry and strain. Ultimaker printed bars give E_PETG / E_PLA = 1939 / 3250 ≈ 0.6, so the same 0.45 mm catch holds with about 40 % less force (computed; Spectrum's printed-bar moduli in the 2019 TDS, 266–360 MPa, are an order lower and look like a different test basis, so not used) | – | yes |
| Rounded bead | At 255 °C and 200 mm/s outer wall the 0.45 mm bead and 0.45 mm skin over the recess may print even rounder than in PLA | – | yes |

Shrinkage: no PETG shrinkage figure in either Spectrum TDS; not assessed.

## Questions for Martyn

1. (Answered earlier: label temperatures, see Sources.)
2. (Answered 2026-11-16, Martyn: spool new, not dried. Moisture therefore less likely, not excluded: Bambu gives 2–12 h at about 55 % RH for dried filament to degrade, and the plate ran about 4 h; whether the bag was sealed with desiccant is unknown. Temperature and the untuned profile become the leading stringing causes.)
3. Snap: do the halves **not go together** (stop before the click), or **click but fall apart**? Is it the same along the whole seam or worst at the pommel end?
4. Calipers, if possible: tongue thickness and groove width at mid-height on the PETG halves (and the same on the PLA v21 halves for comparison).

## Fixes and checks (in order; no Fusion or slicer change without Martyn's go)

| # | Fix | Check |
| --- | --- | --- |
| 1 | Clean the plate-1 halves: heat gun pass for strings, knife along groove and tongue, trim elephant foot at the base. Try the snap again. | If it now clicks and holds, the cause was print artefacts, not geometry or material. |
| 2 | Dry the spool (60–70 °C, 8 h, Bambu) before the next PETG print; keep it in a dry box/AMS with desiccant. | Strings on the next print visibly fewer at the same settings. |
| 3 | Nozzle temperature lower within the label's standard range: start at 240 °C (Spectrum test-bar value). Optionally a temperature tower 230–255 °C. | Strings and bead shape on a 10 mm A2 seam ring. |
| 4 | Avoid crossing walls is already on; if still stringy after 2–3, raise retraction in the filament profile in small steps. | Same ring or a stringing test with two towers. |
| 5 | Flow-rate calibration for this PETG in Bambu Studio (Calibration > Flow rate) and save a Spectrum PETG filament profile. | Tongue and groove widths within about 0.05 mm of CAD by calipers (tolerance chosen, no source). |
| 6 | Only if the halves engage cleanly but do not hold after 1–5: PETG seam rings with a larger catch (e.g. 0.45 / 0.60 / 0.75 mm), same method as the 2026-11-09 PLA rings. | Ring pass criteria from 2026-11-09: clicks shut by hand, holds when shaken, opens without cracking. |

Rings (10 mm seam slices) are faster and cheaper than the full sleeve for steps 3–6.

## D10

The evidence does not yet question PETG as the final sleeve material ([[D10 Handheld mechanism]], 2026-11-16). Stringing and blocked or oversized seams are process faults, and D10's own "Revisit if" (A2 cracks or loosens in the PETG cycle test) has not been reached. The lower PETG modulus is a real design effect (≈ 0.6 × the PLA holding force at the same catch, computed), so the A2 catch may need to grow for PETG (step 6). If dried, calibrated PETG rings with a larger catch still do not hold or crack, D10 should be revisited through record-decision.

## Open

- Answers to the four questions above.
- Bambu's own PETG stringing and fan guidance (page not readable today).
- PETG–PETG friction coefficient (still none found).
