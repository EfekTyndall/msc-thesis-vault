---
date: 2026-11-06
tags: [research, handheld, real-path, purchase]
---
# M3 heat-set inserts for the real-adapter boss

Question: which M3 heat-set inserts to buy for the joint between real-adapter (bow pad) and the boss on the sleeve front half ([[D10 Handheld mechanism]]; `msc-thesis/hardware/PRINT-TEST-QUEUE.md` §3), and what hole the boss needs. Current CAD: 2 × M3, holes 4.0 mm × 6 mm deep, PETG.

Method: manufacturer pages and datasheets read on 2026-11-06 (web search, standard; no deep research). No source is in `msc-thesis/thesis/literature.bib`; all are product documents. CAD not changed (Fusion in use by another thread); boss outer diameter and pad thickness not read from Fusion.

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
4. **If the boss cannot take 7 mm** (boss height or the wall underneath unknown), the M3 × 3.0 Short fits the current 4.0 mm × 6 mm hole unchanged (needs ≥ 4.0 mm). Its holding strength in PETG is not given by any source read (unverified).
5. **Wall:** at least 1.6 mm around the 4.0 mm hole, i.e. boss Ø ≥ 7.2 mm (computed from the poster W).
6. **Screw length:** pad thickness + up to the insert length (5.7 mm), with the tip not bottoming in the hole; set once the pad thickness is read from Fusion.

Availability: both CNC Kitchen and ruthex are sold in packs of 100 by 3DJake (EU) and the makers' own shops. Whether the internal Storefront lists either is unknown ([[Waiting]], company IT access).

## Open
- Read boss outer diameter, boss height and pad thickness in Fusion; then set hole depth (7.0 mm) and screw length.
- Print fit: print a small coupon with 3.9 / 4.0 / 4.1 mm holes in the lab PETG before the boss, since the review above saw 4.0 mm tight (suggestion; not yet in `PRINT-TEST-QUEUE.md`).
- Pull-out strength in PETG is not covered by any source read; check by hand on the first print.
