---
title: Troubleshooting
---

# Troubleshooting

## Policy folder is ignored

A discovered policy folder must contain both YAML files and at least one `.onnx` file. Confirm the folder is directly under `policies_root` and readable by the running controller.

## Unknown policy or duplicate name

Policy names come from `policy.yaml/name` or, by default, the folder name. Names must be unique across all loaded folders, and `default_policy` must match one of them.

## Observation size mismatch

Check, in order:

1. YAML observation list order;
2. each term's base size;
3. history values;
4. joint-group sizes;
5. `command.size` and `base_orientation.index`;
6. history concatenation order;
7. aliases and convention defaults.

The runtime checks dimensions but not semantic equivalence with training.

## Encoder size error

`joint_pos` and `joint_vel` accept either:

- a complete encoder vector whose size equals `refJointOrder()`; or
- an empty encoder vector, in which case they fall back to `mbc().q` or `mbc().alpha`.

A non-empty partial vector causes an error. Configure the observer pipeline so the selected observation robot exposes either a complete vector or no vector.

## ONNX action size mismatch

The ONNX output size must equal `action.joints`, not `controlled_joints`. Use `controlled_joints` only to choose which output components affect `q_rl`.

## Robot moves immediately on RL transition

Inspect:

- current posture versus `q_zero`;
- action scale and raw action;
- joint mapping order;
- initial history contents;
- policy convention;
- gain values and gain ratio;
- phase reset and command values.

A zero raw action still commands `q_zero`, so a mismatched default posture can cause an abrupt target change.

## Body or sensor does not exist

Confirm the exact mc_rtc model names. Distinguish robot bodies, body sensors, and force sensors; they are resolved through different APIs.

## Excessive terminal output

The latest HRP5P source contains unconditional warning-level dumps in the policy loop. Remove or guard them and use mc_rtc logger plots for high-frequency vectors.

## QP solver fails

First prove that observation and action pipelines are correct. Then inspect:

- transition posture discontinuity;
- task gains and weights;
- contact state and surfaces;
- collision and compound-joint constraints;
- position, velocity, and torque limits;
- direct versus QP torque behavior;
- robot-specific force/contact logic.

!!! tip "Read robot-specific information first"
    Many failures that appear to originate in RL are actually caused by assumptions in sensor selection, contact logic, robot joint order, or task configuration.
