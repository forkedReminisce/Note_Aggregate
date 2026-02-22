---
draft: true
title: 

params: 
    desc: 
    author: Andrew Nguyen 
---



<!-- TODO: address this -->
<!-- if stackDistance is constant, the variable likely sees capacity misses -->
stackDistance (r_1, r_2) == number of distinct cache lines referenced between r_1 and r_2. In this model, we will ignore conflict misses and only retain cold/compulsory and capacity misses. A capacity miss is when stackDistance is greater than the size of the cache. A conflict miss is when stackDistance is not larger. Large cache model assumes only cold misses, and small cache model introduces the capacity misses.