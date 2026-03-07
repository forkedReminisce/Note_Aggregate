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



# {{< heading "Shared-memory" >}}
Every processor has a cache of their own. Caches must be transparent, meaning the result should be the same even without the presence of caches. However, shared-memory programs jeopardize this property. Thus, cache coherence is necessary.

This problem is solved with **bus snooping**. The shared bus connect cores with each other and shared memory. Every cache in core snoops the bus. If a cache writes, it performs a write-invalidate: before writing, it sends a bus transaction that the cache is now invalid, and every other core notices. A write-broadcast provides the new value to every core. 

{{< subtext >}}
    Remember, bus transaction are expensive.
{{< /subtext >}}

*MESI Protocol* implements write-invalidate. Cache lines in each core's cache have four states: 
- Modified: only core with the valid cache line
- Exclusive: same as main memory and only one core has it cached 
- Shared: same as main memory but copies may exist in other caches
- Invalid: not in cache or out of date

<!-- TODO: write-back; CACHES DO THE SNOOPING!!! -->
On a local read miss, the core will make a bus transaction. If other caches have the cache line, they will copy the value onto the bus, prematuring ending memory access. If the cache line is modified, the original core must write back to main memory in addition to giving the value to the other core, and it can set the state to shared.

With multi-level caches, if the levels are inclusive (e.g., all of L1 is present in L2), only the lowest level cache needs to snoop. However, the capacity is less. Having L1 snoop directly adds latency to cache accesses in general. 

However, snoopy schemes do not scale with the number of cores. Therefore, directory-based schemes eliminate the bus. Censier and Feautrier designed a scheme where each block in main memory has number-of-cores presence bits and one dirty bit. Similarly, each block in cache has a valid and dirty bit.

To conclude, another C miss is added in the form of coherence misses. This is because a cache line can become invalid, resulting in a cache miss when it is referenced again. The scenarios this tends to pop up are:
<!-- ping-ponging -->
- True-sharing: multiple cores are reading and writing to the same memory address
- False-sharing: multiple cores are reading and writing to what happens to be the same cache line