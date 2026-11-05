---
date: 2026-11-05
tags: [research, handheld, ergonomics]
---
# Trigger travel and grip span for the copy-path trigger

Question: how much squeeze travel can the multi-finger trigger of the copy path ([[D10 Handheld mechanism]]) have, given the grip sleeve it sits on? The 20 mm used in `copy-trigger` v2 was an unsourced placeholder; at 30 mm from the pivot it needs a 34° blade opening and puts the blade end about 40 mm off the sleeve (Fusion, 2026-11-05).

Method: ergonomics sources found by web search on 2026-11-05, read as abstracts or chapter text (full papers not read). None of them is in `msc-thesis/thesis/literature.bib`. Sleeve dimensions read from `ASM copy-path` v9 in Fusion.

## Findings

| Source | What it says | Where |
| --- | --- | --- |
| Lee, Kong, Lowe, Song 2009, "Handle grip span for optimising finger-specific force capability as a function of hand size", *Ergonomics* ([PubMed 19424925](https://www.pubmed.gov/19424925/)); **not in bib**; authors and journal from the title search, unverified | Spans 45–65 mm tested. 50 and 55 mm were rated most comfortable and gave the largest grip strength (430.8 N, 433.6 N). Small and medium hands preferred 50–55 mm, large hands 55–60 mm; 65 mm was least preferred | Abstract |
| ILO Encyclopaedia of Occupational Health and Safety, Ergonomics, hand tools ([item 634](https://iloencyclopaedia.org/part-iv-66769/ergonomics-52353/item/634)); **not in bib** | "an optimal span, which ranges from 45 to 55 mm for both men and women"; short-term grip strength 450–500 N (men), 250–300 N (women); "for repetitive action the recommended requirement is probably closer to 90 to 100 newtons for men, and 50 to 60 newtons for women" | Grip Strength and Hand Span |
| Same | Power tools: trigger "operated by the index finger"; it "should incorporate an easily reset latching mechanism to hold the power on when required" | Saws and power tools |
| Sengupta, IE665 *Hand tools* lecture (NJIT, [PDF](https://web.njit.edu/~sengupta/IE665/Hand%20tools.pdf)); **not in bib** | If a trigger must be held, reduce the activation force or "provide a power grip switch or a bar – instead of a single finger switch" | Slide 4 |
| Chang, Freivalds, Sharkey, Kong, Kim, Sung, Kim 2014, "Investigation of triggering force by the index finger for optimal trigger design using a cadaver experiment", HFES Proc. pp. 1546–1550 ([Penn State](https://pure.psu.edu/en/publications/investigation-of-triggering-force-by-the-index-finger-for-optimal/)); **not in bib** | Trigger spans 40, 50, 60 mm; highest triggering force at 50 mm; 50 mm recommended | Abstract |

No source gives a trigger *travel*; they give the span (distance from the palm side of the handle to the finger side) at which the hand is strongest and most comfortable.

## What follows for the copy path

- **Grip span is set by the sleeve, before any travel.** Sleeve A measures 56.9 mm across the grip from back to underside (n = 58.9 to 115.8 mm in the grip frame; Fusion). With the blade (4 mm) and a 3 mm gap, the closed span is about 64 mm, already above the 45–55 mm optimum and at the 65 mm that Lee et al. found least preferred.
- **Travel therefore adds to an already wide span.** Each millimetre of travel widens the open span. With 20 mm of travel the open span would be about 84 mm, outside every range above (computed).
- **Consequence for the design (for Martyn to decide):**
  1. Keep the travel small, about 8–10 mm (unverified; no source gives a travel), and let the link do the rest. 41.6 mm per finger side from 10 mm needs about 4:1 (computed).
  2. Recess the blade into the sleeve underside so that the closed span drops towards the sleeve's 57 mm; this needs a pocket in the sleeve front half.
  3. Force budget: at 4:1, the finger force is about four times the carriage force (spring plus friction, ideal linkage). Keeping it below the ILO repetitive figure for women (50–60 N) limits the carriage-side force to about 12–15 N (computed).
- The ILO latch note and the NJIT "bar instead of a single finger switch" note support the already chosen multi-finger trigger and continuous lock ([[D10 Handheld mechanism]]).

## Open
Reviewed 2026-11-05: the copy path is frozen as fallback ([[D01 Handheld gripper body]] History), so these apply only if it is resumed.
- Hand size of the operators is unknown; Lee et al. relate the best span to hand length (normalised grip span 27–29 %).
- Measure the real squeeze travel and force on the first print.
