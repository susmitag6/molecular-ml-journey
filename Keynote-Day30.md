# Research Journal - Day 30
**Project 1: Human Acetylcholinesterase (AChE) Bioactivity Prediction**  
**Topic:** Data Quality, Chemical Similarity, and Scaffold Generalization  
**Status:** Completed

## 1. Objectives

Today, I investigated the reliability and chemical generalization of my Random Forest (RF) and Tanimoto-SVR models.

My objectives were to:

1. Quantify the influence of extreme ChEMBL activity labels on test performance.
2. Compare the chemical similarity of validation and test molecules to the training set.
3. Evaluate model errors separately for test molecules with seen and unseen molecular scaffolds.
4. Identify limitations that must be communicated in the final scientific report.

All analyses used previously trained, frozen models. I did not retrain or tune either model.

## 2. Experiment 1 — Sensitivity to Extreme Activity Labels

### Hypothesis

On Day 29, I identified 17 molecules with an unusually high pIC50 value of 14.301030, corresponding to an IC50 of approximately 5 femtomolar.

All 17 records originated from ChEMBL assay `CHEMBL4342697` and carried the data validity warning `Outside typical range`.

I hypothesized that these extreme labels would contribute disproportionately to model errors, especially RMSE.

### Method

I identified the affected molecules within the clean test set and recalculated RF and Tanimoto-SVR metrics after excluding them.

This was a post hoc sensitivity analysis. The original frozen test results remained unchanged.

### Results

Four of the 1,256 clean test molecules belonged to the flagged group.

| Metric | Original test (n=1,256) | Excluding flagged (n=1,252) |
|---|---:|---:|
| RF MAE | 0.5654 | 0.5597 |
| RF RMSE | 0.8155 | 0.7962 |
| RF R² | 0.6970 | 0.6797 |
| SVR MAE | 0.5480 | 0.5389 |
| SVR RMSE | 0.7889 | 0.7656 |
| SVR R² | 0.7165 | 0.7038 |

For the four flagged molecules alone:

| Metric | RF | Tanimoto-SVR |
|---|---:|---:|
| MAE | 2.3547 | 3.3828 |
| RMSE | 3.2281 | 3.4588 |

### Interpretation

Although the four flagged compounds represented only 0.32% of the clean test set, they produced much larger errors than the average test molecule.

Excluding them reduced RF RMSE by approximately 2.4% and SVR RMSE by approximately 3.0%.

Interestingly, R² decreased despite improvements in MAE and RMSE. I understood that this can happen because removing extreme activity values also reduces the variance of the target distribution, which changes the denominator in the R² calculation.

RF performed better than SVR on the four flagged molecules, whereas SVR remained better on the overall test set.

The flagged records are not necessarily incorrect, but their shared assay origin and ChEMBL validity warning warrant caution.

**Conclusion:** A very small group of extreme activity records can disproportionately influence model evaluation. The sensitivity analysis did not change the overall model ranking.

## 3. Experiment 2 — Validation versus Test Chemical Similarity

### Hypothesis

Both models performed better on the clean random test set than on scaffold-disjoint validation.

I hypothesized that the test molecules were, on average, more chemically similar to the training molecules.

### Method

Using radius-2, 2048-bit Morgan fingerprints, I compared each validation and test molecule with its most similar training molecule using Tanimoto similarity.

I summarized the resulting nearest-training similarity distributions.

### Results

| Metric | Validation (n=930) | Clean test (n=1,256) |
|---|---:|---:|
| Mean similarity | 0.7067 | 0.7700 |
| Median similarity | 0.7143 | 0.7931 |
| 25th percentile | 0.5949 | 0.6798 |
| 75th percentile | 0.8261 | 0.8837 |
| Fraction with similarity below 0.5 | 13.8% | 7.8% |
| Fraction with similarity ≥ 0.85 | 22.0% | 32.8% |

The test-set median similarity was 0.0788 higher than the validation median.

The corresponding model performances were:

| Model | Validation MAE | Clean test MAE |
|---|---:|---:|
| Random Forest | 0.661 | 0.565 |
| Tanimoto-SVR | 0.651 | 0.548 |

### Interpretation

The random test set contained more molecules that were structurally similar to the training data.

In particular, 32.8% of test molecules had nearest-training similarity of at least 0.85, compared with 22.0% of validation molecules.

This supports the explanation that the random test set was chemically easier than the scaffold-disjoint validation set.

