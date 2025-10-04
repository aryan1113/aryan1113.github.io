---
last_modified: 2025-07-10 17:55:29 +0000
layout: ml-guides
title: Long Tailed Visual Recognition, Tsinghua University
description: Amazing paper, flows smoothly, why not give this a shot
usemathjax: true
---

# Long Tailed Visual Recognition

As part of an undergrad course on PRML, I had the opportunity to present this to the class. 
<a href="https://docs.google.com/document/d/1fgFY8qZcP4lgbgRmLKfxIe1CewoPNhDTT2sxper8fsg/edit?usp=sharing" target="_blank" rel="noopener noreferrer">
This document
</a>
might be unavailable, linked to a uni account.

* Table of Contents
{:toc}

## What are Long Tailed Distributions ?
We have a long tail when the frequency for a large number of classes is low and the data is concentrated only in a few classes. Data in the tail classes is often insufficient to represent the true distribution. When a class is severely underrepresented, it becomes difficult to determine the decision boundary in our parameter/search space.

<figure style="text-align: center;">
  <img src=
  "https://miro.medium.com/v2/resize:fit:440/format:webp/1*vApXs8oGzkbt66RwtFby4A.jpeg" 
  alt="Illustration" style="max-width: 100%;" />
  <figcaption><em>Most examples are from few classes, and a lot of classes do not have a lot of samples</em></figcaption>
</figure>

Data quality affects performance of our learner, and due to data being dominated from few classes, the learning of tail classes is severely underdeveloped.

<figure style="text-align: center;">
  <img src=
  "https://miro.medium.com/v2/resize:fit:440/format:webp/1*0yVxsUW-aycDORcahypr7Q.png" 
  alt="Illustration" style="max-width: 100%;" />
  <figcaption><em>
  common classes like cat-dog wil have million images as acquiring data is easy, but examples for whales / sea horses will be lesser in number due to tougher data acquisition</em></figcaption>
</figure>


## How to measure long tailed-ness of a distribution ?
<ol>
<li> <strong>Imbalance factor</strong> : ratio of maximum number of samples in a class and minimum number of samples in a class </li>
<li> <strong>Standard deviation</strong> : difficult to objectively express, as it is relative</li>
<li> <strong>Mean / median ratio</strong> : reflects skew</li>
<li> <strong>Gini coefficient</strong> : measure of inequality <br>
    0 perfect equality ; 1 one class has all samples <br>
    Differences btw i and j, n is the total number of samples and μ is the mean of the distribution
</li>
</ol>

$$
\sum_{i=1}^{n} \sum_{j=1}^{n} \left| x_i - x_j \right| \over 2n^2 ~\mu
$$ 

<br>
Using conventional methods for learning the distributions often result in poor performance as these assume that the data (both train and test) satisfy the I.I.D (independent, identically distributed) condition.

## Naive methods to treat Long Tailed Distributions
These methods are simple, data processing based, more of these are explored in the paper “A survey on Long Tailed Visual Recognition” by the Beijing University of Posts and Telecommunications.

Aim of these methods is to eliminate / minimize the imbalance between head and tail classes.

1. **Over sampling**
Increase the instance number of the tail classes. <br>
Class Aware sampling ensures that the probability of occurrence of each class is the same in each training batch. <br>
Probability of each class is 1/C, where C is the number of classes in our entire set.

2. **Under sampling**
Decreases the instance number of head classes. <br>
Random undersampling randomly removes instances of the head classes. <br>
For long tailed distributions, we loose a lot of information because of the large difference between the head and tail classes. <br>

<strong> Drawbacks </strong> :
a. May lead to overfitting, as we reduce overall size of our set
b. Effects of noise / other defects are exaggerated
c. If we remove too many instances, we risk not learning anything, due to under learning of the head classes as well.

3. Data Augmentation
Generate and synthesize new samples from the tail classes. <br>
Some common ways to augment image data are. <br>
Image flipping, Scaling (zoom), Rotating and cropping. <br>

### SMOTE is a common method for data synthesis for the minority class.
<ol>
<li>For each sample in the minoriotty class, identify k-nearest neighbours within the same class based on distance.</li>
<li>Randomly choose a neighbour and interpolate a new datapoint using the formula, where λ is between 0 and 1</li>
x_newsample = x + λ ( x_neighbour — x )

<figure style="text-align: center;">
  <img src=
  "https://miro.medium.com/v2/resize:fit:440/format:webp/0*f0uAUIJ4Jp0JZdZb" 
  alt="Illustration" style="max-width: 100%;" />
  <figcaption><em> </em></figcaption>
</figure>
</ol>

## What are Contrastive pairs ?

Samples used to help the learner make decisions, what’s similar and what’s different. <br>
There could be positive samples, i.e similar to the class we are talking about. The learner tries to push these closer together in the feature space. <br>
Negative samples are ones different from the current class, and the learner tries to push these apart in the feature space. <br>
For Supervised Learning approaches,
- Positive pairs: Any two samples from the same class label
- Negative pairs: Any two samples from different class labels

