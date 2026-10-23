---
date: 2026-10-19
status: accepted
tags: [decision]
---
## Context
Which camera the policy sees on both robot and handheld. Source: `msc-thesis/docs/proposal.md` §2 (approved by Yanmei on 19 Oct 2026).

## Options
| Option | Strengths | Weaknesses | Verdict |
| --- | --- | --- | --- |
| Dex1-1 RGB camera | Wide view; supplied with the gripper; mount defined by Unitree | Model undocumented; specifications disagree (118° H / 140° D vs 160°); interface not yet confirmed; must be moved between robot and handheld | **Recommended** |
| Wide-angle USB fisheye on both sides (e.g. the industrial camera with \~185° lens in [UMI-3D](https://arxiv.org/pdf/2604.14089)) | Documented; wide; low latency; full control | A new camera mount must also be designed for the robot gripper | **Fallback** if the Dex1-1 camera cannot be sourced or used |
| D405 as the only camera | Identical units purchasable; global shutter; ROS 2 support | Narrow view; its depth goes unused by GR00T and π0.5 | Not recommended as the sole camera |

*Also considered:* the [Arducam AR0234](https://eu.robotshop.com/products/arducam-23mp-ar0234-color-global-shutter-usb-30-camera-module-w-encolsure) global-shutter module (98° diagonal as shipped, accepts 120–180° M12 lenses) as a concrete fallback camera.

## Decision
The Dex1-1's own RGB camera and removable mount, supplied with each gripper and moved between robot and handheld, without mirrors.

## Evidence
A wide view has the strongest evidence (cropping [UMI](https://arxiv.org/pdf/2402.10329)'s view to 69° dropped success from 20/20 to 11/20), and the same camera on robot and handheld avoids the lens mismatch that dropped a π0.5 policy to 0.0025 in the [2026 fisheye study](https://arxiv.org/pdf/2603.02139v1).

## Revisit if
Camera interface unusable, or remounting proves unrepeatable.

## History
- 2026-10-19: accepted (proposal approved)
