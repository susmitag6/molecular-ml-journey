# Day 16 — From Fixed Fingerprints to Graph Neural Networks

## 1. Learning objective

Today I moved from predicting molecular properties using predefined Morgan fingerprints toward representing molecules directly as graphs and learning molecular representations with graph neural networks (GNNs).

$$
\boxed{\text{Morgan fingerprint + ML}\rightarrow\text{Molecular graph + GNN}}
$$

Both approaches use information from local atomic neighborhoods, but they differ in how molecular representations are constructed.

- **Morgan fingerprints** encode local atomic environments using a predefined algorithm, followed by a machine-learning model.
- **GNNs** take atom features, bond features, and molecular connectivity as inputs and learn how to transform and aggregate this information for a particular prediction task.

The molecular connectivity is supplied to the GNN; the model learns how to use that connectivity.

## 2. Molecular graphs and message passing

A molecule can be represented as a graph:

$$
G=(V,E)
$$

where nodes \(V\) represent atoms and edges \(E\) represent chemical bonds.

Each atom starts with an initial feature vector describing properties such as element identity, degree, formal charge, hybridization, aromaticity, and hydrogen count.

During message passing, an atom receives information from its neighboring atoms and bonds. This produces a new, context-dependent representation of the atom.

$$
\boxed{
\text{initial atom features}
\xrightarrow{\text{message passing}}
\text{context-dependent node embeddings}
}
$$

These outputs are called **learned node representations** or **node embeddings**, rather than new hand-designed chemical features. Their numerical dimensions do not necessarily correspond to individually interpretable chemical properties.

In a standard local message-passing architecture:

$$
\boxed{k\text{ GNN layers}\Rightarrow
\text{information can propagate approximately }k\text{ bonds}}
$$

For example, in a four-atom chain \(A-B-C-D\), two message-passing layers allow information originating at \(D\) to influence the representation of \(B\).

This resembles the local-neighborhood principle behind radius-based Morgan fingerprints, but a GNN learns the transformations applied to those neighborhoods.

## 3. From atom embeddings to molecular predictions

Molecules contain different numbers of atoms, but molecular-property prediction requires one output per molecule.

A GNN therefore uses a global **pooling or readout operation** to combine atom embeddings into a fixed-dimensional molecular embedding.

Two common approaches are:

- **Sum pooling:** adds atom embeddings, retaining sensitivity to the number of contributing atoms.
- **Mean pooling:** averages atom embeddings, normalizing by atom count.

Neither method is universally superior.

The complete pipeline is:

$$
\boxed{
\text{Atoms}
\rightarrow
\text{Message passing}
\rightarrow
\text{Atom embeddings}
\rightarrow
\text{Global pooling}
\rightarrow
\text{Molecular embedding}
\rightarrow
\widehat{\mathrm{pIC50}}
}
$$

A GNN trained for AChE pIC50 may learn different molecular embeddings from one trained for aqueous solubility, even when the same molecular graphs are supplied, because the optimization objectives differ.

## 4. Graph tensors in PyTorch Geometric

I learned how PyTorch Geometric (PyG) represents a molecular graph using three main tensors:

$$
\boxed{
x:[N,F_{\text{atom}}]
}
$$

where \(N\) is the number of atoms and \(F_{\text{atom}}\) is the number of atom features.

$$
\boxed{
edge\_index:[2,E]
}
$$

which specifies the source and destination atom indices for each directed message-passing edge.

$$
\boxed{
edge\_attr:[E,F_{\text{bond}}]
}
$$

which stores the bond features associated with those directed edges.

An undirected chemical bond is typically represented by two directed edges, allowing messages to flow in both directions.

For ethanol (`CCO`), our initial toy representation produced:

$$
x:[3,4],\qquad
edge\_index:[2,4],\qquad
edge\_attr:[4,4].
$$

The atom features were atomic number, degree, formal charge, and aromaticity. The bond features initially encoded single, double, triple, and aromatic bond types.

## 5. GCN versus bond-aware GINE

I first implemented a simple GCN using `GCNConv`.

Its forward pass used:

$$
\text{GCN}:(x,edge\_index).
$$

