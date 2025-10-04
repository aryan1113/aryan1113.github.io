---
last_modified: 2025-07-10 17:55:29 +0000
layout: ml-guides
title: Positional Encoding for Transformers
description: well all transform blogs talk about the same 3 matrices, so here's positional encoding, something which is interesting to discuss as well
usemathjax: true
--- 

# Positional Encoding in Transformers

* Table of Contents
{:toc}


## Why even use Transformers ?
Recurrent Networks are by design able to process sequential data much better than traditional feedforward neural networks. However, due to the recurrence relation, RNNs suffer from Vanishing Gradient when processing longer sequences. <br>
This limits the context size of RNNs.

<figure style="text-align: center;">
  <img src="https://miro.medium.com/v2/resize:fit:1100/format:webp/1*NKhwsOYNUT5xU7Pyf6Znhg.png" alt="Illustration" style="max-width: 100%;" />
  <figcaption><em>An RNN unit, having a recurrance relation</em></figcaption>
</figure>

Dealing with one token/word at a time step also makes RNNs painfully slow to train, but this makes sure the model keeps notes of the order of words it is processing. <br>
Transformers do not make use of any architectural relations (no notion of word order), and the model does not have any sense of position/order of the tokens/words being processed. They take all the embeddings into processing at once, vastly improving it’s parallelization potential (and therefore speeding things up), the downside is that they lose critical information related to order of words in our sequence.

### Why do positions matter ?
NLP tasks are dependent on the order of words to a great extent, hence it is crucial to supply some context to the model as well. <br>
To soak in context and to analyze relations with neighbouring words, we make use of positional encoding; describing the position of a token by a unique representation.

**Context** of words affect its meaning.
Example :
1. Kashmir Apples are in great demand. due to their superior vitamin content.
2. Apple is an American Multinational Corporation, known for innovative products for personal computing devices.

The word “Apple” has very different meanings, which can only be interpreted based on the context. <br>
Our representation has to be “rich and deeper” to incorporate information that blends in the context.

### Why not use index value ? Why don’t we assign the 1ˢᵗword P₀ , 2ⁿᵈ word P₁ and so on….

1. This <strong>distorts the embedding values</strong> for tokens that appear late in the sequence, giving them a higher value. This high value may not indicate anything meaningful, impacting our learning.
2. For extremely long sequences, the index can grow to a large number which the model might confuse with a very large input, causing other issues like <strong>gradient overflow</strong>.
3. Our model can face examples with <strong>sequence length</strong> greater than what it had observed in the training set. In addition, the model may not have seen any sample with a <strong>specific length</strong>, which hurts generalization,
4. Having a length-related encoding scheme makes the <strong>process dependent on the length of the sequence</strong>. This becomes an issue as sequences by nature are of varying length.

Each position is mapped to a vector, and the output of the positional layer is a matrix; wherein each row represents an encoded object of the sequence summed with its positional information
{ NOTE : this information is not concatenated }.

## Requirements of the encoding
Does not depend on any characteristics of the input, be it length or semantic meaning. <br>
Distance of embeddings between time steps should be consistent across sentences with different lengths. That is only possible when embeddings are obtained by using something fundamental, in this case sinusoidal functions.

## How to compute this encoding ?
Instead of a single number to denote the position, transformers make use of multi-dimensional vectors. This encoding is supplied as a part of the input, equipping each token with information about its position in the sequence.

<figure style="text-align: center;">
  <img src="https://miro.medium.com/v2/resize:fit:440/format:webp/1*uF0D-vwr_Xs3vMVae1SrGg.png" alt="Illustration" style="max-width: 100%;" />
  <figcaption><em>sinusoidal function mapping</em></figcaption>
</figure>

pos : position of the word/token in the sequence <br>
i : refers to one of the embedding dimensions <br>
d : size of position embedding, equal to size of word embeddings <br>
While “d“ is fixed, pos and i vary

A sinusoid is repetitive in nature, and therefore the positional encodings might repeat after a few token right ? <br>
Take note of the “i” component of the sinusoid. It can be understood as a component which affects the frequency of our embedding function. As “i” varies, we get different values for different dimensions.

<figure style="text-align: center;">
  <img src="https://miro.medium.com/v2/resize:fit:440/format:webp/1*ToFuz5h6IVQSMPVEoIV1qA.png" alt="Illustration" style="max-width: 100%;" />
  <figcaption><em>Simple visualization of how frequency alters the position encoding</em></figcaption>
</figure>

If two points are close by on the curve, they will remain close even at slightly higher frequencies. The difference is noticeable only at high frequency curves, where even for small variations, the output can wildly differ. leading to very different vectors.

> In reality, both sin and cosine functions are used to generate embeddings. <br>
> Even positions : sin <br>
> Odd positions : cosine

To add the positional embedding to our input embedding, we must note that the dimensions for the two have to be compatible. Hence “d” for the positional encoding is equal to the “d” of the word embedding. In the original paper, <strong>d is set at 512</strong>. <br>
To avoid losing this position/order information deeper down the network, transformers make use of residual connections, giving deeper layers access to this encoding. <br>
Another advantage of using sinusoidal functions to create embeddings is that these can be used even for sequences larger than the ones seen during training, as extrapolation is simple.

This can be simply written as :

```
class PositionalEncoding(nn.Module):

    def __init__(self, d_model, max_sequence_length):
        super().__init__()
        self.max_sequence_length = max_sequence_length
        self.d_model = d_model

    def forward(self):
        even_i = torch.arange(0, self.d_model, 2).float()
        denominator = torch.pow(10000, even_i/self.d_model)
        position = torch.arange(self.max_sequence_length).reshape(self.max_sequence_length, 1)
        even_PE = torch.sin(position / denominator)
        odd_PE = torch.cos(position / denominator)
        stacked = torch.stack([even_PE, odd_PE], dim=2)
        PE = torch.flatten(stacked, start_dim=1, end_dim=2)
        return PE
```
For more on Transformers, I’ll publish another blog in a day or two !! <br>

Checkout more on my 
<a href="{{ site.twitter_url }}" 
  target="_blank" 
  rel="noopener noreferrer"> 
Twitter page 
</a> 

References
1. <a href="https://kazemnejad.com/blog/transformer_architecture_positional_encoding/#what-is-positional-encoding-and-why-do-we-need-it-in-the-first-place" 
  target="_blank" 
  rel="noopener noreferrer">
  Blog by Kazemnejad
  </a>
