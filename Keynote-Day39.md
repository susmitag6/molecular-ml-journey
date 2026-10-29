# Day 39 — Scientific Learning Journal

**Date:** 29 October 2026  
**Project 3:** Geometry-Aware Molecular Machine Learning  
**Topic:** Full-Molecule Message Passing, Receptive Fields, and Geometric Invariance  
**Status:** Completed

---

## 1. Learning Objective

The objective of Day 39 was to extend the geometric message-passing model developed during Days 35–38 from individual atoms to an entire molecule.

On Day 38, I learned how atomic embeddings and distance-dependent neural filters combine to produce environment-dependent atomic representations.

Today, I focused on three deeper questions:

1. How can the same message-passing operation be applied to every atom in a molecule?
2. How does stacking message-passing layers allow information to travel beyond immediate neighbors?
3. Does the resulting multilayer molecular representation preserve translation and rotation invariance?

By the end of the session, I had implemented a complete two-layer, SchNet-inspired message-passing model for ethanol and verified its principal structural properties.

---

## 2. Full-Molecule Message Aggregation

### Mathematical formulation

For each atom \(i\), messages are received from all atoms \(j\) in its spatial neighborhood:

$$
\mathbf m_i = \sum_{j\in\mathcal N(i)} \mathbf h_j \odot W_\theta\left(\boldsymbol\phi(d_{ij})\right).
$$

Here:

- \(\mathbf h_j\) is the feature vector of the sending atom.
- \(d_{ij}\) is the interatomic distance.
- \(\boldsymbol\phi(d_{ij})\) is its Gaussian radial basis expansion.
- \(W_\theta\) is the learned continuous-filter network.
- \(\odot\) denotes elementwise multiplication.

Previously, this operation was calculated for selected carbon atoms.

On Day 39, I generalized it into a reusable Python function that iterates over all atoms and returns the complete message matrix:

$$
M= \begin{bmatrix}
\mathbf m_0\\
\mathbf m_1\\
\vdots\\
\mathbf m_{N-1}
\end{bmatrix}.
$$

### Coding concepts learned

**Finding spatial neighbors**

```python
neighbors = np.where(
    (distances[i] < cutoff) &
    (distances[i] > 0)
)[0]
```

This expression identifies atoms whose distances from atom \(i\) are positive and below the chosen cutoff.

An important NumPy detail is that `np.where(condition)` returns a tuple of index arrays. For a one-dimensional condition, `[0]` extracts the array containing the matching indices.

**Constructing neighbor messages**

```python
m_ij = H[j] * w_ij
```

This combines the sending atom's features with the filter determined by its distance from the receiving atom.

**Aggregating messages**

```python
aggregated_i = torch.stack(messages_i).sum(dim=0)
```

`torch.stack()` collects the individual neighbor messages into a tensor.

`.sum(dim=0)` sums across neighbors while preserving the feature dimension.

**Handling atoms without neighbors**

```python
aggregated_i = torch.zeros_like(H[i])
```

This ensures the function remains valid even when an atom has no neighbors within the cutoff.

### Scientific understanding

The same local message-passing rule can be applied to molecules with different numbers of atoms.

The number of messages varies with neighborhood size, but the aggregated feature dimension remains fixed.

Summation also ensures that the result does not depend on the order in which neighboring atoms are processed.

I additionally verified that the generalized implementation reproduced the earlier single-atom calculation. This reinforced the importance of regression testing when refactoring scientific code.

---

## 3. Simultaneous Atomic Feature Updates

After constructing the complete message matrix, I updated every atomic representation using a residual connection:

$$
\boxed{H^{(1)}=H^{(0)}+M(H^{(0)},D)}
$$

where \(D\) is the pairwise distance matrix.

The implementation was simply:

```python
H_updated = h + M
```

### Key learning

The entire molecule can be updated through one matrix operation once the messages have been aggregated.

Each updated atom representation combines:

- The atom's own initial chemical identity.
- The identities of neighboring atoms.
- The distances between the atom and its neighbors.

A particularly important detail is that the updates are **synchronous**.

All first-layer messages are calculated using the same initial feature matrix \(H^{(0)}\). We do not update one atom and immediately use its new features to calculate another atom's first-layer messages.

This preserves a clear mathematical definition of one message-passing layer.

---

## 4. Multiple Layers and the Receptive Field

### What is a receptive field?

In a graph neural network, the receptive field of an atom is the region of the graph from which information can influence its representation.

After one message-passing layer, an atom can receive information from its immediate neighbors.

