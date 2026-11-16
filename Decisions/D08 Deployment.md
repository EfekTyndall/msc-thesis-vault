---
date: 2026-10-19
status: accepted
tags: [decision]
---
## Context

On which robot the handheld data is validated first. Source: `msc-thesis/docs/proposal.md` §8 (approved by Supervisor on 19 Oct 2026).

## Options

| Option | Strengths | Weaknesses | Verdict |
| --- | --- | --- | --- |
| UR10e first, then H2 | Pipeline debugged on a simpler robot; lower risk | Two integrations | **Recommended** |
| H2 directly | One integration | Data and control problems mixed together | **Fallback** if the UR10e is not available |

## Decision

Validation on the UR10e first, then the H2, with demonstrations the robot cannot execute filtered out.

## Evidence

Confirmed by the supervisor on 2026-11-16 ([[2026-11-16 Yanmei]]): UR10e first, then bimanual manipulation on the H2.

A fixed-base arm separates data-quality problems from humanoid-control problems, and [HuMI](https://arxiv.org/html/2602.06643v2) shows that humanoid deployment needs extra care with reach and posture.

## Revisit if

–

## History

- 2026-10-19: accepted (proposal approved)
- 2026-11-16: confirmed by the supervisor: UR10e first, then bimanual manipulation on the H2 (2026-11-16 talk, [[2026-11-16 Yanmei]]; reported by Martyn). `msc-thesis/docs/thesis-plan.md` §2 still lists bimanual and H2 deployment as stretch goals; not changed
