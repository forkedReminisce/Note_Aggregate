---
draft: false
title: Caches

params: 
    desc: Cache misses are bad for performance. There are a number of ways to improve locality, but blocking allows for a greater level of locality.
    author: Andrew Nguyen 
---



`stackDistance` measures the number of distinct cache lines referenced between two references. This notation makes it easy to define the types of misses; a capacity miss is when `stackDistance` is greater than the cache size, and a conflict miss is otherwise. 

To understand how the miss ratio changes as the problem size increases, it good to think about the situation in two ways. In the **large cache model** the problem size is small relative to cache size, so it only experiences cold misses. The **small cache model** shrinks this gap, adding in capacity misses. Here is where `stackDistance` comes in handy, where if it is constant for two references to the same object, that implies a capacity miss. 

Locality is the key to optimizing cache operations. There are a couple of transformations for maximizing locality:
- Loop permutations: interchange loop order
- Strip-mining: cut the loop into parts (i.e., blocking)
- Loop tiling: combines strip-mining and loop permutations
- Manipulate the data structure (e.g., turn a row-major matrix into column-major)

{{< subtext >}}
    Blocking exploits the fact that the large cache model has no capacity misses by only working with an amount of data that can all fit in the cache one at a time.

    Compilers can make these optimizations.
{{< /subtext >}}



# {{< heading "ATLAS" >}}
It's possible to take blocking a step further by implementing it for multiple levels of the memory hierarchy. ATLAS is a library generator that can do this for registers and L1 cache. Square tiles of size `NB` are stored in *cache* in mini-MMM. *Registers* store nested blocks of size `MU` or `NU` for micro-MMM. There is an addition `KU` optimization parameter to keeping the loop unrolling here from generating capacity misses in the I-cache.

The optimization parameters are not set by the programmer but are rather optimized for. A model approach is way too closed-minded, so *orthogonal line search* is used. It works by optimizing one variable at a time, using dummy values for parameters not yet optimized.  