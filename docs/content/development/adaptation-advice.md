---
title: Adaptation advice
---

# Adaptation advice

While the policy runtime is designed to be robot-agnostic, integrating a new robot often requires adapting parts of the controller to its hardware and control architecture.

This page highlights examples of adaptations encountered when integrating a new robot.

## Initial state

Some robots can safely start directly in torque control and immediately execute the RL policy.

Others require an initialization phase before enabling RL, for example to:

- reach a stable posture;
- initialize estimators;
- wait for operator validation;
- smoothly switch from another control mode.

In this case, consider introducing a dedicated `Initial` state before entering the RL state.

!!! example "Example"

    The HRP5P controller includes a dedicated `Initial` state before switching to RL, while the H1 controller starts directly in the RL state.

---

## Contact handling

Choose a contact model appropriate for the robot.

Typical options include:

- force sensor based contact detection;
- externally estimated contact forces;
- fixed or manually controlled contact states.

The selected strategy may also determine which observers and constraints are required.

!!! example "Example"

    The HRP5P implementation relies on force sensors, whereas the H1 implementation illustrates how to operate without dedicated foot force sensors.

---

## Available sensors

Not every robot provides the same measurements. Modifications of the observations implementations might be needed.

When configuring `observations.yaml`, only enable observations that can be computed from the available hardware.

Typical differences include:

- force sensors;
- IMU availability;
- floating-base estimation;
- external force estimation.

!!! tip

    The H1 and HRP5P repositories provide examples of observation configurations for robots with different sensing capabilities.

---

## Tilt observer

The `Tilt` observer estimates the robot orientation with respect to an anchor frame. It is usually necessary in this controller for floating-base robots.

Unlike most observations, enabling this observer also requires controller-side integration. The controller must register an `anchorFrameFunction` callback in the datastore before the observer is created.

The callback returns the anchor frame used as the tilt reference. Its implementation is robot-specific and may depend on the robot's contact model.

!!! warning

    Enabling the `Tilt` observer in `observations.yaml` is not sufficient. The corresponding datastore callback must also be provided by the controller.

!!! example "Example"

    The H1 implementation demonstrates how to register the `anchorFrameFunction` callback and implement the robot-specific anchor frame computation.

---

## External Forces Observer

The `ExternalForcesObserver` estimates external contact forces and should be configured according to the sensing capabilities of the robot.

Robots equipped with force/torque sensors can directly exploit these measurements, whereas robots without dedicated force sensors generally rely on momentum-based estimation.

Important parameters include:

- `estimation_method`
- `torque_source_type`
- `residual_gain`
- `use_forces_from_ft_sensors`

These parameters directly depend on the robot hardware and should be reviewed when adapting the controller.

!!! example "Examples"

    The H1 implementation illustrates a momentum-based configuration (`MomentumObserver`) without force/torque sensors, while the HRP5P implementation provides an example of a force-sensor-based configuration (`ForceSensorBased`).

---

## Control modes

Robots differ in the control interfaces they expose. Choose the controller architecture accordingly.

!!! example "Example"

    The H1 controller is designed around torque control, while the HRP5P implementation demonstrates a controller that transitions from an initialization phase to RL execution.

---

## Controller tasks and constraints

The default task and constraint set is intended as a starting point.

Depending on the robot, additional components may be required, such as constraints or robot-specific mechanisms.

Adapt the task stack to the robot rather than trying to preserve the default configuration.
