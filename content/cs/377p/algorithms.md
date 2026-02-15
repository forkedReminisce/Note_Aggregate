---
draft: true
title: 

params: 
    desc: 
    author: Andrew Nguyen 
---



# {{< heading "Compressing Graphs" >}}
Adjacency matrices explicitly note that two nodes are not connected. For large graphs, this is bad for space and increases processing overhead. So instead:
- Coordinate storage: 2d array, where each column represents an edge of source node, destination, and edge weight
- Compressed sparse row (CSR): every source node is mapped to an index of a 1d array. The value is an index into a 2d array of destinations and edge weight. 
- Compressed sparse column (CSC): like CSR, but destinations map to the 1d array and the 2d array is source nodes and edge weights
- Adjacency list

{{< subtext >}}
    Deleting edges in adjacency lists is as simple as marking the edge and handling it later.
{{< /subtext >}}



# {{< heading "Algorithms" >}}
One way to define an algorithm is with **operator formulation**. An algorithm has an operator and schedule. In graphs, the *operator* is the computation performed at a selected node. *Schedule* decides which active node gets selected next. This makes comparing algorithms simpler, and makes it possible to pick and choose an operator or schedule from different possible to create the best algorithm.

{{< subtext >}}
    A schedule might seem bad, but it might better support parallelism.
{{< /subtext >}}

Active nodes are the subset of nodes ready to be processed. *Data-driven algorithms* are what we are most familiar with, and it creates active nodes as it processes a node. *Topology-driven algorithms* execute in rounds, and in each round, all nodes are active. 



## {{< heading "Machine Learning" >}}
When making a web search, the webpages should be in some order. There's an offline part with crawlers, which adds an index from keywords to webpages. The online part will use the index to find pages that contain the most keywords of the search. There are multiple approaches to ranking pages:
- Manual ranking
- Word counts: most occurrences of keywords (easy to exploit)
- Citations: most number of webpages linking to it (exploit with useless pages)
- **Page rank**: citations but every page has an importance weight

Imagine a web graph where the nodes are webpages and edges are links. In the iterative version, all nodes will start with the weight \(\frac{1}{N}\). The operator that follows is:

\[
    \frac{1 - d}{N} + d \times \sum_{u \in \textrm{in_neighbors(v)}} \frac{\mathnormal{PR}_i}{\textrm{out_degree(u)}}
\]

Where \(d\) is the damping factor. It is a fraction that corrects the importance of nodes with no outgoing edges.  

A recommender system solves a problem where with a database of users, items, and ratings given by each user to some of the items, it must be predicted how each user would rate items they have not rated yet. One way this is achieved is with **non-negative matrix factorization**.

Two representations of the database will be maintained. The sparse matrix has rows as users, columns for items, and the value be the user's rating. The bipartite graph will have one side be users and the other be movies, and the edge between a node from each set has a label matching the rating. The goal is to either predict the missing entries or missing edges.

<!-- i hate this -->
For the matrix, decompose it such that \(A \approx WH\). In the graph, a user node will be given a label of a row in \(W\). Likewise, an item node will get a column of \(H\). *Stochastic gradient descent* will then predict the values. It is a topology-driven algorithm that visits edges instead of nodes. The operator at a selected edge is to dot product the labels of its endpoints to calculate the residual. This residual is used to adjust the nodes' labels. 

{{< subtext >}}
    \(W\) is size \(m \times k\), while \(H\) is \(k \times n\). \(k\) is a number.
{{< /subtext >}}



## {{< heading "Computational Science" >}}
Computational science is the backbone to simulations. These require continuous and discrete models. 

Differential equations are one kind of continuous model. However, they cannot be solved exactly. Therefore, **discretization** converts calculus to matrix computations. One way to go about this is solving linear systems. Imagine \(Ax = b\), where \(A\) is a matrix and \(x\) and \(b\) are vectors. Direct methods (e.g., LU decomposition) do not produce useful information until it finishes, so iterative methods are preferred. 

The basic premise of iterative methods is to start with an initial guess, calculate a residual, use that to correct the guess, and loop. *Jacobi* uses the formula \(x_{x+1} = x_i - M^{-1}(Ax_i - b)\), where \(M\) is the diagonal of \(A\), for the residual of any variable. This does not always converge, and if it does, it does very slowly. However, it does use Matrix Vector Multiply (MVM).

<!-- TODO: MVM can be imagined as a graph. Each node has two labels x and y. y is set by retrieving the x of each neighbor and multiplying it by the edge weight, and summing it all up. Alternatively, there's some formulas with coordinate storage or CSR. -->

Finite differences concerns the initial value problem: figuring out how a recursive equation grows. This requires calculus the computer cannot do. Therefore, algorithms like Forward-Euler uses the difference definition of derivatives. 

\[
    \frac{f(nh + h) - f(nh)}{h}
\]

\(h\) is the step size, and the smaller it is makes it more accurate at the cost of more computations. \(n\) is the step amount. Backward-Euler steps backwards instead of forward, so the terms of the numerator are flipped and \(f(nh + h)\) becomes \(f(nh - h)\). Alternatively, centered differences:

\[
    \frac{f(nh + h) - f(nh - h)}{2h}
\]

Picking a discretization scheme depends on *stability*, can it blow up, and *accuracy*, how small \(h\) has to be for a good estimate. Backward-Euler and Centered differences are always stable.

Solving partial differential equations is the same idea. An iterated derivative will find the difference between a Forward-Euler and Backward-Euler and divide that by \(h\). 