After two layers, information can also arrive through those neighbors from atoms two graph edges away.

For the second layer, I implemented:

$$
\boxed{H^{(2)}=H^{(1)}+M(H^{(1)},D)}
$$

The crucial difference is that the second layer uses \(H^{(1)}\), not the original embeddings \(H^{(0)}\).

### Physical and molecular interpretation

Consider the carbon–carbon–oxygen chain in ethanol:

$$
\mathrm C_0-\mathrm C_1-\mathrm O_2.
$$

In the spatial graph used for the experiment, oxygen is not a direct neighbor of carbon 0, but it is a neighbor of carbon 1.

During the first layer, carbon 1 receives information from oxygen.

During the second layer, carbon 0 receives information from carbon 1's updated representation.

Consequently, oxygen-related information can reach carbon 0 through the pathway

$$
\boxed{
\mathrm O_2
\longrightarrow
\mathrm C_1
\longrightarrow
\mathrm C_0
}
$$

### Key scientific insight

Increasing message-passing depth expands the range of graph information available to each atom.

However, a larger receptive field does not automatically imply a better model. Deeper networks may introduce optimization difficulties, oversmoothing, or unnecessary complexity.

The useful depth depends on the molecular property, graph construction, architecture, and training data.

---

## 5. A Controlled Test of Two-Hop Information Propagation

One of the most important exercises was to test the receptive-field concept rather than relying only on a theoretical explanation.

### Experimental design

I created a copy of the initial atomic feature matrix and modified only the oxygen embedding.

All atomic coordinates, distances, graph connectivity, and neural-network parameters remained unchanged.

I then applied the same message-passing model to both the original and modified inputs.

The question was:

**At which layer does carbon 0 become sensitive to a change in oxygen's initial representation?**

### Interpretation

Carbon 0 was unaffected after the first layer but became sensitive to the oxygen perturbation after the second layer.

This supported the predicted two-hop information pathway.

The experiment was a useful example of controlled computational reasoning:

1. Change one input variable.
2. Keep other inputs and parameters fixed.
3. Observe where the effect first appears.
4. Relate the observation to the architecture's information-flow structure.

### Why did the perturbation remain in one feature channel?

Our simplified message function uses elementwise multiplication:

$$
\mathbf m_{ij} =
\mathbf h_j\odot W_\theta(d_{ij}).
$$

The filter scales each feature channel separately.

Because this message function does not mix channels, a perturbation introduced into one feature channel propagates through that same channel.

A more complete interaction block can include learned linear transformations that mix information between feature channels.

This exercise therefore revealed both a strength and a limitation of the simplified architecture.

---

## 6. Translation and Rotation Invariance

The final exercise examined whether the complete two-layer model respects rigid-body transformations.

### Mathematical principle

Under a rigid-body transformation,

$$
\mathbf r_i'=R\mathbf r_i+\mathbf t,
$$

where \(R\) is a rotation matrix and \(\mathbf t\) is a translation vector.

The distance between two atoms is unchanged:

$$
\begin{aligned}
d_{ij}'
&=\|\mathbf r_i'-\mathbf r_j'\|\\
&=\|R(\mathbf r_i-\mathbf r_j)\|\\
&=\|\mathbf r_i-\mathbf r_j\|\\
&=d_{ij}.
\end{aligned}
$$

Since the model uses atomic identities and interatomic distances, its scalar atomic features should remain invariant:

$$
\boxed{
H^{(2)}(R\mathbf r+\mathbf t) = H^{(2)}(\mathbf r) }
$$

### Computational test

I wrote a reusable function that:

1. Recalculates the distance matrix from molecular coordinates.
2. Performs the first message-passing layer.
3. Performs the second message-passing layer.
4. Returns the final atomic feature matrix.

I applied this function to the original, translated, and rotated ethanol coordinates.

Both numerical invariance tests passed.

### Scientific interpretation

The resulting atomic representations do not depend on the molecule's absolute position or orientation.

This is an essential property for molecular scalar representations because rotating or translating a molecule does not change its chemical identity or internal structure.

The numerical tests validated the implementation for the transformations used. More generally, the invariance follows from the model's use of rigid-body-invariant distance features.

### Invariance versus equivariance

An important distinction is:

**Invariant quantity**

$$
f(R\mathbf r+\mathbf t)=f(\mathbf r).
$$

An invariant quantity remains unchanged under the transformation.

**Equivariant vector quantity**

$$
\mathbf F(R\mathbf r+\mathbf t)=R\mathbf F(\mathbf r).
$$

