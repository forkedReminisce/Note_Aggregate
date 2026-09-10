---
draft: true
title: 

params: 
    desc: 
    author: Andrew Nguyen 
---



Digital images are really just a 2D array of values or channels. 

<!-- filters can be added then applied or applied separated then added -->
When an image has noise, replace the pixels with a weighted average of its neighbors. This is done with a linear filter—a matrix smaller than the image of weights. When the filter and image are multiplied and summed, a pixel is generated. However, the produced image will have less pixels than the image. This is under valid filtering, where we don't allow the filter to go beyond the edge. Same filtering allows the filter to go only beyond enough to not lose any pixels, while full goes beyond that. When the filter goes beyond, the ambiguous cells of the image can be padded with 0s. This is would produce a black border. More commonly, symmetric padding copies the values on the edge. Circular, or wrap, copies the values on the opposite edge.

Convolution flips the filter across the x=y line through the center. Not doing so performs cross-correlation. Convolution is commutative and associative. It can be distributed. Cross-correlation is not commutative or associative. The identity element (think identity matrix) has a single `1` at the center with every other pixel being `0`. This produces the same image with an extended black border.

<!-- smooth == blurry -->
The box filter (equal weights) smoothens the image, but everything looks boxy. Ideally, higher weights would be reserved for being closer to the center pixel. The Gaussian filters are a good choice. Larger standard deviation creates smoother images. The size of the filter should be \(6\sigma\), because a too small filter cuts off too abruptly. 

Applying a Gaussian filter is \(O(N^2 M^2)\). Separability improves this by separating the vertical filter (through the center) from the horizontal then applying each separately.

Gaussian is also not a universal solution because outlier signals can distort neighbors. This is because it uses mean as average. Non-linear filters use median. 

Filtering is good for determining what objects are in the scene, even if they're at a different angle. It's also good for stitching images together. 