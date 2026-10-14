# Day 20 — Diagnosing GNN Underfitting and Improving Molecular Activity Prediction

**Project 1:** Machine Learning for Human Acetylcholinesterase (AChE) Bioactivity Prediction  
**Focus:** GNN model capacity, message-passing depth, optimization, and generalization

## 1. Research question

In previous experiments, I observed that my Morgan-fingerprint Random Forest substantially outperformed my graph neural network (GNN) in predicting AChE inhibitory activity.

My original GNN used two GINE message-passing layers, 64 hidden features, and global mean pooling.

The Random Forest achieved a validation MAE of approximately **0.653 pIC50 units**, whereas my original GNN performed considerably worse.

This raised an important question:

**Why does a relatively simple fingerprint-based Random Forest outperform a graph neural network that learns molecular representations directly from atomic connectivity?**

I considered several possible explanations:

- The GNN might have insufficient representational capacity.
- Two message-passing layers might not capture enough molecular context.
- The model might not have been trained for long enough.
- The GNN might be overfitting or failing to generalize across molecular scaffolds.
- Morgan fingerprints might provide a more effective representation for this dataset.

Rather than immediately testing many architectural modifications, I decided to diagnose the existing model systematically.

My initial hypothesis was:

**The original GNN is underfitting the training data, and increasing model capacity should improve its predictive performance.**

---

## 2. Diagnosing the original GNN

I evaluated the two-layer GNN on both the training and validation datasets.

The dataset contained:

| Dataset | Molecules |
|---|---:|
| Training | 4,099 |
| Validation | 930 |

The training and validation sets were separated using a scaffold-aware split.

### Results

| Metric | Training | Validation |
|---|---:|---:|
| MAE | 0.874 | 0.882 |
| RMSE | 1.160 | 1.124 |
| R² | 0.426 | 0.426 |

The training and validation errors were very similar.

This was an important observation.

If the model were severely overfitting, I would expect a much lower training error and substantially higher validation error.

Instead, the model had relatively high error even on the training molecules.

Its training R² was only 0.426, indicating that it explained approximately 43% of the variance in the training pIC50 values.

### Interpretation

The results suggested that the original GNN was **underfitting**, or at least was limited by its architecture, representation, or optimization.

The similar training and validation errors were not sufficient to prove underfitting by themselves. However, combined with the relatively poor training performance, they provided a reasonable basis for testing a higher-capacity model.

**Key lesson:** Before trying to improve generalization, I should first determine whether a model can adequately fit the training data.

---

## 3. Hypothesis: Increasing message-passing depth

My original GNN contained two GINE convolutional layers.

In a conventional message-passing GNN, each layer allows atoms to aggregate information from their immediate neighbors.

Consequently:

- After one layer, an atom receives information from approximately one bond away.
- After two layers, it can incorporate information from approximately two bonds away.
- After four layers, its receptive field can extend approximately four bonds away.

Morgan fingerprints with radius 2 explicitly encode local atomic environments up to two bonds away.

Although the receptive fields may appear similar, the representations are fundamentally different.

Morgan fingerprints explicitly construct structural identifiers, while a GNN must learn how to transform and combine local molecular information.

I hypothesized that increasing the GNN depth from two to four layers would improve its ability to fit the training data and possibly improve validation performance.

---

## 4. Constructing the four-layer GNN

I implemented a new model called `AChEGNNDeep`.

The architecture contained:

1. A linear atom encoder transforming 34 atom features into 64 hidden features.
2. A linear bond encoder transforming 7 bond features into 64 hidden features.
3. Four GINE message-passing layers.
4. ReLU activation after each message-passing layer.
5. Global mean pooling to produce a molecular representation.
6. A linear regression layer predicting pIC50.

The hidden dimension remained 64.

The primary architectural change was increasing the number of message-passing layers from two to four.

### Model sizes

| Architecture | Parameters |
|---|---:|
| Two-layer GNN | 19,457 |
| Four-layer GNN | 36,097 |

The deeper GNN contained approximately **86% more trainable parameters**.

This meant that the experiment changed both message-passing depth and total model capacity.

Therefore, any performance improvement could not be attributed exclusively to the larger receptive field.

---

## 5. Training the deeper GNN

I trained the four-layer GNN using the same training and validation molecules as the original model.

The training configuration was:

| Hyperparameter | Value |
|---|---|
| Random seed | 42 |
| Hidden dimension | 64 |
| Message-passing layers | 4 |
| Optimizer | Adam |
| Learning rate | 0.001 |
| Weight decay | 0.0001 |
| Maximum epochs | 60 |
| Early-stopping patience | 10 |
| Batch size | 64 |
| Loss function | Mean squared error |

The best validation checkpoint occurred at epoch 60.

