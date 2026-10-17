# Day 24 - Chemical Similarity and Applicability Domains

**Project 1: AChE Bioactivity Prediction using Molecular Machine Learning**

## Today's Objective

Today, I investigated whether the chemical similarity of validation molecules to the training dataset can help explain Random Forest prediction errors.

On Day 23, I found that disagreement among Random Forest trees was associated with larger prediction errors. Today, I wanted to determine whether **chemical novelty provides another useful indicator of prediction reliability**.

My hypothesis was that molecules less similar to the training compounds would generally have larger prediction errors.

## What I Did

**1. Calculated nearest-training molecular similarity**

I used Morgan binary fingerprints (radius 2, 2,048 bits) and Tanimoto similarity to compare each of the 930 validation molecules against all 4,099 training molecules.

For each validation molecule, I recorded its maximum similarity to any training molecule:

$$
S_i=\max_{j\in\mathrm{train}}T(F_i,F_j)
$$

The similarity distribution was:

| Statistic | Tanimoto similarity |
|---|---:|
| Minimum | 0.170 |
| 25th percentile | 0.595 |
| Median | 0.714 |
| 75th percentile | 0.826 |
| Maximum | 1.000 |

This showed that the validation set contains both relatively familiar and chemically distinct molecules.

A similarity of 1.0 indicates identical hashed fingerprint bit vectors, but does not necessarily mean identical molecular structures.

**2. Investigated the relationship between chemical similarity and prediction error**

I calculated the Spearman correlation between nearest-training similarity and Random Forest absolute prediction error.

Results:

- Spearman correlation: **−0.214**
- p-value: **4.64 × 10⁻¹¹**

I also compared the lowest- and highest-similarity quartiles:

| Group | Molecules | MAE |
|---|---:|---:|
| Lowest similarity 25% | 233 | 0.809 |
| Highest similarity 25% | 233 | 0.537 |

The least-similar molecules had approximately **51% higher prediction error** than the most-similar molecules.

This supported my hypothesis that chemically unfamiliar molecules tend to be more difficult for the Random Forest to predict. However, the correlation was relatively weak, indicating that chemical similarity alone cannot explain most prediction errors.

**3. Compared chemical similarity with Random Forest tree disagreement**

Next, I investigated whether chemically unfamiliar molecules also produce greater disagreement among the 300 Random Forest trees.

Results:

- Spearman correlation between similarity and tree disagreement: **−0.435**
- p-value: **3.63 × 10⁻⁴⁴**

| Chemical similarity group | Mean tree disagreement |
|---|---:|
| Lowest-similarity quartile | 0.917 |
| Highest-similarity quartile | 0.644 |

Molecules less similar to the training data tended to have greater tree disagreement.

This suggests that the Random Forest may produce less stable predictions for unfamiliar molecular structures.

**4. Tested whether similarity provides additional information beyond tree disagreement**

Finally, I divided the validation molecules into lower- and higher-tree-disagreement groups. Within each group, I compared the errors of molecules with lower and higher chemical similarity.

| Tree disagreement | Lower similarity MAE | Higher similarity MAE |
|---|---:|---:|
| Lower disagreement | 0.556 (162 molecules) | 0.444 (303 molecules) |
| Higher disagreement | 0.855 (306 molecules) | 0.806 (159 molecules) |

In both uncertainty groups, lower-similarity molecules had higher average errors.

The difference was larger in the lower-disagreement group (0.112 pIC50) than in the higher-disagreement group (0.049 pIC50).

These results suggest that chemical similarity may provide useful additional information about prediction reliability, even when tree disagreement is already known.

However, this simple grouping analysis does not prove that combining the two indicators would improve performance on an independent dataset.

## What I Learned

Today, I learned about the **chemical applicability domain** of a molecular machine learning model.

A model is generally expected to be more reliable when predicting compounds that resemble its training data, although structural similarity does not guarantee similar biological activity.

I also learned that chemical similarity and Random Forest tree disagreement describe related but different aspects of reliability:

- **Chemical similarity** measures how familiar a molecule is relative to the training dataset.
- **Tree disagreement** measures how consistently the trained Random Forest trees predict its activity.

My results showed that both are associated with prediction error, with tree disagreement showing the stronger individual correlation in this dataset.

An important limitation is that I have not established a validated similarity threshold defining reliable and unreliable predictions. The analysis was exploratory and used the same scaffold-disjoint validation set involved in model development.

## Conclusion

Today, I demonstrated that **chemical novelty is associated with reduced prediction reliability in my AChE Random Forest model**.

The nearest-training similarity had a Spearman correlation of −0.214 with absolute prediction error. Molecules in the lowest-similarity quartile had an MAE of 0.809, compared with 0.537 for the highest-similarity quartile.

Chemical similarity was also moderately negatively correlated with Random Forest tree disagreement (−0.435), suggesting that unfamiliar molecules tend to produce less consistent predictions.

The conditional analysis indicated that chemical similarity may offer complementary information beyond tree disagreement, although this requires further validation.

**My main takeaway:** Reliable molecular machine learning requires considering not only the predicted activity and model uncertainty, but also how well a new molecule is represented by the training chemistry.

**Next — Day 25:** Explore how chemical similarity and Random Forest tree disagreement can be combined into a practical prediction-reliability assessment.
