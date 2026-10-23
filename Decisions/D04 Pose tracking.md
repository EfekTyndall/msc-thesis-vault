---
date: 2026-10-19
status: accepted
tags: [decision]
---
## Context
How the 6-DoF pose of each handheld is measured during collection. Source: `msc-thesis/docs/proposal.md` §4 (approved by Yanmei on 19 Oct 2026).

## Options
| Option | Strengths | Weaknesses | Verdict |
| --- | --- | --- | --- |
| PICO 4 Ultra controllers via [XRoboToolkit](https://arxiv.org/html/2508.00097v2) | Already available; same headset as the teleop stack; 90 Hz with timestamps; ringless; used in [GR00T-WholeBodyControl](https://github.com/NVlabs/GR00T-WholeBodyControl) | Accuracy unmeasured; LED placement and timestamp meaning undocumented; headset must be present | **Recommended**, after measurement on the UR10e |
| Meta Quest 3 / 3S controllers, used by [ActiveUMI](https://arxiv.org/pdf/2510.01607) and [ViTaMIn-B](https://arxiv.org/html/2511.05858v1) | The setup behind the only published head-to-head win over SLAM; ringless | Extra purchase; accuracy unmeasured | **Alternative** if the PICO is an older ring-controller model |
| PICO Motion Trackers (object mode) | 27 g; €89 per pair; streamed by XRoboToolkit on the PICO 4 Ultra ([Road to VR](https://roadtovr.com/pico-4-ultra-quest-3s-release-europe-asia/)) | PICO claims only < 5 cm and < 6° ([Skarredghost](https://skarredghost.com/2024/08/20/pico-4-ultra-launch-hands-on/)); use of two trackers in object mode unverified | Test only |
| Vive Tracker 3.0 + Lighthouse base stations, used by [TacUMI](https://arxiv.org/pdf/2601.14550) and [RDT2](https://arxiv.org/pdf/2602.03310) | Sub-mm, sub-degree error against a UR10 ([Sensors 2023](https://research.uni-luebeck.de/en/publications/static-and-dynamic-accuracy-and-occlusion-robustness-of-steamvr-t/)); no headset | Stations tie collection to where they are mounted; about €776–1,254 ([HTC shop](https://myshop.vive.com/vive_de/accessories.html?p=1)) | **Secondary**: accuracy reference and fallback |
| [Vive Ultimate Tracker](https://shop-ca.vive.com/products/21048155), used by [HuMI](https://arxiv.org/html/2602.06643v2) | No base stations | 10.6 mm in low light vs 2.6 mm in bright light ([arXiv 2409.01947](https://arxiv.org/html/2409.01947v1)) | **Fallback** |

*Also considered:* optical motion capture such as [OptiTrack](https://www.optitrack.com/cameras) (sub-0.3 mm, but a fixed truss); only useful for a one-off validation.

## Decision
The VR controllers of the PICO headset (PICO 4 Ultra expected; model to be confirmed), one rigidly fixed to each handheld, streamed to the recording PC with PICO's open-source XRoboToolkit, and measured against the UR10e before data collection. A Vive Tracker 3.0 + Lighthouse setup remains the accuracy reference and fallback.

## Evidence
In [ViTaMIn-B](https://arxiv.org/html/2511.05858v1), VR controller tracking gave 100% valid demonstrations and 70% policy success, against 16% and 10% with onboard SLAM; the PICO headset is already available, and [XRoboToolkit](https://arxiv.org/html/2508.00097v2) streams its controller poses at 90 Hz with timestamps.

## Revisit if
Measured error or dropouts too high → Vive Tracker 3.0 + base stations (ordered only then).

## History
- 2026-10-19: accepted (proposal approved)
