# ARM5R-RL — Project Requirements

| Field | Value |
|---|---|
| Document ID | P0-REQ-001 |
| Version | 0.1 — Draft |
| Last updated | 2026-11-15 |
| Project | ARM5R-RL |
| Phase | Phase 0 — Project Definition |
| Status | Draft for review |
| Related documents | [ROBOT_HARDWARE_CHARACTERIZATION.md](ROBOT_HARDWARE_CHARACTERIZATION.md), [MECHANICAL_MEASUREMENTS.md](MECHANICAL_MEASUREMENTS.md) |

## 1. Project Overview

ARM5R-RL is a university capstone project to design, implement, and evaluate a reinforcement-learning-based control system for a physical five-degree-of-freedom serial robotic arm with a two-finger gripper.

The required final demonstration is a real pick-and-place task between predefined pickup and placement locations. The initial demonstration will use fixed object and target locations. Camera-based perception is an extension and is not required for the first successful demonstration.

The project will investigate both joint-space and Cartesian-space reinforcement-learning control. To manage the one-month delivery constraint, joint-space control is the first implementation target. Cartesian-space control follows only after the robot can be operated reliably and the first physical pick-and-place task is working.

## 2. Project Goals

### 2.1 Primary Goal

Develop and deploy a reinforcement-learning-based control system that can perform a physical pick-and-place task with the ARM5R robot.

### 2.2 Technical Goals

1. Establish a reliable software interface to the seven DYNAMIXEL actuators through the ROBOTIS CM-550 controller.
2. Create a consistent robot description, coordinate-frame model, and kinematic model.
3. Build a physics-based simulation and an RL-compatible environment.
4. Train and evaluate a joint-space RL policy.
5. Transfer the trained policy to the physical robot with safety limits and controlled testing.
6. Implement a physical pick-and-place demonstration using fixed pickup and placement locations.
7. Investigate Cartesian-space RL control after the joint-space system is operational.
8. Keep the software modular, testable, documented, and reproducible.

### 2.3 Research Scope

The selected research focus is the design, implementation, and evaluation of an RL-based robotic control system. A comparison against a classical controller is not a required research objective in the current scope.

The project must still report quantitative results for its own system, including task success, position error, execution time, grasp outcome, and failure cases. Numerical target thresholds will be defined after the initial hardware and task characterization.

## 3. Scope and Priorities

### 3.1 Required for the One-Month Deliverable

- Physical ARM5R operation through the CM-550.
- Verified mapping between physical motor IDs and logical joints.
- Safe joint-level command and feedback handling.
- A robot kinematic model consistent with the measured reference pose.
- A simulation environment suitable for RL training.
- A trained joint-space RL policy.
- A controlled transfer and test procedure for the real robot.
- At least one successful physical pick-and-place demonstration between predefined locations.
- Experiment logs and a documented evaluation.

### 3.2 Planned After the Core Demonstration

- Cartesian-space RL policy or control interface.
- Camera-based object detection and localization.
- Pick-and-place with variable object positions.
- Broader object sets and more complex manipulation tasks.

### 3.3 Out of Scope for the Initial Demonstration

- General-purpose manipulation of arbitrary objects.
- Unrestricted autonomous operation in an unstructured environment.
- Camera-based localization as a dependency for the first pick-and-place demonstration.
- Claims of robust performance outside the tested workspace, objects, payloads, and operating conditions.

## 4. Robot and Hardware Baseline

The physical platform is reported as a Pishrobot Arm6x kit configured as a five-revolute-joint manipulator with a two-finger gripper.

| Property | Current information | Status |
|---|---|---|
| Logical arm joints | 5 revolute joints: J1–J5 | Owner-confirmed |
| Gripper | Two-finger, gear-driven, one motor | Owner-reported |
| Physical actuators | 7 DYNAMIXEL motors in the controller scan | Read from robot |
| Motor model | XL430-W250 for IDs 1–7 | Read from robot |
| Shoulder actuation | Two motors drive logical J2 | Owner-reported |
| Controller | ROBOTIS CM-550 | Read from robot |
| Host connection | USB serial, reported device `/dev/ttyACM0` | Read from robot |
| Supply voltage | Approximately 11–12 V; snapshot values 11.4–11.7 V | Read/reported |
| CAD model | Not yet available; mechanical reconstruction is required | Owner-confirmed |

