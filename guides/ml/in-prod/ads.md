---
last_modified: 2025-10-04T19:40:10+05:30
layout: ml-guides
title: Ads ranking
description: August was a pretty wild month, first time I heard about Demand Side Platforms / Ad-Exchanges and boom, ranking for ads as well. Sidenote, I also went to an inmobi conference in August.
usemathjax: true
--- 

# Ads Ranking

Saw a [video](#acknowledgements) on YouTube talking about Ads Ranking at Meta, closely after I read about Ads in general, and what goes behind serving ads in August. So here's a deep dive.

Meta makes a lot of revenue from running ads, and it's a pretty large problem to solve, as both sides—users and advertisers—are very diverse on the platform. This requires a lot of feature engineering and selection to really grasp what to model.

* Table of Contents
{:toc}

<hr style="border:1px solid gray">
ᓚᘏᗢ ᓚᘏᗢ ᓚᘏᗢ ᓚᘏᗢ ᓚᘏᗢ <br>
cats as a line break

# The Ranking Formula: Predicting Total Value

At its core, ad ranking is about predicting the total value an ad impression will generate. This isn't just about whether a user will click, but also what happens after the click. This is often summarized by a metric called **eCPM (effective Cost Per Mille)**. To calculate this, we first need to predict user actions.

## pCTR: Will the user click?
**pCTR (predicted Click-Through Rate)** is the probability that a user will click on an ad after seeing it. A high pCTR means the ad is likely relevant and engaging to a specific user at that moment.

$$ \text{pCTR} = P( ~ \text{Click} ~ | ~ \text{User, Ad, Context} ~ ) $$

## pCVR: Will the user convert?
**pCVR (predicted Conversion Rate)** is the probability that a user will take a desired action *after* clicking the ad. A "conversion" could be anything the advertiser values: add to wishlist/cart, making a purchase, or installing an app (which TataClip / TataNeu loves).

$$ \text{pCVR} = P(~ \text{Conversion} ~ | ~ \text{Click, User, Ad, Context} ~ ) $$

This is crucial because a click alone doesn't mean the advertiser achieved their goal. High pCVR suggests the ad's landing page and offering are compelling to the user.

## Calculating the Ad Score (eCPM)
The final ranking score combines these predictions with the advertiser's bid. It balances the platforms goal of showing relevant content to users with the advertiser's goal (click purchase whatever).

A simplified version of the formula looks like this:

$$ \text{Ad Score} = ( ~ \text{pCTR} \times \text{Bid}_{\text{click}} ~ ) + ( ~ \text{pCVR} \times \text{Bid}_{\text{conversion}} ~ ) $$

This score represents the total expected value. An ad with a slightly lower pCTR but a much higher pCVR and conversion bid might outrank an ad that's just simple clickbait.

<hr style="border:1px solid gray">

# Modeling: The Two-Tower Architecture

To predict pCTR and pCVR, you need a model that understands the relationship between users and ads. Given the massive scale at Meta, a **Two-Tower Model** is a very effective architecture for this.

<figure style="text-align: center;">
  <img src="https://miro.medium.com/v2/resize:fit:600/1*cCEjrsThpCooecYt4A59AQ.jpeg" 
  alt="Two Tower architectrue" style="max-width: 100%;" />
  <figcaption><em>A Two Tower model</em></figcaption>
</figure>

It works by separating the user and the ad into two different "towers" (which are essentially two separate deep neural networks).

1.  **User Tower**: This network takes all the user-side features (age, interests, past interactions) and computes a fixed-size numerical vector, or **embedding**, that represents the user.
2.  **Ad Tower**: This network does the same for ad-side features (category, brand, text, image) and also computes an embedding of the same size.

During serving, the model calculates the embeddings for the user and for many candidate ads. The final score (a proxy for pCTR/pCVR) is just the **dot product** of the user embedding and the ad embedding. A higher dot product implies a better match.

This is an efficient design as the ad embeddings can be pre-computed and stored. When a user logs in, you only need to compute their user embedding once and then use approximate nearest neighbor search to find the best-matching ad embeddings from millions of candidates.

<hr style="border:1px solid gray">
~.~"~._.~"~._.~"~._.~"~._.~"~._.~"~. <br>
a wave as a line break

# The Features

The models rely on a rich set of features from both the user and the ad.

## User-side Features
These are split into two main types:

**User Description Features**
- Demographics: `age`, `country`, `gender`
- Inferred Interests: Topics they've engaged with

**User Interaction Features**
- Past ad engagement: `clicks`, `add-to-carts`, `purchases` over a 90-day period (not fixed)
- Ad IDs that led to a conversion.

**Context Features**
- Ongoing or expected events: `user local time`, `user city` (for hyperlocal demand), `season`, `is_holiday`.

## Ad-side Features

**Content Features**
- `Title`, `description`
- `ad_category`
- `brand_name`
- `avg_price` of the product(s)
- `image`

**Historical Performance Features**
- `avg_CTR` across different user segments
- number of `likes`, `comments`, and `shares` the ad already has.

<hr style="border:1px solid gray">

> End of video contents, below is generic information about ad systems focusing on auctioning of slots. 

<hr style="border:1px solid gray">

# Auctions and Bidding

When a user opens their feed there isn't just one ad waiting; there's an auction in a **ad exchange**.

Here's how it generally works:

1.  **Ad Request**: User opens app => sends an ad request to ad exchange
2.  **Candidate Selection**: The system retrieves a few hundred relevant ads from a pool of millions
3.  **Bidding & Ranking**: For each of these ads, advertisers have already placed bids. The ranking model calculates the **Ad Score** (eCPM) for each one.
4.  **The Auction**: The ads are ranked by their Ad Score, one with the highest score wins the auction.

## Generalized Second-Price Auction
The winner does not pay what they bid, instead a **Generalized Second-Price (GSP)** auction is used.

- The ad with the highest **Ad Score** wins the top slot.
- The winner pays the bid of the advertiser ranked just below them, plus a small increment (e.g. \$0.01).

### Price Calculatation in a GSP Auction

In GSP auctions, **ads are ranked** by **Ad Rank**, computed as:

$$ \text{Ad Rank}_i = \text{Bid}_i \times \text{Quality}_i $$

Then, ads are ordered by Ad Rank (highest first).  
The **winner doesn’t pay their own bid** , instead they pay *just enough* to maintain their position above the next advertiser.  
So the **price per click (PPC)** for advertiser *I* is:

$$
\text{Price}_i = \frac{\text{Ad Rank}_{i+1}}{\text{Quality}_i} + \epsilon
$$

where  
- $\text{Ad Rank}_{i+1}$ is the next advertiser’s score,  
- $\text{Quality}_i$ is the ads own quality score,  
- and $\epsilon$ is a tiny increment (like \$0.01).  

---

#### Example Table

| **Advertiser** | **Bid ($)** | **Quality (pCTR etc.)** | **Ad Rank (Bid × Quality)** | **Price Paid ($)** |
|:----------------|------------:|------------------------:|-----------------------------:|-------------------:|
| **A** | 2.00 | 0.50 | 1.00 | (0.81 ÷ 0.50) + 0.01 = **1.63** |
| **B** | 1.80 | 0.45 | 0.81 | (0.48 ÷ 0.45) + 0.01 = **1.08** |
| **C** | 1.20 | 0.40 | 0.48 | ~ (lowest rank, pays reserve price) = **0.01** |

#### Step-by-step for Advertiser A:
1. Next advertiser (B) has Ad Rank = 0.81  
2. A’s Quality = 0.50  
3. Apply formula:  
   $$ \text{Price}_A = \frac{0.81}{0.50} + 0.01 = 1.63 $$
So A pays **\$1.63 per click**, not their full \$2.00 bid.

---

- This system rewards **high-quality ads**, because if ad quality is better (higher pCTR, better engagement), advertiser can **pay less for the same position**.  
- It also discourages overbidding as price depends on competitors.  

<hr style="border:1px solid gray">

# Cold Start For Ads ?

How do you rank ads or personalize for users when they are brand new to the platform ?

| | User Cold Start | Ad Cold Start |
| :--- | :--- | :--- |
| **The Problem** | A new user has no interaction history, making it difficult to build a user embedding or personalize their feed. | A new ad has no performance data (no historical CTR or conversion rate), making it hard to predict its eCPM. |
| **Solutions** | - Use sign-up information (age, gender, location) as initial features.<br>- Serve globally or regionally popular/trending ads.<br>- Use a simpler exploration model to quickly learn user preferences. | - Use content-based features from the ad tower (text, image, category, price) to infer its likely performance.<br>- The system can give the ad a temporary "exploration budget" to show it to a small, diverse audience to quickly gather performance data. |

<p style="text-align: center;">Table showing how different cold start for ads is, comparing to users
</p>

<br>
<br>

# Further Reading

## Acknowledgements

[Codementor talk](https://www.codementor.io/events/intro-to-the-major-ad-ranking-algorithms-gcy5jotwyx) by a Ranking Engineer at Meta [YouTube link](https://www.youtube.com/watch?v=KDhqEyIO3LA)

## Multi Objective Loss Function

Ads ranking as a multi objective loss function, as we have to combine engagement and value. Balance between user experience (relevace, fatigue, diversity) and value for advertiser.

$$ L = -(\alpha \cdot \text{Revenue Gain} + \beta \cdot \text{User Engagement}) $$
