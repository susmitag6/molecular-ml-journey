# Day 3 — 22.09.2026

## From Molecular Simulation to Molecular ML

In molecular dynamics, I am familiar with thinking in terms of:

$$
x(t) \rightarrow \text{states / ensembles / kinetics / free-energy landscape}
$$

Today I learned how molecular machine learning starts from a different representation:

$$
\text{molecule}
\rightarrow
\text{representation}
\rightarrow
X
\rightarrow
\text{ML model}
\rightarrow
\text{property}
$$

The main goal today was to understand how a 2D molecular structure can be transformed into numerical features that can be used by machine-learning algorithms.

---

## Morgan Fingerprints

I learned how Morgan fingerprints encode the local structural environments within a molecule.

Starting from a molecular graph, RDKit examines atoms and their neighborhoods up to a chosen radius. With radius 2, information about local environments extending two bonds from each atom is generated.

These environments are then mapped into a fixed-size fingerprint. I used:

$$
2048 \text{ bits}
$$

so that each molecule was represented by a binary vector:

$$
\mathbf{x}\in\{0,1\}^{2048}.
$$

This can be viewed as:

$$
\boxed{
\text{rich chemical structure}
\rightarrow
\text{compressed numerical representation}
}
$$

The active or ON bits represent fingerprint features detected in the molecular graph.

An important lesson is that the number of ON bits is **not simply related to the number of atoms in a molecule**.

Morgan fingerprints encode different local atomic environments. Highly symmetric molecules can contain many atoms but relatively few distinct local environments.

For example, benzene and buckminsterfullerene both produced only three ON bits in my fingerprints despite their very different sizes.

By contrast, serotonin produced many more active bits because it contains a greater diversity of local environments, including aromatic carbons, heteroatoms, an amine, a hydroxyl group, and different environments within its fused ring system.

---

## Tanimoto Similarity

I learned how to compare two molecular fingerprints using Tanimoto similarity:

$$
T(A,B)=
\frac{|A\cap B|}
{|A|+|B|-|A\cap B|}
$$

where \(A\) and \(B\) are the sets of active fingerprint bits.

The number of active bits alone does not determine similarity. What matters is **which fingerprint positions are shared**.

For example, benzene and buckminsterfullerene both had three ON bits, but their active positions were completely different:

$$
A\cap B=\varnothing
$$

and therefore:

$$
T=0.
$$

This demonstrated that:

$$
\boxed{
\text{same number of ON bits}
\neq
\text{same molecular fingerprint}
}
$$

---

## Similarity Matrix and Nearest Neighbors

I calculated the pairwise Tanimoto similarity matrix for my ten molecules.

This allowed me to identify nearest neighbors in fingerprint space.

The largest off-diagonal similarity in my small dataset was:

$$
T(\text{Ibuprofen},\text{Phenylalanine})=0.382.
$$

I also learned that nearest-neighbor relationships do not have to be mutual. Molecule A can have molecule B as its closest neighbor while molecule B can have another molecule C as an even closer neighbor.

Most importantly:

$$
\boxed{
\text{molecular similarity depends on the chosen representation and metric}
}
$$

Therefore, Tanimoto similarity of Morgan fingerprints is one definition of molecular similarity, not an absolute definition of chemical similarity.

---

## Chemical Space and Machine Learning

Molecular similarity becomes important when thinking about the chemical space represented by a machine-learning training set.

If a new molecule is structurally similar to molecules already present in the training data, prediction is closer to:

$$
\boxed{\text{interpolation within represented chemical space}}
$$

If a molecule is very different from everything in the training set, the model is being asked to move toward:

$$
\boxed{\text{extrapolation outside familiar chemical space}}
$$

and its predictions may be less reliable.

This will become important later when evaluating QSAR models.

A random train/test split can accidentally place structurally similar compounds in both sets:

$$
\text{similar chemistry}
\rightarrow
\boxed{\text{TRAIN}}
\qquad
\text{similar chemistry}
\rightarrow
\boxed{\text{TEST}}
$$

The resulting test performance can then look strong partly because the test molecules resemble chemistry already represented in the training set.

This motivates the later use of approaches such as scaffold-based splitting.

---

## From Fingerprints to an ML Feature Matrix

I converted the RDKit fingerprint objects into a NumPy matrix.

For ten molecules represented by 2048-bit fingerprints:

$$
X\in\mathbb{R}^{10\times2048}
$$

with binary entries.

This gives the complete bridge:

$$
\boxed{
\text{SMILES}
\rightarrow
\text{RDKit Mol}
\rightarrow
\text{Morgan fingerprint}
\rightarrow
X
\rightarrow
\text{machine learning}
}
$$

This \(X\) is the type of feature matrix that can later be used as input to a QSAR model.

---

## Variance and Sparse Fingerprints

Out of the 2048 fingerprint dimensions, I found:

$$
1922 \text{ zero-variance bits}
$$

and only:

$$
126 \text{ varying bits}.
$$

A zero-variance bit has the same value for every molecule in this dataset and therefore cannot distinguish between them.

After removing those dimensions:

$$
X:\;(10,2048)
\rightarrow
X_{\mathrm{varying}}:\;(10,126).
$$

This showed how sparse molecular fingerprints can be, particularly for a very small molecular dataset.

---

## PCA and Visualization of Chemical Space

I used PCA to reduce the 126 varying fingerprint dimensions to two dimensions.

The first two principal components explained:

$$
PC1=18.79\%
$$

and

$$
PC2=18.39\%.
$$

Together:

$$
PC1+PC2=37.18\%.
$$

Therefore, the 2D PCA representation retained only about 37% of the variance.

An important observation came from comparing Tanimoto similarity with distances in the PCA plot.

For ibuprofen and phenylalanine:

$$
T=0.382,\qquad d_{\mathrm{PCA}}=0.427
$$

and they appeared close in the PCA projection.

However, for glucose and buckminsterfullerene:

$$
T=0
$$

while:

$$
d_{\mathrm{PCA}}=0.485.
$$

Therefore, they appeared relatively close in the 2D PCA plot despite having no overlapping fingerprint bits.

This demonstrates an important limitation of dimensionality reduction:

$$
\boxed{
\text{proximity in a 2D projection}
\neq
\text{high similarity in the original representation}
}
$$

The PCA plot is only a projection of the chosen molecular representation. It should not automatically be interpreted as the true chemical relationship between molecules.

---

## Key Takeaways

My main conceptual pipeline from Day 3 is:

$$
\boxed{
\text{Molecular structure}
\rightarrow
\text{fingerprint}
\rightarrow
\text{numerical feature space}
\rightarrow
\text{similarity}
\rightarrow
\text{chemical space}
\rightarrow
\text{machine learning}
}
$$

I also learned to be careful when interpreting molecular ML results:

$$
\boxed{
\text{Representation}
+
\text{similarity metric}
+
\text{dimensionality reduction}
\text{ determine what patterns we observe}
}
$$

A visually convincing cluster in a 2D plot does not automatically imply strong molecular similarity.

## Research Questions

1. How reliably does proximity in a low-dimensional visualization reflect similarity in the original molecular representation?

2. How does the similarity of a test molecule to the training set relate to prediction error and model uncertainty?

3. Can information from MD-derived conformational ensembles improve predictions when 2D molecular fingerprints provide insufficient information?
