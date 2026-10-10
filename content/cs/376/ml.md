---
draft: true
title: 

params: 
    desc: 
    author: Andrew Nguyen 
---



The input vector (A.K.A. feature vector or data point) is transformed into a label—the output vector. Labels could be given (supervised) or not (unsupervised). Likewise, the output could be discrete or continuous. Some applications based on this are:
- Supervised & Discrete: classification or categorization
- Supervised & Continuous: regression—infer attributes of the depicted
- Unsupervised & Discrete: clustering—relate a set of images and create categories
- Unsupervised & Continuous: dimensionality reduction—map dimensions to details of the image

Model fitting may also be known as training or learning. Inference or testing then "deploys" the model. There is the notion of a mutually exclusive training set and test set to best evaluate the model.

<!-- features are columns -->
A linear regression is a method of training when given any number of variables (features). The model is defined as:

\[
    w* = (X^T X)^{-1} X^T y
\]

{{< subtext >}}
    There is always an extra feature for intercept and bias.
{{< /subtext >}}

Sometimes, \(X^T X\) is rank deficient, meaning that the inverse cannot be taken and, more importantly, underdetermined—infinite solutions. Regularized fixed squares solves this by preferring some solutions over others.



# {{< heading "Classification" >}}
Memorization is a terrible strategy of classification because any slight deviation will throw off the model. The real easiest form computes the distance between the training set and the new input image. The image gets categorized in the same category as the nearest neighbor. 

{{< subtext >}}
    K-Nearest Neighbor is a better algorithm that places the new image in the category that appears the most amongst its K neighbors. An additional validation set can be used to figure out the value of K.
{{< /subtext >}}

Alternatively, a linear model may be used for classification. Each category has a weight vector and the input vector is multiplied with it. Whichever product is largest, the image gets classified as. These weight vectors can be transposed and stacked vertically into a matrix \(W\). The resultant scores can be converted into probabilities with softmax.

{{< subtext >}}
    Regularization can be applied.
{{< /subtext >}}



# {{< heading "Optimization" >}}
It is desired to find the \(w\) that would minimize the loss function \(L(w)\). Grid search tries every coordinate, and random search iterates a set number of times on random samples and picks the one with the smallest evaluation. With many dimensions, grid search takes forever and random search is more likely to fail.

Since gradients are like derivatives, maybe move against the gradient. This is the idea of gradient descent. If the step size (or learning rate) is too small, the algorithm will finish before finding the \(w\). Too large will be divergence. There is also the concern of oscillation. Therefore, exponentially decay step size and, for each step, incorporate the average of previous steps (in decreasing contribution the further back in the past it is). 

{{< subtext >}}
    Averaging previous steps is known as momentum.

    When there are multiple minima, gradient descent finds just one. Which one depends on initialization. However, many functions are convex, meaning there is only one global minima anyway.
{{< /subtext >}}

It's possible to just derive \(L(w)\) instead. If each term is recursively decomposed and turned into a bidirectional function, forward would be \(x \to f(x)\) and backward \(y \to yf`(x)\). For optimization, go forward with \(w_i\) then start backward with \(1\). To get \(w_{i+1}\), subtract the final backward result times a step size from \(w_i\). When there are multiple forward inputs, the backward will send \(y\) times the partial derivative with respect to that input. When there are multiple backward inputs, they sum. 

{{< subtext >}}
    Sometimes, it's better not to completely decompose part of an expression. It might be able to simplify into one function instead of many.
{{< /subtext >}}