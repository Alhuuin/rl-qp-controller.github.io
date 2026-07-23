---
title: Policy options
---

# Policy options

This page documents fields read by the current policy parser.

## Minimal accepted `policy.yaml`

```yaml
action:
  joints: all

control:
  kp:
    jointName1: 100.0
    jointName2: 100.0
  kd:
    jointName1: 10.0
    jointName2: 10.0
```

`action.joints`, `control.kp`, and `control.kd` are the essential policy declarations. Gain maps must cover the controller joints required by the runtime.

## Complete structure

```yaml
name: my_policy
onnx: my_policy.onnx
use_QP: true  # legacy top-level fallback

action:
  joints: all
  controlled_joints: legs
  scale: 0.25
  q0:
    jointName1: 0.0
    jointName2: 0.0

control:
  use_QP: true
  period_s: 0.02
  # frequency_hz: 50.0
  pd_gains_ratio: 1.0
  phase_period: 1.0
  kp:
    jointName1: 100.0
    jointName2: 100.0
  kd:
    jointName1: 10.0
    jointName2: 10.0
```

## Top-level fields

| Field | Required | Default | Description |
|---|---:|---|---|
| `name` | No | folder name | Runtime name. |
| `onnx` | No | `<name>.onnx` | ONNX filename. |
| `use_QP` | No | `true` | Legacy fallback when `control.use_QP` is absent. |

## `action`

| Field | Required | Default | Description |
|---|---:|---|---|
| `joints` | Yes | — | Convention group defining ONNX output order and size. |
| `controlled_joints` | No | `joints` | Applied subset; must be contained in `joints`. |
| `scale` | No | `1.0` | Action scale accepted by the runtime. |
| `q0` | No | controller/default posture | Full default-pose map when provided. |

The ONNX output size is checked against `action.joints`, not `controlled_joints`.

## `control`

| Field | Required | Default | Description |
|---|---:|---|---|
| `use_QP` | No | top-level value, then `true` | QP path or direct bypass. |
| `period_s` | No | `0.02` | Inference period in seconds. |
| `frequency_hz` | No | — | Alternative to `period_s`. |
| `pd_gains_ratio` | No | `1.0` | Runtime gain scaling. |
| `phase_period` | No | observation value, then `1.0` | Runtime phase period. |
| `kp` | Yes | — | Base stiffness map. |
| `kd` | Yes | — | Base damping map. |

Specify only one of `period_s` and `frequency_hz`.

## Phase precedence

The runtime first reads top-level `phase_period` from `observations.yaml`, defaulting to `1.0`. `control.phase_period` overrides it.

## Validation failures

Loading fails for unresolved paths/names, missing or empty `action.joints`, simultaneous period and frequency, non-positive timing or gain ratio, incomplete gain or provided `q0` maps, invalid controlled subset, or ONNX output mismatch.
