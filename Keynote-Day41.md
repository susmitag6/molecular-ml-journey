# Scientific Learning Journal — Day 41

**Project 3: Geometry-Aware Molecular GNNs for Energy and Force Learning**

**System:** Ethanol (C₂H₆O), 9 atoms with explicit hydrogens  
**Energy reference:** MMFF94 potential energies  
**Primary objective:** Transition from a single synthetic energy target to a multi-conformer molecular energy-learning problem.

## 1. Scientific objectives

Day 41 focused on constructing a small molecular energy dataset, training a geometric graph neural network, diagnosing poor energy prediction, and investigating numerical and physical limitations of the model.

The key questions were:

1. Can a distance-based GNN distinguish ethanol conformers?
2. Can its learned representations support energy prediction?
3. Why does end-to-end training produce nearly constant predictions?
4. How do feature scaling and readout architecture affect optimization?
5. Does improved training performance generalize to held-out conformers?
6. What are the implications of a hard distance cutoff for molecular energy and force learning?

## 2. Molecular dataset construction

Generated 20 ethanol conformers using RDKit's ETKDGv3 conformer embedding algorithm with a fixed random seed.

The coordinate dataset had shape:

`(20, 9, 3)`

Each conformer contained 9 atoms with three Cartesian coordinates per atom.

For every conformer, MMFF94 potential energy was calculated without geometry minimization.

The resulting energies ranged from approximately 1.317 to 5.881 kcal/mol in the MMFF94 energy convention.

**Scientific distinction:** ETKDG-generated conformers are not an equilibrium Boltzmann-distributed molecular dynamics trajectory. MMFF94 provides an empirical potential-energy function, not free energies or experimental measurements.

## 3. Geometric representations

For each conformer, computed the pairwise atomic distance matrix:

$$
D_{ij}=\|\mathbf r_i-\mathbf r_j\|_2.
$$

The complete distance dataset had shape:

`(20, 9, 9)`

Pairwise distances are invariant under rigid-body translation and rotation.

These distance matrices served as inputs to the previously constructed two-layer geometric GNN.

## 4. Training and test separation

Randomly partitioned the 20 conformers into:

- 16 training conformers
- 4 held-out test conformers

Defined relative energy using the minimum energy from the training set only:

$$
E_i^{\mathrm{relative}} = E_i^{\mathrm{MMFF94}} - \min_{j\in\mathrm{train}}E_j^{\mathrm{MMFF94}}.
$$

This avoids using the test-set minimum to define the training target.

The resulting test energies could legitimately be negative because the test set contained conformers lower in energy than the lowest-energy training example.

## 5. End-to-end GNN training

The network comprised:

- A trainable atomic-number embedding
- A continuous-filter network using Gaussian radial distance features
- Two residual message-passing stages
- Sum pooling across atoms
- A nonlinear molecular energy prediction head

Total trainable scalar parameters: **173**.

Updated the forward function to recompute atomic embeddings during each forward pass, allowing gradients to propagate correctly through the entire model.

Verified that gradients reached the embedding, continuous-filter network, and prediction head.

Trained using Adam with a learning rate of 0.001.

Training MSE decreased from approximately 1.8643 to 1.2058 over 200 epochs.

However, this reduction alone did not establish successful learning.

## 6. Constant-baseline comparison

The mean training energy was approximately 1.8747 kcal/mol.

A constant predictor returning this mean achieved:

**Baseline training MSE: 1.1948 (kcal/mol)²**

The trained GNN achieved:

**GNN training MSE: 1.2056 (kcal/mol)²**

The GNN therefore performed slightly worse than the constant baseline.

Its prediction standard deviation was only 0.0544 kcal/mol, compared with a reference energy standard deviation of 1.0931 kcal/mol.

**Conclusion:** The end-to-end model had learned an almost constant energy predictor.

## 7. Molecular representation diagnostics

Extracted four-dimensional molecular representations before the prediction head.

The representation tensor had shape:

`(16, 4)`

Feature standard deviations were approximately:

