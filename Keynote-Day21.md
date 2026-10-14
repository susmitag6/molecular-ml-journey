# Day 21 : Comparing Random Forest and GNN Prediction Errors and Testing Ensemble Learning

**Project 1:** Machine Learning for Human Acetylcholinesterase (AChE) Bioactivity Prediction  
**Focus:** Model comparison, error correlation, complementary predictions, and ensemble learning

## 1. Research question and motivation

In Day 20, I improved my graph neural network (GNN) by increasing message-passing depth and extending training with a lower learning rate.

Although these changes improved validation performance, the Morgan-fingerprint Random Forest (RF) remained substantially more accurate.

This raised a new question:

**Do the Random Forest and GNN make errors on the same molecules, or do they capture complementary information about molecular activity?**

A model can perform worse overall while still predicting certain molecules more accurately than a stronger model.

If two models make sufficiently different errors, combining their predictions may improve performance.

My hypothesis for Day 21 was:

**The Random Forest and GNN make partially complementary errors, and a weighted ensemble may outperform the Random Forest alone.**

I designed the day's experiments to test this hypothesis systematically.

---

## 2. Establishing a fair model comparison

Before comparing molecule-level errors, I needed to ensure that both models were evaluated on exactly the same compounds.

The fine-tuned four-layer GNN and its validation indices were available in my notebook. However, the previously trained Random Forest and its predictions were no longer available as notebook variables.

I therefore reconstructed a new Random Forest baseline.

### Molecular representation

I generated Morgan count fingerprints using RDKit with:

- Radius: 2
- Fingerprint size: 2,048
- Representation: Count-based molecular fingerprints

I used the same molecular SMILES and pIC50 targets as the GNN experiment.

### Dataset alignment

| Dataset | Molecules |
|---|---:|
| Training | 4,099 |
| Validation | 930 |

I used `gnn_train_idx` and `gnn_val_idx` to construct the Random Forest training and validation datasets.

I explicitly verified that the Random Forest validation targets matched the GNN validation targets.

The alignment check returned:

```text
Training fingerprints: (4099, 2048)
Validation fingerprints: (930, 2048)
Targets aligned: True
```

This was essential because molecule-level error comparisons would be invalid if the predictions corresponded to different compounds or different ordering.

**Key lesson:** A fair model comparison requires identical evaluation samples and correctly aligned predictions.

---

## 3. Reconstructing the Random Forest baseline

I trained a new `RandomForestRegressor` with the following settings:

| Hyperparameter | Value |
|---|---|
| Number of trees | 300 |
| Minimum samples per leaf | 2 |
| Maximum features | 1.0 |
| Random seed | 42 |
| Parallel processing | All available CPU cores |

This was a newly specified baseline, not necessarily an exact reproduction of my earlier Random Forest experiment.

Both the Random Forest and GNN were evaluated on the same 930-molecule scaffold-aware validation set.

### Validation results

| Metric | Random Forest | Four-layer GNN |
|---|---:|---:|
| MAE | **0.661** | 0.813 |
| RMSE | **0.907** | 1.058 |
| R² | **0.626** | 0.492 |

The Random Forest achieved approximately 18.7% lower MAE than the GNN.

### Interpretation

The Random Forest's advantage persisted when both models were compared on identical molecules.

This strengthened the conclusion that the RF was the better-performing model under the configurations tested.

However, overall metrics could not tell me whether the RF was more accurate for every individual molecule.

I therefore moved to molecule-level error analysis.

---

## 4. Calculating molecule-level prediction errors

For each validation molecule, I calculated the signed prediction error:

$$
e_i=\hat y_i-y_i
$$

where:

- \(y_i\) is the experimental pIC50.
- \(\hat y_i\) is the model-predicted pIC50.

I also calculated the absolute error:

$$
|e_i|=|\hat y_i-y_i|
$$

These quantities answer different questions.

**Absolute errors** measure the magnitude of prediction mistakes, regardless of direction.

**Signed errors** indicate whether a model overpredicts or underpredicts the experimental activity.

I then compared the Random Forest and GNN errors across all 930 validation molecules.

---

## 5. Correlation between model errors

I calculated Pearson correlations between the two models' absolute errors and signed errors.

### Results

| Error comparison | Pearson correlation |
|---|---:|
| Absolute errors | 0.634 |
| Signed errors | 0.783 |

### Absolute-error correlation

The absolute-error correlation was **0.634**, indicating a moderately strong positive relationship.

This means that molecules with large Random Forest errors also tended to have large GNN errors.

The models therefore shared some difficult prediction cases.

However, the correlation was not perfect, suggesting that the two models did not make identical errors.

### Signed-error correlation

The signed-error correlation was **0.783**, indicating a strong positive relationship.

This means that the models often made errors in similar directions.

For example, if the Random Forest substantially overpredicted the potency of a molecule, the GNN was also relatively likely to overpredict it.

