## Day9- 29th Sep, 2026

#Molecular Representation

There are three types of molecular representations are considered here:
Descriptor
Binarymorgan fingerprint
Count morgan fingerprint

I learned about the extra physological property along with logP, MWW, TPSA, HBA/HBD, no of rotating bond, Rings
rdMolDescriptors.CalcFractionCSP3(mol) calculates the fraction of carbon atoms in a molecule that are sp³ hybridized.
Increasing a molecule's \[F_{sp^{3}}\] introduces several crucial advantages: 
* Improved Solubility
* Higher Success Rates in Clinical Trials
* Better Target Selectivity
* Lower Melting Points

The Balance: Too little sp³ character makes a molecule flat and poorly soluble. Too much sp³ character can sometimes make chemical synthesis highly complex or introduce too many flexible bonds, which can hurt binding affinity.  

# Hypthesis
Count Morgan may improve predictions when the multiplicity of molecular environments matters for AChE bioactivity. Whether that improvement actually occurs must be determined experimentally.

Both Morgan representations substantially outperform the eight global descriptors.
