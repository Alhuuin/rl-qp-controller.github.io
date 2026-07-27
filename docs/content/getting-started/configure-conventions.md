---
title: Configure conventions.yaml
---

# Configure conventions.yaml

This file defines training-environment conventions (joint groups and order, eventual type aliases and default observation parameters). This enables the use of policies trained in different conditions in the same environment.

It should be checked and configured if informations are missing, before the `policy.yaml` and `observations.yaml` of a new policy.

## Location

In this project structure, conventions are typically defined in `<policy_root>/conventions.yaml`:

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
      all: [hip_yaw, hip_roll, hip_pitch, knee, ankle_pitch, ankle_roll, ...]
      legs: [hip_yaw, hip_roll, hip_pitch, knee, ankle_pitch, ankle_roll]
```

This is the minimum required: one convention name, at least one joint group (typically `all`), with joint order matching your training exactly.

Joint groups can be used to specify either the actions joints or the joints to consider for a specific observation.

## Optional

- **`type_aliases`** : to make the integration seamless, it is possible to map the names of the observations as the controllers knows them to the name they had in the training environment. Mostly useful for conventions that will be heavily used. See [type aliases](../reference/conventions.md#type_aliases) for more informations.
- **`observation_defaults`** : Set default parameters for observations to avoid repetition in `observations.yaml`. See [observation defaults](../reference/conventions.md#observation_defaults) for more informations.

!!! warning "Defaults can produce unexpected behavior if overlooked"
    Convention defaults are easy to forget when deploying a new policy against an existing convention.  
    Always check `observation_defaults` — both in `conventions.<name>` and in `conventions.general` — before assuming an observation uses only the parameters written in `observations.yaml`.

For complete structure documentation, all configuration options, and validation details, see [Training conventions reference](../reference/conventions.md).

## Validation checklist

- [ ] Every `joint_groups` entry order matches training/export order exactly.
- [ ] The `training_convention` referenced by each policy exists in this file.
- [ ] All `type_aliases` values are registered internal observation types (see [Available observations](../reference/observations.md)).
- [ ] Every observation name used in `observations.yaml` resolves either directly or through an alias.
- [ ] `observation_defaults` — both in `conventions.general` and the convention profile — match your training setup.
- [ ] Convention-specific defaults can be overridden in policy-local `observations.yaml` parameters when needed.
