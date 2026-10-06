# Day 15 — Comparing Random Forests and Neural Networks for AChE Activity Prediction

Today I started with the question:

**Can a neural network outperform the Random Forest model for AChE activity prediction?**

To make the comparison meaningful, I kept the **count-Morgan fingerprint representation** and the **scaffold-aware data split** fixed while changing the learning algorithm:

$$
\text{Count Morgan} \rightarrow \begin{cases} \text{Random Forest}\\ \text{Neural Network} \end{cases}
$$

This allows me to investigate the effect of the model architecture without simultaneously changing the molecular representation.

## 1. Neural-network architecture

My initial neural-network architecture was

$$
2048 \rightarrow 512 \rightarrow 128 \rightarrow 1.
$$

The 2048 input features correspond to the count-Morgan fingerprint. The final output is one continuous value: predicted pIC50.

For a hidden layer,

$$
\mathbf{h}=f(W\mathbf{x}+\mathbf{b}),
$$

where \(W\) contains the weights, \(\mathbf{b}\) contains the biases, and \(f\) is the activation function.

I used ReLU:

$$
\mathrm{ReLU}(z)=\max(0,z).
$$

Without nonlinear activation functions, stacking linear layers is mathematically equivalent to another linear transformation:

$$
\boxed{
\text{Linear}\rightarrow\text{Linear}\rightarrow\text{Linear}
\equiv
\text{one linear transformation}
}
$$

Adding ReLU prevents this collapse and allows the network to represent nonlinear relationships and interactions between fingerprint features.

Thus, the network is

$$
\underbrace{2048}_{\text{count-Morgan}} \rightarrow \underbrace{512}_{\text{hidden}} \xrightarrow{\mathrm{ReLU}} \underbrace{128}_{\text{hidden}} \xrightarrow{\mathrm{ReLU}} \underbrace{1}_{\widehat{\mathrm{pIC50}}}.
$$

This network contains **1,114,881 trainable parameters**. This is a very high-capacity model relative to the approximately 5,000 molecules initially available for training, so overfitting is an important concern.

$$
\boxed{\text{Very low training loss does not necessarily mean a good molecular model.}}
$$

## 2. How a neural network learns

Training follows the sequence

$$
\boxed{\text{Forward pass} \rightarrow \text{Loss} \rightarrow \text{Backpropagation} \rightarrow \text{Weight update}
}
$$

For a particular weight \(w\), backpropagation calculates

$$
\frac{\partial L}{\partial w},
$$

which describes how the loss changes with respect to that weight.

For simple gradient descent, the update can be written as

$$
w_{\mathrm{new}} = w_{\mathrm{old}} - \eta\frac{\partial L}{\partial w},
$$

where \(\eta\) is the learning rate.

In PyTorch, the important operations are

$$
\texttt{model(X\-batch)} \rightarrow \text{forward pass},
$$


$$
\texttt{criterion(...)} \rightarrow \text{calculate loss},
$$

$$
\texttt{loss.backward()} \rightarrow \text{calculate gradients},
$$

and

$$
\texttt{optimizer.step()} \rightarrow \text{update parameters}.
$$

Before calculating gradients for a new batch, I use `optimizer.zero_grad()` because PyTorch otherwise accumulates gradients from successive backward passes.

## 3. Batches and epochs

Rather than processing the entire training dataset before every parameter update, I divided the molecules into mini-batches.

For a batch,

$$
\text{batch} \rightarrow \text{forward} \rightarrow \text{loss} \rightarrow \text{backpropagation} \rightarrow \text{weight update}.
$$

Processing all training batches once constitutes

$$
\boxed{1\text{ epoch}}.
$$

Repeating this process over many epochs trains the neural network.

I also used `shuffle=True` for the training DataLoader. Without shuffling, every epoch would contain the same mini-batches in the same sequence. Shuffling changes the batch composition between epochs and reduces dependence on arbitrary ordering and fixed batch composition.

Shuffling itself, however, does not prevent overfitting.

## 4. Training, validation, and test data

I learned an important distinction between the three datasets:

$$
\text{Development data} \begin{cases}
\text{Training} &\rightarrow \text{learn model parameters}\\
\text{Validation} &\rightarrow \text{select architecture/hyperparameters and monitor training}
\end{cases}
$$

After model-development decisions are complete,

$$
\text{Final selected model}
\rightarrow
\boxed{\text{Independent test set}}.
$$

The test set should ideally remain untouched until the model-development process is finished.

This is particularly important for this project because the original random test set has already been inspected repeatedly during earlier Random Forest experiments. Therefore, it should not later be described as a pristine, untouched final test set.

## 5. Scaffold-aware neural-network validation

Because my intended application involves prediction for new scaffold families, I used a scaffold-aware validation split rather than a random validation split.

Starting from the existing 5,029 training molecules, I obtained

$$
4099\text{ NN-training molecules}
$$

and

$$
930\text{ validation molecules}.
$$

The Murcko-scaffold overlap was

$$
\boxed{0}.
$$

Thus, the validation molecules belonged to scaffold groups absent from the NN-training set.

This does not necessarily mean that every validation molecule is chemically very dissimilar from every training molecule, because different Murcko scaffolds can still have relatively high fingerprint similarity. Therefore, this experiment is best described as **unseen-scaffold generalization**, rather than automatically as pure chemical extrapolation.

## 6. Early stopping

During training, the training MSE continued decreasing while the scaffold-validation MSE stopped improving.

For the first large NN experiment, the best validation result occurred around epoch 8:

$$
\text{Validation MSE}\approx1.223.
$$

The validation RMSE was approximately

