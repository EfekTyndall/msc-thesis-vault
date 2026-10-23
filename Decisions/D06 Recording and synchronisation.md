---
date: 2026-10-19
status: accepted
tags: [decision]
---
## Context
Where data is recorded and how all streams share one time base. Source: `msc-thesis/docs/proposal.md` §6 (approved by Yanmei on 19 Oct 2026).

## Options
| Option | Strengths | Weaknesses | Verdict |
| --- | --- | --- | --- |
| Tethered PC, one clock, measured latency | Light handheld; one clock; unlimited recording headroom; 10 ms alignment target ([ViTaMIn-B](https://arxiv.org/html/2511.05858v1)) | Cable drag and tangling; connector wear | **Recommended** for a fixed workstation |
| Backpack PC with battery ([ActiveUMI](https://arxiv.org/pdf/2510.01607)) | Same software; untethered from the station | Battery handling; worn weight | **Fallback** if collection spans several stations or cables are not allowed |
| Raspberry Pi 5 or Jetson per handheld ([Grabette](https://huggingface.co/blog/grabette), [TacUMI](https://arxiv.org/pdf/2601.20239)-style board) | Fully self-contained handheld | No hardware video encoder; extra weight and a battery on each handheld; three clocks to synchronise; Wi-Fi risk on a factory floor | Last resort |
| Hardware trigger ([HiFi-UMI](https://arxiv.org/pdf/2607.25895): < 40 µs; [UMI-3D](https://arxiv.org/pdf/2604.14089): STM32 clock) | Most precise | Needs a triggerable camera; mainly for fast, dynamic tasks | Not needed for pick-and-place |



## Decision
No onboard computer as the first option. Each handheld connects by one cable to a single PC, which is the only clock; every sensor's latency is measured and compensated to a residual of about 10 ms or less. Whether the industrial use case is one fixed station or a moving operator is still open, so an onboard or backpack computer remains the fallback.

## Evidence
[ViTaMIn-B](https://arxiv.org/html/2511.05858v1) removed its onboard Raspberry Pi in favour of a host PC for "improved ergonomics and reduced weight", one PC gives one clock, and latency compensation raised ViTaMIn-B's final-step success from 20% to 70%.

## Revisit if
Collection must span several stations, or cables are not allowed.

## History
- 2026-10-19: accepted (proposal approved)
