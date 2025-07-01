---
layout: guides
title: amazing reads
description: notes, blogs and writings compiled over the years, primarily shared via discord by Aryan Pandey from IIIT Jabalpur
---

Some really cool blogs/guides compiled over the years

1. [Overfitting explained simply](https://www.ibm.com/cloud/learn/overfitting)

2. [Leetcode for Data Science](https://www.stratascratch.com/)

3. [Fundamentals of ML, IITD](https://www.cse.iitd.ac.in/~parags/teaching/2013/au13/csl341/)

4. [Why read, a blog by an alum of 2013-17 batch](https://indiaai.gov.in/article/read-and-watch-lectures-to-build-a-foundation)

5. [ML Primer, using xkcd templates](https://www.confetti.ai/assets/ml-primer/ml_primer.pdf)

6. [DL Playlist by Prof Bryce, must watch alongside IDL by Goodfellow](https://www.youtube.com/playlist?list=PLgPbN3w-ia_PeT1_c5jiLW3RJdR7853b9)

7. [Ensemble Learning](https://fritz.ai/ensemble-learning/)

8. [Revise ML through Stanford cheatsheets](https://stanford.edu/~shervine/teaching/cs-229/)

9. Activation Functions
- [Comparing activations, link broken perhaps](https://wandb.ai/shweta/Activation%20Functions/reports/Activation-Functions-Compared-With-Experiments--VmlldzoxMDQwOTQ)
- [Short discussion on ReLU and Sigmoid](https://wandb.ai/ayush-thakur/dl-question-bank/reports/ReLU-vs-Sigmoid-Function-in-Deep-Neural-Networks--VmlldzoyMDk0MzI)

10. [Batch Norm explained](https://e2eml.school/batch_normalization.html)

11. AutoEncoders and Variational Auto Encoders
- [Watch vids 44 - 50](https://www.youtube.com/playlist?list=PL6Xpj9I5qXYEcOhn7TqghAJ6NAPrNmUBH)
- [Variational AutoEncoders](https://jaan.io/what-is-variational-autoencoder-vae-tutorial/)
- [Paper, Tutorial on VAE](https://arxiv.org/pdf/1606.05908.pdf)

12. Do we Use Dropouts with CNN's ?
- [Experimenting with networks](https://nchlis.github.io/2017_08_10/page.html)
- [KdNuggets Blog](https://www.kdnuggets.com/2018/09/dropout-convolutional-networks.html)

12. [Deep Dream](https://www.americanscientist.org/sites/americanscientist.org/files/20151081452611494-2015-11Hayes.pdf). Firing select neurons in the network and propogate changes to the input image, gives weird but interesting patterns.

13. [Exhaustive Blog on LSTMs](https://towardsdatascience.com/tutorial-on-lstm-a-computational-perspective-f3417442c2cd)

14. [Enforcing Constraints at Amazon](https://www.amazon.science/blog/how-to-compute-the-optimal-way-to-package-amazon-products). Really cheeky way to enforce constraints.

- If a package size had no reported damage for the product, dummy datapoints were added to the DB such that even for bigger package sizes no damage was reported. 
- Similarly to enforce the negative constraint, if damage was reported for a particular size, all sizes below it were added in the DB with damage = True