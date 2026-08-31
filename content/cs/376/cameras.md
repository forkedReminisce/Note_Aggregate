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