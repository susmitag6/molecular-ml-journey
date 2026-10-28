# Day 38 — Scientific Learning Journal

**Date:** 28 October 2026  
**Phase:** Geometric Molecular Machine Learning  
**Portfolio Project:** Project 3 — Geometry-Aware Molecular Learning  
**Topic:** Learned Atomic Embeddings and Environment-Dependent Atomic Representations  
**Status:** Completed — Six coding exercises

---

## 1. Objective of the Day

The objective of Day 38 was to incorporate chemical identity into the geometric message-passing framework developed during Days 35–37.

Previously, I had constructed spatial molecular graphs, Gaussian radial basis distance features, learnable continuous filters, and vector-valued message aggregation. However, on Day 37, every neighboring atom was assigned the same manually specified feature vector.

That simplification meant that the model could distinguish neighboring atoms by distance but not explicitly by chemical element.

The central question for Day 38 was:

**How can a geometric graph neural network combine atomic identity and three-dimensional molecular geometry to construct environment-dependent atomic representations?**

The computational goals were to:

1. Construct trainable atomic embeddings using PyTorch.
2. Combine atomic embeddings with learnable distance-dependent filters.
3. Isolate the effect of atomic identity by comparing different elements at the same distance.
4. Aggregate messages using the actual chemical identities and distances of ethanol's neighboring atoms.
5. Update an atomic representation using a residual connection.
6. Demonstrate that two carbon atoms with identical initial embeddings can acquire different representations because their chemical environments differ.

All six exercises were completed successfully.

---

## 2. Molecular System and Computational Setup

The molecular system was the ethanol conformer generated on Day 35 using RDKit.

The molecule contained nine atoms, including explicit hydrogens:

$$
\mathrm{CH_3-CH_2-OH}
$$

The atomic numbers were:

```text
tensor([6, 6, 8, 1, 1, 1, 1, 1, 1])
```

The atom indexing was:

| Atom index | Element | Chemical role |
|---|---|---|
| 0 | Carbon | Methyl carbon, CH₃ |
| 1 | Carbon | Methylene carbon, CH₂ |
| 2 | Oxygen | Hydroxyl oxygen |
| 3–5 | Hydrogen | Methyl hydrogens |
| 6–7 | Hydrogen | Methylene hydrogens |
| 8 | Hydrogen | Hydroxyl hydrogen |

The spatial graph was defined using an interatomic distance cutoff of 2.0 Å:

$$
\mathcal N(i)=
\{j\ne i:d_{ij}<2.0\ \text{Å}\}.
$$

The distance-filter network from Day 37 was reused:

```python
vector_filter_net = nn.Sequential(
    nn.Linear(5, 8),
    nn.SiLU(),
    nn.Linear(8, 4)
)
```

The network received five Gaussian radial basis features and produced a four-dimensional distance-dependent filter.

The existing Gaussian basis was

$$
\phi_k(d)=\exp[-\gamma(d-\mu_k)^2],
$$

with centers

$$
\boldsymbol\mu=[0,\ 0.75,\ 1.5,\ 2.25,\ 3.0]\ \text{Å}
$$

and width parameter

$$
\gamma=2.0\ \text{Å}^{-2}.
$$

The experiments used PyTorch, NumPy, RDKit, and the previously computed ethanol distance matrix.

---

## 3. Theoretical Background

### 3.1 Atomic identity as a categorical variable

The atomic number \(Z_i\) identifies the chemical element of atom \(i\).

For ethanol:

$$
Z_{\mathrm C}=6,\qquad Z_{\mathrm O}=8,\qquad Z_{\mathrm H}=1.
$$

An atomic number is an element identifier, but using it directly as a continuous scalar input may suggest an artificial numerical relationship between elements.

A learned embedding instead maps each element to a trainable feature vector:

$$
\boxed{\mathbf h_i^{(0)} = E_\theta(Z_i)}
$$

where

$$
\mathbf h_i^{(0)}\in\mathbb R^F.
$$

For this exercise, the embedding dimension was

$$
F=4.
$$

The embedding table was a trainable matrix:

$$
E_\theta\in\mathbb R^{10\times4}.
$$

Each atomic number selects a row of this matrix.

