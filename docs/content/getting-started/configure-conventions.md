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
| `conventions.<name>.observation_defaults` | No | Convention-specific default observation parameters. |
| `conventions.general.observation_defaults` | No | Defaults shared by all conventions. |

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
- [ ] Aliases map to registered internal observation types.
- [ ] Defaults do not silently override values you expect to define in `observations.yaml`.
- [ ] The `training_convention` used by each policy exists in this file.
