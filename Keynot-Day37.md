# Day 37 - Scientific Learning Journal

**Date:** 28 October 2026  
**Phase:** Month 2 - Geometric Molecular Machine Learning  
**Project:** Project 3 - Geometry-Aware Molecular Learning  
**Topic:** Learnable Distance-Dependent Filters and SchNet-Style Message Passing  
**Status:** Completed

---

## 1. Objective of the Day

The objective was to extend the fixed distance-dependent message-passing operation developed on Day 36 into a learnable neural-network formulation using PyTorch.

The central scientific question was:

**How can a neural network learn distance-dependent atomic interactions while preserving the rotational and translational invariance required for scalar molecular representations?**

The specific goals were to:

- Replace manually selected Gaussian radial basis coefficients with trainable parameters.
- Understand PyTorch automatic differentiation and parameter gradients.
- Demonstrate gradient-based optimization of a distance filter.
- Extend scalar filters to nonlinear, vector-valued filters.
- Aggregate geometric messages from multiple neighboring atoms.
- Verify that learnable geometric message passing preserves rotational and translational invariance.

This work builds on the spatial neighborhood graphs, Gaussian radial basis functions, and geometric invariance tests completed on Day 36.

## 2. Theoretical Concepts Studied

### 2.1 From fixed filters to learnable filters

On Day 36, a distance-dependent filter was constructed using fixed coefficients:

$$
w(d)=\sum_{k=1}^{K}a_k\phi_k(d),
$$

where the Gaussian radial basis functions are

$$
\phi_k(d)=\exp[-\gamma(d-\mu_k)^2].
$$

The coefficients were manually selected rather than learned from data.

On Day 37, the filter was replaced by a trainable linear transformation:

$$
\boxed{f_\theta(d)=\mathbf w^\mathsf T\boldsymbol\phi(d)+b
}
$$

where \(\mathbf w\) and \(b\) are learnable parameters.

This allows the distance dependence of the filter to be adjusted through optimization.

### 2.2 Automatic differentiation

PyTorch constructs a computational graph for operations involving trainable parameters.

For the linear filter,

$$
f_\theta(d)=\sum_k w_k\phi_k(d)+b,
$$

the analytical derivatives are

$$
\frac{\partial f_\theta}{\partial w_k}=\phi_k(d)
$$

and

$$
\frac{\partial f_\theta}{\partial b}=1.
$$

These derivatives describe how sensitively the filter output responds to changes in each parameter.

Because Gaussian basis functions are localized in distance, the filter output is most sensitive to coefficients associated with Gaussian centers near the input distance.

### 2.3 Gradient-based optimization

A demonstration supervised-learning problem was introduced using the artificial target

$$
y=1.0.
$$

The squared-error loss was

$$
\mathcal L=(f_\theta(d)-y)^2.
$$

The gradient of this loss with respect to each filter coefficient is

$$
\frac{\partial\mathcal L}{\partial w_k} = 2(f_\theta(d)-y)\phi_k(d).
$$

Stochastic gradient descent updates the parameters according to

$$
w_k^{(t+1)} = w_k^{(t)} - \eta\frac{\partial\mathcal L}{\partial w_k},
$$

where \(\eta\) is the learning rate.

This demonstrates how distance-dependent filters can be optimized from data instead of being manually specified.

### 2.4 Nonlinear vector-valued filters

The scalar filter was extended to a neural network producing four output channels:

$$
\mathbf W_\theta(d)\in\mathbb R^4.
$$

The architecture was

$$
5\text{ RBF features} \rightarrow 8\text{ hidden units} \rightarrow 4\text{ filter channels}.
$$

A SiLU activation function introduced nonlinearity between the two linear layers.

The geometric message from atom \(j\) to atom \(i\) was defined as

$$
\boxed{ \mathbf m_{ij} = \mathbf h_j\odot \mathbf W_\theta(\boldsymbol\phi(d_{ij}))
}
$$

where \(\mathbf h_j\) is a four-dimensional atomic feature vector and \(\odot\) denotes elementwise multiplication.

This is a simplified form of SchNet-style continuous-filter message passing.

