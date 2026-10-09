---
title: Configure the controller
---

# Controller configuration

The controller configuration file (`etc/<Controller>.in.yaml`) is structured like any standard mc_rtc controller configuration. This page documents the RL-specific settings it must contain, field by field, and the ObserverPipeline usually used.

If you are working through [the tutorial](../tutorial/index.md), see [Configure the controller](../tutorial/02-controller-configuration.md) for a worked example of filling in this file for a Unitree G1. If you are using an already-adapted version of this controller (such as G1, H1, or HRP5P), most of this is already done for you and you likely only need to verify a few settings rather than build them from scratch.

Standard mc_rtc settings (FSM states, transitions, etc.) can be added or modified freely; the policy handling will not interfere with them.

## Essential configuration

### 1. Select the robot state used by observations

```yaml
robot:
  base_body: pelvis
  observation_source: realRobot
```

| Key | Options | Meaning |
|---|---|---|
| `base_body` | robot body name | Default body used by floating-base observations. Should exist on your observation robot. |
| `observation_source` | `realRobot` or `robot` | Which robot state is used to compute observations. Use `realRobot` for measured hardware state; use `robot` only when the control robot is intentionally the source. |

### 2. Point to the policy directory

```yaml
policies_root: "@CMAKE_CURRENT_SOURCE_DIR@/policies"
default_policy: my_policy
```

| Key | Meaning |
|---|---|
| `policies_root` | Directory containing `conventions.yaml` and one subdirectory per policy. All policies under this root are loaded and can be switched to using the GUI. |
| `default_policy` | Name of the first policy to run. Must match a loaded policy. |

### 3. Configure the observer pipeline

The selected observation sources must provide everything requested in the `observations.yaml` of the default policy. Verify or adjust the observer pipeline as needed:

```yaml
ObserverPipelines:
  name: MainPipeline
  gui: true
  log: true
  observers:
    - type: Encoder
      config:
        velocity: encoderVelocities
```

For common policy observations, observer types include:

- **Encoder**: Provides joint positions and velocities.
- **BodySensor**: Estimates the floating-base state from body sensor measurements and kinematics.
- **Tilt**: Estimates floating-base tilt from IMU measurements and robot kinematics.
- **ExternalForcesObserver**: Estimates external torques applied to the robot; implementation varies by robot.

## Set FSM entry point

If needed, FSM states, transitions, and robot-specific state logic can be added as needed. The RL runtime will not interfere.

One example of this would be the addition of a stantionary state, from which to start the actual RL state. Useful for robots which need a few seconds to stabilize. 

## Validation checklist

- [ ] `base_body` exists on the selected observation robot.
- [ ] `policies_root` directory exists.
- [ ] `observation_source` provides complete encoders and every signal required by the `observations.yaml` of the default policy.
- [ ] `default_policy` matches a loaded policy name.
- [ ] The configured initial FSM state is compiled and exported.

## Behavior implemented in C++, not YAML alone

Different robot adaptations implement varying control logic in C++, such as:

- Runtime mode switching between position and torque control
- Solver feedback selection based on the active control mode
- External-force estimator toggling through datastore calls
- Force-sensor-driven foot-contact management
- QP enforcement or bypass paths for torque commands

Inspect the main controller and active FSM states together with the YAML configuration to understand the full control behavior. For context-based guidance, see [Adaptation advice](adaptation-advice.md).

For a concrete example of that C++-side behavior — registering an `anchorFrameFunction` datastore callback for the `Tilt` observer — see [Wire up state estimation](../tutorial/03-state-estimation.md) in the tutorial.
