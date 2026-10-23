# Research Journal : Day 31
**Project 1:** Human Acetylcholinesterase (AChE) Bioactivity Prediction  
**Topic:** Reproducibility, Frozen Model Verification, and GNN Pipeline Reconstruction  
**Status:** Completed

## 1. Objectives

Today, I focused on making my AChE molecular machine learning project reproducible and suitable for a scientific GitHub portfolio.

My objectives were to:

1. Audit the cleaned dataset, frozen models, and evaluation protocol.
2. Preserve and verify the exact training, validation, and test splits.
3. Reproduce Random Forest and Tanimoto-SVR predictions from saved artifacts.
4. Document the software environment and inference dependencies.
5. Recover and verify the previously trained 4-layer GINE GNN.
6. Reconstruct the original GNN architecture and SMILES-to-graph featurization code.

I did not retrain or optimize any models. My focus was on verifying the existing results and preserving the computational procedures needed to reproduce them.

## 2. Dataset and Artifact Audit

I first checked the essential project artifacts:

- `ache_ic50_clean.csv`
- `ache_rf_frozen.joblib`
- `ache_tanimoto_svr_frozen.joblib`
- `ache_evaluation_protocol.json`

All files were present.

The cleaned dataset contained **6,287 molecular records**, with molecule ChEMBL IDs, canonical SMILES, pIC50 values, and measurement counts.

I verified that the saved evaluation protocol was internally consistent. The clean test set contained **1,256 molecules**, and the two previously identified duplicate test entries, indices 1108 and 3392, were excluded.

All six protocol consistency checks passed.

### Interpretation

The core dataset and model artifacts were available, and the evaluation protocol correctly identified the intended test subset.

However, I discovered that the training and validation indices had not been saved separately, which represented an important reproducibility gap.

## 3. Freezing and Verifying Dataset Splits

I saved the exact original split indices in:

`ache_frozen_splits.npz`

The splits contained:

| Split | Molecules |
|---|---:|
| Training | 4,099 |
| Validation | 930 |
| Clean test | 1,256 |

I verified that all indices survived serialization and that the three splits were pairwise disjoint.

I also compared the saved CSV against the original notebook DataFrame.

The molecule IDs, SMILES, and pIC50 values matched in their original row order.

The SHA-256 checksum of the cleaned dataset was:

`cd3dafeb44d4f7a649c02976f365cc48f706aeba742acafa4575291d97bc357b`

### Interpretation

Preserving exact split membership and row ordering is essential because model features and predictions must remain associated with the correct molecular records.

For precomputed-kernel methods, training-sample ordering is particularly important.

## 4. Random Forest Inference Reproduction

I reconstructed the Random Forest evaluation using the saved dataset, frozen model, and Morgan count fingerprints.

The recomputed clean-test results were:

| Metric | Result |
|---|---:|
| MAE | 0.565433 |
| RMSE | 0.815533 |
| R² | 0.696993 |

These matched the original Day 28 results.

However, an initial elementwise comparison between the original and reconstructed predictions failed, with a maximum difference of approximately **10.211 pIC50 units**.

### Debugging the discrepancy

I discovered that the original test indices were shuffled, whereas the saved test indices were sorted.

Both arrays contained the same molecules, but in different orders.

After realigning predictions using the original dataset indices:

- Target values matched.
- Predictions matched.
- Maximum prediction difference was approximately \(5.33\times10^{-15}\).

### Interpretation

The discrepancy was caused by test-set ordering rather than incorrect predictions.

This experiment demonstrated that **reproducing aggregate metrics is not sufficient to verify prediction-level reproducibility**. Predictions must also be aligned by molecular identity.

## 5. Tanimoto-SVR Inference Reproduction

I reconstructed the precomputed Tanimoto kernel using radius-2, 2,048-bit Morgan binary fingerprints.

The test-to-training kernel had dimensions:

$$
K_{\mathrm{test}}\in\mathbb{R}^{1256\times4099}
$$

