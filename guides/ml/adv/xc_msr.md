---
last_modified: 2025-10-04 17:55:29 +0000
layout: ml-guides
title: Extreme Classification at Microsoft
description: A bigger question, can I ever get into MSR tho
usemathjax: true
---

# Extreme Classification by Microsoft Research

* Table of Contents
{:toc}
<hr style="border:1px solid gray">

What is extreme classification, and how is it different from traditional multi-label classification?

Similar to ordering a family meal at a restaurant from a menu, one needs to decide a subset of dishes that will be the most satisfying within the given budget.  
This is a much larger problem than traditional multi-class problems, where choosing a single dish from the menu is sufficient.  
Extreme classification opens up the avenue to think about query recommendation on search engines as a multi-label classification task.

- **Multi-class:** pick a class from \(L\) labels  
- **Multi-label:** pick the most relevant subset of these \(L\) labels  

With very large \(L\) (this notional value changes with time) we refer to this as **Extreme Classification**.

At this extreme scale, most fundamental machine learning techniques fail, as only partial answers exist for each question.

Initially, a tree-based classifier was used, but it was too slow for Bing's scale (note that Microsoft sponsored this research, so applications focused on Bing).

---

### Development of appropriate loss-functions

**Requirements:**
- Unbiased with respect to missing labels  
- Provide greater rewards for accurate prediction of rare labels (long-tail labels)

---

## How to speed things up: Slice

Slice stands for **Scalable Linear Extreme Classifiers** (not the best acronym but okay).  

Slice learns a linear classifier per label and improves upon simple tree-based methods by **10,000×**, leveraging the observation that only a small number of labels (usually logarithmic) are active in any given region of feature space.  

Given a test point, it determines which region of feature space it belongs to using approximate nearest neighbor search.  
It then evaluates the classifiers for only the labels active in that region, incurring just \(\log(L)\) prediction costs.

---

### During training

Given \(N\) points in \(D\) dimensions with \(L\) labels, we reduce complexity from

\[
O(N \times D \times L)
\]

to

\[
O(N \times D \times \log(L))
\]

by training each linear classifier on only

\[
O\left(\frac{N \times \log(L)}{L}\right)
\]

of the hardest negative examples rather than all \(O(N)\) negative examples.

This is done via a negative sub-sampling technique based on a generative model approximation of the discriminative linear classifier.  

This is particularly effective for low-dimensional features, where we can efficiently sub-sample a few hundred of the hardest negative samples from hundreds of millions of points.

---

## Application in ranking and recommendation problems

By treating each item to be ranked or recommended as a separate label, we can learn an extreme classifier that takes the user's feature vector as input, predicts the subset of relevant labels as output, and then returns the items corresponding to the predicted labels to the user.

This approach can be faster than Slice as well, as we only need to do approximate nearest neighbor search in feature space to get the top (K) neighbors.
