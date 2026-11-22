---
date: 2026-10-19
status: accepted
tags: [decision]
---
## Context

How the 6-DoF pose of each handheld is measured during collection. Source: `msc-thesis/docs/proposal.md` §4 (approved by Supervisor on 19 Oct 2026).

## Options

| Option | Strengths | Weaknesses | Verdict |
| --- | --- | --- | --- |
| PICO 4 Ultra controllers via [XRoboToolkit](https://arxiv.org/html/2508.00097v2) | Already available; same headset as the teleop stack; 90 Hz with timestamps; ringless; used in [GR00T-WholeBodyControl](https://github.com/NVlabs/GR00T-WholeBodyControl) | Accuracy unmeasured; LED placement and timestamp meaning undocumented; headset must be present | **Recommended**, after measurement on the UR10e |
| Meta Quest 3 / 3S controllers, used by [ActiveUMI](https://arxiv.org/pdf/2510.01607) and [ViTaMIn-B](https://arxiv.org/html/2511.05858v1) | The setup behind the only published head-to-head win over SLAM; ringless | Extra purchase; accuracy unmeasured | Dropped 2026-11-03: only the PICO 4 Ultra Enterprise is available in the lab |
| PICO Motion Trackers (object mode) | 27 g; €89 per pair; streamed by XRoboToolkit on the PICO 4 Ultra ([Road to VR](https://roadtovr.com/pico-4-ultra-quest-3s-release-europe-asia/)) | PICO claims only < 5 cm and < 6° ([Skarredghost](https://skarredghost.com/2024/08/20/pico-4-ultra-launch-hands-on/)); use of two trackers in object mode unverified | Test only |
| Vive Tracker 3.0 + Lighthouse base stations, used by [TacUMI](https://arxiv.org/pdf/2601.14550) and [RDT2](https://arxiv.org/pdf/2602.03310) | Sub-mm, sub-degree error against a UR10 ([Sensors 2023](https://research.uni-luebeck.de/en/publications/static-and-dynamic-accuracy-and-occlusion-robustness-of-steamvr-t/)); no headset | Stations tie collection to where they are mounted; about €776–1,254 ([HTC shop](https://myshop.vive.com/vive_de/accessories.html?p=1)) | **Secondary**: accuracy reference and fallback; mount on the handheld not yet designed (not a priority) |
| [Vive Ultimate Tracker](https://shop-ca.vive.com/products/21048155), used by [HuMI](https://arxiv.org/html/2602.06643v2) | No base stations | 10.6 mm in low light vs 2.6 mm in bright light ([arXiv 2409.01947](https://arxiv.org/html/2409.01947v1)) | Set aside (accuracy drops in low light); not a fallback |

*Also considered:* optical motion capture such as [OptiTrack](https://www.optitrack.com/cameras) (sub-0.3 mm, but a fixed truss); only useful for a one-off validation.

## Decision

The VR controllers of the PICO headset (PICO 4 Ultra Enterprise, 12 GB; confirmed 2026-11-02), one rigidly fixed to each handheld, streamed to the recording PC with PICO's open-source XRoboToolkit, and measured against the UR10e before data collection. Meta Quest controllers are no longer an option, since only the PICO is available in the lab. A Vive Tracker 3.0 + Lighthouse setup remains the accuracy reference and fallback if the PICO controllers prove unreliable; the other trackers stay listed as alternatives, since they add hardware.

Headset placement (2026-11-17): the controller tracking is tested with the headset in two placements, worn on the head and hung around the operator's neck with the proximity sensor taped over; the UR10e accuracy and dropout measurement covers both, and the placement used for collection follows from the results.

## Evidence

In [ViTaMIn-B](https://arxiv.org/html/2511.05858v1), VR controller tracking gave 100% valid demonstrations and 70% policy success, against 16% and 10% with onboard SLAM; the PICO headset is already available, and [XRoboToolkit](https://arxiv.org/html/2508.00097v2) streams its controller poses at 90 Hz with timestamps.

Neck-hung placement: no precedent found in the sources checked (none of the UMI variants above describes it). Feasibility measured 2026-11-17: off the head the headset sleeps via its proximity sensor ([PICO 4 Ultra user guide, p. 8](https://p16-platform-static-va.ibyteimg.com/tos-maliva-i-jo6vmmv194-us/pico4-ultra-user-guide-eu.pdf)); with tape over the sensor and the headset on the neck, a 120 s XRoboToolkit SDK log at 30 Hz on the station Jetson had 0.4 % stalled samples (longest 0.27 s), no zero poses and one 0.077 m step between samples (fast move or tracking jump, unknown). Accuracy not measured. Details: [[2026-11-17 PICO control from the Mac and neck-hung headset]].

## Revisit if

Measured error or dropouts too high → Vive Tracker 3.0 + base stations (ordered only then). Neck-hung placement measurably worse than worn → worn only.

## History

- 2026-10-19: accepted (proposal approved)
- 2026-11-02: headset model confirmed as PICO 4 Ultra Enterprise, 12 GB (Martyn). Enterprise and consumer controllers reported as the same hardware by secondary sources ([ArborXR](https://arborxr.com/blog/pico-4-ultra/), [VR/AR Wiki](https://vrarwiki.com/wiki/Pico_4_Ultra_Enterprise)); no official confirmation found (unverified)
- 2026-11-03: Meta Quest dropped (only the PICO 4 Ultra Enterprise is in the lab); other trackers kept as fallbacks only, Vive mount on the handheld not a priority (supervisor meeting; they add hardware)
- 2026-11-17: headset placement added: tracking tested both worn and neck-hung (proximity sensor taped) (Martyn; neck-hung 2 min SDK test kept tracking; measure the difference instead of assuming one placement)
- 2026-11-22: Vive Ultimate Tracker verdict in the option table corrected from "Fallback" to set aside, matching the Decision text (fallback is only Vive Tracker 3.0 + Lighthouse) and the October decision-log row; decision unchanged (weekly consistency audit 2026-11-22, Martyn's board choice)
