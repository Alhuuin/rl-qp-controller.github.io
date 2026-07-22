---
title: Configure the controller
---

# Configure the controller

The robot-level controller file configures mc_rtc, policy discovery, the observation source, constraints, FSM transitions, and robot-specific gains.

## Reusable RL configuration

```yaml
robot:
  base_body: pelvis
  observation_source: realRobot

policies_root: /path/to/policies
default_policy: gazelle2
```

| Option | Default | Description |
|---|---|---|
| `robot.base_body` | `base_link` | Fallback body for observations such as projected gravity. |
| `robot.observation_source` | `realRobot` | `realRobot` or `robot`. |
| `policies_root` | `policies` | Directory scanned for policy subfolders. |
| `default_policy` | First loaded policy when absent | Name of the policy selected at startup. |

## mc_rtc integration

The HRP5P implementation configures:

- a ground robot module;
- encoder, body-sensor, and external-force observer components;
- acceleration contact constraints;
- compound-joint constraints;
- collision constraints;
- FSM state libraries, state files, transitions, and initial state.

These sections are robot/controller integration details, not policy-format requirements.

## Initial posture behavior

The HRP5P controller includes `is_initial_posture_rl`. When enabled, the initial state uses the active policy's `q_zero` as its posture target. Otherwise, the regular mc_rtc posture-task target remains the reference.

This option is useful when a learned policy expects a crouched or otherwise non-standard neutral posture.

## High gains

The HRP5P file also defines full `high_kp` and `high_kd` maps. These are separate from the policy gains and are used by robot-specific controller behavior. A new robot adaptation must either provide valid maps or remove the feature consistently from the controller source.

!!! warning "Separate generic and robot-specific configuration"
    Do not copy HRP5P observer names, surfaces, force sensors, gain maps, or contact logic unchanged into a new robot. Keep the reusable policy/runtime keys, then rebuild the mc_rtc integration around the target robot model.
