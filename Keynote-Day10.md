# Day 10 - Hyperparameter Tuning
**30 September 2026**

## Learning objectives

- Distinguish model parameters from hyperparameters.
- Understand how tree complexity and minimum leaf size affect training and validation performance.
- Select hyperparameters using cross-validation rather than the test set.
- Recognize why good random-split performance does not guarantee generalization to unfamiliar chemical scaffolds.

## Key concepts

**Model parameters** are learned during training, such as a decision tree's split locations and leaf predictions. **Hyperparameters** are specified before training, such as `max_depth`, `min_samples_leaf` and `n_estimators`.

Increasing regularization generally reduces model flexibility, but it does not necessarily improve validation performance.

## Experiment 1 — Effect of minimum leaf size

I trained Random Forest models using count Morgan fingerprints and compared five minimum leaf sizes with five-fold cross-validation on the training set.

| Minimum leaf size | Mean train R² | Mean validation R² | Validation std |
|---:|---:|---:|---:|
| 1 | 0.9621 | 0.7456 | 0.0113 |
| 2 | 0.9337 | 0.7396 | 0.0123 |
| 5 | 0.8528 | 0.7052 | 0.0138 |
| 10 | 0.7588 | 0.6470 | 0.0196 |
| 20 | 0.6431 | 0.5692 | 0.0135 |

Increasing the minimum leaf size consistently reduced both training and validation performance within the tested range.

The training–validation gap was largest at `min_samples_leaf=1`, but this setting also produced the highest validation R².

**Interpretation:** A large training–validation gap can indicate overfitting, but it does not establish that additional regularization will improve generalization. Model selection should be based on validation performance rather than the size of the gap alone.

## Experiment 2 — Joint hyperparameter tuning

I used `GridSearchCV` to compare:

- `max_depth`: 10, 20 and None
- `min_samples_leaf`: 1, 2 and 5

The search used five-fold cross-validation and 100 trees per Random Forest.

| Maximum depth | Minimum leaf size | Mean validation R² |
|---|---:|---:|
| None | 1 | **0.7433** |
| None | 2 | 0.7380 |
| None | 5 | 0.7036 |
| 20 | 1 | 0.6693 |
| 20 | 2 | 0.6660 |
| 20 | 5 | 0.6461 |
| 10 | 1 | 0.5589 |
| 10 | 2 | 0.5573 |
| 10 | 5 | 0.5478 |

The highest mean validation R² among the tested configurations was achieved with unrestricted tree depth and a minimum leaf size of one.

Restricting tree depth or increasing leaf size reduced validation performance in this search.

## Final model evaluation

I refitted the selected configuration on the complete training set using 300 trees.

| Test metric | Result |
|---|---:|
| MAE | 0.5147 |
| RMSE | 0.7486 |
| R² | 0.7455 |

These results reproduced the previous Day 9 count Morgan experiment.

Because I had already inspected this test set during exploratory work, this evaluation is a reproducibility check rather than a fully independent final assessment.

## What I learned

1. **Regularization is not automatically beneficial.** In these experiments, more restrictive trees performed worse on both training and validation data.
2. **The training–validation gap is a diagnostic, not a model-selection criterion.** The most flexible model had the largest gap but the highest validation R².
3. **Cross-validation should guide hyperparameter selection.** Repeatedly choosing settings based on test-set performance would compromise the test set's independence.
4. **Validation must reflect the intended prediction task.** Random cross-validation does not establish how well the model will predict activity for molecules with previously unseen scaffolds.

## Connection to molecular modeling

A flexible model can learn detailed relationships between molecular fingerprint features and bioactivity. However, some relationships may be specific to familiar chemical families or experimental data.

A high random-CV score therefore does not establish that the model has learned transferable molecular principles. Scaffold-aware evaluation will help investigate this limitation.

## Next step

Day 11: Compare random and scaffold-aware cross-validation, then examine whether the hyperparameters selected using random CV remain appropriate when predicting molecules from unseen scaffold groups.
