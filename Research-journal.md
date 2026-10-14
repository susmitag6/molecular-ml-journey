# Research Journal — Are Graph Neural Networks Really Useful in Drug Discovery?

**Date:** October 15, 2026  
**Project:** AChE Bioactivity Prediction — Molecular Machine Learning  
**Topic:** Current evidence, limitations, and recent developments in GNNs

## 1. Motivation

During my AChE bioactivity prediction project, I observed that a Random Forest trained on Morgan fingerprints outperformed my four-layer graph neural network (GNN).

The Random Forest achieved a validation MAE of **0.661 pIC50**, compared with **0.813 pIC50** for the GNN.

This led me to ask a broader question:

**If traditional molecular fingerprints work so well, is there convincing scientific evidence that GNNs are useful for drug discovery?**

## 2. What I learned

The scientific literature provides substantial evidence that GNNs are valuable for molecular property prediction and drug discovery. However, they do not consistently outperform simpler machine-learning approaches, particularly on modest-sized bioactivity datasets.

I identified four important research directions:

**3D geometric GNNs:** These models incorporate atomic coordinates, distances, and geometric relationships rather than relying exclusively on molecular connectivity. They are especially relevant when molecular geometry influences the predicted property.

**Molecular foundation models:** Researchers increasingly pretrain models on large molecular datasets before adapting them to smaller, task-specific datasets. This may reduce the limitations of training a GNN from scratch with only a few thousand molecules.

**Hybrid fingerprint–graph models:** Recent approaches combine established molecular descriptors with learned graph representations, recognizing that both may contain useful and complementary information.

**Advanced graph transformers:** New architectures use attention mechanisms and richer graph representations to capture molecular relationships that conventional message-passing networks may miss.

## 3. Important limitations

Despite these developments, accurate bioactivity prediction remains challenging.

Molecular structure alone does not fully describe protein conformational changes, solvent effects, binding-site interactions, experimental conditions, or measurement uncertainty.

Activity cliffs are another major difficulty: two structurally similar molecules may exhibit dramatically different potencies.

Therefore, even sophisticated GNNs may struggle when predicting unfamiliar chemical scaffolds or subtle structure–activity relationships.

I also learned that strong benchmark performance does not automatically imply reliable predictions for a new drug-discovery dataset.

## 4. Connection to my AChE project

My results do not demonstrate that GNNs are ineffective.

Instead, they show that a relatively small dataset and a basic graph architecture may not provide enough information for a GNN to outperform a well-established molecular fingerprint baseline.

My Day 21 analysis also showed that the GNN predicted 39% of validation molecules more accurately than the Random Forest, although combining their predictions produced only a negligible overall improvement.

This reinforces the importance of evaluating models carefully rather than assuming that newer architectures must be superior.

## 5. Implications for my future learning

I plan to complete the current AChE project using rigorous validation, activity-cliff analysis, and uncertainty estimation.

After establishing this foundation, I want to explore pretrained molecular representations and geometric GNNs.

Given my background in molecular dynamics and statistical physics, I am particularly interested in approaches that combine molecular graphs with 3D conformations, protein environments, and dynamical information.

These methods may provide a meaningful connection between my previous simulation experience and modern molecular machine learning.

## 6. Papers for further reading

**Reference 1 — Review**

*Graph Neural Networks in Modern AI-Aided Drug Discovery.*

PubMed: https://pubmed.ncbi.nlm.nih.gov/40959983/

**Why I want to read it:** To understand the broader applications of GNNs in drug discovery, their architectural developments, and their current limitations.

**Reference 2 — Recent architecture development**

*Enhancing molecular property prediction of transformer models with dual graph representation.*

*Nature Communications* (2026).

Article: https://www.nature.com/articles/s41467-026-75005-9

**Why I want to read it:** To examine how richer graph representations and transformer-based architectures can improve molecular property prediction, and how their performance is evaluated against alternative approaches.

## 7. Final reflection

My main conclusion is that **GNNs are scientifically valuable, but their usefulness depends on the problem, dataset, molecular representation, and evaluation strategy**.

A sophisticated model is not automatically a better model.

For my development as a computational drug-discovery researcher, understanding why a model succeeds or fails is more important than simply implementing the latest architecture.

**Key takeaway:** I should build strong foundations in molecular representations, validation, and error analysis before progressing to geometric GNNs, molecular foundation models, and physics-informed machine learning.
