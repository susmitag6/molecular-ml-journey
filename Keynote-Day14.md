# Day 14 : From Model Performance to Scientific Model Evaluation
**4 October 2026**

## Main Theme

Today I consolidated the different components of the AChE bioactivity-prediction workflow:

$$
\boxed{ \text{Representation}
\rightarrow
\text{Validation strategy}
\rightarrow
\text{Chemical-space novelty}
\rightarrow
\text{Generalization}
\rightarrow
\text{Prediction reliability}
}
$$

The central lesson is that a single performance metric such as \(R^2\) is not sufficient to determine whether a molecular ML model is useful.

---

## 1. Random vs Scaffold-Aware Evaluation

Random splitting evaluates generalization to held-out molecules that may nevertheless be chemically similar to molecules in the training set and may belong to the same scaffold families.

Scaffold-aware evaluation instead prevents the same Murcko scaffold groups from appearing in both training and validation folds. It therefore evaluates a more challenging form of generalization to unseen scaffold groups.

For the AChE Random Forest model:

$$
R^2_{\mathrm{random\ CV}}\approx0.746
$$

whereas:

$$
R^2_{\mathrm{scaffold\ CV}}\approx0.565
$$

If the intended application is predicting AChE activity for molecules containing previously unseen scaffolds, the lower scaffold-CV score is more relevant to the deployment scenario than the better-looking random-CV score.

Therefore:

$$
\boxed{
\text{The best validation strategy is not the one giving the highest score.}
}
$$

Instead:

$$
\boxed{
\text{The validation strategy should approximate the intended deployment scenario.}
}
$$

An important distinction is that random CV does **not** mean that validation molecules themselves were seen during training. Rather, chemically related molecules and scaffold families may occur in both training and validation folds.

---

## 2. Generalization Gap

Comparing training and validation/test performance can help diagnose model behavior.

However, a small train–test performance gap does not automatically indicate a good model.

For example, a severely underfitting model can perform poorly on both training and test data and therefore have a small generalization gap.

Thus, both the absolute predictive performance and the train–test gap need to be considered.

---

## 3. Model Quality Depends on Intended Use

Suppose two models behave differently under random and scaffold-aware evaluation.

If the intended application involves predicting new scaffold families, I should prefer the model that performs better under scaffold-aware evaluation.

However, this does not imply that the same model is superior for every possible application.

If the task instead involves predicting additional molecules belonging to already represented chemical families, performance under a more interpolation-like evaluation may also be relevant.

Therefore:

$$
\boxed{\text{Best model} = \text{model performing best under evaluation relevant to deployment}}
$$

Model quality cannot be separated completely from the scientific problem the model is intended to solve.

---

## 4. Complexity Does Not Guarantee Scientific Improvement

A future GNN may obtain slightly better scaffold-CV performance than the current Random Forest.

For example:

$$
R^2_{\mathrm{RF}}=0.56
$$

versus:

$$
R^2_{\mathrm{GNN}}=0.60.
$$

This difference alone would not be sufficient to establish that the GNN is meaningfully better.

I would also examine whether the improvement is consistent across scaffold folds, whether MAE and RMSE improve, whether difficult molecules and activity cliffs are predicted better, and whether the improvement justifies the additional computational and methodological complexity.

Therefore:

$$
\boxed{
\text{More sophisticated model}
\neq
\text{automatically better scientific model}
}
$$

If scaffold-fold results vary substantially, I should also avoid making an overly strong statement such as:

> “The GNN outperforms the Random Forest.”

A more scientifically defensible conclusion would be that the GNN achieved a higher mean score, while fold-to-fold variability limits the strength of the claim that it consistently performs better.

---

## 5. What Should Be Reported for Deployment?

If a medicinal chemist asked whether the model could be trusted, I would not report only \(R^2\).

I would summarize four major aspects.

### 1. Generalization

Report performance under a validation strategy relevant to the intended application, particularly scaffold-aware validation when prediction on unseen scaffold families is important.

Random CV can be reported alongside scaffold CV to show how model performance changes as the evaluation becomes more chemically challenging.

Cross-validation variability should also accompany these results.

### 2. Absolute Prediction Error

Report metrics such as:

$$
MAE,\qquad RMSE
$$

in addition to \(R^2\).

These quantities provide a more direct indication of the typical magnitude of errors in pIC50 units.

### 3. Prediction Reliability

Consider whether a query molecule resembles the represented training chemical space and examine model-based reliability signals.

For the current Random Forest, lower tree disagreement was empirically associated with lower prediction error on our exploratory test set.

However, tree disagreement is a reliability-ranking signal and should not automatically be interpreted as a calibrated prediction uncertainty.

### 4. Failure Modes and Activity Cliffs

A useful model evaluation should explicitly investigate cases where the model fails.

Activity-cliff analysis showed that molecules with highly similar 2D fingerprints can nevertheless have very different experimental activities.

Therefore, high molecular similarity alone cannot guarantee accurate prediction.

---

## Final Perspective

A useful molecular ML model should not be summarized simply as:

$$
\boxed{R^2=0.75}
$$

Instead, the model should be evaluated through four complementary questions:

$$
\boxed{
\text{Generalization}
+
\text{Absolute error}
+
\text{Prediction reliability}
+
\text{Failure modes}
}
$$

The main lesson from Day 14 is that **model evaluation must reflect the scientific application**.

A lower but deployment-relevant performance estimate can be more valuable than an impressive metric obtained under an easier validation regime.

This framework will also provide the benchmark for evaluating more sophisticated models such as neural networks and GNNs. A new model should not be considered better merely because it is more complex; it should demonstrate a meaningful and robust improvement under scientifically relevant evaluation.
