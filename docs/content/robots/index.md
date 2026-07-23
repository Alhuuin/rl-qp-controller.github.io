---
title: Robot-specific information
---

# Robot-specific information

The repositories share policy management, observation assembly, ONNX inference, and action mapping, but their mc_rtc integration differs substantially.

| Variant | Control/state architecture | Main robot-specific additions |
|---|---|---|
| [Base template](base-template.md) | One RL-running state | Placeholders and `TODO(robot)` adaptation points |
| [Unitree H1](h1.md) | One RL-running state, torque mode from startup | Tilt anchor callback and H1 observer pipeline |
| [HRP5P](hrp5p.md) | Position-controlled Initial state, dedicated torque-controlled RL state | Mode switching, posture tasks, contacts, force sensors, high gains, external-force toggling |

Start with the [side-by-side comparison](comparison.md) before porting code between variants.

!!! warning "Shared filenames do not imply shared behavior"
    The policy runtime is broadly reusable, but the main controller, observer pipeline, constraints, state lifecycle, and command interface are robot adaptations.
