---
draft: true
title: 

params: 
    desc: 
    author: Andrew Nguyen 
---



The way a vector unit works is it places the modules in parallel (e.g., a number of adders side by side dedicated to a specific part of the vector). This is known as lanes. This increases data-parallelism and reduces power draw. However, depending on the number of lanes, latency may increase for less lanes. But more lanes takes up valuable space.

The Vector Length Register (VLR) is essentially a constant that sets the number of elements every vector register contains. Instructions then only work on those elements. This reduces power draw. There are instructions for overwriting VLR.

Mask registers contains booleans that designate indexes to operate on. Masked vector operations is also called predicated execution. This is great for loops with a conditional. 

Vector memory operations has three modes:
- Stride 1: row-wise access
- Stride k: column-wise access, width of row must also be specified
- Scatter/gather: row-wise access into/from non-sequential indices, a separate vector specifies the indices and the order

Banked memory has a set of banks that can operate independently. For example, if a request is made to one bank, it's still possible to access other locations as long as they're in another bank. Interleaving will assign addresses to banks round-robin style (i.e., 0 goes to the first bank, 1 goes to the next bank, etc. and loop around if necessary). Stride 1 is fastest since all banks operate in parallel. 



# {{< heading "Vectorization" >}}
Scalar code in loops can be vectorized. Some strategies include: 
- Stripmining: when the vector size is larger than the vector register length
- Loop distribution: if legal, put each statement in a shared loop its own loop

<!-- semantically: a node has a dependence on a node from any iteration -->
To determine whether loop distribution is legal, dependence graphs are employed. Unroll the loop and graphing dependencies per and between iterations then collapse it all into a single graph. This single graph will add a distance to the edge the indicates how much iterations before this dependency emerges from.

By decomposing the graph into strongly connected components, vectorize the components that contain just one statement. However, if this statement has a self-loop, you cannot vectorize. 

Scalar expansion helps with how scalars mess with loop dependence graphs (can create large strongly connect components). Instead, turn the scalar variable into a vector variable. This uses more storage and less locality. 

Reductions help with self-loops. As long as the operation is associative, keep two vector registers: one that holds the results and another that is stripmining. Perform the operation between the two.