### 2.5 Spatial message aggregation

Messages from neighboring atoms were combined using summation:

$$
\boxed{\mathbf m_i = \sum_{j\in\mathcal N(i)} \mathbf m_{ij}
}
$$

Summation produces a fixed-dimensional representation regardless of the number of neighbors.

It is also mathematically invariant to the ordering of neighbors, which is important because atom numbering and neighbor-list ordering should not affect the underlying molecular representation.

### 2.6 Rotational and translational invariance

The distance-based message-passing operation was constructed using invariant quantities:

- Atomic feature vectors treated as scalar feature channels.
- Interatomic distances.
- Gaussian radial basis features derived from distances.
- Distance-based spatial neighborhood selection.

Therefore, the aggregated message is expected to satisfy

$$
\boxed{\mathbf m_i(Q\mathbf R+\mathbf t) = \mathbf m_i(\mathbf R)}
$$

for a global rotation \(Q\) and translation \(\mathbf t\).

Although the aggregated message has four components, these components are scalar feature channels, not Cartesian components of a physical vector.

Thus, they are invariant rather than rotationally equivariant.

---

## 3. Computational Work and Numerical Results

### Exercise 1 — Constructing a learnable scalar distance filter

Reused the Gaussian radial basis representation of the C–C bond in ethanol from Day 36.

The input was

```text
tensor([0.0107, 0.3190, 0.9999, 0.3304, 0.0115])
```

A PyTorch linear layer was created:

```python
filter_net = nn.Linear(5, 1)
```

With random seed 42, the initialized parameters were:

```text
Learnable weights:
tensor([[ 0.3419,  0.3712, -0.1048,  0.4108, -0.0980]])

Learnable bias:
tensor([0.0902])
```

The calculated filter output was

```text
tensor([0.2421], grad_fn=<ViewBackward0>)
```

The output demonstrates that the Gaussian radial basis features can be mapped to a scalar distance-dependent filter using trainable parameters.

The `grad_fn` attribute indicates that PyTorch is tracking the computation for automatic differentiation.

At this stage, the output was not physically meaningful because the parameters had not been trained.

### Exercise 2 — Verifying automatic differentiation

Calculated the derivatives of the scalar filter output using PyTorch's `backward()` operation.

Results:

```text
Weight gradients:
tensor([[0.0107, 0.3190, 0.9999, 0.3304, 0.0115]])

Bias gradient:
tensor([1.])

Original RBF features:
tensor([0.0107, 0.3190, 0.9999, 0.3304, 0.0115])

Do weight gradients equal RBF features?
True
```

The numerical gradients agreed with the analytical result:

$$
\nabla_{\mathbf w}f_\theta = \boldsymbol\phi(d).
$$

The Gaussian centered at 1.5 Å had the largest activation and therefore the largest corresponding weight gradient.

This verified that localized radial basis functions determine the parameter sensitivity of the filter at a given distance.

### Exercise 3 — Training the scalar filter using SGD

The filter was trained to reproduce an artificial target value of 1.0 at the C–C bond distance.

The optimizer was stochastic gradient descent with learning rate

$$
\eta=0.1.
$$

Training was performed for 20 optimization steps.

| Training stage | Loss | Prediction |
|---|---:|---:|
| Before training | — | 0.242148 |
| Epoch 1 | 0.574339 | 0.242148 |
| Epoch 5 | 0.005382 | 0.926635 |
| Epoch 10 | 0.000016 | 0.996038 |
| Epoch 20 | 0.000000 (rounded) | 0.999988 |
| After final update | — | 0.999994 |

The updated parameters were

```text
Updated weights:
tensor([[ 0.3456,  0.4805,  0.2380,  0.5241, -0.0940]])

Updated bias:
tensor([0.4330])
```

The final prediction was

$$
\boxed{f_\theta(d_{\mathrm{CC}})=0.99999356
}
$$

which was very close to the target value of 1.0.

The coefficient associated with the Gaussian centered at 1.5 Å changed substantially, from approximately −0.1048 to 0.2380.

This was consistent with the high activation of that basis function at the C–C distance.

