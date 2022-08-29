---
title: Cute Counter
author: Jorge Duque
date: 2021-08-27
categories: [Algorithms, Probability]
math: true
mermaid: false
---


## The Problem
Even though counting seems like an easy problem, it can have its difficulties. When data is streaming, we assume a nearly endless influx of data. This means that when we are counting something, we could reach some large numbers. We would like to get a way to estimate large counts, but use memory efficiently.  

## Moris's Algorithm
[Moris's Algorithm](https://en.wikipedia.org/wiki/Approximate_counting_algorithm) is a clever solution for this. The algorithm is as follows.

1. Intitialize $x \to 0$
2. When event of interest occurs, increment $x$ by 1 with probability $\frac{1}{2^x}$
3. Repeat step 2

Below is a class that implements this counter in python.

```python
from random import random


class CuteCounter:
    def __init__():
        self.x = 0

    def query() -> int:
        return (2 ** self.x) - 1

    def update() -> None:
        p = 1 / (self.query() + 1)
        if random() < p:
            self.x += 1

```


## Analysis

### (Unbiased) Estimator

$$
E\left[b^{X_n} | X_{n-1} \right] = b^{X_{n-1} + 1} \frac{1}{b^{X_{n-1}}} + b^{X_{n-1}} \left( 1 - \frac{1}{b^{X_{n-1}}}\right) 
= b + b^{X_{n-1}} - 1
$$

Which means
$$
 E\left[b^{X_n}\right]
= \sum E\left[b^{X_n} | X_{n-1} \right] P(X_{n-1})
$$

$$
= \sum \left[ b + b^{X_{n-1}} - 1\right] P(X_{n-1})
= b + \sum b^{X_{n-1}}P(X_{n-1}) - 1
$$

$$
= b - 1 + E\left[ b^{X_{n-1}} \right]
$$

This forms a recursive formula for the expected value. Pairing it with the fact that $X_0 = 0$, we can solve this exactly. 

$$
E\left[b^{X_n}\right] 
= (b - 1)n + 1
$$

This means that in order to have an unbiased estimator, we should use the following as our estimator, $\hat{n}$.

$$ 
\hat{n} = \frac{b^{X_n} - 1}{b - 1}
$$

For the case of the base being 2, we get the value we used in the code snippet shown above.

### Error & Variance
Ideally, the variance of the estimator should be small. Because the variance measures how far a value deviates from the mean, the variance can be thought of as an error for our problem.

$$
\text{Var}\left[\hat{n} \right] = \text{Var}\left[\frac{b^{X_n} - 1}{b - 1} \right] = \frac{1}{(b - 1)^2}  \text{Var}\left[b^{X_n} \right]
$$

Recall that $\text{Var}\left[b^{X_n} \right] = E\left[b^{2X_n}\right] - E\left[b^{X_n}\right]^2 $. The second term in the right-hand expression is something we already know, so let's solve for the first term. Since we're going to solve for it using conditionals, let's solve the conitional expectation. 


$$
E\left[b^{2X_n} | X_{n-1} \right] 
= b^{2(X_{n-1} + 1)} \frac{1}{b^{X_{n-1}}} + b^{2X_{n-1}} \left( 1 - \frac{1}{b^{X_{n-1}}} \right) 
= b^{X_{n-1} + 2} + b^{2X_{n-1}} - b^{X_{n-1}}
$$

This then implies the following. 

$$
 E\left[b^{2X_n}\right]
= \sum \left( b^{X_{n-1} + 2} + b^{2X_{n-1}} - b^{X_{n-1}} \right) P(X_{n-1})
$$

$$
= b^2 \sum b^{X_{n-1}} P(X_{n-1}) + \sum b^{2X_{n-1}} P(X_{n-1}) - \sum b^{X_{n-1}} P(X_{n-1})
$$

$$
= \left(b^2 - 1 \right) \sum b^{X_{n-1}} P(X_{n-1}) + \sum b^{2X_{n-1}} P(X_{n-1}) 
$$

$$
= \left(b^2 - 1 \right) E\left[ 2^{X_{n-1}}\right] +  E\left[b^{2X_{n-1}}\right]
$$

Like we did for the mean, we can solve this recursively and get the following result.

$$
 E\left[b^{2X_n}\right] = \left( b^2 - 1\right) \sum_{k=1}^n k 
 = \left( b^2 - 1\right)\frac{n (n+1)}{2}
$$

Plugging this into the variance formula, we get:


$$
\text{Var}\left[b^{X_n} \right] 
= \left( b^2 - 1\right)\frac{n (n+1)}{2}  - ((b - 1)n + 1)^2 
$$

Putting it all together, we get:

$$
\text{Var}\left[\hat{n} \right] 
= \frac{1}{(b - 1)^2} \left[\left( b^2 - 1\right)\frac{n (n+1)}{2}  - ((b - 1)n + 1)^2 \right]
$$


$$
= \frac{(b + 1)n(n+1)}{2(b - 1)} - \left(n + \frac{1}{b-1}  \right)^2
$$

$$
= - \frac{(b-3)(b-1)(n-1)n + 2}{2 (b-1)^2}
$$

This means that the average error grows quadratically with respect to $n$. The typical way the error was redices was by having several counters and averaging their results. That analysis can be shown to reduce the error, but that is for another post.



#### Note
The formula above with the variance, might have an error somewhere because it is pretty much negative for most of the bases. An interesting thing to note is that if $b=3$, the variance of the estimator is actually a constant, -0.25. I will review later, because I am tired right now. Lol