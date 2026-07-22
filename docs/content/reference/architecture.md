---
title: Architecture
---

# Architecture

This page describes the architecture implemented by the current HRP5P controller. Class names and behavior are taken from the source code rather than from an abstract design proposal.

## Main components

<div class="component-grid" markdown>

<div class="component-card" markdown>
### `PolicyManager`
Discovers valid policy folders, parses each policy, checks duplicate names, and selects the default policy.
</div>

<div class="component-card" markdown>
### `RLPolicyRuntime`
Owns the active policy, convention, joint maps, observations, action buffers, gains, command, phase, and policy timer.
</div>

<div class="component-card" markdown>
### `ObservationManager`
Builds observations from YAML, maintains their history buffers, and concatenates the final ONNX input.
</div>

<div class="component-card" markdown>
### `RLPolicyInterface`
Loads one ONNX model, validates its input/output shape, and executes inference using ONNX Runtime.
</div>

<div class="component-card" markdown>
### `RLStateRunner`
Connects the runtime to mc_rtc FSM states through common `start`, `run`, and teardown behavior.
</div>

<div class="component-card" markdown>
### `HRP5pRLQPController`
Owns mc_rtc tasks, GUI and logger entries, robot-specific contacts, limits, observers, and torque application.
</div>

</div>

## Startup sequence

When the controller is constructed, the runtime performs the following sequence:

```text
RLPolicyRuntime::configure
├── copy controller configuration
├── read robot.base_body and robot.observation_source
├── store robot().refJointOrder()
├── create the default ObservationRegistry
├── PolicyManager::load(...)
└── loadPolicy(default_policy)
    ├── configureControl(...)
    ├── load training convention
    ├── configureAction(...)
    ├── configureNetwork(...)
    ├── configureObservations(...)
    ├── reset observation history
    └── validate observation size against ONNX input
```

A policy folder is considered loadable when it contains `policy.yaml`, `observations.yaml`, and an `.onnx` file. Policy folders are sorted alphabetically before loading.

!!! note "Current-folder policy mode"
    The policy manager also contains support for treating the current folder itself as a policy folder. In the standard installed setup, `policies_root` points to a directory containing policy subdirectories.

## Runtime loop

`RLStateRunner::run()` calls `RLPolicyRuntime::runPolicyStepIfNeeded()` every controller cycle. The runtime increments two independent timers:

- `policyTimer_`, which controls when ONNX inference runs;
- `phaseElapsedTime_`, which generates a normalized cyclic phase.

Inference only runs when `policyTimer_ >= policyStepSize_`. Between policy updates, the latest action and target are retained.

At each policy update:

1. `ObservationManager` computes and concatenates the observation vector.
2. The vector length is compared with the ONNX input size.
3. `RLPolicyInterface::predict()` produces the raw action.
4. The action length is compared with the resolved `action.joints` mapping.
5. Every output is scaled in controller joint order.
6. Only joints selected by `action.controlled_joints` are applied to `q_rl`.
7. Non-controlled joints remain at `q_zero`.

The resulting target follows:

```text
q_rl[j] = q_zero[j] + action_scale[j] * action[action_index]
```

for controlled joints only.

## Action mappings

The runtime keeps two separate mappings:

| Mapping | Purpose |
|---|---|
| `actionToControllerMap_` | Defines the exact ONNX output layout using `action.joints`. Its size must equal the ONNX action size. |
| `controlledActionControllerIndices_` | Defines which mapped outputs are actually applied. It must be a subset of `action.joints`. |

This separation supports policies that output a whole-body action while a controller experiment applies only a subset, such as the legs.

!!! warning "Do not confuse action layout and controlled subset"
    Changing `controlled_joints` does **not** change the ONNX output size or ordering. The ONNX output must still match `action.joints` exactly.

## Observation source

The controller accepts two robot-state sources:

| `robot.observation_source` | Runtime object |
|---|---|
| `realRobot` | `ctl.realRobot(robotName)`; this is the default |
| `robot` | `ctl.robot()` |

This choice is applied consistently through `ObservationContext`. An invalid value causes startup to fail.

## Default pose resolution

`q_zero` is stored in full mc_rtc reference-joint order.

- When `action.q0` is provided, it must contain every controller joint and is copied into `q_zero`.
- Otherwise, the runtime extracts the current mc_rtc posture-task target for revolute joints.

This makes the default posture task the fallback reference pose while allowing a policy-specific trained pose to override it.

## QP and direct torque paths

The controller owns a `mc_tasks::TorqueJointTask`. The runtime updates its stiffness and damping from the active policy gains. Depending on `use_QP`:

- **QP enforced:** the torque target is handled through the QP task and controller constraints;
- **QP bypassed:** the controller writes the computed torque command directly to the robot joints.

The precise safety constraints available depend on the robot controller configuration. In the HRP5P configuration, contact and compound-joint constraints are enabled, and collision constraints are configured separately.

## FSM integration

The shipped HRP5P configuration starts in `HRP5pRLQPController_Initial` and defines transitions between the initial and RL states. `RLStateRunner` adds state-specific GUI labels, resets the phase and history when an RL state starts, and catches runtime exceptions during the state loop.

!!! tip "Prefer an initial still state"
    The HRP5P adaptation deliberately starts in a posture-controlled initial state before switching to RL. This limits the discontinuity between robot initialization and the learned controller target.
