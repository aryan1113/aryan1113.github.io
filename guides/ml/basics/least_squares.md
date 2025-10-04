---
last_modified: 2025-07-10 17:55:29 +0000
layout: ml-guides
title: Linear Least Squares Regression
description: regress using matrices
usemathjax: true
---

## Loss function for the algorithm?
loss function in machine learning is simply a measure of how different the predicted value is from the actual value.
The quadratic Loss Function calculates the loss or error in our model. It can be defined as:

$$ \text{SSR} = \sum_{i=1}^{n} (y_i - \hat{y}_i)^2 $$

Minimizing it by finding the partial derivative of L, equating it to 0, and then finding an expression for m and c. After we do the math, we are left with these equations:

$$
c = \hat{y} - m\hat{x} \\
m = \frac{\sum_{i=1}^{n} (x_i - \hat{x})~(y_i - \hat{y})} {\sum_{i=1}^{n} (x_i - \hat{x})^2}
$$

 
## Least Squares using matrices
The idea is to find the set of parameters ($\theta$) such that:
$ X \theta = y $
where,

X: input data with dimensions (n,m), <br>
Θ: parameters with dimensions (m,1), <br>
y: output data with dimensions (n,1), <br>
n: number of samples, <br>
m: number of features

The parameter matrix \theta can be directly determined by multiplying both sides of the equation with the inverse of X, as : $ \theta = X^{-1}y $ <br>
But because X might be a non-square matrix, its inverse may not be defined.

<figure style="text-align: center;">
  <img src=
    "https://substackcdn.com/image/fetch/$s_!Bb0O!,w_1456,c_limit,f_webp,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2Ffa8f21f8-6337-4ed9-b7ab-6e2e7b2695fa_2397x861.png"
  alt="Illustration" style="max-width: 100%;" />
  <figcaption><em>Note: Inverse still may not exist for a square matrix if it is a singular matrix</em></figcaption>
</figure>

To resolve this, first, we multiply with the transpose of X on both sides, as : $ X^{T}X\theta = X^{T}y $

This makes the product of X with its transpose, a square matrix.

<figure style="text-align: center;">
  <img src="https://substackcdn.com/image/fetch/$s_!CvAx!,w_1456,c_limit,f_webp,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2F93008674-7f3a-443b-83ca-cc707559e55e_2024x481.png" 
  alt="Illustration" style="max-width: 100%;" />
  <figcaption><em>Square matrix from the product</em></figcaption>
</figure>

The obtained matrix, being square, can be inverted. Taking the collective inverse of the product to get the following:

$$ \theta = ( X^{T}~X ) ^ {-1}~X^{T}~y $$

No randomness. Thus, it will always return the same solution, which is also optimal.
This is precisely what the Linear Regression class of Sklearn implements, Instead of gradient descent.<br>
Head over here to see projects being <a href="https://github.com/bsoc-bitbyte" 
  target="_blank" 
  rel="noopener noreferrer">
  mentored over at BSoC
  </a>, which is a programme ran by the Programming Club at IIIT Jabalpur, aimed at freshmen transitioning over to their sophomore year, with projects.
