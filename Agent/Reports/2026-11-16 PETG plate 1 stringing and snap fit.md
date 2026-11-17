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
2. (Answered 2026-11-16, Martyn: spool new, not dried. Moisture therefore less likely, not excluded: Bambu gives 2–12 h at about 55 % RH for dried filament to degrade, and the plate ran about 4 h; the spool was opened and loaded into the AMS just before the print (Martyn), so exposure was only the print time. Moisture is unlikely; a factory-wet spool cannot be excluded (unverified). Temperature and the untuned profile become the leading stringing causes.)
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

## Update 2026-11-16 evening

- AMS 2 Pro (closed, desiccant inside) read 30 % humidity at 21:17 (Martyn, screen photo). Bambu's drying page gives 2–7 days of good quality at 20 % RH and 2–12 h at about 55 %.
- Plan agreed in the thread: dry the PETG spool overnight in the AMS 2 Pro (AMS 2 Pro PETG preset shortened from its 12 h to 8 h by Martyn, matching the Bambu wiki 60–70 °C / 8 h; PLA spools removed first). Then print A2 seam rings at 240 °C and 255 °C from the dried spool, so temperature is the only variable. AMS drying steps and maximum temperature not checked against Bambu's manual.

## Update 2026-11-17 morning (Martyn)

- AMS humidity after 8 h PETG drying: 14 % (was 30 % before).
- Snap: the PETG v22 halves **do** engage and hold, but needed several tries; on 2026-11-16 they were hard to snap, on 2026-11-17 they snap well. So the failure mode was "hard to engage", not "won't hold".
- Calipers ("around", position along the seam and whether over the bead not recorded):

| Print | Groove width | Tongue thickness |
| --- | --- | --- |
| v22 PETG | ≈ 1.30 mm | ≈ 1.30 mm |
| v21 PLA | 1.20–1.50 mm, varies | 1.40–1.50 mm, varies |

- CAD reference: last read in v19 (2026-11-09) as tongue 0.98 mm in a 1.27 mm groove, 0.14–0.15 mm clearance per side; the A2 rework lengthened the tongue (1.0 → 2.0 mm) and deepened the groove but is assumed to keep these widths (not re-read in v22).

Reading (reasoning, not measured further):
- The PETG tongue measures as wide as the groove, i.e. no running clearance where CAD has about 0.29 mm total. If the tongue reading excludes the bead, the tongue is about 0.3 mm over CAD, which points to over-extrusion or swelling of the thin feature (Generic PETG profile, flow not calibrated). If it includes the bead, the PETG bead is lower than the PLA one.
- Hard first engagement that eased after a few cycles fits a zero-clearance fit plus strings or edge burrs that were worn off by the first attempts.
- The PLA values scatter by up to 0.3 mm, the same order as the catch (0.45 mm) and the clearance; repeatability of these thin features is the limiting factor in both materials.
- PETG holds once engaged, so the lower-modulus concern (step 6, larger catch) is not needed for now, and the result supports D10 (PETG final sleeve).

