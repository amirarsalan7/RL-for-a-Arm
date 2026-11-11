# ARM5R-RL: Reinforcement Learning for a 5R Robotic Manipulator

**Status:** Phase 0 — project definition and robot characterization  
**Target schedule:** Approximately 30 days of full-time work  
**Documentation language:** English (Markdown)

## 1. Project overview

ARM5R-RL is a robotics engineering and research project focused on a physical serial manipulator with **five revolute joints (5R)** and a motorized two-finger gripper. The platform is an Arm6x kit supplied by Pishrobot, built with ROBOTIS DYNAMIXEL actuators and a CM-550 controller.

The project aims to deliver a working, documented, and experimentally evaluated manipulation system. It will build on the existing **ARMPy** Python control framework rather than reimplementing low-level motor communication.

The initial demonstration uses **fixed, predefined pickup and placement locations**. Camera-based object localization is a planned extension, not a prerequisite for the first demonstration.

## 2. Objectives and scope

### Mandatory engineering outcome

Demonstrate a **physical pick-and-place task**: approach an object at a predefined location, grasp it, lift and transfer it, release it at a predefined destination, and return to a safe state. The system must provide a repeatable software-controlled procedure and appropriate motion-safety checks.

### Research outcome

Develop and evaluate a reinforcement-learning approach for at least the **reaching** component of manipulation. Compare it with a conventional or scripted control baseline using a consistent evaluation protocol. Where feasible, integrate the learned policy with the physical robot. The report must clearly distinguish learned behavior from scripted behavior.

### Optional extensions

- Camera-based object detection and pose estimation for variable pickup locations.
- More extensive reinforcement learning for grasping or full pick-and-place.
- ROS 2 integration and additional algorithm comparisons, if time permits.

## 3. Hardware baseline

| Component | Current understanding | Verification status |
| --- | --- | --- |
| Robot | Arm6x serial manipulator | Reported; assembled and previously operated |
| Arm joints | J1 base rotation; J2 shoulder; J3 elbow; J4 wrist pitch; J5 wrist rotation | Reported |
| Arm degrees of freedom | Five revolute joints | Reported |
| Arm actuators | Six DYNAMIXEL motors; two drive J2 | Reported |
| Gripper | Two-finger geared gripper with one motor | Reported; previously operated |
| Total actuators | Seven DYNAMIXEL XL430-W250 motors | Reported; confirm labels and IDs |
| Shoulder coupling | Two motors on a common nominal joint axis, rotating in opposite motor directions | Reported; calibration must be verified |
| Joint axes | J1 vertical; J2–J4 parallel; J5 perpendicular to J2–J4 | Reported; coordinate frames pending |
| Controller | ROBOTIS CM-550 connected to the host through USB serial | Previously operated; software configuration to be verified |
| Power | Laboratory supply, approximately 11–12 V | Voltage reported; current capacity and limits unknown |
| Geometry | No CAD/URDF yet; measurements will be obtained by reverse engineering | Pending |

The host computer runs **Ubuntu 24.04.5 LTS** with **Python 3.12.3**, an **Intel Core i5-11400H**, approximately **16 GiB RAM**, and an **NVIDIA RTX 3050**. The NVIDIA driver is not currently functioning correctly; GPU acceleration must not be assumed until verified.

> **Important:** Five arm degrees of freedom do not mean five motors. J2 uses two actuators, and the gripper uses a seventh actuator. Motor IDs and joint labels must not be assumed to be identical.

## 4. Planned system architecture

1. **Task and perception layer:** fixed pickup/placement coordinates and a task state machine; optional camera perception later.
2. **Learning and simulation layer:** a robot model in MuJoCo, a Gymnasium-compatible environment, and reinforcement-learning experiments (initially considering SAC).
3. **Robot control and safety layer:** joint-space commands, state feedback, shoulder-motor coordination, limits, command timeouts, and safe task execution.
4. **Hardware interface layer:** the existing ARMPy library, DYNAMIXEL SDK, CM-550, and physical actuators.

Simulation and real-hardware control should expose a consistent high-level interface. ROS 2 is a possible integration layer, but is **not a prerequisite** for the initial physical demonstration.

## 5. Relationship to ARMPy

ARMPy is maintained as a **separate Git repository** and reused as the low-level hardware-control foundation.

- Upstream: https://github.com/amirarsalan7/Armpy
- Initial reviewed revision: `7887a4a30c8a6419916a98b8a319b60701dbe1dd`

Recommended local workspace:

```text
~/projects/
├── Armpy/       # Existing hardware-control repository
└── arm5r-rl/    # This research and integration repository
```

Do not copy ARMPy's `.git` directory into this repository. Any future dependency installation or version update must be documented and tested.

## 6. Repository organization

```text
arm5r-rl/
├── README.md
├── .gitignore
├── PROJECT_REQUIREMENTS.md   # Planned Phase 0 deliverable
├── docs/
│   ├── phase0/               # Hardware characterization and measurements
│   ├── architecture/         # Design decisions and interfaces
│   └── experiments/          # Research protocols and analysis
├── assets/
│   ├── photos/               # Selected documented robot photographs
│   └── measurements/         # Mechanical measurement records
├── src/                      # Future simulation, control, and RL code
└── tests/                    # Future unit and integration tests
```

Directories may initially be empty; Git tracks files, not empty directories. The structure will evolve as implementation proceeds.

## 7. Development milestones

1. **Phase 0:** confirm project requirements, hardware inventory, kinematic structure, power and communication setup, and unresolved risks.
2. **Hardware validation:** verify ARMPy connectivity, motor mapping, calibration, joint limits, and safe movement.
3. **Classical baseline:** demonstrate repeatable physical pick-and-place with predefined object locations.
4. **Simulation and learning:** build a robot model, implement a reaching environment, and train/evaluate an RL policy.
5. **Integration and evaluation:** compare methods, document physical trials, and prepare the final demonstration and report.

## 8. Evaluation and reproducibility

Experiments will record the software revision, configuration, robot calibration version, task setup, random seed where applicable, number of trials, success/failure outcomes, and measured performance. Candidate metrics include **pick-and-place success rate**, **reaching position error**, **task completion time**, and **failure categories**. Exact acceptance thresholds will be defined in `PROJECT_REQUIREMENTS.md`.

## 9. Safety and current limitations

**The current repository does not yet constitute a validated safety controller.** Before automated motion, verify the calibration data, joint limits, opposite-direction shoulder motor mapping, power-supply capacity, and a reliable stop procedure. Never treat previously saved encoder values as safe targets without checking their operating mode and calibration context.

Open items include exact link geometry, gripper reference frame, current limits, calibration consistency, object dimensions and mass, and camera availability. These must be recorded explicitly rather than replaced with assumptions.

## 10. Documentation conventions

All official project documentation is written in **English** using **Markdown (`.md`)**. Each technical claim or measurement should be marked as **Confirmed**, **Reported**, **Estimated**, or **Unknown** where relevant. Use SI units, identify coordinate frames, record test conditions, and keep configuration and experiment changes under version control.

**Next document:** `PROJECT_REQUIREMENTS.md` — the formal specification and Phase 0 acceptance criteria.