$$
\sqrt{1.223}\approx1.106\text{ pIC50 units}.
$$

This demonstrated why the final training epoch is not necessarily the best model.

I therefore implemented **early stopping**. With a patience of five epochs, training stops after five consecutive epochs without validation improvement, but the weights from the epoch with the lowest validation loss are restored.

Thus,

$$
\boxed{\text{stopping epoch}\neq\text{best-model epoch}}
$$

in general.

Early stopping is a form of regularization because it prevents the model from continuing to adapt indefinitely to the training data.

## 7. Random Forest versus neural network

I then compared the Random Forest and neural network using the **same count-Morgan representation and exactly the same scaffold-disjoint train/validation split**.

The results were:

| Model | Validation MAE | Validation RMSE | Validation \(R^2\) |
|---|---:|---:|---:|
| Random Forest | **0.653** | **0.900** | **0.633** |
| Large NN | 0.793 | 1.106 | 0.445 |

The Random Forest clearly outperformed this initial neural network on this validation split.

This result does **not** mean that Random Forests are universally superior to neural networks. It means that, with this dataset, count-Morgan representation, scaffold split, and the NN configuration tested, the Random Forest generalized better.

$$
\boxed{\text{More sophisticated model} \neq \text{automatically better prediction}}
$$

## 8. Reducing neural-network capacity

Because the original network had approximately 1.1 million parameters, I hypothesized that reducing model capacity might reduce variance and improve generalization.

I changed the architecture to

$$
2048\rightarrow128\rightarrow32\rightarrow1,
$$

which contains

$$
\boxed{266,433\text{ parameters}}.
$$

This represents approximately a 76% reduction in parameter count.

However, the smaller network produced

$$
\text{MAE}=0.828,
$$

$$
\text{RMSE}=1.132,
$$

and

$$
R^2=0.419.
$$

Thus, simply reducing the number of parameters did **not** improve scaffold generalization.

This experiment illustrates that reducing model complexity does not automatically improve validation performance. Excessive reduction in capacity can also increase bias, and a single experiment cannot establish the exact bias–variance mechanism responsible for the observed result.

## 9. Weight decay as regularization

Instead of removing network capacity, I next investigated **weight decay**, which discourages the optimizer from learning excessively large parameter values.

Conceptually, L2 regularization can be written as

$$ L_{\mathrm{total}} = L_{\mathrm{data}} + \lambda\sum_i w_i^2. $$

Here, \(\lambda\) controls the regularization strength.

If \(\lambda\) is too small, regularization may have little effect. If it is too large, the network can become unable to fit the training data adequately and may underfit.

With

$$
\lambda=10^{-4},
$$

the best validation MSE was approximately

$$
1.259,
$$

which was slightly worse than the unregularized large NN.

However, with

$$
\lambda=10^{-3},
$$

the best validation MSE improved to

$$
\boxed{1.110}.
$$

The corresponding metrics were

$$
\text{MAE}=0.780,
$$

$$
\text{RMSE}=1.054,
$$

and

$$
R^2=0.496.
$$

Thus, stronger weight decay improved this NN configuration in this particular run, although the Random Forest remained substantially better:

| Model | MAE ↓ | RMSE ↓ | \(R^2\) ↑ |
|---|---:|---:|---:|
| Random Forest | **0.653** | **0.900** | **0.633** |
| NN + weight decay \(10^{-3}\) | 0.780 | 1.054 | 0.496 |
| Large NN, no weight decay | 0.793 | 1.106 | 0.445 |
| Small NN | 0.828 | 1.132 | 0.419 |

I should not conclude that \(10^{-3}\) is universally optimal because only a few configurations were tested, the same validation set was used for model-development decisions, and neural-network training is stochastic.

## 10. Representation remains a fundamental limitation

The comparison also highlighted an important limitation shared by both models.

Both the Random Forest and feed-forward neural network receive a predefined count-Morgan fingerprint:

$$
\text{molecular structure}
\rightarrow
\text{fixed fingerprint}
\rightarrow
\text{ML model}.
$$

The fingerprint compresses the molecular graph before the ML model sees it. Some explicit structural information can therefore be lost or obscured, including detailed graph connectivity beyond the fingerprint radius, the correspondence between features and particular atoms, and 3D conformational information. Hashing can also map different local environments to the same fingerprint position.

Our previous activity-cliff analysis demonstrated why this matters. Structurally related compounds can have very different measured activities, and some molecules can even become indistinguishable under a particular fingerprint representation.

Therefore,

$$
\boxed{
\text{A model cannot recover information that its input representation has discarded.}
}
$$

This helps motivate the next stage of molecular deep learning. Instead of supplying a neural network with a predefined Morgan fingerprint, a graph neural network can operate on the molecular graph and learn its own molecular representation.

However, a GNN is not guaranteed to solve activity cliffs either. A 2D molecular graph still does not explicitly contain all information relevant to measured activity, such as protein–ligand geometry, conformational ensembles, assay conditions, and other experimental factors.

## Main lesson

Today I learned that evaluating molecular ML models requires more than choosing a sophisticated algorithm.

For this AChE dataset:

$$
\boxed{\text{representation} + \text{validation strategy} + \text{model capacity} + \text{regularization} \rightarrow \text{observed generalization} }
$$

The Random Forest currently remains the stronger model for count-Morgan fingerprints under our scaffold-aware evaluation. Neural-network regularization improved performance, but did not close the gap.

The next important question is therefore not simply:

**“Can I make the neural network larger?”**

but rather:

$$
\boxed{\text{Can the model learn a better molecular representation?}}
$$

This provides the motivation for moving from fixed fingerprints toward graph-based molecular learning.