The embedding values are initially random. They do not automatically correspond to electronegativity, atomic mass, charge, or any other physical quantity.

### 3.2 Identical elements share initial embeddings

Because the embedding lookup depends only on atomic number,

$$
Z_i=Z_j \quad\Longrightarrow\quad \mathbf h_i^{(0)}=\mathbf h_j^{(0)}.
$$

Therefore, the two carbon atoms in ethanol initially receive identical feature vectors.

However, the carbon atoms are chemically distinguishable:

- Carbon 0 belongs to the methyl group and has three hydrogen neighbors.
- Carbon 1 belongs to the methylene group and has an oxygen neighbor.

The embedding layer alone cannot distinguish these environments.

That distinction emerges through message passing.

### 3.3 Combining atomic identity and geometry

The message from atom \(j\) to atom \(i\) was defined as

$$
\boxed{ \mathbf m_{ij} = \mathbf h_j^{(0)} \odot W_\theta(\boldsymbol\phi(d_{ij})) }
$$

where:

- \(\mathbf h_j^{(0)}\) is the learned embedding of the sending atom.
- \(d_{ij}\) is the interatomic distance.
- \(\boldsymbol\phi(d_{ij})\) is the Gaussian radial basis expansion.
- \(W_\theta\) is a nonlinear neural-network filter.
- \(\odot\) denotes elementwise multiplication.

This operation combines chemical identity and geometry.

At a fixed distance, different elements can generate different messages because their embeddings differ.

For the same element, different distances can generate different messages because the neural filter depends on distance.

### 3.4 Neighborhood aggregation

Messages from all neighbors of atom \(i\) were summed:

$$
\boxed{\mathbf m_i = \sum_{j\in\mathcal N(i)} \mathbf m_{ij}}
$$

or equivalently,

$$
\mathbf m_i = \sum_{j\in\mathcal N(i)} \mathbf h_j^{(0)} \odot W_\theta(\boldsymbol\phi(d_{ij})).
$$

This produces a fixed-dimensional representation of the local environment.

The summation is independent of neighbor ordering, an important property for graph neural networks.

### 3.5 Residual atomic feature update

The aggregated message was incorporated into the atom's existing representation through a simple residual update:

$$
\boxed{\mathbf h_i^{(1)} = \mathbf h_i^{(0)}+\mathbf m_i
}
$$

This combines the initial element identity with information received from neighboring atoms.

The updated representation can therefore depend on both chemical composition and local geometry.

This is a simplified SchNet-style update. A full SchNet interaction block contains additional learned transformations.

---

## 4. Computational Exercises and Results

### Exercise 1 — Constructing learned atomic embeddings

The first task was to map ethanol's atomic numbers into trainable four-dimensional feature vectors.

The main PyTorch operation was:

```python
torch.manual_seed(42)

atomic_numbers = np.array([
    atom.GetAtomicNum() for atom in mol.GetAtoms()
])

Z = torch.tensor(
    atomic_numbers,
    dtype=torch.long
)

atom_embedding = nn.Embedding(
    num_embeddings=10,
    embedding_dim=4
)

h = atom_embedding(Z)
```

The observed output was:

```text
Atomic numbers:
tensor([6, 6, 8, 1, 1, 1, 1, 1, 1])

Embedding shape:
torch.Size([9, 4])

Carbon atom 0:
tensor([ 0.0349,  0.3211,  1.5736, -0.8455])

Carbon atom 1:
tensor([ 0.0349,  0.3211,  1.5736, -0.8455])

Oxygen atom 2:
tensor([-1.4181,  0.8963,  0.0499,  2.2667])

Hydrogen atom 3:
tensor([ 0.6784, -1.2345, -0.0431, -1.6047])

Same embedding for both carbon atoms? True
Embedding weights trainable? True
```

**Scientific interpretation**

The embedding tensor had shape \((9,4)\), meaning nine atoms, each represented by four feature channels.

Both carbon atoms received the same initial embedding because they have the same atomic number.

Hydrogen and oxygen received different embeddings because they correspond to different rows of the embedding table.

The embedding weights were trainable, as confirmed by `requires_grad=True`.

This established the mechanism by which chemical identity enters the neural-network representation.

