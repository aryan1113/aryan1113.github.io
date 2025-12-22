---
last_modified: 2025-09-25 17:55:29 +0000
layout: guides
title: Game Theory for DS
description: Missed out on a lot in uni, here's my attempt to read it all
usemathjax: true
--- 


# Game Theory in the context of DS

Well as mentioned in <a href="./adv_cs_explore" target="_blank"> Adv CS topics to explore </a>, I could not read about Game Theory back in university, due to well, academic restrictions (that somehow only allowed me to pick a single elective for each semester in my sophomore year). This led me to choose between Probability and Random Processes by Prof. SNS and Game Theory by Prof. Ojha. I ended up picking PRP as I wanted to end up somewhere in DS.

Okay, now that we’re done with a bit of backstory, let’s jump into the juicy part.


* Table of Contents
{:toc}

<hr style="border:1px solid gray">
𓆝 𓆟 𓆞 𓆝 𓆟<br>
fish as a line break

## Basics of Game Theory

Before jumping into continuous games and equations, it helps to ground ourselves in a very small example. 

Consider a simple two-player game:

- Players: **Sam** and **Tom**
- Actions available to both players: **A** or **B**
- Each outcome gives a *utility* (payoff) to both players

The payoff matrix below lists utilities in the form **(Sam, Tom)**.

| Sam \ Tom | A       | B        |
|-----------|---------|----------|
| **A**     | (1, 1)  | (1, -1)  |
| **B**     | (-1, 1) | (0, 0)   |


### Utility

Utility is simply a numerical representation of how good an outcome is for a player. Higher utility means the player prefers that outcome more.

In this table:
- If both play **A**, both get utility `1`
- If Sam plays **A** and Tom plays **B**, Sam still gets `1`, but Tom gets `-1`
- Utilities need not be symmetric, positive, or “fair”; they only encode preferences

You can think of utility as:
- Profit
- Click-through rate
- Engagement
- Any scalar objective a player is trying to maximize


### Dominant Strategy

A **dominant strategy** is an action that gives a player higher utility *regardless of what the other player does*.

Let’s look at Sam’s choices:
- If Tom plays **A**:
  - Sam gets `1` from **A**, `-1` from **B** → prefers **A**
- If Tom plays **B**:
  - Sam gets `1` from **A**, `0` from **B** → prefers **A**

So **A is a dominant strategy for Sam**.

Now look at Tom:
- If Sam plays **A**:
  - Tom gets `1` from **A**, `-1` from **B** → prefers **A**
- If Sam plays **B**:
  - Tom gets `1` from **A**, `0` from **B** → prefers **A**

**A is also a dominant strategy for Tom**.

When a dominant strategy exists, players do not need to reason deeply about the other player — they simply play it.


### Nash Equilibrium

A **Nash equilibrium** is an action profile where *no player can improve their utility by unilaterally deviating*.

In this game:
- Both players playing **A** is a Nash equilibrium
- Given Tom plays **A**, Sam has no incentive to switch
- Given Sam plays **A**, Tom has no incentive to switch

Formally, each player is playing a **best response** to the other’s action.

Not all games have dominant strategies, but many still have Nash equilibria. Nash equilibrium is therefore the more general and important concept.


## Cournot Duopoly

The Cournot duopoly is a textbook example of a **simultaneous-move game** where:

- Two firms produce the *same* good
- Products are perfectly substitutable
- There is no customer loyalty
- Firms choose quantities **simultaneously**
- **Strategies** are quantities produced
- **Utilities** are profits

This is especially relevant for commodities and FMCG goods.

In a marketplace context, this can be loosely mapped to two sellers on a niche SERP, listing highly similar products with little to no differentiation. Excess supply reduces prices, compresses margins, and can reduce result diversity — eventually leading to user fatigue.


### Model setup

Let:
- `s1` = quantity produced by firm 1
- `s2` = quantity produced by firm 2

The inverse demand function is:

$$
P(s_1, s_2) = A - B(s_1 + s_2)
$$

Price decreases as total quantity in the market increases.

Let:
- Cost per unit = `C`

Total costs:
- Firm 1: `Cs1`
- Firm 2: `Cs2`


### Utilities (profits)

Utility is profit = revenue − cost.

For firm 1:

$$
u_1(s_1, s_2) = [A - B(s_1 + s_2)] s_1 - Cs_1
$$

Rewriting:

