# Day 36 - Scientific Learning Journal

**Date:** 26 October 2026  
**Phase:** Month 2 — Geometric Molecular Machine Learning and ML for Molecular Dynamics  
**Topic:** Spatial Neighborhood Graphs, Radial Basis Functions, and Geometric Message Passing  
**Status:** Completed

---

## 1. Objective of the Day

The objective was to understand how three-dimensional molecular coordinates can be converted into spatial graphs and how geometric information can be incorporated into graph neural network message passing.

The central scientific question was:

**How can a molecular graph neural network incorporate interatomic distances and spatial neighborhoods while preserving physical symmetries?**

This work builds directly on Day 35, where translation invariance, rotational invariance, and force equivariance were investigated.

The longer-term goal is to develop the mathematical and computational foundations required to understand and implement geometric GNNs such as SchNet and EGNN.

## 2. Theoretical Concepts Studied

### 2.1 Chemical graphs versus spatial graphs

A conventional molecular graph is represented as

$$
G=(V,E),
$$

where atoms correspond to nodes and covalent bonds correspond to edges.

In a spatial molecular graph, edges can instead be constructed using interatomic distances.

For atoms \(i\) and \(j\),

$$
d_{ij}=\|\mathbf r_i-\mathbf r_j\|.
$$

A spatial connection is introduced when

$$
0<d_{ij}<r_c,
$$

where \(r_c\) is a selected cutoff radius.

Spatial connections are not necessarily chemical bonds. They indicate that two atoms are sufficiently close to exchange information during geometric message passing.

### 2.2 Undirected spatial edges

Because

$$
d_{ij}=d_{ji},
$$

the spatial adjacency matrix is symmetric:

$$
A_{ij}=A_{ji}.
$$

Each undirected edge appears twice in the adjacency matrix.

Consequently,

$$
|E|=\frac{1}{2}\sum_{i,j}A_{ij}.
$$

Equivalently, the sum of node degrees satisfies the handshaking lemma:

$$
\sum_i \deg(i)=2|E|.
$$

### 2.3 Gaussian radial basis functions

A binary adjacency matrix identifies neighboring atoms but does not describe their exact distances.

To construct continuous geometric edge features, Gaussian radial basis functions were introduced:

$$
\phi_k(d)=\exp[-\gamma(d-\mu_k)^2],
$$

where \(\mu_k\) is the center of the Gaussian and \(\gamma\) controls its width.

The scalar distance is expanded into a feature vector:

$$
d_{ij}\rightarrow \boldsymbol\phi(d_{ij}) = [\phi_1(d_{ij}),\ldots,\phi_K(d_{ij})].
$$

This representation provides smooth distance-dependent features that can be processed by a neural network.

### 2.4 Geometric message passing

A simplified distance-dependent message was defined as

$$
m_{ij}=Z_jw(d_{ij}),
$$

where \(Z_j\) is the atomic number of neighboring atom \(j\).

The distance-dependent filter was constructed as

$$
w(d_{ij})=
\sum_k a_k\phi_k(d_{ij}).
$$

Messages were aggregated over spatial neighbors:

$$
m_i=\sum_{j\in\mathcal N(i)}m_{ij}.
$$

This provides a simplified illustration of the continuous-filter message-passing principle used in SchNet.

### 2.5 Geometric invariance

Because the message function depends only on atomic identities and interatomic distances, the aggregated scalar message should remain invariant under global translations and rotations:

$$
m_i(Q\mathbf R+\mathbf t)=m_i(\mathbf R).
$$

This invariance is built into the mathematical representation.

## 3. Computational Work and Results

### Exercise 1 — Spatial adjacency matrix

Reused the RDKit-generated ethanol conformer from Day 35.

The molecule contained nine atoms, including explicit hydrogens.

A spatial adjacency matrix was constructed using a cutoff radius of 2.0 Å.

Results:

- Number of atoms: 9.
- Number of undirected spatial edges: 12.
- Symmetric adjacency matrix.
- Self-connections excluded.

