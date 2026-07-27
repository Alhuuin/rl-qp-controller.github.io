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

Use this basic template:

```yaml
name: my_policy
onnx: my_policy.onnx

action:
  joints: all
  scale: 1.0

control:
  use_QP: true
  period_s: 0.02
  kp:
    jointName1: 100.0
    jointName2: 100.0
    jointName3: 100.0
  kd:
    jointName1: 10.0
    jointName2: 10.0
    jointName3: 10.0
```

Adjust `kp` and `kd` to cover all controller joints. For all available options, see [Policy options reference](../reference/policy-options.md).

## 4. Fill observations.yaml (training input)

Use this basic template:

```yaml
training_convention: mjlab
history_order_newest_first: true

observations:
  - type: joint_pos
    parameters:
      history: 1
      joints: all

  - type: joint_vel
    parameters:
      history: 1
      joints: all

  - type: projected_gravity
    parameters:
      history: 1
```

!!! danger "Must match training exactly"
    The observation order, joint order, scales, and history lengths must exactly match your training configuration. Mismatches produce silent failures.

!!! warning "Convention defaults matter"
    Check `observation_defaults` in both `conventions.general` and your selected convention profile. Defaults are merged into each observation before your local parameters, so omitting a parameter does not mean it is absent.

For all available observation types and their parameters, see [Available observations reference](../reference/observations.md).
    

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