**Scientific interpretation:** The experiment demonstrated that PyTorch autograd and gradient descent can optimize a Gaussian distance filter.

However, the training used only one artificial distance–target pair. It did not establish that the filter had learned a physically meaningful interaction function or that it would generalize to other distances.

### Exercise 4 — Constructing a nonlinear vector-valued filter

Implemented a neural network using the architecture

```python
vector_filter_net = nn.Sequential(
    nn.Linear(5, 8),
    nn.SiLU(),
    nn.Linear(8, 4)
)
```

The Gaussian radial basis input had five components.

The network produced a four-dimensional filter:

```text
Vector filter:
tensor([-0.3505, -0.1570,  0.3958, -0.2568])
```

A manually specified atomic feature vector was used:

```text
Atomic features:
tensor([ 1.0000,  0.5000, -0.5000,  2.0000])
```

The geometric message was calculated through elementwise multiplication:

```text
Geometric message:
tensor([-0.3505, -0.0785, -0.1979, -0.5137])
```

The tensor shapes were

```text
RBF input shape: torch.Size([5])
Vector filter shape: torch.Size([4])
Atomic feature shape: torch.Size([4])
```

The result demonstrated that one interatomic distance can control several feature channels independently.

The filter was randomly initialized and was not trained during this exercise.

### Exercise 5 — Aggregating messages from ethanol's spatial neighbors

Reused the ethanol spatial neighborhood graph from Day 36 with cutoff radius 2.0 Å.

Atom 0 had four spatial neighbors: atoms 1, 3, 4, and 5.

The same four-dimensional atomic feature vector was assigned to every neighbor to isolate the influence of distance.

The resulting messages were:

| Neighbor | Distance (Å) | Message vector |
|---|---:|---|
| Atom 1 | 1.506 | [−0.3505, −0.0785, −0.1979, −0.5137] |
| Atom 3 | 1.112 | [−0.3309, −0.1031, −0.1622, −0.4910] |
| Atom 4 | 1.113 | [−0.3310, −0.1031, −0.1624, −0.4912] |
| Atom 5 | 1.119 | [−0.3315, −0.1027, −0.1630, −0.4921] |

The aggregated message was

$$
\boxed{\mathbf m_0=[-1.3438,\,-0.3874,\,-0.6856,\,-1.9881]
}
$$

with shape

```text
torch.Size([4])
```

The three hydrogen neighbors produced similar messages because their distances from atom 0 were nearly identical.

The carbon neighbor produced a somewhat different message because its distance was larger.

Since all neighbors shared the same manually specified feature vector, the message differences arose only from geometry.

This demonstrated how a nonlinear distance filter can influence information aggregation in a spatial molecular graph.

### Exercise 6 — Verifying geometric invariance

Recomputed the vector-valued aggregated message using the original, translated, and rotated ethanol coordinates.

Results:

```text
Original:
tensor([-1.3438, -0.3874, -0.6856, -1.9881])

Translated:
tensor([-1.3438, -0.3874, -0.6856, -1.9881])

Rotated:
tensor([-1.3438, -0.3874, -0.6856, -1.9881])

Translation invariant? True
Rotation invariant? True
```

The aggregated message remained unchanged to the displayed precision.

The experiment verified the expected invariance for the tested transformations.

This result is consistent with the mathematical construction: the neural network receives Gaussian features derived from invariant interatomic distances.

The learnable filter does not need to infer rotational or translational invariance from training examples because the geometric input representation already preserves these symmetries.

The aggregation implementation also retained the PyTorch computational graph, allowing gradients to propagate through the message-passing operation in future training experiments.

---

## 4. Connection to SchNet

SchNet uses continuous-filter convolutions to incorporate interatomic distances into learned atomic representations.

A simplified formulation is

$$
\mathbf m_i= \sum_{j\in\mathcal N(i)} \mathbf h_j\odot W_\theta(\boldsymbol\phi(d_{ij})).
$$

The exercises completed today implemented the central components of this expression:

