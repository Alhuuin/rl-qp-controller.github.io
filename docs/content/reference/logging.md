---
title: Logging and GUI
---

# Logging and GUI

The controller exposes three complementary diagnostic surfaces:

1. mc_rtc logger entries for time-series analysis;
2. structured console messages during loading and validation;
3. mc_rtc GUI controls for runtime inspection and modification.

## Logger entries

### Gains and control state

| Entry | Content |
|---|---|
| `HRP5pRLQPController_kp_base` | Unscaled policy stiffness vector |
| `HRP5pRLQPController_kd_base` | Unscaled policy damping vector |
| `HRP5pRLQPController_kp_current` | Currently applied stiffness vector |
| `HRP5pRLQPController_kd_current` | Currently applied damping vector |
| `HRP5pRLQPController_pd_gains_ratio` | Runtime gain ratio |
| `HRP5pRLQPController_useQP` | Whether QP enforcement is enabled |

### Policy runtime

| Entry | Content |
|---|---|
| `HRP5pRLQPController_RL_q` | Current full-order `q_rl` target |
| `HRP5pRLQPController_RL_qZero` | Current full-order default pose |
| `HRP5pRLQPController_RL_currentObservation` | Concatenated ONNX input |
| `HRP5pRLQPController_RL_currentAction` | Raw ONNX output |
| `HRP5pRLQPController_RL_currentActionScaled` | Full controller-order scaled action |
| `HRP5pRLQPController_RL_actionScale` | Full controller-order scale vector |
| `HRP5pRLQPController_RL_command` | `[vx, vy, yaw_rate]` command |
| `HRP5pRLQPController_currentPolicy` | Active policy name |
| `HRP5pRLQPController_observationConvention` | Active convention name |

### Per-observation history

Every configured observation is also logged under:

```text
HRP5pRLQPController_Observations_<observation-name>
```

For observations with history, the current sample uses suffix `_t`, while older entries use `_t-1`, `_t-2`, and so on. The implementation compensates for the configured storage order so the logger naming remains relative to the current sample.

!!! tip "Give repeated terms unique names"
    When the same observation type appears more than once, set `name` explicitly. Otherwise logger names can collide or become ambiguous.

## GUI panels

### Policy

- current policy name;
- current policy folder;
- observation convention;
- policy selection combo box;
- **Reload current policy** button.

Reloading reconstructs the ONNX interface, mappings, gains, observations, and history from the files on disk.

### PD Gains

- **PD Gains Ratio** slider from `0.0` to `2.0`;
- current `kp` vector;
- current `kd` vector.

The runtime applies the same ratio to both base gain vectors through this slider.

### Control

- **Toggle QP Control** button;
- QP status label (`Enforced` or `Bypassed`);
- **Toggle print joint limits** button;
- joint-limit print status.

!!! danger "Changing QP mode is immediate"
    The GUI toggle changes `useQP` at runtime. Make this control unavailable or protected in deployments where accidental bypass is unacceptable.

### Command

Editable number inputs for:

- `vx`;
- `vy`;
- `yaw_rate`.

These values feed the `command` observation directly.

### RL state panel

Each RL FSM state adds labels for:

- whether a policy is loaded;
- ONNX observation size;
- ONNX action size;
- policy step size;
- current policy.

## Important console messages

The code emits structured messages with component prefixes such as:

```text
[PolicyManager]
[PolicyConfig]
[RLPolicyRuntime]
[ObservationManager]
[ObservationConvention]
[ObservationRegistry]
[Observation:<name>]
[RLStateRunner]
```

High-value messages include:

- number of policies loaded and active policy;
- resolved controller indices for `action.joints` and `controlled_joints`;
- ONNX input/output shapes and selected execution provider;
- loaded observation count and convention;
- observation and action dimension mismatches;
- missing bodies, sensors, joints, gain entries, or required YAML fields.

## Development-level verbose output

The current HRP5P source still contains several unconditional warning/error-level debug prints, including full observations, raw actions, scaled actions, `q_zero`, `q_rl`, gain-map sizes, and action-scale vectors.

!!! warning "Clean up before long experiments"
    These messages run at controller frequency in some paths and can flood the terminal or distort real-time performance. Convert useful values to logger entries—which already exist—or guard console output behind an explicit debug option before production deployment.

## Recommended debugging sequence

1. Confirm the selected policy folder and convention in the GUI.
2. Check the policy-load success message and ONNX shapes.
3. Inspect resolved action joint indices.
4. Compare `currentObservation` dimension with the ONNX input.
5. Plot individual per-observation logger entries, not only the concatenated vector.
6. Compare raw action, scaled action, `q_zero`, and `q_rl`.
7. Verify QP status and gain ratio.
8. Only then investigate solver or robot-specific contact behavior.
