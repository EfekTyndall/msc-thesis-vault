---
date: 2026-11-06
tags: [research, handheld, real-path, purchase]
---
# M3 heat-set inserts for the real-adapter boss

Question: which M3 heat-set inserts to buy for the joint between real-adapter (bow pad) and the boss on the sleeve front half ([[D10 Handheld mechanism]]; `msc-thesis/hardware/PRINT-TEST-QUEUE.md` §3), and what hole the boss needs. Current CAD: 2 × M3, holes 4.0 mm × 6 mm deep, PETG.

Method: manufacturer pages and datasheets read on 2026-11-06 (web search, standard; no deep research). No source is in `msc-thesis/thesis/literature.bib`; all are product documents. CAD not changed (Fusion in use by another thread). Boss and pad geometry read in Fusion by the CAD thread on 2026-11-06: boss is a 14 × 14 mm block (face at z −31, 8.5 mm into the sleeve wall), holes Ø4.0 × 6 at (x −7, y 57) and (x −7, y 65); about 9.3 mm of material under the boss face at the holes; bow pad 5.0 mm thick with 2 × Ø3.4 holes, 0.3 mm gap to the boss face.

## Findings

| Insert | L (mm) | Insert Ø (mm) | Hole Ø (mm) | Min. hole depth | Min. wall (mm) | Source |
| --- | --- | --- | --- | --- | --- | --- |
| CNC Kitchen M3 × 5.7 Standard | 5.7 | 4.6 | 4.0 | L + 1 mm = 6.7 mm | 1.6 | CNC Kitchen poster "Dimensions & Design Guidelines" ([PDF](https://cdn.shopify.com/s/files/1/0654/4821/4767/files/Poster_Dimensions-Guidlines.pdf?v=1751444185), linked from [cnckitchen.store downloads](https://cnckitchen.store/pages/insert-cad-models)) |
| CNC Kitchen M3 × 3.0 Short | 3.0 | 4.6 | 4.0 | L + 1 mm = 4.0 mm | 1.6 | Same poster |
| CNC Kitchen M3 × 5 × 4 VORON | 4.0 | 5.0 | 4.4 | L + 1 mm = 5.0 mm | 1.6 | Same poster |
| ruthex RX-M3x5.7 | 5.7 | 4.6 (unlabelled "D1") | 4.0 (unlabelled "D3") | not given | 1.6 (unlabelled "W") | Length and brass: [ruthex.de](https://www.ruthex.de/en/products/ruthex-gewindeeinsatz-m3-100-stuck-rx-m3x5-7-messing-gewindebuchsen); table: reseller [3DJake](https://www.3djake.com/ruthex/threaded-insert-m3-100-pieces) |
| PEM IUTFA/B/C (flanged, straight wall) M3 | 5.74 (reading unsure) | – | 4.01 (reading unsure) | 6.5 mm (reading unsure) | boss Ø ≈ 2 × insert Ø (summary, not verbatim) | [PEM SI catalogue](https://www.pemnet.com/wp-content/uploads/sites/2/2022/06/sidata.pdf) p. SI-8; PDF text garbled, column mapping unverified |

- The poster's blind-hole rule "min. L+1mm" is a diagram label; the poster does not say whether it applies to every insert (assumed here).
- CNC Kitchen tip (3DJake product text): install 10–20 °C above the print temperature; melt only about 90 % of the insert in with the iron, press the rest.
- ruthex gives no hole size on its own pages. One customer review on ruthex.de (2022) found 4.0 mm too tight and 4.4 mm too loose in their print, and used 4.1 mm; anecdotal, printer-dependent.
- Tappex (robots-blocked), RS, Würth, Bossard, Böllhoff and Kerb Konus gave no readable M3 heat-set values in this pass. Industrial inserts (PEM) are specified for moulded/drilled holes, not prints.

## What follows for the boss

1. **Hole diameter 4.0 mm stays.** It matches the Standard and Short inserts of CNC Kitchen (poster) and the ruthex RX-M3x5.7 table value.
2. **Hole depth: 6 mm is too short for a 5.7 mm insert.** The poster minimum is 6.7 mm; 7.0 mm would meet it (computed, rounded up).
3. **Recommendation:** CNC Kitchen M3 × 5.7 Standard (ruthex RX-M3x5.7 has the same published dimensions and is an equivalent second source). Boss hole 4.0 mm × 7.0 mm. Reason: the only insert with a manufacturer-stated hole and depth, and the longest thread engagement of the options at the same 4.0 mm hole.
4. **Fallback** (not needed for depth, see 6): the M3 × 3.0 Short fits the current 4.0 mm × 6 mm hole unchanged (needs ≥ 4.0 mm). Its holding strength in PETG is not given by any source read (unverified).
5. **Wall: too thin along y.** The poster minimum is 1.6 mm. Hole edge to boss edge is 5.0 mm along x but only 1.0 mm along y (holes at y 57 and 65, Ø4.0, boss y 54..68). Two fixes: (a) grow the boss 1 mm at each end (y 53..69), giving 2.0 mm, pad holes unchanged; (b) move the holes 0.6 mm inwards (y 57.6 and 64.4), which also moves the pad holes and leaves 2.8 mm between the holes. (a) is simpler because the pad stays as it is; check it against the ~0.5 mm scan intrusion near the boss (`PRINT-TEST-QUEUE.md` §2). This applies to the Short fallback too.
6. **Depth fits.** A 7.0 mm hole in 9.3 mm of material leaves about 2.3 mm below it (computed).
7. **Screw: M3 × 10 socket head** (ISO 4762). Under the head: 5.0 mm pad + 0.3 mm gap (assumed to be a gap between pad and boss face) = 5.3 mm, so 4.7 mm goes into the 5.7 mm insert, and the tip stays 2.3 mm above the hole bottom (7.0 mm hole). M3 × 12 would put the tip 0.3 mm above the bottom, too close; M3 × 8 engages only 2.7 mm (computed). With the Short fallback in the 6 mm hole: M3 × 8 (2.7 mm into a 3.0 mm insert, tip 3.3 mm above the bottom).

Availability: both CNC Kitchen and ruthex are sold in packs of 100 by 3DJake (EU) and the makers' own shops. Whether the internal Storefront lists either is unknown ([[Waiting]], company IT access).

## Open
- CAD change in controller-grip (real-path front halves): hole depth 6 → 7.0 mm and boss wall fix (a), grow boss to y 53..69 (chosen by Martyn 2026-11-06); not yet applied in Fusion.
- Print fit: print a small coupon with 3.9 / 4.0 / 4.1 mm holes in the lab PETG before the boss, since the review above saw 4.0 mm tight (suggestion; not yet in `PRINT-TEST-QUEUE.md`).
- Pull-out strength in PETG is not covered by any source read; check by hand on the first print.
