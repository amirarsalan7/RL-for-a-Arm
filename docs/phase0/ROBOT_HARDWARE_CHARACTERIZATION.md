# ARM5R-RL — Robot Hardware Characterization

| Field | Value |
|---|---|
| Document ID | P0-HW-001 |
| Version | 0.2 |
| Date | 2026-11-15 |
| Project | ARM5R-RL |
| Status | Phase 0 — in progress |
| Related document | [MECHANICAL_MEASUREMENTS.md](MECHANICAL_MEASUREMENTS.md) |

## 1. Purpose and Status Conventions

This document records the physical robot, actuators, electronics, and the agreed reference-pose geometry. It is the hardware baseline for kinematic modeling, simulation, control, and experiments.

- **Owner-confirmed:** explicitly confirmed by the project owner for the current robot/reference pose.
- **Reported:** provided by the owner or existing software, but not independently checked.
- **Unknown:** still requires measurement, inspection, or testing.

A documented geometric convention does not by itself validate motor calibration or make automatic motion safe.

## 2. Robot Overview

| Property | Current specification | Status |
|---|---|---|
| Platform | Arm6x kit, Pishrobot | Reported |
| Manipulator | Serial arm with five revolute joints (5R) | Owner-confirmed |
| Arm joints | Base, shoulder, elbow, wrist pitch, wrist rotation | Owner-confirmed |
| Gripper | Two-finger, gear-driven, one motor | Reported |
| Arm actuators | Six DYNAMIXEL XL430-W250 motors; two actuate J2 | Reported |
| Total actuators | Seven, including the gripper motor | Reported |
| Controller | ROBOTIS CM-550 | Reported |
| Previous operation | Previously controlled with Python and DYNAMIXEL SDK; object grasping demonstrated | Reported |
| CAD / URDF | Not yet available; geometry is being reconstructed | Owner-confirmed |

`J1`–`J5` identify logical joints. `M1`–`M7` are temporary actuator labels, not verified DYNAMIXEL IDs.

## 3. Joint and Actuator Map

| Joint | Function | Actuator(s) | Positive axis in Base frame at P0 |
|---|---|---|---|
| J1 | Base rotation | M1 | `+Z_B` |
| J2 | Shoulder pitch | M2, M3 | `+Y_B` |
| J3 | Elbow pitch | M4 | `+Y_B` |
| J4 | Wrist pitch | M5 | `+Y_B` |
| J5 | Wrist rotation | M6 | `+X_B` |
| Gripper | Two-finger opening/closing | M7 | To be characterized |

The two J2 motors drive one logical degree of freedom. Their motor directions are opposite for the same logical shoulder movement. The logical positive J2 direction is defined as `+Y_B`; the physical motor signs, offsets, IDs, and synchronization limits still require validation.

## 4. Coordinate-Frame Convention and Reference Pose

The agreed base frame is right-handed:

- Origin `O_B`: base reference origin defined in `MECHANICAL_MEASUREMENTS.md`.
- `+Z_B`: vertically upward, along the J1 rotation axis.
- `+X_B`: forward direction in the reference pose.
- `+Y_B`: completes the right-handed frame.

The reference pose is `P0`: J2 and J3 are aligned vertically; J4, J5, and the TCP extend horizontally in `+X_B`.

Each joint frame has its local `+Z` axis aligned with that joint's positive rotation axis. At `P0`, the joint-axis directions expressed in the base frame are:

| Joint frame | Local `+Z` direction in Base frame |
|---|---|
| J1 | `+Z_B` |
| J2 | `+Y_B` |
| J3 | `+Y_B` |
| J4 | `+Y_B` |
| J5 | `+X_B` |

### 4.1 Reference-pose landmark coordinates

Coordinates are in millimetres and expressed in the Base frame.

| Landmark | X | Y | Z |
|---|---:|---:|---:|
| Base origin | 0 | 0 | 0 |
| J1 origin | 0 | 0 | 80 |
| J2 origin | 0 | 0 | 116 |
| J3 origin | 0 | 0 | 253 |
| J4 origin | 65 | 0 | 253 |
| J5 origin | 135 | 0 | 253 |
| TCP | 215 | 0 | 253 |

The J2-to-J3 physical displacement is 137 mm upward in the Base frame. Since the J2 frame has `+Y_J2` directed downward at `P0`, the same displacement is `(0, -137, 0) mm` in the J2 frame.

### 4.2 Fixed transforms at P0

Translations are expressed in the parent frame. Orientation uses roll-pitch-yaw with:

`R = Rz(yaw) · Ry(pitch) · Rx(roll)`

| Parent → child | Translation (mm) | Roll | Pitch | Yaw |
|---|---:|---:|---:|---:|
| Base → J1 | `(0, 0, 80)` | 0° | 0° | 0° |
| J1 → J2 | `(0, 0, 36)` | −90° | 0° | 0° |
| J2 → J3 | `(0, −137, 0)` | 0° | 0° | 0° |
| J3 → J4 | `(65, 0, 0)` | 0° | 0° | 0° |
| J4 → J5 | `(70, 0, 0)` | 0° | +90° | 0° |
| J5 → TCP | `(0, 0, 80)` | 0° | 0° | 0° |

