---
last_modified: 2025-07-10 17:55:29 +0000
layout: ml-guides
title: Towards reproducable SOTA Energy Disaggregation, IITGn
description: Smart meters push out consumption data to suppliers in real time, disaggregation helps increase granularity
usemathjax: true
---

# Towards reproducable SOTA Energy Disaggregation

Well unlike other notes that have an appendix at the end, I'll give some context at the very beginning so the entirety of the paper makes sense to you.

* Table of Contents
{:toc}

## Context
Smart meters are basically digital electricity meters that transmit consumption data to energy providers in real time, through IoT or some wired connection, this is to help them plan better and get energy consumption data in near real time. <br>
Disaggregation is used to get an idea of power consumption by a particular device or appliance within a building or metered unit.<br>
<strong>NILM : Non Intrusive Load Monitoring</strong> <br>
Historically, NILM has had fragmented/inconsistent metrics <br>
Newer work mostly compares with benchmark models, which are quite. The authors propose a platform to compare newer-work with SOTA by keeping current StateOfTheArt work as benchmarks.

### DE-aggregation ?
Smart meters would provide us with results from a single connection / meter, which in most cases would be an entire commercial/residential unit. <br>
We aim to measure appliance drawn energy $X_i$ from an aggregate $Y$ which is given by our smart meter in near real time.

### Choice of train test split
- Can train and test on buildings from the same dataset
- Train and test samples are picked from different datasets

>
Geographies and user-behaviour affects power consumption, which makes device/appliance level performance poor if the test set is widely different from the train set <br>
Take care to have buildings from similar geographies, if available use data from different datasets. <br>
The paper discusses a case where a mix of European and American buildings causes poor performance, which has nothing to do with the model.

## What does this paper talk about ?
### New Experiment API
The authors simplify train/eval process. <br>
Aiming to make experimenatation modular, lowering barrier for new researchers and helping simplify benchmarking of results against SOTA. <br>
Earlier users had to iterate over data chunks across diff buildings from diff datasets, combine predictions for each of these blocks and then pass these through an interface to obtain metrics.

### Unified benchmarking
3 baseline + 9 SOTA NILM algorithms <br>
New ideas can be easily evaluated against SOTA architectures (plug and play) for publically available datasets, which is usually aggregated power consumption for buildings. <br>
It was hard to assess generalizability of NILM work as most papers were evaluated on a single dataset, or sometimes even subsets of these datasets, which would make comparing diff works very difficult.

## From the paper
Decouple the data loader and model train function <br>
For algorithms that require pre-processing, a `call_preprocessing` method was added which allows users to store pre-processed data in HDF5 file format. <br>
This way instead of saving data to limited memory, we can simply export this to another file format and use this at train-time.

<strong>Chunk </strong>: Contiguous portion of time series data that fits into memory <br>
<strong>Window </strong>: 
Portion of the chunk on which model trains in 1iteration, the window can be rolling (with or without overlap) as shown in the graphic below.
 
<figure style="text-align: center;">
  <img src="
    https://hackmd.io/_uploads/ryWlMCrNxe.png
  " alt="Illustration" style="max-width: 100%;" />
  <figcaption><em>
    A single window (window == chunk) is used for traditional algorithms like FHMM (Hidden Markov Models), whereas for seq-to-point we would consider multiple overlapping windows.
</em></figcaption>
</figure>

## Algorithms

### mean
predict a constant mean power value for each appliance <br>
Useful as a sanity check for always-on appliances, does not work well for appliances that have on/off patterns

### Edge Detection
find significant changes in power signal to identify appliance switching between on/off states

#### misses out on : 
- cannot handle when multiple devices switch at the same time
- appliances with multiple states, example a fan usually has 5power modes


### Combinatorial Optimization
Assign states to each appliance, so that $\Sigma~states$ is close to aggregate power consumed

### Denoising Autoencoder
Denoise on a per-appliance basis, consider the aggregate power usage as appliance usage + NOISE.

### Recurrent Neural Network
From a sequence of readings of power consumption from our Smart meter, output a single value for power consumption for a particular appliance.

<blockquote>
This is not a many-to-many RNN architecture as the outputs are not linked to each other / fed back into the model / determined by intermediate signals.
</blockquote>
<br>

### Seq-2-seq
predict a window of target appliance power from a window of aggregate power consumed

### Seq-2-point
similar to [Seq-2-seq explained above](#Seq-2-seq) <br>
Instead of predicting an entire window for a particular appliance, we only predict the mean of the window.

### Online GRU
Lightweight RNN for quicker inference, as this method is used to get appliance usage on the fly. <br>
Gated Recurrent Units are basically RNN's but with gates that limit the flow of information to some degree, improving the ability of the model to learn context.

## What new could be done ?

### newer SOTA ?
- Update SOTA models, include a few more seq-to-seq models as the paper is from 2019 (6 yrs old, something new might have popped up)
- Can incorporate a very simple transformer architechture as well.
- Can evaluate these for Temporal CNN's

### More focus on Online Inference 
- Work with streaming data, as most smart meters would be feeding data into our databases on the fly, judging energy consumption by appliance would be useful

### Fix Typo
- Typo at beginning of section 5.4 `In the this experiment` should be `In this experiment`

## Things I didn't understand
(I plan to revisit foundational concepts like Hidden Markov Models and sparse coding)

### Factorial HMM Metrics
I read about Hidden Markov Models in the context of predicting the next speech token as part of an undergrad course on Speech Processing <br>
But honestly it's been a long long time, and I will have to revisit it in entirety to understand the underlying workings. <br>
From a very naive understanding, these methods might work to predict the appliance energy consumption, or the next state (on/off) based on a prior sequence of energy usage. <br>
We have a HMM for all of our appliances, which makes scaling difficult, and to combine predictions we take these models in proportion, hence the term factorial in its name.

### Discriminative Sparse Coding
Haven't heard of this, will need to dive deep <br>
Left this for now, to prioritise on other tasks at hand, apologies.

### What is the artifical aggregate
Do we have the truth values for energy consumed by each appliance and sum it all up for a single commercial/residential unit to get the aggregate ?

<hr>

## Personal notes

Well I kinda was dumbfounded for most of the paper, until it talked about getting the energy consumption for each device/appliance, after which I could piece it all together. <br>
Electricity providers in India are pushing for wider adoption of 'smart meters' and a faster rollout, as this would reduce theft and provide energy-generators with crucial information in door-to-door consumptions metrics in Near-Real-Time, which would help make better predictions for base load demand and the fluctuations we see within a day and help understand seasonality in demand.