The number of spatial neighbors per atom was

```text
[4 4 2 3 3 3 2 2 1]
```

The total degree was 24, corresponding to 12 undirected edges.

### Exercise 2 — Chemical versus spatial connectivity

Constructed a chemical adjacency matrix using RDKit covalent bond information and compared it with the spatial adjacency matrix.

Results:

| Graph | Number of edges |
|---|---:|
| Chemical bond graph | 8 |
| Spatial graph, 2.0 Å cutoff | 12 |
| Additional nonbonded spatial edges | 4 |

The four additional connections were:

| Atom pair | Distance |
|---|---:|
| H3–H4 | 1.782 Å |
| H3–H5 | 1.773 Å |
| H4–H5 | 1.839 Å |
| H6–H7 | 1.764 Å |

These hydrogen pairs are spatially close but are not connected by covalent bonds.

The result demonstrates that spatial graphs contain geometric neighborhood information beyond conventional molecular connectivity.

### Exercise 3 — Cutoff-radius dependence

Investigated how the spatial graph changes as the cutoff radius increases.

| Cutoff (Å) | Undirected edges | Average neighbors |
|---|---:|---:|
| 1.2 | 6 | 1.33 |
| 1.5 | 7 | 1.56 |
| 2.0 | 12 | 2.67 |
| 2.5 | 25 | 5.56 |
| 3.0 | 30 | 6.67 |

For nine atoms, the maximum number of undirected edges without self-connections is

$$
E_{\max}=\frac{N(N-1)}{2}=36.
$$

At a cutoff of 3.0 Å, the spatial graph therefore contained 30 of the 36 possible edges.

The experiment demonstrated that graph connectivity is strongly dependent on the cutoff radius.

A larger cutoff increases the number of potential message-passing interactions, but also increases computational cost.

### Exercise 4 — Gaussian radial basis expansion

Used the C–C distance

$$
d_{\mathrm{CC}}=1.505851346\ \text{Å}
$$

with Gaussian centers

$$
\boldsymbol\mu=[0,\ 0.75,\ 1.5,\ 2.25,\ 3.0]\ \text{Å}
$$

and \(\gamma=2.0\) in consistent inverse-square distance units.

The calculated radial basis features were

```text
[0.0107 0.3190 0.9999 0.3304 0.0115]
```

The Gaussian centered at 1.5 Å had the largest activation, approximately 0.9999.

This demonstrated how an interatomic distance can be represented by a smooth vector of overlapping basis-function responses.

### Exercise 5 — Radial basis visualization

Plotted five Gaussian radial basis functions over the distance interval 0–3 Å.

A vertical reference line marked the C–C bond distance.

The visualization confirmed that:

- Each Gaussian has its maximum near its center.
- Neighboring Gaussian functions overlap.
- A given distance can activate multiple basis functions.
- The representation changes smoothly with interatomic distance.

This provides a useful foundation for understanding learned distance-dependent filters.

### Exercise 6 — Simplified geometric message passing

Implemented a scalar message-passing operation using atomic numbers and fixed Gaussian filter coefficients.

The coefficients were

$$
\mathbf a=[0.1,\ 0.3,\ 1.0,\ 0.3,\ 0.1].
$$

Messages received by the first carbon atom were:

| Neighbor | Distance (Å) | Filter | Message |
|---|---:|---:|---:|
| Carbon 1 | 1.506 | 1.1970 | 7.1818 |
| Hydrogen 3 | 1.112 | 1.0014 | 1.0014 |
| Hydrogen 4 | 1.113 | 1.0027 | 1.0027 |
| Hydrogen 5 | 1.119 | 1.0084 | 1.0084 |

The total aggregated message was

$$
\boxed{m_0=10.194261159545166}.
$$

The carbon neighbor contributed the largest message because its atomic number was six, compared with one for hydrogen, and because of the selected distance-dependent filter.

This contribution should not be interpreted as a physical interaction energy or a quantitative measure of chemical importance.

The experiment demonstrated the computational mechanism of distance-dependent message aggregation.

