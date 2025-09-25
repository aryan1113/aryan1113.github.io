---
last_modified: 2025-08-20 11:38:14 +0000
layout: ml-guides
title: Summarizing Netflix ML Blogs
description: Had quite some free time, went through 3 netflix blogs over the weekend
usemathjax: true
--- 

# Netflix Tech blogs

* Table of Contents
{:toc}

---

## Implementing Match Cutting

<a 
href="https://netflixtechblog.com/match-cutting-at-netflix-finding-cuts-with-smooth-visual-transitions-31c3fc14ae59" 
target="_blank" 
rel="noopener noreferrer">
Link to article
</a>

The term [match cutting](#match-cutting) is explained in the terminology section.

Items in the Netflix catalogue (series/movies/shows) have millions of frames and to create manually match cuts, one has to label cuts and match them based on memory. This method misses out on lot of possible combinations and is very time consuming.<br>
To automate selecting similar shots for transitions, we make use of neural networks.<br>
A **frame** can be understood as a snapshot and a **shot** is a collection of frames.

### Approaches

<strong>Simplying the problem</strong>

1. Frames are visually similar within a single shot so only middle frame of each shot was considered.
2. Similar frames can be captured in different shots, so to remove redundancies, image deduplication was performed.
3. Keep frames having humans, discard others to keep things simple

<strong>redundancies</strong>

1. **Shot de duplication** <br>
Early attempts surfaced many near-duplicate shots. <br>
Imagine two people having a conversation in a scene. It’s common to cut back and forth as each character delivers a line.<br>
Near-duplicate shots are not very interesting for match cutting. Given a sequence of shots, we identified groups of near-duplicate shots and only retained the earliest shot from each group.

2. **Identifying near-duplicate shots**<br>
Shots are put into an encoder model, which computes a vector representation of each shot and similarity is calculated using [cosine similarity](#cosine-similarity).<br>
Shots with very similar vector representations are removed.<br>

3. **Avoiding very small** shots<br>
These can arise from a clip of the cast having a conversation, the camera shifts very frequently and can falsely create many such redundant clips.

<figure style="text-align: center;">
  <img src="https://hackmd.io/_uploads/HJceJRV4h.png" 
    alt="Illustration" style="max-width: 50%;" />
  <figcaption><em>
Rough Implementation
</em></figcaption>
</figure>


### Match silhouettes of people using Instance segmentation

<figure style="text-align: center;">
  <img src="https://hackmd.io/_uploads/BJxgbRVV2.png" 
  alt="Illustration" style="max-width: 75%;" />
  <figcaption><em>
how exactly
</em></figcaption>
</figure>

Output of segmentation models is a pixel mask telling which pixels belong to which object.
Basically the similarity between character-outlines is calculated.<br>
Compute [IoU](#iou) for two different frames, pairs with high IoU are selected as candidates.

### Action Matching using Optical Flow
Match cut involving continuation of motion of an object or person.<br>
Intensity of the color represents the magnitude of the motion. Cosine similarity is once again used here.<br>
Brought out scenes with similar camera movement.

---

> <strong>End of Article 1</strong>

---

## Improving Video Quality with Neural Nets

<a 
href="https://netflixtechblog.com/for-your-eyes-only-improving-netflix-video-quality-with-neural-networks-5b8d032da09c" 
target="_blank" 
rel="noopener noreferrer">
Link to article
</a>

### Not much is given in article, so skip this if you want

### Why ?
As netflix is accessed by devices with different screen resolutions which work on different network qualities, video downscaling is deemed necessary. <br>
A 4K source video will be downscaled to 1080p, 720p, 540p and so on, for different users.

### Approach

1. <strong>Preprocessing block</strong><br>
Prefilter the video signal prior to the subsequent resizing operation.
2. <strong>Resizing block</strong><br>
Yields lower-resolution video signal that serves as input to an encoder.

<figure style="text-align: center;">
  <img src="https://hackmd.io/_uploads/B10v04S4h.png" 
    alt="Illustration" style="max-width: 75%;" />
  <figcaption><em>
Architechture of model
</em></figcaption>
</figure>

---

><strong>End of Article 2</strong>

---

## Scaling Machine Learning

<a 
href="https://netflixtechblog.com/scaling-media-machine-learning-at-netflix-f19b400243" 
target="_blank" 
rel="noopener noreferrer">
Link to article, scaling machine learning
</a>

1. Challenges of applying machine learning to media assets
2. Infrastructure components built to address them
2. Case study : To optimize, scale, and solidify an existing pipeline

### Infrastructure Components

<figure style="text-align: center;">
  <img src="https://miro.medium.com/v2/resize:fit:1100/0*4Xx8ArF2deYn4Vf5" 
  alt="Illustration" style="max-width: 75%;" />
  <figcaption><em>
someday I'll understand this better, probably requires me to understand System Design first
</em></figcaption>
</figure>

<strong>Jasper for Media Access</strong><br>
To streamline and standardize media assets

<strong>Amber Feature Store for Media Storage</strong><br>
[Memoizes](#memoization) features/embeddings tied to media entities. <br>
Prevents computation of identical features for same asset, enables different pipelines have access to these features.

<strong>Amber Compute for handling data streams</strong><br>

* Models run over newly arriving assets, and to handle the new incoming data, various trigger-mechanisms and [Orchestration](https://hackmd.io/hC53pheET-aNdXejN_2J4Q?both#Orchestration) components were developed for each pipeline.
* Over time this became difficult to manage, so to handle this Amber Compute was developed. 
* It is a suite of multiple infrastructure components that offers triggering capabilities to initiate the computation of algorithms with recursive dependency resolution.

<strong>To lower computational Load</strong>

1. Multi GPU/ multi node, [distributed training (bottom of this doc)](#multi-node-distributed-training) ===> 
<a 
href="https://drive.google.com/drive/folders/1qyXrT6OKb6hDFaUwOmt0My-ofO-MmLwA?usp=share_link" 
target="_blank" 
rel="noopener noreferrer">
notes for the same, in drive
</a>
2. Pre-compute the dataset
3. Offload pre-processing to CPU instances
4. Optimize model operators within the framework
5. Use file system to resolve data-loading bottleneck

### Scaling match cutting 
Need to control amount of resources used per step

### Initial Approach
<strong>Step 1 : Define shots</strong>

* Download a video file, and produce boundary shot metadata.
 That is, divide a video into various shots.
* Materialize each shot into an individual file clip.

<strong>Step 2 : Deduplication</strong>

1. Extract representation/embedding of each file using an encoder model.
Use the encoder values (vector) to identify and remove duplicate shots (performing [de-duplication](#removing-redundancies))
2. Surviving files are passed on to step 3.

<strong>Step 3 (vaguely mentioned in article)</strong><br>
Compute another representation per shot, depending on the flavor of match cutting

<strong>Step 4 : Score the pairs</strong><br>
Enumerate all pairs and compute a score for each pair of representations. Scores are stored along with the shot metadata

<strong>Step 5 : Sort the pairs</strong><br>
Sort the pairs based on similarity score, and use only the top k-pairs, **k** being the number of match-cuts required by design team.


### Problems with the initial approach

1. Lack of **Standardization** <br>
    The representations we extract in Steps 2 and Step 3 are sensitive to the characteristics of the input video files. <br>
    In some cases such as instance segmentation, the output representation in Step 3 is a function of the dimensions of the input file.<br>
    Not having a standardized input file format creat quality-matching issues when representations across titles with different input files needed to be processed together (e.g. multi-title match cutting).

2. Wasteful **repeated computations**<br>
    Segmentation at the shot level is a common task used across many media ML pipelines. 
    Also, deduplicating similar shots is a common step that a subset of those pipelines share.<br>
    [Memoizing](#memoization) these computations not only reduces waste but also allows for [congruence](#congruency) between pipelines that share the same preprocessing step.

3. **Pipeline triggering**<br>
Triggering logic : whenever new files land, trigger computation<br>
* Lack of standardization meant that the computation was sometimes re-triggered for the same video file due to changes in metadata, without any content change.
* Many pipelines independently developed similar bespoke components for triggering computation, which created inconsistencies.
    
---

### Final solution for scaling match cutting

<strong>Standardized</strong> video encoder<br>
Entire Netflix catalog is pre-processed and stored for reuse. <br>
Match Cutting benefits from this standardization as it relies on homogeneity across videos for proper matching.

<strong>Shot segmentation and deduplication reuse</strong><br>
Videos are matched at the shot level. <br>
Breaking videos into shots is a common task, the infrastructure team provides this canonical feature that can be used as a dependency for other algorithms.<br>
Using this feature values were [memoized](#memoization), saving on compute costs and guaranteeing [coherence](#coherence) of shot segments across algos.

<figure style="text-align: center;">
  <img src="https://miro.medium.com/v2/resize:fit:1100/0*kjmUeSLly_dGqDmO" 
  alt="Match cutting pipeline" style="max-width: 75%;" />
  <figcaption><em>
    Match cutting pipeline. Interactions are expressed as a feature mesh
</em></figcaption>
</figure>

---

> <strong>End of article 3</strong>

---

## Terminologies

### Match Cutting
Video editing technique, that acts as a transition between two shots using similar visual frames, composition, action etc. <br>
In film-making, a match cut is a transition between two shots that uses similar visual framing, composition, or action to fluidly bring the viewer from one scene to the next.

### IoU
Also reffered to as the Jaccard Index.<br>
Intersection over Union, has theoretical maximum value of 1, when both sets are equal.

$$J(A,B)= {A∪B \over A∩B}$$

### Cosine Similarity 

To visualize, consider two vectors in 2D space. Thier cosine similarity is simply given by computing the cosine of two vectors. 

<figure style="text-align: center;">
  <img src="https://hackmd.io/_uploads/Bk4Lh4BNn.png" 
    alt="Cosine similarity" style="max-width: 50%;" />
  <figcaption><em>
Cosine similarity for vectors
</em></figcaption>
</figure>

<a 
href="https://www.youtube.com/watch?v=e9U0QAFbfLI" 
target="_blank" 
rel="noopener noreferrer">
Video explaining cosine similarity
</a>

### Multi node distributed training
<a 
href="https://drive.google.com/drive/folders/1qyXrT6OKb6hDFaUwOmt0My-ofO-MmLwA?usp=sharing" 
target="_blank" 
rel="noopener noreferrer">
Notes on distributed learning for Machine Learning
</a>
Using many worker nodes to make use of parallelization to help speed up computation.

### Memoization
I should add this here

### Orchestration
Orchestration coordinates multiple microservices to achieve a common goal using a central platform like Kubernetes

### Congruency
congruency can be considered as a factor that influences the convergence of optimization methods

### Coherence
cohesion refers to the degree to which the elements inside a module belong together. In one sense, it is a measure of the strength of relationship between the methods and data of a class and some unifying purpose or concept served by that class.