These are fixed frame relationships at `P0`; the variable joint angles are separate. The J5 rotation axis is `+X_B` at `P0`, and the TCP lies on that axis, 80 mm from the J5 origin.

The coordinate convention and values above are the agreed project baseline. Their measurement methods and evidence are maintained in `MECHANICAL_MEASUREMENTS.md`.

## 5. Communication and Software Baseline

### 5.1 Communication path

```text
Ubuntu host / Python / ARMPy
          |
          | USB serial
          v
     ROBOTIS CM-550
          |
          | DYNAMIXEL bus
          v
   Arm actuators + gripper
```

The robot has previously been operated through the USB-connected CM-550 using Python and the DYNAMIXEL SDK. The current serial device path, baud rate, protocol version, firmware, bus wiring, and physical motor-ID map still need to be recorded.

### 5.2 ARMPy

- Repository: https://github.com/amirarsalan7/Armpy
- Reference commit: `7887a4a30c8a6419916a98b8a319b60701dbe1dd`
- Local checkout: `~/projects/Armpy/`
- ARM5R-RL repository: `~/projects/arm5r-rl/`

ARMPy is the existing low-level hardware-control foundation. Its calibration, motor mapping, operating modes, and safety limits must be checked against the current physical assembly before automated operation.

## 6. Power and Safety

The robot is powered by a laboratory DC supply. The owner reports an operating voltage of approximately 11–12 V. A photograph shows 11.7 V and 0.00 A at the instant captured; this is not a measurement of current during motion or grasping.

| Item | Current information | Status |
|---|---|---|
| Operating voltage | Approximately 11–12 V | Reported |
| Voltage in photograph | 11.7 V | Visually observed |
| Current in photograph | 0.00 A at that instant | Visually observed |
| Supply model and maximum current | Not recorded | Unknown |
| Current-limit setting | Not recorded | Unknown |
| Power distribution / grounding | Not documented | Unknown |
| Operator stop / power-disconnect procedure | Not documented | Unknown |

Do not run automated multi-joint motion until the motor map, calibration, joint limits, shoulder synchronization, supply capacity, and operator stop procedure have been checked.

## 7. Mechanical Structure and Gripper

The robot has a metal base and metal link brackets. Detailed dimensions, frame origins, and reference-pose transforms are recorded in [MECHANICAL_MEASUREMENTS.md](MECHANICAL_MEASUREMENTS.md); this document does not duplicate the measurement worksheet.

Still to be characterized:

- Gripper minimum and maximum opening width.
- Gripper geometry and TCP definition against the physical tool.
- Mechanical joint limits and collision/interference regions.
- Link masses and centres of mass, if needed for the selected dynamics model.

## 8. Pick-and-Place Scope

The mandatory demonstration is physical pick-and-place between predefined pickup and placement locations. Camera-based localization is a planned extension, not a prerequisite for the initial demonstration.

| Task parameter | Status |
|---|---|
| Test-object shape, material, dimensions, and mass | To be selected and measured |
| Pickup and placement coordinates in Base frame | To be defined |
| Gripper opening suitable for the test object | To be measured |
| Required success rate and number of trials | To be defined in `PROJECT_REQUIREMENTS.md` |
| Camera-based object localization | Optional future extension |

A close command alone does not prove a successful grasp; object retention and placement must be checked during trials.

## 9. Open Issues

| ID | Open issue | Required action |
|---|---|---|
| HW-001 | Physical motor IDs and actuator mapping | Record each DYNAMIXEL ID and its physical joint/motor location |
| HW-002 | Dual-motor shoulder calibration | Verify signs, offsets, operating modes, and synchronization limits |
| HW-003 | Joint limits and stop procedure | Establish and test safe limits and operator stop procedure |
| HW-004 | Power-supply capacity | Record supply model, current rating, and current-limit setting |
| HW-005 | Gripper dimensions and test object | Measure opening range; select and weigh a test object |
| HW-006 | Communication configuration | Record serial path, baud rate, protocol, firmware, and bus topology |
| HW-007 | NVIDIA driver | Diagnose the `nvidia-smi` communication failure before relying on GPU training |

## 10. Phase 0 Completion

Phase 0 is ready for review when:

- [ ] Physical actuator IDs are mapped to J1–J5 and the gripper.
- [ ] Shoulder motor signs, offsets, and safe synchronization limits are verified.
- [ ] Mechanical reference measurements and evidence are recorded.
- [ ] Power capacity and operator stop procedure are documented.
- [ ] Gripper geometry and the test object are specified.
- [ ] Communication settings and the ARMPy configuration are recorded.
- [ ] Remaining risks and deferred work are listed in `PROJECT_REQUIREMENTS.md`.

*This document records the current project baseline. It is not a certified robot specification or a validated safety controller.*
