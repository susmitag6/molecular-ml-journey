# Research Journal : Day 31
**Project 1:** Human Acetylcholinesterase (AChE) Bioactivity Prediction  
**Topic:** Reproducibility, Frozen Model Verification, and Pipeline Engineering  
**Status:** Completed

## 1. Objectives

Today, I focused on making my AChE molecular machine learning project reproducible and suitable for a scientific GitHub portfolio.

My objectives were to:

1. Audit the cleaned dataset, frozen models, and evaluation protocol.
2. Verify the integrity of the saved test indices.
3. Preserve the exact training, validation, and test splits.
4. Confirm that the saved dataset retained the original molecular ordering.
5. Reconstruct Random Forest and Tanimoto-SVR predictions from saved artifacts.
6. Document the software environment and inference dependencies.

Unlike previous days, I did not train or optimize any models. My focus was on verifying that the existing results could be reproduced reliably.

## 2. Experiment 1 — Artifact Audit

### Method

I checked whether the essential project artifacts existed and inspected the saved dataset and evaluation protocol.

### Results

All four required artifacts were present:

| Artifact | Status |
|---|---|
| `ache_ic50_clean.csv` | Found |
| `ache_rf_frozen.joblib` | Found |
| `ache_tanimoto_svr_frozen.joblib` | Found |
| `ache_evaluation_protocol.json` | Found |

The cleaned dataset contained **6,287 rows and four columns**:

- `molecule_chembl_id`
- `canonical_smiles`
- `pIC50`
- `n_measurements`

The evaluation protocol contained information about dataset sizes, excluded test indices, primary model definitions, applicability-domain criteria, and test-set interpretation.

### Interpretation

The essential dataset, model, and evaluation artifacts were available. However, their existence alone did not establish reproducibility, so I proceeded to verify their internal consistency.

## 3. Experiment 2 — Evaluation Protocol Integrity

### Method

I checked whether the saved test indices were valid, unique, and consistent with the dataset.

I also verified that the two previously identified duplicate test entries had been excluded.

### Results

All six protocol consistency checks passed.

| Property | Result |
|---|---|
| Dataset size | 6,287 |
| Clean test size | 1,256 |
| Test indices within dataset bounds | Passed |
| Test indices unique | Passed |
| Excluded indices absent from clean test | Passed |
| Number of excluded indices | 2 |

The excluded indices were **1108 and 3392**.

### Interpretation

The saved evaluation protocol was structurally consistent with the dataset.

This reduced the risk of accidentally evaluating the frozen models on the wrong test subset.

## 4. Experiment 3 — Identifying a Missing Reproducibility Dependency

### Method

I investigated whether the original training and validation indices had been preserved outside the notebook.

### Results

The original index arrays were still available in notebook memory:

| Split | Molecules | Stored in evaluation protocol |
|---|---:|---|
| Training | 4,099 | No |
| Validation | 930 | No |
| Clean test | 1,256 | Yes |

I identified a reproducibility gap: the training and validation indices had not been included in the saved evaluation protocol.

### Interpretation

This was particularly important for Tanimoto-SVR because its precomputed similarity kernel depends on the exact training-molecule ordering.

Simply recovering the same training molecules would not be sufficient if their order changed.

## 5. Experiment 4 — Freezing the Original Dataset Splits

### Method

I saved the exact ordered training, validation, and clean test indices into:

`ache_frozen_splits.npz`

I then reloaded the file and verified index preservation and split disjointness.

I also calculated a SHA-256 checksum of the cleaned CSV.

### Results

All six verification checks passed:

- Training indices preserved.
- Validation indices preserved.
- Test indices preserved.
- Training and validation disjoint.
- Training and test disjoint.
- Validation and test disjoint.

The dataset checksum was:

`cd3dafeb44d4f7a649c02976f365cc48f706aeba742acafa4575291d97bc357b`

### Interpretation

I successfully preserved the original split membership and ordering.

The checksum provides a way to detect future changes to the saved CSV. It does not independently establish that the CSV was the original training input, so I performed a separate row-alignment check.

## 6. Experiment 5 — Dataset Row-Alignment Verification

### Method

I compared the saved CSV with the original notebook DataFrame using molecule IDs, canonical SMILES, and pIC50 values.

### Results

All three checks passed:

| Check | Result |
|---|---|
| Molecule ID ordering | Matched |
| Canonical SMILES ordering | Matched |
| pIC50 values | Matched |

### Interpretation

The saved CSV preserved the molecular identities, structures, and activity values in the same row order as the notebook DataFrame.

This was essential because the frozen split indices refer to specific dataset row positions.

## 7. Experiments 6–8 — Random Forest Reproduction and Ordering Debugging

### Objective

I attempted to reconstruct the original Random Forest test predictions using only the saved CSV, frozen split indices, and serialized model.

### Method

I regenerated radius-2, 2048-dimensional Morgan count fingerprints for the clean test molecules and loaded the frozen Random Forest model.

I then compared the reconstructed predictions against the original Day 28 prediction array.

### Initial results

The reconstructed model reproduced the original aggregate metrics exactly:

| Metric | Reconstructed RF |
|---|---:|
| MAE | 0.565433 |
| RMSE | 0.815533 |
| R² | 0.696993 |

However, the initial elementwise prediction comparison failed.

The maximum absolute prediction difference was **10.2111 pIC50 units**.

### Investigation

I compared the original and reconstructed target arrays and discovered that their ordering differed.

The original test indices were shuffled, while the test indices stored in the evaluation protocol were sorted.

