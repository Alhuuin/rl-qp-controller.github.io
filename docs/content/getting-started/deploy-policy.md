---
title: Deploy a policy
---

# Deploy a policy

<div class="quick-path" markdown>
Deploy in this order: `conventions.yaml` -> `policy.yaml` -> `observations.yaml`.
This avoids most ordering/alias mismatches.
</div>

## 1. Prepare conventions first

Before filling policy files, ensure your training convention exists and is correct.

- It defines joint order for actions and joint-based observations.
- It defines aliases and optional defaults for observation entries.

See [Configure conventions.yaml](configure-conventions.md) for the full template and checks.

!!! warning "Reused conventions may not match your setup"
    Always verify aliases and defaults when reusing an existing convention.

## 2. Create the policy folder

By default, policies are searched under the controller's `policies_root` directory. By default, it is the `policies` directory of the controller.

```text
policies/
├── conventions.yaml
└── my_policy/
    ├── my_policy.onnx
    ├── policy.yaml
    └── observations.yaml
```

`conventions.yaml` is shared at the policies root. Inside each policy folder, `onnx`, `policy.yaml`, and `observations.yaml` are required.

## 3. Fill policy.yaml (action and control)

Use a short baseline first:

```yaml
name: my_policy
onnx: my_policy.onnx

action:
  joints: all
  controlled_joints: all
  scale: 1.0

control:
  use_QP: true
  period_s: 0.02
  pd_gains_ratio: 1.0
  phase_period: 1.0
  kp:
    jointName1: 100.0
    jointName2: 100.0
    jointName3: 100.0
  kd:
    jointName1: 10.0
    jointName2: 10.0
    jointName3: 10.0
```

Add `action.q0` only when your trained policy expects a different default pose than the controller posture target.

### policy.yaml options

| Entry | Required | Default | Description |
|---|---:|---|---|
| `name` | No | policy folder name | Runtime policy name. |
| `onnx` | No | `<name>.onnx` | ONNX model file in the policy folder. |
| `action.joints` | Yes | - | Convention joint group defining ONNX output order and size. |
| `action.controlled_joints` | No | `action.joints` | Optional subset actually applied to the robot. |
| `action.scale` | No | `1.0` | Scalar or per-joint scale for action output. |
| `action.q0` | No | controller/default posture | Optional full per-joint default pose map. |
| `control.use_QP` | No | `true` | Enable/disable QP enforcement. |
| `control.period_s` | No | `0.02` | Policy inference period in seconds. |
| `control.pd_gains_ratio` | No | `1.0` | Runtime gain multiplier. |
| `control.phase_period` | No | `1.0` | Phase period, used by `phase` observation. |
| `control.kp` | Yes | - | Per-joint stiffness map (full controller joint coverage). |
| `control.kd` | Yes | - | Per-joint damping map (full controller joint coverage). |

For complete parser behavior details, see [Policy options](../reference/policy-options.md).

## 4. Fill observations.yaml (training input)

Use a compact baseline:

```yaml
training_convention: mjlab
history_order_newest_first: true

observations:
  - type: joint_pos
    name: joint_positions
    parameters:
      history: 1
      joints: all
      relative_to_default_pose: true
      scale: 1.0

  - type: joint_vel
    parameters:
      history: 1
      joints: all
      relative_to_default_velocity: true
      scale: 1.0

  - type: projected_gravity
    parameters:
      history: 1
      body: pelvis
      scale: [1.0, 1.0, 1.0]
```

!!! danger "The observations in this file should reflect exactly the expected policy observations"
    The exact same term order, joint order, scaling, and history lengths must match training.

!!! warning "Convention defaults silently affect your observations"
    Before finalising `observations.yaml`, check `observation_defaults` in both `conventions.general` and your selected convention profile. These defaults are merged into each observation entry before your local parameters, and omitting a parameter does not mean it is absent — it may come from the convention. Override any default that does not match training.

### observations.yaml options

| Entry | Required | Default | Description |
|---|---:|---|---|
| `training_convention` | Yes | - | Convention profile name from `conventions.yaml`. |
| `history_order_newest_first` | No | `true` | Concatenation order inside each observation history buffer. |
| `observations` | Yes | - | Ordered list of observation terms used as ONNX input. |

### Basic Per-observation fields

| Entry | Required | Default | Description |
|---|---:|---|---|
| `type` | Yes | - | Internal observation name or alias resolved by the selected convention. |
| `name` | No | same as `type` | Optional debug/log label. |
| `parameters` | No | `{}` | Observation-specific parameters. |
| `parameters.history` | No | `1` | Number of samples stored for that term. |
| `parameters.scale` | No | `1.0` | Scalar or vector scale of the observation term. |

For all supported observation types and parameters, see [Available observations](../reference/observations.md).
    

## 5. Select the policy

Set the initial policy in the controller configuration:

```yaml
policies_root: /absolute/path/to/policies
default_policy: my_policy
```

All other valid policy folders remain available through the controller GUI.

## 6. Validation checklist

To avoid potential issues, use the following checklist before testing your policy:

- [ ] Check the size of the input and output of the policy. Use the provided `inspect_policy` script if needed
- [ ] The ONNX input size must equal the concatenated observation size.
- [ ] The selected `training_convention` exists in `conventions.yaml` and its aliases/defaults are the ones you expect.
- [ ] Convention joint groups match the training order exactly.
- [ ] The ONNX output size equals the resolved `action.joints` size.
- [ ] History order matches training.
- [ ] `kp`, `kd`, and optional `q0` cover all controller joints.
- [ ] The selected observation source contains valid encoders and sensors for the chosen robot.
