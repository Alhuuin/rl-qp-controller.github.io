---
title: Configure conventions.yaml
---

# Configure conventions.yaml

<div class="quick-path" markdown>
Configure `conventions.yaml` before `policy.yaml` and `observations.yaml`.
It defines the joint ordering and optional aliases used to interpret both policy actions and observations.
</div>

## Location

In this project structure, conventions are typically defined in `policies/conventions.yaml`:

```text
policies/
├── conventions.yaml
└── my_policy/
    ├── my_policy.onnx
    ├── policy.yaml
    └── observations.yaml
```

## Minimal example

```yaml
conventions:
  my_convention:
    joint_groups:
      all: [hip_yaw, hip_roll, hip_pitch, knee, ankle_pitch, ankle_roll]
      legs: [hip_yaw, hip_roll, hip_pitch, knee, ankle_pitch, ankle_roll]
```

This is the minimum required: one convention name, at least one joint group (typically `all`), with joint order matching your training exactly.

## What to do next

- **`joint_groups`**: List the exact joint order from your training environment. The order must match precisely.
- **`type_aliases`** (optional): Map training-environment observation names to controller observation types. See [conventions reference](../reference/conventions.md#type_aliases).
- **`observation_defaults`** (optional): Set default parameters for observations to avoid repetition in `observations.yaml`. See [conventions reference](../reference/conventions.md#observation_defaults).

!!! warning "Defaults can produce unexpected behavior if overlooked"
    Convention defaults are easy to forget when deploying a new policy against an existing
 convention.
    Always check `observation_defaults` — both in `conventions.<name>` and in `conventions
.general` — before assuming an observation uses only the parameters written in `observation
s.yaml`.


## Full reference

For complete structure documentation, all configuration options, and validation details, see [Training conventions reference](../reference/conventions.md).
