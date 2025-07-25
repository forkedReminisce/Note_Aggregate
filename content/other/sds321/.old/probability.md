# Probability
An **experiment** has multiple possible outcomes decided by chance. An **event** is a set of outcomes; **sample space ($\Omega$)** is the set of all possible outcomes. 

There are two ways of measuring simple probability:
- Divide the number of favorable outcomes by the size of the sample space
- Keep repeating the experiment and measure the proportion that favorable events occur—relative frequency approach to probability. Eventually, the proportion will converge with the calculated probability from the above method.

## Probability Law
$P(E)$ is the probability of event *E*. With this are the following axioms:
- Non-negativity
- Additivity — $P(A \cap B) = P(A) + P(B)$
- Normalization — $P(\Omega) = 1$

Actually, additivity only applies to mutually exclusive events. Events are said to be **mutually exclusive** if the occurrence of one implies the others do not. In formulaic terms, $P(A ∩ B) = 0$. 

To calculate the general probability of a union of events, consider a venn diagram of the events with size proportional to probability. It will be found that a combination of adding unions and subtraction intersections must be used. For example, $P(A \cap B) = P(A) + P(B) - P(A \cup B)$.

-# $P(A \cup B) = P(A) + P(A^c \cap B)$ is also valid.

## Partition
Two sets are disjoint if their intersection is an empty set. By creating some union of disjoint events of a parent set, a partition is created. 