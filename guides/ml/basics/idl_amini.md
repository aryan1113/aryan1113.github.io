---
last_modified: 2025-08-17 09:51:11 +0000
layout: ml-guides
title: Deep Learning by Alexander Amini
description: Well course notes from youtube lectures, might be useful
usemathjax: true
--- 

# Introduction to Deep Learning Alexander Amini

* Table of Contents
{:toc}

## L1 Introductory Lecture 
### How to set learning rates
1. Try out exhaustive set, which works fine
2. Design an adaptive learning rate that adapts to landscape

### Adaptive Learning Rates Factors
1. How large the gradient is
2. How fast the learning is happening
3. Size of particular weights

### Commonly used Optimizers
- SGD
- Adam
- Adadelta
- Adagrad
- RMSProp

### Gradient Descent 

|Stochastic Gradient Descent | Mini Batch Gradient Descent|
| -------- | -------- |
|Takes a point at random (stochastically) and computes gradient for that particular point|GD in batches of hundreds     |
| Much faster to compute  |Although not as fast as SGD, it is also blazingly fast as we can parallelize computation |
|Computationlly very cheap|Much better estimate of True Gradient|
|Extremely stochastic (bad thing)|Smoother convergence, as stochasticity is significantly reduced|
|Larger learning rates can result in huge unwanted learning rates|Allows for larger learning rate without the oscillations|

### Regularization 
Contrains optimization to discourage complex models, **To avoid overfitting** <br>
Improves eneralization of data on unseen data. <br>
Dropout and Early Stopping are two ways we can implement regularization.
___

## L2 Deep Sequence Models

### Sequence Modelling Applications
1. One to one -> Binary Classification
2. One to many -> Image Captioning; where we have one image as the input and output are words with some sequence.
4. Many to One -> Sentiment Classification
5. Many to Many -> Machine Translation

### Feedforward Neural Network
(broken image link)

### Handling Individual Time Steps
No relation between time steps t₀,t₁,t₂,...tₙ

$\hat{y} = f(x_t)$

Does not take sequences into account

### Neurons with recurrance
Relates networks computation at a particular time instant with its prior history, via a recurrance relation. <br>
We do this via internal memory, that is state history.

$ \hat{y} = f(x_t,t_{t-1}) $

RNN's have a state $h_t$ that is updated at each time step as a sequence is processed.
Apply recurrance relation at every time step to process a sequence.

$ h_t~=f_w~(w_t~,h_{t-1}) $

Where, <br>
h_t represents cell state <br>
f_w is a function with weights w <br>
x_t is the input <br>
h_{t-1} is the previous state <br>

(Add image showing all weights)

$W_{hy}$ Accepts input from hidden layer and passes to the output layer <br>
$W_{hh}$ Accepts input from hidden layers and passes to hidden layer <br>
$W_{xh}$ Accepts input layer and passes to the hidden layer <br>

### Forward pass through networks
For all instances $t_0, t_1, t_2.....t_n~$, we compute individual losses and from these we get the combined loss (total loss)

As neural networks cannot understand words directly, we need to encode/represent words as numbers, for this we use **embeddings**, which transform indexes into vectors.

1. Vocabulary (bag/corpus of words)
2. Indexing of words
3. Embedding, which can be one hot encoding (bad idea, not scalable) or Learned embedding.

(image showing similar words are given similar vectors)

Words that are semantically similar in menaing will have similar encoding. <br>
We also need to handle **variable sequence lengths** in Language processing, a simple feedforward neural net will fail the job, as we predefine the number of input neurons.

<strong>Corner case showing the need to capture differences in sentence order : </strong> <br>
The food was good, not bad at all. <br>
The food was bad, not good at all.

### Backpropogation Through Time (BPTT)
(image showing rnn gradient flow)

### Exploding and Vanishing Gradients

>
Exploding
Overflow => multiply series of big numbers, result goes to infinity.
Vanishing 
Underflow => multiply series of small numbers (less than 1) and result tends to zero.

Vanishing gradients make it difficult to know which direction the parameters should move to improve the cost function while exploding gradients make learning unstable.

