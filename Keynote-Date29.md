# Day 29 : Statistical Confidence, Model Disagreements, and Data-Quality Audit

**Project 1:** AChE Bioactivity Prediction  
**Date:** October 21, 2026

## Objectives

Today, I aimed to determine whether Tanimoto-SVR's small performance advantage over Random Forest (RF) was statistically convincing, understand which molecules contributed to their prediction differences, and investigate suspiciously extreme pIC50 values.

## What I Did and Results

### 1. Paired bootstrap analysis

I performed 5,000 paired bootstrap resamples of the 1,256-molecule clean test set. In each replicate, I sampled molecule indices with replacement while keeping RF and SVR errors paired.

| Metric | Result |
|---|---:|
| RF test MAE | 0.5654 |
| Tanimoto-SVR test MAE | 0.5480 |
| Mean MAE difference (RF − SVR) | +0.0175 |
| 95% bootstrap confidence interval | [−0.0048, 0.0397] |
| Bootstrap replicates favoring SVR | 93.54% |

Although SVR achieved a lower observed MAE, the confidence interval included zero. I therefore cannot claim a statistically clear performance advantage.

I also learned that the bootstrap estimates sampling variability rather than uncertainty from retraining. Molecular similarities and correlated assay measurements may make a molecule-level bootstrap interval optimistic.

### 2. Molecule-level error comparison

I compared the absolute errors of both models on every test molecule.

- SVR performed better on **618 molecules**.
- RF performed better on **638 molecules**.
- The median paired error difference was **−0.0033**, slightly favoring RF.
- The mean difference was **+0.0175**, favoring SVR.

This showed that SVR's lower overall MAE did not result from winning on more molecules. Instead, the magnitude of its improvements outweighed its losses.

I also excluded molecules with the largest absolute disagreements as a diagnostic sensitivity analysis:

| Comparison | Mean MAE difference |
|---|---:|
| All molecules | +0.0175 |
| Excluding largest 1% disagreements | +0.0171 |
| Excluding largest 5% | +0.0047 |
| Excluding largest 10% | +0.0059 |

The SVR advantage became substantially smaller after removing the largest 5–10% of disagreements. I retained all molecules in the official evaluation.

### 3. Investigation of extreme prediction errors

I inspected the five molecules where SVR most improved upon RF and the five where RF performed best relative to SVR.

Several large disagreements involved molecules with **pIC50 = 14.301030**, corresponding to an implied IC50 of **0.000005 nM (5 femtomolar)**.

I checked the dataset and found **19 molecules with pIC50 ≥ 12**, including **17 molecules with exactly pIC50 = 14.301030**.

The repeated, exceptionally high potency values raised a potential data-quality concern.

### 4. ChEMBL provenance audit

I retrieved the original ChEMBL activity records for two representative molecules and then expanded the audit to all 17 compounds sharing the extreme label.

All 17 records had:

- **Standard IC50:** 0.000005 nM
- **Standard relation:** `=`
- **Assay:** CHEMBL4342697
- **Document:** CHEMBL4342404
- **Data validity comment:** `Outside typical range`

This confirmed that the extreme values were present in ChEMBL before my pIC50 conversion. They were not introduced by my modeling pipeline.

However, all 17 values originated from one assay and carried the same validity warning. This suggests a shared source-specific data-quality concern, although it does not prove the measurements are incorrect.

## What I Learned

The most important lesson was that **a lower average test error does not automatically establish that one model is reliably superior**. RF performed better on slightly more individual molecules, while SVR benefited from larger improvements on a subset.

I also learned the importance of examining individual prediction failures rather than relying exclusively on aggregate metrics.

Finally, I saw how source-data quality can influence molecular ML results. A small number of extreme activity labels may have a disproportionate impact on model evaluation, particularly RMSE.

## Conclusions

Tanimoto-SVR retained a modest observed test MAE advantage over RF, but the paired bootstrap confidence interval included zero. I therefore consider the two models competitive rather than declaring a decisive winner.

The data-quality audit identified **17 extreme potency labels from a single ChEMBL assay, all flagged as outside the typical range**. This is a concrete limitation that must be documented in the final portfolio.

I will preserve the frozen test metrics and treat any exclusion of these records as a clearly labeled **post hoc sensitivity analysis**, not a replacement for the original evaluation.

## Next Steps — Day 30

1. Quantify how the 17 flagged extreme-label compounds affect test MAE and RMSE without retraining or changing the primary results.
2. Compare training-set similarity and scaffold overlap between validation and test molecules.
3. Consolidate findings on chemical generalization, activity cliffs, uncertainty, and dataset limitations for the portfolio report.
