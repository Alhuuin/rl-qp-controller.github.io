
# General information

The RL QP Controller loads reinforcement-learning policies, prepares their observations, evaluates their ONNX models, and applies the resulting commands through an mc_rtc controller.

## Policy discovery

Policies are loaded from the configured policy base directory. By default, this directory is `policies/`. Every valid policy subdirectory found there is loaded and can be selected at runtime.

## Observation history

Observation history is concatenated newest-first by default. Confirm that this matches the ordering used during training, because changing the history order changes the policy input even when its total size remains correct.

## Initial posture

By default, the initial state uses the default posture of the robot's mc_rtc posture task. The controller may instead use the default pose defined by the selected default policy. This helps avoid an unnecessary posture transition when the policy was trained around a slightly different nominal configuration.
