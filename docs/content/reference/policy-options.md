---
title: Policy options
---

# Policy options

Each policy directory contains a `policy.yaml`. The current parser expects `action` and `control` sections and expands joint-indexed values into full mc_rtc reference-joint order.

## Complete structure

```yaml
name: walking_policy
onnx: walking_policy.onnx

action:
  joints: all_no_grippers
  controlled_joints: legs
  scale: 1.0
  q0:
    joint_a: 0.0
    joint_b: 0.4

control:
  use_QP: true
  policy_step_size: 0.02
  phase_period: 0.8333333333
  kp_scale: 1.0
  kd_scale: 1.0
  kp:
    joint_a: 100.0
    joint_b: 120.0
  kd:
    joint_a: 10.0
    joint_b: 12.0
```

## Top-level options

| Option | Required | Behavior |
|---|---:|---|
| `name` | No | Defaults to the policy folder name. Must be unique across loaded policies. |
| `onnx` | No | Defaults to `<name>.onnx`. Resolved relative to the policy folder. |

## `action`

| Option | Required | Behavior |
|---|---:|---|
| `joints` | Yes | Convention joint group defining ONNX output size and order. |
| `controlled_joints` | No | Subset actually applied to `q_rl`; defaults to `joints`. |
| `scale` | No | Scalar applied to every controller joint, or a per-joint map. Default scalar: `1.0`. |
| `q0` | No | Full per-joint default pose map. When present, it must cover every controller reference joint. |

!!! warning "Per-joint maps are expanded in mc_rtc order"
    `kp`, `kd`, action scales, and `q0` are stored in full controller order, even when the ONNX action covers only a subset.

## `control`

| Option | Default | Behavior |
|---|---:|---|
| `use_QP` | `true` | Initial QP enforcement mode. |
| `policy_step_size` | `0.02` s | ONNX inference period. Must be positive. |
| `kp_scale` | `1.0` | Initial stiffness multiplier. Must be positive. |
| `kd_scale` | `1.0` | Initial damping multiplier. Must be positive. |
| `phase_period` | `1.0` s | Period used by the `phase` observation. Must be positive. |
| `kp` | — | Per-joint stiffness map. Current parser requires all controller joints. |
| `kd` | — | Per-joint damping map. Current parser requires all controller joints. |

The parser retains fallback compatibility with older top-level keys such as `use_QP`, `policy_step_size`, and `pd_gains_ratio`, but new policies should use the structured sections above.

!!! info "`period_ms` versus current parser"
    The provided HRP5P example file contains `period_ms: 20.0`, but the current parser reads `control.policy_step_size` in seconds and defaults it to `0.02`. To avoid ambiguity, write `policy_step_size: 0.02` explicitly in new policy files.

## Validation rules

Policy loading fails when:

- the name, folder, ONNX path, or `action.joints` is empty;
- `policy_step_size`, `kp_scale`, or `kd_scale` is non-positive;
- `kp` or `kd` does not cover all reference joints;
- a provided `q0` does not cover all reference joints;
- `controlled_joints` contains a joint absent from `action.joints`;
- the resolved action-joint count differs from the ONNX output size.
