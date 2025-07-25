A random variable allows for the ability to model and predict the outcomes of an experiment. There are two types which specify the possible values of a random variable:
- Discrete: Only specified values
- Continuous: *Can be* uncountably infinite 

[there CAN BE stuff in between]: #
## Discrete
A PMF assigns probabilities to each value of a discrete random variable $X$. It is commonly in the form $P(X = x)$, $x$ is a particular value. An important property to note is $\sum_{x} P(X = x) = 1$.

A Cumulative Distribution Function $F_{X}(x)$ is the sum of all probability masses up to $x$.

Joint PMFs are defined by the function $P(X = x, Y = y) = P(X = x \cap Y = y)$. It is possible to derive the PMFs of the individual random variables by summing over the other variable. For example, the marginal PMF of $X$ would be $\sum_y P(X = x, Y = y) = P(X = x)$. Joint CDFs $F_{X, Y}(x, y)$ also exist. This may be easily extended to more than two discrete random variables.

Conditional PMFs are defined as $P(X = x | Y = y) = \frac{P(X = x \cap Y = y)}{P(Y = y)}$. The multiplication rule works as one would expect. Extending the multiplication rule to three random variables gets $P(X = x \cap Y = y \cap Z = z) = P(X = x | Y = y \cap Z = z) \times P(Y = y | Z = z) \times P(Z = z)$. Random variables may be independent according to definitions one could expect. Additionally, functions of these random variables are independent too as long as they do not share random variables. Feel free to extend to even more random variables.

A distribution of random variable $Y$ derived from $X$ can be obtained by associating values of $X$ to $Y$. A PMF can be obtained by adding up the $P(Y = g(x))$.

The **uniform distribution** describes an equal $P(X = x)$ for all $x$. 

A **Bernoulli** random variable $X \sim Ber(p)$, where $p$ is the probability of success, describes the probability of a success and failure. In other words, $P(X = 0) = 1 - p$ and $P(X = 1) = p$.

-# $x$ only has two possible values: $0$ or $1$.

The following three distributions build off Bernoulli. When interested in the number of successes there are in independent, finite number of trials, a **binomial** random variable $X \sim Bin(n, p)$ is used, where $n$ is the number of trials. The PMF is
$$P(X = x) = \binom{n}{x} \times p^x \times (1 - p)^{n - x}$$
where $x$ is an integer defined on $[0, n]$. Skewness of the graph is determined by $p$:
- $p < \frac{1}{2}$: Skewed right
- $p = \frac{1}{2}$: Symmetric
- $p > \frac{1}{2}$: Skewed left

The **Poisson** random variable $X \sim Poi(\lambda)$ is like binomial but there must be a very large $n$, very small $p$, and a rate $\lambda$. Usually, $\lambda = np$. The PMF is
$$P(X = x) = \frac{e^{-\lambda}\lambda^x}{x!}$$

A **geometric** random variable $X \sim Geo(p)$ is for the number of trials to see a success. The PMF is
$$P(X = x) = (1 - p)^{x - 1} \times p$$
where $x$ is defined from 1 onward. Additionally,
- $P(X \geq x) = (1 - p)^{x - 1}$
- $P(X = a + b | X > a) = P(X = b)$, the memoryless property

## Continuous
The **probability density function** $f_X(x)$ is the continuous equivalent to the PMF. For the probability of an interval, take the definite integral. This interval does not care for whether the bounds are included. The property $\int_{-\infty}^{\infty} f_X(x)\; dx = 1$ must be satisfied to be a valid PDF. There is also the cumulative density function $F_X(x)$. It is simply defined as $\int_{-\infty}^{x} f_X(x)\; dx$. May be a piecewise function.

For a joint PDF of two random variables, imagine a 3D graph. The probability of the interval is essentially the shared area. Therefore, $P(X < x, Y < y) = \int_x  \int_y f_{X, Y}(x, y) dy dx$. If both random variables are bound by themselves, make the inner integral have the limit of the other variable and make the outer integral go across the entire number line. $X$ and $Y$ are jointly continuous if they individually take the same values. If integrating across the entire space, the double integral would sum to 1. A *marginal PDF* of X $f_X(x) = \int_y f_{X, Y}(x, y)$. The bounds of $f_X(x)$ will eliminate $Y$. Easy to expand to more than two continuous random variables.

$f_{X, Y}(x, y) = f_X(x) \times f_Y(y)$ if continuous random variables $X$ and $Y$ are independent of each other. Otherwise, $f_{X, Y}(x, y) = f_X(x) \times f_{Y | X}(y | x)$. We can derive the conditional PDF from this. Its bounds would be defined by the marginal PDF bounds.

The steps for getting the PDF of a derived distribution of one continuous random variable $Y = g(X)$ is as follows:
1. Redefine the definition of $Y$ to be in terms of $X$
2. Evaluate $F_X(g^{-1}(y))$ to get $F_Y(y)$
3. $\frac{d}{dy}F_Y(y) = f_Y(y)$
4. Plug in the endpoints of $X$ into $g(X)$

There are some shortcuts. If $Y = aX + b$, then $f_Y(y) = \frac{1}{|a|}f_X(\frac{y - b}{a})$. If $g(X)$ is a strictly monotonic function, then $f_Y(y) = f_X(g^{-1}(y)) \times |\frac{d}{dy}g^{-1}(y)|$. A strictly monotonic function either is always increasing or decreasing; exhibits no plateauing.

For the PDF of the distribution of continuous random variable $Z$ derived from two continuous, independent random variables $X$ and $Y$, consider that $P(Z \le z) = P({X \le z} \cap {Y \le z}) = P(X \le z) \times P(Y \le z)$. Then it's simply a matter of getting the CDF of $Z$ and deriving for the PDF. If $Z = X + Y$, then we can find $f_Z(z)$ with the *convolution* of $f_X(x)$ and $f_Y(y)$. To find the limits of integration for $f_Z(z) = \int_{-\infty}^{\infty} f_X(x)f_Y(z - x) dx$, generate a $ZX$ graph. The limits will be whatever vertically restricts an arbitrary portion of the generated shape.

A continuous **uniform** random variable $X \sim Unif[a, b]$ has a piecewise PDF where on the domain $[a, b]$, $f_X(x) = C$. 

The **normal** distribution $X \sim N(\mu, \sigma^2)$ is an accurate model for the amount of items that fall into each $x$, where applicable. $\mu$ is the expectation whereas $\sigma^2$ is the variance. $f_X(x) = \frac{1}{\sqrt{2\pi}\sigma}e^{-\frac{(x - \mu)^2}{2\sigma^2}}$ is a bell-shaped curve. As such, a normal random variable has the property of *symmetry*. Since the PDF is not easy to integrate, we will normalize $X$ such that $Z \sim N(0, 1)$ when we want to get the probability of a range of $X$.
$$Z = \frac{X - \mu}{\sigma}$$
Then look at the standard normal table to get the $P_X(x < k)$. Use the property of symmetry to calculate more complex expressions.
-# The area between three standard deviations from the mean is 99.7%

[TODO: standard normal table here]: #
The **Exponential** distribution $X \sim Exp(\lambda)$ models the amount of time until an event happens. $\lambda$ is the reciprocal of average. As with a number of things, the exponential distribution also shares the memoryless property with the geometric distribution.
$$f_X(x) = \lambda \times e^{-\lambda x}$$