Both arrays contained the same 1,256 molecule indices, but in different orders.

Consequently, comparing the prediction arrays position by position incorrectly associated predictions with different molecules.

### Resolution

I constructed an index-based mapping to align the reconstructed predictions with the original molecule ordering.

After realignment:

| Check | Result |
|---|---|
| Same test-index membership | True |
| Same test-index order before alignment | False |
| Targets match after alignment | True |
| Predictions match after alignment | True |
| Maximum prediction difference | \(5.33 \times 10^{-15}\) |

### Interpretation

The discrepancy was caused by **test-set ordering**, not by an error in the model or fingerprint reconstruction.

I learned that reproducing aggregate metrics is insufficient to establish prediction-level reproducibility.

Each prediction must remain associated with the correct molecular identity.

This was one of the most important technical lessons of the day.

## 8. Experiment 9 — Tanimoto-SVR Reproduction

### Method

I loaded the frozen Tanimoto-SVR model and reconstructed its test-to-training kernel using:

- Radius-2 Morgan binary fingerprints.
- 2,048 fingerprint bits.
- Tanimoto similarity.
- The exact original training-index order.

The reconstructed kernel had shape:

\[
K_{\mathrm{test}}\in\mathbb{R}^{1256\times4099}
\]

### Results

| Metric | Reconstructed Tanimoto-SVR |
|---|---:|
| MAE | 0.547969 |
| RMSE | 0.788877 |
| R² | 0.716477 |

The original training order was preserved.

After aligning the test molecules by identity, the reconstructed SVR predictions matched the original predictions **exactly**, with a maximum difference of **0.0**.

### Interpretation

I successfully demonstrated reproducible inference for both primary models.

This also verified the importance of distinguishing their molecular representations:

- Random Forest used Morgan **count** fingerprints.
- Tanimoto-SVR used Morgan **binary** fingerprints and a precomputed similarity kernel.

The two representations are not interchangeable.

## 9. Experiments 10–11 — Environment Documentation

### Method

I recorded the software versions used in the verified environment and saved them in:

`ache_environment_versions.json`

I then generated a minimal inference dependency specification:

`ache_requirements_inference.txt`

### Results

The recorded environment included:

| Software | Version |
|---|---|
| Python | 3.11.15 |
| NumPy | 2.4.6 |
| pandas | 3.0.3 |
| scikit-learn | 1.9.1 |
| RDKit | 2026.3.6 |
| joblib | 1.6.0 |
| SciPy | 1.17.1 |
| PyTorch | 2.14.1 |
| PyTorch Geometric | 2.8.0.post1 |

The inference requirements file contained six pinned dependencies: NumPy, pandas, scikit-learn, RDKit, joblib, and SciPy.

### Interpretation

I documented the software environment in which the frozen model predictions were successfully reproduced.

The requirements file is a version-pinned specification, not a complete lockfile. Installation and execution in a clean environment remain to be tested.

## 10. Final Reproducibility Artifacts

By the end of Day 31, I had verified or created the following files:

| File | Purpose |
|---|---|
| `ache_ic50_clean.csv` | Cleaned molecular dataset |
| `ache_rf_frozen.joblib` | Frozen Random Forest |
| `ache_tanimoto_svr_frozen.joblib` | Frozen Tanimoto-SVR |
| `ache_evaluation_protocol.json` | Frozen evaluation definitions |
| `ache_frozen_splits.npz` | Exact ordered dataset splits |
| `ache_environment_versions.json` | Recorded software environment |
| `ache_requirements_inference.txt` | Pinned inference dependencies |

These artifacts provide the foundation for a standalone evaluation pipeline.

## 11. What I Learned

Today, I learned several important principles of reproducible scientific machine learning.

**First, saving trained models is not enough.** Reproducibility also requires preserving the dataset, feature definitions, split membership, split ordering, and evaluation protocol.

**Second, molecular identity alignment is critical.** Two prediction arrays can produce identical aggregate metrics while appearing completely different under an incorrect elementwise comparison.

**Third, precomputed-kernel models require special care.** Tanimoto-SVR depends on the exact ordering of training molecules because the kernel columns correspond to individual training samples.

**Fourth, reproducible inference and reproducible training are different goals.** I verified that the frozen models reproduce their predictions, but I have not yet demonstrated a complete retraining workflow from a clean environment.

**Finally, debugging a reproducibility failure can reveal important methodological assumptions.** The ordering discrepancy helped me understand how seemingly minor data-handling choices can affect scientific verification.

## 12. Final Conclusion

Day 31 marked a transition from experimental model development to reproducible research engineering.

I successfully audited the project artifacts, preserved the original dataset splits, verified dataset row alignment, and independently reconstructed the frozen RF and Tanimoto-SVR test predictions.

The Random Forest predictions matched to floating-point precision after correcting test-index ordering, while the Tanimoto-SVR predictions matched exactly.

**My main achievement was establishing reproducible inference for both primary AChE bioactivity models using saved artifacts.**

The remaining work is to convert these verified components into a clean project structure, produce publication-quality scientific figures, document the methodology and limitations, and perform a final reproducibility audit.

## 13. Next Steps — Day 32

**Scientific Figures and Results Presentation**

I will prepare figures and tables that communicate the key scientific findings:

- Predicted versus observed pIC50.
- Residual and prediction-error distributions.
- Applicability-domain coverage and accuracy.
- Chemical similarity and scaffold-generalization effects.
- RF versus Tanimoto-SVR performance comparisons.

I will use the frozen evaluation results and build the figures one at a time, ensuring that each visualization has a clear scientific purpose.