### Exercise 2 — Combining atomic identity with geometry

The second task was to calculate a message from carbon atom 1 to carbon atom 0.

The C–C distance was

$$
d_{01}=1.5059\ \text{Å}.
$$

The sending atom's embedding was

$$
\mathbf h_1^{(0)} = [0.0349,\ 0.3211,\ 1.5736,\ -0.8455].
$$

The distance-dependent filter was

$$
W_\theta(d_{01}) = [-0.3505,\ -0.1570,\ 0.3958,\ -0.2568].
$$

The message was calculated using elementwise multiplication:

```python
message_ij = h[j] * filter_ij
```

The observed output was:

```text
Distance: 1.5059 Å
Sender atomic number: 6

Sender embedding:
tensor([ 0.0349,  0.3211,  1.5736, -0.8455])

Distance filter:
tensor([-0.3505, -0.1570,  0.3958, -0.2568])

Message:
tensor([-0.0122, -0.0504,  0.6229,  0.2172])

Message shape:
torch.Size([4])
```

Thus,

$$
\boxed{\mathbf m_{01} = [-0.0122,\ -0.0504,\ 0.6229,\ 0.2172]}
$$

The third channel illustrates the calculation:

$$
m_{01,3} \approx 1.5736\times0.3958 \approx0.6229.
$$

**Scientific interpretation**

The message depended on both the element of the sending atom and its distance from the receiving atom.

The resulting vector was a latent feature representation, not a physical force vector or an interaction energy.

Because the embedding and filter computations remained connected to the PyTorch computational graph, gradients could propagate to both parameter sets during future training.

### Exercise 3 — Isolating the effect of chemical identity

The third task was a controlled comparison between carbon and hydrogen messages at exactly the same distance.

The C–C distance of 1.5059 Å was reused for both elements.

The same distance filter was applied to the carbon and hydrogen embeddings.

The resulting output was:

```text
Shared distance: 1.5059 Å

Carbon embedding:
tensor([ 0.0349,  0.3211,  1.5736, -0.8455])

Hydrogen embedding:
tensor([ 0.6784, -1.2345, -0.0431, -1.6047])

Carbon message:
tensor([-0.0122, -0.0504,  0.6229,  0.2172])

Hydrogen message:
tensor([-0.2378,  0.1938, -0.0170,  0.4121])

Are the messages identical? False
```

Mathematically,

$$
\mathbf m_{\mathrm C}(d) = \mathbf h_{\mathrm C}\odot W_\theta(d)
$$

and

$$
\mathbf m_{\mathrm H}(d) = \mathbf h_{\mathrm H}\odot W_\theta(d).
$$

Because the distance was fixed, the filter was identical in both cases.

The different messages therefore arose from the different element embeddings.

For the third channel,

$$
m_{\mathrm C,3}\approx0.6229,
$$

whereas

$$
m_{\mathrm H,3}\approx-0.0170.
$$

**Scientific interpretation**

The architecture can produce element-dependent messages even when geometry is held constant.

This is important because molecular interactions depend on chemical identity as well as spatial arrangement.

However, the experiment did not establish physically correct carbon–hydrogen interaction differences. The parameters were randomly initialized.

### Exercise 4 — Aggregating messages using real atomic identities

The fourth task was to calculate the aggregated message arriving at carbon atom 0 from its actual spatial neighbors.

The cutoff was 2.0 Å.

The neighbors were:

$$
\mathcal N(0)=\{1,3,4,5\}.
$$

This neighborhood contained one carbon and three hydrogen atoms.

For each neighbor, the calculation was

$$
\mathbf m_{0j} = \mathbf h_j^{(0)} \odot W_\theta(\boldsymbol\phi(d_{0j})).
$$

The observed messages were:

| Neighbor | Element | Distance (Å) | Message |
|---|---|---:|---|
| 1 | Carbon | 1.506 | [−0.0122, −0.0504, 0.6229, 0.2172] |
| 3 | Hydrogen | 1.112 | [−0.2245, 0.2546, −0.0140, 0.3940] |
| 4 | Hydrogen | 1.113 | [−0.2245, 0.2545, −0.0140, 0.3941] |
| 5 | Hydrogen | 1.119 | [−0.2249, 0.2537, −0.0140, 0.3949] |