This finding was particularly important for ensemble learning.

If two models make errors in opposite directions, averaging their predictions may cancel part of those errors.

However, when their signed errors are strongly positively correlated, averaging provides less opportunity for error cancellation.

### Scientific interpretation

The models had some complementary behavior, but they also shared substantial systematic prediction difficulties.

**Key lesson:** Different architectures do not necessarily produce independent errors.

---

## 6. Determining which model performs better for individual molecules

I compared the absolute errors of the two models for each validation compound.

For molecule \(i\):

- The Random Forest wins if its absolute error is smaller.
- The GNN wins if its absolute error is smaller.

### Results

| Better-performing model | Molecules | Percentage |
|---|---:|---:|
| Random Forest | 567 | 61.0% |
| GNN | 363 | 39.0% |
| Ties | 0 | 0% |

The Random Forest was more accurate on most molecules, consistent with its lower overall MAE.

However, the GNN performed better on **363 of the 930 validation molecules**.

This was an important observation.

Despite its weaker average performance, the GNN was not universally inferior.

It captured some relationships or produced some predictions that were more accurate than those of the Random Forest.

Nevertheless, this result alone did not establish that an ensemble would perform better.

The size and direction of the errors also mattered.

A model that wins on many molecules could still worsen an ensemble if its losses on other molecules are sufficiently large.

---

## 7. Hypothesis: A weighted ensemble may improve prediction accuracy

After identifying partially complementary errors, I tested whether combining the two models improved validation performance.

I used a simple weighted-average ensemble:

$$
\hat y_{\mathrm{ensemble}} =(1-w)\hat y_{\mathrm{RF}} + w\hat y_{\mathrm{GNN}}
$$

where:

- \(\hat y_{\mathrm{RF}}\) is the Random Forest prediction.
- \(\hat y_{\mathrm{GNN}}\) is the GNN prediction.
- \(w\) is the contribution of the GNN.

For example:

- \(w=0\): Random Forest only.
- \(w=0.1\): 90% RF and 10% GNN.
- \(w=0.5\): Equal contribution from both models.
- \(w=1\): GNN only.

I tested several predefined weights without training an additional meta-model.

This approach was intentionally simple because I wanted to determine whether ensemble learning was worthwhile before introducing more complexity.

---

## 8. Ensemble experiment results

I evaluated six different GNN weights.

| GNN weight | RF weight | Validation MAE | Validation RMSE |
|---|---:|---:|---:|
| 0.00 | 1.00 | 0.661 | 0.907 |
| 0.10 | 0.90 | **0.660** | **0.902** |
| 0.25 | 0.75 | 0.669 | **0.902** |
| 0.50 | 0.50 | 0.698 | 0.927 |
| 0.75 | 0.25 | 0.748 | 0.980 |
| 1.00 | 0.00 | 0.813 | 1.058 |

### Observation 1: A small GNN contribution produced a marginal improvement

The 90% RF / 10% GNN ensemble achieved:

$$
\mathrm{MAE}=0.660
$$

compared with:

$$
\mathrm{MAE}_{RF}=0.661
$$

The improvement was approximately **0.001 pIC50 units**.

RMSE decreased from 0.907 to 0.902.

These results suggested that the GNN contained a small amount of complementary predictive information.

However, the improvement was extremely small and might not be reproducible.

### Observation 2: Increasing the GNN contribution worsened MAE

At a GNN weight of 0.25, ensemble MAE increased to 0.669.

At equal weighting, MAE increased further to 0.698.

Larger GNN contributions produced progressively worse results.

This indicated that the weaker predictive performance of the GNN outweighed any complementary benefit when its contribution became substantial.

### Observation 3: RMSE and MAE did not select exactly the same weight

Both 10% and 25% GNN weights achieved an RMSE of approximately 0.902 at the displayed precision.

However, the 10% weight achieved a lower MAE.

This demonstrated that model selection can depend on the evaluation metric.

MAE measures average absolute prediction error, while RMSE gives greater weight to large errors.

---

## 9. Interpreting the ensemble results

My original hypothesis was that combining the Random Forest and GNN could improve predictive accuracy because the models made partially complementary errors.

The experiment provided **weak support** for this hypothesis.

A small GNN contribution improved the reported validation metrics slightly.

However, the gain was too small to justify a strong claim that ensemble learning was beneficial.

The strong signed-error correlation of 0.783 helped explain the limited improvement.

Although the GNN was better on 39% of molecules, both models frequently made errors in similar directions.

Therefore, combining their predictions offered relatively little error cancellation.

### Practical model-selection decision

I decided that the Random Forest should remain my preferred model for Project 1.

The fine-tuned GNN remained scientifically useful as a secondary model, but the simple ensemble did not offer a meaningful practical advantage.

