---
title: Robot-specific information
---

# Robot-specific information

## Available variants

| Variant | Purpose | Status |
|---|---|---|
| [Base template](https://github.com/example/rl-qp-controller-template) | Starting point for a new robot | Generic base |
| [Unitree H1](https://github.com/example/rl-qp-controller-h1) | H1-specific controller and policies | Robot adaptation |
| [HRP5P](https://github.com/example/rl-qp-controller-hrp5p) | HRP5P-specific controller and latest architecture | Most recent reference |

## What is reusable

The following modules are intended to transfer across robots with limited changes:

- policy folder discovery and parsing;
- ONNX Runtime inference wrapper;
- observation registry and history manager;
- convention and alias resolution;
- action-output versus controlled-subset mapping;
- command and phase handling;
- common RL FSM lifecycle;
- policy-selection and gain GUI panels.

## HRP5P-specific integration

The supplied latest implementation additionally contains:

- HRP5P joint groups and aliases in `policies/conventions.yaml`;
- `pelvis` as the configured base body;
- HRP5P posture, torque, compliant-posture, and FSM states;
- left/right foot force sensor names and contact surfaces;
- Schmitt-trigger contact activation logic;
- external-force estimator datastore calls;
- HRP5P high-gain maps;
- HRP5P-specific logger and GUI prefixes;
- force/limit monitoring and contact changes.

### Foot-contact logic

The controller reads world-force norms from `LeftFootForceSensor` and `RightFootForceSensor`. Independent Schmitt triggers classify each sensor as active or inactive. When both are active, the controller compares their force difference:

- similar loads produce double support;
- significantly larger left load produces left-only support;
- significantly larger right load produces right-only support.

Contacts are added or removed only when the desired state changes.

!!! warning "This contact strategy is not generic"
    Sensor names, thresholds, surfaces, friction, and the assumption that load difference determines support must be validated for each robot and experiment.

## Before porting

Create an explicit robot inventory:

- reference joint order and single-DoF actuated joints;
- body and body-sensor names;
- force sensor names and frames;
- foot/contact surfaces;
- available torque interface;
- encoder and velocity availability;
- posture task behavior;
- observer pipeline requirements;
- joint, velocity, torque, and collision constraints.

Then follow [Adapt to a new robot](../development/adapt-new-robot.md).
