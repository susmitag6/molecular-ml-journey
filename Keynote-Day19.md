# Day 19 (13.09.2026)— Chirality, Molecular Graph Representations, and GNN Predictive Performance

**Project 1:** Predicting human acetylcholinesterase (AChE) inhibitory activity using molecular machine learning  
**Focus:** Investigating whether explicit stereochemical information improves graph neural network predictions

## 1. Research question and motivation

In previous experiments, I trained a graph neural network (GNN) to predict AChE inhibitory activity from molecular structures. My original model used atom and bond features derived from RDKit, followed by graph message passing and global mean pooling.

The original GNN performed worse than my Morgan-fingerprint-based random forest. This motivated me to examine whether the graph representation was missing chemically relevant information.

One important omission was **chirality**.

Two molecules can have identical atomic connectivity but different three-dimensional arrangements. Such stereoisomers may interact differently with a biological target because proteins are themselves chiral.

My main research question for Day 19 was:

**Does adding explicit chirality information to the molecular graph improve AChE pIC50 prediction, especially for molecules containing assigned stereocenters?**

My initial hypothesis was that the chirality-aware GNN would perform better on chiral molecules, even if the improvement in overall validation performance was small.

---

## 2. Identifying the limitation of my original graph representation

My original molecular graph contained:

- **34 atom features**, encoding element identity, degree, formal charge, hybridization, aromaticity, and hydrogen count.
- **7 bond features**, encoding bond type, conjugation, and ring membership.
- Graph connectivity represented by `edge_index`.

These features describe molecular composition and connectivity but do not explicitly represent tetrahedral stereochemistry.

To test the limitation, I constructed two stereoisomers of alanine:

```python
smiles_a = "C[C@H](N)C(=O)O"
smiles_b = "C[C@@H](N)C(=O)O"
```

RDKit recognized the structures as different stereoisomers, with opposite assigned configurations.

However, converting them using my original `smiles_to_graph()` function produced identical atom features, bond features, and connectivity.

This revealed a fundamental representational limitation:

**If two stereoisomers have identical input graphs, a deterministic GNN must produce identical predictions for them, regardless of training quality.**

The model cannot learn a distinction that its input representation does not contain.

---

## 3. Adding chirality features to the graph

I extended the atom representation by encoding RDKit's atom chiral tags as four one-hot features:

1. `CHI_UNSPECIFIED`
2. `CHI_TETRAHEDRAL_CW`
3. `CHI_TETRAHEDRAL_CCW`
4. `CHI_OTHER`

The atom feature dimension increased from **34 to 38**.

I implemented a new graph conversion function, `smiles_to_graph_chiral()`, that preserved the original graph representation and appended the chirality features to each atom.

When I repeated the alanine test:

- Both graphs still contained six atoms and five bonds.
- Their graph connectivity remained identical.
- Their atom feature matrices were now different.

This demonstrated that the new representation could distinguish the two input structures.

**Important limitation:** RDKit's clockwise and counterclockwise chiral tags depend on local atom-neighbor ordering. They are not equivalent to absolute R/S stereochemical descriptors. Therefore, my encoding was a simple exploratory feature extension, not a fully robust stereochemical representation.

---

## 4. Auditing stereochemistry in the AChE dataset

My cleaned AChE dataset contained **6,287 molecules**, each associated with an experimental pIC50 value.

I used RDKit to identify molecules containing assigned tetrahedral stereocenters.

The audit showed:

| Dataset property | Result |
|---|---:|
| Total molecules | 6,287 |
| Molecules with assigned tetrahedral chirality | 968 |
| Percentage with assigned chirality | 15.4% |
| Total assigned tetrahedral centers | 3,162 |
| Mean assigned centers per affected molecule | 3.27 |

In the scaffold-disjoint validation set:

| Subgroup | Molecules |
|---|---:|
| Assigned chirality | 145 |
| No assigned chirality | 785 |
| Total | 930 |

Thus, only about **15.6% of the validation molecules** contained assigned tetrahedral stereochemistry.

This mattered because any improvement specific to stereochemistry might have a limited effect on overall validation metrics.

### Initial subgroup observation

Using the earlier GNN, I found that the chiral subgroup actually had lower prediction error than the nonchiral subgroup:

| Subgroup | MAE | RMSE |
|---|---:|---:|
| Assigned chirality | 0.775 | 1.000 |
| No assigned chirality | 0.956 | 1.225 |

This contradicted a simple expectation that chiral molecules would necessarily be harder to predict.

However, the chiral subgroup also had a narrower pIC50 distribution, so the lower error could not be attributed to stereochemistry alone.

**Lesson:** Comparing errors across chemically different subgroups can be confounded by differences in target distributions and molecular composition.

---

## 5. Preparing a controlled model comparison

I generated the chirality-aware graph dataset successfully:

| Dataset | Number of graphs |
|---|---:|
| Complete dataset | 6,287 |
| Training set | 4,099 |
| Validation set | 930 |

Each new graph contained **38 atom features**.

I then prepared two models:

- **Baseline GNN:** Original 34-feature representation.
- **Chirality-aware GNN:** Extended 38-feature representation.

Both models used:

- Two GINE message-passing layers.
- Hidden dimension of 64.
- Global mean pooling.
- A linear regression head.
- Adam optimization.
- Learning rate of 0.001.
- Weight decay of 0.0001.
- Maximum 60 epochs.
- Early stopping with patience 10.
- The same scaffold-disjoint training and validation molecules.
- Random seed 42.

I also created seeded DataLoaders to control training-data shuffling.

The model parameter counts were:

| Model | Parameters |
|---|---:|
| Baseline GNN | 19,457 |
| Chirality-aware GNN | 19,713 |

The chirality-aware model had **256 additional parameters**, approximately 1.3% more than the baseline.

This was a relatively small change in model capacity, although the two models were not perfectly parameter-matched.

I also recognized that using the same random seed does not guarantee identical initialization of shared layers when the input dimensions differ.

Therefore, this was a **single-seed controlled comparison**, not yet a statistically robust multi-seed experiment.

---

## 6. Training results: Did chirality improve overall performance?

The training experiment produced:

| Metric | Baseline GNN | Chirality-aware GNN |
|---|---:|---:|
| Best epoch | 53 | 60 |
| Validation MSE | **1.2640** | 1.3362 |
| Validation RMSE | **1.1243** | 1.1559 |

The chirality-aware model performed slightly worse.

Its validation RMSE increased by approximately **2.8%**.

The best checkpoint for the chirality-aware model occurred at epoch 60, the maximum allowed training epoch. This suggested that its optimization might not have fully stabilized, although additional training would not necessarily improve generalization.

At this stage, the result did not support the claim that adding chirality features improved overall prediction accuracy.

However, I still hypothesized that the model might perform better specifically on molecules with assigned stereochemistry.

---

## 7. Testing performance on chiral and nonchiral molecules

I evaluated both trained models separately on the 145 validation molecules with assigned chirality and the remaining 785 molecules.

| Validation subgroup | N | Baseline MAE | Chiral GNN MAE | Difference |
|---|---:|---:|---:|---:|
| Assigned chirality | 145 | **0.812** | 0.842 | +0.030 |
| No assigned chirality | 785 | **0.895** | 0.907 | +0.013 |
| All molecules | 930 | **0.882** | 0.897 | +0.016 |

A positive difference means the chirality-aware model performed worse.

My hypothesis was **not supported**.

The chirality-aware model had higher MAE in both subgroups, including the molecules for which the additional stereochemical features were expected to be most useful.

This result led to several possible explanations:

1. Chirality may not provide strong additional predictive information for much of this dataset.
2. The simple chiral-tag encoding may be insufficient for learning stereochemical effects.
3. The available training data may provide limited examples of activity differences between stereoisomers.
4. The observed performance differences may partly reflect random initialization and optimization variability.

I could not distinguish these explanations using overall and subgroup MAE alone.

---

## 8. Looking for matched stereoisomers in the dataset

I realized that **having a stereocenter does not automatically imply that chirality strongly affects a molecule's measured potency**.

A more direct test would compare molecules with identical connectivity but different specified stereochemistry.

I therefore removed stereochemical information from copies of the RDKit molecules and grouped the original records by their stereochemistry-free canonical SMILES.

This identified:

| Property | Result |
|---|---:|
| Connectivity groups with multiple stereochemical representations | 138 |
| Molecules in those groups | 319 |

Some groups contained both specified stereoisomers and molecules with unspecified stereochemistry.

