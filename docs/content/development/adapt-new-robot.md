---
title: Adapt to a new robot
---

# Adapt to a new robot

Use the base template repository as the starting point, and use the H1 and HRP5P repositories as implementation examples rather than drop-in configurations.

## 1. Rename the controller

Rename the controller class, exported constructor, source files, FSM states, YAML configuration, logger prefix, and GUI category. Verify that the name used in `mc_rtc.yaml` matches the exported controller name.

## 2. Define the controller joint order

The runtime uses `robot().refJointOrder()` as its canonical storage order. Confirm that:

- every joint expected by policy gain maps appears there;
- joint observations resolve to valid robot joints;
- selected joints are single-DoF where the current joint observation implementations require it;
- convention groups reproduce training order exactly.

## 3. Add a convention

In `policies/conventions.yaml`, define:

- joint groups such as `all`, `legs`, `arms`, or `all_no_grippers`;
- aliases from training-environment names to internal observation types;
- default parameters for those external names.

Keep convention data separate from the C++ implementations whenever the difference is only naming, ordering, grouping, or default scaling.

## 4. Configure robot-state sources

Choose `realRobot` or `robot`, set the base body, and make sure the selected object provides the encoders, body poses, body velocities, body sensors, and force sensors required by the chosen observations.

## 5. Rebuild mc_rtc integration

Review the main controller file and configuration for:

- posture and torque tasks;
- solver constraints;
- collisions;
- observer pipelines;
- contacts and surfaces;
- FSM states and transitions;
- robot command interface;
- GUI and logger naming.

## 6. Review every observation

Even generic-looking terms can encode assumptions about frames and sensor names. In the current HRP5P source, for example, base velocity observations validate a configurable name but compute from the literal body `Body`. Fix or generalize assumptions before using a different robot.

## 7. Start safely

1. Start in a stable non-RL posture state.
2. Confirm `q_zero` and current posture are close.
3. Verify observations without enabling motion.
4. Validate inference and targets in simulation.
5. Check QP constraints and contact behavior.
6. Move to the real robot only after logs and mappings are consistent.

<div class="next-links" markdown>
[Architecture reference](../reference/architecture.md){ .md-button }
[Observation reference](../reference/observations.md){ .md-button }
[Troubleshooting](../troubleshooting.md){ .md-button }
</div>
