# Day 19 — Molecular GNN: Mean Pooling vs Sum Pooling

**Objective:** Investigate whether replacing global mean pooling with global sum pooling improves AChE pIC50 prediction and the model's ability to distinguish weak and potent inhibitors.

### Hypothesis

Sum pooling might improve molecular property prediction because it preserves accumulated atom contributions and information related to molecular size. However, the effect must be determined experimentally.

### Method

I compared two bond-aware GINE graph neural networks using the same molecular graph features, architecture size, scaffold-disjoint training/validation split, and training hyperparameters. Both models contained 19,457 trainable parameters. The principal architectural difference was global mean pooling versus global sum pooling.

The validation set contained 930 molecules. I evaluated overall prediction error, systematic bias across potency groups, prediction variability, and within-group Spearman rank correlation.

### Results

| Metric | Mean pooling | Sum pooling |
|---|---:|---:|
| Validation MAE | 0.928 | 0.926 |
| Validation RMSE | 1.193 | 1.211 |
| Validation R² | 0.354 | 0.334 |

Sum pooling reduced the mean signed error for low-potency inhibitors from +0.804 to +0.338 pIC50 units and for high-potency inhibitors from −1.082 to −0.730.

However, its higher prediction standard deviation did not consistently translate into better accuracy. It improved MAE at the low- and high-potency extremes but worsened MAE for medium-potency inhibitors.

Within the high-potency group, Spearman rank correlation was 0.395 for mean pooling and 0.403 for sum pooling. The improvement was negligible.

### Interpretation

Prediction standard deviation alone is not sufficient to assess model quality. A model can produce a wider range of predictions without correctly assigning those predictions to individual molecules.

Groupwise mean signed error reveals systematic overestimation or underestimation, MAE measures error magnitude, and Spearman correlation evaluates ranking agreement.

The pooling comparison showed a trade-off: sum pooling reduced directional bias at the potency extremes but did not improve overall predictive performance.

### Limitations

The mean- and sum-pooling models were not trained with matched random seeds. Consequently, the observed differences cannot be attributed exclusively to the pooling operation.

The evaluation also used a validation set examined repeatedly during model development, so it should not be treated as an untouched final test set.

### Conclusion

Changing the graph pooling operation alone did not close the performance gap between the GNN and the random forest using count Morgan fingerprints. Future experiments should investigate graph representation, architecture, optimization, and data preprocessing through controlled comparisons.
