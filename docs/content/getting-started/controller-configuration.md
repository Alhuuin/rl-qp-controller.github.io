---
title: Configure the controller
---

# Configure the controller

This page contains the **minimum controller-level setup** required before loading a policy. Every available option is documented separately in [Controller configuration reference](../reference/controller-options.md).

## 1. Select the robot state used by observations

```yaml
robot:
  base_body: pelvis
  observation_source: realRobot
```

`base_body` is the default body used by floating-base observations. `observation_source` selects either `realRobot` or `robot`.

Use `realRobot` when observations must reflect measured hardware state. Use `robot` only when the control robot is intentionally the source.

## 2. Point to the policy directory

```yaml
policies_root: "@CMAKE_CURRENT_SOURCE_DIR@/policies"
default_policy: my_policy
```

`policies_root` contains `conventions.yaml` and one subdirectory per policy. `default_policy` must resolve to a discovered policy name.

## 3. Configure the minimum observer pipeline

The selected observation source must provide every signal requested in `observations.yaml`. Joint observations typically need an encoder observer:

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

Add body-sensor, tilt, force, or external-force observers only when the robot and selected observations require them.

## 4. Configure constraints deliberately

The template contains examples, not universal defaults:

```yaml
collisions:
  - type: collision
    useMinimal: true

# constraints:
#   - type: contact
#     contactType: acceleration
#   - type: compoundJoint
```

Do not copy contact surfaces, force-sensor names, observer callbacks, or compound-joint assumptions from another robot without checking the target robot module.

## 5. Define the FSM entry point

The base template and H1 adaptation use one state that runs the policy immediately. HRP5P starts in a posture-controlled `Initial` state and transitions to a dedicated `RL` state.

A single-state configuration looks like:

```yaml
transitions:
  - [MyController_Initial, OK, MyController_Initial, Strict]
init: MyController_Initial
```

Choose the state architecture before copying transitions. See the [variant comparison](../robots/comparison.md).

## Validation checklist

- [ ] `base_body` exists on the selected observation robot.
- [ ] `observation_source` provides complete encoders and every requested sensor.
- [ ] `policies_root` exists at runtime.
- [ ] `default_policy` matches a discovered policy.
- [ ] constraints and collisions reference valid robot resources.
- [ ] the configured initial state is compiled and exported.
- [ ] the robot command interface supports the control mode used by the controller.

For all integration settings and variant-specific fields, continue with [Controller configuration reference](../reference/controller-options.md).