$$
u_1(s_1, s_2) = [A - C - B(s_1 + s_2)] s_1
$$

Firm 1 chooses `s1` assuming `s2` is fixed. The question becomes:  
**what quantity maximizes firm 1’s profit?**


### Best response

To find the best response of firm 1, we maximize `u1` with respect to `s1`.

$$
\frac{\partial u_1}{\partial s_1}
= A - C - 2Bs_1 - Bs_2
$$

Setting this to zero:

$$
s_1^* = \frac{A - C - Bs_2}{2B}
$$

This best-response function tells us that firm 1 produces less when it expects firm 2 to produce more.

By symmetry, firm 2’s best response is:

$$
s_2^* = \frac{A - C - Bs_1}{2B}
$$


### Nash equilibrium (competitive equilibrium)

At Nash equilibrium, **both firms play their best responses simultaneously**. Solving the system:

$$
s_1^* = s_2^* = \frac{A - C}{3B}
$$

This point is called the **Cournot–Nash equilibrium** or **competitive equilibrium**.

Neither firm can increase its profit by changing output alone, given the other firm’s quantity.


### Joint profit maximization vs competition

Now ask a different question:  
*What if both firms coordinated to maximize total profit?*

Total (joint) utility is:

$$
u_T = u_1(s_1, s_2) + u_2(s_2, s_1)
$$

Maximizing `u_T` corresponds to **joint profit maximization**, similar to a cartel or monopoly outcome.

This outcome yields:
- Higher total profit
- Lower total quantity
- Higher prices

However, it is **not stable** without enforcement. Each firm has an incentive to deviate and produce slightly more, which pushes the system back toward the competitive (Nash) equilibrium.

This is a recurring theme in game theory:
> Nash equilibria are stable, but not necessarily Pareto optimal.

## Mixed Strategy Games

So far, we’ve mostly discussed games where players pick a **pure strategy**, i.e. they deterministically choose a single action. Some of these games have dominant strategies, some do not — but even when dominant strategies are absent, a stable outcome *may* still exist in pure strategies.

However, there are games where **no pure-strategy Nash equilibrium exists at all**. In such cases, any deterministic choice can be exploited by the opponent. This is where **mixed strategies** become necessary.

A **mixed strategy** is a probability distribution over a player’s actions. Instead of always choosing a single action, the player randomizes — not out of indecision, but as a deliberate strategy to remain unpredictable.


### Matching Pennies

Matching Pennies is a classic two-player, zero-sum game.

- Both players simultaneously choose **Heads (H)** or **Tails (T)**
- If both coins match, **Player A wins**
- If the coins differ, **Player B wins**

The payoff matrix below lists utilities as **(Player A, Player B)**:

| Player A \ Player B | Heads       | Tails       |
|---------------------|-------------|-------------|
| **Heads**           | (1, -1)     | (-1, 1)     |
| **Tails**           | (-1, 1)     | (1, -1)     |


### Best responses and cycling

Let’s look at best responses:

- If Player B plays **Heads**, Player A’s best response is **Heads**
- If Player B plays **Tails**, Player A’s best response is **Tails**

So:
- `BR_A(H) = H`
- `BR_A(T) = T`

Now for Player B:
- If Player A plays **Heads**, Player B’s best response is **Tails**
- If Player A plays **Tails**, Player B’s best response is **Heads**

So:
- `BR_B(H) = T`
- `BR_B(T) = H`

This creates a **best-response cycle**:

- (H, H) → Player B deviates
- (H, T) → Player A deviates
- (T, T) → Player B deviates
- (T, H) → Player A deviates

No action pair is stable.  
**There is no pure-strategy Nash equilibrium.**


### Why randomization is necessary

Suppose Player A always plays Heads.

Player B immediately exploits this by always playing Tails and winning every round.

The same happens for any deterministic strategy. Predictability is punished.

To avoid being exploited, each player must randomize in a way that makes the opponent **indifferent** between their available actions.

In Matching Pennies, the only such equilibrium is:

- Player A plays Heads with probability `0.5`
- Player B plays Heads with probability `0.5`

This is the **mixed-strategy Nash equilibrium**.

At this point:
- Neither player can improve their expected payoff by changing probabilities
- The game becomes stable *in expectation*, even though outcomes vary each round

This idea — using randomness as a strategic tool — shows up frequently in:
- Security systems
- Ad auctions
- Online experimentation
- Adversarial ML settings


