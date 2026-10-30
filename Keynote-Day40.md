# Day 40 — Scientific Learning Journal

**Date:** 30 October 2026  
**Project 3:** Geometry-Aware Molecular Machine Learning  
**Topic:** Molecular Readout, Prediction Heads, and Gradient-Based Training  
**Status:** Completed

---

## 1. Learning Objectives

The objective of Day 40 was to transform the atom-level representations developed in previous sessions into a complete molecular property prediction pipeline.

On Day 39, I implemented a two-layer geometric graph neural network (GNN) that produced environment-dependent atomic representations while preserving translation and rotation invariance.

However, a molecular property such as total energy is associated with the entire molecule rather than an individual atom.

The central question for Day 40 was:

**How can a geometric GNN convert atomic representations into a molecular prediction, and how can that prediction be optimized using reference data?**

The session covered molecular pooling, neural-network prediction heads, loss functions, automatic differentiation, gradient descent, and iterative training.

By the end of the day, I had connected all the components into a working molecular prediction pipeline.

---

## 2. From Atomic Representations to Molecular Representations

### Scientific motivation

After geometric message passing, each atom has a feature vector that encodes information about its identity and local molecular environment.

The full atomic representation can be written as

$$
H^{(L)} = \begin{bmatrix}
\mathbf h_1^{(L)}\\
\mathbf h_2^{(L)}\\
\vdots\\
\mathbf h_N^{(L)}
\end{bmatrix}.
$$

Here, \(N\) is the number of atoms and \(L\) is the number of message-passing layers.

To predict a whole-molecule property, these atomic features must be combined into a molecular representation.

This operation is called **molecular readout** or **global pooling**.

### Sum pooling

I implemented the readout

$$
\boxed{
\mathbf h_{\mathrm{mol}}
\sum_{i=1}^{N}\mathbf h_i^{(L)}
}
$$

using PyTorch:

```python
h_mol = H_second.sum(dim=0)
```

The operation sums over the atom dimension while preserving the feature dimension.

### Why sum pooling is useful

Sum pooling has two important mathematical properties.

**Permutation invariance**

The molecular representation should not depend on how atoms are numbered.

For any permutation \(\pi\),

\[
\sum_i\mathbf h_i
=
\sum_i\mathbf h_{\pi(i)}.
\]

I tested this by randomly rearranging the rows of the atomic feature matrix and recalculating the sum.

The molecular representation remained unchanged.

This confirms permutation invariance of the readout operation.

For the complete GNN to be invariant to atom reindexing, atomic features and graph geometry must also be permuted consistently throughout message passing.

**Compatibility with extensive molecular properties**

Sum aggregation is a natural choice when modeling extensive properties, such as total molecular energy, which can often be expressed through contributions associated with different atoms or local environments.

However, summing latent atomic features is not equivalent to summing physical atomic energies.

A prediction architecture must learn how the representation relates to the target property.

### Sum versus mean pooling

Mean pooling provides another option:

$$
\mathbf h_{\mathrm{mol}}
=
\frac{1}{N}\sum_i\mathbf h_i.
$$

Sum pooling retains information about the overall magnitude and number of atomic contributions, while mean pooling normalizes by molecular size.

The appropriate readout depends on whether the target property is extensive, intensive, or governed by a more complicated molecular relationship.

**Key learning:** A molecular GNN requires a permutation-invariant readout to transform a variable-size collection of atomic representations into a fixed-size molecular representation.

---

## 3. Building a Molecular Prediction Head

Once the molecular representation has been constructed, a neural network can map it to a target property.

Mathematically,

\[
\boxed{
\hat y=f_\theta(\mathbf h_{\mathrm{mol}})
}
\]

where \(f_\theta\) is a trainable prediction function.

I implemented a small multilayer perceptron using `nn.Sequential`:

```python
prediction_head = nn.Sequential(
    nn.Linear(4, 8),
    nn.SiLU(),
    nn.Linear(8, 1)
)
```

### Role of each component

**First linear layer**

Transforms the molecular representation into a hidden feature space:

$$
\mathbf z=W_1\mathbf h_{\mathrm{mol}}+\mathbf b_1.
$$

**Nonlinear activation**

Applies a nonlinear transformation to the hidden features.

**Final linear layer**

Maps the hidden representation to a scalar prediction:

$$
\hat y=W_2\mathbf a+\mathbf b_2.
$$

### Why is a nonlinear activation necessary?

