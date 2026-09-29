# Day 9 — Molecular Representations
**29 September 2026**

## What I learned

Today I compared three molecular representations for predicting AChE pIC50:

1. Eight RDKit physicochemical descriptors
2. Binary Morgan fingerprints
3. Count Morgan fingerprints

The descriptors were molecular weight, LogP, TPSA, hydrogen-bond donors and acceptors, rotatable bonds, ring count and FractionCSP3.

### FractionCSP3

`rdMolDescriptors.CalcFractionCSP3(mol)` calculates the fraction of carbon atoms that are sp³-hybridized.

FractionCSP3 is a useful descriptor of molecular character, but it is not a direct measure of solubility, flexibility or binding affinity. Higher sp³ character can introduce greater three-dimensionality and sometimes improve particular physicochemical properties. Its effects depend on the complete molecular structure.

## Hypothesis

Count Morgan fingerprints may improve AChE bioactivity predictions because they preserve the multiplicity of hashed molecular environments, whereas binary Morgan fingerprints record only their presence or absence.

However, retaining more information does not necessarily improve predictive performance. The additional information must be relevant to the target property and usable by the learning algorithm.

## Representation comparison

I trained Random Forest models using identical training/test molecule indices and the same model configuration.

| Representation | MAE | RMSE | Test R² |
|---|---:|---:|---:|
| Descriptors | 0.7323 | 1.0016 | 0.5445 |
| Binary Morgan | 0.5331 | 0.7650 | 0.7343 |
| Count Morgan | 0.5147 | 0.7486 | 0.7455 |

Both Morgan representations substantially outperformed the eight global descriptors in this experiment. Count Morgan also showed a modest improvement over binary Morgan.

This suggests that local molecular connectivity contains useful predictive information beyond these eight global descriptors, and that retaining environment multiplicity may provide additional information.

## Five-fold cross-validation

I compared binary and count Morgan using the same five folds within the training set.

| Fold | Binary Morgan R² | Count Morgan R² |
|---|---:|---:|
| 1 | 0.7420 | 0.7571 |
| 2 | 0.7398 | 0.7607 |
| 3 | 0.7136 | 0.7320 |
| 4 | 0.7124 | 0.7375 |
| 5 | 0.7297 | 0.7405 |
| **Mean** | **0.7275** | **0.7456** |
| **Standard deviation** | **0.0125** | **0.0113** |

Count Morgan achieved a higher validation R² in all five folds, with similar fold-to-fold variability. The improvement in mean CV R² was approximately 0.018.

This supports the observed advantage of count Morgan for this particular dataset, Random Forest configuration and random validation strategy. The folds are not independent experiments, and these results do not establish universal superiority or generalization to unseen chemical scaffolds.

## Molecular-pair investigation

Revisiting the Day 5 example showed why representation and prediction must be considered separately.

1. **Representation:** Can the fingerprint distinguish two molecules? Count Morgan could distinguish the selected pair, while binary Morgan could not under our settings.
2. **Learning:** Has the model learned a reliable relationship between the distinguishing features and bioactivity?
3. **Physical and experimental context:** Does the representation contain enough information to explain the observed activity difference?

A fingerprint may distinguish molecules without capturing the effects of their conformational ensembles, protein–ligand interactions, solvent, protonation conditions or experimental assay differences.

## Main conclusion

Molecular representation determines what information is available to a learning algorithm. In this experiment, count Morgan retained additional information and produced modestly better predictive performance than binary Morgan across all five random CV folds.

However, better representation does not guarantee that a model captures the physical mechanisms responsible for differences in bioactivity. Validation on unseen scaffolds remains an important next step.
