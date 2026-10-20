# Day 28 : Independent Test Evaluation and Applicability-Domain Generalization

**Project 1:** Human Acetylcholinesterase (AChE) Bioactivity Prediction  


## Objectives

Today, I aimed to evaluate my frozen Random Forest (RF) and Tanimoto-kernel Support Vector Regression (SVR) models on an independent test set and determine whether the applicability-domain (AD) strategy developed during validation generalizes to unseen test molecules.

I also wanted to test whether combining RF uncertainty with chemical similarity provides additional predictive reliability compared with using RF uncertainty alone.

## What I Did

### 1. Verified Tanimoto kernel reproducibility

Before evaluating the test set, I reconstructed the validation-to-training Tanimoto similarity matrix using the original training fingerprint order.

- Original and reconstructed kernel shapes: **(930, 4099)**
- Kernel matrices identical: **True**
- Maximum kernel difference: **0.0**
- SVR predictions unchanged: **True**

This confirmed that the precomputed-kernel SVR could be evaluated reproducibly without accidentally changing the training fingerprint ordering.

### 2. Evaluated frozen models on the independent test set

I evaluated both models on the **1,256-molecule clean test set**, defined on Day 27 by excluding two exact molecular duplicates shared with training.

I made no changes to model parameters or preprocessing.

| Model | Dataset | MAE | RMSE | R² |
|---|---|---:|---:|---:|
| Random Forest | Validation | 0.661 | 0.907 | 0.626 |
| Random Forest | Clean test | **0.565** | **0.816** | **0.697** |
| Tanimoto-SVR | Validation | 0.651 | 0.889 | 0.642 |
| Tanimoto-SVR | Clean test | **0.548** | **0.789** | **0.716** |

Both models performed better on test than validation.

Tanimoto-SVR retained the best overall performance, with a test MAE advantage of **0.017 pIC50 units** over RF.

I interpreted this cautiously because the test set originated from a random split, whereas validation was disjoint under a chirality-aware scaffold definition. The test set may therefore contain more familiar chemistry.

### 3. Evaluated the frozen applicability-domain rule

I applied the Day 26 thresholds without modification:

- RF tree-disagreement standard deviation ≤ **0.7581356**
- Maximum Morgan-bit Tanimoto similarity to training ≥ **0.7142857**

The combined rule selected **636 of 1,256 molecules**, corresponding to **50.6% coverage**.

| Test subset | RF MAE | Tanimoto-SVR MAE |
|---|---:|---:|
| Inside AD (636) | **0.396** | **0.406** |
| Outside AD (620) | 0.740 | 0.693 |
| Full test (1,256) | 0.565 | 0.548 |

Inside the AD, RF MAE was approximately **29.9% lower** than on the full test set. Tanimoto-SVR MAE was approximately **25.9% lower**.

This supported the hypothesis that the frozen AD rule identifies a subset with lower prediction errors.

However, it does not provide calibrated uncertainty intervals or guarantee accurate predictions for individual molecules.

### 4. Compared combined AD against uncertainty-only selection

To determine whether chemical similarity added useful information beyond RF uncertainty, I compared both methods at identical coverage.

The uncertainty-only method selected the **636 molecules with the lowest RF tree disagreement**.

| Selection method | Coverage | RF MAE |
|---|---:|---:|
| Combined uncertainty + similarity | 50.6% | 0.396 |
| Uncertainty only | 50.6% | **0.376** |

Contrary to my Day 26 validation result, uncertainty-only selection performed better on the independent test set.

The combined rule had approximately **5.25% higher MAE** than uncertainty-only selection.

This was an important negative result: the additional benefit of chemical similarity observed during validation did not generalize at the tested coverage.

### 5. Investigated the disagreement between selection methods

The two methods selected **563 molecules in common**, while each selected 73 different molecules.

| Selection group | Molecules | RF MAE | Mean RF uncertainty | Mean similarity |
|---|---:|---:|---:|---:|
| Selected by both | 563 | **0.363** | 0.490 | 0.874 |
| Combined AD only | 73 | 0.644 | 0.726 | 0.856 |
| Uncertainty only | 73 | **0.472** | 0.558 | 0.602 |

The combined rule favored 73 molecules with higher chemical similarity but also higher uncertainty and prediction error.

The uncertainty-only method instead selected 73 molecules with lower similarity but better predictive accuracy.

This explained why the combined rule underperformed at matched coverage.

## What I Learned

1. **Independent testing is essential.** A selection strategy that improves validation performance does not necessarily outperform alternatives on held-out data.

2. **Chemical similarity and predictive reliability are not equivalent.** Highly similar molecules can still have large prediction errors, potentially because of activity cliffs, structural effects not adequately captured by the model, or experimental variability.

3. **Applicability-domain performance must be reported with coverage.** Lower MAE on a selected subset is meaningful only when the fraction of molecules receiving predictions is also specified.

4. **Matched-coverage comparisons are necessary.** Comparing selective errors at different coverage levels can produce misleading conclusions.

5. **Negative
