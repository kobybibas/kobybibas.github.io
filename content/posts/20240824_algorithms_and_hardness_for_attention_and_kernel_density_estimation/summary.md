---
title: "[Lecture notes] Algorithms and Hardness for Attention and Kernel Density Estimation" 
date: 2024-08-24
tags: 
- Kernel Density Estimation
- Attention
draft: false 
---

## TL;DR
Kernel Density Estimation is a statistical task with many applications in many different fields: from estimating the distribution function of a random variable t o computing the attention layer of Transformers
It has a straightforward quadratic-time algorithms. In this presentation two algorithmic techniques are used (the polynomial method, and the Fast Multipole Method) leading to almost linear-time algorithms in different parameter regimes.

## Kernel Density Estimation
Given 
* 2n points: \(x1,x2,...,xn\) and \(y1,y2,...,yn\).
* weight vector w
* error parameter \epsilon
The goal is to find the kernel matrix K such that:
![KDE Goal](/posts/content/posts/20240824_algorithms_and_hardness_for_attention_and_kernel_density_estimation/kde_goal.png)

A Naive algorithm is with a complexity of  \(O(mn^2)\): Construct the matrix K, figurING out the \(m \times n\) entries of K are and multiple by the vector w??

Input is m*n, output is O(n).
Can we don't explicitly compute K?

The Gaussian Kernel is the most widespread 
![Gaussian Kernel](/posts/content/posts/20240824_algorithms_and_hardness_for_attention_and_kernel_density_estimation/gaussian_kernler_function.png)

KDE applications define different parameters regimes. 
1. Estimate the probability density function of a random variable
2. n-body problem in physics
3. Attention computation in Large Language Models: The attention matrix is rescaling of Gaussian Kernel Matrix. n is the length of the context sequence. m is the vector size associate with a token. m = O(log n)

## KDE results
What error can be achieved quickly based on the input dimension regime.  
TBD image

## Moderate dimension algorithm: polynomial method
1. Find a low rank approximation for K (LR^T = ~K)
2. Then we can compute Kw quickly: L(R^T w)

This can be achieved by a polynomial approximation of e^{-x}

### Chebyshev polynomials are a good approximators of e^{-x}
![Gaussian Kernel](/posts/content/posts/20240824_algorithms_and_hardness_for_attention_and_kernel_density_estimation/e_-x_polynimials_approximation.png)
Using Chebyshev, less expansion terms (less j) are needed to reach a certain error bound


## Low dimensional algorithm: Fast Multiple Method
For physics application.
We can apply the same method of the Moderate dimension,
If m=O(1) we can get \(\epsilon = 1/poly(n) \)  
However, there's a better algorithm : Fst Multiple Method

Steps:
1. Partition the space to boxes.
2. Sum the contribution to Kw over pairs of nonempty boxes.
There can be n different boxes so we may up with n^2 computations. 

* If boxes are far apart, we can ignore their contribution to the summation since their contribution is negligible
* If boxes are well-separated, we can use constant degree polynomial.
* If boxes are too close, keep partition the space to smaller boxes until one of the previous points are met. 

## Lower bound
The above algorithm are optimal, there's a tight lower bound that shows we can not do better.
The main idea of the proof is the problem of KDE is equivalent to solve th HAmming closest pair problem in subquadratic time, which was already proven to require quadratic time. 


## Resource
[Youtube](https://www.youtube.com/watch?v=6Dsf1E6ZGP8)