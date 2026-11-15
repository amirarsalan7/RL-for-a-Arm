# ARM5R-RL — Mechanical Measurements and Joint-Frame Specification

| Field | Value |
|---|---|
| Document ID | `P0-MECH-001` |
| Version | `0.2` |
| Updated | 2026-11-15 |
| Status | Draft — internally consistent at reference pose `P0`; physical measurement evidence pending |
| Robot | Arm6x / ARM5R: five revolute arm joints and a two-finger gripper |
| Units | Millimetres (mm), degrees (°), grams (g) |

## 1. Purpose and Conventions

This document records the geometry required to create the ARM5R forward-kinematics model and robot description. Every revolute joint has a **right-handed local frame** `{i}` whose **local `+Z_i` axis is the chosen positive joint rotation axis**.

For each transition from parent frame `{P}` to child frame `{C}`, record two separate quantities:

- **Translation** `t_P,C = [x, y, z]`: location of the child origin expressed along the **parent's axes**, in mm.
- **Fixed orientation** `R_P,C^0`: orientation of the child axes relative to the parent axes at `P0`, recorded as roll, pitch and yaw in degrees.

The variable revolute joint angle `q_i` is separate from the fixed orientation. An orientation of `(0°, 0°, 0°)` means **aligned frames**, not a missing orientation.

**Evidence status:** `Defined` = coordinate choice; `Owner-confirmed` = explicitly confirmed by the robot owner; `Reported` = entered measurement without supporting method/photo; `Pending` = not yet determined or checked.

## 2. Reference Pose and Base Frame

### 2.1 Reference pose `P0`

The robot owner confirmed the following physical arrangement at `P0`:

- `J2 → J3` points vertically upward.
- `J3 → J4 → J5 → TCP` lies horizontally along `+X_B`.
- The `J5` rotation axis points along `+X_B`.
- The TCP lies **on the J5 rotation axis**, 80 mm along its positive direction from `O_5`.

Define the **geometric** joint angles as `q1 = q2 = q3 = q4 = q5 = 0` at `P0`. These zeros are **not necessarily** the saved DYNAMIXEL encoder or ARMPy HOME values.

### 2.2 Fixed base frame `{B}`

| Axis / point | Definition |
|---|---|
| `O_B` | Intersection of the J1 axis with the stationary base's bottom reference plane |
| `+Z_B` | Vertical, upward, along J1 |
| `+X_B` | Horizontal, toward the arm/TCP at `P0` |
| `+Y_B` | Completes the right-handed frame: `X_B × Y_B = Z_B` |

Frame `{B}` is attached to the **stationary base**, not to the rotating structure. Verify the selected physical bottom plane and mark these directions in a reference photograph.

### 2.3 Joint origins

Each `O_i` is a selected, repeatable point **on the physical rotation axis** of joint `Ji`. `O_2` is one logical shoulder-joint origin despite its two oppositely driven motors. `O_TCP` is the grasp-center reference point for the chosen gripper state.

**Measurement safety:** Disconnect actuator power and support the arm before handling or measuring joints. Do not force the dual-motor shoulder or the gripper.

## 3. Joint-Axis Directions at `P0`

By convention, every joint's axis is `[0, 0, 1]` **in its own frame**. The directions below express those same axes in the fixed base frame.

| Joint | Chosen positive `+Z_i` in `{B}` at `P0` | Basis |
|---|---|---|
| J1 — Base | `+Z_B` | Defined from the vertical base axis |
| J2 — Shoulder | `+Y_B` | Owner-approved positive-axis convention |
| J3 — Elbow | `+Y_B` | Chosen to match the parallel J2 axis; positive hardware sign pending |
| J4 — Wrist pitch | `+Y_B` | Chosen to match J2/J3; positive hardware sign pending |
| J5 — Wrist rotation | `+X_B` | Owner-confirmed at `P0` |

Motor encoder directions, especially the two shoulder motors, must be mapped to these **logical** joint signs separately.

## 4. Landmark Coordinates in the Base Frame

All positions below refer to `P0` and are expressed in `{B}`. The numeric entries come from the owner's measurement worksheet; the measurement endpoints, tolerances and photographic evidence have not yet been documented.

| Landmark | Reference | `X_B` (mm) | `Y_B` (mm) | `Z_B` (mm) | Status |
|---|---|---:|---:|---:|---|
| `O_B` | Base datum | 0 | 0 | 0 | Defined |
| `O_1` | J1 axis reference point | 0 | 0 | 80 | Reported |
| `O_2` | Common shoulder axis | 0 | 0 | 116 | Reported |
| `O_3` | Elbow axis | 0 | 0 | 253 | Reported; consistent with 137 mm J2→J3 |
| `O_4` | Wrist-pitch axis | 65 | 0 | 253 | Reported |
| `O_5` | Wrist-rotation axis | 135 | 0 | 253 | Reported |
| `O_TCP` | Grasp center on J5 axis | 215 | 0 | 253 | Reported; 80 mm axial offset owner-confirmed |

**Check before declaring these coordinates verified:** confirm the selected shaft-center references, the origin of the 80 mm J1 height, and that all recorded `Y_B = 0` values represent actual alignment rather than unmeasured offsets.

## 5. Fixed Frame-to-Frame Translation

The following are **parent-frame components**, not automatically base-frame differences. They are consistent with Sections 4 and 6 at `P0`.

