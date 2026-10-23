---
date: 2026-10-19
status: accepted
tags: [decision]
---
## Context
How handheld episodes are recorded and exported so they match the VR teleop data. Source: `msc-thesis/docs/proposal.md` §7 (approved by Yanmei on 19 Oct 2026).

## Options
| Option | Strengths | Weaknesses | Verdict |
| --- | --- | --- | --- |
| New recorder writing the teleop format | Fits the chosen sensors; one format for both data sources | Built from scratch | **Recommended** (format to be confirmed with the teleop stack) |
| Adapting the [Grabette](https://huggingface.co/blog/grabette) open-source stack | Recorder, processing and LeRobot export already exist | Built around its own cameras and SLAM | **Fallback** / reference implementation |
| Porting the [UMI pipeline](https://github.com/real-stanford/universal_manipulation_interface) | Reference for latency handling and data filtering | Tied to GoPro and SLAM; most stages unused | Not advised |



## Decision
A new recorder that writes episodes directly in the same format as the VR teleop pipeline, using the action representation of [UMI](https://arxiv.org/pdf/2402.10329).

## Evidence
One format for handheld and teleop data is what makes the comparison possible, and the [UMI code](https://github.com/real-stanford/universal_manipulation_interface) is tied to the GoPro and SLAM, which this design does not use.

## Revisit if
–

## History
- 2026-10-19: accepted (proposal approved)
