---
layout: guides
title: Vision Transformers
description: Oh wow, everything can be applied in Vision it seems
usemathjax: true
--- 

## What are Vision Transformers ?

Well, transformers are a high-capacity network architecture, having the ability to approximate complex functions. These are general-purpose architectures (like autoencoders) and hence can be used for two of the most popular paradigms/frontiers of Deep Learning: Language Processing and Computer Vision. <br>
Vision Transformers in short are referred to as ViT.

<figure style="text-align: center;">
  <img src="https://miro.medium.com/v2/resize:fit:440/format:webp/1*SoXHGxDPUqFQHFbJKYoVLg.png" alt="Illustration" style="max-width: 100%;" />
  <figcaption>
    <em>
    Simplified ViT
    </em>
  </figcaption>
</figure>

* Table of Contents
{:toc}


## Key Blocks in a Vision Transformer

<ol>
<li>
<strong>Image to patches</strong><br>
Cut the input image into n x n patches, resulting in n² patches and linearly transform them, to create a deeper embedding.
For each patch, we flatten the sub-image (from a matrix to an array) and apply a learned weight vector to obtain a linear projection.
<figure style="text-align: center;">
  <img src="https://miro.medium.com/v2/resize:fit:1100/format:webp/1*dOlsOuqN7KC6xtkyIcpcWA.jpeg" alt="Illustration" style="max-width: 100%;" />
  <figcaption>
    <em>
    Cut the input image into patches, then flatten each patch and obtain embedding, after which a positional matrix is added/concat to the embedding vector. The result is fed to the attention block.
    </em>
  </figcaption>
</figure>
</li> 

<li> 
<strong>Positional embedding</strong><br>
The main reason why transformers grew in popularity was their ability to be parallelized, as computations need not wait for some previous computations or ‘previous cell_state’ as in the case of Traditional RNNs which have a recurrence relation to follow. All input embeddings are fed at once to the architecture thereby losing a sense of position/order of the input tokens. <br>
<br>
<blockquote><strong>Example</strong> :
“Don’t drink and drive, take a cab” VS “Don’t take a cab, drink and drive”
are two sentences with the same words and word_frequencies, only the order of words has changed, which results in a large difference in the semantic meaning.
</blockquote>
<br>
To understand the context of the input better, we must also take into account the position in which the input embeddings are fed into the model. <br>
For language-specific tasks, we make use of <strong>sinusoidal positional encoding</strong> ==> <a href="/guides/ml/adv/pos_encoding/" target="_blank" rel="noopener noreferrer">Blog on positional encoding</a>

<br>
These are added to the embeddings, not concatenated. Hence, the size of embedding is conserved.
</li>
<br>

<li> 
<strong>Attention Block</strong>
<br>
Some parts of the input example have far more information that helps us with the input-output mapping. We use attention blocks to learn which input parts are important.

<figure style="text-align: center;">
  <img src="https://miro.medium.com/v2/resize:fit:1100/format:webp/1*78cjahc8GVUkQijMEFnbSw.jpeg" alt="Illustration" style="max-width: 100%;" />
  <figcaption>
    <em>
    The most eye grabbing details are of the engine/wagon in this image, the human eye ignores other structures on the top right, to save on computation. Attention aims to mimic the same, by only focusing on certain parts of the input image, we can understand context of the entire image.
    </em>
  </figcaption>
</figure>

<blockquote>
Note : Due to compute restrictions / limitations, we do not compute attention between pixels, instead we compute attention between an aggregate of pixels, also known as a block / patch.
</blockquote>
<br>

<h3> How do we compute the Attention values ? </h3>
Easier to understand with the example of a YouTube search. We have many videos in our DB and want to show only relevant ones to the user based on the Query he/she has entered. For each video, some sort of similarity is calculated with the query. <br>
If you remember Linear Algebra from 10th grade, Dot Product returns a scaled degree of similarity between -1 and 1. For two vectors pointing in the exact same direction, the dot product is 1 and for opposite directions, it is -1.

$$
\text{Attention}(Q, K, V) = \text{softmax} \left( \frac{QK^\top}{\sqrt{d_k}} \right) V
$$

$\sqrt{d_k}$ is used to stabilize results, as matrix QKt has variance of d_k. Keeping variance close to 1 is essential to avoid saturating the softmax towards only “some results”. <br>
Q, K and V matrixes have the same shape, Kᵀ is taken to allow multiplication with Q (rule for multiplication in linear algebra). <br>
Q, K and V matrixes can be understood as simple linear transformations of the input provided ( numerical embeddings of some sort ). The weights for the linear transformation are learnt during training.

**Some numbers** : <br>
For an image which is divided into N patches, the number of embeddings we feed to the encoder block is N + 1. After each embedding vector has been obtained, we stack these to form a matrix. <br>
This matrix is multiplied with the key_wt, query_wt and value_wt matrices to obtain the keys, queries and value matrices.
</li>
<br>

<li>
<strong>Layer Norm</strong>
<br>
Normalize all values coming out of a block / layer to prevent cascading effects of exploding gradients. <br>
Very similar to batch_norm, we make the layer output follow a gaussian with zero mean and variance 1, by subtracting the layer_mean from each output, and divide by the variance. <br>
For numerical stability, a small constant epsilion ε is added to the denominator, for cases when the variance is close to 0.
</li>
<br>

<li>
<strong>Skip Connections</strong>
<br>
Provide an additional path for gradients to flow during backdrop, originally developed for CNN based architectures, that primarily suffered from loss of gradients as we moved deeper into networks, the later layers tend to forget what the input distribution was like and could only remember some of its receptive field. <br>
Providing a scaled version of the input deeper down the network brings faster convergence and stabilizes training as updates are less stochastic (random). <br>
There is no absolute proof for why this works, these are just intuitions.
</li>
<br>

<li>
<strong>Multi Head Attention Blocks (optional)</strong>
<br>
Learn multiple Q, K, V matrices as having different representations of the same input improves generalization. <br>
Each attention head learns its Q, K and V matrices separately. Concat output matrices from all heads to a single matrix. <br>
To keep computational cost similar to that of single head attention, we limit the dimensions to D_single / H, where H is the number of heads.
</li>
<br>

<li>
<strong>Output Block</strong>
<br>
Feed everything to a simple Neural Network, have sigmoid as the activation in the last layer to obtain probabilistic values for each output class (in the case of classification, which is the case for majority of image based applications).
</li>
</ol>

## But Why Does the Transformer work ?

Deep Learning in general works better than traditional Machine Learning algorithms as we do not make use of handcrafted features, which make heavy use of human knowledge and are similar to hard-code inductive biases. <br>
Transformers are a very general architecture, with an encoding block and an (optional) decoding block, which makes them data-driven and have less dependence on human-specified biases. <br>
**Small trick** : Along with the input-embeddings, a zeroᵗʰ token is also fed into the transformer block along with the class. For supervised learning applications, we have the true label with us for every sample from the training set. Feeding this along with the input patches gives a sense of belongingness of the class with features from the patches.

## Summary
Very similar to the original transformer architecture, just break image into chunks, flatten these chunks and treat these like any other sequence that you would feed into a black box, similar to what we do for Language tasks. <br>
**Time Complexity** of computing Attention is O(N² * D),

### References
1. <a href="https://jalammar.github.io/illustrated-transformer/" 
  target="_blank" 
  rel="noopener noreferrer">
  Simple and concise explanation by Jay 
  </a>
2. <a href="../pos_encoding" 
  target="_blank" 
  rel="noopener noreferrer">
  Positional Encoding explained 
  </a> 