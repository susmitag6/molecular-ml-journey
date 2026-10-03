# Day 13 : Applicability Domain & Prediction Uncertainty
**3 October 2026**

## Research Question

Today I moved from asking whether the model generalizes to asking:

**When the model makes a prediction for a new molecule, how much should I trust that prediction?**

I investigated two potential indicators of prediction reliability:

1. Chemical-space familiarity, measured by maximum count-Morgan Tanimoto similarity to the training set.
2. Random Forest tree disagreement, measured by the standard deviation of predictions across the 300 individual trees.

## Applicability Domain

A molecule that is structurally similar to molecules in the training set is more likely to lie within the chemical space represented during model training.

However, high fingerprint similarity only tells me that the molecule is well represented according to the chosen molecular representation and similarity metric.

It does not guarantee an accurate activity prediction.

Activity cliffs may arise from information inadequately represented by the fingerprints, including 3D binding geometry, conformational behavior, stereochemistry, protonation state, or experimental and assay variability.

Therefore:

$$
\boxed{
\text{Inside applicability domain}
\neq
\text{prediction guaranteed correct}
}
$$

## Experiment 1 — Chemical-Space Familiarity

For every molecule in the random test set, I calculated its maximum count-Morgan Tanimoto similarity to any molecule in the training set.

Before the experiment, I predicted that the median maximum similarity would be relatively high, approximately 0.7–0.9, because random splitting can place related chemical series in both training and test sets.

### Results

| Statistic | Maximum training similarity |
|---|---:|
| Minimum | 0.213 |
| Median | **0.830** |
| Mean | 0.805 |
| Maximum | 1.000 |

The median similarity of 0.830 supported my hypothesis.

Most test molecules were relatively close to at least one training molecule according to the count-Morgan/Tanimoto representation, although some test molecules were considerably more distant.

## Experiment 2 — Does Similarity Predict Error?

I next tested whether molecules with higher maximum training similarity tended to have smaller absolute prediction errors.

My hypothesis was:

$$
\text{higher similarity} \rightarrow \text{lower prediction error}
$$

### Results

$$
r_{\mathrm{Pearson}}=-0.207
$$

$$
\rho_{\mathrm{Spearman}}=-0.143
$$

with:

$$
p_{\mathrm{Spearman}}\approx3.6\times10^{-7}
$$

Both correlations were negative, supporting the expected direction.

However, the magnitude of the relationship was weak. Therefore, nearest-training similarity contains some information about prediction reliability, but it is not sufficient by itself to determine whether an individual prediction should be trusted.

A high Tanimoto similarity threshold should therefore not automatically be interpreted as a guarantee of prediction accuracy.

Instead, similarity is one piece of evidence about whether a molecule resembles the chemical space represented in the training data.

## Experiment 3 — Random Forest Tree Disagreement

A Random Forest prediction is the average prediction from many individual decision trees.

For each test molecule, I calculated the standard deviation of the predictions from all 300 trees:

$$
\sigma_{\mathrm{trees}} = \mathrm{std} (\hat y_1,\hat y_2,\ldots,\hat y_{300})
$$

My initial hypothesis was that greater disagreement would correspond to larger prediction errors, but I expected the relationship to be relatively weak and perhaps similar to nearest-neighbor similarity.

### Distribution of Tree Disagreement

| Statistic | Tree disagreement |
|---|---:|
| Minimum | 0.099 |
| Median | 0.663 |
| Mean | 0.708 |
| Maximum | 3.249 |

I then compared tree disagreement with absolute prediction error.

### Results

$$
r_{\mathrm{Pearson}}=0.433
$$

$$
\rho_{\mathrm{Spearman}}=0.437
$$

with:

$$
p_{\mathrm{Spearman}}\approx6.8\times10^{-60}
$$

Tree disagreement therefore showed a substantially stronger association with prediction error than nearest-training similarity did in this test set.

Greater disagreement among the Random Forest trees tended to correspond to larger prediction errors.

However:

$$
\boxed{
\text{Model agreement}
\neq
\text{model correctness}
}
$$

All trees use the same training data and molecular representation. If important information is absent from that representation, many trees could agree while still making an incorrect prediction.

Therefore, tree disagreement should not automatically be interpreted as a calibrated prediction uncertainty.

## Experiment 4 — Error Across Disagreement Quartiles

To make the relationship easier to interpret, I divided the 1,258 test molecules into four approximately equal groups according to their tree disagreement.

| Disagreement group | n | Mean tree disagreement | MAE | Median absolute error |
|---|---:|---:|---:|---:|
| Q1 - lowest | 315 | 0.357 | **0.271** | 0.164 |
| Q2 | 314 | 0.563 | **0.387** | 0.273 |
| Q3 | 314 | 0.771 | **0.561** | 0.448 |
| Q4 - highest | 315 | 1.142 | **0.840** | 0.678 |

The MAE increased monotonically:

$$ 0.271 < 0.387 < 0.561 < 0.840 $$

Before the experiment, I predicted that the highest-disagreement group might have approximately 1.5 times the MAE of the lowest-disagreement group.

Instead, I observed:

$$ \frac{MAE_{Q4}}{MAE_{Q1}} = \frac{0.840}{0.271} \approx3.1
$$

Thus, the highest-disagreement quartile had approximately **3.1 times the MAE** of the lowest-disagreement quartile in this test set.

This provides empirical evidence that tree disagreement is useful for **ranking predictions by reliability** for this particular model and evaluation set.

It does not establish that tree standard deviation is a calibrated prediction interval.

For example, a prediction reported as:

$$
\hat{pIC50}=7.5,\qquad\sigma_{\mathrm{trees}}=0.5
$$

should not automatically be interpreted as:

$$
pIC50=7.5\pm0.5
$$

with a defined statistical coverage probability.

## Comparison of Reliability Signals

The two reliability indicators provided different information.

**Nearest-training similarity** asks:

> How familiar is this molecule relative to the chemical space represented in training?

**Tree disagreement** asks:

> How consistently do the individual trees make predictions for this molecule?

For this experiment:

$$
\rho(\text{similarity},|\text{error}|) = -0.143
$$

whereas:

$$
\rho(\text{tree disagreement},|\text{error}| = +0.437
$$

Tree disagreement was therefore more informative about actual prediction error than nearest-neighbor similarity alone.

## Main Conclusion

Prediction reliability cannot be reduced to a single measure such as a Tanimoto similarity threshold.

For this AChE model, chemical-space similarity provided a weak signal of prediction reliability, while Random Forest tree disagreement showed a stronger relationship with observed prediction error.

Activity-cliff analysis from Day 12 also showed that high structural similarity does not guarantee similar bioactivity.

A more informative assessment of prediction reliability should therefore consider multiple pieces of evidence:

$$
\boxed{ \text{chemical-space familiarity} + \text{model disagreement} + \text{activity-cliff awareness} }
$$

The results also demonstrate an important distinction between **ranking predictions by apparent reliability** and constructing **calibrated uncertainty estimates**. Tree disagreement was useful for the former, but has not yet been demonstrated to provide the latter.
