# Day 12 — Molecular Features and Activity Cliffs
**2 October 2026**

## Motivation

During the previous two days, I focused on how molecular representation, model hyperparameters, and validation strategy affect model performance.

Today I moved from the question:

**“How well does the model generalize?”**

to:

**“Why can the model fail even for structurally similar molecules?”**

Earlier analysis showed that two molecules can have highly similar, or even identical, binary Morgan fingerprint representations while having very different measured bioactivities. This led to the concept of an **activity cliff**.

An activity cliff broadly describes structurally similar molecules that show a large difference in biological activity.

## Defining Candidate Activity Cliffs

For this exploratory analysis, I defined a candidate activity-cliff pair using:

$$
T_{\mathrm{Morgan}} \geq 0.85
$$

and

$$
|\Delta pIC50| \geq 2
$$

A difference of 2 pIC50 units corresponds to approximately a 100-fold difference in IC50:

$$
10^2=100
$$

These thresholds are operational choices for this analysis and should not be interpreted as a universal definition of an activity cliff.

Before performing the calculation, I predicted that approximately 500 candidate pairs might satisfy these criteria.

## Experiment 1 — Identifying Candidate Activity Cliffs

Using binary radius-2 Morgan fingerprints for the 6,287 AChE molecules, I searched for molecular pairs satisfying both thresholds.

I identified:

$$
\boxed{316\text{ candidate activity-cliff pairs}}
$$

An interesting observation was that several of the strongest candidate cliffs had:

$$
T_{\mathrm{binary}}=1.0
$$

Despite having identical binary Morgan fingerprints, some of these pairs showed differences of more than 3–4 pIC50 units.

This does not mean that the molecules themselves are identical. It means that the chosen binary Morgan representation cannot distinguish them.

## Experiment 2 — Examining a Strong Candidate Cliff

I investigated one of the strongest candidate pairs:

**CHEMBL105537 vs CHEMBL1669486**

Their measured activities were:

$$
pIC50_{\mathrm{CHEMBL105537}}=3.863
$$

$$
pIC50_{\mathrm{CHEMBL1669486}}=8.301
$$

giving:

$$
|\Delta pIC50|=4.438
$$

This corresponds to approximately:

$$
10^{4.438}\approx27,400
$$

fold difference in reported IC50.

However, their binary Morgan similarity was:

$$
T_{\mathrm{binary}}=1.000
$$

Visual inspection showed that both molecules had similar aromatic end groups, while CHEMBL105537 contained a longer carbon chain.

## Binary versus Count Morgan Fingerprints

I then calculated their count-Morgan similarity:

$$
T_{\mathrm{count}}=0.878
$$

Thus:

$$
T_{\mathrm{binary}}=1.000
\qquad\text{but}\qquad
T_{\mathrm{count}}=0.878
$$

This illustrates an important limitation of binary fingerprints.

A binary Morgan fingerprint records whether an encoded local environment is present. Repeated occurrences do not provide additional information once the corresponding bit is active.

Count Morgan fingerprints retain the multiplicity of these environments. Therefore, differences in repeated environments along the carbon chains can distinguish these two molecules even when their binary fingerprints are identical.

## Experiment 3 — Physicochemical Descriptors

I calculated four additional molecular descriptors for the pair.

| Property | CHEMBL105537 | CHEMBL1669486 |
|---|---:|---:|
| pIC50 | 3.863 | 8.301 |
| Molecular weight | 586.46 | 530.35 |
| LogP | 1.18 | -0.38 |
| TPSA | 7.76 | 7.76 |
| Rotatable bonds | 13 | 9 |

The longer-chain molecule, CHEMBL105537, had higher molecular weight, higher calculated LogP, and more rotatable bonds. TPSA remained unchanged.

The molecular-weight difference was approximately 56 Da, consistent with roughly four additional methylene (`CH2`) units:

$$
4\times14\approx56\text{ Da}
$$

These results demonstrate that two molecules with identical binary Morgan fingerprints can nevertheless differ in global physicochemical properties.

## Scientific Interpretation

The analysis provides an example of a **representation collision**: two chemically distinct molecules map to the same binary Morgan fingerprint.

Count Morgan fingerprints recover some of the lost information because they retain environment multiplicity.

However, distinguishing two molecules in feature space does **not** guarantee that an ML model can explain their difference in bioactivity.

The additional information is useful only if it contains features relevant to the observed activity difference.

Other factors that could potentially contribute to an activity cliff include differences in 3D binding geometry, conformational behavior, stereochemistry, protonation state, protein–ligand interactions, or experimental and assay-related variability.

These possibilities are hypotheses rather than demonstrated explanations for this particular pair.

Therefore:

$$
\boxed{\text{More informative representation} \neq \text{guaranteed better prediction}}
$$

## Connection to Previous Results

Previously, the Random Forest using count Morgan fingerprints slightly outperformed the binary Morgan model:

$$
R^2_{\mathrm{binary}}\approx0.734
$$

$$
R^2_{\mathrm{count}}\approx0.746
$$

Today's analysis provides one possible representation-level reason why count fingerprints can sometimes be useful: they distinguish some molecules that collide in the binary representation.

However, this example alone does not establish that count Morgan fingerprints will systematically predict activity cliffs better.

## Main Conclusion

Activity cliffs reveal an important challenge in molecular machine learning: high fingerprint similarity does not necessarily imply similar biological activity.

Among the 6,287 AChE molecules, I identified 316 candidate activity-cliff pairs using binary Morgan Tanimoto similarity ≥ 0.85 and |ΔpIC50| ≥ 2.

Detailed examination of CHEMBL105537 and CHEMBL1669486 showed that their binary Morgan fingerprints were identical despite a ~4.44 pIC50-unit difference. Count Morgan fingerprints distinguished the pair because they retained differences in local-environment multiplicity.

This demonstrates that model errors can arise not only from model architecture or validation strategy, but also from limitations in how molecular structure is represented.

**Next:** investigate prediction reliability and applicability—when should I trust a molecular ML prediction, and when should I be cautious?