The validation MSE was 1.1977.

### Results after 60 epochs

| Metric | Training | Validation |
|---|---:|---:|
| MAE | 0.792 | 0.840 |
| RMSE | 1.060 | 1.094 |
| R² | 0.520 | 0.457 |

Compared with the original two-layer model, the deeper GNN improved both training and validation performance.

### Comparison

| Metric | Two-layer GNN | Four-layer GNN |
|---|---:|---:|
| Training MAE | 0.874 | **0.792** |
| Validation MAE | 0.882 | **0.840** |
| Training R² | 0.426 | **0.520** |
| Validation R² | 0.426 | **0.457** |

Training MAE decreased by approximately 9.4%.

Validation MAE decreased by approximately 4.8%.

### Interpretation

The improvement in training performance supported my hypothesis that the original architecture had insufficient effective capacity under the training setup.

The deeper network could fit the training data better.

Importantly, some of this improvement transferred to the scaffold-disjoint validation set.

However, the validation improvement was smaller than the training improvement.

A generalization gap also began to appear:

- Training R²: 0.520
- Validation R²: 0.457

This suggested that increasing model capacity improved fitting but did not eliminate the challenge of predicting activities for unseen molecular scaffolds.

---

## 6. Investigating whether 60 epochs were sufficient

The four-layer GNN achieved its best validation performance at epoch 60, which was also the maximum training epoch.

This raised another question:

**Was the deeper GNN still improving when training stopped?**

I formulated a second hypothesis:

**The four-layer GNN has not fully converged after 60 epochs, and additional optimization may improve predictive performance.**

Rather than increasing the number of layers again, I continued training the existing model.

I first preserved the 60-epoch checkpoint.

Then I fine-tuned the model using:

- Learning rate: 0.0003
- Weight decay: 0.0001
- Maximum additional epochs: 40
- Early-stopping patience: 10

The reduced learning rate allowed smaller parameter updates during the additional optimization stage.

This was a continuation of the existing training run, not an independent retraining experiment.

---

## 7. Fine-tuning results

The best checkpoint occurred after **22 additional epochs**, corresponding to approximately 82 total epochs.

The validation MSE improved from 1.1977 to 1.1186.

### Results after fine-tuning

| Metric | Training | Validation |
|---|---:|---:|
| MAE | 0.757 | 0.813 |
| RMSE | 1.014 | 1.058 |
| R² | 0.561 | 0.492 |

The model improved on both the training and validation sets.

### Comparison across the three stages

| Metric | Two-layer GNN | Four-layer GNN, 60 epochs | Fine-tuned four-layer GNN |
|---|---:|---:|---:|
| Training MAE | 0.874 | 0.792 | **0.757** |
| Validation MAE | 0.882 | 0.840 | **0.813** |
| Validation RMSE | 1.124 | 1.094 | **1.058** |
| Validation R² | 0.426 | 0.457 | **0.492** |

Compared with the original two-layer GNN:

- Training MAE improved by approximately **13.4%**.
- Validation MAE improved by approximately **7.8%**.
- Validation R² increased from **0.426 to 0.492**.

### Interpretation

The additional optimization improved the deeper model's performance.

This supported the hypothesis that the original 60-epoch training limit was premature for this model and optimization schedule.

However, I cannot conclude that longer training alone caused the improvement because I also reduced the learning rate.

The experiment tested a combined intervention: **continued training with a smaller learning rate**.

---

## 8. Understanding the generalization gap

After fine-tuning, the model achieved:

$$
\mathrm{MAE}_{train}=0.757
$$

$$
\mathrm{MAE}_{validation}=0.813
$$

The difference was:

$$
\Delta\mathrm{MAE}=0.813-0.757=0.056
$$

This was a modest generalization gap.

The model fit the training molecules more accurately than the validation molecules, but the difference was not extremely large.

The main remaining problem was that the training error itself was still substantial.

Therefore, I concluded that the model's limitations were not simply due to severe overfitting.

Possible remaining factors included:

- Limitations of the molecular graph representation.
- The effectiveness of global mean pooling.
- The GNN's ability to learn useful molecular substructures.
- The relatively limited size of the dataset.
- Noise and uncertainty in experimental IC50 measurements.
- Differences between training and validation molecular scaffolds.

These explanations remain hypotheses rather than established causes.

---

## 9. Comparison with the Morgan-fingerprint Random Forest

The Random Forest remained the strongest model among the candidates evaluated so far.

| Model | Validation MAE |
|---|---:|
| Morgan fingerprint + Random Forest | **0.653** |
| Original two-layer GNN | 0.882 |
| Four-layer GNN, 60 epochs | 0.840 |
| Fine-tuned four-layer GNN | **0.813** |