`[0.1140, 0.0851, 0.8383, 0.3729]`

The Euclidean distance between the first two representations was approximately 2.7688.

These observations established that the GNN did not produce identical representations for every conformer.

Calculated the prediction head's local gradient with respect to molecular features:

$$
\nabla_{\mathbf h}\hat E \approx (0.0609,-0.0466,0.0504,0.0325).
$$

Gradient norm: approximately 0.0974.

This indicated relatively weak local sensitivity of the prediction head to changes in its input features, although it did not identify a unique cause of model failure.

## 8. Linear probing

Fitted an ordinary least-squares linear model to the frozen molecular representations:

$$
\hat E=\mathbf w^\top\mathbf h+b.
$$

Achieved a training MSE of:

**0.7016 (kcal/mol)²**

This was substantially better than the original GNN and constant baseline.

**Interpretation:** The frozen representations contained information useful for fitting training energies, even though the original prediction head did not exploit it effectively.

Because this diagnostic used only 16 training examples and five linear coefficients, it did not establish generalization.

## 9. Feature scaling and optimization

Trained a five-parameter linear head using Adam.

Without standardization, after 500 epochs:

**Training MSE: 1.1443**

Examined the feature means:

`[2.3594, -6.2879, 21.9014, -0.5406]`

and population standard deviations:

`[0.1104, 0.0824, 0.8116, 0.3610]`.

Standardized the features using training-set statistics:

$$
z_{ik}=\frac{h_{ik}-\mu_k}{\sigma_k}.
$$

After standardization, each feature had approximately zero mean and unit variance.

A new linear head trained on standardized features reached:

**MSE after 500 epochs: 0.8076**

After 1,000 additional epochs:

**MSE: 0.7054**

This closely approached the least-squares optimum of 0.7016.

**Conclusion:** Feature scaling and sufficient optimization substantially improved the linear readout.

## 10. Held-out energy prediction

Evaluated the standardized linear head on four held-out ethanol conformers.

Reference relative energies:

`[1.2170, -0.4206, -0.9999, 1.1890]` kcal/mol

Predicted relative energies:

`[2.6850, 0.5266, 0.1918, 2.4906]` kcal/mol

Test metrics:

- Linear-head MAE: **1.2271 kcal/mol**
- Linear-head RMSE: **1.2416 kcal/mol**
- Constant-baseline MAE: **1.6283 kcal/mol**

The linear head improved test MAE relative to the constant baseline, but its absolute energy errors remained substantial.

Only four test conformers were available, so these metrics are statistically uncertain.

## 11. Energy offsets versus energy differences

All four test predictions were higher than their reference energies.

Mean prediction error:

**+1.2271 kcal/mol**

Pearson correlation:

**0.9932**

After subtracting the mean separately from test predictions and reference energies:

- Centered MAE: **0.1576 kcal/mol**
- Centered RMSE: **0.1892 kcal/mol**

This indicated that the predicted energy variations across these four test conformers were considerably more accurate than their absolute values.

However, test-set centering used reference labels and was strictly a retrospective diagnostic, not a valid prospective evaluation.

A correction calculated using training residuals was approximately −0.00000191 kcal/mol and did not materially change test MAE.

**Conclusion:** The positive test-set offset could not be corrected by a global bias estimated from training data.

## 12. Training and test energy distributions

Training energy range:

`0.0000 to 3.5637 kcal/mol`

Test energy range:

`-0.9999 to 1.2170 kcal/mol`

Training mean:

`1.8747 kcal/mol`

Test mean:

`0.2464 kcal/mol`

Two of four test conformers had energies below the minimum training energy.

This demonstrated target-space extrapolation for half of the test set.

Target-space extrapolation does not necessarily imply geometric extrapolation.

## 13. Geometric coverage diagnostics

For each test conformer, found its nearest training conformer using the RMS difference between pairwise atomic distance matrices.

Nearest geometric distances:

`[0.0650, 0.0840, 0.1450, 0.1436] Å`

The nearest training conformers nevertheless had substantially different energies.