The seven physical actuator IDs have been discovered, but the mapping of IDs 1–7 to J1–J5 and the gripper has not yet been verified. Do not infer this mapping from ID order.

### 4.1 Controller and Bus Snapshot

The following values were extracted from the robot on `2026-09-24T15:25:15+03:30`. They describe that diagnostic session and must be rechecked before deployment.

| Parameter | Extracted value |
|---|---|
| Controller type | CM-550 |
| Controller device ID | 200 |
| Serial device at capture | `/dev/ttyACM0` |
| Baud rate | 57600 |
| DYNAMIXEL protocol | 2.0 |
| DYNAMIXEL power-state field | 1 |
| Controller mode field | 2 (meaning not yet interpreted) |
| Bypass port | 2 |
| Bus scan count | 7 |
| Broadcast discovery count | 7 |
| Expected motor count | 7 |
| Count matches expected | Yes |

The serial device path may change between connections or boots. The meanings of the controller `mode` and `dxl_power_state` fields must be confirmed from the relevant CM-550 documentation or the existing control software before they are used as safety assumptions.

### 4.2 Per-Motor Diagnostic Snapshot

All seven discovered motors reported model number `1060`, model name `XL430-W250`, and firmware version `47`.

Shared configuration reported for every motor:

- Drive mode: `0`
- Operating mode: `3` (DYNAMIXEL Position Control Mode)
- Configured position limits: raw minimum `0`, raw maximum `4095`
- Torque enabled: `0` for all motors
- Hardware error: `0` for all motors

| DYNAMIXEL ID | Present position (raw) | Reported position (deg) | Goal position (raw) | Voltage (V) | Temperature (°C) | Hardware error |
|---:|---:|---:|---:|---:|---:|---:|
| 1 | 1995 | 175.3418 | 1995 | 11.7 | 35 | 0 |
| 2 | 3054 | 268.4180 | 3054 | 11.7 | 35 | 0 |
| 3 | 1050 | 92.2852 | 1050 | 11.5 | 35 | 0 |
| 4 | 2918 | 256.4648 | 2918 | 11.5 | 35 | 0 |
| 5 | 2090 | 183.6914 | 2090 | 11.5 | 35 | 0 |
| 6 | 2845 | 250.0488 | 2845 | 11.5 | 35 | 0 |
| 7 | 2036 | 178.8574 | 2036 | 11.4 | 36 | 0 |

For ID 7, the extracted raw `position_raw` field was `2035`, while `present_position` and `goal_position` were both `2036`. This one-tick discrepancy must be retained in the diagnostic record and checked in a fresh read; it must not be silently normalized.

These are static diagnostic readings, not verified joint-zero offsets, mechanical limits, or calibrated joint angles. The reported degree values must be checked against the conversion and calibration used by the control software. The fact that torque was disabled during this snapshot is not evidence that the robot is safe to move.

## 5. Coordinate Frames and Kinematic Baseline

The agreed base frame is right-handed:

- `+Z_B`: vertically upward and aligned with the J1 rotation axis.
- `+X_B`: forward direction in the reference pose.
- `+Y_B`: completes the right-handed frame.

The reference pose `P0` has J2 and J3 vertically aligned, with J4, J5, and the TCP extending horizontally in `+X_B`.

At `P0`, the positive joint-axis directions expressed in the base frame are:

| Joint | Positive rotation axis |
|---|---|
| J1 | `+Z_B` |
| J2 | `+Y_B` |
| J3 | `+Y_B` |
| J4 | `+Y_B` |
| J5 | `+X_B` |

The current reference-pose landmark coordinates are:

| Landmark | X (mm) | Y (mm) | Z (mm) |
|---|---:|---:|---:|
| Base origin | 0 | 0 | 0 |
| J1 origin | 0 | 0 | 80 |
| J2 origin | 0 | 0 | 116 |
| J3 origin | 0 | 0 | 253 |
| J4 origin | 65 | 0 | 253 |
| J5 origin | 135 | 0 | 253 |
| TCP | 215 | 0 | 253 |

The corresponding fixed frame transforms at `P0` are documented in `MECHANICAL_MEASUREMENTS.md`. These values form the current geometry baseline, but physical measurement evidence and kinematic validation are still required before they are treated as a validated robot model.

## 6. Software and Compute Baseline

