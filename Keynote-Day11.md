# Day 11 — Scaffold-Aware Cross-Validation
**1 October 2026**

## Research question

Does scaffold-aware cross-validation provide a different estimate of model performance from random cross-validation? Does changing the validation strategy also change the preferred Random Forest hyperparameters?

## Hypothesis

I expect scaffold-aware CV to produce a lower mean R² than random CV because it evaluates the model on previously unseen Bemis–Murcko scaffolds.

Random CV can place molecules from the same scaffold family in both training and validation partitions, potentially making the prediction task easier. Scaffold-aware CV prevents exact scaffold overlap within each fold.

## Experiment 1 : Random versus scaffold-aware cross-validation

I evaluated the same Random Forest model using count Morgan fingerprints and two five-fold cross-validation strategies.

The model used 300 trees, unrestricted maximum depth and a minimum leaf size of one. Both experiments were conducted within the original training set.

| Fold | Random CV R² | Scaffold CV R² |
|---|---:|---:|
| 1 | 0.7571 | 0.5412 |
| 2 | 0.7607 | 0.6191 |
| 3 | 0.7320 | 0.5538 |
| 4 | 0.7375 | 0.5628 |
| 5 | 0.7405 | 0.5471 |
| **Mean** | **0.7456** | **0.5648** |

Scaffold-aware CV reduced mean R² by 0.1808 points relative to random CV.

Scaffold-CV fold 2 performed noticeably better than the other folds. This suggests that predictive difficulty varies among held-out scaffold groups. Examining the chemical composition, activity distributions and nearest-training-molecule similarities of each fold could help explain this difference.

### Scientific interpretation

The model predicted held-out molecules more accurately when the validation strategy allowed scaffold families to be shared between training and validation.

Requiring unseen Murcko scaffolds made the prediction task more challenging for this dataset. This demonstrates why the validation strategy should reflect the model's intended application.

However, scaffold separation does not guarantee low fingerprint similarity: molecules with different Murcko scaffolds can still have highly similar Morgan fingerprints.

## Experiment 2 : Scaffold-aware hyperparameter tuning

I used `GridSearchCV` with five-fold scaffold-aware CV to compare maximum tree depths of 10, 20 and None, and minimum leaf sizes of 1, 2 and 5.

The grid search used 100 trees per Random Forest.

| Maximum depth | Minimum leaf size | Mean train R² | Mean scaffold-CV R² | CV std |
|---|---:|---:|---:|---:|
| None | 1 | 0.9621 | **0.5635** | 0.0308 |
| None | 2 | 0.9344 | 0.5567 | 0.0317 |
| None | 5 | 0.8559 | 0.5326 | 0.0293 |
| 20 | 1 | 0.8346 | 0.5155 | 0.0381 |
| 20 | 2 | 0.8168 | 0.5109 | 0.0391 |
| 20 | 5 | 0.7687 | 0.4968 | 0.0367 |
| 10 | 1 | 0.6519 | 0.4425 | 0.0351 |
| 10 | 2 | 0.6450 | 0.4397 | 0.0361 |
| 10 | 5 | 0.6245 | 0.4310 | 0.0340 |

**Selected hyperparameters:**
- `max_depth=None`
- `min_samples_leaf=1`

The highest mean scaffold-CV R² was 0.5635.

## Three scientific conclusions

### 1. Validation strategy changes estimated performance

Random CV produced substantially higher scores than scaffold-aware CV, but both strategies selected the same hyperparameters among the configurations tested.

The difference in performance estimates was more consequential than the difference in hyperparameter selection.

### 2. Stronger regularization did not solve the unfamiliar-scaffold problem

A highly flexible Random Forest can learn detailed relationships that may be specific to familiar chemical families. A more restrictive model might sometimes learn relationships that transfer better, but it can also underfit.

In this experiment, restricting tree depth or increasing minimum leaf size reduced scaffold-CV performance. For example, limiting depth to 10 reduced mean scaffold-CV R² from 0.5635 to 0.4425 when the minimum leaf size was one.

Restricting model flexibility is not a substitute for informative molecular representations and representative training chemistry.

### 3. Scaffold-aware performance varies more between folds

The standard deviation for the selected configuration was 0.0308 under scaffold-aware CV, compared with 0.0133 under random CV in the corresponding 100-tree grid search.

Different held-out scaffold groups therefore presented different prediction challenges. These fold-to-fold variations are descriptive and should not be interpreted as formal uncertainty intervals for future predictions.

The 100-tree scaffold grid search achieved R² = 0.5635, while the earlier 300-tree scaffold-CV experiment achieved R² = 0.5648. The small difference is consistent with the change in the number of trees.

## Model applicability

A molecule can have a Murcko scaffold absent from the training set while retaining high Morgan fingerprint similarity to a training molecule.

High similarity alone does not guarantee a reliable prediction. Before trusting an individual prediction, I should consider scaffold-aware validation performance, the measured activities of nearby training molecules, possible activity cliffs and model uncertainty.

Scaffold-aware CV evaluates performance across groups of unseen scaffolds, but it cannot establish the reliability of every individual prediction.

## Main conclusion

Scaffold-aware cross-validation reduced mean R² from 0.7456 to 0.5648 for the same count Morgan Random Forest model. Both random and scaffold-aware hyperparameter searches selected unrestricted tree depth and a minimum leaf size of one.

The main limitation was therefore not resolved by stronger tree regularization. Predictive performance depends on the chemical space represented in training, the molecular features available to the model and the evaluation strategy.

**Next:** Investigate molecular features and activity cliffs to understand why high structural similarity does not necessarily imply similar bioactivity.
