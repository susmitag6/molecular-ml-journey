# Medicinal Chemistry — Lesson 21 Journal
**Topic:** Stereochemistry, Drug Activity, and Stereoselective Metabolism  
**Example:** Omeprazole and Esomeprazole

## Objective

Today, I aimed to understand how stereochemistry influences drug behavior, particularly why two enantiomers can produce different pharmacological outcomes even when they act on the same molecular target.

## What I Learned

### 1. Real-world example: Omeprazole and Esomeprazole

I studied omeprazole, a proton pump inhibitor used to reduce gastric acid secretion.

Omeprazole contains a chiral sulfur center and exists as two enantiomers:

- **R-omeprazole**
- **S-omeprazole (esomeprazole)**

Omeprazole is administered as a mixture of both enantiomers, whereas esomeprazole contains the S-enantiomer.

Both ultimately inhibit the gastric proton pump, but they can behave differently in the body because biological systems are chiral.

### 2. Binding affinity, potency, and pharmacokinetics

I learned to distinguish three important concepts:

| Concept | Meaning |
|---|---|
| Binding affinity | How strongly a molecule binds to its target |
| Potency | The concentration required to produce a specified biological effect |
| Pharmacokinetics | How a drug is absorbed, distributed, metabolized, and eliminated |

An important insight was that **a stereoisomer can show improved therapeutic performance without necessarily having stronger intrinsic binding affinity**.

In the case of esomeprazole, stereoselective metabolism contributes to differences in drug exposure.

### 3. Why stereochemistry matters biologically

Proteins, enzymes, and receptors are three-dimensional, chiral structures.

Consequently, two mirror-image molecules can interact differently with the same biological environment.

This can influence:

- Target recognition and binding
- Enzyme-mediated metabolism
- Drug exposure and duration of action
- Potency and selectivity

I learned that stereochemistry affects not only molecular recognition at the therapeutic target but also how the body processes the drug.

### 4. Connection to Molecular Machine Learning

I connected this lesson to our AChE bioactivity prediction project.

Our 2D graph neural network represents molecular connectivity using atom and bond features. Adding chirality descriptors provides stereochemical information, but it does not explicitly represent the molecule's three-dimensional arrangement inside a protein binding pocket.

A geometric GNN can use atomic coordinates to describe spatial relationships. However, a model based exclusively on pairwise atomic distances cannot distinguish isolated mirror-image structures because their distance matrices are identical.

Therefore, geometric ML architectures need appropriate stereochemical information or representations sensitive to chirality when enantiomer discrimination matters.

This will become particularly relevant when we study 3D geometric GNNs and protein–ligand interactions in Months 2 and 3.

## Key Takeaway

**Different biological activities of enantiomers do not always arise from differences in target-binding affinity.**

Stereoselective metabolism and pharmacokinetics can also produce important differences in drug behavior.

I should therefore distinguish target affinity, biological potency, and pharmacokinetic effects when interpreting experimental drug activity or developing molecular ML models.

## Conclusion

Today's lesson strengthened my understanding of how molecular stereochemistry influences drug behavior.

The omeprazole–esomeprazole example demonstrated that chirality can affect pharmacokinetics even when two enantiomers share the same therapeutic target.

I also recognized why explicitly representing three-dimensional molecular geometry and stereochemistry is important for future protein–ligand ML applications.

## Next Lesson

**Noncovalent interactions in drug binding:** Hydrogen bonding, hydrophobic interactions, electrostatics, van der Waals interactions, and the role of solvation and entropy in binding affinity.

In later lessons, I will also study **antibodies**, including their structure, antigen recognition, binding affinity, therapeutic applications, and connections to molecular dynamics and geometric machine learning.
