---
title: Architecture
---

# Architecture

The controller separates the mc_rtc integration from the policy-dependent runtime code, while keeping the pieces that must access robot state connected through `NewRLQPController`.

At the top level, the template contains:

- the **mc_rtc controller and FSM state**;
- the **policy runtime**, which owns policy loading, observations, ONNX inference, action mapping, and policy-dependent gains;
- the **observation implementations** and their registry;
- the **policy folders and training conventions**.

## Project structure

The source tree of the base template is organized as follows:

```text
new-rl-qp-controller/
├── etc/
│   └── NewRLQPController.in.yaml
│
├── include/
│   ├── NewRLQPController.h
│   ├── api.h
│   ├── observation/
│   │   ├── Observation.h
│   │   ├── ObservationManager.h
│   │   └── Observations.h
│   └── policy/
│       ├── PolicyConfig.h
│       ├── RLPolicyInterface.h
│       ├── RLPolicyRuntime.h
│       └── RLStateRunner.h
│
├── src/
│   ├── NewRLQPController.cpp
│   ├── lib.cpp
│   ├── observation/
│   │   ├── Observation.cpp
│   │   ├── ObservationManager.cpp
│   │   └── Observations.cpp
│   ├── policy/
│   │   ├── PolicyConfig.cpp
│   │   ├── RLPolicyInterface.cpp
│   │   ├── RLPolicyRuntime.cpp
│   │   └── RLStateRunner.cpp
│   └── states/
│       ├── CMakeLists.txt
│       ├── NewRLQPController_Initial.cpp
│       └── NewRLQPController_Initial.h
│
├── policies/
│   ├── conventions.yaml
│   ├── minimalExample/
│   └── fullExample/
│
├── scripts/
│   └── inspect_policy.py
│
├── tests/
└── ext/onnxruntime
```

## Main objects and ownership

The actual ownership relationships in the template are:

```text
NewRLQPController
│
├── TorqueJointTask
│
├── RLStateRunner
│
└── RLPolicyRuntime
    │
    ├── PolicyManager
    │   └── PolicyConfig(s)
    │
    ├── ObservationRegistry
    │
    ├── ObservationManager
    │   └── Observation instances
    │
    └── RLPolicyInterface
        └── ONNX Runtime session
```

The runtime is generic with respect to a particular robot model, but it is not isolated from mc_rtc: it receives `NewRLQPController` when it needs the controller joint order, the selected robot state, the posture task, or the `TorqueJointTask`.

## Main components

### NewRLQPController

`NewRLQPController` is the mc_rtc controller implementation.

In the base template it:

- configures the dynamics and self-collision constraints;
- removes the default posture task from the solver;
- creates and adds a `TorqueJointTask`;
- stores `robot().refJointOrder()` as the controller joint order;
- initializes the policy runtime;
- exposes GUI and logging entries;
- runs the FSM controller with `ClosedLoopIntegrateReal` feedback;
- optionally bypasses the QP and writes the RL PD torques directly to `robot().mbc().jointTorque`;
- reports measured position, velocity, and torque limit violations.

Robot-specific adaptations can change these responsibilities when the target robot requires different tasks, constraints, observers, contacts, or mode-switching logic.

### PolicyManager and PolicyConfig

`PolicyManager` is declared alongside `PolicyConfig` in `PolicyConfig.h` and is owned by `RLPolicyRuntime`.

It:

- scans `policies_root` for subdirectories containing `policy.yaml` and `observations.yaml`;
- also accepts the current working directory when it contains `policy.yaml`, `observations.yaml`, and an `.onnx` file;
- loads each policy through `PolicyConfig::load()`;
- rejects duplicate policy names;
- selects the configured default policy;
- provides policy selection and cycling at runtime.

`PolicyConfig` stores the parsed paths, action selectors, control options, gains, action scaling, default posture, and the raw policy/observation YAML configurations.

The ONNX model itself is loaded later by `RLPolicyInterface` when the selected policy is activated.

### RLPolicyRuntime

`RLPolicyRuntime` owns the active policy session.

It is responsible for:

- loading and switching policies;
- resolving the training convention;
- resolving action and controlled-joint mappings into `robot().refJointOrder()` indices;
- configuring gains, action scaling, `q_zero`, timing, and phase;
- owning the observation registry and manager;
- selecting the robot state used for observations;
- owning the ONNX inference interface;
- computing observations at the policy update rate;
- running inference;
- converting policy outputs into `q_rl`;
- validating observation and action dimensions.

Full-size runtime vectors such as gains, `q_zero`, `q_rl`, and action scaling are stored in controller joint order.

### ObservationRegistry

`ObservationRegistry` maps an internal observation type name to the factory that creates the corresponding `Observation` subclass.

The runtime initializes it with `makeDefaultObservationRegistry()`. `ObservationManager` then uses that registry when loading the active `observations.yaml`.

