---
title: Adapt to a new robot
---

- Adjust and tune Observers oc

# Adapt to a new robot

Start from the base template. Use H1 and HRP5P as examples of different adaptation strategies, not as layers that must both be reproduced.

## 1. Inventory the robot interface

Record the robot module, `refJointOrder()`, actuated joint types, torque and position interfaces, runtime mode-switch capability, encoder sources, base/body sensors, force sensors, surfaces, limits, collision model, and available observer/datastore APIs.

## 2. Choose the state architecture

### Direct RL startup

Template/H1 pattern:

```text
startup -> torque control -> RL-running state
```

### Stable Initial state

HRP5P pattern:

```text
startup -> position posture -> explicit transition -> torque RL
```

The second pattern requires coordinated task handoff, command-mode switching, solver feedback selection, and state teardown. It cannot be added through YAML alone.

## 3. Rename exported integration points

Rename the controller class/files, exported constructor, state classes and export names, CMake targets, configuration file, GUI/logger prefixes, and test selection.

## 4. Define canonical order and conventions

The runtime stores full vectors in `robot().refJointOrder()` order. Verify policy gains cover required joints, action groups reproduce ONNX output order, controlled joints form a subset, observation groups match training, and fixed/multi-DoF/finger joints are handled intentionally.

## 5. Review every main-controller function

- constructor: tasks, weights, constraints, solver setup;
- `run()`: feedback mode, QP failure, direct bypass;
- `initializeRobotBasics()`: control mode, joint order, datastore callbacks;
- `byPassQPControl()`: command destination and measured state;
- `computeLimits()`: control robot versus real robot;
- robot helpers: contact anchors, estimator toggles, contact updates;
- GUI and logging.

See [Variant comparison](../robots/comparison.md).

## 6. Build observers from observation requirements

For every observation identify whether it needs encoders, velocities, multibody state, body sensors, force sensors, command, phase, or previous action. Configure only observers that provide those signals.

## 7. Configure constraints and contacts

Validate collision pairs, surfaces, friction, contact type, compound joints, CBF gains, damping/interaction distances, velocity percentages, and whether contacts are static, state-driven, or force-driven.

## 8. Adapt policies

Replace placeholder joints, gains, default pose, bodies, sensors, and ONNX files. Keep `minimalExample` and `fullExample` as format references or preserve equivalent documentation elsewhere.

## 9. Validate progressively

1. Build and load controller/state libraries.
2. Verify joint order, bodies, sensors, and observer output.
3. Load ONNX and validate dimensions.
4. Plot each observation.
5. Check action, scale, `q_zero`, and `q_rl`.
6. Test tasks and QP in simulation.
7. Test state transitions separately from policy behavior.
8. Use direct torque bypass only diagnostically.
9. Move to hardware after validating command and safety interfaces.

## Final review search

```bash
grep -RIn "TODO(robot)\|TODO(policy)" .
grep -RIn "H1\|HRP5\|pelvis\|FloatingBase\|LeftFoot\|RightFoot" \
  src include etc policies
```

Every remaining robot-specific name must be intentional.
