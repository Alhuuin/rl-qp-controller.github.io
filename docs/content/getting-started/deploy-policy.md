---
title: Deploy a policy
---

# Deploy a policy

<div class="quick-path" markdown>
**Fast path:** when the policy uses existing observations and an existing convention, deployment requires only a new folder containing an ONNX model and two YAML files.
</div>

## 1. Create the policy folder

By default, policies are discovered under the controller's `policies_root` directory.

```text
policies/
└── my_policy/
    ├── my_policy.onnx
    ├── policy.yaml
    └── observations.yaml
```

All three file types are required for the folder to be discovered. The ONNX filename itself is configurable.

## 2. Describe the action and control

```yaml
name: my_policy
onnx: my_policy.onnx

action:
  joints: all_no_grippers
  controlled_joints: all_no_grippers
  scale: 1.0

control:
  use_QP: true
  policy_step_size: 0.02
  phase_period: 0.8333333333
  kp_scale: 1.0
  kd_scale: 1.0
  kp: { ...all controller joints... }
  kd: { ...all controller joints... }
```

Add a full `q0` map when the policy was trained around a reference posture that differs from the controller posture task.

## 3. Reproduce the training input

```yaml
training_convention: mjlab
history_order_newest_first: false

observations:
  - type: base_lin_vel
    parameters: {history: 5}
  - type: base_ang_vel
    parameters: {history: 5}
  - type: projected_gravity
    parameters: {history: 5}
  - type: joint_pos_rel
    parameters:
      joints: all_no_grippers
      history: 5
  - type: joint_vel_rel
    parameters:
      joints: all_no_grippers
      history: 5
  - type: last_action
    parameters:
      joints: all_no_grippers
      history: 5
  - type: generated_commands
    parameters: {history: 5}
```

The exact term order, joint order, scaling, history length, and temporal order must match training.

## 4. Select the policy

Set the initial policy in the controller configuration:

```yaml
policies_root: /absolute/path/to/policies
default_policy: my_policy
```

All other valid policy folders remain available through the controller GUI.

## 5. Validate before motion

Use the following checklist:

- [ ] The model has exactly one input and one output.
- [ ] The ONNX input size equals the concatenated observation size.
- [ ] The ONNX output size equals the resolved `action.joints` size.
- [ ] Convention joint groups match the training order.
- [ ] History order matches training.
- [ ] `kp`, `kd`, and optional `q0` cover all controller joints.
- [ ] The selected observation source contains valid encoders and sensors.
- [ ] The robot starts near `q0` before entering the RL state.

!!! danger "Do not use QP bypass as the first real-robot test"
    QP bypass is useful for controlled simulation diagnosis, but it removes the controller's QP constraint path. Validate observations and action mappings in simulation and inspect logs before real-robot activation.
