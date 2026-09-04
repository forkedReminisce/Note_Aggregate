---
draft: true
title: 

params: 
    desc: 
    author: Andrew Nguyen 
---



Camera obscura is how cameras can narrow the amount of light rays it captures. In essence, it is a barrier with a pinhole. The captured image is composed of projections. Due to the nature of this dimensionality reduction machine, a point of the scene may be occluded by another point, meaning its projection doesn't appear. Another consequence is that size and distance may become distorted.

{{< subtext >}}
    Orthographic cameras maintain size by capturing every light ray straight forward instead of at an angle. 
{{< /subtext >}}

A point in the scene can be described as \((x, y, z)\), where \(z\) is the focal length—distance to the vanishing point (barrier). This point projects onto the image at \((\frac{fx}{z}, \frac{fy}{z})\), where \(f\) is the distance between the vanishing point and image.

<!-- not relevant until final part of the course -->
The 3D point can be thought of as a homogeneous coordinate. To collapse a dimension, just divide said dimension from every other dimension. To add a dimension, concatenate a \(1\) to the vector. However, equality (=) is harder to come by, so use equivalency (≡) instead.

Lines can also be represented as homogeneous coordinates: one vector is the coefficients and another vector is the variables (1 for no variable). To find the intersection point between two lines, take the cross product of their coefficient vectors. To find the line between two points, take the cross product of the two points. 

{{< subtext >}}
    When finding the intersection between two lines, if the extra coordinate is 0, the lines intersect at the point at infinity. 
{{< /subtext >}}

<!-- continue -->
Ideally, the size of the pinhole is infinitely small but not zero. In other words, a point in the scene only generates one projection, not multiple—image becomes blurry. But not only is this unrealistic, but not enough photons would be captured to make the photo bright. Additionally, too small apertures results in defraction. Aperture is inversely correlated with depth of field.

Lens can make up for a larger pinhole by coalescing the light rays from a single point of the scene, for every point of the scene (thin lens model). All rays parallel to the optical axis (imagine a laser shooting from the camera) converge at the focal point (the barrier). Only a range of distance from the lens are in focus, but other distances will not converge and project a "circle of confusion."

<!-- wtf -->
object distance (object to lens) D, plane distance (lens to image) D\`, focal length (lens to focal point) f, actual height of object y, projected height of object y\`. theres a set of similar triangle between f, y, and y\`. theres another between y\`, D, y, and D\`. by equating the relations, we ultimately get 1/D + 1/D\` = 1/f. we typically can't change f.

<!-- f is actually D`??? -->
<!-- can simplify this to focal length is inversely proportional with FOV -->
<!-- smaller FOV and long distance removes depth -->
Field of view is determined by the size of the photosensitive material. FOV small phi = tan^-1(d/2f), where d is the size of the photosensitive material. For the most part, d is not a variable we can change. tan^-1 is a monotonically increasing function. Distance (viewpoint) can exacerbate perspective (or lack thereof) when paired with FOV. 

Our eyes are comparable to cameras. Our pupils is the aperture, which is controlled by the iris. Our eyes do have lenses. The retina is basically the film. We have two types of photo receptors: cones and rods. Rods are more sensitive and they help with vision in darkness.

Digital images are typically a 2D array of rgb values. Origin is top left, rows are first index.

When an image has noise, replace the pixels with a weighted average of its neighbors. This can be done with a linear filter—a matrix that's smaller than the image's with fractions. When the filter and image are multiplied and summed, a pixel is generated. The filter then moves one pixel over. However, the produced image will have less pixels than the image. This is because of valid filtering, where we don't allow the filter to go beyond the edge. Same filtering allows the filter to go just beyond enough to not lose any pixels, while full goes beyond that as long as the filter is touching the image. When the filter goes beyond, the ambiguous cells of the image should be padded with 0s. This is would typically produce a black border. Alternatively, symmetric padding copies the values on the edge, whereas circular/fold copies the values on the opposite edge.