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



# {{< heading "Feature Detection" >}}
Applications of filtering include identifying objects in the same scene from different angles and stitching images together. These rely on feature detection, and there are several kinds of features.

If we already know a feature, it's possible to find multiple of another feature. The feature is not necessarily the same as the feature we already know. Take the difference between the feature we know and, individually, two nearest neighbors that might depict the same feature. A ratio between these two differences that's close to 1 implies that the two nearest neighbors are the same feature.


## {{< heading "Edges" >}}
Gradients are essentially partial derivatives of the image. Convoluting the gradient with specific kernels produce \(I_x\) and \(I_y\). These results highlight the edges perpendicular to the x-axis or y-axis respectively. Magnitude can be calculated with \(\sqrt{I_x^2 + I_y^2}). In any case, these edge-detecting kernels include (for producing \(I_x\)):

<!-- vertical version positive to negative, not negative to positive -->
Prewitt: 
\[
    \begin{bmatrix}
    -1 & 0 & 1 \\
    -1 & 0 & 1 \\
    -1 & 0 & 1
    \end{bmatrix}
\]

Sobel:
\[
    \begin{bmatrix}
    -1 & 0 & 1 \\
    -2 & 0 & 2 \\
    -1 & 0 & 1
    \end{bmatrix}
\]

Noise in the image can muddle the gradient, so smoothen the image with a Gaussian kernel. However, the tradeoff is blur, making it hard to localize the exact location. Deriving the kernel then convoluting the image produces an equal result, but the Gaussian derivative is a lot like the Sobel filter, so just use Sobel as it's more efficient.

The second moment matrix \(M\) is defined as:
\[
    \begin{bmatrix}
    \sum_{x, y} I_x^2 & \sum_{x, y} I_x I_y \\
    \sum_{x, y} I_x I_y & \sum_{x, y} I_y^2
    \end{bmatrix}
\]

Since finding the eigenvectors and eigenvalues of \(M\) takes too long, we approximate with a response function \(R = \mathrm{det}(M) - \alpha \mathrm{trace}(M)^2). 
- \(R \approx 0\): flat
- \(R \ll 0\): edge
- \(R \gg 0\): corner


## {{< heading "Blobs" >}}
<!-- scales are dictated by the coefficient multiplied with the standard deviation -->
The Laplacian of Gaussian (LoG) is the sum of the double gradients of Gaussian. LoG has ideal scales for each blob size. Therefore, convolve the image with several LoGs. LoG can be approximated with the Difference of Gaussians: \(G(x, y, k\sigma) - G(x, y, \sigma)\). A blob is a local maxima in scale-space—a pixel on a particular scale result that's larger than every other scale result at that same \((x, y)\) coordinate. 