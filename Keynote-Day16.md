Day 16 : From Fixed Fingerprints to Graph Neural Networks

$$
\boxed{\text{Morgan fingerprint + ML}\rightarrow\text{Molecular graph + GNN}}
$$

both build information from local atomic neighborhoods, but Morgan uses a predefined fingerprint algorithm, whereas a GNN can learn how neighborhood information should be transformed for the prediction task.

$$
\boxed{k\text{ GNN layers}\Rightarrow
\text{information can propagate approximately }k\text{ bonds}}
$$

The important terminology is that these are not really new chemical features in the hand-designed sense. We usually call them learned node representations or node embeddings.
So:

$$
\boxed{
\text{initial atom features}
\xrightarrow{\text{message passing}}
\text{context-dependent learned atom embeddings}
}
$$

GNNs commonly use a global pooling/readout operation.

$$
\boxed{
\text{Atoms}
\rightarrow
\text{Message passing}
\rightarrow
\text{Atom embeddings}
\rightarrow
\text{Global pooling}
\rightarrow
\text{Molecular embedding}
\rightarrow
\widehat{\mathrm{pIC50}}
}
$$

- Sum pooling retains information related to molecular size because contributions accumulate.
- Mean pooling normalizes by the number of atoms and represents something closer to the average learned atomic environment.
So the distinction is:

$$
\text{Morgan: molecular graph}
\xrightarrow{\text{fixed RDKit algorithm}}
\text{fingerprint}
\xrightarrow{\text{learned model}}
\widehat{\mathrm{pIC50}}
$$

versus

$$
\text{GNN: molecular graph}
\xrightarrow{\text{learned-message-passing}}
\text{molecular embedding}
\xrightarrow{\text{learned predictor}}
\widehat{\mathrm{pIC50}}.
$$

The molecular graph supplied to the GNN can be the same, but the GNN learns how to transform atom, bond, charge, and connectivity information according to the prediction target. Therefore, a model trained to predict aqueous solubility may learn different molecular embeddings from a model trained to predict AChE pIC50.