Without a nonlinear activation, successive linear transformations collapse into a single affine transformation.

For example,

$$
W_2(W_1\mathbf x+\mathbf b_1)+\mathbf b_2
=
W_{\mathrm{eff}}\mathbf x+\mathbf b_{\mathrm{eff}}.
$$

Consequently, stacking linear layers alone does not introduce nonlinear expressive power.

The activation function allows the network to learn nonlinear relationships between molecular features and the target property.

---

## 4. Understanding the SiLU Activation Function

An important concept introduced today was **SiLU**, the Sigmoid Linear Unit.

Its mathematical definition is

$$
\boxed{
\operatorname{SiLU}(x)
=
x\sigma(x)
=
\frac{x}{1+e^{-x}}
}
$$

where \(\sigma(x)\) is the sigmoid function.

### Important properties

SiLU is smooth and differentiable.

Unlike ReLU, which sets all negative inputs to zero, SiLU permits negative outputs.

For positive inputs, it behaves approximately like the identity function when the input becomes sufficiently large.

For strongly negative inputs, its output approaches zero from below.

SiLU operates elementwise, preserving the shape of the input tensor.

It also has no trainable parameters.

### Relevance to molecular machine learning

Smooth nonlinearities are useful in molecular modeling, particularly when learning functions of continuous geometric variables.

For example, models predicting molecular energies and deriving forces through coordinate differentiation benefit from smooth energy functions.

However, smoothness of the activation alone does not guarantee physically accurate or conservative predictions; the entire architecture and training objective must be considered.

**Key learning:** SiLU introduces smooth nonlinearity without changing feature dimensions or adding trainable parameters.

---

## 5. Understanding Neural-Network Parameters

I examined how trainable parameters are distributed across the prediction head.

For a linear transformation

$$
\mathbf y=W\mathbf x+\mathbf b,
$$

the number of parameters is

$$
\boxed{
N_{\mathrm{parameters}}
=
N_{\mathrm{in}}N_{\mathrm{out}}
+
N_{\mathrm{out}}
}
$$

The first term counts the elements of the weight matrix, while the second counts the bias parameters.

In PyTorch, I used:

```python
sum(p.numel() for p in prediction_head.parameters())
```

Here:

- `prediction_head.parameters()` iterates over trainable parameter tensors.
- `p.numel()` returns the number of scalar elements in each tensor.
- `sum()` adds these counts.

This exercise clarified the distinction between a neural network's feature dimensions and its trainable parameter count.

It also reinforced that activation functions such as SiLU do not necessarily introduce additional parameters.

---

## 6. Defining a Molecular Prediction Loss

A prediction head produces an output, but training requires a reference target and an objective function.

For a regression problem, I used the mean squared error (MSE):

\[
\boxed{
\mathcal L
=
\frac{1}{B}
\sum_{k=1}^{B}
(\hat y_k-y_k)^2
}
\]

where \(B\) is the number of training examples.

For a single example, this reduces to the squared difference between prediction and target.

In PyTorch:

```python
loss_fn = nn.MSELoss()
loss = loss_fn(y_pred, target)
```

### Scientific interpretation

The loss measures disagreement between model predictions and reference labels.

Minimizing the loss encourages the model to reproduce the targets.

However, the scientific meaning of the optimization depends on the quality and physical meaning of the reference data.

For Day 40, I used a synthetic target solely to verify the computational training procedure.

The exercise therefore tested optimization mechanics rather than molecular predictive accuracy.

**Key learning:** A loss function connects the model's output to a training objective, but low training loss alone does not establish physical validity.

---

## 7. Automatic Differentiation and Backpropagation

A central achievement of Day 40 was calculating gradients of the loss with respect to neural-network parameters.

For a model parameter \(\theta\), training requires

$$
\frac{\partial\mathcal L}{\partial\theta}.
$$

I used PyTorch's automatic differentiation system:

```python
loss.backward()
```

### What happens during backpropagation?

PyTorch records differentiable tensor operations during the forward pass.

When `.backward()` is called, it applies the chain rule to propagate derivatives backward through the computational graph.

For a prediction head,

$$
\mathbf h_{\mathrm{mol}}
\rightarrow
\mathbf z
\rightarrow
\mathbf a
\rightarrow
\hat y
\rightarrow
\mathcal L,
$$

the backward pass follows the reverse dependency structure.

The resulting parameter gradients are stored in each parameter's `.grad` attribute.

### Inspecting gradients

I examined the gradient of the first linear layer:

```python
prediction_head[0].weight.grad
```

I also used:

```python
prediction_head[0].weight.grad.norm()
```

to measure the magnitude of the gradient tensor.

The experiment confirmed that the loss produced a nonzero gradient signal for the prediction head.

### Important conceptual distinction

**Backpropagation computes gradients; it does not update parameters.**

The gradient describes how the loss changes locally when a parameter changes.

A separate optimization algorithm uses this information to modify the parameters.

This distinction is fundamental to understanding neural-network training.

---

## 8. Gradient Descent and Parameter Updates

After computing gradients, I performed an optimization step using stochastic gradient descent (SGD).

The mathematical update rule is

$$
\boxed{
\theta_{t+1}
=
\theta_t
-
\eta\nabla_\theta\mathcal L
}
$$

where \(\eta\) is the learning rate.

In PyTorch:

```python
optimizer = torch.optim.SGD(
    prediction_head.parameters(),
    lr=0.01
)

optimizer.step()
```

### Understanding the learning rate

The learning rate controls how far the parameters move along the negative gradient direction.

If the learning rate is too small, optimization may proceed slowly.

If it is too large, updates may overshoot useful parameter regions or make training unstable.

### Prediction overshooting

During the exercise, the prediction moved past the target after an optimization step.

Nevertheless, the loss decreased.

This demonstrated an important principle:

**The prediction does not need to approach the target monotonically from one side for gradient descent to make progress.**

The relevant criterion is whether the objective function decreases.

In a nonlinear model, the relationship between parameter changes and output changes can be complicated.

---

## 9. Building a Complete Training Loop

The final exercise combined the individual training operations into an iterative optimization procedure.

The essential structure was:

```python
for epoch in range(num_epochs):

    optimizer.zero_grad()

    prediction = prediction_head(h_mol_fixed)

    loss = loss_fn(prediction, target)

    loss.backward()

    optimizer.step()
```

### Understanding the sequence

**1. Clear gradients**

```python
optimizer.zero_grad()
```

PyTorch accumulates gradients by default.

Clearing them before each backward pass prevents unintended accumulation across training iterations.

**2. Forward pass**

```python
prediction = prediction_head(h_mol_fixed)
```

The model calculates its current prediction using the latest parameter values.

**3. Loss calculation**

```python
loss = loss_fn(prediction, target)
```

The objective function quantifies prediction error.

**4. Backward pass**

```python
loss.backward()
```

Automatic differentiation calculates the gradients.

**5. Parameter update**

```python
optimizer.step()
```

The optimizer modifies the trainable parameters.

### Training outcome

The training procedure successfully fitted the synthetic target.

This demonstrated that the prediction head, loss function, automatic differentiation, and optimizer were correctly connected.

However, the exercise used only one training example.

A model can memorize a single target without learning a relationship that generalizes to new molecules.

---

## 10. Freezing the Geometric GNN

An important coding concept introduced today was:

```python
h_mol_fixed = h_mol.detach()
```

The `.detach()` operation returns a tensor that shares the underlying values but is disconnected from the existing automatic differentiation graph.

Consequently, the subsequent training loop treats the molecular representation as a fixed input.

### Why is this useful?

Our pipeline contains several trainable components:

- Atomic embeddings.
- Continuous-filter network.
- Molecular prediction head.

During Day 40, I wanted to focus specifically on training the prediction head.

Detaching the molecular representation prevented gradients from flowing backward into the earlier geometric GNN.

In addition, the optimizer was constructed using only the prediction head's parameters.

These two choices made the optimization scope explicit.

### Frozen representations versus end-to-end training

**Frozen-representation training**

$$
\mathbf h_{\mathrm{mol}}=\text{constant}
$$

Only the prediction head is optimized.

**End-to-end training**

$$
\mathbf h_{\mathrm{mol}}
=
g_\phi(Z,\mathbf R)
$$

Both the geometric representation model \(g_\phi\) and the prediction head \(f_\theta\) can be optimized jointly.

End-to-end training allows the atomic representations to adapt to the prediction task.

However, frozen representations can be useful for debugging, transfer learning, and isolating individual model components.

**Key learning:** The computational graph and optimizer parameter selection jointly determine which parts of a neural network are trained.

---

## 11. The Complete Molecular ML Pipeline

The full architecture developed across Days 35–40 can now be written as

$$
\boxed{
\hat y
=
f_\theta
\left(
\sum_{i=1}^{N}
\mathbf h_i^{(L)}
\right)
}
$$

