---
draft: true
title: 

params: 
    desc: 
    author: Andrew Nguyen 
---



Digital images are really just a 2D array of values or channels. 

<!-- filters can be added then applied or applied separately then added -->
When an image has noise, replace the pixels with a weighted average of its neighbors. This is done with a linear filter—a matrix smaller than the image of weights. When the filter and image are multiplied and summed, a pixel is generated. However, the produced image will have less pixels than the image. This is under valid filtering, where we don't allow the filter to go beyond the edge. Same filtering allows the filter to go only beyond enough to not lose any pixels, while full goes beyond that. When the filter goes beyond, the ambiguous cells of the image can be padded with 0s. This is would produce a black border. More commonly, symmetric padding copies the values on the edge. Circular, or wrap, copies the values on the opposite edge.

The technical term for filtering is cross-correlation. Convolution adds onto cross-correlation by flipping the filter across the \(x = y\) line. Convolution is commutative and associative, unlike cross-correlation, and can be distributed out. Convolution also has an identity element (think identity matrix), consisting of a single `1` at the center with every other pixel being `0`. This produces the same image padded with black.

A filter with equal weights is called the box filter. It smoothens the image but creates a boxy effect. Ideally, higher weights would be given closer to the center. This is where Gaussian filters come in. It is a family of filters for each standard deviation \(\sigma\). The larger the standard deviation, the smoother the image *without the boxy effect*. A Gaussian filter should be size \(6\sigma\). Applying a Gaussian filter of size \(M \times M\) on an image \(N \times N\) is \(O(N^2 M^2)\). Separability improves this by extracting one vertical slice through the center of the filter and a horizontal slice then applying each separately.

Gaussian filters are not always the best because outlier pixels can distort its neighbors. This is because Gaussian filters use mean. Non-linear filters use median. 

Filtering is good for determining what objects are in the scene, even if they're at a different angle. It's also good for stitching images together. 

Gradients are essentially partial derivatives. Like the original function, gradients have dimensions. Looking at these can help with edge detection. Noise can muddle the derivative, so smoothing via convolution can help. However, the tradeoff is a blur effect, making it hard to localize the exact location. Deriving the filter then convoluting the image also works, but is not as efficient as the Sobel filter.

{{< subtext >}}
    The filter to use to detect either horizontal or vertical edges should be perpendicular to that edge.
{{< /subtext >}}

There is flat, edge, and corner regions. When viewing only a small part of an image, moving the viewing window around the full image helps identify the type of region. Flat means the color distribution does not change. 

<!-- M is a 2x2 matrix Ix^2 IxIy IxIy Iy^2. all terms are summations -->
<!-- R is response function -->
Detecting a corner uses the formula \(E(u, v) = \sum_{(x, y)} (I[x + u, y + v] - I[x, y])^2\). Edge and flat regions is all `0`. However, this operation is expensive over time. The second moment matrix \(M\) greatly simplifies things, but it requires finding the image gradients along both dimensions. By evaluating the matrix, only the horizontal values matter. If they're all high, it is likely a corner. To detect this, \(R = det(M) - \alpha sum(M)^2\). A corner will have a response function \(R\) significantly greater than \(0\). Close to \(0\) is flat, significantly less is edge.

We want our detectors to be invariant to some things. This means the corners stay at the same place with certain transformations. But some transformations require shifting the corners, which means the detector should also be equivariant to other things. For example, convolution is equivariant. Image scaling is not equivariant, but downsampling and checking for corners each time works. 

{{< subtext >}}
    Keeping the image size but doubling the filter is an option, but more expensive.
{{< /subtext >}}

<!-- scales are dictated by the coefficient multiplied with the standard deviation -->
Blobs are another good option for detecting features. The Laplacian of Gaussian (LoG) is the sum of the double gradients of Gaussian. Like Gaussian, there is an ideal size of the filter for each blob size. The characteristic scale creates the maximum response. Therefore, convolved with a number of scales. Local maxima is when a pixel is larger than its scale-space neighbors (one pixel circle on this scale, across all scales). Difference of Gaussian is an approximation of LoG, which is a difference of Gaussian scales. A lot of the effort takes place in the first and second scales, but it substantially gets faster with later scales.

<!-- does the algorithm rotate the image or was it provided a rotated image -->
SIFT descriptors finds the gradient of equally sized blobs of part of a rotated and scaled image that has been normalized. Computing a histogram for each blob and concatenating it all. This mitigates illumination effects. It's very complicated nonetheless.

The trick of the second nearest neighbor is the ratio between the feature and its difference between one nearest neighbor and difference with another nearest neighbor. Closer to 1 is good.