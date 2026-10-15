# Day 22 - Activity Cliffs and Their Impact on AChE Bioactivity Prediction

**Project 1:** Machine Learning for Human Acetylcholinesterase (AChE) Bioactivity Prediction  
**Topic:** Structure–activity relationships, activity-cliff detection, and prediction-error analysis  
**Models:** Morgan Fingerprint Random Forest and Four-Layer Graph Neural Network

---

## 1. Research motivation

During Day 21, I established that the Morgan-fingerprint Random Forest outperformed my four-layer graph neural network (GNN) on the same scaffold-disjoint validation set.

The Random Forest achieved a mean absolute error (MAE) of 0.661 pIC50, compared with 0.813 for the GNN.

However, both models still made substantial prediction errors.

This raised an important scientific question:

**Are molecules exhibiting sharp changes in biological activity particularly difficult for machine-learning models to predict?**

In medicinal chemistry, structurally similar molecules can sometimes have dramatically different biological activities. These cases are known as **activity cliffs**.

Activity cliffs are challenging because molecular machine-learning models generally learn statistical relationships between chemical structure and biological activity.

If two molecules have very similar structures but very different potencies, the relationship between structural similarity and activity similarity becomes less reliable.

### Day 22 hypothesis

**Molecules involved in activity cliffs will exhibit larger prediction errors than molecules not associated with identified activity cliffs.**

My objective was to identify activity cliffs in the AChE dataset and determine whether they help explain the prediction errors of the Random Forest and GNN.

---

## 2. Dataset and molecular representation

I used the same curated AChE bioactivity dataset as in the previous experiments.

| Property | Value |
|---|---|
| Biological target | Human acetylcholinesterase (AChE) |
| Data source | ChEMBL |
| Total molecules | 6,287 |
| Activity endpoint | pIC50 |
| Validation molecules | 930 |
| Molecular similarity representation | Morgan bit fingerprints |
| Fingerprint radius | 2 |
| Fingerprint size | 2,048 bits |
| Similarity metric | Tanimoto similarity |

For activity-cliff detection, I used binary Morgan fingerprints rather than the count-based fingerprints used for Random Forest training.

This distinction matters because the two fingerprint representations encode related but not identical information.

---

## 3. Defining an activity cliff

For this investigation, I defined an activity-cliff pair using two conditions:

**Structural similarity:**

$$
T(A,B)\geq 0.85
$$

where \(T(A,B)\) is the Tanimoto similarity between the molecular fingerprints.

**Potency difference:**

$$
|\mathrm{pIC50}_A-\mathrm{pIC50}_B|\geq 2.0
$$

A difference of 2 pIC50 units corresponds to a 100-fold difference in IC50.

For example, two compounds with IC50 values of 10 nM and 1,000 nM would differ by 2 pIC50 units.

These thresholds were chosen as an operational definition for this experiment. They are not universal criteria for identifying activity cliffs.

---

## 4. Experiment 1 — Random sampling of molecules

### Hypothesis

A random subset of AChE molecules might contain structurally similar pairs with large potency differences.

I randomly selected 300 molecules from the full dataset and compared every possible molecular pair.

The total number of comparisons was:

$$
\frac{300(300-1)}{2}=44,850
$$

### Results

| Measurement | Result |
|---|---:|
| Molecular pairs examined | 44,850 |
| Highly similar pairs | 11 |
| Activity-cliff pairs | 0 |

### Interpretation

The random sample contained only 11 pairs with Tanimoto similarity of at least 0.85.

None of those pairs exhibited a potency difference of at least 2 pIC50 units.

However, this did not establish that activity cliffs were absent from the dataset.

The experiment showed that random sampling was inefficient for finding closely related molecules in a chemically diverse dataset.

**Lesson:** Rare structural relationships are better investigated through systematic similarity searches than through small random samples.

---

## 5. Experiment 2 — Full-dataset activity-cliff detection

### Hypothesis

Searching the entire dataset would reveal activity cliffs missed by random sampling.

