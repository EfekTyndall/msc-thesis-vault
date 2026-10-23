---
date: 2026-10-19
status: accepted
tags: [decision]
---
## Context
Whether depth is used as a policy input, given the D405 that was ordered with the grippers. Source: `msc-thesis/docs/proposal.md` §3 (approved by Yanmei on 19 Oct 2026).

## Options
| Option | Strengths | Weaknesses | Verdict |
| --- | --- | --- | --- |
| RGB as policy input; D405 mounted and recorded for a later comparison | Matches GR00T and π0.5 as they are; the comparison is possible with the ordered hardware | Extra USB bandwidth and weight on the handheld while the D405 is mounted | **Recommended** |
| Depth as a policy input from day one ([D405](https://eu.robotshop.com/products/mybotshop-intel-realsense-depth-camera-d405-camera-only)) | Global shutter; 7 cm minimum range; strong community | Requires modifying GR00T or π0.5; mixed evidence (above); narrow 87° × 58° view | Not in the core design |

*Also considered:* the [Orbbec Gemini 305](https://store.orbbec.com/products/gemini-305) (4 cm minimum range, new in 2026, no Dex1-1 bracket); not needed, since D405s are ordered.

## Decision
Depth is not a policy input in the core design. The D405 cameras that have been ordered are mounted on the Dex1-1 brackets and recorded for an optional RGB vs RGB + depth comparison.

## Evidence
GR00T and π0.5 take RGB images, and naively stacked RGB-D did not reliably help in [Point Cloud Matters](https://arxiv.org/html/2402.02500v3); recording the D405 keeps the comparison possible without changing the policies.

## Revisit if
Supervisor requests the depth comparison.

## History
- 2026-10-19: accepted (proposal approved)
