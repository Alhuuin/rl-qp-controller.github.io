---
title: Add an observation
---

# Add an observation

The observation framework separates configuration, implementation, and registration.

## 1. Declare the class

Add a subclass to `include/observation/Observations.h`.

```cpp
/** @brief Describe the quantity, frame, ordering, and units. */
class MyObservation : public Observation
{
public:
  MyObservation(const ObservationConfig &, const ObservationConvention &);

  void configure(const ObservationContext & context) override;
  int size() const override { return size_; }
  void compute(const ObservationContext & context,
               Eigen::Ref<Eigen::VectorXd> out) const override;

private:
  int size_ = 0;
};
```

`configure()` should resolve parameters and validate robot resources once. `compute()` should perform only the per-cycle computation.

## 2. Implement it

Add the implementation to `src/observation/Observations.cpp`.

Use:

- `context.observationRobot` for the selected robot state;
- `context.controllerJointOrder` for canonical controller order;
- `context.policyJointControllerIndices` for policy action order;
- `context.qZeroControllerOrder` for the active default pose;
- `context.command` and `context.phaseNormalized` for runtime signals;
- `context.convention.resolveObservationParameters(...)` to merge convention defaults and local overrides;
- `readScale(...)` for scalar-or-vector scale handling.

!!! warning "Validate dimensions in `configure()`"
    A wrong `size()` affects history allocation and the final ONNX input. Fail early when a parameter vector, index list, sensor, body, or joint selector is inconsistent.

## 3. Register the type

Add the implementation name to `makeDefaultObservationRegistry()` in `src/observation/Observation.cpp`:

```cpp
registry.registerType("my_observation", &makeObservation<MyObservation>);
```

It can then be used directly:

```yaml
- type: my_observation
  parameters: { ... }
```

## 4. Add optional aliases

Use `conventions.yaml` when a training environment calls the same concept by another name or requires default parameters:

```yaml
type_aliases:
  training_name: my_observation

default_parameters:
  training_name:
    scale: 0.5
```

## 5. Test history and logging

Test at least:

- one sample and multiple history samples;
- oldest-first and newest-first concatenation;
- scalar and vector scales;
- repeated instances with distinct `name` values;
- final dimension against an ONNX model;
- logger suffixes `_t`, `_t-1`, and so on.
