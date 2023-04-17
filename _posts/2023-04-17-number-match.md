---
title: 'Number Match: No-Match Game(s)'
author: Jorge Duque
date: 2023-04-17 
categories: [Games, Math, Combinatorics]
math: true
mermaid: false
---


## Introduction to Number Match
Easy Brain does it again! I've been playing their game [Number Match](https://easybrain.com/news/new-puzzle-game-in-easybrain-portfolio-number-match) and it's pretty fun. The game is relatively straight forward: match pairs of numbers and clear the board. It's a grid of 3x9 squares initialized with numbers from 1 to 9. You can pair numbers that are adjacent to one another horizontally, vertically and diagonally. Additionally, you can pair a number at the beginning of one line with the one at the end of the previous line. Numbers that are the same or 10's complements can be matched. If every number in a row is matches, the row is removed. If you run out of matches, you are allowed to duplicate the board up to five times. ie every number that is unmatched gets added to the bottom of the grid in order. It's a pretty fun game and I recommend it if you're into puzzle games.


## No-Match Configuration(s)
After playing this game for a while, I wondered how many (if any) initializations don't allow any matches. Like it has no matches, even if you duplicate. A brute-force search would be pointless as there are 5,814,973,700,304,005,969,0390,169 different board configurations, so we need a better way of solving this problem. We begin our journey by going back to elementary school and turning our counting problem into a coloring problem.

