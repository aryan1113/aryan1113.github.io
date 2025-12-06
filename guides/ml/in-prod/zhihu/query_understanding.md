---
layout: ml-guides
title: "Query Understanding for Search"
description: "shoutout to Zhihu"
usemathjax: true
---

# Query Understanding for Search

* Table of Contents
{:toc}

<hr style="border:1px solid gray">

## Made up of multiple blocks

To transform a raw, noisy user string into useful string that can be better used to serve our user.

**Standard Flow:** <br>
`Raw Query` $\rightarrow$ `Correction` $\rightarrow$ `NER/Tagging` $\rightarrow$ `Intent Class` $\rightarrow$ `Weighting` $\rightarrow$ `Expansion` $\rightarrow$ `Retrieval`

1.  **Pre-processing:** Cleaning query, spell correction.
2.  **Structural Analysis:** Named Entity Recognition (NER).
3.  **Semantic Analysis:** Intent Classification, Term Weighting.
4.  **Enrichment:** Query Expansion, Rewriting.

## Query Rewriting & Correction

This module handles input errors to ensure downstream tasks receive valid tokens.

### Error Classification
* **Homophones:** Same pronunciation, different meaning (e.g., *piece* vs *peace*).
* **Ambiguous Sounds:** Phonetically similar but not identical.
* **Visual Similarity:** Character stroke similarity (common in OCR or CJK {CJK refers to Chinese Japanese Korean} languages).
* **Non-word Errors:**
    * Permutations: `fiappy` $\rightarrow$ `flappy`
    * Numeric: `2408` $\rightarrow$ `2048`
* **Word Errors:**
    * Omission: `apple 17` $\rightarrow$ `apple iphone 17`
    * Reversal: `Throne of Games` $\rightarrow$ `Game of Thrones`

### Algorithmic Approach: HMM & Viterbi

<details markdown="1" style="border:1px solid; padding:10px; border-radius:6px;">
<summary style="font-weight:bold; cursor:pointer;">
    Term Weighting: Math Derivations (Click to expand)
</summary>

For error correction, we model the problem using a **Hidden Markov Model (HMM)**. <br>
We treat the **intended correct words** as the *hidden states* and the **user's typed query** as the *observations*.

We aim to find the sequence of intended words $W = (w_1, w_2, ..., w_n)$ that maximizes the probability of the observed query $O = (o_1, o_2, ..., o_n)$.

$$
\hat{W} = \underset{W}{\text{argmax}} P(W | O)
$$

Using Bayes' Theorem:

$$
\hat{W} = \underset{W}{\text{argmax}} \frac{P(O | W) P(W)}{P(O)}
$$

Since $P(O)$ is constant for ranking candidates, we maximize:

$$
\hat{W} \approx \underset{W}{\text{argmax}} \prod_{i=1}^{n} \underbrace{P(o_i | w_i)}_{\text{Emission}} \cdot \underbrace{P(w_i | w_{i-1})}_{\text{Transition}}
$$

**Emission Probability**:  
$P(o_i \mid w_i)$  
The probability that a user types $o_i$ when they intended $w_i$.  
This models the “Noise Channel” (e.g., keyboard layout distance, phonetic edit distance).

**Transition Probability**:  
$P(w_i \mid w_{i-1})$  
The probability of word $w_i$ following $w_{i-1}$.  

</details>
<br>

## Named Entity Recognition (NER)

NER transforms unstructured text into structured slots. This is critical for queries with multiple constraints (e.g., travel or e-commerce).

**Example:** *"mumbai bangalore evening flight"*

Without NER, this is a bag of words. With NER, the query is parsed into a structured object:

| Token | Entity Tag | Semantic Role |
| :--- | :--- | :--- |
| **mumbai** | `LOC_CITY` | `origin` |
| **bangalore** | `LOC_CITY` | `destination` |
| **evening** | `TIME_RANGE` | `departure_time` |
| **flight** | `CATEGORY` | `product_type` |

**Approaches:** <br>
Not really sure, will need to read more of it.
* **Linear CRF (Conditional Random Fields):** Classic approach for sequence labeling. <br>
* **Bi-LSTM-CRF / BERT:** To capture bidirectional context.

## Intent Recognition

To determine the user's goal. Deals with ambiguity where one term maps to multiple domains <br> (e.g., "Apple" $\rightarrow$ Fruit vs. Electronics).

### Types of Intent
1.  **Precise Intent:** 
<br> High confidence, usually specific entity searches (e.g., "iPhone 15 Pro Max"). 
<br> Long-tail queries fall here and often suffer from sparse behavioral data.

2.  **Ambiguous Intent:** 
<br> Broad queries (e.g., "best gifts").
    * *Strategy:* Treat as a Multi-Label Classification task.
    * *Features:* Correlate query terms with clicked product categories from historical logs.

## Term Importance (Weighting)

Not all words in a query carry equal weight. Can classify terms into:
1.  **Core Terms:** Essential for retrieval (removal changes intent).
2.  **Qualifiers:** Refine the search (e.g., colors, sizes).
3.  **Noise:** Stop words or conversational filler.

### Statistical Metrics for term importance

**1. Neighbor Entropy**
Measures the diversity of terms that appear to the left/right of a target term.
* **High Entropy:** The term appears in many contexts 
<br> e.g., "iPhone" can be followed by case, charger, screen. $\rightarrow$ **Core/Ambiguous**.
* **Low Entropy:** The term has very few valid neighbors 
<br> e.g., "Defragmenting" is almost always followed by "disk". $\rightarrow$ **Specific/Niche**.

**2. Independent Search Ratio**
Quantifies how often a term is searched alone versus as part of a phrase.

$$
\text{ISR}(t) = \frac{\text{count}(t)}{\text{count}(t \in \text{supersets})}
$$

* Example: `troubleshooting`. If users rarely search this word alone, but always search `router troubleshooting`, the *ISR* is low, indicating it is a dependency term.

### Embedding-Based

To determine term $t$'s importance in query $Q$, we can compare the semantic vector of the full query against the query without the term.

**Method A: Naive Subtraction**
$$
\text{Score} = || \vec{V}_{Q} - (\vec{V}_{Q} - \vec{V}_{t}) ||
$$
* *Logic:* Subtract term vector from pooled query vector. Large resultant change implies high importance.
* *Latency:* Ultra-low. Simple element-wise operation.
* *Drawback:* Vector subtraction in high-dimensional space often yields geometrically meaningless results in semantic models (like BERT).

**Method B: Cosine Similarity**
$$
\text{Importance} = 1 - \text{CosineSimilarity}(\vec{V}_{Q}, \vec{V}_{Q \setminus \{t\}})
$$
* *Logic:* Embed the full query. Embed the query *excluding* term $t$. Calculate similarity.
    * High Similarity $\rightarrow$ Term $t$ didn't add semantic value (Noise).
    * Low Similarity $\rightarrow$ Term $t$ changed the meaning significantly (Core).
* *Latency:* Higher than subtraction (requires Dot Product and Norm calculations). 
<br> However, for short queries ($<10$ tokens), the overhead might be negligible.

## Query Normalization & Synonyms

**Goal:** Align user vocabulary with catalog SKUs (e.g., `sneakers` $\rightarrow$ `shoes`).

**Challenge:** Distinguishing "Synonyms" (semantic equivalents) from "Related Concepts".
* *Synonym:* Cell phone $\leftrightarrow$ Mobile Phone.
* *Related:* iPhone $\leftrightarrow$ Apple.

**Solution: Cosine Thresholding & Graph Constraints**
Mining synonyms via simple co-occurrence or embedding similarity often captures "Related" terms erroneously.
1.  **Strict Thresholds:** Set `Cosine Similarity > 0.K` for synonyms, `0.slighty_less_than_K - 0.K` for related terms.
2.  **Knowledge Graph:** Enforce constraints. If `Term A` and `Term B` share the same parent node in a taxonomy (e.g., both are children of `Brand`), they are likely *siblings/substitutes*, not synonyms.

## Query Suggestions

As the request volume is massive, suggestions are usually pre-computed, using a prefix tree / trie. For retrieval we can store this in a Top-K Min Heap.

**Ranking Criteria:**
1.  **Historical Volume:** What did others search? (Bias: Hard for new trends to surface).
2.  **CTR (Click-Through Rate):** If a suggestion leads to a result page with valid clicks, boost its score.

<hr style="border:1px solid gray">

## Acknowledgements
- Based on concepts from [Zhihu Blog - Query Understanding](https://zhuanlan.zhihu.com/p/112719984)
- Nothing else honestly, blogs on Zhihu are quite comprehensive, if you can somehow choose to ignore their 10 thousand sign up requests.
