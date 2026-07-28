---
title: Adapt to a new robot
---

# Adapt to a new robot

Start from the base template, then apply the robot-specific changes below.
Existing robot variants are useful references when you need examples of different adaptation strategies.

## 1. Create a new fork

Fork the template repository and name it after the new robot.

Example: `robotname_rl_qp_controller`

## 2. Rename exported integration points

Rename all integration points consistently: controller class/files, exported constructor, state classes and export names, CMake targets, configuration file, GUI/logger prefixes, and test selection.

```bash
sed -i -e's/NewRLQPController/RobotNameRLQPController/g' `find . -not -path '*/.*' -type f`
git mv src/NewRLQPController.cpp src/RobotNameRLQPController.cpp
git mv src/NewRLQPController.h src/RobotNameRLQPController.h
git mv src/states/NewRLQPController_Initial.cpp src/states/RobotNameRLQPController_Initial.cpp
git mv src/states/NewRLQPController_Initial.h src/states/RobotNameRLQPController_Initial.h
git mv etc/NewRLQPController.in.yaml etc/RobotNameRLQPController.in.yaml
```

You can also customize the project name in `vcpkg.json`, following the [vcpkg manifest rules](https://github.com/microsoft/vcpkg/blob/master/docs/users/manifests.md).

## 3. Complete required robot adaptations

The required robot-specific changes are tagged as `//TODO(robot)` in the codebase.
Use the command below to track what is still pending:

```bash
grep -rn "TODO(robot)" .
```

Every remaining robot-specific name must be intentional.

Apply the following modifications :

### 3.1 In the controller configuration file

- Adjust the body used by the floating-based observations.

- Adjust the `ObserverPipeline` depending on the observations' needs.

### 3.2 In the controller's header

- Tune the CBF gains.

### 3.3 In the controller's source file

- Review selected constraints.

- Tune stiffness and weight for the used `mc_task`.

- Validate contact/constraint consistency (surfaces, friction, collision pairs, and contact behavior expected by your states).

### 3.4 In `conventions.yaml`

- Replace placeholder joint names and add your own convention(s).

- Ensure the convention order is exactly the one the policy is waiting for, since runtime vectors and policy-related mappings depend on it.

You should now be able to add your own policies and configure their observations. If any observation is missing by default, please refer to [Add an observation](./add-observation.md).