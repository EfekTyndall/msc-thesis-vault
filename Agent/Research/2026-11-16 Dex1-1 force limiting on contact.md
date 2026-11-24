---
date: 2026-11-16
tags: [research, dex1-1, gripper, safety, gaps]
---
# Dex1-1 force limiting on contact (gap G9)

Question (colleague at [[2026-11-16 Yanmei]], gap G9 in [[2026-11-10 Research gaps from the PICO sim test]]): when the part resists and the fingers cannot close, what limits the force, and how is force-closing avoided?

Method: source read of [`dex1_1_service` @ ced3216](https://github.com/unitreerobotics/dex1_1_service/tree/ced3216290bb22dae356213993786beeaa7143ab), [`unitree_sdk2` @ aaf69cb](https://github.com/unitreerobotics/unitree_sdk2/tree/aaf69cb) IDL headers, the Dex1-1 controller in [`xr_teleoperate` @ 817fb00](https://github.com/unitreerobotics/xr_teleoperate/tree/817fb00c63cde15e5f24a0f8fa08e1e33ed89d3b), the [Unitree product page](https://www.unitree.com/Dex1-1), the URDFs in `dex1-1-handheld/hardware/dex1-1-reference/`, the UMI deployment code [`universal_manipulation_interface` @ d095ba9](https://github.com/real-stanford/universal_manipulation_interface/tree/d095ba9590df789df5189eea5ee7e431689038a6), and the bib papers chi2024, choi2026, surendran2026, ai2026, zhaxizhuoma2025, cheng2026 (arXiv HTML). The Unitree developer docs ([support page](https://support.unitree.com/home/en/dex1-1_gripper/about_dex1_1)) did not render outside a browser (empty JavaScript shell), so they were **not read**. Nothing was tested on hardware.

## Answer

1. **Nothing on the open software side limits force.** `dex1_1_service` passes `q`, `dq`, `tau`, `kp`, `kd` straight to the motor, only scaled by the gear ratio (`main.cpp` l. 71–77). It has no clamp, no stall check and no torque limit. If a command says "close to 0" while a part blocks the fingers, the motor pushes with `kp · (q − q_target) + tau` until the motor's own (closed-source) limit.
2. **The force is set by the client through the PD law.** Unitree states the force range "5~120N" is reached "by adjusting motor stiffness, damping, and torque" ([product page](https://www.unitree.com/Dex1-1), footnote 2). So force limiting is the job of the program that sends commands (ours, on the station Jetson).
3. **Unitree's own teleop already limits it, implicitly.** `xr_teleoperate` clips each target to the measured position ± 0.18 rad (`DELTA_GRIPPER_CMD`, `robot_hand_unitree.py` l. 326, 380–382; same in `robot_arm.py` l. 357, 545) with `kp = 5.0`, `kd = 0.05`, `tau = 0`. When the fingers are blocked, the position error can never exceed 0.18 rad, so the steady squeeze torque is at most 5.0 × 0.18 = **0.9 N·m at the output shaft**. The code comment calls this "to avoid overflow"; it does not call it a force limit (the force-limit reading is ours).
4. **Rough force for that limit, unverified:** the same file states "The motor rotates 5.4 radians, the clamping jaw slide open 9 cm", i.e. about 16.7 mm of opening per rad. Ideal (lossless) transmission gives 0.9 N·m / 0.0167 m ≈ **54 N** squeeze, an upper bound since friction is ignored. The stroke itself is uncertain (90 mm product page vs about 82–83 mm from STEP/URDF), so measure it.
5. **No firmware protection is documented in what was read.** The product page lists "Motor Peak Torque 5N.m", "Peak Line Current 3.5A", "Motor Reduction Ratio 1:25" and feedback "Angle, Angular Velocity, Torque, Temperature", but no overcurrent, stall or over-temperature behaviour. Whether 5 N·m is at the rotor or the output is not stated. The motor library is closed source.

## What the cmd/state messages offer

| Field | Where | Use for force limiting |
| --- | --- | --- |
| `kp`, `kd`, `q`, `dq`, `tau` (cmd) | `MotorCmd_.hpp`; service l. 71–77 | Only knobs. No torque-limit or current-limit field exists in the DDS command, nor in the serial packet `ControlData_hg` (`tor_des`, `spd_des`, `pos_des`, `k_pos`, `k_spd`) |
| `mode` (cmd) | `MotorCmd_.hpp` | **Ignored** by the service (it sets FOC itself) |
| `q`, `dq`, `tau_est` (state) | service l. 83–85 | Published. `tau_est` = estimated torque, the only force signal (no F/T sensor) |
| `temperature`, `lost` (state) | `MotorState_.hpp` | Fields exist but the service **does not fill them** (stay 0) |
| `temp1`/`temp2` (case/winding °C), `MError` error bits, `timeout` | serial reply `MotorData_hg` (`motor_msg_HG.h`) | Motor reports them, but the service drops them. `MError` bit table ("表1") is not in the repo. Whether the M4010 uses this packet is unverified (lib closed) |
| Command timeout | service l. 65–68 | No DDS command for a while → BRAKE with all gains 0. Whether BRAKE keeps holding a part or lets go is unverified |

URDF effort limits are no help: the official URDF 261030 has `effort="0"`, the unitree_ros URDF `effort="20"` (a simulation value, source unstated).

## How prior handheld/UMI work handles grasp force

| Work | Robot gripper control | Force limit | Source |
| --- | --- | --- | --- |
| UMI (chi2024) | Policy outputs gripper width; on the WSG-50 a position PD script runs on the gripper | **Yes, firmware force limit:** `travel_force_limit=80.0` N, `blocked_force_limit` (defaults to the same) applied when `mc.blocked()` | `umi/real_world/wsg_binary_driver.py` l. 308–317, `cmd_measure.lua` l. 44–57. Paper: "UMI can implicitly record and control grasp forces by regulating the deformation of soft fingers" (§HD5) |
| UMI-FT (choi2026) | Policy outputs "desired gripper width and grasp force"; grasp admittance law on a WSG50 | Grasp force measured by CoinFT on each finger; safety thresholds "25 N in the grasp direction" (§V) | arXiv 2601.09988 §IV-A, §IV-B, §V |
| DM-UMI (surendran2026) | "absolute gripper width" command | Nothing found | arXiv 2606.26603 §III–IV |
| HiFi-UMI (ai2026), FastUMI (zhaxizhuoma2025), TacUMI (cheng2026) | Width / opening commands | Nothing found on the robot side | arXiv HTML read |

Reading: the UMI line commands **width only** and relies on the gripper (WSG-50 firmware force limit, soft fingers) to bound the squeeze. Only UMI-FT makes grasp force an explicit, measured action. No bib paper uses the Dex1-1.

## Options for our gripper bridge (all client-side, Unitree code unchanged)

| Option | How | Precedent |
| --- | --- | --- |
| A. Clip the position error | `q_target` limited to `q ± Δ`; squeeze ≤ `kp · Δ` | `xr_teleoperate` (Δ = 0.18 rad, kp = 5) |
| B. Explicit torque cap | Choose `Δ = τ_max / kp` from a measured N per N·m, so the limit is stated in newtons | Same law as A; stating it in N follows the WSG `travel_force_limit` (UMI) |
| C. Lower limit when blocked | Detect blocked (`|dq|` ≈ 0 and error at the clip) and drop to a lower hold force | UMI `blocked_force_limit` in `cmd_measure.lua` |
| D. Torque-mode grasp | `kp = 0`, small `kd`, `tau` = wanted closing torque | Product page lists torque, stiffness and damping control; no code precedent found |
| E. Force as data | Log `tau_est` (calibrated to N) next to width | UMI-FT records grasp force; no Dex1-1 precedent |

A is the default if the teleop code is reused; B makes the limit explicit and defensible. C and E are optional. This is a list, not a decision. The choice also touches G2 (mapping): with A/B the commanded width and the measured width differ once the part is gripped, so the dataset must say which one it stores.

## Open (needs the real gripper)

Clean-up 2026-11-24 ([[2026-11-24 Open items clean-up]]): items 1–6 and the choice of limit are tracked as one G9 item in [[Waiting]], Dex1-1 delivery.

1. Force gauge between the fingers: squeeze force vs `kp · Δ` and vs `tau`, to get N per N·m and check the 54 N estimate and the 5–120 N range.
2. `tau_est` vs gauge: is it usable as a force estimate?
3. Stall test: close on a rigid block (e.g. the metal sleeve) at the chosen limit for 60 s; watch for heating or cut-out. Temperature and `MError` need a small change to read (fields are dropped by the service), so first just touch-check the motor case (unverified safe duration).
4. BRAKE behaviour while holding a part (stop sending commands).
5. Read the Unitree developer docs in a browser for protection and error codes; ask Unitree (issue tracker) if not covered.
6. Wider web search (other Dex1-1 users, Unitree forum) only after Martyn's go.