The saved training order was preserved exactly.

The reconstructed clean-test results were:

| Metric | Result |
|---|---:|
| MAE | 0.547969 |
| RMSE | 0.788877 |
| R² | 0.716477 |

After aligning test indices, the reconstructed predictions matched the original SVR predictions **exactly**, with a maximum difference of zero.

### Interpretation

I successfully reproduced the frozen inference results of both primary classical ML models.

This also confirmed the importance of preserving their distinct representations:

- Random Forest: Morgan **count** fingerprints.
- Tanimoto-SVR: Morgan **binary** fingerprints with a precomputed similarity kernel.

## 6. Software Environment Documentation

I recorded the software environment in:

`ache_environment_versions.json`

The key versions were:

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

I also created `ache_requirements_inference.txt`, containing the six pinned dependencies needed for the classical ML inference workflow.

These files document the verified environment, although a clean-environment installation and rerun remain outstanding.

---

# Additional Investigation — GNN Reproducibility

## 7. Recovering the Original GNN Checkpoint

I decided to preserve the graph neural network as part of the final scientific report, even though it performed worse than the fingerprint-based baselines.

The GNN is scientifically useful because it provides a direct comparison between learned molecular graph representations and established molecular fingerprint methods.

I initially saved the current model as:

`ache_gine_4layer_frozen.pt`

I then recovered an earlier checkpoint from Day 21:

`ache_gnn_4layer_best.pt`

The Day 21 checkpoint contained additional metadata:

- Hidden dimension: 64
- Number of GINE layers: 4
- Atom feature dimension: 34
- Bond feature dimension: 7
- Recorded validation MAE: 0.813
- Recorded validation MSE: 1.118583
- Random seed: 42

I compared the two saved checkpoints and verified that **their model-state tensors were identical**.

### Interpretation

I selected `ache_gnn_4layer_best.pt` as the canonical checkpoint because it contains the same weights along with more complete configuration information.

The duplicate checkpoint can be removed during final repository cleanup, after confirming that the canonical file is safely preserved.

## 8. Resolving the GNN Parameter-Count Discrepancy

During checkpoint verification, I initially obtained two different model-size counts:

- 36,101 elements in the checkpoint's `state_dict`.
- 36,097 parameters reported by `model.parameters()`.

I investigated this discrepancy by separately counting parameters and buffers.

| Component | Count |
|---|---:|
| Model parameters | **36,097** |
| Non-trainable buffers | 4 |
| Total state-dict elements | 36,101 |

The four additional elements were the `eps` buffers stored in the four `GINEConv` layers:

- `convs.0.eps`
- `convs.1.eps`
- `convs.2.eps`
- `convs.3.eps`

### Interpretation

The discrepancy was explained by the distinction between model parameters and non-trainable buffers.

**The correct model parameter count for the final report is 36,097.**

This investigation improved my understanding of PyTorch checkpoint structure and the information contained in a `state_dict`.

## 9. Recovering the Original GNN Architecture

I attempted to retrieve the model source code using Python's `inspect.getsource()`, but this produced an `OSError`.

The error occurred because the model class had been defined interactively in Jupyter, where source-code inspection is not always reliable.

I recovered the original class definition from the notebook execution history instead.

The recovered architecture was:

| Component | Implementation |
|---|---|
| Atom encoder | Linear(34, 64) |
| Bond encoder | Linear(7, 64) |
| Message-passing layers | Four GINEConv layers |
| GINE internal MLP | Linear → ReLU → Linear |
| Activation | ReLU after each GINE layer |
| Graph pooling | Global mean pooling |
| Regression head | Linear(64, 1) |

I saved the recovered architecture in:

`ache_gnn_model.py`

The original checkpoint loaded into this standalone model with `strict=True`, and all checkpoint keys matched successfully.

### Interpretation

I successfully reconstructed the model architecture independently of the notebook class definition.

