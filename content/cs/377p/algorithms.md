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



# {{< heading "Machine Learning" >}}
Many ML algorithm are sparse graph algorithms. 

Page rank is related to a web search, and the webpages should be returned in some order. There's an offline part with crawlers, which adds an index from keywords to webpages. The online part will use the index to find pages that contain the most keywords of the search. There are multiple approaches to ranking pages:
- Manual ranking
- Word counts: most occurrences of keywords, easy to game
- Citations: most webpages pointing to it, game it by creating useless pages
- **Page rank**: citations but every page has an importance weight

The iterative version will calculate the weights in rounds. The first iteration will assume all vertices are equally important, \[1/N\]. For every other iteration, \[\frac{1-d}{N} + d \times \sum_{u \ein in_neighbors(v)} \frac{PR_{i-1}}{out-degree(u)}\]. 

{{< subtext >}}
    \[d\] is the damping factor, and it's necessary to handle nodes with no outgoing edges. It's typically 0.85.
{{< /subtext >}}

**Recommender system** solves a problem where a database of users, items, and ratings given by each user to some of the items, and that it must be predicted how each user would rate items they have not rated yet. 

Non-negative matrix factorization has two views of the problem: sparse matrix view has rows as users, columns for movies, and the cell contain the rating the user gave the movie; graph view is a bipartite graph with the user set and movie set, and an edge between the two represents the rating given. The goal is to either predict the missing entires or missing edges.

For the matrix, decompose it such that \(A = WH\). H (kxn) and W (mxk) are dense so all missing values are predicted via dot product. The graph will have labels, where a user node is a vector corresponding to a row in W. Same for a movie node and H. Then use stochastic gradient descent (SGD) on the graph. Initialize all node labels to some arbitrary values. In one round, visit all edges in some order, dot product the labels, and use the residual to adjust the labels. Do it for some more rounds. Finally, predict by finding the dot products between labels.

{{< subtext >}}
    k = k << min(m, n).
{{< /subtext >}}



# {{< heading "Computational Science" >}}
Basically simulations; trying to model things on the computer. These require continuous models (e.g., differential equations) or discrete models. However, differential equations cannot be solved exactly (requires **discretization**—convert calculus problem to matrix computations). This issue persists with all continuous models.

One way to solve continuous models is with linear systems. \(Ax = b\), where \(A\) is a matrix and \(x\) and \(b\) are vectors. Direct methods (e.g., LU decomposition) doesn't produce useful information until the end. On the other hand, iterative methods will start with a guessed approximation \(x_0\). The error is \(Ax_0 - b\), which is called the residual. This is used to correct the approximation. Loop.

Jacobi: guess the 0 matrix. for each equation in the system, solve for a different variable. This is how we update the approximate matrix. \(x_{x+1} = x_i - M^{-1}(Ax_i - b)\), where \(M\) is the diagonal of A. This does not always converge, and if it does, it does very slowly. However, the operation is Matrix Vector Multiply (MVM).

MVM can be imagined as a graph. Each node has two labels x and y. y is set by retrieving the x of each neighbor and multiplying it by the edge weight, and summing it all up. Alternatively, coordinate storage will store the cell's value, row number, and column number. Or CRS will coalesce rows that point to column numbers and the values.

Finite differences concerns the initial value problem: figuring out the closed form of a recursive equation. This requires calculus the computer cannot do. Therefore, algorithms like Forward-Euler uses the difference definition of derivatives. \(h\) is the step size, and the smaller it is makes it more accurate at the cost of more computations. Backward-Euler steps backwards instead of forward. Picking a discretization scheme depends on *stability*, can it blow up, and *accuracy*, how small \(h\) has to be for a good estimate. 

{{< subtext >}}
    Backward-Euler is unconditionally stable, meaning it won't go crazy with a large \(h\), unlike Forward-Euler. 

    Centralized scheme is always stable.
{{< /subtext >}}

Solving partial differential equations is the same idea. It is implemented with two matrices, `current` and `next`. Basically using the five-point stencil.