## Bayesian Games

So far, we’ve assumed that all players know everything about the game and each other. In many real systems, this assumption breaks down.

**Bayesian games** model situations of **incomplete information**, where players do not know some aspects of the environment — such as the type, intent, or constraints of other players.


### Core ingredients of a Bayesian game

A Bayesian game introduces three new elements:

1. **Nature**  
   A random process that determines unknown parameters of the game.

2. **Types**  
   Each player may have a private type, known only to themselves.

3. **Beliefs**  
   Other players hold probabilistic beliefs over these types.

The solution concept is a **Bayesian Nash Equilibrium (BNE)**, where:
- Each player’s strategy is optimal given their beliefs
- Beliefs are updated consistently using observed actions


### Marketplace example: signaling and trust

Consider a generic online marketplace with buyers and sellers.

Some sellers are genuine, others are scammers.

#### Nature and types
- Nature chooses seller type:
  - Genuine seller with probability `0.99`
  - Scammer with probability `0.01`
- The seller observes their own type
- The buyer does not

#### Actions
- **Seller actions**:
  - Direct transfer
  - Insured payment (platform holds funds until transaction completion)

- **Buyer actions**:
  - Buy
  - Abort

The payment method acts as a **signal**.


### Payoffs (illustrative)

For a **genuine seller**:
- Direct transfer yields slightly higher utility (lower fees)
- Insured payment has a small cost, but still positive payoff

For a **scammer**:
- Direct transfer yields high utility (can keep money without delivering)
- Insured payment yields zero utility (cannot successfully scam)

The buyer prefers to transact only when the probability of scam is sufficiently low.


### Belief updates

The buyer updates beliefs based on the observed payment method:

- If the seller chooses **insured payment**:
  - Probability seller is genuine becomes very high
- If the seller chooses **direct transfer**:
  - Probability seller is a scammer increases

This belief update is what drives buyer behavior.


### Separating equilibrium

In equilibrium:

- **Genuine sellers** choose insured payment
- **Scammers** choose direct transfer
- Buyers:
  - Buy when they see insured payment
  - Abort when they see direct transfer

This is called a **separating equilibrium**, because:
- Different types choose different actions
- Actions reveal private information
- Beliefs become sharp after observing the signal

The key requirement is that:
- Insured payment is **too costly** for scammers to mimic
- But affordable enough for genuine sellers


### Why this matters

In isolation, a genuine seller might prefer direct transfer due to lower fees.  
But once signaling and belief updates are introduced, insured payment becomes the rational choice.

This explains why:
- Insured payment options exist even if adoption is low
- Platforms subsidize or encourage them
- Trust mechanisms are essential for market existence

Game theory predicts that **without credible signals, trust collapses**, and mutually beneficial trades stop happening — even when most participants are honest.


## Auction Theory

Auctions are one of the most important applications of game theory in real systems — especially in ads, marketplaces, and platform economics.

At a high level, auctions solve a **price discovery** problem.

The seller does not know how much buyers value an item. Each buyer knows their own willingness to pay, but this information is private. An auction is a **mechanism** designed to extract this private information through strategic interaction.


### Values, bids, and utility

Before comparing auction formats, it’s important to separate three concepts:

- **Value (`v_i`)**  
  The true willingness to pay of bidder `i`.

- **Bid (`b_i`)**  
  The number bidder `i` submits to the auction.

- **Utility (`u_i`)**  
  The gain from participating in the auction:

utility = value − price paid (if you win)
= 0 (if you lose)


A well-designed auction aligns incentives so that:
- bids reveal true values
- the item goes to the bidder who values it the most
- prices reflect competition in the market


## First-Price vs Second-Price Auctions

We restrict attention to **single-item sealed-bid auctions**.


### First-Price Sealed-Bid Auction (FPA)

- Each bidder submits one bid
- Highest bidder wins
- Winner pays **their own bid**

#### Strategic behavior: bid shading

If you bid your true value in a first-price auction and win, you pay exactly what the item is worth to you — yielding zero utility.

To achieve positive utility, bidders **shade their bids downward**:

bid = true value − shading


This creates a trade-off:
- Bid too high → risk overpaying
- Bid too low → risk losing

As a result:
- Observed bids are **not equal to true values**
- The seller sees distorted signals
- Price discovery becomes noisy and equilibrium-dependent

This is why first-price auctions require sophisticated bidding strategies and strong assumptions about competitors.


