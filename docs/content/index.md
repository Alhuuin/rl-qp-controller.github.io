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

<!-- <div class="status-strip" markdown>
<span>Template-first design</span><span>ONNX inference</span><span>Convention-aware observations</span><span>Runtime policy switching</span>
</div> -->

## Project variants

The project is maintained as a reusable base with adaptation repositories. Here are the currently available versions:

<div class="repo-grid" markdown>

<div class="repo-card" markdown>
### :material-source-repository: Base template

The robot-agnostic starting point. Use this repository when adapting the controller to a new robot.

[Open template repository](https://github.com/example/rl-qp-controller-template){ .md-button .md-button--primary }
</div>

<div class="repo-card" markdown>
### :material-robot: Unitree H1

The H1 adaptation, including its robot configuration, policies, and controller-specific integration.

[Open H1 repository](https://github.com/example/rl-qp-controller-h1){ .md-button }
</div>

<div class="repo-card repo-card--featured" markdown>
### :material-robot: HRP5P

The HRP5P adaptation, including its robot configuration, policies, controller-specific integration, and FSM states.
This adaptation is currently closed-source.

[HRP5P repository (closed-source)](https://github.com/example/rl-qp-controller-hrp5p){ .md-button }
</div>

</div>

## What the controller does

At runtime, the controller:

1. searches for policy folders under `policies_root`;
2. for the selected policy, loads `policy.yaml`, `observations.yaml` and ONNX model;
3. resolves training-environment aliases and joint ordering through `conventions.yaml`;
4. constructs the policy input according to the observations configuration;
5. runs ONNX inference at the configured policy period;
6. maps and scales the action into controller joint order;
7. produces a target posture and PD torque command;
8. applies torque through the configured control path (`use_QP`).

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


## Documentation map

- **Getting started** walks you through the basics: general information, configuring conventions, setting up the controller, and deploying your first policy.
- **Reference** documents the full system architecture, available controllers and policies, observation types, and logging outputs.
- **Development** explains how to add custom observations, adapt the template to a new robot, and apply context-based integration advice.
- **Troubleshooting** covers common issues and solutions.