This is essential for making the GNN usable by another researcher without relying on the original interactive session.

## 10. Debugging a GNN Prediction Mismatch

I initially compared predictions from the original notebook's `deep_model` and the standalone model.

Both models processed the same 930 validation graphs, but their predictions differed.

The maximum absolute difference was approximately **1.049 pIC50 units**.

I investigated their `state_dict` tensors and found that **22 tensors differed**, including atom-encoder, bond-encoder, and message-passing weights.

Further comparison established that:

| Model | Matched Day 21 checkpoint |
|---|---|
| Standalone model | **True** |
| Current notebook `deep_model` | False |

The saved checkpoint files themselves remained identical.

### Interpretation

The prediction discrepancy was caused by differences between the current in-memory model and the frozen checkpoint weights.

The precise point at which the notebook model changed was not established.

This taught me an important reproducibility lesson: **the current state of an interactive notebook variable should not automatically be treated as the frozen experimental model**.

Saved checkpoints and explicit state comparisons provide a more reliable reference.

## 11. Reproducing GNN Validation Performance

I loaded the original Day 21 checkpoint into the standalone `AChEGNNDeep` implementation and evaluated it on the existing validation graphs.

### Results

| Metric | Recomputed result |
|---|---:|
| Validation molecules | 930 |
| MAE | **0.812569** |
| MSE | 1.118583 |
| RMSE | 1.057631 |
| R² | 0.492385 |

The recomputed MSE matched the checkpoint's recorded value within numerical precision.

### Interpretation

I demonstrated that the **standalone GNN architecture and saved checkpoint reproduce the original validation performance**.

This was a stronger reproducibility test than simply verifying checkpoint metadata.

The GNN can now be included in the final report as a reproducible validation benchmark.

However, its results must remain clearly distinguished from the frozen RF and SVR held-out test results.

## 12. Recovering the Molecular Graph-Featurization Pipeline

A saved GNN checkpoint and architecture are insufficient for full reproducibility unless the molecular graph representation is also preserved.

I therefore recovered the original SMILES-to-graph code and its dependencies from notebook history.

The original converter used RDKit and PyTorch Geometric to construct molecular graphs.

Each chemical bond was represented by two directed edges.

### Atom features

| Feature group | Dimensions |
|---|---:|
| Element identity | 14 |
| Atom degree | 6 |
| Formal charge | 4 |
| Hybridization | 4 |
| Aromaticity | 1 |
| Total hydrogen count | 5 |
| **Total** | **34** |

### Bond features

| Feature group | Dimensions |
|---|---:|
| Bond type | 5 |
| Conjugation | 1 |
| Ring membership | 1 |
| **Total** | **7** |

I also recovered the original one-hot encoding function, including its handling of unknown categories.

The exact ordering of feature categories was preserved.

I saved the complete implementation as:

`ache_gnn_features.py`

### Scientific interpretation

The frozen GNN uses molecular connectivity and local chemical features but does not explicitly encode atom chirality in its 34-dimensional representation.

My separate chirality-aware GNN experiment used additional atom features and should be distinguished from this frozen 4-layer GINE model.

Preserving the original category ordering is essential because changing one-hot positions would change the chemical meaning of the model inputs.

## 13. Verifying Standalone Graph Equivalence

I compared the graphs generated by the original notebook function and the standalone feature module for 100 molecules from the cleaned dataset.

For each molecule, I compared:

- Atom feature tensor, `x`.
- Graph connectivity, `edge_index`.
- Bond feature tensor, `edge_attr`.

### Results

| Check | Result |
|---|---|
| Molecules tested | 100 |
| Identical graph representations | **100** |
| All comparisons passed | **True** |

### Interpretation

The standalone graph-featurization module reproduced the original molecular representations exactly for all 100 tested molecules.

This provides direct evidence that the extracted implementation preserves the original GNN input representation.

It does not yet establish equivalence across the entire dataset, but it is a strong targeted verification.

## 14. Final Model Comparison

