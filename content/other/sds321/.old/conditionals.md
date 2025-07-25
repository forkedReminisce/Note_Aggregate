Given the partial information that event $B$ is true, the probability of event $A$ may change. The formula for **conditional probability** is:
$$P(A | B) = \frac{P(A \cap B)}{P(B)}$$
The general multiplication rule is a rearrangement: $P(A \cap B) = P(B) \times P(A | B)$.
## Probability Trees
A technique for calculating **joint probability**—$P(A \cap B)$. Vertices are points in the experiment, and edges are the event given previous events. The weight of an edge is the conditional probability; the sum of the weights from one vertex should equal 1. At the end, go down a path and multiply the weights with each other.
**Bayes Rule** is just the reverse of the conditional: $P(B | A) = \frac{P(B) \times P(A | B)} {P(A)}$. To find $P(A)$, **Total Probability Theorem** is used; sum every $P(A \cap E)$, for every preceding event(s) $E$.
## Independent Events
The occurrence of event $B$ does not affect the probability of event $A$. In formulaic terms, $P(A | B) = P(A)$. Additionally, there is a special multiplication rule $P(A \cap B) = P(A) \times P(B)$. 
**Mutual independence** is when three or more events are all independent of each other. Say for three arbitrary events $X$, $Y$, and $Z$, the following conditions must be met for them to be mutually independent:
- $P(X \cap Y) = P(X) \times P(Y)$
- $P(X \cap Z) = P(X) \times P(Z)$
- $P(Y \cap Z) = P(Y) \times P(Z)$
- $P(X \cap Y \cap Z) = P(X) \times P(Y) \times P(Z)$

The first three belong to a class known as pairwise independent. This may be expanded for a greater number of events.
**Conditional independence** means that events events $A$ and $B$ do not provide information about each other *given event $C$*. The associated equations are:
- $P(A | B \cap C) = P(A | C)$
- $P(A \cap B | C) = P(A | C) \times P(B | C)$