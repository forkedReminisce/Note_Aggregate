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

Aperture is the size of the pinhole. Ideally, it is infinitely small but not zero. In other words, a point in the scene only generates one projection. Too large and the image becomes blurry. However, that ideal aperture is not only impossible, but not enough photons would be captured and there would be diffraction.

Lens can make up for a larger aperture by coalescing the light rays from a single point of the scene for every point of the scene. All rays parallel to the optical axis (imagine a laser shooting from the camera) pass the focal point. Only a narrow range of distances from the camera actually collapse and get in focus; all other distances will not coalesce and project a "circle of confusion."

Focal length \(f\) is the distance from the lens to focal point. Now imagine a point in the scene emitting two light rays at different angles. The distance from this point to the lens is the object distance \(D\) and the height of the object itself is \(y\). These two light rays hit the lens and then project onto the image. The distance from the lens to the image is the plane distance \(D`\) and the projected height is \(y`\). Two sets of similar triangles can be observed. After some math, we are left with \(\frac{1}{D} + \frac{1}{D`} = \frac{1}{f}).

Field of view is inversely proportional to focal length. Object distance can exacerbate perspective. For example, small FOV and large object distance removes the depth of the scene.

<!-- TODO: probably move to another article -->
Digital images are really just a 2D array of values or channels. 

When an image has noise, replace the pixels with a weighted average of its neighbors. This is done with a linear filter—a matrix smaller than the image of weights. When the filter and image are multiplied and summed, a pixel is generated. However, the produced image will have less pixels than the image. This is under valid filtering, where we don't allow the filter to go beyond the edge. Same filtering allows the filter to go only beyond enough to not lose any pixels, while full goes beyond that. When the filter goes beyond, the ambiguous cells of the image can be padded with 0s. This is would produce a black border. More commonly, symmetric padding copies the values on the edge. Circular, or wrap, copies the values on the opposite edge.