## Logit Adjustment 
modify the log loss function to account for frequency of the class as well. <br>
In the usual log loss, we treat all classes as equals and hence the learner is biased towards reducing loss majorly for the dominant class. Factoring in the frequency of the classes help the learner also focus on rarer classes (present on the tail). <br>
For class i, if the score at the output layer is z_i, the class probability is given by :

$$ p_i = {e^{z_i} \over \sum e^{z_i}} $$


$$ 
\text{For adjusted logit, use } z_i = z_i - \lambda \log(f_i)
$$

We normalize all attributes, by dividing each value with the norm of the attribute.


## von Misher Fisher vMF distribution

Equivalent of a Gaussian distribution; for data on a sphere/hypersphere where all points lie on the surface of a unit sphere (length = 1). <br>
Defined by two parameters:
- $\mu$ : mean direction
- kappa: concentration parameter; how tightly clustered is the data

Imagine spreading butter on a sphere: <br>
$\mu$ tells you the center point of where you’re spreading <br>
K tells you how widely you spread it <br>
Higher K = more concentrated (like cold butter) <br>
Lower K = more spread out (like melted butter)

Notice the bessel function Ip/2–1 term in Cp(k)

<figure style="text-align: center;">
  <img src=
  "https://miro.medium.com/v2/resize:fit:440/format:webp/0*ZVG4YykYGCZY6e2E" 
  alt="Illustration" style="max-width: 100%;" />
  <figcaption><em>Probability density function for a vMF distribution</em></figcaption>
</figure>


#### Visual Analogy
<figure style="text-align: center;">
  <img src=
  "https://miro.medium.com/v2/resize:fit:1100/format:webp/0*bQmHzBaSRKeDuWdI" 
  alt="Illustration" style="max-width: 100%;" />
  <figcaption><em>A higher kappa shows data is concentrated in a small region</em></figcaption>
</figure>

## proCO; solution proposed

<figure style="text-align: center;">
  <img src=
  "https://miro.medium.com/v2/resize:fit:560/format:webp/1*m3A3YZX4IzgxDULxOvrHEw.png" 
  alt="Illustration" style="max-width: 100%;" />
  <figcaption><em>The two approaches</em></figcaption>
</figure>


### Issues with Supervised Contrastive Learning
- Long Tailed distributions make it even harder to have +ve samples, as for rarer classes we might need very large batches to get adequate number of samples.

- If batch is too small, we might <br>
    <ol type="a">
    <li>not have examples with the same labels and </li>
    <li>miss out on important -ve examples</li>
    </ol>
- Having a large batch causes <br>
    <ol>
    <li> longer training time </li>
    <li> higher memory usage </li> 
    <li> higher computational cost </li> 
    </ol>

### Parameter Estimation
Here fp is the probabiliy distribution function of a vMF, and $\pi$y is the probability of class y. <br>
We estimate the mean and kappa params of the feature distribution.

$$ P(z) = \Sigma ~P(y) ~P(z\mid y) = \Sigma ~\pi _y * f_p $$

### Loss Function
To compute higher order bessel functions, make use of recurrence relation; which is numerically unstable with lower values of kappa, the concentration parameter

$$
I_{v+1}(k) = {2~v\over k} I_v(k) - I_{v-1}(k)
$$

### Two Branch Design
Imagine a tree, with the root feeding into two branches

1. Classification Branch
    Uses Linear classifier, optimized with Adjusted Logit
    Learns to classify

2. Representation Branch
    Uses neural network, optimized with ProCO
    α is the strength parameter
    Learns what the good features are
The two branches handle rare cases without sacrificing performance for examples from common classes <br>

Final Loss function 
$
L = L_{\text{adjusted logit}} + \alpha L_{\text{ProCO}}
$

## Some relevant details
<ol type="a">
<li>I got to know about Long Tailed Distributions from an introductory video fo the Vision and AI Lab at IISc </li>

<li>The ProCo paper is from Tsinghua University, which is the Chinese equivalent of IISc</li>
<li>While reading through a literature review paper, I identified a small error in the paper and even mailed the authors regarding the same.</li>

<figure style="text-align: center;">
  <img src=
  "https://miro.medium.com/v2/resize:fit:828/format:webp/1*HUXq8l9dia-tqDKagZEiLw.png" 
  alt="Illustration" style="max-width: 100%;" />
  <figcaption><em>This is incorrect, as Gini coefficient is a measure of inequality.</em></figcaption>
</figure>

A coeff of 1 signifies complete inequality with one class having all the examples, while 0 signifying no inequality => complete equality, all classes have the same number of samples. So a smaller Gini Coeff should point to a balanced dataset.

</ol>

### References
1. Probabilistic Contrastive Learning for Long-Tailed Visual Recognition by Tsinghua University
