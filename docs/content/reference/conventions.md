
# Training conventions

A convention describes how controller-side data is translated into the representation expected by a training environment.

It can define:

- the policy-facing joint order;
- named joint subsets used by observations and actions;
- aliases between training-environment observation names and registered controller observation types.

Set the convention in `observations.yaml`:

```yaml
training_convention: mc_rtc
```

A convention changes representation, not semantics. Always compare it with the actual training configuration rather than relying only on the convention name.
