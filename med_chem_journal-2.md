## Medicinal Chemistry - Day 2: Stereochemistry and Molecular Recognition

Today, I studied how **stereochemistry influences molecular recognition, binding affinity, potency, and selectivity**.

Enantiomers are nonsuperimposable mirror-image molecules with identical connectivity but different three-dimensional arrangements. Since biological targets such as enzymes are chiral, two enantiomers can interact differently with the same binding pocket, resulting in different binding affinities.

From thermodynamics, a binding free-energy difference of approximately **1.36 kcal/mol corresponds to a tenfold difference in binding affinity** at room temperature. Thus, even small changes in molecular interactions can significantly affect biological activity.

I also learned how stereochemistry is represented in SMILES:
- `@` and `@@` indicate local tetrahedral chirality.
- `/` and `\` encode double-bond stereochemistry.
- RDKit can assign absolute configurations such as R and S.

**Connection to my AChE GNN project:** My original 2D GNN did not explicitly encode tetrahedral chirality. Adding chirality features did not improve validation performance (MAE increased from 0.882 to 0.897), possibly because the dataset contained relatively few informative stereoisomer pairs.

**Key takeaway:** Molecular connectivity alone is insufficient to describe all structure–activity relationships. Correct stereochemical representation is important, but encoding chirality does not guarantee that a machine learning model can learn stereoselective biological activity.

**Next topic:** Real examples of stereoselective drug–target interactions and their implications for molecular docking and 3D GNNs.
