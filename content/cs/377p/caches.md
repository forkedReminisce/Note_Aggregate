---
draft: true
title: 

params: 
    desc: 
    author: Andrew Nguyen 
---



To get the illusion of large fast memory, have small fast memory supported by lots of slow memory. This is the essence of the memory hierarchy. 

With set-associative caches, the index bits does not signify which cache set. Instead, the index must be checked in every cache set. If the tag bit is not found, you must go into memory to retrieve the block.  

Although the L1 and L2 caches are hardware managed, it's possible for software to trick it into doing what it wants. 

<!-- if stackDistance is constant, the variable likely sees capacity misses -->
stackDistance (r_1, r_2) == number of distinct cache lines referenced between r_1 and r_2. In this model, we will ignore conflict misses and only retain cold/compulsory and capacity misses. A capacity miss is when stackDistance is greater than the size of the cache. A conflict miss is when stackDistance is not larger. Large cache model assumes only cold misses, and small cache model introduces the capacity misses.