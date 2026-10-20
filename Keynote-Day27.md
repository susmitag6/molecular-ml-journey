# Day 27 - Reproducibility, Data Leakage, and Model Freezing

**Project 1:** Human Acetylcholinesterase (AChE) Bioactivity Prediction  
**Date:** October 20, 2026

## Objectives

Today, I aimed to audit the integrity of my training, validation, and test splits, identify possible molecular data leakage, and freeze the evaluation protocol and trained models before independent testing.

## What I Did

### 1. Verified dataset partitioning

I checked that the training, validation, and test indices were mutually exclusive and collectively covered the entire dataset.

| Subset | Molecules |
|---|---:|
| Training | 4,099 |
| Validation | 930 |
| Test | 1,258 |
| Total | 6,287 |

All pairwise index overlaps were zero, and the complete-partition check passed.

### 2. Audited molecular identity and scaffold overlap

I compared canonical SMILES and Bemis–Murcko scaffolds across subsets.

- **Training–validation:** Zero identical SMILES, but 20 shared chirality-agnostic scaffolds.
- **Training–test:** Two identical SMILES and 373 shared chirality-agnostic scaffolds.
- **Validation–test:** Zero identical SMILES and 100 shared chirality-agnostic scaffolds.

The 20 shared training–validation scaffolds affected 53 validation molecules (5.7%). The remaining 877 molecules (94.3%) had scaffolds absent from training.

I compared prediction errors between these groups:

| Validation subset | RF MAE | Tanimoto-SVR MAE |
|---|---:|---:|
| Shared scaffolds (53) | 0.607 | 0.516 |
| Unseen scaffolds (877) | 0.664 | 0.659 |
| Full validation (930) | 0.661 | 0.651 |

Both models performed better on the shared-scaffold subset, particularly Tanimoto-SVR. However, this subset was small, and the comparison was exploratory.

### 3. Investigated the scaffold-definition discrepancy

I repeated the scaffold audit with stereochemistry included.

- **Chirality-agnostic scaffold overlap:** 20 scaffolds.
- **Chirality-aware scaffold overlap:** Zero scaffolds.

I learned that the training and validation sets are disjoint under the chirality-aware Bemis–Murcko definition, but not under the chirality-agnostic definition. I still need to verify the original splitting code before claiming which definition was used to construct the split.

### 4. Investigated duplicate molecules across training and test

I identified two test molecules whose canonical SMILES were identical to molecules in training, despite having different ChEMBL IDs.

| Training row | Test row | Absolute ΔpIC50 |
|---|---|---:|
| 277 | 1108 | 0.236 |
| 1140 | 3392 | 0.700 |

For both pairs, the Morgan count fingerprints were identical, and the Morgan bit-fingerprint Tanimoto similarity was 1.0.

This revealed potential molecular-identity leakage and demonstrated that identical structures can have different recorded bioactivities, possibly reflecting experimental or registration differences.

### 5. Created a clean test subset

Before examining any test predictions, I excluded the two duplicate test entries (rows 1108 and 3392).

- Original test set: **1,258 molecules**
- Clean test set: **1,256 molecules**
- Remaining exact canonical-SMILES overlap with training or validation: **Zero**

I designated the clean subset for primary independent test reporting, while retaining the original split information for transparency.

### 6. Froze the evaluation protocol and models

I saved `ache_evaluation_protocol.json`, recording the clean test indices, excluded entries, primary models, and applicability-domain thresholds.

The frozen applicability-domain rule is:

- RF tree-disagreement standard deviation ≤ **0.7581355859721572**
- Maximum Morgan-bit Tanimoto similarity to training ≥ **0.7142857142857143**

I also saved the fitted models as `ache_rf_frozen.joblib` and `ache_tanimoto_svr_frozen.joblib`.

After reloading them, I verified that their validation predictions were unchanged:

- Random Forest maximum difference: \(3.55 \times 10^{-15}\)
- Tanimoto-SVR maximum difference: **0.0**

## What I Learned

Today reinforced that **index-level separation does not guarantee chemical independence**. Molecular datasets require additional checks for duplicate structures, stereochemistry, scaffold definitions, and potentially inconsistent experimental labels.

I also learned that the definition of a molecular scaffold directly affects claims about chemical generalization. Chirality-aware scaffold novelty and scaffold-connectivity novelty are not equivalent.

Finally, I recognized the importance of freezing preprocessing decisions, model parameters, test exclusions, and applicability-domain thresholds before examining independent test performance.

## Conclusions

Day 27 was a reproducibility and quality-control milestone rather than a model-development day.

I confirmed that the dataset indices were correctly partitioned, identified two exact molecular duplicates crossing the training–test boundary, characterized the scaffold-overlap discrepancy, and established a clean 1,256-molecule test subset.

I also preserved the fitted Random Forest and Tanimoto-SVR models and verified that serialization did not alter their predictions.

The validation results remain useful, but I must describe their scaffold-generalization implications precisely. The final random held-out test will assess predictive performance on unseen molecule entries, not strict unseen-scaffold generalization.

## Next Steps — Day 28

1. Verify the training-fingerprint ordering required by the Tanimoto-SVR prediction kernel.
2. Evaluate the frozen RF and Tanimoto-SVR on the clean independent test set.
3. Report test MAE, RMSE, and R² without further model tuning.
4. Evaluate the frozen applicability-domain rule, including test coverage and selective prediction error.
5. Document limitations involving scaffold overlap, chemical standardization, and experimental label variability.