### Second-Price Sealed-Bid Auction (SPA)

Also known as a **Vickrey Auction**.

- Each bidder submits one bid
- Highest bidder wins
- Winner pays the **second-highest bid**

At first glance, this looks odd — why shouldn’t the winner pay what they bid?

The answer lies in incentives.


## Why truthful bidding works in a Second-Price Auction

In a second-price auction, bidding your true value is a **weakly dominant strategy**.

Let:
- `v_i` = true valuation of bidder `i`
- `b_i` = bid of bidder `i`
- `B` = highest bid among all other bidders
- `u_i` = utility of bidder `i`

The utility function is:

$$
u_i(b_i, B) =
\begin{cases}
v_i - B & \text{if } b_i > B \\
0       & \text{if } b_i < B
\end{cases}
$$

We compare truthful bidding (`b_i = v_i`) against deviations.

---

### Case A: Underbidding (`b_i < v_i`)

1. **If `B < b_i`**  
   You win. Utility = `v_i − B`.  
   Same outcome as bidding truthfully.

2. **If `B > v_i`**  
   You lose. Utility = `0`.  
   Same outcome as bidding truthfully.

3. **If `b_i < B < v_i`**  
   - Underbidding → you lose → utility `0`
   - Truthful bidding → you win → utility `v_i − B > 0`

Underbidding can strictly reduce utility.

---

### Case B: Overbidding (`b_i > v_i`)

1. **If `B < v_i`**  
   You win. Utility = `v_i − B`.  
   Same outcome as truthful bidding.

2. **If `B > b_i`**  
   You lose. Utility = `0`.  
   Same outcome as truthful bidding.

3. **If `v_i < B < b_i`**  
   - Truthful bidding → you lose → utility `0`
   - Overbidding → you win and pay `B > v_i`  
     → utility is **negative**

Overbidding risks losses.

---

### Conclusion

In all possible scenarios:
- Bidding `b_i = v_i` gives utility ≥ any other bid
- Sometimes strictly better

Therefore, **truthful bidding is a weakly dominant strategy** in a second-price auction.

Crucially, this strategy:
- Does not depend on beliefs
- Does not depend on the number of bidders
- Does not depend on bid distributions

This robustness is what makes second-price auctions special.


## Second-Price Auctions and Price Discovery

Now we connect incentives to **price discovery**.

In a second-price auction:
- Bids ≈ true valuations
- The highest bidder wins → **allocative efficiency**
- The price paid = second-highest valuation

Conceptually:

True values (sorted): v1 < v2 < v3 < v4
Winner: v4
Price paid: v3


The price reflects:
- Competition from the marginal (next-best) buyer
- Market demand
- Scarcity

Because bids reveal values, the seller learns:
- How much the market values the item
- How competitive demand is

This is why second-price auctions are often described as **truth-revealing mechanisms**.


## Revenue Equivalence Theorem

A common misconception is that second-price auctions generate less revenue because the winner does not pay their own bid.

Under specific assumptions, this is not true.

The **Revenue Equivalence Theorem** states:

> In auctions with risk-neutral bidders, independent private values, and no reserve price, all auction formats that allocate the item to the highest bidder yield the same expected revenue.

Key assumptions:
- Independent private valuations
- Symmetric bidders
- Risk neutrality
- No reserve prices

Interpretation:
- First-price auctions extract revenue via bid shading
- Second-price auctions extract revenue via truthful bids
- Expected seller revenue is the same
- But **information revelation and incentives differ**

This is why mechanism design focuses on incentives, not just revenue.


## Generalized Second Price (GSP) Auctions

Many ad auctions are not single-item auctions but **slot auctions** (e.g. multiple ad positions).

The most common mechanism used in practice is the **Generalized Second Price (GSP) auction**.

Simplified view:
- Ads are ranked by bid (or bid × quality)
- Highest-ranked advertiser gets the top slot
- Each advertiser pays the bid of the advertiser ranked just below them

GSP is inspired by second-price auctions, but:
- Truthful bidding is **not** a dominant strategy
- Theoretical guarantees are weaker
- Equilibria still resemble efficient outcomes under assumptions

Despite this, GSP remains popular because:
- It is simple
- It performs well empirically
- It approximates the incentives of second-price auctions

Modern ad systems often combine:
- GSP-style pricing
- Reserve prices
- Quality adjustments
to balance revenue, relevance, and fairness.

---