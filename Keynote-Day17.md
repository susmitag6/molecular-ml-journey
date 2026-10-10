# Day 17 - Training Our First Molecular Graph Neural Network

**Project 1:** AChE Bioactivity Prediction  
**Tools:** RDKit, PyTorch, PyTorch Geometric  
**Objective:** Convert the cleaned AChE molecular dataset into graph representations, construct scaffold-aware training and validation sets, and train a bond-aware graph neural network to predict pIC50.

## 1. Validate Molecular Graph Construction

Before training a graph neural network (GNN), I needed to verify that every molecular structure could be represented consistently as a graph.

In a molecular graph:

- **Nodes** represent atoms and contain atom-level features.
- **Edges** represent chemical bonds and contain bond-level features.
- **Graph-level labels** represent measured molecular properties, in this case pIC50.

A credible molecular ML workflow must document how invalid structures, disconnected fragments, salts, counterions, and stereochemical information are handled. These choices affect the chemical meaning of the model inputs and can influence predictive performance.

### Disconnected molecular structures

I examined the SMILES string:

`CCO.[Na+]`

The period separates two disconnected components: ethanol and a sodium ion.

RDKit represents this structure as a single molecular object containing:

- Four atoms
- Two chemical bonds
- Two disconnected components

PyTorch Geometric can also represent this structure as one graph containing disconnected components. Message passing operates along existing edges, so the sodium ion cannot exchange messages with ethanol atoms. However, graph-level pooling can still combine information from both components.

**Key lesson:** A molecular graph does not have to be connected, but disconnected components require a deliberate preprocessing policy.

## 2. Handle Molecules with No Bonds

A special case occurs when a molecular structure contains a single atom, such as `[Na+]`.

This structure has one node but no edges.

To ensure consistent PyTorch Geometric tensor shapes, I modified the graph converter to construct empty edge tensors when no bonds exist:

```python
if edge_list:
    edge_index = torch.tensor(
        edge_list, dtype=torch.long
    ).t().contiguous()

    edge_attr = torch.tensor(
        edge_features, dtype=torch.float32
    )
else:
    edge_index = torch.empty(
        (2, 0), dtype=torch.long
    )

    edge_attr = torch.empty(
        (0, 7), dtype=torch.float32
    )
```

This preserves the expected tensor dimensions:

- `edge_index`: `[2, number_of_directed_edges]`
- `edge_attr`: `[number_of_directed_edges, 7]`

For a molecule with no bonds, these become `[2, 0]` and `[0, 7]`.

**Key lesson:** Molecular graph conversion must handle edge cases without producing inconsistent tensor dimensions.

## 3. Validate the Complete AChE Dataset

I converted all **6,287 cleaned AChE molecules** into PyTorch Geometric `Data` objects.

**Results:**

- Successfully converted molecules: **6,287**
- Failed conversions: **0**
- Atom feature dimension: **34**
- Bond feature dimension: **7**

Each graph stores atom features, directed bond connections, bond features, and the experimental pIC50 target.

### Disconnected-fragment analysis

The number of connected components per molecule was:

| Number of fragments | Molecules |
|---|---:|
| 1 | 5,252 |
| 2 | 785 |
| 3 | 214 |
| 4 | 24 |
| 5 | 12 |

A total of **1,035 molecules (approximately 16.5%)** contained multiple disconnected components.

Fragment counts alone cannot identify whether these components are counterions, solvents, salts, or additional organic structures.

I therefore inspected representative examples, including quaternary ammonium compounds with chloride counterions and structures containing organic counterions such as tosylate, methyl sulfate, or tartrate.

### Preprocessing decision

For the initial GNN benchmark, I **retained the original full molecular graphs**, including disconnected components.

I did not automatically discard counterions because the largest-fragment rule is not universally safe. Removing a component can change the chemical representation, and some records contain more than one substantial organic fragment.