I generated Morgan bit fingerprints for all 6,287 molecules and compared their pairwise Tanimoto similarities.

To make the calculation more efficient, I used RDKit's `BulkTanimotoSimilarity` function.

Each unique molecular pair was evaluated once.

The dataset contained approximately 19.8 million possible molecular pairs.

### Results

| Measurement | Result |
|---|---:|
| Highly similar pairs (Tanimoto ≥ 0.85) | 4,094 |
| Candidate activity-cliff pairs | **316** |
| Percentage of highly similar pairs classified as cliffs | **7.7%** |

### Interpretation

The full-dataset search identified 316 candidate activity-cliff pairs.

This confirmed that the AChE dataset contained structurally similar molecules with substantial differences in recorded potency.

Approximately 7.7% of the highly similar pairs met the activity-cliff criterion.

However, this percentage describes the fraction of highly similar pairs, not the fraction of all molecular pairs or all molecules.

---

## 6. Investigating the most extreme activity cliff

The largest identified candidate activity cliff had:

| Property | Value |
|---|---:|
| Tanimoto similarity | 0.887 |
| Absolute pIC50 difference | 7.146 |
| Implied IC50 ratio | Approximately 14 million-fold |

The two molecules shared many structural fragments but differed in how these fragments were connected around an amide group.

This observation suggested that local fingerprint similarity may not fully capture important differences in molecular connectivity and biological recognition.

However, a 7.146-unit pIC50 difference is extraordinarily large.

Possible explanations include genuine differences in binding, differences in experimental conditions, or errors and inconsistencies in the underlying activity records.

I therefore treated this pair as a **candidate activity cliff requiring data-quality verification**, rather than definitive evidence of a 14-million-fold structure–activity effect.

---

## 7. Experiment 3 — Distribution of activity-cliff magnitudes

### Hypothesis

Most activity cliffs would have potency differences closer to the 2-unit threshold, while exceptionally large differences would be less common.

I examined the distribution of absolute pIC50 differences among the 316 candidate cliff pairs.

### Results

| Statistic | Absolute ΔpIC50 |
|---|---:|
| Minimum | 2.000 |
| Median | 2.573 |
| 90th percentile | 3.717 |
| 95th percentile | 4.431 |
| Maximum | 7.146 |

The numbers of particularly large cliffs were:

| Potency difference | Number of pairs |
|---|---:|
| ΔpIC50 ≥ 3 | 84 |
| ΔpIC50 ≥ 4 | 24 |
| ΔpIC50 ≥ 5 | 10 |

### Interpretation

The median activity-cliff magnitude was 2.573 pIC50 units, corresponding to approximately a 374-fold difference in IC50.

The largest difference was much greater than the typical candidate cliff.

Only 10 pairs had potency differences of at least 5 pIC50 units, equivalent to differences of at least 100,000-fold.

These extreme examples deserve careful examination of their original experimental measurements.

**Lesson:** Computationally detected activity cliffs should not automatically be interpreted as experimentally confirmed structure–activity effects.

---

## 8. Experiment 4 — How many molecules participate in activity cliffs?

Activity cliffs are defined as molecular pairs.

A single molecule can participate in several cliff pairs, so the number of cliff pairs is not the same as the number of distinct cliff-associated molecules.

### Hypothesis

Activity cliffs would be concentrated in a relatively small subset of molecules, with some compounds participating in multiple cliff pairs.

### Results

| Measurement | Result |
|---|---:|
| Total molecules | 6,287 |
| Candidate activity-cliff pairs | 316 |
| Unique cliff-associated molecules | **271** |
| Fraction of dataset | **4.3%** |
| Median cliff participation per associated molecule | 1 |
| Maximum cliff participation | 16 |

One molecule participated in 16 candidate cliff pairs, while several others participated in more than 10.

### Interpretation

Only 4.3% of the molecules were involved in identified activity cliffs.

Most cliff-associated molecules participated in only one pair, but a smaller number were connected to multiple cliffs.

