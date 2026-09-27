# Day 2 - 22.09.2026

## What I Learned

Today I reviewed important molecular functional groups, including **–OH, –COOH, –NH₂, –CONH–, C=O, and aromatic groups**, and learned how these groups influence:

* hydrogen-bond donation and acceptance (HBD/HBA),
* protonation and ionization,
* molecular charge,
* hydration and solvation,
* lipophilicity,
* and passive membrane permeability.

## 1. Protonation, Charge, and pKa

For an acidic group such as a carboxylic acid:

$$
R-COOH \rightleftharpoons R-COO^- + H^+
$$

A key relationship is:

$$
pH < pK_a \Rightarrow \text{protonated form favored}
$$

$$
pH = pK_a \Rightarrow \text{approximately 50:50}
$$

$$
pH > pK_a \Rightarrow \text{deprotonated form favored}
$$

For an amine:

$$
R-NH_2 + H^+ \rightleftharpoons R-NH_3^+
$$

Low pH favors the protonated \(R-NH_3^+\) form, whereas sufficiently high pH favors the neutral \(R-NH_2\) form.

## 2. Charge and Hydration

The carboxylate group \(COO^-\) is strongly hydrated because its negative charge produces favorable **electrostatic/ion–dipole interactions** with water. Its oxygen atoms can also accept hydrogen bonds from surrounding water molecules.

Therefore:

$$
\text{charge}
\rightarrow
\text{strong hydration}
\rightarrow
\text{larger desolvation penalty}
$$

Transferring a charged group such as \(COO^-\) from water into the nonpolar, low-dielectric interior of a lipid membrane requires losing favorable hydration interactions. This creates a substantial free-energy penalty and generally makes passive membrane permeation less favorable.

The neutral \(COOH\) form is therefore generally more favorable for passive membrane penetration than the negatively charged \(COO^-\) form, all else being equal.

## 3. LogP vs LogD

I learned an important distinction between **LogP and LogD**.

**LogP** describes partitioning of the neutral form of a molecule between an organic phase and water.

**LogD** describes the pH-dependent distribution of an ionizable compound, accounting for the ionized and unionized forms present at a specified pH.

Therefore, for ionizable drug molecules, pH and pKa are important because they determine the population of different charge states.

This gives the conceptual connection:

$$
pH/pK_a
\rightarrow
\text{protonation state}
\rightarrow
\text{charge}
\rightarrow
\text{hydration}
\rightarrow
\text{distribution/permeability}
$$

## 4. Net Charge Is Not the Whole Story

Another important concept is that **net charge alone does not determine molecular hydration or interactions**.

For example, phenylalanine can exist predominantly as a zwitterion:

$$
^{+}NH_3-CHR-COO^-
$$

Its total net charge is zero, but it contains strong local positive and negative charges and can therefore interact strongly with water.

Thus, molecular behavior depends on **local charge distribution, polarity, functional groups, and molecular structure**, not only on total charge.

## 5. Aromaticity and Molecular Interactions

Aromatic groups contain delocalized π-electron systems. Aromatic residues such as **phenylalanine, tyrosine, and tryptophan** can participate in ligand-binding interactions, including π–π and cation–π interactions.

Aromatic rings can also contribute hydrophobic surface and influence molecular rigidity and shape.

## 6. Stereochemistry

A protein-binding pocket is three-dimensional and asymmetric. Therefore, two stereoisomers can interact differently with the same protein even when they have the same molecular formula and connectivity.

As a result, stereochemistry can influence molecular recognition and binding affinity.

## 7. Connection to My MD Background

A molecule should not always be thought of as a single rigid structure. Flexible molecules can occupy an ensemble of conformations whose populations are governed by their free-energy landscape.

A simple cheminformatics model might use:

$$
\text{2D molecular representation}
\rightarrow
\text{property}
$$

whereas molecular dynamics allows us to think in terms of:

$$
\text{conformational ensemble from MD}
\rightarrow
\text{dynamic/3D features}
\rightarrow
\text{property}
$$

This raises an important question for my future molecular-ML work:

**Can information derived from MD conformational ensembles provide predictive information that is missing from static 2D molecular representations?**

## Key Takeaway

My main conceptual connection from Day 2 is:

$$
\boxed{
\text{Chemical structure}
\rightarrow
\text{functional groups}
\rightarrow
\text{protonation/charge}
\rightarrow
\text{molecular interactions}
\rightarrow
\text{solvation}
\rightarrow
\text{molecular behavior}
}
$$

This provides a physical basis for understanding why molecular descriptors and representations are useful in cheminformatics and molecular machine learning.
