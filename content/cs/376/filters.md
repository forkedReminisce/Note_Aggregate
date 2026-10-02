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

Noise in the image can muddle the gradient, so smoothen the image with a Gaussian kernel. However, the tradeoff is blur, making it hard to localize the exact location. Deriving the Gaussian kernel removes noise without blurring. Since the Gaussian derivative is a lot like the Sobel filter, just use Sobel as it's more efficient.

<!-- harris -->
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


# {{< heading "Model Fitting">}}
RANSAC tries to find the best fit model through many trials. 
1. Create a data subset
2. Run fit algorithm (e.g., least-squares) on subset 
3. Measure number of inlines—points below error threshold
4. If number of inlines is highest so far, record the model as "best"
5. Repeat for number of trials
6. Rerun fit algorithm on inlines

{{< subtext >}}
    Subset size should be the minimum according to model (e.g., lines are size 2). This maximizes the chances of selecting only inlines.
{{< /subtext >}}

RANSAC does have a chance of failing, but that is laughably unlikely with reasonable parameters. However, too many outliers substantially increases the likelihood of failure.



# {{< heading "Transformations" >}}
- Scaling: multiply each \((x, y)\) by a scalar from the scaling matrix \(S\)
- 2D rotation: 
\[
    R_\theta = \begin{bmatrix} 
        \cos(\theta) & -\sin(\theta) \\
        \sin(\theta) & \cos(\theta)
    \end{bmatrix}
\]
- Identity
- Shear: 
\[
    \begin{bmatrix} 
        1 & \mathrm{sh}_x \\
        \mathrm{sh}_y & 1
    \end{bmatrix}
\]
- Mirror: identity but make some \(1\)s negative as necessary
- Affine: linear transformation with translation
- Perspective (or homography): with homogeneous coordinates, the bottom row of the transformation matrix is not \([0 \hspace{1mu} 0 \hspace{1mu} 1]\)

{{< subtext >}}
    Uniform scaling is when each dimension is multiplied by the same scalar.

    Some transformation matrices may need to use homogeneous coordinates to stay linear.

    Transformation matrices can be combined via matrix multiplication. This is known as matrix composition. Remember, matrix multiplication is not commutative.
{{< /subtext >}}

<!-- far away scenes can be treated as a plane because the depth between objects is relatively large compared to the distance between the camera and scene. can use homography to accomplish this -->
Homographies are particularly powerful because they can relate two images of arbitrary views. 

A point on an image can be related to a point on another image by multiplying the vector by a matrix \(M\) and adding a translation \(t\)—an affine transformation. The objective function is the squared distance with the actual new coordinate. Express this as only a matrix multiplication. if the data matrix has too many entries, consider using a fit algorithm.

<!-- homography is with homogenous coordinates -->
Solving for a homography requires a data matrix of nine columns:
\[
    0^T & -p_i^T & y`_1p_i^T \\
    p_i^T & 0^T & -x`_ip_i^T
\]

The point matrix is multiplied by the homography vector. \(\mathrm{argmin} ||Ah||^2\), where \(h\) is a row of \(H\) and is non-zero. The eigenvector of \(A^TA\) with the smallest eigenvalue will help with non-zero. It is also an algebraic error, but a geometric error is desired and it is really ugly. It's possible to use RANSAC to find the homography.

Forward warping maps the original pixel to a location on the transformed image. Sometimes, an original pixel cannot map to an exact new location. Splatting will give the value to the neighboring pixels. Inverse warping maps from the new image to the old image. Similarly, it may need to pull values from neighboring pixels of the old image.

Mosaicing blends multiple images together. There might be regions where only one image contributes, and there might be regions where multiple contribute. 