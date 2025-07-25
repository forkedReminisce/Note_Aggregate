Accurately calculating probability of an event requires a correct count of outcomes. By finding the probability of one outcome, $P(E)$ is as simple as multiplying this by the number of outcomes.
## Counting Principle
In a probability tree, at each level, if each vertex has the same number of vertexes that are equally-likely, counting is quite simple. Going down one path, count the number of children of each passed vertex. Then multiply these counts.
## Sampling Types
Let $n$ be the number of elements in a set, and $k$ be the amount to sample. It is worth pointing out that sets may contain sets.
- **Ordered** sampling without replacement: $P_k^n = \frac{n!}{(n - k)!}$
- **Unordered** sampling without replacement: $\binom{n}{k} = \frac{n!}{k!(n - k)!}$
#### A set of k-permutations contains each unordered sequence $k!$ times.
## Balls in Bins
Simply another way to look at a scenario. Imagine that some balls have to be placed between some bins. The goal is to figure out how many different ways the bins can be filled. 
**Distinguishable** balls and distinguishable bins use multinomial coefficients. This is where the set of objects is split into disjoint subsets of which order does not matter within them. Let $n$ be the size of the parent set, and $n_1, n_2, \cdots, n_r$ be the cardinalities of each subset. The counting formula is $\frac{n!}{n_1!n_2! \cdots n_r!}$.
**Indistinguishable** objects and distinguishable boxes require making an equation between the number of balls in each bin and the total number of balls. If $x_i$ is the number of balls in bin $i$, and $n$ is the number of balls, the equation should look something like:
$$x_1 + x_2 + \cdots + x_k = n$$
- If $x_i > 0$, $\binom{n - 1}{k - 1}$ 
- If $x_i \geq 0$, $\binom{n + k - 1}{k - 1}$