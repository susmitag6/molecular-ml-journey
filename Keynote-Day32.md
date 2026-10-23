# Research Journal : Day 32
**Project 1:** Human Acetylcholinesterase (AChE) Bioactivity Prediction  
**Topic:** Scientific Visualization, Prediction Bias, Applicability Domains, and Chemical Generalization  
**Status:** Completed

## 1. Objectives

Today, I focused on converting my validated molecular machine learning results into clear, scientifically interpretable figures and a final model-comparison table.

My objectives were to:

- Visualize predicted versus experimental pIC50.
- Quantify systematic prediction bias across activity ranges.
- Analyze residual distributions and large prediction errors.
- Evaluate the relationship between applicability-domain coverage and accuracy.
- Investigate prediction error as a function of chemical similarity.
- Compare performance on seen and unseen molecular scaffolds.
- Present Random Forest, Tanimoto-SVR, and GINE results without mixing validation and test protocols.

I used the previously frozen model predictions and evaluation procedures. I did not retrain or optimize any models.

## 2. Experiment 1 — Predicted Versus Observed pIC50

I generated a two-panel scatter plot comparing predicted and experimental pIC50 for Random Forest and Tanimoto-SVR on the clean held-out test set.

The test set contained **1,256 molecules**.

| Model | Test MAE | Test RMSE | Test R² |
|---|---:|---:|---:|
| Random Forest | 0.565 | 0.816 | 0.697 |
| Tanimoto-SVR | **0.548** | **0.789** | **0.716** |

Both models captured the overall activity trend, with predictions broadly following the perfect-prediction diagonal.

However, I observed systematic deviations at the extremes. Weak inhibitors tended to have overpredicted pIC50, while strong inhibitors tended to have underpredicted pIC50.

Several large outliers were also visible, including molecules with unusually high experimental pIC50 values.

**Interpretation:** The models captured meaningful structure–activity relationships but showed regression toward the central activity range. Their overall accuracy metrics concealed activity-dependent prediction errors.

I retained the extreme observations in the primary analysis rather than silently excluding them, while recognizing the ChEMBL data-quality concerns identified on earlier days.

**Output:** `figure_01_predicted_vs_observed.png`

## 3. Experiment 2 — Systematic Prediction Bias

To quantify the visual pattern, I calculated signed prediction errors:

\[
e_i=\hat{y}_i-y_i
\]

Positive errors indicate overprediction of pIC50, and negative errors indicate underprediction.

I divided the test molecules into four equally sized quartiles based on experimental pIC50, with 314 molecules per quartile.

### Results

| Activity quartile | RF mean error | SVR mean error | RF MAE | SVR MAE |
|---|---:|---:|---:|---:|
| Q1 — lowest | +0.560 | +0.476 | 0.639 | 0.597 |
| Q2 | +0.154 | +0.149 | 0.391 | 0.411 |
| Q3 | −0.166 | −0.135 | 0.462 | 0.489 |
| Q4 — highest | −0.600 | −0.471 | 0.769 | 0.695 |

The mean signed error changed progressively from positive in the lowest-activity quartile to negative in the highest-activity quartile.

Tanimoto-SVR showed smaller mean bias in the extreme quartiles, whereas Random Forest had slightly lower MAE in the middle two quartiles.

I generated a grouped bar chart of mean signed error by activity quartile.

**Interpretation:** The analysis quantitatively supported regression toward the mean. It also showed that SVR's overall advantage was not uniform across activity ranges.

These were descriptive subgroup comparisons. They did not establish the cause of the bias.

**Output:** `figure_02_signed_error_by_activity_quartile.png`

## 4. Experiment 3 — Residual Distributions

I plotted overlapping residual histograms for the two frozen models and calculated summary statistics.

### Results

| Statistic | Random Forest | Tanimoto-SVR |
|---|---:|---:|
| Mean signed error | −0.0127 | +0.0049 |
| Median signed error | +0.0113 | +0.0199 |
| Residual standard deviation | 0.8154 | 0.7889 |
| Fraction with absolute error above 1 | 18.23% | 14.97% |
| Fraction with absolute error above 2 | 2.39% | 2.63% |

Both residual distributions were concentrated near zero, with substantial overlap and extended tails.

Tanimoto-SVR had a lower residual standard deviation and fewer errors exceeding one pIC50 unit. However, Random Forest had a slightly smaller fraction of errors exceeding two units.

The overall mean signed errors were close to zero, despite the systematic biases found across activity quartiles.

**Interpretation:** Positive and negative subgroup biases can cancel in an overall mean. SVR reduced moderately large errors but did not consistently eliminate the most extreme errors.

Because pIC50 is logarithmic, a one-unit absolute error corresponds to a tenfold difference in the implied IC50 value.

**Output:** `figure_02_residual_distributions.png`