The aggregated message was:

```text
Aggregated message for atom 0:
tensor([-0.6861,  0.7124,  0.5809,  1.4001])

Shape:
torch.Size([4])
```

Therefore,

$$
\boxed{\mathbf m_0 =
[-0.6861,\ 0.7124,\ 0.5809,\ 1.4001]}
$$

**Scientific interpretation**

The carbon neighbor and hydrogen neighbors contributed different messages because their embeddings and distances differed.

The three hydrogen messages were very similar because they shared the same element embedding and had nearly identical distances from carbon atom 0.

The sum combined the contributions into a four-dimensional local-environment representation.

This was the first exercise in which the message aggregation simultaneously used actual atomic identities, actual ethanol geometry, and a learnable continuous-filter network.

### Exercise 5 — Updating an atomic representation

The fifth task was to combine carbon atom 0's initial embedding with its aggregated neighbor message.

The residual update was

$$
\mathbf h_0^{(1)} = \mathbf h_0^{(0)}+\mathbf m_0.
$$

The observed output was:

```text
Initial carbon representation:
tensor([ 0.0349,  0.3211,  1.5736, -0.8455])

Aggregated neighbor message:
tensor([-0.6861,  0.7124,  0.5809,  1.4001])

Updated carbon representation:
tensor([-0.6512,  1.0335,  2.1545,  0.5546])

Updated representation shape:
torch.Size([4])

Does the representation change?
True
```

The updated representation was

$$
\boxed{\mathbf h_0^{(1)} = [-0.6512,\ 1.0335,\ 2.1545,\ 0.5546]
}
$$

**Scientific interpretation**

The initial representation encoded the identity of carbon.

The aggregated message introduced information from the neighboring carbon and hydrogen atoms and their distances.

The residual update combined both sources of information.

This was the first explicit atomic feature update in the geometric message-passing implementation.

The updated vector was still a latent neural representation without a direct physical interpretation.

### Exercise 6 — Distinguishing two carbon environments

The final task was to determine whether the two carbon atoms in ethanol would acquire different representations after one message-passing update.

Initially,

$$
\mathbf h_0^{(0)}=\mathbf h_1^{(0)}
$$

because both atoms have atomic number 6.

However, their local environments differ.

For carbon 0:

$$
\mathcal N(0)=\{1,3,4,5\},
$$

containing one carbon and three hydrogens.

For carbon 1:

$$
\mathcal N(1)=\{0,2,6,7\},
$$

containing one carbon, one oxygen, and two hydrogens.

The observed neighbor list for carbon 1 was:

```text
Neighbor 0, Z=6, distance=1.506 Å
Neighbor 2, Z=8, distance=1.378 Å
Neighbor 6, Z=1, distance=1.104 Å
Neighbor 7, Z=1, distance=1.119 Å
```

The resulting comparison was:

```text
Initial carbon embeddings identical? True

Updated carbon 0:
tensor([-0.6512,  1.0335,  2.1545,  0.5546])

Updated carbon 1:
tensor([ 0.0649,  0.6246,  2.1875, -0.4224])

Updated carbon embeddings identical? False
```

Therefore,

$$
\boxed{\mathbf h_0^{(1)} \ne \mathbf h_1^{(1)}
}
$$

even though

$$
\mathbf h_0^{(0)} = \mathbf h_1^{(0)}.
$$

The updated representations were:

| Carbon atom | Initial representation | Updated representation |
|---|---|---|
| Carbon 0, CH₃ | [0.0349, 0.3211, 1.5736, −0.8455] | [−0.6512, 1.0335, 2.1545, 0.5546] |
| Carbon 1, CH₂ | [0.0349, 0.3211, 1.5736, −0.8455] | [0.0649, 0.6246, 2.1875, −0.4224] |

**Scientific interpretation**

The model distinguished two atoms of the same element because their neighbor compositions and geometries differed.

Carbon 0 received information from one carbon and three hydrogens.

Carbon 1 received information from one carbon, one oxygen, and two hydrogens.

Their aggregated messages were therefore different, producing different updated representations.

The distinction arose from the complete local environment, not solely from the presence of oxygen.

