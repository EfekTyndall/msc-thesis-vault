---
date: 2026-11-18
tags: [report, handheld, real-adapter, print-test, petg]
---
# Plate 2b clamp: failure diagnosis

Plate 2b is the clamp part of real-adapter v13: the rear plate, the tie beam (tab) and the split clamp ring. It was printed in Spectrum PETG on the H2S, standing on the plate's rear face, with grid supports and 2 PLA contact layers (`dex1-1-handheld/hardware/PRINT-TEST-QUEUE.md` §3). Martyn reported three failures on 2026-11-18 (photos in the project thread). This note gives the cause of each, taken from the Fusion geometry and the slicer project, and a proposed fix. Nothing in Fusion or the slicer was changed.

## Observed (Martyn, photos, 2026-11-18)

1. The ring snapped off the tab while the supports were being removed, on two prints. Both fracture faces are flat and show the grid infill. The stub stays on the plate.
2. The plate cracks in its thickness, starting at the locating groove and running sideways across the plate.
3. The ring cannot be opened far enough to fit the gripper, and it breaks when forced.
4. Overall, the part feels brittle.

## Inputs checked

- **Fusion, read only** (real-adapter v13, ASM real-path v17, 2026-11-18):
  - Plate: 36 × 55 × 5 mm (y −16.94 to −11.94).
  - Locating groove: on the rear face, 5.2 mm wide, 1.7 mm deep, 45° flanks, z −33 to 5.
  - Tab: 10 × 7 mm (x ±5, z −33 to −26), runs y −11.94 to 30, with an R3 fillet at the plate root only.
  - Ring: Ø49.2 bore, 4 mm wall, y 18 to 30. The split is at the top (2 mm slot) with one M3 lug pair.
- **Slicer project** (`real-adapter/print-files/plate2b-real-adapter-v13-clamp-PETG.3mf`, local on the Mac): 0.2 mm layers, Arachne, 4 walls, 15 % grid infill, normal supports (threshold 30°, support also on the model), PETG 255 °C, fan 40–90 %.

## Diagnosis

| # | Failure | Cause | Evidence |
| --- | --- | --- | --- |
| 3 | Ring can't be fitted | **The one-piece closed ring cannot be assembled at all.** Ring flex is not the problem. Fitting the ring over the Ø49.0 housing from the side means opening the 2 mm slot to about 49 mm. Sliding the gripper in axially from the front fails too: the parts that must end up between the plate and the ring stick out past the Ø49.2 bore. These are the cable guard cover (out to r = 33.7 mm), its 2 countersunk M3 × 14 screws (r = 30.1 mm) and the flange block (r = 24.65 mm against a bore radius of 24.6). Sliding it in from behind is also ruled out, because the gripper is 159 mm long and the gap between plate and ring is 30 mm. | Fusion ASM real-path v17: radial extents of the dex1-1-ref bodies in the plate-to-ring gap (mesh, low quality). The insertion path was never checked (open item since v10, memory). |
| 1 | Ring snaps off the tab | In this orientation the tab stands vertical, so each layer is a cross-section of the 10 × 7 mm tab. Any load on the ring bends the tab across the layer lines, which is the weakest direction of an FDM part. The section also steps abruptly from tab to ring at y = 18, and that step lies on a layer plane. Prying grid supports out from under a ring of about 57 mm diameter gives a long lever arm on that joint. | Flat fracture faces showing infill (photos). FDM parts are "weakest … along the Z axis—perpendicular to the layers" and should be oriented so the "greatest stresses are along the X or Y axes" ([Polymaker wiki](https://wiki.polymaker.com/the-basics/fun-3d-printing-facts/print-orientation-affects-strength), not in bib). |
| 2 | Plate cracks at the groove | Layer separation. The groove is on the bed face, so its roof (1.7 mm, about layer 9) is a layer plane. The groove's sharp roof corners act as a notch on that plane, and the crack runs sideways along it. | The crack in the photos runs parallel to the plate faces at about groove depth (inferred from the photos, not measured). Same mechanism as #1. |
| 4 | Brittle overall | Mostly #1 and #2: the loads cross the layers. Material and settings are less likely to be the cause. Nozzle 255 °C is inside the Spectrum range, and the fan (40–90 %) is at or below the TDS's 75–100 %. The 15 % sparse core in the 10 × 7 tab adds to the weakness. | Spectrum PETG HS TDS (see the 2026-11-16 plate 1 report). [Prusa: layer separation](https://help.prusa3d.com/article/layer-separation-and-splitting-fdm_1806) (not in bib) names too much cooling and too low a temperature as causes, but for ABS/PLA. Not shown for this PETG (unverified). |

## Martyn's ideas checked against this

- **More flexible ring (thinner, or TPU):** this does not fix #3, because no realistic flex lets the gripper pass. It would also soften the controller-to-gripper chain, and stiffness is the top requirement for the real path ([[D10 Handheld mechanism]], 2026-11-12 review). Not recommended.
- **Two-piece ring with two screws:** this is what the geometry needs. Ruland: "two-piece shaft collars can be disassembled and installed in position without the need to remove other components", while "one-piece designs must slide over the end of a shaft", and "two-piece clamp style collars possess more holding power than one-piece styles" ([Ruland, What are shaft collars](https://www.ruland.com/technical-resources/technical-articles/technical-article-what-are-shaft-collars), not in bib). Recommended.
- **Stronger clamp-to-body joint:** yes. The biggest gain comes from print orientation (below), not from adding section.

## Proposed fix (needs Martyn's go before any CAD change)

1. **Split the ring horizontally at the axis height (z = 0)** into a lower half on the tab and a separate cap. Use 2 lug pairs at x ≈ ±31, each with an M3 cap screw and nut, as on a two-piece collar. The gripper is lowered into the lower half from above, and the cap is then screwed on. Nothing has to pass through the bore.
2. **Print orientations:**
   - Clamp base: on its z = −33 face. The plate's bottom edge and the tab's bottom face are coplanar there (557 mm²). The tab and plate then lie in the layers, the groove runs vertically (no notch on a layer plane), and the lower half ring is a self-supporting arch.
   - Cap: flat on a ring end face (y = 18 or 30), so the hoop stress stays in the layers.
3. **Small additions:** add a fillet where the ring meets the tab. Raise the infill of the clamp parts or make them solid, since they are small. These are standard practice; I have no specific source for the values (unverified).
4. **Check before printing:** in ASM real-path, check the lug and screw clearance against the cable guard cover (x 21 to 33, y −4.4 to 15.6; the ring starts at y = 18) and against the bow and fingers. Check the screw access too.

## Open

- Martyn's go on the split and orientation, then the CAD change (Fusion real-adapter v14) and the export and print queue (dex1-1-handheld PR).
- The fan and cooling effect on PETG layer strength is unverified. If the reoriented print still feels brittle, run a short layer-strength coupon test.
