# Medicinal Chemistry Learning Journal : Day 1

**Date:** 11 October 2026  
**Learning track:** Medicinal Chemistry for Computational Drug Discovery  
**Background:** Statistical physics, biomolecular simulations, molecular dynamics, and molecular machine learning  
**Current project:** AChE Bioactivity Prediction

## 1. Learning Objective

Develop the medicinal chemistry knowledge needed to interpret molecular machine learning results and understand the trade-offs involved in drug discovery.

The objective is not to become a synthetic organic chemist, but to learn how molecular structure influences potency, binding, solubility, permeability, and drug-like properties.

Today's focus:

- Binding affinity versus individual molecular interactions
- Protonation and solvation
- Membrane permeability
- Potency and pIC50
- Structure–activity relationships (SAR)
- Lipophilicity and aqueous solubility
- Lipophilic ligand efficiency (LLE)
- logP versus logD

---

## 2. Binding Affinity Is Determined by Free Energy

### Question

If protonating an amine strengthens its electrostatic interaction with a protein, must the protonated inhibitor bind more strongly?

### My reasoning

No. The final binding affinity also depends on three-dimensional geometry, solvation, and preferred molecular conformations.

### Scientific interpretation

Binding is determined by the total free-energy difference between bound and unbound states:

$$
\Delta G^\circ_{\mathrm{bind}} = RT\ln\left(\frac{K_d}{C^\circ}\right)
$$

A favorable electrostatic interaction does not guarantee stronger overall binding.

Other important effects include:

- **Desolvation:** A charged group may lose favorable interactions with water when entering a protein binding pocket.
- **Geometry:** Interacting groups must be positioned appropriately.
- **Conformational changes:** The ligand and protein may need to adopt less favorable conformations.
- **Entropy:** Binding changes the accessible conformational and solvent states.
- **Protonation equilibria:** Different protonation states may have different binding affinities.

**Key lesson:** Drug–target binding cannot be predicted from one favorable interaction alone.

### Connection to my background

This is an application of statistical thermodynamics. The binding affinity reflects an ensemble free-energy difference, not merely the potential energy of a single optimized protein–ligand structure.

---

## 3. Protonation and Membrane Permeability

### Question

Which compound would generally cross a lipid membrane more easily?

- Compound A: Neutral amine
- Compound B: Permanently charged quaternary ammonium group

### My reasoning

Compound A should generally cross more easily because the charged compound is strongly stabilized by water. Transferring a charged group into the lipid membrane incurs a substantial energetic penalty.

### Scientific interpretation

Passive membrane permeation depends partly on transferring a molecule from an aqueous environment into a hydrophobic membrane.

$$
\Delta G^\circ_{\mathrm{transfer}} = G^\circ_{\mathrm{membrane}} - G^\circ_{\mathrm{water}}
$$

Strong hydration of an ion often makes this transfer less favorable.

A neutral compound may partition more readily into the membrane, although actual permeability also depends on molecular size, polarity, conformation, and diffusion.

A neutral amine can become protonated depending on pH and pKa. A quaternary ammonium group remains permanently charged under ordinary physiological conditions.

**Key lesson:** Strong binding to a biological target and efficient membrane permeation are different molecular properties.

---

## 4. Potency and the Meaning of pIC50

### Question

An AChE inhibitor's pIC50 increases from 6 to 7. How much does its IC50 change?

### My answer

Its IC50 decreases tenfold.

### Calculation

$$
\mathrm{pIC}_{50} = -\log_{10}\left(\mathrm{IC}_{50}\,[\mathrm{M}]\right)
$$

For pIC50 = 6:

$$
\mathrm{IC}_{50}=10^{-6}\,\mathrm{M}=1000\,\mathrm{nM}
$$

For pIC50 = 7:

$$
\mathrm{IC}_{50}=10^{-7}\,\mathrm{M}=100\,\mathrm{nM}
$$

The compound is therefore ten times more potent in terms of measured IC50.

### Important distinction

IC50 measures the concentration needed to reduce activity by 50% under specified assay conditions.

It is not automatically equal to the equilibrium dissociation constant Kd or the inhibition constant Ki.

**Key lesson:** A one-unit increase in pIC50 represents a tenfold decrease in IC50.

---

## 5. Structure–Activity Relationships (SAR)

### Question

Adding an aromatic ring improves AChE pIC50 from 6 to 7. Can we conclude that the aromatic ring creates a specific favorable interaction with the protein?

### My reasoning

No. The potency improvement alone does not identify the specific molecular interaction responsible.

### Scientific refinement

An aromatic ring is electrically neutral overall, but it can still participate in favorable interactions:

- Aromatic π–π interactions
- Cation–π interactions
- Dispersion interactions
- Hydrophobic packing

The ring may also change ligand conformation, solvation, and other properties.

Therefore, increased potency does not prove that any particular interaction caused the improvement.

Additional structural, biochemical, or computational evidence would be needed to support a mechanism.

### Connection to molecular ML

This is relevant to **activity cliffs**: small structural changes can produce large changes in experimental potency.

A predictive model may identify these changes without explaining their physical origin.

**Key lesson:** SAR observations establish relationships between structure and measured activity, but do not necessarily establish causal binding mechanisms.

---

## 6. Lipophilicity, Solubility, and Drug-Likeness

### Question

If two inhibitors have identical pIC50 values but different lipophilicities, is the more lipophilic compound automatically the better drug candidate?

