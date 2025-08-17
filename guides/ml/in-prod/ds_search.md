---
last_modified: 2025-07-10 21:19:02 +0530
layout: ml-guides
title: Positional Encoding for Transformers
description: well all transform blogs talk about the same 3 matrices, so here's positional encoding, something which is interesting to discuss as well
usemathjax: true
--- 

# DataScience for Search ?

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

7. Identify profanities / problematic words
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

## Precision and Recall in Search
Out of all results, how much is actually relevant ==> Precision <br>
Out of all relevant content, how much did we retrieve ==> Recall

<strong> Precision </strong>: out of all items shown, how many are relevant to the query, P@4, P@8 is a metric. <br>
<strong> Recall </strong>: out of all results, how many could my algorithm retrieve. This is a bit hard to compute, as getting the raw counts for all relevant items is tough.
	
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
	
### Ranking (can skip this section)

From the huge set of documents collected during the retrieval phase, we want to sort items based on a relevancy / recency score. <br>
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
* How do we evaluate Search results ?
Not sure, haha will have to read and learn
* Phrase completion / query suggestions

## Acknowledgements
1. <a href="https://www.youtube.com/watch?v=dqRDyeFJUvk&list=WL&index=11" target="_blank" rel="noopener noreferrer">Talk by Scott Hajek, PhD guy from New York</a>

2. Amazing book, have read the first 8 Chapters  
<a href="https://nlp.stanford.edu/IR-book/information-retrieval-book.html" target="_blank" rel="noopener noreferrer">Introduction to Information Retrieval by Manning</a>
	
3. Lots of documentation as well, haha
	
4. Zhihu, which is the Chinese version of Qoura
