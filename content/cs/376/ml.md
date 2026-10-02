---
draft: true
title: 

params: 
    desc: 
    author: Andrew Nguyen 
---



<!-- input vector aka feature vector -->
Machine learning is about transforming an input vector into a more desirable output vector using a transition function. The setting could be unsupervised (just data) or supervised (data and labels). Likewise, the output could be discrete or continuous. Some applications based on this are:
- Supervised & Discrete: classification or categorization
- Supervised & Continuous: regression—inferences
- Unsupervised & Discrete: clustering—relate a set of images and create categories
- Unsupervised & Continuous: dimensionality reduction—map dimensions to details of the image

<!-- finding a linear model is known as linear regression -->
<!-- x^TX is rank deficient, meaning that there is no inverse (underdetermined) -->
Model fitting may also be known as training or learning. In any case, the goal is to find the model that minimizes error. Inference or testing then uses the model given any input. However, separated testing is important or else there will be overfitting—fits too precisely to the data. Additionally, have a separate data set for testing in addition to the training set. That way, the ML model doesn't see the data and can be evaluated in a truthful environment. 

Memorization is a terrible strategy of classification because any slight deviation will throw off the model. The real easiest form computes the distance between the training set and the new input image. The image gets categorized in the same category as the nearest neighbor. K-nearest neighbors; figure out K from the validation set. 

<!-- there can be regularization -->
Alternatively, a linear model may be used for classification. Each category has a weight vector and the input feature vector is multiplied with it. Whichever product is largest, the image gets classified as. These weight vectors can be transposed and stacked vertically into a matrix \(W\). The resultant scores can be converted into probabilities (softmax).