| Component | Current status |
|---|---|
| Operating system | Ubuntu 24.04.5 LTS |
| Python | 3.12.3 |
| CPU | Intel Core i5-11400H, 6 cores / 12 threads |
| RAM | Approximately 15 GiB |
| GPU | NVIDIA GeForce RTX 3050 |
| NVIDIA driver | Unresolved: previous `nvidia-smi` could not communicate with the driver |
| Git | Installed; local ARM5R-RL repository initialized |
| Existing hardware software | Python and DYNAMIXEL SDK used previously in ARMPy |
| ROS 2 Jazzy | Candidate; compatibility and installation to be verified |
| MuJoCo | Planned |
| Gymnasium | Planned |
| PyTorch | Planned |
| Stable-Baselines3 | Planned |
| TensorBoard | Planned |

The software stack is a proposal, not a statement that these packages are already installed or verified. Package and Python-version compatibility must be checked before creating the project environment. CPU-based development and small experiments should remain possible if GPU acceleration is unavailable.

Existing control code repository: [ARMPy](https://github.com/amirarsalan7/Armpy). The ARM5R-RL project must preserve a clear separation between low-level hardware communication, robot kinematics, simulation, RL training, and real-robot deployment.

## 7. Functional Requirements

| ID | Requirement | Priority | Acceptance evidence |
|---|---|---|---|
| FR-001 | The system shall communicate with all seven discovered DYNAMIXEL actuators through the CM-550. | Must | Repeatable discovery and read test; verified ID-to-joint map |
| FR-002 | The system shall expose five logical arm joint states and a separate gripper state. | Must | Correct state mapping documented and tested |
| FR-003 | The system shall account for the opposing motor directions used by the dual-motor shoulder. | Must | Verified J2 command/feedback sign and synchronization test |
| FR-004 | The system shall support bounded joint-space commands and feedback. | Must | Command and feedback test within verified safe limits |
| FR-005 | The system shall provide a kinematic model consistent with the documented coordinate convention. | Must | Forward-kinematics checks against measured reference points |
| FR-006 | The system shall provide an RL-compatible simulation environment. | Must | Environment reset, observation, action, reward, and termination tests |
| FR-007 | The initial RL policy shall operate in joint space. | Must | Trained policy evaluated in simulation |
| FR-008 | The system shall support a controlled deployment path from simulation to the physical robot. | Must | Staged deployment test with safety checks |
| FR-009 | The physical robot shall complete a pick-and-place task between predefined locations. | Must | Recorded physical trial with pickup and placement outcome |
| FR-010 | The system should support Cartesian-space RL control after the joint-space milestone. | Should | Cartesian action interface and evaluation |
| FR-011 | The system may integrate camera-based object localization as an extension. | Could | Camera-to-robot calibration and localization test |

## 8. Safety Requirements

Real-robot RL deployment is a high-risk stage because a learned policy can produce unexpected actions. Simulation success is not sufficient evidence that direct physical execution is safe.

| ID | Safety requirement | Acceptance evidence |
|---|---|---|
| SAF-001 | Physical motor IDs and their mechanical functions shall be verified before motion commands are enabled. | Reviewed actuator map |
| SAF-002 | Mechanical joint limits shall be measured or conservatively established before automated motion. | Per-joint limit table and verification record |
| SAF-003 | The two shoulder motors shall be tested at low speed with verified direction, offset, and synchronization limits. | Supervised J2 test record |
| SAF-004 | A physical operator stop and power-disconnect procedure shall be available during every robot test. | Documented and rehearsed procedure |
| SAF-005 | Communication loss, invalid commands, and controller errors shall lead to a defined safe response. | Fault-injection or supervised disconnect test |
| SAF-006 | Initial real-robot tests shall use reduced speed, a clear workspace, and an operator within reach of the power disconnect. | Test checklist |
| SAF-007 | Supply current capacity, wiring, and voltage limits shall be checked against the robot's requirements before loaded operation. | Electrical verification record |
| SAF-008 | RL actions shall be bounded by a safety layer independent of the learned policy. | Software tests and deployment review |

Do not enable torque or run an RL policy on the physical robot until the actuator mapping, calibration, joint limits, power capacity, and operator stop procedure have been verified.

## 9. Evaluation Requirements

The project shall record results for simulation and physical trials. Numerical pass thresholds are intentionally left open until the task object, workspace, repeatability protocol, and measurement method are fixed.

| Metric | Definition / measurement |
|---|---|
| Pick-and-place success rate | Successful completed transfers divided by valid trials |
| Grasp success rate | Trials in which the object is securely acquired |
| Placement error | Distance between the final object reference point and target |
| TCP position error | Distance between commanded and measured/estimated TCP positions |
| Task completion time | Time from task start to completed placement |
| Object drop rate | Trials in which the object is dropped before placement |
| Joint-limit violations | Number of commanded or measured limit violations |
| Collision events | Number of detected or observed collisions |
| Training performance | Episode return, success rate, and learning curves |
| Reproducibility | Results across repeated runs with recorded configuration and seeds |

Each experiment shall record the policy/checkpoint identifier, software revision, environment configuration, random seed where applicable, object properties, task locations, trial outcome, and relevant logs.

## 10. One-Month Delivery Plan

The one-month deadline is a project constraint. The calendar start date and exact defense date must be confirmed.

| Priority | Milestone | Required result |
|---|---|---|
| M1 | Hardware access and safe control | Verified motor map, feedback, limits, and safe low-speed control |
| M2 | Kinematic model and simulation | Validated reference-pose model and working simulation |
| M3 | RL environment and training | Joint-space environment and a policy with repeatable simulated performance |
| M4 | Real-robot integration | Staged policy deployment and a physical fixed-location pick-and-place demonstration |

Cartesian-space RL and camera integration are secondary to the required physical demonstration. If time becomes constrained, these extensions must not displace hardware safety, joint-space training, or the physical pick-and-place milestone.

## 11. Open Issues and Required Decisions

| ID | Open issue | Required action |
|---|---|---|
| OPEN-001 | DYNAMIXEL IDs 1–7 are discovered but not mapped to physical joints | Identify each motor physically and document its logical joint |
| OPEN-002 | J2 dual-motor direction, offsets, and synchronization | Verify at low speed with a safe test procedure |
| OPEN-003 | Raw position values and reported degree conversion | Verify conversion, motor direction, and zero offsets in ARMPy |
| OPEN-004 | Configured raw limits 0–4095 are not verified mechanical limits | Establish safe per-joint operating limits |
| OPEN-005 | Controller `mode` value 2 and `dxl_power_state` value 1 | Confirm field meanings from authoritative documentation or code |
| OPEN-006 | Supply model, rated current, and current limit | Record supply specifications and verify capacity |
| OPEN-007 | Current serial device path may change | Implement reliable device identification |
| OPEN-008 | NVIDIA driver unavailable to `nvidia-smi` in the previous system audit | Diagnose driver before relying on GPU training |
| OPEN-009 | Gripper opening range, object dimensions, and payload | Measure gripper and select a lightweight test object |
| OPEN-010 | Fixed pickup and placement coordinates | Define and record task locations in the Base frame |
| OPEN-011 | Numerical evaluation thresholds | Set thresholds after initial repeatability and measurement tests |
| OPEN-012 | Calendar deadline | Record project start date and defense date |

## 12. Phase 0 Exit Criteria

Phase 0 may be closed when the following baseline is reviewed and accepted:

- [ ] Project objective, scope, one-month priority, and camera extension are approved.
- [ ] Five logical joints, gripper, and seven physical actuators are documented.
- [ ] Every DYNAMIXEL ID is mapped to its physical location and logical function.
- [ ] Shoulder motor direction and synchronization approach are documented.
- [ ] Mechanical reference frames and measurements have evidence and a validation status.
- [ ] Communication parameters and controller field meanings are understood.
- [ ] Power capacity and operator stop procedure are documented.
- [ ] Software stack compatibility and GPU-driver risk are assessed.
- [ ] Functional, safety, and evaluation requirements are reviewed.
- [ ] Remaining open issues have owners or next actions.
- [ ] Phase 1 work plan is approved.

## 13. Decision Record

The following scope decisions were explicitly selected by the project owner:

| Decision | Selected direction |
|---|---|
| Research focus | Design, implementation, and evaluation of an RL-based system; classical-controller comparison is not required |
| Camera | Extension after the fixed-location pick-and-place milestone |
| RL control spaces | Both joint-space and Cartesian-space are planned |
| Implementation order | Joint-space first; Cartesian-space after the core system works |
| Mandatory final demonstration | Physical pick-and-place |
| Delivery constraint | Target completion within one month |

---

*This document is a project requirements baseline, not a certified hardware specification or a safety certification. Values extracted from the controller are time-stamped diagnostic data and must be rechecked before physical deployment.*