A future standardization experiment can compare original graphs with standardized parent structures, while preserving the active component's formal charge and applying a consistent policy across models.

**Key lesson:** Molecular standardization is an experimental design choice, not merely a technical cleaning step.

## 4. Understand PyTorch Geometric Batching

Unlike fixed-length molecular fingerprints, molecular graphs contain different numbers of atoms and bonds.

PyTorch Geometric batches graphs by concatenating their node and edge tensors.

For three example molecules containing 26, 26, and 30 atoms:

- Total atoms: **82**
- Batched atom features: `[82, 34]`
- Graph targets: `[3]`

PyTorch Geometric also creates a `batch` vector indicating which molecule each atom belongs to.

The edge indices are automatically adjusted to refer to the correct atoms in the concatenated representation.

Graph-level pooling uses the membership vector to produce one molecular embedding per molecule.

**Key lesson:** PyTorch Geometric batches variable-sized molecular graphs without padding every molecule to the same atom count.

## 5. Construct Scaffold-Aware Training and Validation Sets

To compare the GNN with the previously trained fingerprint-based models, I reused the exact Day 15 training and validation molecule indices.

The split contained:

| Subset | Molecules |
|---|---:|
| Training | 4,099 |
| Validation | 930 |

The split was constructed using molecular scaffold groups, with no scaffold overlap between training and validation under the grouping definition used for that split.

An important detail is that `GroupShuffleSplit(test_size=0.2)` selects approximately 20% of the **groups**, not necessarily 20% of the molecules. Therefore, the resulting number of validation molecules does not have to equal exactly 20% of the dataset.

I also observed that recomputing scaffold labels with a different procedure produced different subset sizes. Rather than assume the definitions were identical, I reused the saved Day 15 indices to maintain a consistent molecule-level comparison.

The principle was:

**Same molecules + same labels + same evaluation metrics = a more meaningful comparison between models.**

This does not eliminate all experimental differences, but it avoids comparing performance on different validation molecules.

## 6. Create Graph DataLoaders

I constructed PyTorch Geometric datasets from the selected molecule indices and used `DataLoader` with a batch size of 64.

```python
train_graphs = [
    graphs[int(i)] for i in gnn_train_idx
]

val_graphs = [
    graphs[int(i)] for i in gnn_val_idx
]

train_loader = DataLoader(
    train_graphs,
    batch_size=64,
    shuffle=True
)

val_loader = DataLoader(
    val_graphs,
    batch_size=64,
    shuffle=False
)
```

The training set produced 65 batches and the validation set produced 15 batches.

A representative training batch contained:

- 64 molecular graphs
- 1,904 atoms
- 4,132 directed edges
- 64 pIC50 targets

Because each chemical bond was represented by two directed edges, the batch contained 2,066 undirected chemical bonds.

## 7. Construct the First Bond-Aware GNN

I implemented a graph neural network using **GINEConv**, which incorporates both atom and bond information during message passing.

The model architecture was:

```text
Atom features [34]
       |
       v
Linear atom encoder
       |
       v
Atom embeddings [64]
       |
       + <--- Bond encoder [7 -> 64]
       |
       v
GINEConv + ReLU
       |
       v
GINEConv + ReLU
       |
       v
Global mean pooling
       |
       v
Linear regression head
       |
       v
Predicted pIC50
```

The model contained **19,457 trainable parameters**.

The two message-passing layers allow information to propagate through local graph neighborhoods. The graph pooling operation combines the resulting atom embeddings into a single molecular representation.

I verified that the model returned one prediction for each molecule:

- Prediction shape: `[64]`
- Target shape: `[64]`

Matching shapes are essential for regression loss calculation. A mismatch such as predictions `[64, 1]` and targets `[64]` can cause unintended broadcasting.

## 8. Train the GNN

I trained the model using:

- **Loss function:** Mean squared error
- **Optimizer:** Adam
- **Learning rate:** 0.001
- **Weight decay:** 0.0001
- **Maximum epochs:** 60
- **Early-stopping patience:** 10 epochs
- **Checkpoint selection:** Lowest validation MSE

Training updated model parameters through forward propagation, loss calculation, backpropagation, and optimizer steps.

Validation evaluated the model without updating parameters.

The best model checkpoint was restored after training.

**Best epoch:** 38  
**Best validation MSE:** 1.4233  
**Best validation RMSE:** 1.1930

## 9. Evaluate Predictive Performance

The trained GNN was evaluated on the 930 scaffold-disjoint validation molecules.

| Metric | GNN |
|---|---:|
| MAE | 0.928 |
| RMSE | 1.193 |
| R² | 0.354 |

For comparison, the previously trained random forest using count Morgan fingerprints achieved:

| Metric | Random forest |
|---|---:|
| MAE | 0.653 |
| RMSE | 0.900 |
| R² | 0.633 |

The random forest substantially outperformed this initial GNN configuration.

This does not imply that graph neural networks are generally inferior to random forests. It shows that the tested GNN architecture and training setup did not outperform the fingerprint-based baseline on this validation split.

## 10. Diagnose Prediction-Range Compression

The GNN's predicted pIC50 distribution was narrower than the experimental distribution.

| Statistic | Experimental | Predicted |
|---|---:|---:|
| Mean pIC50 | 5.980 | 5.923 |
| Standard deviation | 1.484 | 0.946 |

The predicted standard deviation was approximately 64% of the experimental standard deviation.

This suggested that the model was producing predictions closer to the middle of the activity range.

To investigate this, I divided validation molecules into low-, medium-, and high-potency groups based on experimental pIC50.

| Activity group | Mean signed error | MAE |
|---|---:|---:|
| Low potency | +0.804 | 1.139 |
| Medium potency | +0.104 | 0.496 |
| High potency | −1.082 | 1.147 |

The GNN systematically overestimated the inhibitory potency of weak inhibitors and underestimated the potency of strong inhibitors.

It also performed much better for medium-potency molecules than for either extreme.

This demonstrated why overall R² alone is insufficient to characterize model performance.

## 11. Distinguish Underfitting from Overfitting

I evaluated the best checkpoint on the training molecules.

| Metric | Training | Validation |
|---|---:|---:|
| MAE | 0.927 | 0.928 |
| RMSE | 1.223 | 1.193 |
| R² | 0.362 | 0.354 |
| Prediction standard deviation | 0.875 | 0.946 |
| Target standard deviation | 1.531 | 1.484 |

Training and validation performance were very similar.

The absence of a large generalization gap, combined with limited training performance and prediction-range compression on both subsets, was more consistent with **insufficient fit or limited extracted predictive signal** than with pronounced overfitting.

However, this observation alone does not identify whether the main limitation comes from graph features, model architecture, optimization, or measurement noise.

## 12. Scientific Conclusions

Day 17 established a complete molecular graph learning workflow:

1. Validate molecular graph construction and edge cases.
2. Inspect disconnected fragments and document preprocessing decisions.
3. Build PyTorch Geometric graph datasets and batches.
4. Reuse scaffold-aware molecule indices for model comparison.
5. Construct a bond-aware GINE graph neural network.
6. Train using validation-based early stopping.
7. Evaluate overall prediction performance and systematic errors.
8. Diagnose training and validation behavior.

The most important finding was that **a GNN does not automatically outperform a well-designed molecular fingerprint baseline**.

The initial GNN learned useful structure–activity relationships but showed limited predictive accuracy, compressed the activity range, and performed poorly at the extremes of inhibitory potency.

These findings motivated Day 18's controlled investigation of graph-level pooling.

## Key Takeaway

**A scientifically credible molecular ML workflow is not just about implementing a neural network. It requires chemically defensible representations, reproducible data splits, meaningful baselines, careful evaluation, and an understanding of model failure modes.**