This experiment demonstrated the fundamental mechanism through which graph neural networks construct environment-dependent atomic representations.

---

## 5. Connection to SchNet and Geometric Deep Learning

The completed exercises implemented a simplified form of continuous-filter message passing.

The full operation can be summarized as

$$
\boxed{ \begin{aligned} \mathbf h_i^{(0)}
&=E_\theta(Z_i),\\[4pt]
\mathbf m_{ij}
&=\mathbf h_j^{(0)}
\odot
W_\theta(\boldsymbol\phi(d_{ij})),\\[4pt]
\mathbf m_i
&=\sum_{j\in\mathcal N(i)}\mathbf m_{ij},\\[4pt]
\mathbf h_i^{(1)}
&=\mathbf h_i^{(0)}+\mathbf m_i.
\end{aligned}
}
$$

This formulation has several desirable structural properties.

**Chemical identity awareness:** Different elements have different learned embeddings.

**Geometry awareness:** Messages depend on continuous interatomic distances.

**Permutation invariance of aggregation:** The sum does not depend on the order in which neighbors are processed.

**Translation and rotation invariance:** With scalar feature channels and distance-based neighborhoods, the message and update operations are invariant to rigid-body transformations.

**Differentiability with respect to model parameters:** The embedding and neural-filter parameters can receive gradients during training.

**Environment dependence:** Atoms of the same element can acquire different updated representations because their local neighborhoods differ.

The implementation is SchNet-inspired rather than a complete reproduction of the original SchNet architecture.

In particular, the residual update was simplified, and no end-to-end molecular property prediction model was trained.

---

## 6. Important Scientific Distinctions

### 6.1 Atomic identity versus atomic environment

An atomic embedding represents the element:

$$
\mathbf h_i^{(0)}=E_\theta(Z_i).
$$

An updated atomic representation includes local-environment information:

$$
\mathbf h_i^{(1)}
=
f_\theta
\left(
Z_i,\{Z_j,d_{ij}\}_{j\in\mathcal N(i)}
\right).
$$

These are not equivalent concepts.

The ethanol experiment demonstrated that two carbon atoms can have identical initial embeddings but different updated representations.

### 6.2 Trainable does not mean trained

Both the atomic embeddings and the distance-filter network contain trainable parameters.

However, the Day 38 experiments used randomly initialized parameters.

The fact that the model distinguishes the two carbon environments demonstrates architectural expressiveness, not learned chemical accuracy.

### 6.3 Feature vectors are not physical vectors

The four-dimensional embeddings and messages are latent feature vectors.

Their components are scalar channels, not Cartesian coordinates.

Therefore, these representations are intended to remain invariant under global molecular rotations.

They should not be confused with physical force vectors, which must rotate with the molecular geometry.

### 6.4 Local environments and message-passing depth

A single message-passing layer incorporates information from immediate spatial neighbors.

Stacking additional layers can allow information from more distant graph neighborhoods to influence an atom's representation.

This motivates the next stage of the project: updating all atoms and studying repeated message-passing layers.

---

## 7. Limitations of the Current Implementation

The main limitations were:

1. **Randomly initialized parameters.** Neither the atomic embedding table nor the nonlinear distance filter was trained on molecular property data during Day 38.

2. **Single molecular conformer.** All experiments used one ethanol geometry. No generalization across molecules or conformations was tested.

3. **Small feature dimension.** The atomic embedding dimension was four, chosen for educational clarity rather than predictive performance.

4. **Simplified interaction block.** The update consisted of directly adding the aggregated message to the initial representation. A full SchNet implementation contains additional learned transformations.

5. **Hard distance cutoff.** The 2.0 Å cutoff produces discrete neighborhood membership and may introduce discontinuities when distances cross the cutoff boundary.

6. **No molecular prediction target.** The model did not predict energies, forces, solvation properties, or experimental molecular observables.

7. **No physical validation.** The latent feature values cannot yet be interpreted as chemically accurate interaction strengths.

8. **Limited symmetry testing on Day 38.** The architecture uses invariant geometric inputs, but an explicit numerical rotation and translation test of the complete embedding-based update was not performed during today's exercises.

