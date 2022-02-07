---
title: N-digit Fibonacci Number
author: Jorge Duque
date: 2020-06-12 
categories: [Math, Project Euler]
math: true
mermaid: false
---

**Note: This is problem 25 on Project Euler**

# Problem Statement


The Fibonacci sequence is defined by the recurrence relation:

$$    F_n = F_{n−1} + F_{n−2} $$

where $F_1 = 1$ and $F_2 = 1$. If you continue to generate thes numbers, you will find that $F_{12} = 144$. The 12th term is the first to contain three digits. Can you find the index of the first term in the Fibonacci sequence that contains 1000 digits?

# Solution(s)
This problem is somewhat similar to Problem 2 on Project Euler, but I actually think it is easier. Below, I present two solutions to the problem. One is what I think is the most straight-forward way of doing it, and the other involves reframing the problem and using generating functions (used in Problem 2). My Python implementation of the second solution can be found <a href="https://github.com/jiduque/project-euler/blob/main/Problem25.py"> here </a>.

## Simple Solution
The simple way to solve this problem is to just keep generating Fibonacci numbers until you find the one that is of the desired length. A solution to that would look like the code below. 

```python
def n_digit_fibonacci(desired_length):
    if desired_length == 1:
        return 1

    n = 2
    fn_1, fn_2 = 1, 1
    fn = fn_1 + fn_2

    while len(str(fn)) < desired_length:
        n += 1
        fn_2 = fn_1
        fn_1 = fn
        fn = fn_1 + fn_2
    
    return n
```

This simple implementation runs in linear time, but we can do better... kinda. First let's think about an $n$-digit number means. 


## N-digit Numbers
The smallest $n$-digit number is $10^{n-1}$. This then means that for a number, $x$, to be $n$ digits long, it has to satisfy the following condition:

$$10^{n-1} \leq x < 10^n $$

Please note that assumes that we are working in base 10. This insight allows us to modify the code above in such a way where we no longer have to convert our number into a string. But more importantly, it reframes our problem to solving an inequality. 


## Solving the Inequality
Plugging in the generating function of the Fibonacci numbers, we get the following inequality.

$$ 10^{n-1} \leq \frac{1}{\sqrt{5}} \left( \phi^k - \psi^k \right) < 10^n$$

This is trancendental so we can't really solve for $k$ exactly, but we can approximate. Notice that $\psi^k \to 0$ as $k$ gets large. This means we can approximate $k$ with the value shown below.

$$k \approx \left\lfloor \frac{\log{\sqrt{5}} + (n-1) \log{10}}{ \log{\phi}} \right\rfloor $$


## The Issue
This shows we can solve the problem essentially in constant time (for large values)... at least in theory. For our particular problem, we have to find a value that has 1000 digits or more, which leads to issues. The issue lies in the fact that $k$ is an approximation, so we still have to check if value satisfies the conditions and check if its neighbors satistfy the condition. Although we can generate Fibonacci numbers in constant time using generating functions, it requires raising a floating point numbers to 1000, at which point we will overflow/underflow. So we still have to generate Fibonacci numbers the usual way.   