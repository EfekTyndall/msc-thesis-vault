---
date: 2026-11-18
tags: [report, handheld, real-adapter, controller-grip, print-test, petg]
---
# Plate 2c fit and sleeve slip: results and diagnosis

Plate 2c is the reprinted clamp of real-adapter v14 (two-piece ring, base printed on its z = −33 face, cap flat, PETG only), see [[2026-11-18 Plate 2b clamp failure diagnosis]]. Martyn assembled the real-path handheld with the dummy gripper on 2026-11-18 and reported the results below (photos in the project thread). The second part diagnoses why sleeve A v22/v23 slides and twists on the PICO controller under the adapter and gripper load. Nothing in Fusion was changed; the CAD was only read.

## Plate 2c results (Martyn, 2026-11-18)

- Prints are good overall; no breakage at the tab or groove (the plate 2b failure modes).
- The surfaces that sat on supports under the collar are rough, including the ring side that touched the support. Martyn: not a problem for the prototype; for later prints a fix is needed so these surfaces come out clean (open, see below).
- Screws and heat-set inserts are not here yet, so the sleeve–rod joint and the rod–base joint are superglued for now.
- The dummy gripper is mounted on the adapter and the glue holds it.
- Open: the real gripper weight on the adapter, and the screwed joints (2 × M3 into inserts, 2 × M3 × 16 + nuts on the ring), once the parts arrive.
- The sleeve slides and twists a lot around the controller, worse with the added weight of the rod adapter and dummy gripper. Diagnosis below.

## Inputs checked

- Fusion controller-grip v23 (read only): scan mesh "PICO 4 Ultra controller (scan reduced to 1 mm)" and the two sleeve A bodies (front, back), exported as meshes and compared in the scan's own frame (y = controller long axis, pommel at +y).
- Timeline of sleeve A: inner surface lofted from sections at y 12–70; "relief y44 … y65" lofts cut into both halves; A2 seam beads and recesses; window fillets (v21/v22).
- Earlier evidence: calipers 32.98 mm side to side against 32.7 mm on the scan, 40.82 mm back to front (Day close 2026-11-10); PETG v22 repeatability: the controller does not self-locate and is shifted to pencil marks by hand ([[2026-11-16 PETG plate 1 stringing and snap fit]]); combined centre of mass about 134 mm ahead of the grip centre (2026-11-12 review, gripper mass unverified).

## Geometry results (computed)

**Controller shape.** Over the sleeve length the grip tapers steadily towards the pommel: scan section 33.9 × 41.3 mm at y = 25, 25.2 × 30.2 mm at y = 70. Between y 5 and 22 a side lump (x up to 12–13 mm) widens the section; above y ≈ 0 the tracking ring begins.

**Clearance, sleeve inner surface to scan** (about 35 000 surface samples; median per 5 mm band):

| y band (mm) | median gap | share under 0.5 mm |
| --- | --- | --- |
| 12–45 | 0.39–0.47 mm | 59–85 % |
| 45–65 | 0.89–0.92 mm (no point under 0.8 mm) | 0 % |
| 65–70 | 0.30 mm | 80 % |

The 45–65 band is the "relief y44–y65" loft. It runs all the way round, not only near the boss: the 2026-11-06 local 0.5 mm relief for a scan intrusion near the screw boss ended up as a full ring of extra gap. The adapter boss sits at y 53–69, so **the load enters the sleeve exactly where it is loosest**. Of the 58 mm sleeve length, only about 38 mm is in near contact.

**Free play of a rigid sleeve on the scan** (scan moved against the sleeve until it enters the sleeve wall; pivot at the grip centre, y = 41):

- Twist about the grip axis: about +0.2° / −2.2° before contact.
- Rocking about the two cross axes: 0.0–0.7° each way.
- Slide towards the ring: 0.3 mm. Slide towards the pommel: 1.1 mm.
- Model limits: rigid sleeve, scan sampled at about 30 000 points, contact counted when more than 3 points enter the wall, so the values are good to a few tenths.

## Diagnosis