1. Gaussian radial basis distance features.
2. Trainable distance-dependent filters.
3. Nonlinear neural-network filter generation.
4. Vector-valued atomic feature channels.
5. Elementwise message construction.
6. Summation over spatial neighbors.
7. Numerical verification of geometric invariance.

The implementation is not yet a complete SchNet model.

In particular, it does not include learned atom embeddings, full interaction blocks, repeated atomic feature updates, molecular readout, or end-to-end training against molecular property data.

Nevertheless, the work establishes the mathematical and computational foundations of continuous-filter message passing.

## 5. Main Scientific Insights

**Learnability:** Replacing fixed radial basis coefficients with PyTorch parameters allows the distance dependence of a filter to be optimized from data.

**Localized sensitivity:** The gradient of a linear Gaussian filter with respect to a basis coefficient equals the activation of the corresponding Gaussian. Distances near a Gaussian center therefore produce stronger sensitivity to that coefficient.

**Optimization:** Gradient descent successfully adjusted the filter parameters to reproduce a demonstration target. This established the computational mechanics of learning, but not physical predictive validity.

**Nonlinear filtering:** A multilayer neural network can transform Gaussian distance features into a nonlinear, vector-valued filter.

**Feature-dependent interactions:** Different atomic feature channels can be modified independently by the same interatomic distance.

**Spatial aggregation:** Summing neighbor messages produces a fixed-dimensional atomic representation and avoids dependence on neighbor ordering.

**Geometric invariance:** Learnable message-passing operations can preserve rotational and translational invariance when their inputs and neighborhood construction are based on invariant quantities.

**Invariance versus equivariance:** A four-dimensional vector of scalar feature channels is invariant under spatial rotations. It is not automatically an equivariant physical vector.

## 6. Limitations and Scope

Several limitations remain important.

First, the scalar distance filter was trained using only one artificial target. It cannot be interpreted as a learned molecular potential.

Second, the nonlinear four-channel filter was randomly initialized and was not trained.

Third, the atomic feature vector was manually specified and identical for every neighboring atom. Chemical identity was therefore not represented through distinct learned embeddings.

Fourth, the experiment used a single ethanol conformer and a fixed spatial cutoff of 2.0 Å.

Fifth, the implementation did not predict molecular energies, forces, or experimental properties.

Sixth, although the aggregation preserved the computational graph in the final invariance test, a complete end-to-end molecular model was not trained.

Finally, a hard distance cutoff can introduce discontinuities when atoms cross the cutoff boundary. A production molecular potential may require smooth cutoff functions and additional physical constraints.

## 7. Progress Toward the Industry Portfolio

Day 37 advanced Project 3 from manually constructed geometric descriptors toward trainable geometric neural-network components.

The work demonstrated practical experience with:

- PyTorch neural-network modules.
- Trainable parameters and automatic differentiation.
- Analytical gradient verification.
- SGD optimization.
- Gaussian radial basis distance features.
- Nonlinear continuous-filter networks.
- Vector-valued message construction.
- Spatial neighborhood aggregation.
- Rotation and translation invariance testing.

These concepts are directly relevant to molecular property prediction, learned interatomic potentials, and geometric deep learning.

They extend the molecular graph learning experience gained in Project 1, which used a GINE architecture for human AChE bioactivity prediction.

The longer-term portfolio goal remains to demonstrate scientifically sound molecular modeling and machine-learning workflows for industry applications planned for late December 2026.

## 8. Next Day — Day 38

**Topic:** Learned Atomic Embeddings and Atomic Feature Updates in a Minimal SchNet-Style Model.

Planned learning objectives:

- Replace manually specified atomic feature vectors with learned atom embeddings.
- Understand how atomic numbers are mapped into latent feature vectors.
- Combine chemical identity with distance-dependent geometric filters.
- Construct a simplified atomic feature-update operation.
- Examine the distinction between initial atomic representations and updated representations.
- Continue testing relevant symmetry properties.

The work will continue through small coding exercises, with numerical output interpreted before moving to the next step.

## 9. Daily Scientific Reflection

Today I progressed from fixed distance-dependent message passing to learnable geometric neural-network components.

The most important mathematical insight was that Gaussian radial basis functions not only provide
