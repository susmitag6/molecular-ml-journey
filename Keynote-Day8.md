
Today I have covered the foloowing concepts:
* Baseline: DummyRegressor — why a model must beat a trivial predictor.
* Ridge regression: additive/linear mapping from Morgan bits to pIC50 and the role of regularization \(\alpha\).
* Tree ensembles: Random Forest vs Extra Trees and nonlinear feature interactions.
* Bias–variance: bias as systematic error across possible training samples; variance as sensitivity to the particular training sample.
* Underfitting vs overfitting: interpreted using train and test performance together.
* Model complexity: experimentally varied RF max_depth.
* Parameters vs hyperparameters: learned coefficients/tree structure vs choices such as alpha and max_depth.
* Train / validation / test: why validation is used for model selection and the final test set should remain untouched.

## Classical ML
* Ridge regression assumes an approximately linear, additive relationship between Morgan fingerprint features and bioactivity. If molecular environments A and B are present, their contributions are essentially added through their coefficients.
* Random Forest can model nonlinear interactions. The influence of environment A can depend on whether environments B or C are also present, allowing the model to capture more complicated structure–activity relationships.
* Extra Trees also captures nonlinear interactions, but introduces additional randomness when constructing the trees. This can reduce variance, but may increase bias. Therefore, greater flexibility or randomization does not automatically produce better generalization.

$$
\text{Dummy}
\rightarrow
\text{no structure information used}
$$

$$
\text{Ridge} \rightarrow \text{structure information + approximately additive linear mapping}
$$

$$
\text{RF / Extra Trees} \rightarrow \text{structure information + nonlinear interactions}
$$

# Bias - Variance Trade off

$$
\text{Bias} = \text{Sysmetic error in the Learning Procedure}
$$

$$
\text{Variance} = \text{Sensivity of the Learned model on the Training Dataset}
$$

$$
\text{high flexibility}\rightarrow\text{potentially higher variance}.
$$

$$
\text{expected prediction error} = \text{bias}^2+\text{variance}+\text{irreducible noise}.
$$

For my ChEMBL IC50 dataset where biological assays themselves contain experimental variability, the cause of bias, variance and ireeducible noise as foloowing:

$$
\begin{aligned}
\text{bias} \Rightarrow \text{model/representation is too restrictiv} \\
\text{variance} \Rightarrow \text{the learned relationship changes strongly with the particular training molecules} \\
\text{irreducible noise} \Rightarrow \text{experimental variability in measured bioactivity}
\end{aligned}
$$

To understand the Bias-Variance Trade off we need to track all three conditions:

$$
\text{Train Performance} + \text{Test Performance} + \text{Generalization gap}
$$

we'd choose hyperparameters based on validation performance, not training performance.
# Learned about how the model parameter influence the model performance
A Random Forest is a collection of decision trees. The depth is how many successive decision rules a tree can make from its root down to a prediction.

we'd choose hyperparameters based on validation performance, not training performance.

$$
\text{underfitting} \rightarrow \boxed{\text{useful complexity}} \rightarrow \text{overfitting}
$$
Model complexity should be selected according to performance on unseen validation data, not according to how well the model fits its training data.

## Model parameters vs hyperparameters 
* Model Parameters : Parameters are learned from the training data
* Hyperparameters : hyperparameters are choices controlling the learning process/model
I identified the max depth, number of trees, regularziation strength as hyparameters 
$$
\text{Train} \xrightarrow{\text{learn}} \text{parameters}
$$

$$
\text{Validation} \xrightarrow{\text{choose}} \text{hyperparameters/model}
$$

$$
\text{Test} \xrightarrow{\text{evaluate}} \text{final generalization}
$$
