# Day 35 — Learning Journal
**Date:** 26 October 2026  
**Phase:** Month 2 — Geometric Molecular Machine Learning and ML for Molecular Dynamics  
**Topic:** Molecular Geometry, Symmetry, Invariance and Equivariance  
**Status:** Completed

## 1. Objective of the Day

The objective was to understand how three-dimensional atomic coordinates can be incorporated into molecular machine learning models while respecting fundamental physical symmetries.

The central scientific question was:

**How can a neural network learn molecular properties from 3D atomic coordinates without depending on the arbitrary position or orientation of a molecule?**

This study establishes the mathematical foundation for geometric graph neural networks, particularly SchNet and EGNN.

## 2. Concepts Studied

### Molecular coordinates and internal geometry

A molecular configuration containing \(N\) atoms is represented by

$$
\mathbf R=(\mathbf r_1,\mathbf r_2,\ldots,\mathbf r_N),
$$

where each atomic position is a three-dimensional Cartesian vector.

Potential energy depends on the molecular configuration. However, for an isolated molecule with no external position-dependent fields, rigid translations and rotations do not change its internal potential energy.

Internal structural changes, such as bond stretching, angle bending, and torsional rotations, can change the energy.

### Translation and rotation invariance

For a translation vector \(\mathbf t\) and an orthogonal rotation matrix \(Q\),

$$
\mathbf r_i'=Q\mathbf r_i+\mathbf t.
$$

Pairwise distances satisfy

$$
d_{ij}'=d_{ij}.
$$

Consequently, an appropriately constructed molecular energy function should satisfy

$$
U(Q\mathbf R+\mathbf t)=U(\mathbf R).
$$

This property is called invariance.

### Force equivariance

Atomic forces are defined by

$$
\mathbf F_i=-\nabla_{\mathbf r_i}U.
$$

Unlike scalar energies, forces transform as vectors:

$$
\mathbf F_i(Q\mathbf R+\mathbf t)=Q\mathbf F_i(\mathbf R).
$$

This property is called equivariance.

An important conceptual distinction was established: **energy remains unchanged under a global rigid-body rotation, whereas force vectors rotate with the molecular system.**

## 3. Computational Work

### Exercise 1 — Generation of a 3D molecular conformer

Used RDKit to generate a three-dimensional conformer of ethanol (`CCO`) with explicit hydrogen atoms.

Results:

- Successful conformer generation (embedding status 0).
- Nine atoms.
- Cartesian coordinate array of shape `(9, 3)`.
- Atomic coordinates expressed in ångströms.

### Exercise 2 — Pairwise distance matrix

Computed all interatomic distances using SciPy.

The resulting distance matrix had shape `(9, 9)` and was symmetric with zero diagonal elements.

Selected bond lengths:

| Bond | Calculated distance |
|---|---:|
| C–C | 1.50585 Å |
| C–O | 1.37810 Å |
| O–H | 0.976 Å |

The calculated distances were physically reasonable for the generated conformer. No force-field minimization was performed.

### Exercise 3 — Translation invariance

Translated all atomic coordinates by

$$
\mathbf t=(5,-3,2)\ \text{Å}.
$$

Results:

- Maximum pairwise distance difference: \(8.88\times10^{-16}\) Å.
- Distance invariance test: `True`.

Conclusion: A global translation changes Cartesian coordinates but preserves internal pairwise distances.

### Exercise 4 — Rotational invariance

Applied a 90° rotation around the \(z\)-axis.

Results:

- Maximum pairwise distance difference: \(4.44\times10^{-16}\) Å.
- Distance invariance test: `True`.
- Rotation matrix orthogonality test: `True`.

Conclusion: Pairwise distances remain invariant under rigid-body rotations.

### Exercise 5 — Vector transformation

Rotated the vector

$$
\mathbf F=(1,0,0)
$$

by 90° around the \(z\)-axis.

The transformed vector was

$$
\mathbf F'=(0,1,0).
$$

The vector magnitude remained equal to 1.

This demonstrated the transformation of a vector under rotation, although it was not yet a force calculated from a potential.

### Exercise 6 — Harmonic bond energy and force

Considered a simplified harmonic potential for the ethanol C–C bond:

$$
U(r)=\frac{k}{2}(r-r_0)^2,
$$

with \(k=100\) in arbitrary consistent energy units and \(r_0=1.54\) Å.

Calculated results:

$$
U=0.05830653
$$

$$
\mathbf F_2=(3.26865,-0.83757,-0.52509).
$$

The C–C distance was slightly smaller than the selected equilibrium bond length, producing a restoring force directed toward increasing the bond length.

This was a simplified two-atom interaction, not a complete molecular force-field calculation.

### Exercise 7 — Energy invariance and force equivariance

Rotated both carbon atoms by 90° and recalculated the harmonic energy and force.

| Quantity | Original | Rotated |
|---|---|---|
| Energy | 0.058306527957115196 | 0.05830652795711443 |
| Force \(F_x\) | 3.26865 | 0.83757 |
| Force \(F_y\) | -0.83757 | 3.26865 |
| Force \(F_z\) | -0.52509 | -0.52509 |

Numerical verification:

- Energy invariant: `True`.
- Force equivariant: `True`.

The energy difference was approximately \(7.63\times10^{-16}\), consistent with floating-point roundoff.

This was the most important computational result of the day. It verified the expected symmetry properties of energy and forces for a simple physically defined potential.

## 4. Connection to Geometric Graph Neural Networks

The previous AChE bioactivity project used a GINE model based on molecular connectivity and atom/bond features.

A conventional 2D molecular graph does not explicitly encode a particular three-dimensional molecular conformation.

Geometric neural networks extend molecular representations by incorporating atomic coordinates and spatial relationships.

**SchNet** uses interatomic distances in continuous-filter convolutions to construct invariant scalar representations. Forces derived from its predicted energy can transform equivariantly.

**EGNN** uses relative coordinates and invariant geometric information to construct equivariant message-passing operations, including coordinate updates.

The symmetry principles verified today are therefore directly relevant to the design and evaluation of these architectures.

## 5. Scientific Insights

1. Molecular potential energy depends on internal configuration, but not on arbitrary global translation or orientation for an isolated system.
2. Cartesian coordinates are not invariant under rigid-body transformations.
3. Pairwise distances are invariant under global translations and rotations.
4. Scalar energy predictions should satisfy the relevant physical invariances.
5. Vector forces should transform equivariantly under rotations.
6. Physical symmetry can be incorporated into a machine learning architecture instead of relying entirely on training data to learn it.
7. Numerical symmetry tests provide a useful way to validate geometric molecular ML implementations.

## 6. Limitations and Scope

Today's work used one RDKit-generated ethanol conformer and a simplified harmonic bond potential.

No geometric neural network was trained, and no molecular dynamics simulation was performed.

The calculations verified mathematical and physical symmetry properties of distances and a simple energy model. They do not establish the predictive accuracy or generalization performance of SchNet or EGNN.

## 7. Progress Toward Industry Portfolio

Day 35 marks the beginning of the geometric molecular machine learning phase.

The concepts developed today will support **Project 3 — Geometry-Aware Molecular Learning with SchNet or EGNN**.

They also provide foundations for physically informed ML models, learned molecular potentials, force prediction, and subsequent integration with molecular dynamics.

This work builds on existing expertise in statistical mechanics and molecular simulation while extending practical skills toward modern geometric deep learning.

The industry application target remains late December 2026.

## 8. Next Day — Day 36

**Topic:** Spatial Neighborhood Graphs and Geometric Message Passing.

Planned activities:

- Distinguish chemical bond graphs from spatial neighborhood graphs.
- Construct a radius-based graph using ethanol's 3D coordinates.
- Explore cutoff distances and neighbor selection.
- Examine geometric edge features.
- Connect spatial graph construction to SchNet and EGNN.

The work will continue with small, individually verified coding exercises.

## 9. Daily Reflection

Today I connected familiar physical concepts from molecular simulation with the mathematical requirements of geometric machine learning.

The most important achievement was moving beyond abstract definitions of invariance and equivariance to numerical verification using molecular coordinates, a potential energy function, and its corresponding force.

I now have a clearer understanding of why geometric neural networks must distinguish between invariant scalar quantities and equivariant vector quantities.

**Day 35 completed successfully.**
