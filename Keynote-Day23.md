
# Day 23 — Prediction Uncertainty and Model Reliability

**Project 1: AChE Bioactivity Prediction using Molecular Machine Learning**

## Today's Objective

Today, I investigated whether my Random Forest model can identify potentially unreliable molecular activity predictions before knowing the experimental pIC50 values.

Previously, my Random Forest achieved an MAE of 0.661 pIC50 on 930 scaffold-disjoint validation molecules. However, this overall error does not tell me which individual predictions are trustworthy.

My hypothesis was that **molecules with greater disagreement among the Random Forest's decision trees would tend to have larger prediction errors**.

## What I Did

**1. Understood how the Random Forest trees differ**

My Random Forest contains 300 decision trees trained using bootstrap samples of 4,099 training molecules. During validation, all 300 trained trees predict the activity of each of the 930 validation molecules.

I inspected the first five trees and found that they all used fingerprint feature 1098 at their root node. However, their subsequent splitting features differed. This helped me understand that trees can share important decision rules while still producing different predictions because of their different training samples and learned structures.

**2. Calculated uncertainty from tree disagreement**

For every validation molecule, I collected predictions from all 300 trees and calculated their standard deviation:

$$
u(x)=\operatorname{std}(\hat y_1(x),\ldots,\hat y_{300}(x))
$$

The resulting prediction matrix had shape `(930, 300)`.

The uncertainty distribution was:

- Minimum: 0.073 pIC50
- Median: 0.758 pIC50
- 90th percentile: 1.107 pIC50
- Maximum: 2.615 pIC50

This showed that some molecules produced much greater disagreement among the trees than others.

**3. Tested whether uncertainty correlates with prediction error**

I calculated the absolute error of each Random Forest prediction and compared it with the corresponding tree disagreement.

The Spearman correlation was **0.359**, with a p-value of **1.26 × 10⁻²⁹**, indicating a statistically significant, moderate positive association.

I also compared molecules in the lowest and highest uncertainty quartiles:

| Group | Molecules | MAE |
|---|---:|---:|
| Lowest uncertainty 25% | 233 | 0.456 |
| Highest uncertainty 25% | 233 | 0.992 |

The highest-uncertainty molecules had approximately **2.18 times larger prediction error** than the lowest-uncertainty molecules.

**4. Performed a risk–coverage analysis**

I sorted the validation molecules by increasing uncertainty and calculated the MAE as progressively more molecules were included.

| Molecules retained | MAE |
|---|---:|
| 25% | 0.456 |
| 50% | 0.483 |
| 75% | 0.551 |
| 100% | 0.661 |

The MAE increased as more uncertain molecules were included. Retaining the lowest-uncertainty 50% gave an MAE of 0.483, approximately **27% lower** than the overall MAE of 0.661.

This does not mean the model improved. Instead, the uncertainty estimate helped identify a subset of molecules on which the existing model performed better.

**5. Examined uncertainty across five bins**

I divided the validation molecules into five equally sized groups according to their uncertainty.

| Bin | Mean uncertainty | MAE |
|---|---:|---:|
| 1 — Lowest | 0.423 | 0.410 |
| 2 | 0.622 | 0.568 |
| 3 | 0.754 | 0.509 |
| 4 | 0.907 | 0.791 |
| 5 — Highest | 1.161 | 1.026 |

The highest-uncertainty bin had approximately 2.5 times the error of the lowest-uncertainty bin.

Although the general trend was positive, the relationship was not perfectly monotonic, confirming that tree disagreement is only an imperfect measure of reliability.

## What I Learned

The most important concept I learned today is the difference between **prediction error, uncertainty estimation, and uncertainty calibration**.

Prediction error measures the difference between predicted and experimental activity. Tree disagreement measures how much the individual Random Forest trees differ in their predictions. Calibration would determine whether an uncertainty estimate can support a meaningful probability or prediction interval.

My results show that tree disagreement is useful for ranking molecules by expected prediction difficulty, but it is **not a calibrated confidence interval**. All trees can agree and still make an incorrect prediction, especially for unfamiliar chemical structures or when important molecular information is missing.

I also learned that uncertainty can support different drug discovery decisions. Low-uncertainty, high-potency predictions may be useful for prioritizing promising compounds, while high-uncertainty compounds may be valuable for experimental testing and active learning.

## Conclusion

Today, I demonstrated that Random Forest tree disagreement provides useful information about the reliability of AChE bioactivity predictions.

The uncertainty–error Spearman correlation was 0.359, and the highest-uncertainty quartile had more than twice the prediction error of the lowest-uncertainty quartile. The risk–coverage analysis further showed that the lowest-uncertainty half of the validation set had substantially lower MAE than the complete set.

These findings are exploratory because I used the same validation set involved in model development, and the uncertainty estimates have not been formally calibrated.

**My main takeaway:** A molecular machine learning model should ideally provide not only a predicted activity but also information about how reliable that prediction may be.

**Next — Day 24:** Investigate whether similarity to the training molecules can help explain prediction errors and define the chemical applicability domain of my Random Forest model.
