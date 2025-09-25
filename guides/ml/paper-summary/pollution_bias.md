---
last_modified: 2025-08-20 11:38:14 +0000
layout: ml-guides
title: Biased media reporting on Air Pollution
description: Another paper I read while applying for the Sustainability Lab at IITGn, talks about how media attention is concentrated to the urban centers of Delhi NCR.
usemathjax: true
---

# Analysing bias in media reporting 

If you're the author, please scroll directly to the [Suggestions section](#Suggested-Future-Work).

>
Here 'we' refers to the authors, it just fits my writing style, I have in no part contributed to the paper and full credits to the authors.

This week as I was travelling to campus for the last time, for PBI presentations, I had a ton of time to read some interesting papers. One of them is from IITGn, 
(
<a 
href="https://nipunbatra.github.io/papers/2022/compass22-samachaar.pdf" 
target="_blank" 
rel="noopener noreferrer">
exploring media bias, paper link
</a>
)
in regards to coverage on air pollution.<br>
We limit our study to only air-pollution and newspapers being our media source, due to easily available transcripts over a large time period. <br>
The study can be extended for other media sources as well, with the only pain-point being availability of transcripts.

* Table of Contents
{:toc}

## Why do we even care ? 
From the paper " False perception could lead to increased exposure to air pollution and increased challenges in implementing mitigation strategies ", as people might underweight this serious issue.

## What do we check for
Also called as research questions
- Does the news media coverage around air pollution exhibit geographical / temporal bias.
- Are pollution sources mentioned in proportion to their contribution to PM levels.
- Does media coverage exhibit deviation from evidence around sources and impact of air pollution 

## Generating the dataset

### Pollution 
Using publicly available data from Central Pollution Control Board (CPCB), which is a govt. organization. Polling rate (granularity) for the data is 15mins, which in simple terms means that the sensors record samples every 15mins. <br>
Some erroneous data (below 0 and above 1000ug/m3) was removed along with rows with missing data.
(
<a 
href="https://rishi-a.github.io/downloads/publications/vartalaap.pdf" 
target="_blank" 
rel="noopener noreferrer">
Citing another paper from IITGn, talking about public perception of air pollution
</a>
)

Bigger cities have more than a single reading station, and to simplify the analysis the authors aggregate (average) across stations in the same city over the day to obtain a single number for a city-day row.

<figure style="text-align: center;">
  <img src="https://hackmd.io/_uploads/rJouO71Vxe.png" 
  alt="Illustration" style="max-width: 100%;" />
  <figcaption><em>With monitoring stations only being deployed recently, most cities (66/88) only have one station, with most being operationalized after 2018</em></figcaption>
</figure>

### Media sources
English dailies : Times of India and The Hindu, given large coverage and easy to access transcripts. 

