# Molecular ML Journey: From Molecular Simulation to Data-Driven Molecular Modeling

This repository documents my week-by-week journey into molecular machine learning and data-driven molecular modeling.

With a background in physics and extensive experience in biomolecular simulations and Soft Matter, I am particularly interested in the intersection of physics, computation, and data. 
My goal is to build on my experience in molecular dynamics, statistical mechanics, and kinetic modeling while developing practical skills in cheminformatics, molecular machine learning, and ML-guided molecular simulation.

This repository will document what I learn, what I build, the questions I encounter, and how I connect new machine-learning concepts with my existing understanding of molecular systems.

This is an evolving journey toward applying physics-based and data-driven approaches to complex molecular problems.

Day1-21.09.2026
## What I Learned Today

1. Today I learned how to use **RDKit**, a cheminformatics toolkit, to represent molecules and calculate simple molecular descriptors that encode different aspects of molecular structure and physicochemical properties.

2. I learned how to organize molecular information and calculated descriptors in a **Pandas DataFrame**, creating a structured dataset that can later be used for analysis and machine learning.

3. I revisited several important molecular descriptors, including:

   * **LogP** — a measure related to lipophilicity/hydrophobicity and molecular partitioning between aqueous and nonpolar environments.
   * **TPSA (Topological Polar Surface Area)** — a descriptor of the polar surface contribution of a molecule.
   * **HBD/HBA (Hydrogen-Bond Donors/Acceptors)** — descriptors related to a molecule's ability to participate in hydrogen-bonding interactions.
   * **Rotatable bonds** — a simple measure related to molecular flexibility.

4. I began to understand how these physicochemical properties are relevant to **drug-like behavior**, including aqueous solubility, oral absorption, membrane permeability, and the ability of some molecules to cross the **blood–brain barrier (BBB)**. Rather than being controlled by a single descriptor, these behaviors depend on a combination of molecular properties such as lipophilicity, polarity, hydrogen bonding, molecular size, flexibility, and ionization state.

5. Comparing **ibuprofen and glucose** helped me understand how molecular structure influences LogP. Ibuprofen has a relatively large hydrophobic region and a higher LogP, whereas glucose contains many hydroxyl groups that interact strongly with water and therefore has a much lower LogP.

6. Comparing **glucose and benzene** demonstrated the meaning of TPSA. Glucose has a high TPSA because of its many polar oxygen-containing groups, whereas benzene has a TPSA of zero because it lacks the heteroatoms and polar functional groups represented by this descriptor.

## Connection to My MD Background

From my molecular-dynamics perspective, I understand that descriptors such as **LogP, TPSA, HBD/HBA, and rotatable-bond counts are simplified molecular descriptors**. They do not directly describe the full conformational ensemble, solvent dynamics, or time-dependent molecular interactions that can be obtained from MD simulations.

This leads to an interesting question for me:

**Can MD-derived information about conformational ensembles, solvation, and molecular interactions provide additional predictive information beyond static 2D molecular descriptors?**
