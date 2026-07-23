---
title: HRP5P
---

# HRP5P adaptation

HRP5P has the largest departure from the base template. It adds a true non-RL Initial state and a dedicated RL state, together with runtime control-mode management and contact-related integration.

## Active FSM states

Only these states are enabled in `src/states/CMakeLists.txt`:

- `HRP5pRLQPController_Initial`
- `HRP5pRLQPController_RL`

The tree also contains Torque, position-control, no-QP, and compliant-arm state sources, but their CMake entries are commented out. Treat them as dormant examples, not available runtime states.

## Initial state

The controller starts in position control. The Initial state:

- enables QP;
- requests position control;
- enables contact constraints in the runtime;
- targets either the robot default posture or policy `q_zero`, according to `is_initial_posture_rl`;
- adds the posture task;
- does not run policy inference.

The active transitions are:

```text
Initial --RL--> RL
RL      --OK--> Initial
```

## RL state

On entry, the RL state:

- enables QP;
- requests torque control;
- disables runtime contact constraints;
- activates external-torque computation;
- adds the torque task and removes the posture task;
- restores policy PD gains;
- starts the RL runner.

On exit, it removes the torque task and deactivates external-torque computation.

## Main controller additions

### Mode-aware run loop

`manageModeSwitching()` applies requested datastore `ControlMode` changes and selects the solver feedback:

- torque control: `ClosedLoopIntegrateReal`;
- position control with real floating base: `OpenLoopWithRealFloatingBase`;
- otherwise: normal FSM `run()`.

### Additional tasks and gains

HRP5P retains a `PostureTask`, `CompliantPostureTask`, and `TorqueJointTask`, stores the default posture target, and loads complete high-gain maps. The compliant task is only referenced by a disabled state in the supplied tree.

### External-force estimator control

`activateExternalTorqueComputation()` uses datastore calls to toggle the estimator only when its requested state differs from its current state.

### Force-sensor contact logic

`updateFootContactsFromForceSensors()` reads `LeftFootForceSensor` and `RightFootForceSensor`, applies independent Schmitt triggers, and chooses left, right, or double support before changing contacts. Its call is currently commented in the active Initial state, so this helper is implemented but not active in the supplied execution path.

## Configuration additions

HRP5P enables contact and compound-joint constraints, updates a `FloatingBase` BodySensor including acceleration, uses a force-sensor-based external-force observer with motor torque measurements, and provides `is_initial_posture_rl`, `high_kp`, and `high_kd`.
