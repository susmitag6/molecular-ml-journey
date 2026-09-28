# Day 8 — Classical ML, Bias–Variance, and Cross-Validation

Today I covered the following concepts:

- **Baseline models:** `DummyRegressor` and why a useful model must outperform a trivial predictor.
- **Ridge regression:** additive/linear mapping from Morgan fingerprint features to pIC50 and the role of regularization \(\alpha\).
- **Tree ensembles:** Random Forest vs Extra Trees and their ability to model nonlinear feature interactions.
- **Bias–variance trade-off:** bias as systematic error of the learning procedure and variance as sensitivity of the learned model to the particular training sample.
- **Underfitting vs overfitting:** interpreted using training performance, validation/test performance, and the generalization gap.
- **Model complexity:** experimentally varied Random Forest `max_depth`.
- **Parameters vs hyperparameters:** learned coefficients/tree structures vs choices such as `alpha`, `max_depth`, and `n_estimators`.
- **Train / validation / test separation:** validation data are used for model selection, while the final test set should remain untouched during model development.
- **Cross-validation:** using multiple train/validation partitions to obtain a more robust estimate of model performance.
- **Cross-validation for hyperparameter selection:** selecting `max_depth` using only the training data before final test evaluation.

## Classical ML Models

Ridge regression assumes an approximately linear, additive relationship between Morgan fingerprint features and bioactivity. If molecular environments A and B are present, their contributions enter the prediction through their corresponding coefficients.

Random Forest can model nonlinear and conditional interactions. For example, the influence of molecular environment A can depend on whether environments B or C are also present.

Extra Trees can also capture nonlinear interactions but introduces additional randomness when constructing trees. This extra randomization can reduce variance but may also increase bias. Therefore, additional randomization or model flexibility does not automatically imply better generalization.

$$
\text{Dummy} \rightarrow \text{no molecular structure information used}
$$

$$
\text{Ridge} \rightarrow \text{structure information + approximately additive linear mapping}
$$

$$
\text{Random Forest / Extra Trees} \rightarrow \text{structure information + nonlinear interactions}
$$

For the same Morgan representation and random train/test split, I obtained:

| Model | MAE | RMSE | Test \(R^2\) |
|---|---:|---:|---:|
| Dummy | 1.157 | 1.484 | ~0.000 |
| Ridge | 0.729 | 1.002 | 0.544 |
| Extra Trees | 0.623 | 0.919 | 0.616 |
| Random Forest | 0.533 | 0.765 | 0.734 |

For this particular experiment, changing only the learning algorithm produced substantial differences in predictive performance. This shows that the relationship learned from a fixed molecular representation depends strongly on the model class.

## Bias–Variance Trade-Off

$$
\text{Bias} = \text{systematic error of the learning procedure}
$$

$$
\text{Variance} = \text{sensitivity of the learned model to the particular training sample}
$$

Greater model flexibility can potentially reduce bias but increase variance:

$$
\text{high flexibility} \rightarrow \text{potentially higher variance}.
$$

For squared-error prediction, the conceptual decomposition is:

$$
\text{expected prediction error} = \text{bias}^2 + \text{variance} + \text{irreducible noise}.
$$

For my ChEMBL AChE IC50 dataset, possible contributions include:

$$
\begin{aligned}
\text{Bias} &\Rightarrow
\text{model or representation is too restrictive},\\
\text{Variance} &\Rightarrow
\text{learned relationship changes strongly with the training molecules},\\
\text{Irreducible noise} &\Rightarrow
\text{experimental variability and other unpredictable components of measured bioactivity}.
\end{aligned}
$$

Training performance, validation/test performance, and the generalization gap should therefore be interpreted together:

$$
\text{training performance}
+
\text{validation/test performance}
+
\text{generalization gap}.
$$

The train–test gap is a useful diagnostic for overfitting, but it is not itself a direct measurement of the formal bias or variance terms.

## Model Complexity

A Random Forest is an ensemble of decision trees. Tree depth determines how many successive decision rules a tree can make from its root to a prediction.

Increasing `max_depth` allows increasingly complicated conditional relationships between fingerprint features to be represented.

For my Random Forest:

| `max_depth` | Train \(R^2\) | Test \(R^2\) | Gap |
|---:|---:|---:|---:|
| 2 | 0.253 | 0.268 | -0.016 |
| 5 | 0.450 | 0.447 | 0.003 |
| 10 | 0.618 | 0.546 | 0.071 |
| 20 | 0.801 | 0.649 | 0.152 |
| None | 0.952 | 0.734 | 0.218 |

Increasing depth increased both training and test performance in the range investigated, while also increasing the generalization gap.

Therefore, the data did not show a regime in which restricting depth improved test performance. This illustrates why experimental results should not be forced to follow a textbook underfitting–overfitting curve.

Conceptually:

$$
\text{underfitting}
\rightarrow
\boxed{\text{useful complexity}}
\rightarrow
\text{overfitting}.
$$

Model complexity should be selected using unseen **validation data**, rather than according to training performance.

## Model Parameters vs Hyperparameters

**Model parameters** are learned from the training data.

Examples include:
- Ridge coefficients \(\beta_j\)
- Decision-tree split structure
- Leaf predictions

**Hyperparameters** control the model or learning procedure and are specified before fitting.

Examples include:
- Ridge regularization strength `alpha`
- Random Forest `max_depth`
- Random Forest `n_estimators`

The workflow is:

$$
\text{Train}
\xrightarrow{\text{learn}}
\text{model parameters}
$$

$$
\text{Validation}
\xrightarrow{\text{choose}}
\text{hyperparameters / model}
$$

$$
\text{Test}
\xrightarrow{\text{evaluate}}
\text{final generalization}.
$$

Repeatedly choosing hyperparameters based on test-set performance indirectly uses information from the test set. Therefore, the test set should ideally remain untouched until model-development decisions have been completed.

## Cross-Validation

In 5-fold cross-validation, the dataset is divided into five subsets. Five models are trained, with each fold serving once as validation data and the remaining four folds serving as training data.

For the Random Forest I obtained:

$$
R^2 =
[0.75, 0.721, 0.788, 0.758, 0.742]
$$

with

$$
\overline{R^2}=0.749
$$

and

$$
\sigma_{R^2}=0.023.
$$

The mean CV score provides an estimate of typical validation performance:

$$
\boxed{
\text{CV mean}
\rightarrow
\text{typical estimated validation performance}
}
$$

while the standard deviation describes how stable that performance is across folds:

$$
\boxed{
\text{CV standard deviation}
\rightarrow
\text{stability of estimated performance across folds}
}
$$

The CV standard deviation should not be confused with the formal variance term in the bias–variance decomposition.

The relatively small standard deviation suggests that Random Forest performance was reasonably stable across these random partitions.

However, stable random cross-validation does **not** demonstrate generalization to unseen chemical scaffolds. My earlier scaffold-split experiment gave substantially lower performance, showing that validation design must match the scientific prediction problem.

## Using Cross-Validation for Hyperparameter Selection

Instead of selecting `max_depth` according to test performance, I performed 5-fold CV using only the training set:

| `max_depth` | Mean CV \(R^2\) | CV std |
|---:|---:|---:|
| 5 | 0.425 | 0.015 |
| 10 | 0.538 | 0.010 |
| 20 | 0.649 | 0.013 |
| None | 0.727 | 0.013 |

Among the tested values, `max_depth=None` produced the highest mean CV performance without increased instability across folds.

The correct workflow is therefore:

$$
\text{Training set}
\xrightarrow{\text{5-fold CV}}
\text{choose hyperparameters}
$$

followed by:

$$
\text{chosen hyperparameters}
\rightarrow
\text{refit using all training data}
\rightarrow
\boxed{\text{evaluate once on the untouched test set}}.
$$

## Main Takeaway

A good molecular ML workflow is not simply about finding a model with a high \(R^2\).

I need to ask:

1. Does the model learn useful signal beyond a trivial baseline?
2. Is the model flexible enough to capture the relevant structure–activity relationship?
3. Is it overfitting training-specific patterns?
4. Are hyperparameters selected without contaminating the final test set?
5. Is performance stable across different validation partitions?
6. Most importantly, does the validation strategy represent the chemical generalization problem I actually care about?

A model can have strong and stable random-split performance while still performing substantially worse on unseen chemical scaffolds. Therefore, **model evaluation and validation design are part of the scientific problem, not merely technical details.**