9. **No multi-layer propagation.** Only one update was computed for each carbon atom. Information propagation across multiple interaction layers remains to be explored.

These limitations define the next steps rather than invalidating the conceptual results.

---

## 8. Progress Toward Project 3

Day 38 advanced the geometric molecular learning project from distance-dependent message generation to environment-dependent atomic representations.

The progression across the four most recent days was:

| Day | Main achievement |
|---|---|
| Day 35 | Molecular coordinates, distances, rigid-body symmetry, and a simple physical force model |
| Day 36 | Spatial graphs, Gaussian radial basis functions, and fixed distance-dependent message passing |
| Day 37 | Learnable continuous filters, autograd, SGD, nonlinear vector-valued messages, and geometric invariance |
| Day 38 | Trainable atomic embeddings, identity-aware messages, residual feature updates, and environment-dependent carbon representations |

The work provides practical foundations for understanding SchNet-style molecular neural networks.

It also builds on the molecular graph machine-learning experience developed in Project 1, while extending that experience to explicit three-dimensional geometry.

Project 2 remains the separate molecular kinetics project involving tICA, Markov State Models, and VAMPnet.

---

## 9. Skills and Concepts Consolidated

**PyTorch**

- `nn.Embedding` for trainable categorical representations.
- Integer index tensors with `dtype=torch.long`.
- Embedding tensor shapes and lookup behavior.
- Reuse of nonlinear neural-network modules.
- Elementwise tensor operations.
- Summation-based message aggregation.
- Residual feature updates.
- Parameter differentiability and computational graphs.

**Molecular representation**

- Atomic numbers as element identifiers.
- Learned atomic feature vectors.
- Chemical identity versus chemical environment.
- Distance-dependent interactions.
- Local spatial neighborhoods.
- Environment-dependent atomic representations.

**Geometric deep learning**

- Continuous-filter message passing.
- Gaussian radial basis expansion.
- Vector-valued latent features.
- Permutation-invariant neighbor aggregation.
- Rotational and translational invariance by construction.
- One-layer neighborhood information propagation.
- The role of interaction depth.

**Scientific reasoning**

- Controlled comparisons that isolate one variable.
- Distinguishing architectural capability from trained predictive performance.
- Interpreting numerical tensor outputs without assigning unsupported physical meaning.
- Connecting mathematical operations to molecular chemistry.

---

## 10. Plan for Day 39

**Proposed topic:** Full-Molecule Message Passing and Multiple Interaction Layers.

The next learning objectives are to:

1. Generalize the message-passing update from the two carbon atoms to all nine atoms in ethanol.
2. Construct a complete updated atomic feature matrix.
3. Verify the shapes of the input and output representations.
4. Compare atomic features before and after message passing.
5. Explore how a second interaction layer allows information to propagate through a larger molecular neighborhood.
6. Test whether the complete molecular feature update preserves rotational and translational invariance.

The exercises will continue one small coding step at a time, with numerical results interpreted before proceeding.

---

## 11. Daily Scientific Reflection

Today I learned how to incorporate chemical identity into geometric molecular message passing.

The first important insight was that an atomic embedding is a trainable lookup representation of an element. At initialization, all atoms of the same element share the same embedding, regardless of their molecular environments.

The second insight was that chemical identity and geometry enter the message function through separate but complementary mechanisms. The atomic embedding describes the sending element, while the continuous filter depends on the interatomic distance.

The controlled comparison between carbon and hydrogen at a fixed distance demonstrated that chemical identity alone can change the message.

The aggregation exercise then showed how messages from real neighboring atoms combine into a representation of the local environment.

The residual update connected these messages to the atom's existing representation.

The most significant result was the comparison between ethanol's two carbon atoms. Their initial embeddings were identical, but after one message-passing update, their representations became different because their neighborhoods contained different atoms and distances.

This demonstrated the essential transition from element-level representations to environment-dependent atomic representations.

The experiment also reinforced an important scientific caution: the neural-network architecture can distinguish molecular environments even before training, but its latent features do not become chemically meaningful merely because they are different.

**Day 38 completed successfully.**

**Central takeaway:** A geometric graph neural network combines learned chemical identity with invariant geometric information to construct atomic representations that depend on the local molecular environment.