*Note: The residual figure was initially saved with the `figure_02` prefix, which overlaps with the numbering used for the signed-error figure. I will standardize figure filenames and numbering during final repository cleanup.*

## 5. Experiment 4 — Applicability-Domain Evaluation

I revisited the applicability-domain criteria established using validation data.

The frozen combined rule required:

- Random Forest tree disagreement no greater than 0.7581356.
- Nearest-training-molecule Tanimoto similarity at least 0.7142857.

I applied these unchanged thresholds to the clean held-out test set.

### Results

| Group | Molecules | Coverage | RF MAE |
|---|---:|---:|---:|
| Full test set | 1,256 | 100% | 0.5654 |
| Inside combined applicability domain | 636 | 50.64% | 0.3956 |
| Outside combined applicability domain | 620 | 49.36% | 0.7397 |
| Uncertainty-only, matched coverage | 636 | 50.64% | 0.3758 |

The frozen combined applicability domain identified a subset with approximately **30% lower MAE** than the full test set.

Its inside-domain MAE was approximately **46.5% lower** than its outside-domain MAE.

However, retrospective selection of the 636 molecules with the lowest RF disagreement achieved an even lower MAE of 0.3758.

I created a bar chart comparing full-test performance, the combined applicability domain, and uncertainty-only selection at matched coverage.

**Interpretation:** Selective prediction can identify lower-error subsets, but adding chemical similarity to uncertainty did not improve accuracy in this matched-coverage test comparison.

The uncertainty-only cutoff was determined retrospectively using the test-set ranking. Therefore, this comparison is exploratory and must not be described as an independently frozen prospective selection policy.

I also recognized the practical trade-off: lower MAE was achieved by retaining predictions for only about half the molecules.

**Output:** `figure_04_applicability_domain_mae.png`

## 6. Experiment 5 — Chemical Similarity and Prediction Error

I investigated whether prediction accuracy depended on the maximum Morgan fingerprint Tanimoto similarity between each test molecule and the training set.

I divided the 1,256 test molecules into four similarity quartiles.

### Results

| Similarity quartile | Median Tanimoto | RF MAE | SVR MAE |
|---|---:|---:|---:|
| Q1 — least similar | 0.575 | 0.754 | 0.742 |
| Q2 | 0.750 | 0.529 | 0.505 |
| Q3 | 0.833 | **0.451** | **0.421** |
| Q4 — most similar | 0.981 | 0.528 | 0.524 |

The least-similar molecules had the highest errors for both models.

However, the relationship was not monotonic. Prediction errors decreased through Q3 and increased again in Q4, despite the extremely high median similarity in that quartile.

I created a grouped bar chart showing MAE across similarity quartiles.

**Interpretation:** Chemical similarity was informative about prediction difficulty, but high similarity did not guarantee accurate bioactivity prediction.

Possible explanations include activity cliffs, assay heterogeneity, and molecular representation limitations. The present analysis did not determine which mechanism caused the Q4 increase.

The results also helped explain why similarity alone was insufficient as an applicability-domain criterion.

**Output:** `figure_05_similarity_vs_error.png`

## 7. Experiment 6 — Scaffold Novelty and Generalization

I next examined whether the test molecules shared chirality-agnostic Bemis–Murcko scaffolds with the training set.

### Results

| Scaffold group | Molecules | RF MAE | SVR MAE |
|---|---:|---:|---:|
| Seen scaffolds | 736 | 0.5160 | 0.4971 |
| Unseen scaffolds | 520 | 0.6355 | 0.6199 |

Approximately **58.6%** of the clean test molecules had scaffolds seen during training, while **41.4%** had unseen scaffolds.

Relative to seen scaffolds, prediction error increased by:

- **23.2%** for Random Forest.
- **24.7%** for Tanimoto-SVR.

I generated a grouped bar chart comparing both models across the two scaffold groups.

**Interpretation:** Both models generalized less accurately to previously unseen molecular frameworks.

This result complemented the chemical-similarity analysis by providing a discrete measure of structural novelty.

Importantly, this was a retrospective subgroup analysis within the random held-out test set, not a separately designed scaffold-disjoint test evaluation.

**Output:** `figure_06_scaffold_generalization.png`

## 8. Experiment 7 — Validation Model Comparison

I created a final validation figure comparing Random Forest, Tanimoto-SVR, and the frozen four-layer GINE graph neural network.

I used only the **930-molecule chirality-aware scaffold-disjoint validation set** to ensure that the three models were evaluated on the same molecules.

### Results

| Model | Validation MAE |
|---|---:|
| Tanimoto-SVR | **0.651** |
| Random Forest | 0.661 |
| 4-layer GINE | 0.813 |

Tanimoto-SVR achieved the lowest validation MAE, followed closely by Random Forest.

The GINE model had approximately **25% higher MAE than Tanimoto-SVR**.

