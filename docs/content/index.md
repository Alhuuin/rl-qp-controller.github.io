---
title: RL QP Controller
---

<div class="hero" markdown>

# RL QP Controller

**Deploy ONNX reinforcement-learning policies inside an mc_rtc controller, with optional QP-based constraint enforcement.**

<div class="hero-actions" markdown>
[Get started](getting-started/general-information.md){ .md-button .md-button--primary }
[Deploy a policy](getting-started/deploy-policy.md){ .md-button }
[Explore the architecture](reference/architecture.md){ .md-button }
</div>

</div>

<div class="status-strip" markdown>
<span>Template-first design</span><span>ONNX inference</span><span>Convention-aware observations</span><span>Runtime policy switching</span>
</div>

## Project variants

The project is maintained as a reusable base and as robot-specific adaptations. The HRP5P repository is currently the most recent implementation and is the source used for this documentation.

<div class="repo-grid" markdown>

<div class="repo-card" markdown>
### :material-source-repository: Base template

The robot-agnostic starting point. Use this repository when adapting the controller to a new robot.

[Open template repository](https://github.com/example/rl-qp-controller-template){ .md-button }
</div>

<div class="repo-card" markdown>
### :material-robot: Unitree H1

The H1 adaptation, including its robot configuration, policies, and controller-specific integration.

[Open H1 repository](https://github.com/example/rl-qp-controller-h1){ .md-button }
</div>

<div class="repo-card repo-card--featured" markdown>
### :material-robot-industrial: HRP5P

The latest adaptation. It currently provides the most complete reference for observations, policy loading, logging, GUI controls, FSM integration, and force-based contacts.

[Open HRP5P repository](https://github.com/example/rl-qp-controller-hrp5p){ .md-button .md-button--primary }
</div>

</div>

!!! warning "Repository links are placeholders"
    Replace the three URLs above and the centralized links in `docs/mkdocs.yml` with the final public repository addresses.

## What the controller does

At runtime, the controller:

1. discovers policy folders under `policies_root`;
2. loads `policy.yaml`, `observations.yaml`, and the selected ONNX model;
3. resolves training-environment aliases and joint ordering through `conventions.yaml`;
4. constructs the policy input, including per-observation history;
5. runs ONNX inference at the configured policy period;
6. maps and scales the action into controller joint order;
7. produces a target posture and PD torque command;
8. either sends that torque through the QP task or bypasses the QP, depending on `use_QP`.

```text
Robot state ──► ObservationManager ──► ONNX policy ──► action mapping
                    ▲                                      │
                    │ history + convention                  ▼
             observations.yaml                    q_rl = q0 + scale·action
                                                           │
                                      ┌────────────────────┴───────────────────┐
                                      ▼                                        ▼
                              TorqueJointTask + QP                     direct torque path
```

!!! danger "QP bypass is a diagnostic mode"
    The GUI can switch QP enforcement off at runtime. In this mode, the controller applies the computed torque directly to the robot command path. Treat this as an expert debugging feature, not as a safety-equivalent operating mode.

## Documentation map

- **Getting started** explains the policy folder format and controller configuration.
- **Architecture** follows the code from configuration loading to action application.
- **Observations** documents every registered observation type and its parameters.
- **Logging and GUI** lists the actual logger entries, console messages, and runtime controls.
- **Development** explains how to add observations or adapt the template to another robot.
- **Robot-specific information** distinguishes reusable components from HRP5P-specific behavior.
