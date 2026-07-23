---
title: Controller configuration options
---

# Controller configuration options

This reference separates reusable RL runtime options from mc_rtc integration and robot-specific extensions.

## Reusable runtime keys

| Key | Template default | Meaning |
|---|---|---|
| `robot.base_body` | `base_link` | Default body used by floating-base observations. |
| `robot.observation_source` | `realRobot` | Robot state used to compute observations: `realRobot` or `robot`. |
| `policies_root` | controller `policies` directory | Directory containing `conventions.yaml` and policy subdirectories. |
| `default_policy` | explicitly configured | Policy selected after discovery. |

The policy and observation formats are documented in [Policy options](policy-options.md) and [Available observations](observations.md).

## Standard mc_rtc FSM keys

| Key | Purpose |
|---|---|
| `Managed` | Whether the controller is managed externally. |
| `StepByStep` | Enables step-by-step FSM operation. |
| `IdleKeepState` | Controls idle-state retention. |
| `StatesLibraries` | Search paths for compiled state libraries. |
| `StatesFiles` | Search paths for state data. |
| `VerboseStateFactory` | Enables verbose state discovery. |
| `states` | Per-state configuration. |
| `transitions` | FSM transition table. |
| `init` | Initial FSM state. |

These keys are not interpreted by the RL policy runtime itself.

## Environment and collisions

All three repositories add `env/ground`. They also use a minimal collision constraint:

```yaml
robots:
  ground:
    module: env/ground

collisions:
  - type: collision
    useMinimal: true
```

Both remain robot- and experiment-dependent.

## Constraint examples

The template leaves contact and compound-joint constraints disabled. HRP5P enables both. H1 does not enable a controller-level contact constraint in the supplied configuration.

```yaml
constraints:
  - type: contact
    contactType: acceleration
  - type: compoundJoint
```

## Observer pipeline differences

### Encoder observer

Template and H1 request encoder velocities. HRP5P currently requests finite-difference velocity. The repositories also use different YAML nesting styles, so verify the syntax accepted by the installed observer version.

### BodySensor observer

H1 and HRP5P update a `FloatingBase` body sensor. HRP5P also requests acceleration update.

### Tilt observer

H1 adds a required Tilt observer using the datastore callback `anchorFrameFunction`, implemented by `createContactAnchor()` in the main controller.

### ExternalForcesObserver

H1 uses a momentum observer with commanded torque and no force-torque sensors. HRP5P uses a force-sensor-based method with motor torque measurements and starts inactive. These are not interchangeable defaults.

## HRP5P-only keys

| Key | Meaning |
|---|---|
| `is_initial_posture_rl` | Use active policy `q_zero` as the Initial-state posture target. |
| `high_kp` | Full high-stiffness map used by HRP5P state logic. |
| `high_kd` | Full high-damping map used by HRP5P state logic. |

## Behavior implemented in C++, not YAML alone

- H1 registers `anchorFrameFunction` in the datastore.
- HRP5P changes `ControlMode` between `Position` and `Torque`.
- HRP5P selects solver feedback according to the active mode.
- HRP5P toggles the external-force estimator through datastore calls.
- HRP5P implements force-sensor-driven foot-contact changes.
- QP bypass writes torque commands directly in the main controller.

Always inspect the main controller and active FSM states together with the YAML.
