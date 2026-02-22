---
draft: false
title: Caches

params: 
    desc: Caches help bridge the speed gap between registers and memory. 
    author: Andrew Nguyen 
---



To get the illusion of large fast storage, have small fast memory storage by lots of slow storage. This is the essence of the storage hierarchy. 

This hierarchy goes hand in hand with the **Principle of Locality**. Understanding and exploiting locality improves performance by reducing the number of times slow storage needs to be accessed. There are two types of locality:
- Temporal: recently-referenced items are likely to be referenced again soon
- Spatial: nearby items will likely get referenced 

Caches help with spatial locality. It serves as a staging area between fast registers and slow memory. When a memory address is referenced, that number is partitioned in bits in order to find out whether the memory block is in the cache. To best understand this process, consider the steps:
1. Index with the index (middle) bits into every *cache way* for a *cache set*
2. In the indexed cache set, look for the *cache line* with the matching tag (most significant) bits
3. Use the offset (least significant) bits to get the desired data from the cache line

All these cache configuration parameters introduces the *ABC model of cache structure*. 
- Cache associativity \(A\): number of cache ways
- Cache line size \(B\)
- Cache capacity \(C\)
- Number of cache sets per cache way \(\mathbb{S}\)

\[
    C = \mathbb{S} \times A \times B
\]

This helps formalize the main cache geometries. 
- Direct-mapped: \(A = 1\)
- Fully-associative: \(\mathbb{S} = 1\)
- Set-associative otherwise



# {{< heading "Cache Misses" >}}
When no cache line has the matching tag, it needs to be retrieved from memory. Each cache line also has a valid bit. Preferably, the memory block would be placed in a cache line with a valid bit of `0`. Otherwise, a memory block needs to get evicted. In any case, a cache miss can be categorized by the *3C model*:
- Compulsory: first access to the memory block
- Capacity: the non-compulsory miss is a miss in an equivalent fully-associative cache
- Conflict: the non-compulsory miss is a hit in an equivalent fully-associative cache

Cache misses take a very long time to handle, which is why locality matters a lot. To assess the spatial locality of a program, analyze the *access stride* of each memory structure (e.g., array). Access stride measures the amount of elements being skipped over each access.