<strong>How to deal with these ?</strong><br>
For exploding gradients, make use of Gradient Clipping, which is defining a threshold and ignoring values that are bigger than the threshold. <br>
For vanishing gradients, we have three solutions
1. Make use of RELu activation function 
(show image of differentiation of sigmoid, relu and tanh functions)
2. Initialize weights as an identity matrix
3. Improvise the network architecture. For LSTM's use gated cells.

<strong>Gates</strong> : To selectively add or remove information within each reccurrant unit. In other words, to optionally let information through the cell. <br>
Gates interact with each other to control information flow.

For LSTM's we have 4 gates : 
1. Forget :  To eliminate irrelevant information from the past.
2. Store : Keeping important information from the current input.
3. Update : Update cell state.
4. Output : Return filtered version of cell state.

<strong>Backpropogation Through Time</strong> (BPTT) becomes much more stable by using gates, by having fewer repeated matrix multiplications. This mitigates the issue of vanishing gradients in LSTM's.

### Limitations of RNN's
1. Encoding bottleneck
2. Slow, no parallelization
3. Not very long memory, cannot process above 1000 words.

To improve upon these shortcomings, we need to eliminate recurrance. <br>
(image showing input stream, feature vector and output stream; that is without any notion of sequence)<br>
One way to do this, is by feeding everything into a dense network (spoiler alert, this is a bad idea)

1. No recurrance
2. No order between inputs, no notion of sequence
3. Not scalable due to large number of connections (weights and biases in the fully connected neural net)
4. As no sense of sequence, we loose capability of any memory.

<h3> <a 
href="https://arxiv.org/abs/1706.03762" 
target="_blank" 
rel="noopener noreferrer">
All you need is attention paper </a> </h3>

The idea of attention was first introduced back in 2017 (6yrs ago). <br>
Main motive is to identify and attend to parts of input that are important; for which we make use of postional aware embedding.
1. Encode position information
2. Extract query, key and value for each search.
3. Compute attention weighting for all parts.
4. Extract features with high attention.

___

## L3 Convulational Neural Networks

### When to use CNN's (IAA)
1. Large dataset to extract features from.
2. Difficult to define features.
3. Image/Video data is more suitable for CNN's as they have large number of pixels which are nothing but a matrix of numerical values.
 
### Why not manual features extraction ?
1. Hard to define all features
2. Difficult to make robust model (scale, exposure, angle)
3. Too many parameters, leads to overfitting.
4. As we put pixel values directly into the network, spatial information is lost, as pixels that were earlier close to nearby (imagine a grid) may now be very far apart (in a 1-D array).

### Convulational Neural Networks

1. Apply filters to produce feature maps
2. Introduce non-linearity to model using ReLU
3. Pooling to downsample the feature matrix.

<figure style="text-align: center;">
  <img src="/assets/cnn_diagram.png" 
  alt="CNN Diagram" style="max-width: 100%;" />
  <figcaption><em>
Everytime we downscale our features, the filters get to attend to a larger region of input space. As we progressively go deeper and deeper into the network, downscaling at each step, the features are now capable of attending to the original image which was orignally very large (10⁶ pixels)
</em></figcaption>
</figure>

<strong>Non linearity</strong><br>
Our images and real data is highly non-linear and to capture the variance in data accurately, we make use of ReLU, which replaces all negative values by zero. <br>
Applied after each convolution operation.

<strong>Pooling</strong><br>
To reduce dimensionality, makes further computation easier. <br>
To make model scalable and still preserving the spatial variance and spatial structure.

<strong>Zero padding</strong><br>
As the Filters go through the image, the central pixels get scanned more times as compared to the pixels on the sides, this might cause the image-intricaties on the centre to get more weightage as compared to the ones on the sides. <br>
To avoid this we pad our image matrix with as the name suggests, zeroes.

### Region proposals 
Advanced things to do with cnn include, detecting objects and even getting the bounding box for each object.<br>
Region proposals are used to localize objects within an image.<br>
Using fast R-CNN (Regions with Convulation Neural Network)

### Semantic Segmentation
Predict class for every single pixel, results in super-high output space. (one classification for each pixel)<br>
First part of model is feature extraction, second part is the upscaling operation which is similar to the inverse of encoding of the images.

Downsampling and upsampling operations. <br>
Upsampling is done with the help of transpose convulations, which are able to increase dimensions.
