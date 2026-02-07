---
draft: true
title: 

params: 
    desc: 
    author: Andrew Nguyen 
---



Adjacency matrices explicitly note that two nodes are not connected. For large graphs, this is bad for space and increases processing overhead.

There are three alternate ways to represent a graph. 
- Coordinate storage: 2d array, where each column corresponds to an edge. the first row is the source node, second is destination, and third is the edge weight.
- Compressed sparse row (CSR): take coordinate storage and coalesce the source nodes into a new array. this new array consists of pointers to the rest of the information. from one source node's pointer to the next source node's pointer all belong to that source node.
<!-- TODO: -->
- Compressed sparse column (CSC): ???
- Adjacency list: for deletion, easier to just mark a node as deleted then handle it later



# {{< heading "Algorithms" >}}
Instead of giving pseudocode, an algorithm will be defined with **operator formulation**. An algorithm is made up of an *operator* and *schedule*. Operator is what is done at the selected node. Schedule is which active node gets selected next. This makes comparing different algorithms simple and possible to pick and choose (schedules) from different algorithms to create the best algorithm. 

{{< subtext >}}
    A schedule might seem bad, but it might better support parallelism.
{{< /subtext >}}

There is a notion of active nodes. Active nodes are nodes that are available to process. Topology-driven algorithms execute in rounds, and in each round, all nodes are active. Data-driven algorithms will have some nodes active, and performing the operation on one may create more active nodes.