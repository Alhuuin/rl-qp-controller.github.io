---
title: Troubleshooting
---

# Troubleshooting

Troubleshoot in layers: discovery and dimensions, then observations and mappings, then tasks/QP, and finally robot-specific state/contact behavior.

## Controller or FSM state is not found

Check the exported controller name in `src/lib.cpp`, selection in `mc_rtc.yaml`, `EXPORT_SINGLE_STATE` names, `src/states/CMakeLists.txt`, and state search paths. A source file may exist without being built. See your robot-specific configuration for which states are enabled.

## Policy folder is ignored

A policy folder must be directly below `policies_root` and contain `policy.yaml`, `observations.yaml`, and the resolved ONNX file. Remember that `@CMAKE_CURRENT_SOURCE_DIR@` is substituted at configure time.

## Default policy is unknown

The runtime name is `policy.yaml/name`, or the folder name when omitted. Names must be unique and `default_policy` must match the runtime name.

## ONNX input size mismatch

Compute:

```text
sum(observation base size x history)
```

Then verify aliases, convention defaults, joint-group sizes, `base_orientation.index`, `command.size`, force-sensor count, history, and repeated terms.

## ONNX action size mismatch

The output size must equal resolved `action.joints`, not `controlled_joints`. Check the logged controller indices.

## Policy loads but behavior is wrong

Dimensions do not prove semantic equivalence. Compare observation order, joint order, frames, signs, scaling, relative/absolute values, history order, phase period/reset, command order, action scaling, and default pose with training.

## Encoder size error

Joint observations accept a complete encoder vector matching `refJointOrder()` or an empty vector that permits multibody fallback. A non-empty partial vector is rejected.

## Body, body sensor, or force sensor is missing

These are different resources and APIs. Confirm the exact robot-module names and whether the observation expects a body, body sensor, or force sensor.

## Base velocity uses the wrong source

Inspect the implementation in the selected repository. Adaptations may compute the same YAML type from different APIs. Verify that configurable names are actually used by `compute()`.

## Robot moves abruptly at policy start

Plot current position, `q_zero`, raw action, scaled action, `q_rl`, `kp`, and `kd`. A zero action still commands `q_zero`. Common causes are wrong default pose/order, excessive scale, stale history, nonzero command, or phase mismatch.

## Initial works but transition to RL fails

1. Verify the same policy works when RL starts directly.
2. Check task membership before and after transition.
3. Compare posture target with first `q_rl`.
4. Check when `ControlMode` changes relative to the solver cycle.
5. Check solver feedback after switching.
6. Verify a valid torque command exists on the first torque cycle.
7. Check estimator/contact changes performed on entry.
8. Check teardown removes only intended tasks.

A stable Initial state does not prove a valid position-to-torque handoff.

## Single-state controller falls immediately

Check startup torque mode, torque task membership, loaded/nonzero gains, policy load, dimensions, `q_zero`, observer validity, and first-cycle QP result.

## QP solver fails

Inspect target discontinuity, task weights/gains, joint bounds, collisions, compound joints, contacts/surfaces, external-force estimates, command mode, and solver feedback—in that order.

## QP bypass differs from QP mode

This is expected. Bypass writes PD torque directly and does not reproduce optimization, collision handling, constraints, or contact behavior. Use it only to isolate the policy/PD path.

## H1 Tilt observer fails

Verify `anchorFrameFunction` is registered before use, the datastore key matches, `createContactAnchor()` resolves required resources, and measured state is valid.

## External-force observer or datastore API is missing

Confirm the plugin is installed, syntax matches the installed version, expected datastore keys exist, and required force or motor-torque measurements are available. H1 and HRP5P use different estimation methods and torque sources.

## HRP5P foot contacts do not update

The helper exists, but its call is commented in the active Initial state. Also verify force-sensor and surface names, Schmitt thresholds, balance threshold, ground robot, friction, and runtime contact flags.

## Limit warnings are inconsistent

H1 checks the measured robot, while other variants may check the control robot. Confirm which state should be monitored before interpreting or disabling warnings.

## Excessive terminal output

Move high-frequency vectors to mc_rtc logger entries and guard development dumps before real-time experiments.

## Template examples do not run

They intentionally contain placeholder joints and zero-byte placeholder ONNX files. Replace joint groups, gains, `q0`, bodies/sensors, model file, and robot integration.
