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

Augmentation is the strategy of applying transformations to images of the training set and include them in the training set. The model should still be able to properly classify the image. 



# {{< heading "Optimization" >}}
It is desired to find the \(w\) that would minimize the loss function \(L(w)\). Grid search tries every coordinate, and random search iterates a set number of times on random samples and picks the one with the smallest evaluation. With many dimensions, grid search takes forever and random search is more likely to fail.

Since gradients are like derivatives, maybe move against the gradient. This is the idea of gradient descent. If the step size (or learning rate) is too small, the algorithm will finish before finding the \(w\). Too large will be divergence. There is also the concern of oscillation. Therefore, exponentially decay step size and, for each step, incorporate the average of previous steps (in decreasing contribution the further back in the past it is). 

{{< subtext >}}
    Averaging previous steps is known as momentum.

    When there are multiple minima, gradient descent finds just one. Which one depends on initialization. However, many functions are convex, meaning there is only one global minima anyway.
{{< /subtext >}}

<!-- computational graph -->
<!-- step size is negative (traveling against the gradient) -->
It's possible to just derive \(L(w)\) instead. If each term is recursively decomposed and turned into a bidirectional function, forward would be \(x \to f(x)\) and backward \(y \to yf`(x)\). For optimization, go forward with \(w_i\) then start backward with \(1\). To get \(w_{i+1}\), subtract the final backward result times a step size from \(w_i\). When there are multiple forward inputs, the backward will send \(y\) times the partial derivative with respect to that input. When there are multiple backward inputs, they sum. 

{{< subtext >}}
    Sometimes, it's better not to completely decompose part of an expression. It might be able to simplify into one function instead of many.
{{< /subtext >}}



# {{< heading "Parameters" >}}
<!-- optimization is applied to the parameters values -->
The number of parameters a fully connected network (FC) has is the sum of \(\mathrm{inputs per weight} \times \mathrm{weights}\) of each layer *plus* the total amount of biases—number of neurons after the first layer. This gets nasty really quickly, so a convolutional neural network (CNN) shrinks the space by averaging. The convolution operation here is analogous to cross-correlation in image processing.

<!-- or is parameters filter area * number of filters * number of filters + number of filters -->
Each neuron produces a separate output that is stacked in the third dimension. Assuming a square input, the side length of one of these outputs would be \(\frac{\mathrm{input side length} - \mathrm{filter side length}}{\mathrm{stride}} + 1\). The number of parameters is \(\mathrm{filter side length} \times \mathrm{filter side length} \times \mathrm{input channels} \times \mathrm{neurons} + \mathrm{neurons}\). The receptive field is the *recursive* area of input cells that one output cell covers. Therefore, chaining convolutions with smaller filters to produce a receptive field equal to a bigger filter would have less parameters than just using that big filter.

{{< subtext >}}
    Stride is how much the filter slides over by.
{{< /subtext >}}

A pool is another type of layer that pairs really well with convolution. Functionally, it is just like convolution, but it has no neurons. Instead, it operates one channel at a time, and the number of output channels is equal to the input channels. Pools have zero parameters.

The goal is to create a CNN that will turn an input into a \(1 \times 1 \times F\) vector for various purposes such as classification. 

Instead of optimizing the parameters from scratch, fine-tuning instead copies layers from an existing network. This can be combined with transfer learning—initializing weights from a pre-trained model. This is ideal for a smaller training set; larger warrants starting from scratch.

<!-- such models require context -->
Some applications require that the output image be the same shape as the input image instead of some long tube. This can be achieved with the encoder-decoder—downsample the image to a point then upsample. Downsampling can be achieved with convolutions and pooling.

<!-- encoder is downsample. decoder is upsample -->
The encoder-decoder generates image context on each input pixel. Upsampling makes use of unpooling and transpose convolution. One unpooling operation is nearest neighbor. The transition point often should not be downsampled to a vector.

{{< subtext >}}
    The encoder and decoder need not be symmetric.
{{< /subtext >}}

In any case, these upscaling strategies produces poor fidelity results. This is improved with U-Net by copying the last downscaling layer of each spatial resolution and concatenating it to the start of the first layer of the same spatial resolution (see matrix stacking along the channel axis). This copy contains high frequency information that helps provide additional context because all the convolutions inflated all the values. The next layers have to figure out how to fuse the two features.

This was semantic segmentation—classify each pixel of an image. Object detection goes a step further by distinguishing between each instance of the class. Remember, it is not possible to have varying size outputs.

<!-- what is a true detection -->
<!-- i think recall is number of true detections vs number of objects (ground truth)  -->
Jaccard coefficient (or IoU) is the metric for determining the accuracy of the bounding box. It is the intersection divided by the union of the ground-truth bounding box and the predicted bounding box. This measure accuracy. With a new set of true detections, precision is this number over the total number detections. Recall is this number over ground truth positives.

<!-- the region needs to be warped in order to be used a common neural network -->
Histogram of Gradient (HOG) trains a gradient template for a particular object (e.g., pedestrian). When convolved with a feature map, a high response indicates an instance of that object. However, sliding window added with using multiple templates (for each object) is very inefficient. Instead, a proposal mechanism returns regions of interest that narrows the search area and template scales before passing through the CNN.

Faster R-CNN produces a feature map from the image through the CNN. This feature map is used to create region proposals ==of the feature map==. Each bounding box is warped and thrown through a small neural network to classify each box. If there were no region proposals, YOLO just predicts at each location. 