For test conformer 0, the nearest training geometry differed by only approximately 0.065 Å under the matrix metric, while its relative energy was approximately 2.347 kcal/mol lower.

**Interpretation:** A global, unweighted distance-matrix metric may not adequately reflect which local geometric changes matter most for potential energy.

## 14. Hard-cutoff graph connectivity

Examined test conformer 0 and its nearest training conformer.

The largest atomic-distance difference occurred between hydrogen atoms 5 and 8:

- Training H–H distance: 1.9951 Å
- Test H–H distance: 2.2209 Å
- Absolute difference: 0.2258 Å

The mean absolute distance-matrix difference was only 0.0399 Å.

At the model's 2.0 Å cutoff, exactly one undirected edge changed connectivity:

**H5–H8: connected in training, disconnected in test.**

This identified a possible source of nonsmooth behavior in the model's energy representation.

## 15. Smooth cutoff functions

Studied the cosine cutoff:

$$
f_c(r)= \begin{cases} \frac12\left[\cos(\pi r/r_c)+1\right],&r<r_c,\\ 0,&r\ge r_c.
\end{cases}
$$

At a cutoff radius of 2.0 Å:

| Distance (Å) | Hard weight | Cosine weight |
|---|---:|---:|
| 1.900 | 1 | 0.006156 |
| 1.950 | 1 | 0.001541 |
| 1.995 | 1 | 0.000015 |
| 2.000 | 0 | 0 |
| 2.050 | 0 | 0 |
| 2.221 | 0 | 0 |

The cosine cutoff smoothly approaches zero with a continuous first derivative.

**Physical relevance:** Smoothly vanishing interactions are important when constructing differentiable potential-energy surfaces and learning forces.

The cutoff experiment did not yet modify the GNN or demonstrate improved energy accuracy.

## 16. Key scientific conclusions

1. Training loss alone is insufficient evidence of meaningful molecular energy learning.
2. A constant-energy baseline is essential for identifying prediction collapse.
3. Intermediate molecular representations can retain useful information even when the final readout performs poorly.
4. Feature standardization can substantially improve gradient-based optimization.
5. Low centered energy error does not substitute for valid held-out absolute energy evaluation.
6. Small test sets can produce deceptively strong correlations.
7. Training and test conformers may occupy different energy distributions.
8. Unweighted geometric similarity does not guarantee energetic similarity.
9. Hard neighbor cutoffs can create discontinuities in learned energy models.
10. Energy accuracy and force accuracy must ultimately be evaluated separately.

## 17. Day 42 research roadmap

The next phase should focus on improving the physical and numerical formulation of the geometric energy model.

**Priority A — Differentiable geometry**

Replace NumPy-based distance construction inside the forward computation with PyTorch operations so gradients can propagate from predicted energy to Cartesian coordinates.

**Priority B — Smooth local interactions**

Integrate a smooth cutoff into the continuous-filter message function and check behavior near the cutoff boundary.

**Priority C — Energy and force consistency**

Compute model forces using:

$$
\mathbf F_i=-\nabla_{\mathbf r_i}E_\theta.
$$

Verify translation invariance, rotation invariance of energy, and rotation equivariance of forces.

**Priority D — Better molecular energy learning**

Improve conformer coverage, compare stronger baselines, and eventually incorporate MMFF94 force labels for energy-and-force training.

**Long-term direction:** Apply the validated geometric energy-learning workflow to alanine dipeptide configurations generated by molecular dynamics.

## 18. End-of-day assessment

Day 41 moved Project 3 from an illustrative GNN prototype toward a measurable molecular machine-learning experiment.

The most important achievement was not a low prediction error, but a sequence of controlled scientific diagnostics that identified:

- A nearly constant initial predictor
- Useful information in frozen geometric representations
- A significant numerical optimization issue
- A difference between absolute energy accuracy and relative energy variation
- Limited energy-range coverage
- A physically relevant hard-cutoff limitation

These findings provide a concrete, evidence-based starting point for Day 42.