I learned that an unspecified stereochemical structure should not automatically be treated as an additional stereoisomer. It may represent an incompletely specified structure or a mixture.

I therefore restricted the next analysis to molecules with assigned tetrahedral stereochemistry.

---

## 9. Quantifying observed activity differences between stereoisomers

After filtering for groups containing multiple distinct structures with assigned tetrahedral stereochemistry, I obtained:

| Property | Result |
|---|---:|
| Groups with multiple specified stereoisomers | 57 |
| Molecules in those groups | 120 |
| Median within-group pIC50 range | 0.403 |
| 90th percentile range | 2.192 |
| Maximum range | 3.088 |

The relationship between pIC50 difference and IC50 ratio is:

\[
\frac{\mathrm{IC}_{50,\mathrm{higher}}}{\mathrm{IC}_{50,\mathrm{lower}}}
=10^{|\Delta\mathrm{pIC}_{50}|}
\]

Thus:

- A median difference of 0.403 corresponds to approximately **2.5-fold** in IC50.
- A difference of 2.192 corresponds to approximately **156-fold**.
- A difference of 3.088 corresponds to approximately **1,225-fold**.

This was an important result.

Although the median difference was moderate, some connectivity-matched groups contained very large reported activity differences.

Therefore, stereochemistry could be biologically important for certain compounds in the dataset, even though adding chiral features did not improve the overall GNN.

**Caveat:** These were within-group ranges, not necessarily pure enantiomer effects. Groups could include diastereomers, multiple stereocenters, or other stereochemical distinctions. Differences in experimental measurements might also contribute.

---

## 10. Identifying held-out stereoisomer groups

I next checked which of the specified stereoisomers appeared in the scaffold-disjoint validation set.

The results were:

| Validation property | Result |
|---|---:|
| Specified stereoisomer molecules | 11 |
| Connectivity groups represented | 8 |
| Groups containing at least two specified stereoisomers | **3** |

Only three groups contained multiple specified stereoisomers in validation.

This was sufficient for a small diagnostic experiment, but not for a statistically reliable evaluation of stereoselectivity prediction.

I then aligned the experimental pIC50 values with predictions from both trained GNNs.

---

## 11. Directly comparing stereoisomer potency differences

For each of the three held-out groups, I calculated the absolute difference between the two measured pIC50 values and compared it with the difference predicted by each model.

| Group | Observed ΔpIC50 | Baseline predicted Δ | Chiral predicted Δ |
|---|---:|---:|---:|
| 1 | 0.221 | 0.000 | 0.432 |
| 2 | 0.766 | 0.000 | 0.051 |
| 3 | 0.039 | 0.000 | 0.433 |

### Group 1

The observed pIC50 values were 5.302 and 5.523.

The baseline GNN predicted 6.971 for both structures, producing exactly zero predicted difference.

The chirality-aware GNN predicted 6.706 and 7.138, producing a difference of 0.432.

The model recognized a difference between the inputs but overestimated its magnitude and predicted the wrong potency ordering.

### Group 2

The observed pIC50 values were 5.222 and 4.456, a difference of 0.766.

The baseline again predicted identical values.

The chirality-aware model predicted 5.906 and 5.855, a difference of only 0.051.

It substantially underestimated the observed activity difference.

This pair involved a change in phosphorus stereochemistry, emphasizing the limitations of simple chiral-tag features and conventional message passing.

### Group 3

The observed pIC50 values were 8.161 and 8.200, a very small difference of 0.039.

The baseline predicted identical values of 7.145.

The chirality-aware model predicted 6.525 and 6.958, a difference of 0.433.

It predicted a much larger difference than observed and again assigned the wrong potency ordering.

### Interpretation

The baseline GNN produced identical predictions for each connectivity-matched stereoisomer pair, consistent with its inability to represent their stereochemical differences.

The chirality-aware GNN produced different predictions for all three pairs.

However, it did not correctly predict the magnitude or direction of the observed potency differences.

This distinction became the central lesson of the day:

**Stereochemical sensitivity is not the same as stereochemical predictive accuracy.**

---

## 12. Final interpretation

My initial hypothesis was:

**Explicit chirality features will improve AChE potency prediction, particularly for molecules containing assigned stereochemistry.**

The results of this experiment did not support that hypothesis.

The evidence was:

- The chirality-aware model had worse overall validation RMSE and MAE.
- It also had worse MAE on the assigned-chirality subgroup.
- It distinguished three held-out stereoisomer pairs that the baseline could not distinguish.
- Nevertheless, its predicted stereoisomer activity differences were inaccurate, including incorrect potency ordering.

I therefore conclude:

**Adding RDKit chiral-tag features made my GNN sensitive to stereochemical differences, but it did not improve predictive performance in this single-seed AChE experiment.**

This does not mean stereochemistry is unimportant for AChE inhibition.

Instead, it suggests that simply appending chirality flags to atom features is not necessarily sufficient to learn stereochemical structure–activity relationships.

The dataset also contains relatively few matched stereoisomer examples, limiting how well I can evaluate this question.

---

## 13. Limitations of the experiment

Several limitations prevent stronger conclusions.

**Single random seed:** I compared the models using one training run each. Repeated runs would be needed to determine whether the observed MAE difference is reproducible.

**Small stereoisomer test set:** Only three held-out connectivity groups contained multiple specified stereoisomers. This is too small for robust statistical conclusions.

**Simplified stereochemical representation:** Local RDKit CW/CCW tags are sensitive to atom-neighbor ordering and do not provide a fully invariant description of molecular stereochemistry.

**Architecture limitations:** A conventional two-dimensional GINE network does not explicitly model molecular three-dimensional geometry or the spatial orientation of substituents.

**Experimental uncertainty:** Measured activity differences may be influenced by assay variation, compound identity, stereochemical purity, or other data-quality factors.

**Unequal parameter counts:** The chirality-aware model had 256 more parameters than the baseline. Although this difference was small, the comparison was not perfectly capacity-matched.

---

## 14. What I learned

The most important conceptual lessons from Day 19 were:

1. **Molecular representation determines what a model can learn.** If stereoisomers produce identical input graphs, a deterministic GNN cannot distinguish them.

2. **Adding information is not the same as using information effectively.** The chirality-aware model received different stereochemical inputs but did not learn accurate activity differences.

3. **Overall model metrics can hide chemically important behavior.** Subgroup analysis and matched-molecule comparisons are useful for investigating specific representation limitations.

4. **Matched molecular comparisons are more informative than broad chemical categories.** Comparing stereoisomers with identical connectivity provides a more direct test of stereochemical sensitivity than comparing all chiral molecules against all nonchiral molecules.

5. **Negative results are scientifically valuable.** My original hypothesis was plausible but was not supported. Understanding why an intervention fails is part of rigorous model development.

6. **Predictive sensitivity, accuracy, and generalization are distinct concepts.** A model can react to a chemical feature without learning the correct relationship between that feature and biological activity.

7. **Small validation subsets require cautious interpretation.** Three stereoisomer pairs can reveal model behavior but cannot establish general performance.

---

## 15. Relevance to computational drug discovery

Stereochemistry is especially important in drug discovery because biological targets are three-dimensional and chiral.

Two stereoisomers may differ in binding affinity, selectivity, metabolism, or toxicity despite having identical connectivity.

However, this experiment showed that including stereochemical labels in a molecular ML model is not sufficient by itself.

To model stereochemical effects reliably, I may eventually need:

- More matched stereoisomer activity data.
- Stereochemically consistent molecular representations.
- Architectures designed to capture stereochemical relationships.
- Potentially three-dimensional conformational information.
- Evaluation protocols that specifically test stereoisomer ranking and activity differences.

For my current portfolio project, the main priority remains developing a reliable and well-validated AChE activity prediction workflow rather than optimizing stereochemical modeling on a very small subset.

---

## 16. Next steps

I will preserve this investigation as a **chirality-feature ablation study** in Project 1.

I do not need to continue tuning the chirality-aware model immediately because the present dataset provides limited matched-stereoisomer validation evidence.

My next priorities are to understand why the Morgan-fingerprint random forest outperforms the GNN, investigate generalization under scaffold-aware validation, and develop a stronger overall evaluation framework.

### Final takeaway

**Day 19 taught me that chemically richer inputs do not automatically produce better molecular predictions. A useful representation must not only encode the relevant chemistry; the model must also learn how that chemistry relates to the experimental property.**

This experiment transformed a simple feature-engineering hypothesis into a more rigorous investigation of representation, stereochemical sensitivity, predictive accuracy, and the limitations of molecular ML datasets.
