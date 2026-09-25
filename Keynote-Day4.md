# Data: 25.09.2026

## Bioactivity Data and Preparing a Molecular ML Dataset

## What I learned

Today I focused on understanding bioactivity data and how experimental measurements can be converted into a dataset suitable for molecular machine learning.

A ligand molecule can interact with a biological target such as a protein or receptor. Experiments, called assays, are used to measure different aspects of this interaction or its biological effect.

Some important bioactivity measurements are:
* Kd - dissociation constant, related to binding affinity.
* Ki - inhibition constant.
* IC50 - concentration of an inhibitor required to produce 50% inhibition under a particular assay condition.

An important point is that Kd, Ki, and IC50 are different experimental quantities and should not automatically be mixed together as the same ML target.

For IC50 data, I learned to transform concentration into potency:

$$
pIC_{50}=-\log_{10}(IC_{50}[M])
$$

Therefore, lower IC50 corresponds to higher pIC50 and generally indicates greater inhibitory potency in that assay.

## Molecular ML Representation

The workflow is:

$$
\text{Ligand SMILES}
\rightarrow
\text{RDKit molecule}
\rightarrow
\text{Morgan fingerprint}
\rightarrow
X
$$