An equivariant vector rotates consistently with the molecule.

Our four-dimensional atomic representations consist of scalar feature channels, so invariance is appropriate.

Physical forces are Cartesian vectors and require rotational equivariance instead.

---

## 7. What I Learned About Geometric GNN Architecture

The complete Day 39 model can be summarized as:

$$
\begin{aligned}
H^{(0)} &= E_\theta(Z),\\
M^{(0)} &= M_\theta(H^{(0)},D),\\
H^{(1)} &= H^{(0)}+M^{(0)},\\
M^{(1)} &= M_\theta(H^{(1)},D),\\
H^{(2)} &= H^{(1)}+M^{(1)}.
\end{aligned}
$$

This formulation connects several important ideas:

**Atomic embeddings:** Represent the identity of each chemical element.

**Continuous distance filters:** Make messages depend smoothly on interatomic separation.

**Neighborhood aggregation:** Combines information from local spatial environments.

**Residual updates:** Preserve the previous representation while incorporating new messages.

**Layer stacking:** Allows information to propagate through increasingly distant graph neighborhoods.

**Geometric invariance:** Ensures that scalar atomic representations do not depend on arbitrary molecular orientation or position.

Together, these components form the conceptual foundation of SchNet-style molecular message passing.

---

## 8. Limitations and Scientific Cautions

The implementation remains an educational model rather than a trained molecular predictor.

The most important limitations are:

- The atomic embeddings and continuous filters remain randomly initialized.
- The same filter network was reused in both layers.
- The update rule is simpler than a full SchNet interaction block.
- The feature channels do not mix during the message operation.
- A fixed spatial cutoff determines which atoms can exchange messages.
- Only one ethanol conformer has been examined.
- No molecular energy, force, or other property prediction was trained or validated.
- The observed feature changes do not have a direct physical interpretation.

These limitations are important because a model can possess the correct mathematical structure without yet learning chemically meaningful relationships.

---

## 9. Connection to Previous Learning

Day 39 brought together the concepts developed over the preceding days.

| Day | Main conceptual achievement |
|---|---|
| Day 35 | Molecular coordinates, pairwise distances, rigid-body symmetry |
| Day 36 | Spatial molecular graphs and Gaussian radial basis functions |
| Day 37 | Learnable continuous filters and differentiable message passing |
| Day 38 | Learned atomic embeddings and environment-dependent representations |
| **Day 39** | **Full-molecule updates, multilayer information propagation, and invariance validation** |

The key progression was from constructing individual messages to understanding the behavior of an entire geometric neural network.

This work contributes to **Project 3**, the geometric molecular learning portfolio project.

---

## 10. Main Takeaways

1. **Message passing is a local operation that can be generalized to the entire molecule.** Each atom receives messages from its spatial neighbors using the same learned function.

2. **Residual updates combine existing atomic information with environmental information.** The atomic feature matrix can be updated simultaneously.

3. **Multiple message-passing layers expand the receptive field.** An atom can become sensitive to information originating beyond its immediate neighborhood.

4. **Controlled perturbations are powerful scientific tests.** Changing only oxygen's initial embedding allowed me to verify a specific information-propagation pathway.

5. **The message function determines how information propagates between feature channels.** Elementwise filtering preserves channel separation in this simplified implementation.

6. **Geometric invariance is an architectural property, not merely a training objective.** Using interatomic distances makes scalar representations invariant to rigid-body rotations and translations.

7. **Architectural correctness is distinct from predictive accuracy.** The next challenge is to connect the model to a meaningful molecular prediction task.

---

## 11. Reflection and Next Steps

Day 39 was an important transition from isolated coding exercises to a functioning multilayer molecular representation model.

I now understand how a geometric GNN constructs atomic features iteratively, how the number of interaction layers controls information propagation, and how to verify these mechanisms through controlled numerical experiments.

The most valuable conceptual result was understanding that a second message-passing layer does more than update features again: it changes which parts of the molecular graph can influence each atom.

I also strengthened my scientific programming practice by using reusable functions, regression tests, controlled perturbations, and symmetry checks.

**Possible Day 40 direction:** Introduce a molecular readout operation that combines atomic representations into a whole-molecule representation, then connect it to a simple scalar prediction head. This would bridge geometric representation learning and molecular property prediction.

**Central takeaway of Day 39:**

*A geometric graph neural network builds molecular representations through repeated local information exchange. Its depth determines how far information can propagate, while its choice of geometric inputs determines which physical symmetries are preserved.*

**Day 39 completed successfully.**
