## Expectation
Expectation is average. The expectation of a discrete random variable $X$ is: 
$$E[g(X)] = \sum_{x} g(x) \times P(X = x)$$ 
If $g(X)$ has multiple terms, feel free to split each one into its own expectation. $E[C] = C$.

-# Moments are defined as the expectation of a random to some power. For example, $E[X^2]$ is the second moment of $X$.

Conditional expectation of a random variable given either an event or random variable simply changes the PMF portion of the summation to contain the same "given" statement. To find the general expectation of the random variable, use Total Expectation Theorem: $E[X] = \sum_i E[X | B_i] \times P(B_i)$. $B_i$ is from partitioning the sample space—essentially every "given" statement that can lead to $X$. If $X$ and $Y$ are independent, then $E[XY] = E[X] \times E[Y]$.

It is possible to find the expectation of distributions. There is some logic behind them, some more complex than simply plugging in values, but it is better to simply list the expectations out.
- Bernoulli: $p$
- Binomial: $np$
- Poisson: $\lambda$
- Geometric: $\frac{1}{p}$

Expectation of a continuous random variable is $E[g(X)] = \int_{-\infty}^{\infty} g(x) \times f_X(x) dx$. Integral in place of summation occurs for conditional expectation too. For the various distributions...
- Uniform: $\frac{a + b}{2}$. 
- Exponential: $\frac{1}{\lambda}$

## Variance
Variance measures spread. There are two forms of the equation: 
$$var(g(X)) = E[g^2(X)] - (E[g(X)])^2$$

Standard deviation of $X$ is also a measure of spread, and it is calculated by the formula $\sigma_X = \sqrt{var(X)}$.

There's some properties to know about variance: $var(X + k) = var(X)$ and $var(aX) = a^2 \times var(X)$.

Like expectation, it is possible to get the variance of distributions.
- Bernoulli: $p(1 - p)$
- Binomial: $np(1 - p)$
- Poisson: $\lambda$
- Geometric: $\frac{1 - p}{p^2}$
- Uniform: $\frac{(b - a)^2}{12}$
- Exponential: $\frac{1}{\lambda^2}$

# Covariance
Like how variance measures spread of one random variable, covariance is of two.
$$cov(X, Y) = E[XY] - E[X]E[Y]$$

-# For $E[XY]$, the argument is like a multivariate function. Additionally, the joint probability is used.

Plotting is probably the best way to understand the types of covariance. When the points follow an upward trend, there is a positive covariance. Conversely, a downward trend means a negative covariance. No trend is zero covariance. On that point, if the two random variables are independent of one another, their covariance is 0. However, this implication does not go the other way. 

There are some properties to know about covariance:
- $cov(X, X) = var(X)$
- $cov(X, aY) = a \times cov(X, Y)$
- $cov(X, Y + b) = cov(X, Y)$
- $var(X \pm Y) = var(X) + var(Y) \pm 2cov(X, Y)$
-# For any more random variables, add covariances for every combination.

One last property is when the arguments include functions of both random variables, split it into individual covariances for every combination. Think the FOIL method for multiplying polynomials.

Covariance by itself doesn't really tell much, though. Therefore, correlation standardizes covariance by placing it on a range of $[-1, 1]$. Obviously the sign represents the type of relationship, but now strength can be deduced from the magnitude as well. 

-# Around 0.5 does the correlation become strong.
$$\rho_{X, Y} = \frac{cov(X, Y)}{\sqrt{var(X)var(Y)}}$$

A minor property to know about correlation is $\rho_{X, aX + b}$ is 1 when $a$ is positive, and -1 when negative