This is the extension point used when adding a new built-in observation implementation.

### ObservationManager

`ObservationManager` owns the active list of observation objects and their history buffers.

When a policy is loaded, it:

- reads `training_convention` and the `observations` list;
- resolves aliases through the active `ObservationConvention`;
- creates each observation through the registry;
- configures the observations from an `ObservationContext`;
- computes the final flattened observation size.

At runtime it computes each observation, updates its history buffer, and concatenates the histories in appropriate order into the vector passed to the ONNX policy.

### ObservationContext and ObservationConvention

`ObservationContext` is the data view passed to observation implementations. It contains:

- the selected observation robot;
- the configured base body;
- controller joint order;
- the policy-to-controller joint mapping;
- `q_zero`;
- the previous raw policy action;
- the high-level command;
- the current normalized phase;
- the active observation convention.

`ObservationConvention` loads convention-specific joint groups, aliases, and default observation parameters from `policies/conventions.yaml`.

### RLPolicyInterface

`RLPolicyInterface` is the ONNX Runtime wrapper.

It:

- creates and owns the ONNX Runtime environment and session;
- reads the model input and output dimensions;
- accepts the flattened Eigen observation vector;
- runs inference;
- returns the Eigen action vector.

It does not build observations or map the returned action to robot joints; those responsibilities remain in `RLPolicyRuntime`.

### RLStateRunner

`RLStateRunner` contains the policy-running logic shared by RL FSM states.

Its `start()` method:

- checks that a policy is loaded;
- adds state-level GUI information;
- resets the runtime, including phase, observation history, timer, and update count.

Its `run()` method calls `RLPolicyRuntime::runPolicyStepIfNeeded()` and catches runtime exceptions so they can be reported through mc_rtc logging.

The FSM state remains responsible for deciding how the runtime output is applied to the controller tasks.

## Base-template FSM state

The base template currently builds one state:

```text
NewRLQPController_Initial
```

Despite the historical name, it is already the RL-running state. It has the behavior previously described

## Runtime sequence

The controller and the policy do not necessarily run at the same frequency.

During an FSM cycle, the active state calls `RLStateRunner::run()`. The runtime advances its timer every controller step and only performs a new policy inference when the configured policy period has elapsed.

```text
NewRLQPController_Initial::run()
    │
    ├── RLStateRunner::run()
    │   └── RLPolicyRuntime::runPolicyStepIfNeeded()
    │       ├── advance policy timer and phase
    │       └── if policy period elapsed:
    │           ├── ObservationManager::compute()
    │           ├── validate ONNX input size
    │           ├── RLPolicyInterface::predict()
    │           ├── validate action size
    │           ├── scale/map action
    │           └── update q_rl
    │
    └── TorqueJointTask::setPosTarget(q_rl)
```

Between policy updates, `q_rl` remains unchanged and the state keeps assigning the existing target to the `TorqueJointTask` every controller cycle.

## Policy execution and action mapping

The ONNX output order is defined by `action.joints` in the policy's `policy.yaml`. The runtime resolves that selector through the active convention.

It is also possible to provide `action.controlled_joints` in the case the policy's output is a larger joint set than the one that should actually be applied on the robot. An example would be a policy that outputs all joints while only the legs should be used. We need to know both that what joints correpond to the output of the policy, and which joints should be used. `action.controlled_joints` should be a subset of `action.joints`.

At every policy update the runtime first resets:

```text
q_rl = q_zero
```

Then, for each ONNX action element:

```text
scaled_action[joint] = action_scale[joint] * action[action_index]
```

Then, the specified joints are applied:

```text
q_rl[joint] = q_zero[joint] + scaled_action[joint]
```

Consequently, controller joints that are not selected remain at their `q_zero` target.

## Observation source

The controller configuration selects which mc_rtc robot object is passed to the observation pipeline.

The selected object is stored in `ObservationContext::observationRobot` and is used by observation implementations when they read robot state.

---

## Default pose

`q_zero` is stored in controller joint order and is used as the reference posture for `q_rl`.

When `action.q0` is present in `policy.yaml`, the runtime loads that complete per-joint map.

When `action.q0` is absent, the runtime reads the controller's FSM posture task and extracts the default posture values into `q_zero`.

---

## QP and direct-torque paths

The policy always produces a position target `q_rl`.

In the base template, the active state sends this target to `TorqueJointTask` every controller cycle. The runtime also applies the policy gains to that task when a policy is loaded or the gain ratio changes.

When QP control is enabled, the `TorqueJointTask` remains part of the mc_rtc solver together with the configured dynamics and self-collision constraints.

When QP control is bypassed, `NewRLQPController::byPassQPControl()` computes, for every controller joint:

```text
tau = kp * (q_rl - q) - kd * q_dot
```

and writes the result directly to `robot().mbc().jointTorque` after the normal controller run.