| Transition | `t_x` (mm) | `t_y` (mm) | `t_z` (mm) | GEO ID(s) |
|---|---:|---:|---:|---|
| `{B} → {1}` | 0 | 0 | 80 | GEO-001 |
| `{1} → {2}` | 0 | 0 | 36 | GEO-002–004 |
| `{2} → {3}` | 0 | −137 | 0 | GEO-005–007 |
| `{3} → {4}` | 65 | 0 | 0 | GEO-008–010 |
| `{4} → {5}` | 70 | 0 | 0 | GEO-011–013 |
| `{5} → {TCP}` | 0 | 0 | 80 | GEO-014–016 |

For measured landmark positions `p_B,P` and `p_B,C`, and parent orientation `R_B,P(P0)`:

```text
t_P,C = transpose(R_B,P(P0)) @ (p_B,C - p_B,P)
```

**Why `J2 → J3` is negative Y:** At `P0`, `+Y_2 = −Z_B`; therefore the 137 mm upward base-frame displacement is `−137 mm` along J2's local Y.

## 6. Fixed Frame-to-Frame Orientation

Use the roll–pitch–yaw convention

```text
R_P,C^0 = Rz(yaw) @ Ry(pitch) @ Rx(roll)
```

`R_P,C^0` maps vectors **from child-frame coordinates into parent-frame coordinates**. Values are fixed at `P0`; joint angles are applied separately.

### 6.1 Relative fixed rotations

| Transition | Roll (°) | Pitch (°) | Yaw (°) | Child `+Z` expressed in parent frame | Status |
|---|---:|---:|---:|---|---|
| `{B} → {1}` | 0 | 0 | 0 | `[0, 0, +1]` | Defined |
| `{1} → {2}` | **−90** | 0 | 0 | `[0, +1, 0]` | Defined; J2 positive axis owner-approved |
| `{2} → {3}` | 0 | 0 | 0 | `[0, 0, +1]` | Defined; parallel axes and aligned X references |
| `{3} → {4}` | 0 | 0 | 0 | `[0, 0, +1]` | Defined; parallel axes and aligned X references |
| `{4} → {5}` | 0 | **+90** | 0 | `[+1, 0, 0]` | Defined; J5 physical direction owner-confirmed |
| `{5} → {TCP}` | 0 | 0 | 0 | `[0, 0, +1]` | Defined tool-frame convention; finger-plane reference pending |

### 6.2 Full local-frame axes expressed in `{B}` at `P0`

The third axis always follows `+Y_i = +Z_i × +X_i`.

| Frame | `+X` in `{B}` | `+Y` in `{B}` | `+Z` in `{B}` |
|---|---|---|---|
| `{B}` | `+X_B` | `+Y_B` | `+Z_B` |
| `{1}` | `+X_B` | `+Y_B` | `+Z_B` |
| `{2}` | `+X_B` | `−Z_B` | `+Y_B` |
| `{3}` | `+X_B` | `−Z_B` | `+Y_B` |
| `{4}` | `+X_B` | `−Z_B` | `+Y_B` |
| `{5}` | `−Y_B` | `−Z_B` | `+X_B` |
| `{TCP}` | `−Y_B` | `−Z_B` | `+X_B` |

These choices form **right-handed, orthonormal frames**. In particular, J2's Y and Z axes are **not parallel**. The `{TCP}` axes are a convenient frame definition, not yet a verified description of the gripper finger plane.

## 7. Kinematic Transform and Reference-Pose Check

For revolute joint `Ji`, with geometric angle `q_i` about its **local `+Z_i`**:

```text
T_P,i(q_i) = [ R_P,i^0 @ Rz(q_i)    t_P,i ]
             [       0  0  0          1   ]
```

The fixed tool transform is `T_5,TCP` (no sixth arm joint):

```text
T_B,TCP(q) = T_B,1(q1) @ T_1,2(q2) @ T_2,3(q3)
           @ T_3,4(q4) @ T_4,5(q5) @ T_5,TCP
```

**Internal consistency check at `P0` (`q = 0`):** the tables reproduce all Section 4 landmark coordinates and predict

```text
p_B,TCP(P0) = [215, 0, 253] mm
```

The predicted TCP `+Z` points along `+X_B`. This is a **mathematical cross-check**, not an independently validated physical measurement.

## 8. Gripper and Pick-and-Place Object — Remaining Measurements

| Item | Value | Unit | Status |
|---|---|---|---|
| Useful maximum gripper opening | 50 | mm | Pending |
| Useful minimum gripper opening | 0 | mm | Pending |
| TCP offset along J5 axis | **80** | mm | Owner-confirmed |
| Gripper finger-plane orientation relative to `{5}` | 360 | degrees / axes | Pending |
| Test object dimensions (length × width × height) | 40 | mm | Pending |
| Test object mass | 100 | g | Pending |
| Fixed pickup location in `{B}` | TBD | mm (XYZ) | Pending |
| Fixed placement location in `{B}` | TBD | mm (XYZ) | Pending |

## 9. Measurement Evidence and Next Actions

Record the actual contact points, tool used, date, uncertainty where known, and labeled photographs under `assets/photos/`. Keep all pose measurements tied to `P0`.

**Remaining checks (in order):**

1. Photograph `P0` and label `O_B`, `O_1`–`O_5`, `O_TCP` and the positive base axes.
2. Confirm the origin selection and measurement method for `O_1 = (0, 0, 80) mm`.
3. Check all joint centers and verify the assumption `Y_B = 0` for each landmark.
4. Recheck the **137 mm** J2-to-J3 separation and record caliper endpoints.
5. Record the finger-plane direction and the test object's dimensions and mass.
6. Verify forward kinematics against a second, safely selected pose before applying the model to hardware.
7. Establish the mapping from geometric joint zeros and positive axes to the ARMPy encoder calibration.

**Model status:** fixed-frame translations and rotations are now **mutually consistent at `P0`**. They are not approved for automatic real-robot motion until physical references, operating limits and calibration have been validated.