### Exercise 7 — Translation and rotation invariance of message passing

Calculated the aggregated message for the original, translated, and rotated molecular configurations.

| Configuration | Aggregated message |
|---|---:|
| Original | 10.194261159545166 |
| Translated | 10.194261159545166 |
| Rotated | 10.194261159545166 |

Numerical verification:

```text
Translation invariant? True
Rotation invariant? True
```

The aggregated messages agreed to the displayed numerical precision.

This confirmed that the simplified message-passing operation preserved translation and rotation invariance for the tested molecular configuration.

The result follows from the use of atomic numbers, invariant interatomic distances, and distance-based neighborhood selection.

## 4. Connection to SchNet and Geometric Deep Learning

The previous AChE bioactivity project used a GINE graph neural network based on molecular connectivity and chemical atom/bond features.

Today's exercises demonstrated how molecular graph learning can be extended to incorporate three-dimensional geometry.

A simplified SchNet-style continuous-filter convolution can be expressed as

$$
\mathbf m_i= \sum_{j\in\mathcal N(i)} \mathbf h_j\odot W_\theta(d_{ij}),
$$

where \(\mathbf h_j\) is a learned atomic feature vector and \(W_\theta\) is a learned distance-dependent filter.

Today's implementation used scalar atomic numbers and manually selected filter coefficients rather than learned representations.

Nevertheless, it reproduced the central computational idea of geometry-dependent message aggregation.

Because the filters depend on invariant distances, the resulting scalar representations can preserve rotational and translational invariance.

This provides the foundation for studying SchNet and, subsequently, equivariant architectures such as EGNN.

## 5. Main Scientific Insights

1. Chemical graphs and spatial graphs encode different types of molecular information.
2. Spatial edges can connect nonbonded atoms that are close in three-dimensional space.
3. The cutoff radius controls graph connectivity and computational cost.
4. A binary spatial adjacency matrix does not retain the precise distance between connected atoms.
5. Gaussian radial basis functions provide smooth numerical representations of interatomic distances.
6. Distance-dependent filters allow geometric information to influence message passing.
7. Aggregating messages from spatial neighbors produces geometry-dependent atomic representations.
8. Scalar message-passing operations based on atomic identities and distances can be invariant by construction.
9. Geometric invariance can be verified numerically before training a neural network.

## 6. Limitations and Scope

The calculations were performed using one ethanol conformer.

The 2.0 Å cutoff was selected for illustration and was not optimized for a physical application.

The message-passing operation used fixed coefficients and scalar atomic-number features. No learnable neural-network parameters were introduced.

No model training, property prediction, force prediction, or generalization evaluation was performed.

The numerical invariance tests covered specific rigid-body transformations. The broader invariance property is supported by the mathematical structure of the representation.

Furthermore, a hard cutoff introduces a discontinuity in neighborhood membership when an interatomic distance crosses the cutoff. Smooth cutoff functions can help address this issue in molecular energy models.

## 7. Progress Toward the Industry Portfolio

Day 36 contributes to **Project 3 — Geometry-Aware Molecular Learning with SchNet or EGNN**.

The work establishes practical familiarity with:

- RDKit molecular geometry.
- Spatial graph construction.
- Distance-based neighbor selection.
- Gaussian radial basis features.
- Geometric message passing.
- Numerical symmetry verification.
- Scientific interpretation of geometric representations.

These skills extend the molecular graph learning experience gained in Project 1 toward three-dimensional molecular machine learning.

They also provide foundations for learned molecular potentials, force prediction, and the integration of geometric representations with molecular dynamics.

The target for beginning industry applications remains late December 2026.

## 8. Next Day — Day 37

**Topic:** Learnable Distance-Dependent Filters and Introduction to SchNet-Style Message Passing.

Planned work:

- Review the distinction between fixed and learnable message-passing filters.
- Introduce atomic embeddings.
- Construct a small distance-dependent neural-network filter in PyTorch.
- Understand how geometric messages are aggregated and used to update atomic representations.
- Verify appropriate symmetry properties of the resulting operations.

