
# Data: 26.09.2026

## Predicting the Bioactive response and Understading the limiation of model


In the previous step, I prepared and saved a dataset containing ligand molecules and their measured bioactivity against the target protein, acetylcholinesterase (AChE). Today, I used this dataset to build my first QSAR model and, more importantly, investigated where and why the model fails.

## Workflow

1. I first checked the dataset for missing and non-finite values and removed problematic entries.

2. I inspected the statistical distribution of the target variable, pIC50.

3. I converted the ligand SMILES representations into Morgan fingerprints.

4. I randomly split the molecular dataset into training and test sets.

5. I trained a Random Forest regression model using the Morgan fingerprints to predict pIC50.

6. I compared the experimental and predicted pIC50 values and evaluated the model using MAE, RMSE, and R².

The model produced approximately:

- MAE = 0.533
- RMSE = 0.765
- R² = 0.734

The fact that RMSE is larger than MAE suggests that some molecules have relatively large prediction errors.

## Investigating Model Failures

Instead of considering only the average model performance, I calculated the residuals

$$
r_i = y_i-\hat{y}_i
$$

and identified test molecules with the largest absolute prediction errors.

This showed that reasonable overall model performance can hide substantial errors for individual molecules.

I then calculated the maximum Tanimoto similarity of each test molecule to molecules in the training set.

The correlation between absolute prediction error and maximum training-set similarity was

$$
r=-0.184.
$$

This suggests a weak tendency for molecules that are less similar to the training set to have larger prediction errors. Such molecules may represent extrapolation into less familiar chemical space.

However, similarity alone could not explain the prediction errors. Some molecules had very high Tanimoto similarity to a training molecule and still showed large prediction errors.

## Limitation of Binary Morgan Fingerprints

I investigated several high-similarity/high-error molecular pairs.

An especially interesting example contained two molecules differing by only one methylene group in a linker:

$$
\ldots CCCCCN \ldots
$$

versus

$$
\ldots CCCCCCN \ldots
$$

Despite being structurally different, their binary Morgan fingerprints had

$$
T_{\mathrm{binary}}=1.0.
$$

Morgan fingerprints encode local molecular environments into hashed fingerprint features. In a binary fingerprint, a bit indicates whether a particular hashed feature is present, but it does not retain how many times that feature occurs.

Therefore, repeated molecular environments can be lost when their counts are converted to simple binary presence/absence information.

I tested this hypothesis using count-based Morgan fingerprints. Three fingerprint features had different counts between the two molecules:

| Fingerprint bit | Test molecule | Training molecule |
|---|---:|---:|
| 80 | 7 | 8 |
| 1143 | 1 | 2 |
| 1911 | 3 | 4 |

The count-based Tanimoto similarity became

$$
T_{\mathrm{count}}=0.972,
$$

compared with

$$
T_{\mathrm{binary}}=1.0.
$$

This demonstrates that the binary representation had discarded some structural information that was retained by the count fingerprint.

## What I Understood Conceptually

A central lesson from today's analysis is that molecular similarity is not an absolute property. It depends on the molecular representation and the similarity metric being used.

The complete modeling pipeline is

$$
\text{Molecular structure}
\rightarrow
\text{representation}
\rightarrow
\text{ML model}
\rightarrow
\text{predicted property}.
$$

The molecular representation therefore acts as an information bottleneck. Information discarded during the conversion from molecular structure to a fingerprint cannot subsequently be recovered by the Random Forest.

Low Tanimoto similarity may indicate that a model is extrapolating into unfamiliar chemical space. However, high fingerprint similarity does not guarantee accurate activity prediction.

Bioactivity can depend on information that is not fully represented by a 2D binary fingerprint, including molecular geometry, stereochemistry, protonation and charge states, conformational flexibility, and interactions with the target and solvent. Experimental and assay heterogeneity may also contribute to prediction errors.

This motivates investigating richer molecular representations later, including GNNs, 3D molecular representations, and simulation-derived features. These approaches may retain information that is absent from simple 2D fingerprints, although they do not automatically eliminate prediction errors.

## Connection to Molecular Simulation

This analysis connects naturally with my molecular simulation background.

A 2D fingerprint represents selected structural information about a molecule, whereas molecular dynamics describes an ensemble of molecular configurations,

$$
P(x)\propto e^{-\beta U(x)}.
$$

Therefore, an important question for my future work is whether conformational, energetic, kinetic, or interaction information obtained from molecular simulations can improve molecular ML predictions in cases where purely 2D representations are insufficient.

## Research Questions

1. How does prediction error depend on the similarity of a test molecule to the training chemical space?

2. When structurally similar molecules have very different activities, is the discrepancy caused by representation limitations, genuine activity cliffs, experimental heterogeneity, or a combination of these factors?

3. Can richer representations such as GNNs, 3D molecular descriptors, or MD-derived features improve predictions for molecules where 2D fingerprints fail?

4. What physical information is missing from the molecular representation used by the ML model?
