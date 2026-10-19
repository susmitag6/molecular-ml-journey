# Day 26 - Building an Applicability Domain for AChE Bioactivity Prediction

## Today's Objective

Today, I investigated whether combining **Random Forest prediction uncertainty** with **chemical similarity to the training set** could help identify molecules for which my AChE bioactivity model makes more reliable predictions.

My hypothesis was that molecules with low Random Forest tree disagreement and high similarity to training compounds would have lower prediction errors.

## What I Did

**1. Defined a combined applicability-domain rule**

Using the results from Days 23–24, I defined two criteria:

- Nearest-training Tanimoto similarity ≥ **0.714**
- Random Forest tree disagreement ≤ **0.758**

A molecule was considered inside the proposed applicability domain only if it satisfied both conditions.

**2. Evaluated prediction accuracy inside and outside the domain**

| Group | Molecules | Coverage | MAE |
|---|---:|---:|---:|
| Inside applicability domain | 306 | 32.9% | **0.446** |
| Outside applicability domain | 624 | 67.1% | 0.766 |
| Full validation set | 930 | 100% | 0.661 |

The molecules inside the applicability domain had approximately **32.5% lower MAE** than the full validation set.

This showed that the combined rule could identify a subset of molecules with more accurate predictions.

**3. Compared individual and combined reliability indicators**

I compared three selection strategies:

| Selection method | Coverage | MAE |
|---|---:|---:|
| Low tree disagreement only | 50.0% | 0.483 |
| High chemical similarity only | 50.5% | 0.577 |
| Combined rule | 32.9% | **0.446** |

Tree disagreement was a stronger individual reliability indicator than chemical similarity.

However, the combined rule selected fewer molecules, so its lower MAE alone was not sufficient to establish superiority.

**4. Performed a matched-coverage comparison**

To make the comparison fair, I selected the same number of molecules (306) using either the combined rule or tree disagreement alone.

| Selection method | Coverage | MAE |
|---|---:|---:|
| Tree disagreement only | 32.9% | 0.476 |
| Combined rule | 32.9% | **0.446** |

At identical coverage, the combined rule achieved approximately **6.3% lower MAE**.

This suggested that chemical similarity provides additional useful information beyond tree disagreement.

## What I Learned

Today, I learned how to construct and evaluate a preliminary **applicability domain (AD)** for a molecular machine learning model.

An applicability domain helps identify predictions that may be more reliable based on how well a molecule is represented by the training data and how consistently the model predicts its activity.

I also learned the importance of the **accuracy–coverage trade-off**. Selecting
