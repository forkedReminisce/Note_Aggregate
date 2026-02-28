---
draft: true
title: 

params: 
    desc: 
    author: Andrew Nguyen 
---



<!-- TODO: address this -->
<!-- if stackDistance is constant, the variable likely sees capacity misses -->
<!-- stackDistance is useful for determining the type of misses. typically comparing references to the same object -->
stackDistance (r_1, r_2) == number of distinct cache lines referenced between r_1 and r_2. In this model, we will ignore conflict misses and only retain cold/compulsory and capacity misses. A capacity miss is when stackDistance is greater than the size of the cache. A conflict miss is when stackDistance is not larger. 

<!-- large cache model: lots of reuse -->
Large cache model is when there are only cold misses, and the transition to small cache model happens when capacity misses start occurring. When capacity misses start happening depends on the cache replacement policy. Optimal replacement is knowing the future to evict an object that won't be used again. In reality, LRU is used.

There are a couple of transformations that improve locality:
- Loop permutations: interchange loop order
- Strip-mining: cut loop into parts
- Loop tiling: combines strip-mining and loop permutations
- Manipulate the data structure (e.g., turn a row-major matrix into column-major)

{{< subtext >}}
    Compilers can make these optimizations.
{{< /subtext >}}



# {{< heading "ATLAS" >}}
There must be blocking in each level of memory hierarchy (e.g., registers and L1 cache). ATLAS eases this process by serving as a library generator for MMM and other BLAS with blocking for registers and L1 cache. The way it works is that it completes a template ready to be compiled. 

The cache-level clocking only works with square tiles. This part is known as a mini-MMM. The size of a tile is specified by the optimization parameter `NB`. Inside each tile is register-level blocking (micro-MMM). The height of the window in \(A\) is determined by parameter `MU`, whereas the width of the window in \(B\) is `NU`. `KU` helps with loop unrolling by making sure the I-cache does not evict looped instructions, leading to cache misses. 

<!-- not really important thanks to OOO execution -->
Scheduling is also optimized by alternating load operations with computations, effectively masking that overhead with useful work. 

Optimizing the parameters can use something known as *orthogonal line search*. This optimizes one variable at a time, using dummy values for parameters not yet optimized. This doesn't always work, but it is frequently close. Search is also robust, whereas smarter strategies suffers from a lack of empirical evidence. 