where the atomic features are generated through geometric message passing:

$$
\mathbf h_i^{(\ell+1)}
=
\mathbf h_i^{(\ell)}
+
\sum_{j\in\mathcal N(i)}
\mathbf h_j^{(\ell)}
\odot
W_\phi\bigl(\boldsymbol\phi(d_{ij})\bigr).
$$

The model therefore consists of four major stages.

### Stage A — Atomic initialization

Atomic numbers are mapped to learned feature vectors.

### Stage B — Geometric message passing

Atoms exchange information using distance-dependent neural filters.

### Stage C — Molecular readout

Atomic representations are combined into a permutation-invariant molecular representation.

### Stage D — Property prediction

A neural-network head maps the molecular representation to a scalar target.

The training objective connects the predicted property to reference data and enables gradient-based parameter optimization.

This represents the first complete forward-and-training pipeline in Project 3.

---

## 12. Scientific Limitations

The Day 40 implementation was intentionally minimal.

Several limitations must be recognized before treating it as a scientifically meaningful molecular ML model.

**Synthetic target:** The training label was artificial and did not represent an experimentally measured or computed molecular property.

**Single-molecule training:** There was no molecular diversity or held-out evaluation.

**Untrained geometric representation:** The atomic embeddings and continuous filters were not optimized for the target property.

**Frozen feature extractor:** Only the prediction head was trained.

**No generalization assessment:** Successful fitting of the training example provides no evidence of predictive performance on unseen molecules.

**No physical units:** The predicted scalar had no established physical interpretation.

**Simplified architecture:** The message-passing network lacks several components commonly found in production molecular models.

These limitations do not diminish the value of the exercise. They define what was actually established: the computational pipeline and optimization mechanism work.

---

## 13. Connection to Previous Days

| Day | Main achievement |
|---|---|
| Day 35 | Molecular geometry and rigid-body symmetry |
| Day 36 | Spatial graphs and radial basis representations |
| Day 37 | Trainable distance-dependent message filters |
| Day 38 | Atomic embeddings and local environment representations |
| Day 39 | Multilayer message passing and expanded receptive fields |
| **Day 40** | **Molecular readout, scalar prediction, backpropagation, and training** |

The progression has moved from molecular coordinates to a trainable molecular prediction architecture.

A particularly important conceptual transition occurred today: the geometric GNN is no longer only a feature-construction exercise. It is now connected to an explicit prediction objective.

---

## 14. Main Scientific Takeaways

1. **Molecular readout bridges atomic and molecular representations.** Sum pooling produces a fixed-dimensional molecular vector from a variable number of atoms.

2. **Permutation invariance is essential for molecular prediction.** Predictions must not depend on arbitrary atom numbering.

3. **A prediction head connects learned representations to target properties.** Nonlinear transformations allow flexible mappings between molecular features and outputs.

4. **SiLU introduces smooth nonlinearity.** It supports differentiable transformations without introducing additional trainable parameters.

5. **Loss functions define the optimization objective.** Their scientific relevance depends on the reference data and the prediction task.

6. **Backpropagation and optimization are distinct operations.** One computes gradients; the other uses those gradients to update parameters.

7. **A training loop repeatedly connects forward evaluation, loss calculation, differentiation, and optimization.**

8. **Detaching a representation is useful for controlling gradient flow.** It enables isolated training of a prediction head.

9. **Low training loss is not equivalent to scientific predictive success.** Generalization requires meaningful data and independent evaluation.

---

## 15. Reflection and Next Steps

Day 40 completed an important architectural milestone in Project 3.

I now understand how to construct a molecular representation from atomic features, map that representation to a scalar prediction, define a training objective, compute gradients, and optimize neural-network parameters.

The most important insight was recognizing the distinction between **building a valid neural-network computation** and **training a model that learns a meaningful molecular relationship**.

The current implementation demonstrates the former.

The next stage should focus on the latter.

A natural direction for Day 41 is to move beyond one molecule and one synthetic target by constructing a small dataset of molecular geometries and associated reference properties. This would introduce dataset preparation, batching, train/test separation, and eventually end-to-end geometric GNN training.

### Central takeaway

*A molecular GNN becomes a trainable property predictor when geometric atomic representations are combined through a permutation-invariant readout, transformed by a prediction head, and optimized against reference targets. A functioning training pipeline is the foundation; scientifically meaningful data and generalization are the next challenges.*

**Day 40 completed successfully.**
