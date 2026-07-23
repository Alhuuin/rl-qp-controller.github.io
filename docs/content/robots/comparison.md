---
title: Variant comparison
---

# Variant comparison

| Feature | Base template | H1 | HRP5P |
|---|---|---|---|
| Built FSM states | One RL-running `Initial` | One RL-running `Initial` | Separate `Initial` and `RL` |
| Startup control mode | Torque | Torque | Position |
| RL begins at startup | Yes | Yes | No |
| Runtime mode switching | No | No | Yes |
| Main run feedback | `ClosedLoopIntegrateReal` | `ClosedLoopIntegrateReal` | Selected by mode |
| Explicit posture task | No | No | Yes |
| Contact constraint YAML | Example disabled | Not enabled | Enabled |
| Compound-joint constraint | Example disabled | Not enabled | Enabled |
| Tilt anchor callback | No | Yes | No |
| External-force observer | Example only | Momentum observer | Force-sensor-based observer |
| Estimator toggled by states | No | No | Yes |
| Foot force contact logic | No | No | Implemented; active call commented |
| High-gain maps | No | No | Yes |
| QP bypass | Yes | Yes | Yes |
| Policies | Placeholder examples | H1 standing/walking | HRP5P walking policies |

## Template to H1

H1 mainly adds robot naming, H1 policies and conventions, `pelvis` as base body, BodySensor/Tilt/external-force observers, datastore `anchorFrameFunction`, `createContactAnchor()`, and measured-robot limit checks. The single-state RL architecture remains unchanged.

## Template to HRP5P

HRP5P adds or substantially changes task construction, position-mode startup, mode-aware solver execution, runtime torque/contact flags, high-gain vectors, estimator toggling, force-sensor contact classification, dedicated Initial/RL lifecycle, and contact/compound-joint constraints.

## Shared policy flow

```text
PolicyManager -> PolicyConfig -> ObservationManager -> ONNX
              -> action mapping and scaling -> q_rl
```

Use the template for a direct torque workflow, H1 as an example of custom state estimation, and HRP5P as an example of explicit startup, mode switching, and contact-aware integration.