1. **The sleeve holds the controller by clearance and friction only.** A clearance fit is the fit for "components that must move relative to each other"; parts that must act "as a single unit" need an interference fit ([Hubs, clearance, transition and interference fits](https://www.hubs.com/knowledge-base/understandings-clearance-transition-and-interference-fit/), not in bib). The design gap is 0.3–0.5 mm per side, plus print scatter of up to 0.3 mm on these thin features (plate 1 report), so nothing presses the sleeve onto the controller once the seams are closed.
2. **The relief band removes contact where the load enters.** The gripper moment (centre of mass about 134 mm ahead, unverified) is applied through the boss at y 53–69, inside the 0.9 mm band. The sleeve can only react it at the ends of the contact zones, which gives a short lever and lets it rock and twist.
3. **The taper gives only a weak stop towards the pommel.** Every section towards the pommel is smaller, so a rigid sleeve slides about 1.1 mm that way before it touches (the contact spot was not traced). Once the halves spread a little under load, nothing stops it. The gripper weight, about 25° below horizontal with the pommel back (D10), pulls the front of the sleeve down and pushes it along the grip.
4. **A rigid sleeve would move only about 2° and 1 mm, but Martyn sees "a lot".** So most of the motion must come from the sleeve itself deforming: the halves flexing and the A2 seam catch (0.45 mm) letting the halves open under the moment, plus low friction of printed PETG on the smooth controller shell (no PETG friction coefficient found; [[2026-11-09 Grip sleeve snap-fit closure]]). This part is inferred, not measured.
5. The scan is 0.28 mm narrower than the real grip side to side (calipers), so the real side gap is a little smaller than computed; back to front it is likely a little larger. One caliper value per direction, so this is approximate.

## Fix options (none applied; need Martyn's go before CAD)

| Option | What | Source / precedent | Cost |
| --- | --- | --- | --- |
| A. Quick test, no CAD | Line the sleeve inside with tape or thin TPU strips, then remount and check twist by hand | Quest 2 gunstock ring: "slightly loose … I put in some electrical tape … which also helps to make it more snug" ([stlfinder copy of the Thingiverse model](https://www.stlfinder.com/model/improved-and-dual-mount-oculus-quest-2-gunstock-controller-ring-7W3MpYmz/4412118), not in bib) | Minutes; tells whether clearance is the cause |
| B. Close the relief band | Bring y 44–65 back to the base gap and keep relief only at the measured intrusion spot | Own scan data; the 2026-11-06 intent was a local relief | Small CAD change, reprint sleeve A |
| C. Clamp the halves with screws | Two or more M3 screws across the seams so the halves press onto the grip with preload, independent of print tolerance; a thin TPU liner can take up the gap | Ruland: "two-piece clamp style collars possess more holding power than one-piece styles" ([Ruland](https://www.ruland.com/technical-resources/technical-articles/technical-article-what-are-shaft-collars), not in bib); already used for the v14 ring | Larger CAD change; check wall room (3.0–3.4 mm) and the hand position |
| D. Axial stop at the pommel end | A lip past y 70 where the grip narrows (18.8 × 22.5 mm at y 75) | No direct precedent found; must leave the 2 pommel LEDs uncovered (D10) | Medium; LED check needed |

Recommendation: A first, since it costs nothing and separates "too much gap" from "halves opening". If tape fixes it, do B (and optionally a TPU liner). If the sleeve still twists with tape, the halves are opening and C is needed.

## Rough support faces (for later prints)

Open, not urgent (Martyn: fine for the prototype). Candidates to check later: support interface layers or a different interface material only for the collar underside, or reorient so the collar underside is not supported. No source checked yet (unverified).

## Open

- Martyn's choice of option; any CAD change only after his go, in a tab he has open. Resolved 2026-11-18: tape test, then v24 variants (below).
- Real gripper weight on the adapter; screwed joints once screws and inserts arrive. Moved 2026-11-18 to [[Waiting]] (Dex1-1 delivery; Purchases).
- Support-face finish fix for later prints. Moved 2026-11-18 to [[2026-11-19]] TODO.

## Tape test (Martyn, 2026-11-18 16:21)

- One layer of tape lining the inside of the **back half** stops the sliding and twisting.
- Tape in **both** halves is too much: the sleeve no longer snaps shut.
- So the cause is the gap (option A confirmed); the halves opening is not the main effect. Computed back-half gap: median 0.39 mm at y 12–44, 0.89 mm at y 44–65, 0.32 mm at y 65–70; the tape took one tape thickness off all of these. Tape thickness not yet measured.
- Next (waits on Martyn's go): in CAD, offset only the back-half inner surface inward by the measured tape thickness, which copies the tested state; front half unchanged so the snap still closes.

## v24 variants (2026-11-18, 16:30–16:45)

- Martyn chose to print both offsets as full sleeves.
- Fusion controller-grip v24 (saved by Martyn): user parameter `back_inner_offset` (0.3 mm in the saved file) drives "v24 back inner offset", an offset of the back half's 4 inner faces towards the controller. The front half is unchanged. Back-half volume 11.430 → 12.377 cm³ (12.068 at 0.2 mm).
- Check against the scan at 0.3 mm: median gap 0.15 mm at y 12–44, 0.58 mm at y 44–65, 0.14 mm at y 65–70, with a few spots up to about 0.1 mm into the scan (similar to the tape).
- Prints: `sleeve-A-back-v24-offset0.2.stl` and `…0.3.stl` with `sleeve-A-front-v23.stl`. Project `plate3-sleeveA-v24-offset-0.2-0.3-PETG.3mf` uses the plate 1 settings with the Spectrum PETG preset, no brim and no prime tower. Print queue: dex1-1-handheld PR #10.
- Open: test both without tape (snap, removal, no slip under the adapter), then carry the better offset into controller-grip. The decision is to be recorded in D10 once chosen. Moved 2026-11-18 to [[2026-11-19]] TODO.