The fine-tuned GNN improved substantially relative to the original GNN, but its validation MAE remained approximately 0.160 pIC50 units higher than the Random Forest.

This result demonstrated that greater neural network complexity did not automatically lead to better molecular activity predictions.

The fingerprint-based Random Forest remained a strong baseline.

One possible explanation is that Morgan fingerprints provide useful local molecular descriptors directly, whereas the GNN must learn an effective molecular representation from data.

However, the experiments conducted today did not isolate this explanation from differences in model architecture, training, regularization, or feature engineering.

I therefore cannot conclude that Morgan fingerprints are inherently superior to GNNs.

I can only conclude that **the fingerprint Random Forest performed better than the GNN configurations tested so far on this development dataset**.

---

## 10. Model selection and evaluation methodology

An important methodological point emerged during the experiments.

I repeatedly used the validation set to:

- Compare the two-layer and four-layer architectures.
- Select the best training checkpoint.
- Evaluate the benefit of additional optimization.
- Decide whether to continue tuning the GNN.

This means the validation set is being used as a **development set**.

Its performance estimates may become optimistic as I make more modeling decisions based on them.

Therefore, I should not treat the best validation MAE as an unbiased estimate of final generalization performance.

For the portfolio project, I must preserve the original held-out test set and use it only after model selection is complete.

The final comparison should evaluate the selected Random Forest and GNN models on the same untouched test molecules.

I should also verify that preprocessing and model selection do not introduce data leakage.

---

## 11. Final conclusions

Day 20 addressed two hypotheses.

### Hypothesis 1: The original GNN has insufficient effective capacity.

**Supported by the experiment.**

Increasing message-passing depth from two to four layers reduced training MAE from 0.874 to 0.792 and validation MAE from 0.882 to 0.840.

This showed that the original model's performance could be improved through increased architectural capacity.

However, the experiment changed both depth and parameter count, so their individual contributions remain unresolved.

### Hypothesis 2: The deeper GNN needs additional optimization.

**Supported under the tested fine-tuning schedule.**

Continuing training with a smaller learning rate improved validation MAE from 0.840 to 0.813.

The model's best checkpoint occurred after approximately 82 total epochs.

### Overall conclusion

**The original GNN's weak performance was partly attributable to limited effective capacity and an insufficient optimization schedule.**

Increasing depth and continuing training improved both training fit and scaffold-disjoint validation performance.

Nevertheless, the Random Forest remained substantially more accurate.

This suggests that additional GNN complexity alone may not be the most efficient path toward the strongest predictive model for this dataset.

---

## 12. What I learned

**1. Training and validation errors should be examined together.**

Validation performance alone cannot distinguish underfitting from overfitting.

The training–validation comparison helped me identify a plausible capacity or optimization limitation.

**2. Increasing model capacity can improve training fit and generalization.**

The deeper GNN reduced training error and achieved a smaller improvement in validation error.

**3. Better training performance does not guarantee equivalent validation improvement.**

The deeper model developed a modest generalization gap.

**4. Training duration and learning-rate scheduling matter.**

A model may appear to have reached its performance limit when the actual limitation is the training schedule.

**5. Strong classical ML baselines are essential.**

The Random Forest continued to outperform the GNN despite being simpler in several respects.

**6. Scientific experiments should isolate variables whenever possible.**

My depth experiment also increased parameter count, and my fine-tuning experiment changed both training duration and learning rate. These limitations should be reported explicitly.

**7. Validation data must not be confused with independent test data.**

Repeated model selection on the validation set means final performance must be measured on a separate held-out test set.

---

## 13. Next steps

I will retain the fine-tuned four-layer GNN as my current best GNN candidate.

Its checkpoint is intended to be saved as:

`ache_gnn_4layer_best.pt`

The next stage of Project 1 will focus on understanding the differences between the Random Forest and GNN predictions rather than continuing architecture tuning.

My next research questions are:

- Do the two models make errors on the same molecules?
- Are their errors related to molecular structure or scaffold novelty?
- Are certain activity ranges systematically difficult to predict?
- Does the GNN provide useful complementary information despite its higher overall MAE?

These analyses should help me understand not only which model performs better, but **where and why each model succeeds or fails**.

---

## Final takeaway

**Day 20 demonstrated that the performance of a molecular GNN depends strongly on both representational capacity and optimization.**

Increasing message-passing depth and extending training improved my AChE prediction model, reducing validation MAE from 0.882 to 0.813.

However, the Morgan-fingerprint Random Forest remained the stronger predictor, with a validation MAE of 0.653.

The broader lesson is that successful molecular machine learning requires more than selecting a sophisticated architecture. It requires diagnosing training behavior, testing clear hypotheses, maintaining strong baselines, and evaluating generalization carefully.
