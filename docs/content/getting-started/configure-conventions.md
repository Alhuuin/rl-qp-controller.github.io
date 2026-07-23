---
title: Configure conventions.yaml
---

# Configure conventions.yaml

<div class="quick-path" markdown>
Configure `conventions.yaml` before `policy.yaml` and `observations.yaml`.
It defines the joint ordering and optional aliases used to interpret both policy actions and observations.
</div>

## Why this file comes first

`training_convention` in `observations.yaml` selects one convention profile from `conventions.yaml`.

That profile controls:

- joint groups and their order (`all`, `legs`, `arms`, custom subsets...);
- aliases from training-environment names to controller observation types;
- optional default parameters for observations.

If this file is wrong, the policy can still run but receive the wrong data ordering or wrong defaults.

!!! warning "Existing conventions are not always interchangeable"
    If you reuse an existing convention, verify all aliases and defaults carefully.
    Two conventions with similar names may still differ in joint order, aliases, or parameter defaults.

## Location

In this project structure, conventions are typically defined in `policies/conventions.yaml`.

```text
policies/
├── conventions.yaml
└── my_policy/
    ├── my_policy.onnx
    ├── policy.yaml
    └── observations.yaml
```

## Quick start (recommended)

```yaml
conventions:
  mjlab:
    joint_groups:
      all: [hip_yaw, hip_roll, hip_pitch, knee, ankle_pitch, ankle_roll]
      legs: [hip_yaw, hip_roll, hip_pitch, knee, ankle_pitch, ankle_roll]
    observation_defaults:
      joint_pos:
        joints: all
        relative_to_default_pose: true
```

This is usually enough to start: one convention, at least one joint group, and only the aliases/defaults you actually need.

## File structure reference

| Path | Required | Purpose |
|---|---:|---|
| `conventions` | Yes | Root map containing named convention profiles. |
| `conventions.<name>.joint_groups` | Yes | Joint groups used by action mapping and joint-based observations. |
| `conventions.<name>.type_aliases` | No | Maps training-environment observation names to internal observation types. |
| `conventions.<name>.observation_defaults` | No | Convention-specific default observation parameters. Applied before `observations.yaml`, which can override them. |
| `conventions.general.observation_defaults` | No | Defaults shared by all conventions, applied before convention-specific ones. |

### type_aliases

Training environments often give observations different names from the controller's internal types. `type_aliases` lets you use those exact training names in `observations.yaml` without any code change: an alias entry maps the training-side name to the matching internal observation type.

For example, if your training environment called joint positions `joint_pos_rel`, you can write:

```yaml
type_aliases:
  joint_pos_rel: joint_pos
```

and then use `type: joint_pos_rel` in `observations.yaml` exactly as you would have in the training config, keeping the integration seamless and the policy files readable.

### observation_defaults

If you know a set of parameters will be the same for most policies using a convention — for example, `joint_pos` always using all joints and being relative to the default pose — you can specify them directly in `conventions.yaml` instead of repeating them in every `observations.yaml`.

These defaults are applied first. Any parameters you specify in a policy's own `observations.yaml` will override them, so adding a policy that deviates from the convention defaults is fine: just set the needed parameters locally.

!!! warning "Defaults can produce unexpected behavior if overlooked"
    Convention defaults are easy to forget when deploying a new policy against an existing convention.
    Always check `observation_defaults` — both in `conventions.<name>` and in `conventions.general` — before assuming an observation uses only the parameters written in `observations.yaml`.

## Build it in 3 steps

1. Create the convention name you will reference in `observations.yaml` (`training_convention: <name>`).
2. Define `joint_groups` with exact training/export order.
3. Add only needed `type_aliases` and `observation_defaults`.

??? info "Optional snippets"
    ### Shared defaults for every convention

    ```yaml
    conventions:
      general:
        observation_defaults:
          base_orientation:
            sensor: Accelerometer
            scale: 1.0
    ```

    ### Convention-specific aliases and defaults

    ```yaml
    conventions:
      isaaclab:
        type_aliases:
          joint_pos_rel: joint_pos
          generated_commands: command
        observation_defaults:
          joint_pos:
            joints: all
            relative_to_default_pose: true
    ```

## Validation checklist

- [ ] Every joint group order matches training/export order exactly.
- [ ] The `training_convention` referenced by each policy exists in this file.
- [ ] All `type_aliases` values are registered internal observation types (check the [observations reference](../reference/observations.md)).
- [ ] Every observation name used in `observations.yaml` resolves either directly or through an alias.
- [ ] `observation_defaults` — both in `conventions.general` and the convention profile — are what you intend for each policy using this convention.
