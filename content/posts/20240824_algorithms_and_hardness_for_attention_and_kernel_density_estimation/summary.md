---
title: "[Lecture notes] Algorithms and Hardness for Attention and Kernel Density Estimation" 
date: 2024-08-24
tags: 
- Kernel Density Estimation
- Attention
draft: false 
---

## TL;DR
Kernel Density Estimation (KDE) is a statistical technique with applications across various fields, such as estimating the distribution of a random variable and computing the attention layer in Transformers. While the standard algorithm for KDE has a quadratic time complexity, this presentation introduces two advanced techniques (the polynomial method and the Fast Multipole Method) that reduce the computation time to nearly linear in certain cases.

## Kernel Density Estimation problem formulation
Given the following inputs: 
* 2n points: \(x_1,x_2,...,x_n\) and \(y_1,y_2,...,y_n\). Each point is m dimensional 
* A weight vector \(w\) which is n dimensional.
* error parameter \(\epsilon\)
* The kernel function \(f(x,y)\)
The goal is to approximately compute (compute up an error) the multiplication \(Kw\)
![KDE Goal](/posts/posts/20240824_algorithms_and_hardness_for_attention_and_kernel_density_estimation/kde_goal.png)

A Naive algorithm is with a complexity of \(O(mn^2)\) with the following implementation
1. Constructing the matrix K: Figuring out the \(m \times n\) entries of K. Each \(f(x_i,j_i)\) is with \(m\) multiplications and there are \(n^2\) combinations the need to be evaluated. 
2. Multiple \(K\) by the vector \(w\). This adds an additional of \(n^2\) computations
KDE almost linear solution can be achieved based on the input dimension regime.

### KDE in practice
The most widespread kernel is the Gaussian Kernel:
![Gaussian Kernel](/posts/posts/20240824_algorithms_and_hardness_for_attention_and_kernel_density_estimation/gaussian_kernler_function.png)

KDE applications define different parameters regimes. 
1. Estimate the probability density function of a random variable: \(m << n\).
2. n-body problem in physics: \(m << n\).
3. Attention computation in Large Language Models: The attention matrix is rescaling of Gaussian Kernel Matrix where \(n\) is the length of the context sequence and \(m\) is the vector size associate with a token. In the common case this \(m = O(\log n)\).


## Moderate dimension algorithm: polynomial method
1. Find a low rank approximation for \(K\): \(LR^T = ~K\)
2. Then we can compute \(Kw\) quickly: \(L(R^T w)\)
This can be achieved by a polynomial approximation of e^{-x}

### Chebyshev polynomials are a good approximators of e^{-x}
![e-x approximation](/posts/content/posts/20240824_algorithms_and_hardness_for_attention_and_kernel_density_estimation/e_-x_polynimials_approximation.png)
Using Chebyshev, less expansion terms (less j) are needed to reach a certain error bound compared to Taylor series.


## Low dimensional algorithm: Fast Multiple Method
We can apply the same method of the Moderate dimension such that when \(m=O(1)\) we can get \(\epsilon = 1/poly(n)\).
However, applying the Fast Multiple Method results is a better computational cost.

Steps:
1. Partition the space into boxes.
2. Sum the contribution to \(Kw\) over pairs of nonempty boxes.

There can be n different boxes so we may up with n^2 computations. We leverage the following insights. 
* If boxes are far apart, their contribution to the summation is negligible and can be ignored.
* If boxes are well-separated, we can use constant degree polynomial.
* If boxes are too close, we keep partition the space to smaller boxes until one of the previous points are met. 

## Lower bound of KDE computation cost
The above algorithm are optimal, there's a tight lower bound that shows we can not do better.
The main idea of the proof is the problem of KDE is equivalent to solve th HAmming closest pair problem in subquadratic time, which was already proven to require quadratic time. 


## Resource
[Youtube](https://www.youtube.com/watch?v=6Dsf1E6ZGP8)