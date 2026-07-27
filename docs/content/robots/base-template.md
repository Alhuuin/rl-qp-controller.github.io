---
title: Base template
---

# Base template

The base repository is intended to be copied and adapted. It is not a universal robot controller.

## State architecture

The template builds one state, `NewRLQPController_Initial`. Despite its historical name, this state starts the RL runner immediately and updates the torque-task target every cycle.

```text
NewRLQPController_Initial
  start: RLStateRunner::start
  run:   RLStateRunner::run -> TorqueJointTask::setPosTarget(q_rl)
```

There is no separate non-RL posture state.

## Main controller behavior

The template:

- creates a `TorqueJointTask`;
- replaces the default dynamics constraint with configured CBF-related parameters;
- uses `ClosedLoopIntegrateReal` in `run()`;
- supports QP enforcement or direct torque bypass;
- obtains canonical joint order from `robot().refJointOrder()`;
- exposes policy selection, gain scaling, command inputs, logs, and limit warnings.

## Deliberate placeholders

The repository marks adaptation points with `TODO(robot)` and `TODO(policy)`. Typical required changes include controller names, robot module, command interface, base body, observers, constraints, CBF gains, joint groups, gains, default pose, and sensor names.

## Example policies

- `minimalExample` shows the smallest configuration accepted by the current parser.
- `fullExample` documents every currently accepted policy and observation field.

The placeholder joints and zero-byte placeholder ONNX files are intentional. The examples document configuration format, not a runnable robot.

## Not provided generically

The template does not provide a stable startup posture, position-to-torque transition, contacts, estimator integration, validated gains, or a robot-independent torque command interface.
