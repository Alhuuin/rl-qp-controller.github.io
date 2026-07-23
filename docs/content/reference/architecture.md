---
title: Architecture
---

# Architecture

This page presents the overall architecture of the RL-QP controller framework.

The goal of the project is to provide a reusable reinforcement-learning controller built on top of **mc_rtc**, while keeping the robot-specific implementation isolated from the generic RL infrastructure.

The architecture is therefore split into three layers:

- the **generic RL framework**, shared by every robot;
- the **robot adaptation**, which connects the framework to a specific robot;
- the **policy configuration**, which allows multiple trained policies to be loaded without modifying the controller.

---

# Project structure

The repository is organized into a few major components.

```text
├── include/
│   ├── controller/
│   ├── observations/
│   ├── policy/
│   ├── runtime/
│   ├── states/
│   └── utils/
│
├── src/
│   ├── controller/
│   ├── observations/
│   ├── policy/
│   ├── runtime/
│   ├── states/
│   └── utils/
│
├── etc/
│   ├── controller configuration
│   ├── FSM configuration
│   └── robot-specific configuration
│
├── policies/
│   ├── policy1/
│   ├── policy2/
│   └── ...
│
└── plugins/
```

The code can also be viewed as four interacting modules.

```text
             Policy folders
                    │
                    ▼
          PolicyManager
                    │
                    ▼
          RLPolicyRuntime
                    │
       ┌────────────┴────────────┐
       ▼                         ▼
ObservationManager      RLPolicyInterface
       │                         │
       └────────────┬────────────┘
                    ▼
             RLStateRunner
                    │
                    ▼
      Robot-specific controller
```

The generic framework never directly manipulates a specific robot. Instead, it produces desired joint targets that are consumed by the robot controller.

---

# Main components

## PolicyManager

Responsible for discovering available policies.

It:

- scans the policy directory;
- validates policy folders;
- parses `policy.yaml`;
- loads training conventions;
- prevents duplicate policy names;
- selects the default policy.

Once a policy has been selected, it is passed to the runtime.

---

## RLPolicyRuntime

`RLPolicyRuntime` is the central object of the framework.

It owns almost every piece of data required while the controller is running, including:

- the active policy;
- the loaded ONNX network;
- the observation manager;
- the action mappings;
- the current policy command;
- the policy timers;
- the phase generator;
- PD gains;
- action scaling;
- `q_zero`;
- `q_rl`;
- convention information.

Almost every other component communicates through the runtime.

---

## ObservationManager

The observation manager constructs the neural-network input.

It:

- creates observations from YAML;
- owns every observation object;
- maintains history buffers;
- concatenates the final observation vector;
- validates the observation size.

Individual observations are implemented as independent classes, making new observations easy to add without modifying the manager itself.

---

## RLPolicyInterface

The policy interface wraps ONNX Runtime.

Its responsibilities are to:

- load the ONNX model;
- validate network dimensions;
- execute inference;
- return the predicted action vector.

It deliberately knows nothing about robots or observations.

---

## RLStateRunner

`RLStateRunner` provides the bridge between the generic RL framework and the mc_rtc FSM.

It centralizes logic common to RL states, including:

- initialization;
- runtime execution;
- phase reset;
- history reset;
- GUI updates;
- exception handling.

Robot-specific states usually only contain the logic required to enter or leave an RL mode.

---

## Robot controller

The controller itself is the robot-dependent part of the project.

It is responsible for everything that depends on the robot implementation, for example:

- robot tasks;
- constraints;
- contact management;
- observers;
- GUI entries;
- logging;
- control mode;
- torque application;
- hardware interfaces.

The exact implementation differs between robot adaptations.

---

# Runtime sequence

Controller startup follows the sequence below.

```text
Controller
    │
    ▼
RLPolicyRuntime::configure()
    │
    ├── load controller configuration
    ├── create ObservationRegistry
    ├── discover available policies
    ├── load default policy
    ├── configure observations
    ├── configure actions
    ├── load ONNX network
    └── validate dimensions
```

After initialization, every controller cycle executes:

```text
mc_rtc
    │
    ▼
RLStateRunner
    │
    ▼
RLPolicyRuntime
    │
    ├── update timers
    ├── update observations
    ├── run ONNX if needed
    ├── compute q_rl
    └── expose targets
    │
    ▼
Robot controller
    │
    ├── QP control
    └── direct control (optional)
```

---

# Policy execution

Policy inference only runs at the configured policy frequency.

During intermediate controller cycles, the previously predicted action is reused.

When inference is performed, the runtime:

1. computes observations;
2. concatenates the observation vector;
3. validates its size;
4. executes the ONNX network;
5. validates the output size;
6. applies action scaling;
7. computes the target posture.

For each controlled joint:

```text
q_rl = q_zero + action_scale × action
```

while joints outside `controlled_joints` remain at their default position.

---

# Action mappings

The runtime maintains two different mappings.

| Mapping | Purpose |
|----------|---------|
| `actionToControllerMap_` | Defines how ONNX outputs correspond to controller joints. |
| `controlledActionControllerIndices_` | Defines which mapped outputs are actually applied. |

This separation allows a policy to predict a full-body action while the controller applies only part of it.

---

# Observation source

Observations can be built from either:

| Configuration | Source |
|--------------|--------|
| `realRobot` | Measured robot state |
| `robot` | Internal controller model |

This choice is propagated automatically through the observation context.

---

# Default pose

`q_zero` is the reference posture used to reconstruct policy targets.

It can either:

- be specified explicitly in the policy configuration;
- or be extracted from the current posture task.

The resulting target is then computed by adding the policy output to this reference posture.

---

# Torque generation

The runtime updates the active PD gains every time a policy is loaded.

Depending on the controller configuration, the resulting command can either:

- be sent through a `TorqueJointTask`, allowing the QP to enforce constraints;
- or be sent directly to the robot.

The choice is entirely robot dependent and is configured by each controller adaptation.