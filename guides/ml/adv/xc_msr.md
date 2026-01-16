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

- **Multi-class:** pick a single class from \(L\) labels  
- **Multi-label:** pick the most relevant subset of these \(L\) labels  

When \(L\) becomes very large (this threshold keeps changing over time), we refer to the problem as **Extreme Classification**.

At this extreme scale, most fundamental machine learning techniques start to fail, mainly because only partial supervision exists for each data point.

Initially, a tree-based classifier was used, but it turned out to be too slow for Bing-scale workloads  
(note that Microsoft sponsored this research, so most applications are framed around Bing).

---

### Development of appropriate loss functions

**Requirements:**
- Unbiased with respect to missing labels  
- Should reward correct predictions of rare (long-tail) labels more strongly  

---

## How to speed things up: Slice

Slice stands for **Scalable Linear Extreme Classifiers** (not the best acronym, but okay).  

Slice learns a *linear classifier per label* and improves upon tree-based methods by **~10,000×**, based on the observation that only a small number of labels—usually logarithmic in \(L\)—are active in any given region of feature space.

Given a test point, Slice first determines which region of feature space it belongs to using approximate nearest neighbor search.  
It then evaluates classifiers only for the labels that are active in that region, resulting in a prediction cost of roughly \(\log L\).

---

### During training

Given:
- \(N\) data points  
- \(D\) feature dimensions  
- \(L\) labels  

the training complexity is reduced from

$$
\mathcal{O}(N \cdot D \cdot L)
$$

to

$$
\mathcal{O}(N \cdot D \cdot \log L)
$$

This is achieved by training each linear classifier on only

$$
\mathcal{O}\left(\frac{N \cdot \log L}{L}\right)
$$

negative examples, instead of all $\(\mathcal{O}(N)\)$ negatives.


In practice, these negatives are chosen as the *hardest* ones, using a negative sub-sampling strategy derived from a generative approximation of the underlying discriminative linear classifier.

This works especially well for low-dimensional feature spaces, where it is feasible to efficiently sample a few hundred hard negatives from datasets containing hundreds of millions of points.

---

## Application in ranking and recommendation problems

By treating each item to be ranked or recommended as a separate label, we can train an extreme classifier that:
- takes a user feature vector as input  
- predicts a subset of relevant labels  
- returns the corresponding items to the user  

In some cases, this approach can be even faster than Slice, since we only need to perform approximate nearest neighbor search in feature space to retrieve the top-\(K\) neighbors.