### Initial reasoning

Higher lipophilicity may favor partitioning into lipid membranes and potentially improve permeability.

### Scientific refinement

Greater lipophilicity can be beneficial in some circumstances, but excessive lipophilicity may cause:

- Poor aqueous solubility
- Increased nonspecific binding
- Unfavorable distribution
- Metabolic or safety liabilities

A compound must generally function in both aqueous and lipid environments.

A simplified partition–diffusion model is:

$$
P\approx\frac{KD}{h}
$$

where:

- \(P\): Membrane permeability coefficient
- \(K\): Membrane–water partition coefficient
- \(D\): Diffusion coefficient within the membrane
- \(h\): Membrane thickness

Increasing \(K\) alone does not guarantee improved drug exposure.

**Key lesson:** Lead optimization requires balancing multiple molecular properties rather than maximizing lipophilicity.

---

## 7. Lipophilic Ligand Efficiency (LLE)

### Definition

Lipophilic ligand efficiency provides a simple way to compare potency with lipophilicity:

$$
\mathrm{LLE}=\mathrm{pIC}_{50}-\log P
$$

Higher LLE can indicate that a compound achieves its potency without requiring as much lipophilicity.

### Example

| Property | Compound B | Compound D |
|---|---:|---:|
| pIC50 | 7.0 | 7.0 |
| logP | 4.8 | 3.5 |
| LLE | 2.2 | 3.5 |

### My conclusion

Compound D has the higher LLE and is preferable by this metric because it maintains the same potency at lower lipophilicity.

However, aqueous solubility data are still needed.

### Scientific interpretation

Lower logP does not guarantee higher aqueous solubility. Solubility also depends on ionization, crystal packing, molecular structure, and other factors.

LLE is a useful optimization metric, but it is not a complete measure of drug quality.

**Key lesson:** A medicinal chemistry improvement may involve maintaining potency while reducing undesirable physicochemical properties.

---

## 8. logP versus logD

### Definitions

**logP:** The logarithm of the octanol–water partition coefficient of the neutral molecular form.

**logD:** The logarithm of the octanol–water distribution coefficient accounting for relevant ionization states at a specified pH.

For ionizable compounds, logD depends on pH.

### Example

Consider a basic amine:

$$
\mathrm{p}K_a=9.0,\qquad \mathrm{pH}=7.4
$$

For a simple monoprotic base:

$$
f_{\mathrm{protonated}} = \frac{1}{1+10^{\mathrm{pH}-\mathrm{p}K_a}}
$$

The protonated fraction is approximately 97.5%.

### My prediction

logD at pH 7.4 should generally be lower than logP because the protonated form is strongly hydrated and tends to remain in the aqueous phase.

### Approximate relationship

Assuming the ionized form contributes negligibly to octanol partitioning:

$$
\log D \approx \log P-\log_{10}(1+10^{\mathrm{p}K_a-\mathrm{pH}})
$$

For pKa = 9.0 and pH = 7.4:

$$
\log D\approx\log P-1.61
$$

If logP = 3.0:

$$
\log D\approx1.39
$$

**Key lesson:** Protonation can strongly alter the observed distribution of a compound between aqueous and organic environments.

---

## 9. Connection to the AChE Molecular ML Project

Our current model predicts pIC50 values derived from biochemical assays.

It does **not** directly predict:

- Membrane permeability
- Aqueous solubility
- Bioavailability
- Metabolic stability
- Toxicity
- Overall drug suitability

Furthermore, our molecular graph features include formal charge, but do not explicitly model pH-dependent protonation equilibria.

Future experiments could use RDKit descriptors such as molecular weight, calculated logP, hydrogen-bond donors and acceptors, and topological polar surface area to explore relationships between molecular properties and AChE potency.

These descriptors would support chemical interpretation, but they should not be mistaken for experimental ADMET measurements.

---

## 10. Personal Learning Assessment

### Concepts I understood well

- Binding affinity reflects multiple competing free-energy contributions.
- Charged molecules generally face larger barriers to passive lipid membrane partitioning.
- A one-unit pIC50 increase corresponds to a tenfold reduction in IC50.
- Potency alone cannot establish a specific protein–ligand interaction.
- Lead optimization requires considering properties beyond potency.
- logD can be lower than logP for an ionized basic compound.

### Concepts to strengthen

- pKa and protonation-state calculations
- logP, logD, and solubility relationships
- Aromatic interactions and protein–ligand recognition
- Distinguishing biochemical potency from cellular efficacy
- Interpreting structure–activity relationships
- Drug metabolism and ADMET

### Main insight

**The most potent compound is not necessarily the best drug candidate.**

Drug discovery involves balancing biochemical potency, molecular recognition, physicochemical properties, biological exposure, selectivity, and safety.

---

## 11. Plan for the Next Session

**Next medicinal chemistry topic:** Stereochemistry and molecular recognition.

Questions to investigate:

1. Why can two enantiomers have different binding affinities?
2. How can stereochemistry influence potency and selectivity?
3. What stereochemical information is present in SMILES?
4. Does our current GNN encode this information correctly?

**Practical connection:** Day 19 of the AChE molecular GNN project.

### Learning strategy

Continue molecular ML as the main activity, with approximately 15–20 minutes of medicinal chemistry integrated into each study day.

The goal is to develop chemical intuition through practical molecular modeling experiments rather than studying medicinal chemistry in isolation.