The reproducibility work allows me to preserve all three main models for the final scientific report.

| Model | Scaffold-disjoint validation MAE | Random held-out test MAE |
|---|---:|---:|
| Random Forest | 0.661 | 0.565 |
| Tanimoto-SVR | **0.651** | **0.548** |
| 4-layer GINE GNN | 0.813 | Not evaluated |

The GNN performed worse than both fingerprint-based models on the scaffold-disjoint validation split.

This result does not imply that graph neural networks are generally inferior. It shows that the specific GINE architecture and training procedure used in this project did not outperform the simpler baselines under the evaluated conditions.

The RF and SVR held-out test results remain the primary final evaluation results. The GNN remains a validation-only comparison unless a separate test evaluation is explicitly performed and reported.

## 15. Final Reproducibility Artifacts

By the end of Day 31, I had preserved the following essential artifacts:

| Artifact | Purpose |
|---|---|
| `ache_ic50_clean.csv` | Cleaned AChE dataset |
| `ache_frozen_splits.npz` | Exact dataset splits |
| `ache_evaluation_protocol.json` | Frozen evaluation protocol |
| `ache_rf_frozen.joblib` | Frozen Random Forest |
| `ache_tanimoto_svr_frozen.joblib` | Frozen Tanimoto-SVR |
| `ache_gnn_4layer_best.pt` | Canonical GNN checkpoint |
| `ache_gnn_model.py` | Standalone GNN architecture |
| `ache_gnn_features.py` | Standalone molecular graph featurization |
| `ache_environment_versions.json` | Software environment |
| `ache_requirements_inference.txt` | Classical ML inference dependencies |

The GNN also requires the recorded PyTorch and PyTorch Geometric versions, which are not included in the minimal classical ML inference requirements file.

## 16. Key Lessons Learned

**Reproducibility requires more than saving model weights.** A reliable scientific ML project must preserve data, split definitions, molecular representations, architecture, software dependencies, and evaluation procedures.

**Data ordering is scientifically important.** The Random Forest debugging experiment demonstrated that identical molecules in different array orders can produce misleading prediction comparisons.

**Interactive notebook state can change.** The GNN investigation showed that an in-memory model can differ from a frozen checkpoint, even when the model class remains unchanged.

**Parameters and checkpoint elements are not identical concepts.** The GINE model had 36,097 parameters and four additional buffers.

**Molecular featurization is part of the model.** Reconstructing the exact atom and bond features was essential to preserving the meaning of the trained GNN weights.

**Negative model comparisons are scientifically valuable.** The GNN's weaker validation performance provides evidence about the limitations of the specific graph-based approach tested here.

## 17. Final Conclusion

Day 31 became a comprehensive reproducibility and scientific software engineering exercise.

I successfully reproduced the frozen Random Forest and Tanimoto-SVR test predictions, preserved the exact dataset splits, documented the software environment, and recovered the original GNN architecture and graph-featurization pipeline.

I also independently reproduced the saved 4-layer GINE model's validation metrics using a standalone Python implementation.

The most important debugging discoveries involved test-index ordering, changes in interactive notebook model state, and the distinction between PyTorch parameters and buffers.

**My main achievement was transforming three previously trained molecular ML models into substantially more reproducible scientific artifacts.**

The remaining work includes a clean-environment rerun, complete dependency packaging, and final repository organization. Full training reproducibility has not yet been demonstrated.

## 18. Next Steps — Day 32

**Scientific Figures and Results Presentation**

I will begin preparing figures and tables for the final Project 1 report, including:

- Predicted versus observed pIC50.
- Prediction residual distributions.
- Applicability-domain coverage and prediction error.
- Chemical similarity and scaffold-generalization effects.
- RF versus Tanimoto-SVR comparisons.
- A clearly labeled validation-only comparison including the GINE GNN.

I will generate and inspect the figures one at a time, using the frozen model results and maintaining the distinction between validation and held-out test performance.