However, chemical similarity alone does not establish causality. Differences in target-value distributions, assay composition, and scaffold overlap could also contribute.

**Conclusion:** Random test performance should not be interpreted as equivalent to performance on genuinely novel chemical scaffolds.

## 4. Experiment 3 — Seen versus Unseen Scaffold Generalization

### Hypothesis

I expected molecules with Murcko scaffolds absent from training to have larger prediction errors than molecules with previously observed scaffolds.

### Method

I generated chirality-agnostic Murcko scaffolds for training and clean test molecules.

I then divided the test set into two groups:

- **Seen scaffolds:** The scaffold appeared in the training set.
- **Unseen scaffolds:** The scaffold did not appear in the training set.

I compared RF and SVR MAE and nearest-training Tanimoto similarity between these groups.

### Results

| Metric | Seen scaffolds | Unseen scaffolds |
|---|---:|---:|
| Number of molecules | 736 | 520 |
| Fraction of clean test | 58.6% | 41.4% |
| RF MAE | 0.5160 | 0.6355 |
| SVR MAE | 0.4971 | 0.6199 |
| Median nearest-training similarity | 0.8198 | 0.7246 |

Relative to seen-scaffold compounds:

- RF MAE increased by approximately **23.2%** on unseen scaffolds.
- SVR MAE increased by approximately **24.7%** on unseen scaffolds.

### Interpretation

Both models were less accurate on molecules with unseen scaffolds.

This provides direct evidence of a scaffold-novelty penalty within the clean test set.

The unseen-scaffold molecules also had lower nearest-training similarity, connecting reduced predictive accuracy with reduced chemical familiarity.

However, an unseen Murcko scaffold does not necessarily imply a completely unfamiliar molecule. The median nearest-training similarity for this group was still 0.7246.

I also recognized that this was a post hoc subgroup analysis of a random test split, not an independently constructed scaffold-disjoint test benchmark.

**Conclusion:** Both models showed reduced accuracy on unseen scaffolds, reinforcing the importance of chemically informed evaluation splits.

## 5. Overall Scientific Findings

Today's analyses led to four main conclusions.

**First, source-data quality matters.** A small number of extreme activity records can contribute disproportionately to prediction errors. These records require careful documentation rather than automatic deletion.

**Second, test-set composition strongly affects apparent model performance.** The random test set was more chemically similar to training than the scaffold-disjoint validation set, providing a plausible explanation for its lower prediction errors.

**Third, scaffold novelty presents a measurable generalization challenge.** Both models exhibited approximately 23–25% higher MAE on unseen-scaffold compounds.

**Fourth, Tanimoto-SVR retained a modest descriptive advantage over RF.** It achieved lower MAE on the full test set and in both scaffold subgroups. However, the Day 29 paired-bootstrap 95% confidence interval for the overall MAE difference included zero, so I cannot claim a conclusive statistical advantage.

## 6. What I Learned

Today, I strengthened my understanding of the difference between model accuracy and model generalization.

A strong test score does not necessarily mean that a model will perform equally well on new chemical series.

I also learned that:

- Data provenance and assay-level warnings are important for interpreting extreme bioactivity labels.
- RMSE is particularly sensitive to large individual prediction errors.
- R² can decrease even when MAE and RMSE improve because the target distribution changes.
- Nearest-neighbor chemical similarity helps characterize the applicability of molecular ML models.
- Scaffold-stratified error analysis reveals limitations hidden by aggregate test metrics.
- Post hoc sensitivity analyses must remain separate from the primary frozen evaluation.

## 7. Final Conclusion

Day 30 provided a more realistic assessment of the strengths and limitations of my AChE bioactivity models.

Although Random Forest and Tanimoto-SVR achieved good performance on the clean random test set, both showed reduced accuracy on chemically less familiar molecules and unseen scaffolds.

The results reinforced an important principle of molecular machine learning:

**Predictive accuracy must be interpreted in the context of chemical novelty, data quality, and evaluation design.**

These findings will form an important part of the limitations and generalization discussion in my final Project 1 portfolio report.

## 8. Next Steps — Day 31

**Reproducibility and Pipeline Engineering**

I will begin organizing the project into a reproducible structure, including:

- Data processing and molecular feature generation
- Saved train, validation, and test indices
- Frozen RF and Tanimoto-SVR models
- Evaluation and applicability-domain calculations
- Dependency specifications and reproducibility checks

The goal is to turn my research notebooks into a clean, auditable, portfolio-ready scientific ML project.
