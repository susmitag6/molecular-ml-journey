# Day 17 — Training Our First Molecular Graph Neural Network

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

A future standardization experiment can compare original graphs with standardized parent structures, while preserving the active