The model transformed three ethanol atom vectors into eight-dimensional node embeddings:

$$
[3,4]\rightarrow[3,8].
$$

Global mean pooling then produced one molecular embedding:

$$
[3,8]\rightarrow[1,8],
$$

and a linear prediction layer produced:

$$
[1,8]\rightarrow[1,1].
$$

I subsequently implemented a two-layer GCN with the sequence:

$$
[3,4]\rightarrow[3,32]\rightarrow[3,32]
\rightarrow[1,32]\rightarrow[1,1].
$$

**ReLU** was used between layers to introduce nonlinearity, allowing the model to learn more complex transformations of graph-based representations.

A limitation of our basic `GCNConv` implementation was that it did not consume the vector-valued bond features in `edge_attr`.

I therefore implemented a bond-aware architecture using `GINEConv`:

$$
\boxed{\text{GINE}:(x,edge\_index,edge\_attr)}.
$$

Separate encoders transformed atom and bond features into compatible 32-dimensional representations before message passing.

To test whether bond information affected the computation, I kept the atom features, connectivity, and model parameters fixed while changing one encoded bond type.

The untrained model returned:

- Original bond encoding: **0.39065**
- Modified bond encoding: **0.40680**

The outputs differed, demonstrating that the bond attributes affected the model's computation.

However, these values have **no meaningful bioactivity interpretation**, because the model had not been trained. The experiment demonstrates sensitivity to bond encoding, not a measured difference in inhibitory potency.

## 6. Chemical concepts: sp, sp², and sp³ hybridization

Hybridization describes how atomic orbitals are represented in a local bonding model and is associated with characteristic bonding geometries.

### sp hybridization

An sp-hybridized center has two sp hybrid orbitals and typically approximately linear geometry, with a bond angle near \(180^\circ\).

Common examples include carbon atoms in alkynes and nitriles:

$$
\mathrm{C\equiv C},\qquad \mathrm{C\equiv N}.
$$

### sp² hybridization

An sp²-hybridized center has three sp² hybrid orbitals and one remaining unhybridized p orbital. It typically has approximately trigonal-planar geometry, with angles near \(120^\circ\).

Examples include alkene carbons, carbonyl carbons, and many atoms in aromatic rings.

The remaining p orbital can participate in π bonding and conjugation.

### sp³ hybridization

An sp³-hybridized center has four sp³ hybrid orbitals, often associated with approximately tetrahedral geometry and bond angles near \(109.5^\circ\).

Examples include saturated carbon atoms in alkanes. Lone pairs can modify the observed geometry and angles around other elements.

### Hybridization in the AChE dataset

I inspected the RDKit-assigned hybridization states across the cleaned AChE molecules:

| Hybridization | Atom count |
|---|---:|
| SP2 | 121,428 |
| SP3 | 65,835 |
| SP | 762 |
| S | 5 |
| SP3D | 2 |

SP2 and SP3 dominate, while SP is comparatively rare.

These are RDKit-assigned hybridization categories; they are useful structural descriptors, not a complete description of molecular electronic structure.

## 7. Conjugated bonds

A conjugated system contains a sequence of atoms with overlapping p orbitals, allowing π electrons to be delocalized across multiple atoms.

For example, 1,3-butadiene contains:

$$
\mathrm{CH_2=CH-CH=CH_2}.
$$

Although its bond pattern is double–single–double, the central single bond participates in the conjugated system.

Therefore:

$$
\boxed{\text{A conjugated bond does not have to be a double bond.}}
$$

Aromatic rings, such as benzene, are important examples of conjugated systems.

Conjugation can influence electronic structure, geometry, rigidity, and chemical reactivity.

In RDKit, I encoded this information using:

`int(bond.GetIsConjugated())`

This feature tells the GNN whether RDKit identifies the bond as participating in a conjugated system. It provides additional information beyond the assigned bond type.

## 8. Designing chemically appropriate atom features

Rather than treating atomic number as an ordinary continuous variable, I chose categorical encoding for element identity.

For example:

$$
C=[1,0,0],\quad N=[0,1,0],\quad O=[0,0,1].
$$

This avoids imposing arbitrary numerical distances between element categories.