This raised an important data-quality question: a molecule with an unusual or inconsistent activity measurement might generate several apparent cliffs.

I also learned that molecular pairs are not statistically independent when the same compounds appear repeatedly.

For subsequent error analysis, I therefore classified individual molecules as cliff-associated or non-cliff-associated instead of treating all cliff pairs as independent observations.

---

## 9. Experiment 5 — Do activity-cliff molecules have larger prediction errors?

This was the central experiment of Day 22.

### Hypothesis

Molecules associated with activity cliffs would have larger prediction errors than other validation molecules.

I used the previously trained Random Forest and four-layer GNN, evaluated on the same 930 validation molecules.

I identified which validation molecules participated in at least one candidate activity-cliff pair.

### Validation-set composition

| Category | Molecules |
|---|---:|
| Cliff-associated | 34 |
| Other molecules | 896 |
| Total | 930 |

Only 3.7% of the validation molecules were classified as cliff-associated.

### Prediction-error results

| Model | Cliff-associated MAE | Other MAE |
|---|---:|---:|
| Random Forest | **1.150** | 0.642 |
| Four-layer GNN | **1.288** | 0.795 |

The Random Forest's MAE was approximately **79% higher** for cliff-associated molecules.

The GNN's MAE was approximately **62% higher** for cliff-associated molecules.

### Interpretation

Both models exhibited substantially larger prediction errors on molecules associated with activity cliffs.

This supported the central Day 22 hypothesis.

The result was consistent with the idea that abrupt structure–activity changes are difficult for models that learn relationships between molecular structure and potency.

However, the experiment did not prove that activity cliffs directly caused the errors.

The higher errors could also be influenced by experimental uncertainty, scaffold novelty, or other molecular characteristics.

The cliff-associated validation group contained only 34 molecules, so this result should be interpreted as a strong descriptive association rather than a definitive general conclusion.

---

## 10. Experiment 6 — Do activity cliffs dominate total prediction error?

Although cliff-associated molecules had much higher average errors, they represented only a small fraction of the validation set.

This raised another question:

**Do these difficult molecules account for most of the models' overall prediction errors?**

### Hypothesis

Cliff-associated molecules would contribute disproportionately to total absolute error, but might not dominate it.

I calculated the fraction of total absolute prediction error contributed by the 34 cliff-associated validation molecules.

### Results

| Measurement | Result |
|---|---:|
| Fraction of validation molecules associated with cliffs | 3.7% |
| RF total absolute error contributed by cliffs | **6.4%** |
| GNN total absolute error contributed by cliffs | **5.8%** |

### Interpretation

Cliff-associated molecules contributed more to the total error than their population fraction would suggest.

For the Random Forest, they represented 3.7% of molecules but contributed 6.4% of total absolute error.

For the GNN, they contributed 5.8%.

However, most prediction error still came from molecules outside the identified cliff group.

Approximately 93.6% of the RF's total absolute error and 94.2% of the GNN's came from other molecules.

This was an important distinction.

**Activity-cliff molecules were individually difficult, but they did not dominate overall prediction error.**

Therefore, improving activity-cliff predictions alone would not solve the broader bioactivity prediction problem.

---

## 11. Methodological limitations

Several limitations are important for interpreting these results correctly.

**Retrospective cliff identification:** I identified activity cliffs using experimental pIC50 values from the complete dataset, including validation and untouched test molecules. This is suitable for retrospective descriptive analysis, but the resulting cliff labels must not be used as model input features or as evidence of prospective uncertainty prediction.

**Experimental data quality:** The dataset contains median-aggregated ChEMBL IC50 measurements. Compounds may have been measured under different experimental conditions, potentially creating apparent activity cliffs that do not reflect comparable biological experiments.

**Fingerprint dependence:** Activity-cliff classification depends on the molecular representation and similarity metric. Different fingerprints could identify different sets of similar molecules.

