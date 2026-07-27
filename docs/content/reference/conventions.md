---
title: Training conventions
---

# Training conventions

A convention describes how controller-side data is translated into the representation expected by a training environment.

It can define:

- the policy-facing joint order;
- named joint subsets used by observations and actions;
- aliases between training-environment observation names and registered controller observation types.

Set the convention in `observations.yaml`:

```yaml
training_convention: mc_rtc
```

A convention changes representation, not semantics. Always compare it with the actual training configuration rather than relying only on the convention name.

## File structure

| Path | Required | Purpose |
|---|---:|---|
| `conventions` | Yes | Root map containing named convention profiles. |
| `conventions.<name>.joint_groups` | Yes | Joint groups used by action mapping and joint-based observations. |
| `conventions.<name>.type_aliases` | No | Maps training-environment observation names to internal observation types. |
| `conventions.<name>.observation_defaults` | No | Convention-specific default observation parameters. Applied before `observations.yaml`, which can override them. |
| `conventions.general.observation_defaults` | No | Defaults shared by all conventions, applied before convention-specific ones. |

## Complete example

```yaml
conventions:
  general:
    observation_defaults:
      base_orientation:
        sensor: Accelerometer
        scale: 1.0
        ...
  
  mjlab_exemple:
    joint_groups:
      all: [hip_yaw, hip_roll, hip_pitch, knee, ankle_pitch, ankle_roll]
      legs: [hip_yaw, hip_roll, hip_pitch, knee, ankle_pitch, ankle_roll]
      arms: [shoulder, elbow]
      ...
    
    type_aliases:
      joint_pos_rel: joint_pos
      joint_vel_rel: joint_vel
      generated_commands: command
      ...
    
    observation_defaults:
      joint_pos:
        joints: all
        relative_to_default_pose: true
      joint_vel:
        joints: all
        relative_to_default_velocity: true
      ...
  ...
```

## joint_groups

Named joint groups define subsets and their order. They are used by:

- `action.joints` and `action.controlled_joints` in `policy.yaml`
- `parameters.joints` in observation entries in `observations.yaml`

**The joint order must match your training environment exactly.** Reordering or omitting joints will silently produce incorrect behavior.

At minimum, define `all` to represent the complete actuated joint set in training order.

```yaml
conventions:
  my_convention:
    joint_groups:
      all: [hip_yaw, hip_roll, hip_pitch, knee, ankle_pitch, ankle_roll]
      legs: [hip_yaw, hip_roll, hip_pitch, knee, ankle_pitch, ankle_roll]
      upper_body: [shoulder_r, elbow_r, shoulder_l, elbow_l]
```

## type_aliases

Training environments often name observations differently from the controller's internal types. `type_aliases` lets you use training names directly in `observations.yaml` without code changes.

For example, if your training environment calls sensor observation `builtin_sensor`:

```yaml
type_aliases:
  builtin_sensor: force_sensor
```

Then in `observations.yaml`, you can write `type: builtin_sensor` exactly as in training, and it will resolve to the internal `force_sensor` observation.

This keeps policy files readable and prevents transcription errors when migrating between environments.

See [Available observations](observations.md) for details on each type.

## observation_defaults

Convention defaults let you specify parameters that apply to most or all policies using that convention, avoiding repetition in every `observations.yaml`.

### General defaults

Defaults under `conventions.general.observation_defaults` apply to all policies using any convention:

```yaml
conventions:
  general:
    observation_defaults:
      projected_gravity:
        scale: 1.0
      base_orientation:
        sensor: Accelerometer
```

### Convention-specific defaults

Defaults under `conventions.<name>.observation_defaults` apply only to policies using that convention:

```yaml
conventions:
  mjlab:
    observation_defaults:
      joint_pos:
        joints: all
        relative_to_default_pose: true
        scale: 1.0
      joint_vel:
        joints: all
        relative_to_default_velocity: true
```

### Override order

1. General defaults are merged first
2. Convention-specific defaults are merged second
3. Local parameters in `observations.yaml` override all of the above

This means a policy's explicit parameters always take precedence.