I also inspected the AChE dataset before defining the feature vocabularies. The dataset contained common elements such as C, N, O, F, Cl, Br, and S, along with less common elements including I, Se, B, Si, and Na.

I used an `"Other"` category for values outside the chosen vocabulary.

The resulting atom representation contained:

| Feature | Dimensions |
|---|---:|
| Element identity | 14 |
| Atom degree | 6 |
| Formal charge | 4 |
| Hybridization | 4 |
| Aromaticity | 1 |
| Total hydrogen count | 5 |
| **Total** | **34** |

Aromaticity was encoded as a binary feature using `atom.GetIsAromatic()`.

Hydrogen count was encoded using `atom.GetTotalNumHs()`, which accounts for implicit hydrogens under RDKit's representation.

The initial feature set does not yet include explicit chirality encoding. Chirality and stereochemical bond information are potential improvements for the portfolio model.

## 9. Designing bond features

I implemented a seven-dimensional bond feature vector containing:

| Feature | Dimensions |
|---|---:|
| Bond type: single, double, triple, aromatic, other | 5 |
| Conjugation | 1 |
| Ring membership | 1 |
| **Total** | **7** |

For example, an ordinary non-conjugated single bond outside a ring is represented as:

$$
[1,0,0,0,0,0,0].
$$

Bond stereochemistry is not yet included in this initial seven-dimensional encoding.

## 10. Variable-size graph batching

PyG can batch molecules with different atom counts without padding them to the same size.

For example, two molecules with 25 and 47 atoms produce a combined node-feature tensor:

$$
[25,F]+[47,F]\rightarrow[72,F].
$$

PyG concatenates the node and edge tensors, adjusts edge indices, and maintains a `batch` vector identifying the molecule to which each atom belongs.

After message passing:

$$
[72,32]\xrightarrow{\text{global pooling}}[2,32]
\rightarrow[2,1].
$$

Thus, the model can predict one property value for each molecule in the batch.

## 11. First real AChE graph

I implemented a reusable `smiles_to_graph()` function that converts an RDKit molecule into a PyG `Data` object with atom features, directed connectivity, bond features, and an optional pIC50 target.

For the first AChE molecule:

`CN(CCOCCNC(=S)NC(=O)c1ccccc1)Cc1ccccc1`

with experimental:

$$
pIC50=6.9208,
$$

the function produced:

$$
\boxed{
Data(x=[26,34],\ edge\_index=[2,54],
edge\_attr=[54,7],\ y=[1])
}
$$

This corresponds to 26 explicit atoms, 27 chemical bonds represented as 54 directed edges, 34 features per atom, seven features per directed edge, and one experimental regression target.

This was the first successful conversion of a real molecule from my AChE dataset into the representation required for graph-based learning.

## 12. Scientific interpretation and next steps

The main lesson is that **representation and predictive model are separate design choices**.

Previously, I used:

$$
\text{molecular graph}
\xrightarrow{\text{fixed Morgan algorithm}}
\text{2048-dimensional fingerprint}
\xrightarrow{\text{RF or NN}}
\widehat{\mathrm{pIC50}}.
$$

Now I am developing:

$$
\text{molecular graph}
\xrightarrow{\text{learned message passing}}
\text{molecular embedding}
\xrightarrow{\text{predictor}}
\widehat{\mathrm{pIC50}}.
$$

A GNN has the flexibility to learn task-dependent molecular representations, but this does not guarantee that it will outperform a random forest using Morgan fingerprints.

A two-dimensional molecular graph also does not automatically capture conformational ensembles, protein–ligand binding geometry, solvent effects, or assay conditions.

**Day 17 will focus on** validating graph conversion across the full AChE dataset, handling problematic molecular records, constructing PyG DataLoaders, training the first bond-aware GNN, and evaluating it using scaffold-aware validation.

The eventual comparison with the random-forest baseline must use comparable representations of the same molecules, appropriate validation splits, and metrics such as MAE, RMSE, and R².

**Key takeaway:** I now understand how molecular connectivity and chemical features become tensors, how message passing produces context-dependent atom embeddings, and how pooling converts variable-size molecular graphs into predictions of molecular properties.
