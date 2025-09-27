---
layout: ml-guides
title: Extreme Classification at Microsoft
description: A bigger question: can I ever get into MSR tho
usemathjax: true
---

# Extreme Classification by Microsoft Research 

* Table of Contents
{:toc}
<hr style="border:1px solid gray">

What is extreme Classification though, how is it any different from traditional multi label classification

Similar to ordering a family meal at a restaurant from a menu, one needs to decide a subset of dishes that will be the most satisfying within the given budget.
This is a much harger problem than traditional multi class problems, where choosing a single dish from the menu is sufficient.
Extreme classification opens up the avenue to think about query recommendation on search engines as a multi label classification task.

Multi class pick a class from L labels
Multi label pick the most relevant subset of these L labels. With very large L (this notional value changes with time) we refer to this as Extreme Classification.

At this extreme scale, most of fundamental machine learning techniques go awry as only partial answers for each question exist.

Initially a tree based classifier was used, but this was too slow for Bing's scale. (note that microsoft sponsored this research so applications would be focused on bing)

<h3>
Development of appropriate loss-functions 
</h3>

Requirements
- unbiased with respect to missing labels 
- provide greater rewards for the accurate prediction of rare labels, or in other words long tails

<h2> 
How to speed things up:> Slice
</h2>
Slice stands for Scalable Linear extreme Classifiers (not the best acronym but ookay).

Slice learns a linear classifer per label 

It improves upon simple Tree based methods by 10,000times by making use of the observation that only a small number of labels (usually logarithmic) are acrive in any given region of feature space. 
Given a test point, it determines which region of feature space it belpng to based on approxiamate nearest neightbor search.
It then evaluates the classifiers for only the labels acrie in the region therebby incurring just log prediction costs.

<h3>
During training
</h3>
Given N points in D dimensions with L labels, we reduce complexity from 

$O(~N \times D \times L~)$ 

to 

$O(~N \times D \times log(L)~ )$

by training each linear classifier on only 

$ O(~N \times D \times L) = O(~ \frac{N \times log(L)} {L} ~) $ 

of the hardest negative examples rather than all O(N) negative examples.

This is done via a -ve sub-sampling technique based on a generative model approximation of the discriminative linear classifier.

This is particularly effective for low-dimensional features for which we can efficiently sub sample a few hundred of the hardest -ve samples from hundreds of millions of points.

<h2>
Application in ranking and reco problems
</h2>
By treating each item to be ranked or recommended as a separate label, we can learn an extreme classifier that takes the user's feature vector as input, predicts the subset of relevant labels as output
and then return the items correspoding to the predicted labels to the user
Note that is can be faster than slice as well, as we just have to do a approxiamate nearest neigbbours space in the feature space to get top K neighbours.