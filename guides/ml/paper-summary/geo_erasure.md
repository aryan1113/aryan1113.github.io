---
layout: guides
title: Geographical Erasure in Language Generation, from KIAC IISc
description: do LLM outputs ignore certain countries ? From the Kotak IISc AI-ML Center
usemathjax: true
---

# Geographical Erasure in Language Generation

The 'Geographical under-representation' 
(
<a 
href="https://medium.com/@aryan1113/geographical-under-representation-in-text2image-943a2fc148db" 
target="_blank" 
rel="noopener noreferrer">
paper I read recently
</a>
)
, got me intrigued with "Erasure" and read 
(
<a 
href="https://arxiv.org/pdf/2310.14777" 
target="_blank" 
rel="noopener noreferrer">
about Geographical Erasure in Language Generation Models (this paper)
</a>
) recently. Another paper from KIAC, IISc (wow, Kotak funds stuff other than finance-driven initiatives).

<hr>

* Table of Contents
{:toc}

<hr>

## Why do we care about Erasure
From the previous paper
> Due to inherent biases of the way we collect data, we can over-represent certain nations and result in an hegemony (dominance of few groups). Adding to this, User Experience would be unpleasent for users residing in these places.

Erasure : minimizing cultural and geographical identities
Think of this in an historical context, where we do not get to hear about the working class as compared to the rich and influential.

## In this paper
	
1. Comparing model outputs to population numbers, to quantify erasure.
2. Impact of model size ?
3. Causes of erasure ?
4. Mitigating erasure

## How is Erasure measured
### Obtain Model Predictions

For prompt c, X being the set of all countries and xᵢ is the country we analyze right now; the authors compute the probability of country xᵢ given prompt c as (simply Bayes rule) : 

$$ p(x_i | c) = {p(x_i, c) \over p(c) } = {p(x_i, c) \over \Sigma_{x\in X} p(c) } $$

Compute p("I live in $x_i$") for all candidate countries and normalize. If a country_name is broken down into multiple tokens, just multiply the probablity of J subtokens.
This happens predominantly for low-GDP countries

### Obtain Ground Truth
To measure erasure, we need to compare the model-output distribution with the ground truth distribution over candidate countries.

As models are trained majorly on an English corpus, and the methods described in the paper only pertain to English words, only the English speaking population of a country is considered. 

Note that model-outputs are prompt dependent whereas the ground truth isn't, which is why we [rephrase prompts a bit later](#prompt-rephrasing)

## Measure Erasure (the fun part)
Define an Erasure Set, for model p and ground truth p^true. Note that r > 1, being the case for erasure, where we fail to represent the country in our model output an equal number of times of the ground truth.

$$ S^c_r = x_k : {p^{true}(x_k) \over p(x_k | c) } > r  $$

r can be understood as an <strong>under-representation factor</strong>, higher r would give us countries facing high erasure. <br>
For r = 3, we get countries that are three times more prevalent in the ground truth than in model-predictions. <br>
A very naive erasure metric could be the number of elements in our erasure set.

### Metric for Erasure : ER

$$ ER^r (p^{true}, p) = \Sigma p^{true}_i log {p^{true}_i \over p_i } $$

The authors justify the selection of the Erasure metric as 
1. **Zero** Erasure case
	For p_true = p, when our model outputs (p) exactly match with the ground truth (p_true), we have ER = 0
2. **Log based** to scale population
The metric should be sensitive to relative rather than absolute errors, to also consider smaller countries, which is why we use a log-ratio.
3. **Penalize** erasure more **for large countries**
Under-predicting countries with large population is more harmful as it affects more users of these ML systems. Hence we multiply by p_true
4. Subset of **KL Divergence**
ER is an addititive component of KL-divergence, this close relation 

$$ KL(p^{true}_i ||~p) = \Sigma_{i \in S_r}     ~~p^{true}_i~log{p^{true}_i \over p_i} + \Sigma_{i \in X \setminus S_r}     ~~p^{true}_i~log{p^{true}_i \over p_i} $$

The first term is our Erasure metric, second one just computes divergence between groundTruth and byModel for `i not in set Sr`

<strong>Differentiability</strong>
For a fixed r, ER is differentiable everywhere except for points when we add new countries to the erasure set. The function has singularity points (where function ceases to be well-defined / or exist).

### Choice of r
Crucial hyperparam, as our erasure set Sr and ER both set in terms on r.

$$ r < {ground~truth \over by~model} $$

For **low values of r**, we cover most of the countries (except the ones that are over-represented and would have a ratio less than 1).
For **higher values of r** we only have countries with high erasure in our set Sr, or possibly none (null set).<br>
Here we only want to understand cases of under-representation, hence we limit our study to r > 1 (counts in groundTruth being bigger than counts byModel). Pick an integral value of r, such that ER(p_true, p) ≈ KL(p_true, p).

<figure style="text-align: center;">
  <img src="
    https://hackmd.io/_uploads/ryEtD2FMxx.png
  " alt="Illustration" style="max-width: 100%;" />
  <figcaption><em>
  Graph on left shows the variation of our Erasure Set Sr with r
Graph on right evaluates the KL-divergence (right) and ER-metric (blue) by varying r
</em></figcaption>
</figure>

### Prompt Rephrasing

Erasure set and hence our notion of erasure (by metric ER) are dependent on r, but we are interested in how the model behaves universally instead of on a specific set of prompts.
Hence the authors try to aggregate effects for all prompts encoding the meaning "home country". A very simple and clever solution is to use a <strong>seed prompt</strong> using which we construct a set of sample prompts. Using 16base prompts, the authors arrived at <strong>955 prompts</strong>, by subtly changing modifying prompts.

From `I live in`, we can arrive at `You live in`, `She lives in`, `He lives in` and so on...

## Cool inferences

### Poor countries suffer

For the 22 countries that suffer from, we just sort countries based on GDP and color the bars based on (count of) how many models erase this. 

<figure style="text-align: center;">
  <img src="
    https://hackmd.io/_uploads/HJ4CxCKfgg.png
  " alt="Illustration" style="max-width: 100%;" />
  <figcaption><em>
Bars are coloured according to counts and sorted by GDP per capita (decreasing from left to right)
</em></figcaption>
</figure>

### Negligible impact of model size
Previous work found that larger models exhibit more stereotyping (better representation of bias in train data), and smaller models do not exhibit subtle biases. <br>
Major stereotypes can be better understood by smaller models as well (just like major patterns in train-distribution can be understood by models easily, and the subtle patterns need more effort to be learned). <br>
For erasure, the authors do not find model size to have any impact.

### Train data impacts everything
To compute the probability of occurrence in training data, we compute the number of times each country was mentioned in the dataset and then weighed by the number of training epochs this document was included in.

It is observed that countries experiencing erasure are indeed under-represented in the training data and the prediction probabilities are similar to their frequency distribution in the training corpus. <br>
Erasure score of training data also closely matches the Erasure score for models trained on this data.

## Mitigation (not sure how this is done)

Not sure how this is done, the authors try a constrained optimization of sorts, that aims to reduce the ER (Erasure score) while not impacting language modelling abilities (measured as perplexity).

### New terms I encountered
1. Warmup epoch
2. Perplexity
3. Masked models
	Is this somehow related to the Language Generation method where we masks some tokens in a sentence and try to predict the token in the middle using the surrounding tokens (in a context window) ?
	
<hr>