The implementation will proceed through small, individually verified coding exercises.

## 9. Daily Reflection

Today I progressed from the geometric invariance principles established on Day 35 to constructing an explicit spatial molecular graph and a simplified geometric message-passing operation.

The most important conceptual development was understanding how interatomic distances can influence information exchange between atoms without introducing dependence on the arbitrary global orientation or position of a molecule.

The cutoff-radius analysis also demonstrated that spatial graph construction involves modeling choices that affect both the information available to the network and its computational cost.

The final invariance test showed that symmetry-preserving geometric representations can be constructed directly from physically meaningful quantities.

### Additional Scientific Discussion — Why Use Gaussian Radial Basis Functions Instead of Raw Interatomic Distances?

**Central question:** What is the advantage of representing interatomic distances using Gaussian radial basis functions rather than supplying the scalar distance directly to a geometric neural network?

The primary motivation is to make it easier for a neural network to learn complex, nonlinear, distance-dependent interactions.

A molecular interaction can vary strongly with interatomic separation. For example, the Lennard-Jones potential,

$$
V(r)=4\epsilon\left[\left(\frac{\sigma}{r}\right)^{12}-\left(\frac{\sigma}{r}\right)^6\right],
$$

contains a strongly repulsive short-range region and an attractive region at larger separations. Learning such nonlinear behavior directly from a scalar distance is possible, but an appropriate basis expansion can provide a more convenient representation.

Gaussian radial basis functions transform a scalar distance into a vector of smooth, localized features:

$$
\phi_k(r)=\exp[-\gamma(r-\mu_k)^2],
$$

where \(\mu_k\) specifies the center of each Gaussian and \(\gamma\) controls its width.

For the C–C bond in ethanol, the Day 36 calculation gave

$$
r_{\mathrm{CC}}=1.50585\ \text{Å}
$$

and

$$
\boldsymbol\phi(r_{\mathrm{CC}})
=
[0.0107,\ 0.3190,\ 0.9999,\ 0.3304,\ 0.0115].
$$

The Gaussian centered at 1.5 Å had the highest activation because its center was closest to the C–C bond distance. Neighboring Gaussians also contributed, producing a smooth, overlapping representation.

**Connection to mathematical physics:** This approach is analogous to expanding an unknown function in a set of basis functions:

$$
f(r)\approx\sum_{k=1}^{K}a_k\phi_k(r).
$$

The coefficients \(a_k\) can be learned from molecular data, allowing the model to construct flexible distance-dependent filters. Unlike a Fourier expansion, Gaussian basis functions are localized around particular distance ranges.

In a SchNet-style continuous-filter convolution, the radial basis features are processed by a learnable neural network:

$$
r_{ij}
\rightarrow
\boldsymbol\phi(r_{ij})
\rightarrow
$$

The resulting filter controls the contribution of neighboring atom \(j\) to the representation of atom \(i\).

**Important scientific distinctions:**

- Gaussian RBF expansion does not introduce new physical information; it re-expresses the original interatomic distance.
- A sufficiently expressive neural network can also learn nonlinear interactions directly from scalar distances. Gaussian expansion is a useful modeling choice, not a mathematical necessity.
- Both raw distances and Gaussian RBF features preserve rotational and translational invariance because they depend only on interatomic separations.
- Gaussian expansion does not automatically guarantee physical accuracy, correct force behavior, or smoothness at a hard cutoff. These depend on the overall model architecture and training procedure.

**Key scientific takeaway:** Gaussian radial basis functions provide a smooth, localized basis for representing distance-dependent interactions. From a statistical physics perspective, their role is best understood as a basis-function representation of an unknown interaction kernel, making nonlinear geometric relationships more convenient for a neural network to learn.

This understanding provides a direct conceptual bridge to Day 37, where fixed Gaussian filter coefficients will be replaced by learnable neural-network parameters.

I now have a clearer understanding of the computational foundations underlying SchNet-style continuous-filter convolutions.

**Day 36 completed successfully.**
