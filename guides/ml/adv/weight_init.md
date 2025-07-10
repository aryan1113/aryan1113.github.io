---
layout: ml-guides
title: Weight Initialization for Neural Networks
description: special hyperparamter, which sometimes just improves glide to the optima point
usemathjax: true
--- 

# Weight Initialization In Neural Networks

* Table of Contents
{:toc}

> 
Remember, Standard Deviation = $\sqrt{\mathrm{Var}(x)}$ <br>
Variance : $\sigma ^ 2$ <br>
Standard deviation : $\sigma$

## What not to do 
### Zero Initialization
All weights are zero, creates what is known as dead neuron as Input-information to each neuron will be zero, no matter the input $x_i$.

> Input fed to neuron: $x_i \cdot w_i$  
> As $w_i = 0$,  
> $x_i \cdot w_i = 0$

* Due to this,during backpropogation even the gradient $\nabla$ will be zero (see the backpropogation weight updation formula)
* The network fails to learn the intricacies of input data, or in other words, fails to map the relationship between the input and output.

### Symmetrical/Constant Initialization
All weights are assigned the same value, bad idea as all inputs are essentially treated as the same.Although the outputs are not zero for neurons, they dont learn anything. <br>
<a href="https://wandb.ai/sauravmaheshkar/initialization/reports/A-Gentle-Introduction-To-Weight-Initialization-for-Neural-Networks--Vmlldzo2ODExMTg#0️⃣-zero-initialization" 
    target="_blank" 
    rel="noopener noreferrer">
    Read more here </a>

### Random Initialization
Randomly select values from a gaussion curve having mean *$\mu$* and standard deviation *$\sigma$* 

> N ( *$\mu$* ,*$\sigma$* )

Decide what should be the deviation *$\sigma$* in these random-weights. Issue here is if the deviation *$\sigma$* is very low, this becomes close to symmetric initialization on the other hand if we choose a high *$\sigma$*, we tend to move towards the exploding and vanishing gradient problems.

## Solutions 
Maintain some variance in weight allocation

### LeCun Initialization
Weights are distributed in such a way that the variance of weights closely follows the variance in output.

> The output of a neuron with a linear activation function is given by:  
> $$y = w_1 x_1 + w_2 x_2 + \dots + w_n x_n + b$$  
>  
> Taking the variance of both sides:  
> $$\mathrm{Var}(y) = \mathrm{Var}(w_1 x_1 + w_2 x_2 + \dots + w_n x_n + b)$$  
>  
> Since the bias term \( b \) is constant, it does not contribute to the variance and can be ignored.  
>  
> Assuming all weights \( w_i \) and inputs \( x_i \) are independent, the variance becomes:  
> $$\mathrm{Var}(y) = \mathrm{Var}(w_1)\mathrm{Var}(x_1) + \mathrm{Var}(w_2)\mathrm{Var}(x_2) + \dots + \mathrm{Var}(w_n)\mathrm{Var}(x_n)$$


As weights are i.i.d (Independent, Identically Distributed),

$$var(y) = N * var(w) * var(x)$$

Where N is the dimension of input vector. As our goal was to match the variance in input and output,  

$$N*var(w) =1$$ $$var(w) = {1 \over N }$$

LeCun's initialization suggests to randomly allocate weights from a gaussian curve with mean 0 and standard deviation equal to $1 \over {\sqrt{N}}$

### Xavier Glorot Initialization

For efficient performance on backpropogation, the variance should also account for the backword pass through the network. <br>
Weights distribution should be a **Gaussian** with zero mean and variance given by following formula
    
$$var(w) = {2 \over fan_{in}+fan_{out}}$$

where fan~in~ represents the number of inputs coming to the specific layer and fan~out~ represents the no. of outputs moving to the next layer. 

$$ W \sim N(0,{2\over {n_{in} +n_{out}}}) $$

### He Initialization

As ReLU function does is defined as $f(x)=max(0,x)$.
It is not a zero mean function, this makes our initial assumption that the weights distribution have zero mean.<br>
To account for this, we slightly modify the xavier/glorot method.

$$ W\sim N(0,{2\over n_{in}}) $$

## Summary 
1. Zero Initialization doesn't work and neither does Initializing to some constant. This leads to what's called the symmetry problem.
2. Random Initialization can be used to break the symmetry.But, if weights are too small we don't get significant variance in the activations as we go deeper into the network on the other side, If the weights are too large then it leads to saturation. 
3. LeCun Initialization can be used to make sure that the activations have significant variance, but the gradients still suffer
4. Xavier Initialization is used to maintain the same smooth distribution for both the forward pass as well the backpropagation.
5. But, Xavier Initialization fails for ReLU, instead we use He Initialization for ReLU.