#### How to know what topics a particular article talks about
Use queries to filter air-pollution related articles, which are handpicked and further snowballed [refer to the end for a short explanation](#Snowball-sampling)

<strong>Sanity checks</strong>
- Filter out queries that have a high proportion of articles not related to air pollution.
- Have two authors annotate 200 articles into related / not-related and the inter-rater agreement (Cohen's kappa) came out to be 94%.
- Amazing how I read about this barely a month ago, while reading the Evaluation of search results from Introduction to Information Retrieval by Manning.

## RQ1 Does media coverage on air pollution exhibit a geographical / temporal bias ?

Based on
<a 
href="https://urbanemissions.info/wp-content/uploads/docs/2019-03-UC-India-APnA20.pdf" 
target="_blank" 
rel="noopener noreferrer">
AirPollutionKnowledgeAssessments we understand :
</a>

- nearly 50% of all reseearch literature is focused on Delhi-NCR.
- district wise air pollution intensity on a choropleth map (albeit this study is a bit older, from 2016)

>
Indo Gangetic Plain region ==>
Covers 11 Cities in Delhi, Punjab, Haryana, UP, West Bengal and Bihar.

With this information (high district-wise annual average pollutant concentration), this research question was further broken down into 
1. Is air pollution a <strong>year-long problem</strong> for cities in the Indo-Gangetic plain ? <br>
How ==> check for how many days, does the PM lvl exceed WHO / Indian standards <br>
out of 11 cities, 9 exceed WHO limits 90% of the year, which highlights how serious of an issue air pollution is throughout the year.

2. Do cities in this region get <strong>media-mentions with regards to their air pollution</strong> ? <br>
How ==> compare article counts by city with days exceeding pollution thresholds. <br>
No, as 9/11 cities experience high Pm2.5 exposure, but Delhi accounts for 36% of all news articles on air pollution. <br>
Even for Delhi, most of these articles are post-2014 due to more monitoring stations + easily accessible data. <br>
Discussions are periodic, focused around the winter months, when the dissipating potential of the environment reduces due to cold winds + geographical conditions.

3. <strong>What cities get high mentions</strong> ? Are these polluted throughout the year ? <br>
How ==>  group rows by 'city' column in the metadata of articles, to get a count of articles for each city. <br>
From the top-10 cities discussed in print-media, all cities (except 3, Delhi, Noida and Gurgaon, which fall under the Indo-Gangetic plain) have less violations / avg PM2.5 lvl as compared to cities in the Indo-Gangetic plain. <br>
News media is more focused on metro cities, although there are many other highly polluted areas in less-urban landscapes.


## RQ2 Does media coverage exhibit deviation from evidence around sources and impact of air pollution ?

### What topics are discussed ? 

Use LDA based topic-modeling [short explanation towards the end](#Topic-Modelling-using-LDA), to get topics from articles.

Remove numbers, mail-id, hyperlinks and stop words from each article. <br>
Removing words with low information content / low differentiating power is a common tactic for NLP / search related tasks. <br>
Authors remove words that occur in more than 80% of all articles and those which occur in less than 15% of all articles for this reason (probably).

We use the LDA results to obtain topics discussed;

>
More the number of topics in an article, less likely it is to belong to one group/topic; consider with logits being spread across multiple classes we are not sure of the right / top class for a search system.

<figure style="text-align: center;">
  <img src="
  https://hackmd.io/_uploads/SJPbySJ4eg.pn
  " alt="Illustration" style="max-width: 100%;" />
  <figcaption><em>Authors only consider articles with at least one topic having probability higher than 0.5 or more. 
Plot topic counts using a 10day rolling window for 2015-2021, which helps us understand patterns in media-reporting</em></figcaption>
</figure>

Using this we can understand the media attention to Air pollution better

### Event specific, periodic
Stubble burning is a periodic practice where farmers burn leftover after harvest season, which only gets media attention in the winter months. <br>

<figure style="text-align: center;">
  <img src="
  https://hackmd.io/_uploads/BJLNlrk4gl.png
  " alt="Illustration" style="max-width: 100%;" />
  <figcaption><em>
High overlap is observed between open fire count (from NASA) and media mentions of 'Stubble burning', but media reporting for this topic is absent for march-april harvest season, which also sees similar levels of open-fires in agriculture dominated regions.
</em></figcaption>
</figure>

### Event specific, episodic
Topics like 'vehicular emissions', 'students', 'Delhi government' get media attention based on some events / actions by the concerned topic / actors.

### Event agnostic, periodic
Topics like 'air quality', 'construction', 'health' are not necessarily tagged to a single event, but most mentions only happen around the winter months.


### Are all pollution sources discussed with regards to their contribution ?
Note: Authors limit this question to only Delhi. <br>
Media coverage is limited to the visible source of pollution. <br>
![image](https://hackmd.io/_uploads/ryLcWrJ4ll.png) <br>
[Suggestions for this have been added in the section below](#Suggested-Future-Work) 

## Ending notes
A very easy to read paper, super smooth flow
Right before this paper, I read about under-representation of country-names in LLM outputs, which was a study from IISc. <br>
Felt amazing to know multiple labs are covering bias (well now that I'm writing this, I don't see much cohesion between these two papers, as here we explore bias in existing data, and the IISc paper explore bias in machine learning outputs, which was caused by bias in the data, wow <br>
Wow, now the two papers seem a bit related)

### Suggested Future Work
1. From the Geographical Erasure paper by KIAC, IISc [notes here](https://hackmd.io/@aryan1113/HkI64dYzlx#Measure-Erasure-the-fun-part), we can calculate an "**erasure**" metric to better quantify under-representation of cities in media. 
Can weigh cities by population and days PM2.5 concentration exceeds India/WHO standards.

2. Instead of article-counts, using % of article mentions would be a better param to showcase difference in pollution-contribution and media-coverage
<figure style="text-align: center;">
  <img src="
  https://hackmd.io/_uploads/BkBvMSkElg.png
  " alt="Illustration" style="max-width: 100%;" />
  <figcaption><em>
section 5.2.2, discussing pollution sources / Figure 9
</em></figcaption>
</figure>



## Refresher Terms

### Snowball sampling 
add mention to types of sampling hackmd note (non-existent)

### Topic Modelling using LDA
Input : <br>
$M$ news articles, each article has $N$ number of words

Output : <br>
$z$ topics, where each topic is a cluster of words. <br>
$\psi$ is the probability distribution of words in a topic. <br>
$\theta$ is the probability distribution of topics per document

#### Concentration params:
$\alpha$ : topic density per document, a lower value is better as we have a clear demarcation between documents. <br>
High alpha might lead to lower-learning ability by the model, as it can assign all possible topics to all documents.

$\beta$ : word density per topic

Topic Coherence measure

#### Interpreting LDA results
Each topic is a circle, compute Shannon divergence between topics and sclae this to 2D using multi-dimension scaling. <br>
Radius of a circle denotes number of articles associated with a single topic