**Interpretation:** The more complex graph neural network did not outperform the fingerprint-based models under the specific architecture, training procedure, and validation conditions used in this project.

This is a useful negative result, not evidence that GNNs are generally inferior for molecular property prediction.

I also preserved the methodological distinction that the validation split was disjoint under a chirality-aware scaffold definition. Earlier auditing identified some scaffold overlap when chirality was ignored.

The GNN was not evaluated on the frozen clean held-out test set.

**Output:** `figure_07_validation_model_comparison.png`

## 9. Experiment 8 — Final Model Results Table

I consolidated the main model results into a machine-readable CSV file:

`results/ache_final_model_metrics.csv`

### Final comparison

| Model | Validation MAE | Test MAE | Test RMSE | Test R² |
|---|---:|---:|---:|---:|
| Random Forest | 0.661 | 0.565 | 0.816 | 0.697 |
| Tanimoto-SVR | **0.651** | **0.548** | **0.789** | **0.716** |
| 4-layer GINE | 0.813 | Not evaluated | Not evaluated | Not evaluated |

The table also recorded each model's molecular representation and whether a held-out test evaluation was available.

For the GNN, the test metrics were stored as missing values with `test_evaluated=False`.

**Interpretation:** The table provides a consistent reference for the final report and README while preventing misleading comparisons between validation-only and held-out test results.

The stored metrics are rounded summaries of previously established results.

## 10. Main Scientific Findings

Today's figures allowed me to move beyond reporting a single predictive accuracy score.

I established that:

1. Both classical models captured meaningful AChE structure–activity relationships but showed activity-dependent bias.
2. Tanimoto-SVR achieved modestly better overall test performance, particularly by reducing errors above one pIC50 unit.
3. The difference between RF and SVR remained small, and the earlier paired bootstrap confidence interval for their MAE difference included zero.
4. Applicability-domain selection substantially reduced prediction error at the cost of reduced coverage.
5. Combining similarity and uncertainty did not outperform retrospective uncertainty-only selection at matched coverage.
6. Low chemical similarity was associated with larger prediction errors, but the relationship was not monotonic.
7. Unseen molecular scaffolds were more difficult to predict than seen scaffolds.
8. The four-layer GINE did not outperform the fingerprint-based baselines on the validation set.

These findings demonstrate why model evaluation should include **bias, error distributions, chemical novelty, and applicability-domain behavior**, rather than relying only on MAE or R².

## 11. Limitations

I identified several limitations that must remain explicit in the final scientific report.

**Random held-out test split:** The primary test set is not scaffold-disjoint, and many test molecules share scaffolds with training molecules. Its performance may be optimistic for structurally novel compounds.

**Validation scaffold definition:** The validation split used chirality-aware scaffolds; chirality-agnostic scaffold overlap was identified during auditing.

**Assay-data quality:** A small group of unusually high pIC50 records originated from ChEMBL measurements flagged as outside the typical range. These observations were retained in the primary results.

**Model-comparison uncertainty:** Although SVR achieved lower observed MAE, the paired bootstrap confidence interval for the test MAE difference included zero.

**Post hoc subgroup analyses:** Similarity quartiles, activity quartiles, and scaffold novelty comparisons were descriptive analyses rather than independently validated model-selection procedures.

**GNN evaluation:** The GINE model has reproducible validation results but no frozen held-out test evaluation.

**Applicability-domain comparison:** The uncertainty-only matched-coverage result is retrospective and should not be presented as a prospective threshold policy.

## 12. Conclusion

Day 32 was devoted to transforming frozen model evaluations into a coherent scientific narrative.

I produced figures demonstrating overall prediction accuracy, activity-dependent bias, residual behavior, applicability-domain trade-offs, chemical-similarity effects, scaffold generalization, and validation model comparisons.

The central conclusion is that **fingerprint-based molecular ML models provided useful AChE bioactivity predictions, but reliability decreased for chemically novel molecules and at extreme activity values**.

Tanimoto-SVR achieved the lowest observed error, although its advantage over Random Forest was modest and statistically uncertain under the earlier bootstrap analysis.

The GNN comparison showed that a more complex architecture did not automatically improve predictive performance.

I also learned that effective scientific visualization should expose model limitations rather than simply emphasize favorable metrics.

## 13. Next Steps — Day 33

My next task is to organize Project 1 into a professional, reproducible GitHub portfolio.

I will focus on:

- Auditing and organizing source code, datasets, checkpoints, configurations, and figures.
- Standardizing figure filenames and captions.
- Writing the README and final scientific report.
- Documenting dataset provenance, model architectures, evaluation protocols, and limitations.
- Ensuring that reported results remain traceable to frozen artifacts.

After the repository is organized, I will perform a final reproducibility and quality-assurance review before beginning Project 2, which will integrate molecular dynamics, Markov state models, and geometric machine learning.
