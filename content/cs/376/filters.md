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