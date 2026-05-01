---
draft: false
title: Vector Registers

params: 
    desc: Vector registers require another set of hardware. But when they are supported, programs can be vectorized in such a way that improves performance.
    author: Andrew Nguyen 
---



# {{< heading "Hardware" >}}
Lanes are how vector registers can be operated on in parallel. It is a line of modules for each separate item. This increases data-parallelism and reduces power draw. However, too little lanes will increase latency, but more will take up physical space.

The Vector Length Register (VLR) dictates the number of elements every vector register contains. Instructions then work on that number. This reduces power draw. Mask registers are another useful tool, consisting of booleans that designate indexes to operate on. This is great for loops with a patterned conditional. 

Vector memory operations have three modes:
- Stride 1: row-wise access
- Stride k: column-wise access, width of row must also be specified
- Scatter/gather: row-wise access of memory for non-sequential indices of the vector register, a separate vector specifies the indices and the order

Banked memory has a set of banks that can operate independently. For example, if a request is made to one bank, it is still possible to access locations belonging in another bank. Interleaving will assign addresses to banks round-robin style (i.e., 0 goes to the first bank, 1 goes to the next bank, etc. and loop around). When loading, it does it by lines. That means if there needs to be a loop around, another line needs to be loaded regardless of it being stride-1.



# {{< heading "Vectorization" >}}
Scalar code in loops can be vectorized. Some strategies include: 
- Stripmining: if the size is larger than the VLR, loop as many times as necessary
- Loop distribution: if legal, put each statement in the shared loop its own loop

To determine whether loop distribution is legal, *loop dependence graphs* are employed. If nodes represent statements, then a directed edge from one to another is a dependence from this or earlier iterations. By decomposing the graph into strongly connected components, those with only a single statement may be vectorized, but only if it doesn't have a self-loop. 

Since scalars can create large strongly connected components, scalar expansion will turn the variable into a vector that separates its values of each iteration. This uses more storage and less locality, though. To handle self-loops, reductions will exploit associativity (if present) to perform the operation in a round-robin way. 