Next:
1. Martyn: say where the calipers sat (over the bead or on the flat tongue, and at which height).
2. Seam rings at 240 °C and 255 °C from the dried spool (stringing test, unchanged).
3. Flow-rate calibration for this PETG; re-measure tongue and groove on a ring. If the tongue is still ≥ 0.2 mm over CAD, consider widening the groove in CAD (Fusion change, waits on Martyn's go).
4. PETG cycle test (20 open/close cycles) and repeatability, as planned in `PRINT-TEST-QUEUE.md` §1.
- 2026-11-17 10:39 (Martyn): tongue measured on the flat part, not over the bead. So the PETG tongue is ≈ 1.30 mm against ≈ 1.0 mm in CAD (+0.3 mm, v19 value, v22 not re-read), while the groove (≈ 1.30 vs 1.27 mm) is near size. The tight fit comes from the thick tongue. First fix: flow-rate calibration (step 3); CAD clearance change only if the calibrated tongue stays ≥ 0.2 mm over.

## Calipers per seam (Martyn, 2026-11-17 11:33; tongue on the flat part, three readings each)

| Print, seam | Tongue (mean, range) | Groove (mean, range) | Groove − tongue |
| --- | --- | --- | --- |
| v21 PLA, short (+x) | 1.38 (1.36–1.40) | 1.36 (1.34–1.38) | −0.01 |
| v21 PLA, long (−x) | 1.40 (1.37–1.44) | 1.46 (1.22–1.75) | +0.05 |
| v22 PETG, short (+x) | 1.35 (1.30–1.42) | 1.34 (1.30–1.41) | −0.01 |
| v22 PETG, long (−x) | 1.45 (1.41–1.50) | 1.36 (1.31–1.42) | −0.09 |

Means computed from the three readings. Caliper jaws in a narrow groove read less reliably than over a tongue (assumption).

Reading:
- In **both** materials the tongue prints about as wide as the groove; the CAD running clearance (≈ 0.29 mm total in v19, v22 not re-read) is gone. Since PLA shows the same, this is mainly a geometry/printing effect of the thin seam features, not a PETG flow problem. Flow calibration may trim it but is unlikely to recover 0.3 mm alone (reasoning).
- The PETG **long seam** is the tightest (tongue ≈ 0.09 mm over the groove), so the hard first engagement most likely came from that side.
- The PLA long groove scatters 1.22–1.75 mm, which may be why PLA v21 still went together (and later loosened).
- Next: read the v22 tongue and groove widths of both seams in Fusion (read only, needs Martyn's go) to get the true CAD reference; then decide a CAD clearance change (e.g. thinner tongue) together with the flow-calibrated ring test.

## CAD reference from the v22 STLs (2026-11-17, read only)

**Provisional:** Fusion did not answer (connector timeout), so the widths were read from `controller-grip/files/sleeve-A-{front,back}-v22.stl` (exported 2026-11-16 13:15) by horizontal sections in print orientation (z = 8, 20, 29, 40, 50 mm; own numpy script, nothing changed):

| Feature (v22 CAD) | Short seam | Long seam |
| --- | --- | --- |
| Groove width (front half) | 1.29–1.31 mm | 1.29–1.32 mm |
| Tongue thickness (back half) | 1.00–1.03 mm | 1.00 mm |
| Tongue length / bead | ≈ 1.95 mm / bead 0.60 mm proud near the tip | same |

Re-read in Fusion once it is reachable (Martyn's rule 2026-11-17: no STL fallback). Provisionally, both seams are the same in CAD: 0.15 mm clearance per side, constant over the height. So the short/long difference Martyn measured is a print (or measuring) effect, not a design difference.

Against the calipers: grooves print close to size (PETG 1.34–1.36, PLA 1.36–1.46 mm), tongues print 0.35–0.45 mm thick in both materials (PETG 1.35–1.45, PLA 1.38–1.40 mm). An oversize this large in both PLA and PETG is more than flow error would explain (reasoning); possible contributors are the 0.60 mm bead and the base of the 1.95 mm tongue being caught by the caliper jaws, or the wall generator on a 1.0 mm feature (not checked in the G-code).

Martyn (2026-11-17): a thinner tongue will not help because small details do not print. Agreed for tweaks of a tenth of a millimetre: the 0.15 mm clearance is already below what these prints hold. Decision on any CAD change waits for the flow-calibrated ring test.

Fusion check (2026-11-17 10:45, Fusion reachable, controller-grip v22, component `sleeve-A barbed tongue`, read-only pointContainment scans at y = 42 mm): short seam at z = −1.5 mm, groove 9.18–10.50 mm → 1.30 mm, tongue 9.34–10.32 mm → 1.00 mm (0.02 mm steps); long seam at z = −1.3/−1.8 mm, groove ≈ 1.36 mm, tongue ≈ 0.96 mm (0.08 mm steps, coarse). This confirms the STL values above (no longer provisional at y = 42; other heights only from the STLs).

## Flow calibration pass 1 (2026-11-17 11:50, Martyn photos)

Bambu Studio flow calibration could not go past pass 1 ("please update Bambu Studio"). Pass-1 blocks −20 … +20 (Generic PETG @BBL H2S, dried spool), judged from photos: +20/+15/+10 rough and ridged (over-extruded), −10/−15/−20 with visible line gaps (under-extruded), 0 and +5 smoothest; Martyn confirmed **+5** as smoothest by touch (Generic PETG @BBL H2S flow ratio 0.95, read from the ring project file; new ratio 0.95 × 1.05 ≈ 1.00). So the profile flow is correct or at most 5 % low: **over-extrusion does not explain the ≈ 0.4 mm thick tongue**. Rings are printed with the unchanged flow (files as sliced, comparable with plate 1; +5 % would change the tongue by ≈ 0.05 mm, computed). The ×1.05 ratio goes into a saved Spectrum PETG preset for later prints.

## Ring test 240 °C vs 255 °C (Martyn, 2026-11-17 13:39; dried PETG, flow 0.95, ring A2 STLs of 2026-11-09)

| | 240 °C | 255 °C |
| --- | --- | --- |
| Stringing | minimal | none |
| Surface | smooth | slightly smoother |
| Snap | engages well, pulls apart easily | same |
| "Supports" | hard to remove, residue on tongue and groove tips | harder to remove |

Reading:
- **Stringing came from moisture, not temperature.** After drying (AMS 30 % → 14 %), 255 °C gives no strings, so 255 °C (label range 230–270 °C) stays. The temperature hypothesis is dropped.
- **Hold:** a 10 mm ring has about 1/6 of the 58 mm sleeve's seam length, so a weak ring hold is expected; the plate-1 PETG sleeve holds once snapped (Martyn, 2026-11-17 morning).
- **Residue:** the ring files were sliced with supports off (slice check 2026-11-17 10:05), so the material on the tongue and groove ends is the 8 mm outer brim (confirmed by Martyn 2026-11-17 13:41; brim-object gap 0.1 mm), which hugs the thin seam features at the bed and bonds harder at 255 °C. The full sleeve prints with the same brim, so the same residue may sit at the bed end of each seam and add to the hard first engagement.

Next (proposed, slicer changes wait on Martyn's go):
1. Photo of the ring residue to confirm brim vs support.
2. If brim: keep brim off the seam ends (painted brim ears away from the seams, or a larger brim-object gap) for the next sleeve.
3. Run the planned 20-cycle open/close and repeatability test on the existing plate-1 PETG sleeve now; reprint (dried, 255 °C, flow 1.00, brim fixed) only if it fails.

## Cycle test (Martyn, 2026-11-17 13:46)

Plate-1 PETG sleeve A v22: 20 open/close cycles, still holds afterwards (Martyn, by hand, not measured). This is the check named in D10 "Revisit if" (A2 cracks or loosens in the PETG cycle test): passed, so the evidence supports PETG for the final sleeve. Still open: repeatability (pencil mark, `PRINT-TEST-QUEUE.md` §1); painted brim away from the seams for the next sleeve print (waits on Martyn's go); flow ratio 1.00 preset for later prints.

## Summary

- Stringing: moisture; fixed by drying (AMS 2 Pro, 8 h); 255 °C kept.
- Hard snap: zero running clearance (tongue prints ≈ 1.35–1.45 vs 1.00 mm CAD in PLA and PETG, not explained by flow) plus brim residue at the seam ends; the sleeve engages after a few cycles and then holds.
- PETG sleeve passes the 20-cycle test; D10 not questioned.

## Repeatability (Martyn, 2026-11-17 14:14)

PETG v22 with pencil marks: the controller does not self-locate in the sleeve; because of the slight play it must be shifted to the marks by hand at each mounting, and it then holds there. Martyn will show in the data-collection tests whether it stays. Open risk (reasoning): a hand-aligned mount can shift the controller-to-gripper offset between sessions, so the offset should be checked per session (calibration, D09) until the tests show it is stable.
