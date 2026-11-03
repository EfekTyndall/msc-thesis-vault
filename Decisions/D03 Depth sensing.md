---
date: 2026-10-19
status: accepted
tags: [decision]
---
## Context

Whether depth is used as a policy input, given the D405 that was ordered with the grippers. Source: `msc-thesis/docs/proposal.md` §3 (approved by Supervisor on 19 Oct 2026).

## Options

| Option | Strengths | Weaknesses | Verdict |
| --- | --- | --- | --- |
| RGB as policy input; D405 mounted and recorded for a later comparison | Matches GR00T and π0.5 as they are; the comparison is possible with the ordered hardware | Extra USB bandwidth and weight on the handheld while the D405 is mounted | Superseded 2026-11-03: D405 not mounted in the core design |
| RGB only; D405 not mounted; RGB vs RGB + depth tested later if time allows | Matches GR00T and π0.5 as they are; no extra camera, bracket, USB bandwidth or weight on the handheld (supervisor's simplicity aim) | No depth recorded, so a later comparison needs new data | **Recommended** |
| Depth as a policy input from day one ([D405](https://eu.robotshop.com/products/mybotshop-intel-realsense-depth-camera-d405-camera-only)) | Global shutter; 7 cm minimum range; strong community | Requires modifying GR00T or π0.5; mixed evidence (above); narrow 87° × 58° view | Not in the core design |

*Also considered:* the [Orbbec Gemini 305](https://store.orbbec.com/products/gemini-305) (4 cm minimum range, new in 2026, no Dex1-1 bracket); not needed, since D405s are ordered.

## Decision

RGB only. Depth is not a policy input, and the ordered D405 cameras are not mounted in the core design. An RGB vs RGB + depth test with the D405 is optional and only done if time remains; it is not a priority.

## Evidence

GR00T and π0.5 take RGB images, and naively stacked RGB-D did not reliably help in [Point Cloud Matters](https://arxiv.org/html/2402.02500v3); leaving the D405 off removes hardware from the handheld, in line with the supervisor's aim of keeping the handheld as simple as possible (supervisor meeting, 2026-11-03, `Daily/2026-11-03`).

## Revisit if

Time remains after the core experiments, or the supervisor requests the depth comparison.

## History

- 2026-10-19: accepted (proposal approved)
- 2026-11-03: RGB only; D405 no longer mounted or recorded in the core design, depth test optional if time allows (supervisor meeting: depth not a priority, keep the handheld simple)
