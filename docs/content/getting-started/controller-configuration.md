---
title: Configure the controller
---

# Configure the controller

If you are using a robot-adapted version of this controller (such as H1 or HRP5P), most of the configuration is already done for you. The controller configuration file is structured like any standard mc_rtc controller, and you likely only need to verify a few RL-specific settings rather than build it from scratch.

This page covers the core RL policy runtime configuration. Standard mc_rtc settings (FSM states, transitions, etc.) can be added or modified freely; the RL runtime will not interfere with them.

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
| `policies_root` | Directory containing `conventions.yaml` and one subdirectory per policy. All policies under this root are discovered automatically. |
| `default_policy` | Runtime name of the policy selected on startup. Must match a discovered policy. |

### 3. Configure the observer pipeline

The selected observation source must provide every signal requested in `observations.yaml`. Verify or adjust the observer pipeline as needed:

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

Common observer types include:

- **Encoder**: Provides joint positions and velocities. Nearly always required.
- **BodySensor**: Provides body-frame orientation and acceleration from IMU sensors.
- **Tilt**: Estimates tilt from body accelerations; some robots use this with datastore callbacks.
- **ExternalForces**: Estimates external torques applied to the robot; implementation varies by robot.
- **Force**: Provides force-sensor measurements from the robot feet or gripper.

For robot-specific observer configurations, see the [Robots](../robots/comparison.md) section.

### 4. Set FSM entry point

Define your initial FSM state and transitions. The exact structure depends on your robot variant—see the [Robots](../robots/comparison.md) section for examples. A simple entry looks like:

```yaml
transitions:
  - [MyController_Initial, OK, MyController_Initial, Strict]
init: MyController_Initial
```

You can add additional FSM states, transitions, and robot-specific state logic as needed. The RL runtime will not interfere.

## Validation checklist

- [ ] `base_body` exists on the selected observation robot.
- [ ] `observation_source` provides complete encoders and every signal required by `observations.yaml`.
- [ ] `policies_root` directory exists at runtime.
- [ ] `default_policy` matches a discovered policy name.
- [ ] The configured initial FSM state is compiled and exported.
- [ ] The robot command interface supports the control mode used by your selected robot variant.

## Behavior implemented in C++, not YAML alone

Different robot adaptations implement varying control logic in C++, such as:

- Runtime mode switching between position and torque control
- Solver feedback selection based on the active control mode
- External-force estimator toggling through datastore calls
- Force-sensor-driven foot-contact management
- QP enforcement or bypass paths for torque commands

Always inspect the main controller and active FSM states together with the YAML configuration to understand the full control behavior. For robot-specific details, see the [Robots](../robots/comparison.md) section.
