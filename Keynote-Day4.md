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
\text{Ligand SMILES} \rightarrow \text{RDKit molecule} \rightarrow \text{Morgan fingerprint} \rightarrow IC_{50} \rightarrow pIC_{50} \rightarrow y
$$

$$
X\rightarrow y
$$
or
$$
\text{ligand molecular structure} \rightarrow \text{predicted inhibitory potency}.
$$

## ChEMBL Data Preparation

I used ChEMBL to obtain experimental bioactivity data.

For this project, I selected:

Target: Human acetylcholinesterase (AChE)

ChEMBL ID: CHEMBL220

Target type: SINGLE PROTEIN

Organism: Homo sapiens

The objective is to predict ligand activity against this fixed protein target.
The main data-preparation steps were:

* Search ChEMBL for acetylcholinesterase.
* Select the human single-protein target rather than acetylcholinesterase from another organism.
* Retrieve IC50 activity records for CHEMBL220.
* Keep IC50 measurements rather than mixing IC50 with Ki, Kd, EC50, etc.
* Inspect standard_relation and keep exact (=) measurements for this first regression dataset.
* Inspect and standardize concentration units. For this project, I retained measurements reported in nM.
* Check for missing SMILES and activity values.
* Convert IC50 values into pIC50.
* Compare the calculated pIC50 values with ChEMBL's pChEMBL values as a sanity check.
* Examine repeated measurements for the same ligand.
* Aggregate repeated measurements using the median pIC50 to obtain one activity label per molecule.
* Validate molecular SMILES with RDKit.
* Save the cleaned dataset as a CSV file.

After aggregation, the dataset contained 6,288 unique molecules.

Important Data-Curation Lessons

A database value being present does not automatically mean that it is suitable for machine learning.

For example:

After aggregation, the dataset contained 6,288 unique molecules.


The main data-preparation steps were:X
$$