A more complex model is not automatically preferable if it produces nearly identical performance while increasing computational and maintenance costs.

---

## 10. Important methodological limitations

### Validation-set model selection

I evaluated multiple ensemble weights on the same validation set.

Therefore, the best reported weight was selected using development data.

The resulting performance should not be interpreted as an unbiased estimate of final test performance.

I must evaluate the final selected approach on the untouched test set.

### Small performance improvement

The best ensemble improved MAE by only 0.001 pIC50 units.

This difference is negligible relative to the overall prediction error and may be sensitive to the validation sample or random seed.

### Correlation does not establish causation

The error correlations describe relationships between model errors.

They do not explain why particular molecules are difficult to predict.

Further analysis would be required to determine whether errors are associated with scaffold novelty, activity cliffs, molecular properties, or experimental uncertainty.

### Single model configurations

The experiment compared one newly trained Random Forest and one fine-tuned four-layer GNN.

It did not establish that all Random Forests outperform all GNNs or that other ensemble approaches would fail.

### Data quality

Both models used the same experimental pIC50 labels.

Shared measurement noise or inconsistent activity records could contribute to correlated prediction errors.

---

## 11. Final scientific conclusions

### Research question 1

**Does the Random Forest outperform the GNN when both are evaluated on identical molecules?**

**Answer: Yes.**

The Random Forest achieved a validation MAE of 0.661, compared with 0.813 for the GNN.

### Research question 2

**Do the models make complementary errors?**

**Answer: Partially.**

The GNN performed better on 39% of validation molecules, but the models' signed errors were strongly correlated.

### Research question 3

**Does combining the models improve prediction accuracy?**

**Answer: Only marginally in this experiment.**

A 90% RF / 10% GNN ensemble reduced validation MAE from 0.661 to 0.660.

The improvement was too small to establish a meaningful practical benefit.

### Overall conclusion

**The Morgan-fingerprint Random Forest remains the preferred AChE activity prediction model, while the GNN provides some complementary predictions but insufficient improvement to justify an ensemble at this stage.**

---

## 12. What I learned

**1. Overall metrics do not reveal the full behavior of a model.**

A weaker model can still outperform a stronger model on a substantial subset of molecules.

**2. Error correlation is useful for evaluating model complementarity.**

Absolute-error correlation identifies shared difficult cases, while signed-error correlation helps assess the potential for error cancellation.

**3. Winning on many individual samples does not guarantee ensemble improvement.**

The GNN was better on 39% of molecules, yet most ensemble weights worsened overall performance.

**4. Ensemble learning is not automatically beneficial.**

Combining models helps only when their predictions provide sufficiently complementary information relative to their individual errors.

**5. Model complexity should be justified by measurable gains.**

The RF-only model is simpler and performs almost identically to the best ensemble.

**6. Evaluation alignment is essential.**

Comparing per-molecule errors requires the same validation compounds in the same order.

**7. Validation performance is not final test performance.**

Model and ensemble selection must be separated from independent final evaluation.

---

## 13. Relevance to computational drug discovery

In drug discovery, a model is useful not only because it achieves a low average prediction error, but also because it helps researchers understand which compounds can be predicted reliably.

Two models may respond differently to molecular features, scaffolds, or activity changes.

Examining these differences can help identify:

- Molecular classes where predictions are unreliable.
- Compounds where models disagree.
- Potentially difficult structure–activity relationships.
- Opportunities for targeted experimental validation.
- Situations where ensemble learning may or may not provide value.

The Day 21 analysis demonstrated why model evaluation should extend beyond a single performance metric.

Understanding prediction errors is an essential part of building trustworthy molecular ML workflows.

---

## 14. Next steps — Day 22

The next stage will focus on **activity cliffs**.

An activity cliff occurs when structurally similar molecules exhibit unexpectedly large differences in biological activity.

This is especially relevant to medicinal chemistry because small structural changes can sometimes produce large changes in potency.

My next questions will be:

- How can I identify activity cliffs using molecular similarity and pIC50 differences?
- How common are activity cliffs in my AChE dataset?
- Do the Random Forest and GNN make larger errors near activity cliffs?
- Can activity-cliff analysis explain some of the remaining model limitations?

I will continue using focused, hypothesis-driven experiments rather than introducing unnecessary model complexity.

---

## Final takeaway

**Day 21 taught me that model comparison should investigate not only average predictive accuracy but also the relationship between individual prediction errors.**

The Random Forest was more accurate overall, but the GNN performed better on 39% of validation molecules.

Their errors were partially complementary, yet strongly correlated.

A small GNN contribution improved ensemble MAE by only 0.001 pIC50 units, providing no compelling practical reason to replace the simpler Random Forest.

The main scientific lesson was:

**A model's value depends not only on whether it makes different predictions, but on whether those differences lead to meaningful and reproducible improvements in predictive accuracy.**