**Threshold dependence:** The chosen Tanimoto threshold of 0.85 and potency threshold of 2 pIC50 units affect the number of identified cliffs. A sensitivity analysis would be necessary for stronger conclusions.

**Small validation subgroup:** Only 34 validation molecules were cliff-associated, limiting statistical precision.

**Cliff-pair dependence:** Some molecules participated in multiple cliff pairs, so pair counts should not be interpreted as independent experimental observations.

These limitations do not invalidate the exploratory results, but they define what can and cannot be concluded.

---

## 12. Main scientific conclusions

Day 22 produced five important findings.

1. The AChE dataset contains substantial candidate activity cliffs. I identified 316 qualifying molecular pairs involving 271 distinct molecules.

2. Activity cliffs are relatively concentrated. Only 4.3% of all molecules participated in identified cliff pairs under the selected thresholds.

3. Some candidate cliffs exhibit extremely large potency differences. These require careful verification against the original experimental records.

4. Both machine-learning models performed considerably worse on cliff-associated validation molecules. The RF MAE increased from 0.642 to 1.150, while the GNN MAE increased from 0.795 to 1.288.

5. Activity cliffs explain a disproportionate but limited fraction of total prediction error. Most prediction error remains associated with molecules outside the identified cliff group.

### Overall conclusion

**Activity-cliff membership is strongly associated with increased prediction error in my AChE dataset, but activity cliffs are not the dominant source of total model error.**

This finding demonstrates why molecular machine-learning models should be evaluated not only through average performance metrics but also through chemically meaningful subsets of the data.

---

## 13. What I learned

Day 22 helped me connect molecular machine learning with the underlying scientific challenges of medicinal chemistry.

I learned that molecular similarity does not guarantee similar biological activity.

I also learned that a model can perform reasonably well overall while struggling with chemically important exceptions.

The activity-cliff analysis introduced a more interpretable way of examining prediction failures than simply comparing overall MAE or R².

Most importantly, I learned to distinguish three separate questions:

- Does the dataset contain apparent activity cliffs?
- Are cliff-associated molecules harder to predict?
- How much do those molecules contribute to total model error?

These questions require different analyses and should not be confused.

The experiments also reinforced the importance of data quality, validation design, and avoiding information leakage.

---

## 14. Connection to my future research

My background in molecular dynamics and statistical physics makes the activity-cliff problem especially interesting.

Two similar molecules may exhibit different potencies because of differences in conformational preferences, binding-site interactions, solvation, or protein dynamics.

A two-dimensional molecular fingerprint or conventional GNN does not explicitly represent all these effects.

This provides a possible motivation for future projects combining molecular simulation with machine learning.

However, these mechanisms remain hypotheses for our dataset; they were not established by today's experiments.

Before introducing more sophisticated models, I need to understand the reliability of the predictions produced by our existing methods.

### Next step — Day 23: Prediction Uncertainty

My next research question will be:

**Can a machine-learning model estimate when its own bioactivity predictions are likely to be inaccurate, before experimental measurements are available?**

I plan to begin with uncertainty estimates derived from the variation among individual Random Forest trees.

I will then investigate whether larger predicted uncertainty is associated with larger observed prediction error.

This will extend the project from evaluating predictive accuracy toward evaluating **prediction reliability**, which is essential for practical drug discovery.

---

## Final reflection

**Day 22 showed me that understanding where molecular machine-learning models fail is as important as improving their average accuracy.**

I identified 316 candidate activity-cliff pairs in the AChE dataset and found that cliff-associated validation molecules had substantially larger prediction errors for both the Random Forest and GNN.

At the same time, these molecules accounted for only a small fraction of total prediction error.

The central lesson is that activity cliffs represent an important and chemically meaningful challenge, but they are only one part of the broader problem of reliable bioactivity prediction.

**Key takeaway:** A trustworthy molecular ML workflow should investigate difficult chemical cases, verify experimental data quality, quantify prediction uncertainty, and recognize the limits of what molecular structure alone can predict.