![Me Coloring](https://cdn.vox-cdn.com/thumbor/AwXKgUqjLLRMlcZ1dhchg4P3Jwo=/7x0:3007x2250/1200x800/filters:focal(7x0:3007x2250)/cdn.vox-cdn.com/uploads/chorus_image/image/47078882/shutterstock_2800547.0.0.jpg)


## Graph Coloring
Let's stop thinking about this problem as numbers on a grid, but as nodes in a network. First notice that for our problem the numbers don't matter so much, what matters is that we don't have a match. We can lump the numbers that match into their own color and now to find a No-Match game corresponds to finding a coloring of the network where neighbors are different colors. If you are not familiar with graph coloring, you might think: "my child colors with five colors all the time, this should be easy." No, no, no. Stop that. This is actually hard and your child colors stupid stuff.


Let's formalize a little more and relate this problem to our original problem. We want to compute the number of grid configurations with no matches, and we shall refer to this number as $X$. Our new problem is: how many ways can we color our graph with five colors (or less) where no neighbors are the same color? We will refer to this number as $P_5$. The five colors correspond to the following numbers: $C_1 = \{1, 9\}$, $C_2 = \{2, 8\}$, $C_3 = \{3, 7\}$, $C_4 = \{4, 6\}$, $C_5 = \{5\}$.

Our two problems are related in the following way:

$$ X = 2^{N-N_5} P_5 $$

where $N$ is the total number of nodes in the graph and $N_5$ is the number of nodes that are colored with $C_5$. Below is a plot of our network.


```python
import warnings
from graff import make_graph

import networkx as nx

warnings.filterwarnings('ignore')

graph = make_graph(3, 9)
nx.draw_networkx(graph, with_labels=True)
```


    
![png](/images/2023-04-17-number-match/output_3_0.png)
    


You can look at the code used to generate the graph [here](https://github.com/jiduque/effective-octo-giggle/blob/main/number-match/graff.py).

## Chromatic Polynomials

How do we find $P_5$? So there's this little thing called the [chromatic polynomial](https://en.wikipedia.org/wiki/Chromatic_polynomial). Conveniently, it's a polynomial $P_G(k)$ that tells us how many ways you can color a graph $G$ using $k$ colors. So now we get

$$P_5 = \sum_{k=1}^5 {5 \choose k} P_G(k)$$

Notice that for our graph we will need at least 4 colors to color this properly (aka $P_G(k) = 0$ for $k \leq 3$). This is because if you get any 2x2 square in the grid, they are all connected to each other. Therefore, we need at least four colors to not have neighbors be the same. This reduces our formula to:

$$P_5 = 5P_G(4) + P_G(5)$$

Now all we have to do is find the chromatic polynomial.

There are actually a couple of algorithms that can be used to compute it, but most are not that efficient; the efficient ones work for special cases. The basic algorithm, called [the deletion-contraction formula](https://en.wikipedia.org/wiki/Deletion–contraction_formula), is a recursive formula that reduces the graphs to a bunch of edgeless graphs. It runs in exponential time, and it is the implementation used in NetworkX's `chromatic_polynomial` function. Wonderful, we've turned our computationally intractable problem into another computationally intractable problem.



```python
# DO NOT RUN THIS UNLESS YOU WANT YOUR COMPUTER TO BE ETERNALLY SLOW AND OUT OF BREATH
%%script echo skipping
nx.chromatic_polynomial(graph)
```

## Chromatic Number
We couldn't get the polynomial of our dreams :broken_heart:. But let's not lose hope, my mom says there are plenty of fish in the sea. What if we were trying to prove that you can't color it with five colors? I think I would try to show that the [chromatic number](https://mathworld.wolfram.com/ChromaticNumber.html), $\chi(G)$, is greater than five. The chromatic number of a graph $G$ is defined as the smallest number of colors needed to color the graph with all neighbors being a different color. It's related to the chromatic polynomial in the following way:


$$ \chi(G) = \min_{k \in N} \{ k: P_G(k) \neq 0 \} $$

Finding the exact value is intractable for our problem, but we don't need the number we just want a lower bound.


### First Try: Hoffman's Bound (You Can Honestly Skip This)
In my search for solving this problem, I found a nice inequality relating a graph's chromatic number and the eigenvalues of its [adjacency matrix](https://en.wikipedia.org/wiki/Adjacency_matrix). It's called [Hoffman's ratio bound](https://arxiv.org/abs/2102.05529), and it is as shown below:

$$\chi(G) \geq  1 - \frac{\lambda_1}{\lambda_n}$$

where $\lambda_1$ and $\lambda_n$ are the largest and smallest eigenvalue respectively.


```python
eigenvalues = nx.adjacency_spectrum(graph)
lambda_max, lambda_min = max(eigenvalues),  min(eigenvalues)
lambda_max, lambda_min
```




    ((7.737569821141115+0j), (-3.6863352947483374+0j))



Since the eigenvalues don't have complex components, we can just get the real parts.


```python
lambda_max, lambda_min = lambda_max.real,  lambda_min.real
1 - (lambda_max / lambda_min)
```




    3.0989869891011503



I was really hoping this would get it done as it's such a nice bound that uses efficient techniques. Unfortunately it wasn't the result we hoped for and have to go back to basics.

### The Classics: Clique Number

If we find a subgraph where all the nodes are connected to each other, called a clique, we can bound the chromatic polynomial of our graph by this number. This is the same argument we used to show $P_G(3) = 0$. If we can find a clique of size 6 or more, we essentially show that we cannot have a no-match configuration.


Finding the largest clique in a graph is another one of those hard problems, but approximating it can actually be done efficiently. I.e. we can find a  large clique, but it's not guaranteed to be the largest one. And for our problem, that's fine so long as it's at least six nodes. Otherwise, we're out of luck. Here we get a large clique and plot it to see the size.


```python
clique_nodes = nx.approximation.max_clique(graph)
clique = graph.subgraph(clique_nodes)
nx.draw_networkx(clique, with_labels=True)
```


    
![png](/images/2023-04-17-number-match/output_13_0.png)
    


$$\chi(G) \geq 6$$

There it is, the *bound* of our dreams: simple, elegant, dreamy.

## Conclusion

You can always score a point in Number Match because there will always be at least one match! So go play the game, get frustrated, and have fun. You can take the same advice for your dating profile, but a match isn't guaranteed there. :heart-broken:
