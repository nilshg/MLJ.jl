# Machines

Under the hood, calling `fit!` on a machine calls either `MLJBase.fit`
or `MLJBase.update`, depending on the machine's internal state, as
recorded in additional fields `previous_model` and
`previous_rows`. These lower-level `fit` and `update` methods dispatch
on the model and a view of the data defined by the optional `rows`
keyword argument of `fit!` (all rows by default). In this way, if a
model `update` method is implemented, calls to `fit!` can avoid
redundant calculations for certain kinds of model mutations (eg,
increasing the number of epochs in a neural network).

The interested reader can learn more on machine internals by examining
the simplified code excerpt in [Internals](internals.md).

```@example machines
using MLJ; color_off() # hide
forest = EnsembleModel(atom=(@load DecisionTreeClassifier), n=20);
X, y = @load_iris;
mach = machine(forest, X, y)
fit!(mach, verbosity=2);
```
    
Generally, changing a hyperparameter triggers retraining on calls to
subsequent `fit!`:
```@repl machines
forest.bagging_fraction=0.5
fit!(mach, verbosity=2);
```

However, for this iterative model, increasing the iteration parameter
only adds models to the existing ensemble:

```@repl machines
forest.n=15
fit!(mach, verbosity=2);
```

Call `fit!` again without making a change and no retraining occurs:

```@repl machines
fit!(mach);
```

However, retraining can be forced:

```@repl machines
fit!(mach, force=true);
```

And is retriggered if the view of the data changes:

```@repl machines 
fit!(mach, rows=1:100);
```

```@repl machines 
fit!(mach, rows=1:100); 
```

For a supervised machine the `predict` method calls a lower-level
`MLJBase.predict` method, dispatched on the underlying model and the
`fitresult` (see below). To see `predict` in action, as well as its
unsupervised cousins `transform` and `inverse_transform`, see
[Getting Started](index.md).

Here is a complete list of the fields of a machine:

- `model` - the struct containing the hyperparameters to be used in
  calls to `fit!`

- `fitresult` - the learned parameters in a raw form, initially undefined

- `args` -  a tuple of the data (in the supervised learning example above, `args = (X, y)`)

- `report` - outputs of training not encoded in `fitresult` (eg, feature rankings)

- `previous_model` - a deep copy of the model used in the last call to `fit!`

- `previous_rows` -  a copy of the row indices used in last call to `fit!`

- `cache`

Instead of data `X` and `y`, the `machine` constructor can be provided
`Node` or `Source` objects ("dynamic data") to obtain a
`NodalMachine`, rather than a regular `Machine` object, which includes
the same fields listed above. See [Composing
Models](composing_models.md) for more on this advanced feature.


### API Reference

```@docs
fit!
```
