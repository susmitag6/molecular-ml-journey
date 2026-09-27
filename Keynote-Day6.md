# Date - 27.09.2026

## Scaffold Split and Model Adaptibility

Yesterday I have learned the  the  molecular information might be  lost  in the morgan fingerprint represenation of molecular structure which might result large prediction error in the bioactive response of of liigand molecule. 

Aslo in the randomforest regression model random spliting of train test data , the test data might have already structural similarity with train data this results large model performnace.
But in practical case our intention is to predict the bioactive response for unfamiliar chemical space.
For this today I learned the scaffold splittin..
Workfolow for Scaffold split:
1. The molecule is orpoed in scaffold
2. Scaffols is based on the rinf and linker removing peripherial substitute
3. Claculate the unique scaffold
4. the size of each scaffold
5. Sort data according to the scafold  group size
6. model fit
7. model evaluation
8. decline in R2
9. calculate max tanimoto similarty of wrost perfomer molecule in scafold spliting
10. corelation between the symeetry and absolute error increases.
In a random split, test molecules may be structurally similar to training molecules and may share the same molecular scaffold. Therefore, the model is often evaluated by interpolation within relatively familiar chemical space. If the intended application is to predict compounds containing new scaffolds, random-split performance can be overly optimistic because it does not adequately test extrapolation to unfamiliar chemical space. A scaffold split provides a more challenging evaluation by preventing exact scaffold overlap between training and test sets.

So Error depends on training chemistry?

Remember, zero shared Murcko scaffolds does not imply low Tanimoto similarity

That reinforces something you learned yesterday:

“Similarity” is not an intrinsic scalar property of two molecules.

It depends on:

$$ \text{representation} + \text{similarity metric}. $$\

versus

Morgan fingerprint Tanimoto similarity.

Applicability domain cannot be reduced to a single similarity threshold.

I have learned Applicability domain cannot be reduced to a single similarity threshold.

Here we have two definitions of structural relatedness:

$$ \text{Murcko scaffold identity} $$
