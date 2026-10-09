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

<!-- optimization -->
One way to find the minimum function evaluation is grid search—try every coordinate. Obviously this will blow up with many dimensions, so random search executes for a number of iterations with random samples. But since there can be so many dimensions, failure is more likely than RANSAC.

<!-- L(w) is loss of w. gradient of L(w) -->
<!-- stochastic and minibatch -->
Since gradients are equivalent to derivatives, it's possible to move in the opposite direction of the gradient. The step size (or learning rate) is small, and each step happens for a predefined number of iterations. Too large a learning rate leads to divergence, and too little falls short. There's also a fear of oscillation. Either exponentially decrease the learning rate per couple steps or average past gradients (momentum), exponentially decaying the influence of earlier gradients. 

When there are multiple minima, gradient descent finds just one. Which one depends on initialization. However, many functions are convex, meaning there is only one global minima.

Optimize \(w\) with minibatch stochastic gradient descent (SGD) to maximize training accuracy. Optimize \(\lambda\) with grid or random search to maximize validation accuracy.

One way of thinking of derivation is recursively abstracting terms, taking the partial derivative of each, and multiply them together. If each "building block" were to have a forward and backward function, the forward would be evaluating \(f(x)\) and the backward would be \(af`(x)\). This can be used for gradient descent by going forward and backward (at a block, the forward input serves the \(x\) in the backward \(f`(x)\)). The result is multiplied by the step size and added to the current value \(w\). At the end, the value is close to minimizing the loss function (i.e., found the value that makes the loss function 0)ate leads to divergence, and too little falls short. There's also a fear of oscillation. Either exponentially decrease the learning rate per couple steps or average past gradients (momentum), exponentially decaying the influence of earlier gradients. 

When there are multiple minima, gradient descent finds just one. Which one depends on initialization. However, many functions are convex, meaning there is only one global minima.

Optimize \(w\) with minibatch stochastic gradient descent (SGD) to maximize training accuracy. Optimize \(\lambda\) with grid or random search to maximize validation accuracy.

One way of thinking of derivation is recursively abstracting terms, taking the partial derivative of each, and multiply them together. If each "building block" were to have a forward and backward function, the forward would be evaluating \(f(x)\) and the backward would be \(af`(x)\). This can be used for gradient descent by going forward and backward (at a block, the forward input serves the \(x\) in the backward \(f`(x)\)). The result is multiplied by the step size and added to the current value \(w\). At the end, the value is close to minimizing the loss function (i.e., found the value that makes the loss function 0). If two backward inputs converge, the outputs are summed. Not every term needs to be broken up; some derivatives are so well known that it's better to keep some terms together.