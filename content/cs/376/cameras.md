---
draft: true
title: 

params: 
    desc: 
    author: Andrew Nguyen 
---



<!-- this is stupid -->
Barrier with pinhole to restrict which light rays reaches the photosensitive material. Known as camera obscura. Dimensionality reduction machine. People used to think our eyes emitted light, but in fact, it collects reflected light. 

<!-- is f distance from vanishing point to image -->
A projection is the point on the image that a point on the object is captured by. A point on the object that doesn't appear in the image because it is covered is occluded by another point. Projection is undefined for points on the barrier. A point at (x, y, z) projects at (fx/z, fy/z), where z is focal length—length from object to vanishing point. However, size and distance may get improperly captured because a 2D line is not a 3D line.

<!-- homogeneous coordinate of a line only contains the coefficient??? -->
<!-- not relevant until final part of the course -->
A homogeneous coordinate adds another dimension with a value of 1. For example, (x, y) becomes (x, y, 1). To convert it back, divide the value of the new dimension across all dimensions. To be safe, use === (equivalent) because of lambda. A line can be represented with an inner product of homogeneous coordinate. Finding the intersection between the two lines is taking the cross product (if z is 0, they intersect at the point at infinity). Finding the line between two points requires taking the cross product of the two points. Projection uses matrix multiplication.

Orthographic cameras maintain size by capturing light rays straight forward instead of at an angle. 