
# General information

The RL QP Controller loads reinforcement-learning policies, prepares their observations, evaluates their ONNX models, and applies the resulting commands through an mc_rtc controller.

## Configuration files

The project intentionally separates configuration by scope so each file has one clear role.

- Controller-level configuration: robot-level and runtime integration settings (observation source, base body, policies root, default policy) belong to the controller configuration file. See [Configure the controller](controller-configuration.md).
- Shared convention configuration: `conventions.yaml` defines training-environment conventions (joint groups and order, eventual type aliases, default observation parameters). Policy confugiration files depend on it. See [Configure conventions.yaml](configure-conventions.md).
- Policy-folder configuration: each policy is described by a directory containing an ONNX model alongside `policy.yaml` and `observations.yaml`.
	- `policy.yaml` defines action mapping and control parameters.
	- `observations.yaml` defines policy input terms, order, and history.
	- See [Deploy a policy](deploy-policy.md) for more information.

This split helps avoid mixing robot integration concerns with policy-format concerns.

## Modular controller design

This controller is modular by design and keeps two logic layers separated:

- Usual mc_rtc controller/FSM logic: main controller integration, tasks, constraints, observers, states, and transitions.
- RL policy runtime logic: policy loading, convention resolution, observation assembly, ONNX inference, and action mapping/scaling.

In practice, the RL layer plugs into the mc_rtc layer instead of the two being intertwined. This makes the codebase easier to adapt to new robots: reuse the RL policy pipeline while rebuilding only the robot-specific controller/FSM integration.

## Policy loading

Policies are loaded from the configured policy base directory. By default, this directory is `policies/`. Every valid policy subdirectory found there is loaded and can be selected at runtime through the mc_rtc GUI.

## Observation history

For each observation (joint velocity, last action...), a `history` can be specified, corresponding to the number of past values of this observation that is needed by the policy.

Observation history is concatenated by observation, newest-first by default. Please confirm that this matches the ordering used during training.
