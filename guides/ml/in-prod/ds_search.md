---
last_modified: 2025-10-04T19:40:10+05:30
layout: ml-guides
title: Data Science in search
description: basically what I work on, all of which seemed really cool and obscure when I started working as intern.
usemathjax: true
--- 

# DataScience for Search Systems ?

While I’m still early in my journey with e-commerce search systems, this note is a repository of search functionalities I could comprehend; which for the most of us, is beautifully abstracted as a black box. <br>
Search and Reco go hand in hand, so to draw a line we'll keep recommendations as the items that pop up separately either on the home page, or under a listing currently being viewed by the user. <br>
Search is user-initiated, while recommendations are system-initiated. Reco has to be **Near Real Time** (NRT), while Search can be a bit relaxed (okay if we don't show up items listed just moments ago).

<hr>

* Table of Contents
{:toc}

## Avenues for DS in Search
We can make 'search smarter' by targeting multiple avenues, discussed below : 

1. Indexing
Indexing items/documents (entire corpus of items) is commonly achieved with the help of an inverted index, which stores the docID in which a particular term appears. ![invertedIndex](https://hackmd.io/_uploads/SJR0P5xMll.jpg)

2. Understanding the Query terms
Extract the important parts from the query, pass it to ElasticSearch (StructuredSearch)
Highly unlikely that the userbase will be mono-lingual, this necessitates understanding of synonym pairs between languages. 

3. Understanding Query Intent
**Entity** Recognition : recognize famous names/places, *`not sure how this works`*, having a pre-defined set is mundane and would be required to be updated frequently. <br>
**Username** search ? <br>
Some platforms allow users to search for other users based on a unique userID/username. A small % of queries might not be marketplace-searches. <br>
Users might want to browse a popular store's page, to scour to new items. <br>
**Browsing Intent** : <br>
Category browse or general marketplace browse ? If we can associate the query with a particular (or a bunch of) category, items attached to the identified categories are boosted (ranked higher).

4. Retrieval and Ranking
Based on the query, get the most suitable items (retrieval) and rank these items based on relevancy and business needs (soft exponential decay based on time gives a lower score to older items).

5. Advanced Search
Query to be broken down into chunks, which are connected by boolean operators. <br>
**Exact match** : `"mango raw"` only returns items with the two words, we do not consider partial matches (mango juice, sweet mango are discarded) <br>
**Partial match** : `"rolex" green watch` should return all items having the term 'rolex' in title/description, with green watch being a preference, but not absolutely. <br>
**Negative match** : `"amul"  - milk` should return items that only match with the "amul" and not "milk" <br>
**Wildcard match** : `run* shoes` evaluates to all variants of the word 'run' followed by any character (running, runic, runner, runway)

6. Identify profanities / problematic words
Can have a dictionary of such words and display 0 results to avoid backlash / legal issues.

## +ve signals for marketplaces
Weights can be assigned to these signals to calculate a relevancy score for each <query-item> pair, 
	
* Transactions (strongest signal 😃 which brings in revenue)
* Add to cart (high purchase intent)
* Add to wishlist (user wants similar items, possibly waiting for a price drop)
* Quality Chat (`num_messages > threshold`, btw buyer and seller)
* Chat
* Like
* View

## metrics for Search
Click Through Rate of all forms are referred to as Search Success Metrics, these can be tracked by the total number of views/likes/add-to-cart/transactions; <br>
Or to measure of how useful the ranked results are, one can monitor the avg/p50/p90 of **listing-index interacted with**. This helps us keep a track of how good the results are.
If users consistently click on items with a high index, this signals poor ranking relevance.

### Precision and Recall in Search
Out of all results, how much is actually relevant ==> Precision <br>
Out of all relevant content, how much did we retrieve ==> Recall

**Precision** : out of all items shown, how many are relevant to the query, P@4, P@8 is a metric. <br>
**Recall** : out of all results, how many could my algorithm retrieve. This is a bit hard to compute, as getting the raw counts for all relevant items is tough.

### Mean Reciprocal Rank (MRR)

**Mean Reciprocal Rank** is the average of the reciprocal ranks of the first relevant item for a set of queries. The reciprocal rank for a query is the multiplicative inverse of the rank of the first correct answer ($1/rank$).

For a set of queries $Q$, the MRR is calculated as:
$$MRR = \frac{1}{|Q|} \sum_{i=1}^{|Q|} \frac{1}{rank_i}$$

Where $rank_i$ is the rank of the first relevant result for the $i^{th}$ query.

---

#### Example 1

If the rank of the first clicked item was $2, 3, 5$ for three different queries, the reciprocal ranks would be $\frac{1}{2}, \frac{1}{3}, \frac{1}{5}$ respectively.

The MRR is calculated as:
$$MRR = \frac{\frac{1}{2} + \frac{1}{3} + \frac{1}{5}}{3} = \frac{\frac{15+10+6}{30}}{3} = \frac{31}{90} \approx 0.344$$

---

#### Example 2 (Poor Ranking)

If the first clicked items were at ranks $10, 5, 4$, the reciprocal ranks would be $\frac{1}{10}, \frac{1}{5}, \frac{1}{4}$.

The MRR would be:
$$MRR = \frac{\frac{1}{10} + \frac{1}{5} + \frac{1}{4}}{3} = \frac{0.1 + 0.2 + 0.25}{3} = \frac{0.55}{3} \approx 0.183$$

---

### Mean Average Precision (MAP)

**Mean Average Precision (MAP)** is a popular metric used in information retrieval that builds upon precision. While precision measures the fraction of retrieved documents that are relevant, MAP evaluates the overall quality of a ranking system across a set of queries. It's the **mean** of the **Average Precision (AP)** scores over all queries in a test set.

An **Average Precision (AP)** score is calculated for a single query. It's the average of the precision scores calculated at each position where a relevant document is found in the ranked list. This rewards algorithms that place relevant documents higher up in the search results.

The formula for AP is:
$$AP = \frac{\sum_{k=1}^{n} (P(k) \times rel(k))}{\text{total number of relevant documents}}$$
Where:
- $n$ is the number of documents in the ranked list.
- $P(k)$ is the precision at rank $k$.
- $rel(k)$ is an indicator function that is 1 if the document at rank $k$ is relevant, and 0 otherwise.

---
#### MAP Example

Let's say for a query, there are **5 relevant documents** in the collection. Your ranking algorithm returns 10 results, with relevant documents (marked with **R**) at positions 2, 3, 5, 8, and 9.

1.  **Ranked Results**: [Irrelevant, **R**, **R**, Irrelevant, **R**, Irrelevant, Irrelevant, **R**, **R**, Irrelevant]

2.  **Calculate Precision at each relevant document**:
    - **Position 2**: 1 relevant out of 2 results so far. $P(2) = 1/2 = 0.5$
    - **Position 3**: 2 relevant out of 3 results so far. $P(3) = 2/3 \approx 0.67$
    - **Position 5**: 3 relevant out of 5 results so far. $P(5) = 3/5 = 0.6$
    - **Position 8**: 4 relevant out of 8 results so far. $P(8) = 4/8 = 0.5$
    - **Position 9**: 5 relevant out of 9 results so far. $P(9) = 5/9 \approx 0.56$

3.  **Calculate Average Precision (AP) for this query**:
    We take the average of the precision scores from the previous step.
    $$AP = \frac{0.5 + 0.67 + 0.6 + 0.5 + 0.56}{5} = \frac{2.83}{5} \approx 0.566$$

4.  **Calculate Mean Average Precision (MAP)**:
    To get the MAP, you would calculate the AP for many different queries and then take the average of all those AP scores.

---

### Normalized Discounted Cumulative Gain (NDCG)

Metrics like MRR and MAP are **highly dependent on the order** of results. They treat all relevant documents as equally important (binary relevance: 1 for relevant, 0 for not).

However, in many real-world scenarios, there are **degrees of relevance**. For example, in an e-commerce search, a product that a user purchased is far more relevant than one they just clicked on. **NDCG** is a metric for handling this "graded relevance".

The core idea is to assign a relevance score (gain) to each item and then "discount" that gain based on its position in the list. Items ranked higher should contribute more to the total score.

Here's how it's built, step-by-step:

1.  **Gain (G)**: This is simply the relevance score assigned to a document at a particular position $i$. For example, `relevance scores = [3, 1, 2, 0, 2]`.

2.  **Cumulative Gain (CG)**: This is the sum of gains up to a certain rank $p$. For the scores above, $CG_3 = 3 + 1 + 2 = 6$. This metric, however, is still insensitive to the order of results.

3.  **Discounted Cumulative Gain (DCG)**: To penalize results that appear lower in the list, we divide the gain at each position by a logarithmic factor. This heavily discounts the gain of items at lower ranks. A common formula is:
    $$DCG_p = \sum_{i=1}^{p} \frac{2^{rel_i} - 1}{\log_2(i+1)}$$
    Where $rel_i$ is the relevance score of the item at position $i$. The term $2^{rel_i} - 1$ is used to give more weight to highly relevant documents.

4.  **Ideal Discounted Cumulative Gain (IDCG)**: This is the maximum possible DCG score for your set of results, calculated by sorting the results by relevance in descending order (the "perfect" ranking).

5.  **Normalized Discounted Cumulative Gain (NDCG)**: Finally, the NDCG is the ratio of the DCG to the IDCG. This normalizes the score to a range between 0 and 1, where 1 represents a perfect ranking. This makes scores comparable across different queries.
    $$NDCG_p = \frac{DCG_p}{IDCG_p}$$

---
#### NDCG Example

Let's assume a query returns 5 documents with the following relevance scores in the given order: `[3, 1, 2, 0, 2]`

1.  **Calculate DCG for the given order**:

    | Position (i) | Relevance ($rel_i$) | Gain ($2^{rel_i} - 1$) | Discount ($\log_2(i+1)$) | DCG per item |
    | :----------: | :-----------------: | :--------------------: | :----------------------: | :------------: |
    | 1            | 3                   | 7                      | 1.000                    | 7.000          |
    | 2            | 1                   | 1                      | 1.585                    | 0.631          |
    | 3            | 2                   | 3                      | 2.000                    | 1.500          |
    | 4            | 0                   | 0                      | 2.322                    | 0.000          |
    | 5            | 2                   | 3                      | 2.585                    | 1.161          |
    
    The total **$DCG_5$** is the sum of the last column: $7.0 + 0.631 + 1.5 + 0.0 + 1.161 = \textbf{10.292}$.

2.  **Calculate IDCG (Ideal Order)**:
    First, we determine the ideal order by sorting the relevance scores in descending order: `[3, 2, 2, 1, 0]`. Now, we calculate the DCG for this perfect ranking.
    
    | Position (i) | Ideal Relevance ($rel_i$) | Gain ($2^{rel_i} - 1$) | Discount ($\log_2(i+1)$) | IDCG per item |
    | :----------: | :-----------------------: | :--------------------: | :----------------------: | :-------------: |
    | 1            | 3                         | 7                      | 1.000                    | 7.000           |
    | 2            | 2                         | 3                      | 1.585                    | 1.893           |
    | 3            | 2                         | 3                      | 2.000                    | 1.500           |
    | 4            | 1                         | 1                      | 2.322                    | 0.431           |
    | 5            | 0                         | 0                      | 2.585                    | 0.000           |
    
    The total **$IDCG_5$** is the sum of the last column: $7.0 + 1.893 + 1.5 + 0.431 + 0.0 = \textbf{10.824}$.
    
3.  **Calculate NDCG**:
    $$NDCG_5 = \frac{DCG_5}{IDCG_5} = \frac{10.292}{10.824} \approx \textbf{0.951}$$
    
    An NDCG score of 0.951 indicates that the algorithm produced a very good ranking, close to the ideal order.

---

## Indexing
Pretty simple<br>
For each word in our corpus, store the docIDs a particular term shows up in with the term-frequency in the doc.<br>
This creates a postings list / inverted index as we saw in the first image.

## Understanding the Query
Before we try to analyze the query, we need to normalize it by converting everything to the same case (usually lowercase) and then stem/lemmatize to get the root word.

| Stemming                         | Lemmatization                          |
|----------------------------------|----------------------------------------|
| Crude process that chops off suffix | Get to the morphological root of a word |
| Easy, quick, based on rules         | Based on a vocab, computation heavy      |
| personalization ⇒ personalize      | personalization ⇒ person                 |

Arriving at the root word, we split our phrase into tokens.

### Query Expansion
Have a learned synonym map across languages and append synonyms to the search query before passing it to ElasticSearch; which will simply perform a union of posting lists of all the query terms to obtain all items in our index that have any of these terms.<br>
Increases number of results.

### Attribute / Structured Search ?
Allows the search bar to work with natural language queries which is seamless as compared to a standard keyword search function wherein the user has to fill in values for attributes he's looking for.<br>
Critical for Fashion / Electronics wherein attributes are specified with the query itself
- iPhone black 16gb
- nike shoes under 3500
- red football shoes 12
	
Atrtribute keys are not mentioned, but the values are passed in the query itself.<br>
This increases number of search results and therefore the avg impressions per query, as we can further include relevant items that match attributes.<br>
Alongside as retrieved results are more relevant, we should observe an increase in num_items viewed/clicked through.<br>
We pass the query to a QueryUnderstanding Service, which performs Entity Recognition and filters out attribute-value pairs for us, that we pass onto ElasticSearch 

## Ranking and Retreival
We want to get the matching-results from our database to present to the user.

### Retrieval
Collect (mostly) relevant documents/items, this can be a massive collection.  <br>
Boolean retrieval is fast, just perform union/intersections of the query terms, usually unions are preferred to improve number of results

Returns a pool of items that user could be interest in, high recall and high speed (low latency). Often having a dumb simple architecture.
	
### Ranking (can skip this section)

From the huge set of documents collected during the retrieval phase, we want to sort items based on a relevancy / recency score. <br>

Focus on accuracy, making use of more features (relative to the retrieval stage). Multiple retrieval - ranking blocks are often used, gradually narrowing down on the set of results.

These methods ignore the order of words in a document, and hence referred to as a 'bag of words' concept.

--- 
#### Term Frequency
Number of occurences of a term in the document <br>
Well not all words in a document are equally important, there are some statistically boring words as well. How do we identify these ?
	
#### Inverse Document Frequency
Certain words have no discriminating power, they exist in a lot of documents and do not help us in determining relevance. <br>
Here, `N` is the total number of documents, and `df` is the number of documents containing the term.

$$IDF= {N \over df }$$

For a large corpus, we usually take the log of this

#### tf-IDF score
We would like to score documents higher that have query terms appear more times and these terms should not appear a lot in the corpus overall. <br>
Words like `and`, `the` would appear in all documents and provide us with no relevance.

$$ tf_-idf = TF * log{N \over df}$$

* Have a higher score when term appears many times within a small number of documents,
* Lower score when the terms appears fewer times in a doc or occurs in many documents.
* Lowest when the term occurs in almost all documents.
	
## Things I don't understand well enough
* Comparing document Vectors with the Query Vectors using cosine similarity is intuitive, but how do we generate the docVectors.
* Phrase completion / query suggestions

## Acknowledgements
1. <a href="https://www.youtube.com/watch?v=dqRDyeFJUvk&list=WL&index=11" target="_blank" rel="noopener noreferrer">Talk by Scott Hajek, PhD guy from New York</a>

2. Amazing book, have read the first 8 Chapters  
<a href="https://nlp.stanford.edu/IR-book/information-retrieval-book.html" target="_blank" rel="noopener noreferrer">Introduction to Information Retrieval by Manning</a>
	
3. Lots of documentation as well, haha
	
4. Zhihu, which is the Chinese version of Qoura

5. Metrics for search, well interviews and going through AB experiment reports :)
