# Day 25 : Comparing Classical ML Models for AChE Bioactivity Prediction

## Today's Objective

Today, I investigated whether alternative machine learning methods could improve the prediction of AChE inhibitory activity (pIC50) compared with my existing Random Forest and GNN models.

I focused on XGBoost, Support Vector Regression (SVR), and Tanimoto-kernel SVR, using the same scaffold-disjoint training and validation sets.

## What I Did

**1. Tested XGBoost**

I trained an XGBoost regressor using Morgan count fingerprints (radius 2, 2,048 features). It achieved an MAE of 0.696, slightly worse than Random Forest (0.661).

**2. Tested RBF-kernel SVR**

I trained an SVR model using standardized Morgan count fingerprints and an RBF kernel. Its validation MAE was 0.743, indicating that this configuration was less effective than Random Forest.

**3. Implemented Tanimoto-kernel SVR**

I constructed pairwise Tanimoto similarity matrices using binary Morgan fingerprints.

- Training kernel: 4,099 × 4,099
- Validation kernel: 930 × 4,099
- Kernel matrix was symmetric with diagonal values equal to 1.

I trained an SVR model using the precomputed Tanimoto kernel with `C=10` and `epsilon=0.1`.

The model used 3,432 support vectors and achieved the best validation performance among the tested methods.

## Model Comparison

| Model | MAE ↓ | RMSE ↓ | R² ↑ |
|---|---:|---:|---:|
| **Tanimoto-SVR** | **0.651** | **0.889** | **0.642** |
| Random Forest | 0.661 | 0.907 | 0.626 |
| XGBoost | 0.696 | 0.914 | 0.621 |
| RBF-SVR | 0.743 | 1.009 | 0.538 |
| 4-layer GNN | 0.813 | 1.058 | 0.492 |

## Per-Molecule Error Analysis

I compared Tanimoto-SVR and Random Forest predictions for all 930 validation molecules.

- Tanimoto-SVR performed better on **480 molecules (51.6%)**.
- Random Forest performed better on **450 molecules (48.4%)**.
- Mean absolute-error improvement: **0.0101 pIC50**.
- Median per-molecule improvement: **0.0147 pIC50**.
- The 90th-percentile absolute error was almost identical: 1.435 for Tanimoto-SVR versus 1.437 for Random Forest.

Although Tanimoto-SVR achieved better overall metrics, its advantage over Random Forest was relatively small.

## What I Learned

The most important finding was that **molecular representation and similarity measures strongly influence model performance**.

Using a chemically meaningful Tanimoto kernel improved SVR performance substantially compared with the standard RBF kernel (MAE decreased from 0.743 to 0.651).

I also learned that a lower aggregate MAE does not necessarily mean a model is consistently better for every molecule. Tanimoto-SVR and Random Forest performed similarly for many validation compounds.

Because several models have now been compared using the same validation set, further model selection could lead to overfitting to that dataset.

## Conclusion

Today, **Tanimoto-kernel SVR became my best-performing model on the scaffold-disjoint validation set**, achieving MAE = 0.651, RMSE = 0.889, and R² = 0.642.

# Day 25 — Comparing Classical ML Models for AChE Bioactivity Prediction

## Today's Objective

Today, I investigated whether alternative machine learning methods could improve the prediction of AChE inhibitory activity (pIC50) compared with my existing Random Forest and GNN models.

I focused on XGBoost, Support Vector Regression (SVR), and Tanimoto-kernel SVR, using the same scaffold-disjoint training and validation sets.

## What I Did

**1. Tested XGBoost**

I trained an XGBoost regressor using Morgan count fingerprints (radius 2, 2,048 features). It achieved an MAE of 0.696, slightly worse than Random Forest (0.661).

**2. Tested RBF-kernel SVR**

I trained an SVR model using standardized Morgan count fingerprints and an RBF kernel. Its validation MAE was 0.743, indicating that this configuration was less effective than Random Forest.

**3. Implemented Tanimoto-kernel SVR**

I constructed pairwise Tanimoto similarity matrices using binary Morgan fingerprints.

- Training kernel: 4,099 × 4,099
- Validation kernel: 930 × 4,099
- Kernel matrix was symmetric with diagonal values equal to 1.

I trained an SVR model using the precomputed Tanimoto kernel with `C=10` and `epsilon=0.1`.

The model used 3,432 support vectors and achieved the best validation performance among the tested methods.

## Model Comparison

| Model | MAE ↓ | RMSE ↓ | R² ↑ |
|---|---:|---:|---:|
| **Tanimoto-SVR** | **0.651** | **0.889** | **0.642** |
| Random Forest | 0.661 | 0.907 | 0.626 |
| XGBoost | 0.696 | 0.914 | 0.621 |
| RBF-SVR | 0.743 | 1.009 | 0.538 |
| 4-layer GNN | 0.813 | 1.058 | 0.492 |

## Per-Molecule Error Analysis

I compared Tanimoto-SVR and Random Forest predictions for all 930 validation molecules.

- Tanimoto-SVR performed better on **480 molecules (51.6%)**.
- Random Forest performed better on **450 molecules (48.4%)**.
- Mean absolute-error improvement: **0.0101 pIC50**.
- Median per-molecule improvement: **0.0147 pIC50**.
- The 90th-percentile absolute error was almost identical: 1.435 for Tanimoto-SVR versus 1.437 for Random Forest.

Although Tanimoto-SVR achieved better overall metrics, its advantage over Random Forest was relatively small.

## What I Learned

The most important finding was that **molecular representation and similarity measures strongly influence model performance**.

Using a chemically meaningful Tanimoto kernel improved SVR performance substantially compared with the standard RBF kernel (MAE decreased from 0.743 to 0.651).

I also learned that a lower aggregate MAE does not necessarily mean a model is consistently better for every molecule. Tanimoto-SVR and Random Forest performed similarly for many validation compounds.

Because several models have now been compared using the same validation set, further model selection could lead to overfitting to that dataset.

## Conclusion

Today, **Tanimoto-kernel SVR became my best-performing model on the scaffold-disjoint validation set**, achieving MAE = 0.651, RMSE = 0.889, and R² = 0.642.

However, its improvement over Random Forest was modest. I will therefore retain both models as leading candidates rather than declaring Tanimoto-SVR definitively superior.

**My main takeaway:** In molecular machine learning, choosing an appropriate chemical representation and similarity function can be more important than simply using a more complex algorithm.

**Next — Day 26:** Consolidate uncertainty estimation and chemical applicability-domain analysis, then prepare for final evaluation on the untouched test set.

Although Tanimoto-SVR achieved the lowest scaffold-disjoint validation MAE (0.651), this does not establish that it will generalize better than Random Forest (0.661). Multiple models were compared using the same validation set, creating a risk of model-selection bias. The improvement was also small. Therefore, I will retain both models as leading candidates and use the untouched test set for final evaluation. Because the test set was randomly split rather than scaffold-disjoint, its results will measure held-out performance but not strictly unseen-scaffold generalization.
