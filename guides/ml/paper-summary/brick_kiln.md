---
layout: guides
title: Brick Kiln Detection IITGn
description: Was thinking of applying to the sustainability lab at IITGn, this was one of the papers I read, on detecting brick kilns using satellite imagery.
usemathjax: true
---

# Brick Kiln Detection and Compliance monitoring with Geospatial Data

Read this quite a while ago, while travelling back to campus for the very last time for PBI presentations (Project Based Internships, basially I swapped a semester with an internship). <br>
<a href="https://arxiv.org/abs/2412.04065" target="_blank" rel="noopener noreferrer">
  Arxiv Link
</a>
 to the very detailed paper by The Sustainability Lab from IIT Gandhinagar, talking about identifying Brick Kilns from satellite imagery.


## Why ?
Well there isn't much information available outside of government surveys on how many kilns, and of what type operate around the country. <br>
This paper, by analyzing kilns across the Gangetic plain proposes an extendable methodology to monitor brick kiln type over time, and compliance with rules / guidelines. <br>
Surveys are expensive and can be gamed (bribing to avoid detection, too tiring to drive out and survey these kilns)

* Table of Contents
{:toc}

## What are Brick Kilns ?
Basically where soft clay/earth is heated for extended periods of time to make bricks, which are then used by the construction industry. <br>
Usually operate on solid fuels like charcoal and coal, and employ a lot of people.

Soil required for brick making requires moderate clay content, which is abundantly found around riverbanks, leading to establishment of many kilns around river banks. 
(this is a violation of norms set by various Govt. agencies as Particulate matter and other pollutants can affect a larger population through water sources). <br>
Kilns typically operate for around 6months in a year, avoiding the monsoon.
### Types of Kilns

Fixed Bull Trench Kiln has bricks stacked in the annular space between the inner and outer walls of the kiln.

<strong>Circular Fixed Chimney Bull Trench Kiln</strong> : In a circular shape. 

<strong>Fixed Chimney Bull Trench Kiln</strong> : Older technology, with limited air supply causing incomplete combustion and thereby creation of harmful product like Carbon Monoxide $CO$ and a lot of particulate matter.

![Illustration of a circular FCBK](https://www.researchgate.net/publication/336454204/figure/fig2/AS:11431281432217299@1746848376354/Schematic-diagram-of-traditional-fixed-chimney-Bull-trench-clay-made-brick-burning-kilns.tif)

<strong>Zigzag </strong> : Arrange the trench in a zigzag manner to better facilitate movement of heat, reduces particular matter and $CO$ production by 70%.

### Guidelines by the Government
Identified cities which fail national standards, refers to them as 'non-attainment' cities, and wants brick kilns to shift to 'Zigzag' firing technology and shift to cleaner fuels (natural gas).

### Brick Kiln Siting Rules

Brick Kilns produce a lot of smoke, which affects plants, animals and life around. To strike a balance between economic activity and health, certain guidelines / rules have been setup as follows : 

- Pollutants like SO~2~ Sulphur Dioxide and HF Hydrogen Flouride negatively impact flowering of fruit trees, hence a minimum distance from fruit orchads must be maintained.
- Minimum distance from residential areas, hospitals and schools. 
- Minimum distance from important structures affected by soil erosion like railway lines and highways. <br>
This is because raw material for bricks if often the topsoil dug upto 2meters deep, this can alter rainwater flow and cause structures nearby to cave-in.

## Object detection
The bounding boxes for a particular object can be drawn in two ways : 

### Axis Aligned Bounding Box
define a rectangular box with edges parallel to the horizontal and vertical axes of the image.

### Oriented Bounding Boxes
Well you guessed it right, bounding boxes which are not aligned with the image, but instead the object.
This help better with localization of objects, as we need to understand the alignment of the -object-to-identify-.

### Satellite imagery (data source)
Pretty interesting <br>
Satellites have low resolution, but provide an image for a large area at once. Image sizes of 4096x4096 are generated, which are then divided into sub-images of size 640x640 with a 64pixel overlap (which corresponds to roughly 300meters). <br>
An <strong>overlap is necessary</strong> to avoid cutting out features of interest that happen to be around the edges of an image-patch.

>
This was an interview question for a Data Science position I applied to last year.
Discussing why we need to have an overlap between image patches, which is especially the case for large images.

Imagery data from the first quarter of 2024 is considered due to the following reasons
- Kilns operate in full swing
- Favourable weather conditions with minimal cloud coverage

## Curating the train set and training
Select regions for initial labelling based on high pollution metrics, high population in non-attainment cities.

### Annotation process
A region of interest is divided into set of grids, and grid size was chosen to be 1kmx1km to balance annotation speed and accuracy (identifying brick kilns by human).
Within each cell, the authors draw Oriented Bounding Boxes around the identified brick kilns.

<figure>
  <img src="https://hackmd.io/_uploads/S1EzUn0Nxe.png" alt="image annotation process">
  <figcaption><em>Figure 1:</em> Image annotation process</figcaption>
</figure>

### Accuracy Metric
<strong>Weighted mAP50</strong>, which is the weighted (by class) Mean Average Precision at 50% IoU for the bounding boxes. <br>
IoU is calculated for the bounding boxes, measuring overlap between model predicted box and the ground truth box.

### Precision and Recall
The first time I wrote about precision and recall metrics, I got a call from my manager within a few hours to edit it, I published it around 2AM and got a ping the very next morning 😭 <a href="../../in-prod/ds_search" target="_blank" rel="noopener noreferrer">
  this particular note
</a>

Similarly, here Precision is defined as, out of all boxes labelled by the model, how many are actually brick kilns. <br>
Recall is defined as, out of all ground truth kilns, how many could we identify.

A very simple way is to understand, <strong>Precision is the inclusion error</strong> (errors in what the model classifies) and <strong>recall is the exclusion errors</strong> (what the model fails to identify).

### Improving Generalization
Leave one out strategy is used, where we train on n-1 regions and then evaluate the model on the left-out region. <br>
This is iteratively done till we cover all regions in the test set.

## Improving the dataset
From kilns identified by the model, that were not previously in our train dataset, we use this as augmentation data to further enhance our data, after a few improvements.

1. If bounding box is incorrect (no brick kiln seen by human evaluators), discard
2. If bounding box is misaligned / partially covers kiln, realign
3. If kiln-type is incorrect, correct the label. 

Out of around 27k brick kilns identified by the model, human annotators labelled 15k to be true, with the precision being 58%, this might sound low, but substantially reduces human effort. <br>
To label the entirety of region sampled (in this case the Federal state of Uttar Pradesh), human annotators would require 2000hrs, validating 27k samples takes the following time : 

$$
\begin{align}
\text{Time to check single model output} &= 15\text{ s} \\
\text{For all 27k model outputs} &= 15 \times 27{,}000 \text{ s} \\
  &= \frac{15}{60} \times 27{,}000 \text{ min} \\
  &= \frac{15}{60 \times 60} \times 27{,}000 \text{ hr} \\
  &= 112 \text{ hr}
\end{align}
$$

##### wow using latex was fun

<hr>

### re train model on augmented data
Combine initial dataset with the 15k newly identified and hand validated kilns. <br>
Although checking accuracy metrics for a model trained on this data doesn't make a lot of sense, as the eval set would have strong overlap with the train set, the authors do see a strong increase in both accuracy and precision.<br>
Strong correlation between kilns identified and govt data (by UP Pollution Control Board) was seen (Pearson correlation score of 0.94)

<figure>
  <img src="https://hackmd.io/_uploads/BJGR40CVel.png" alt="Model outputs compared to UP Govt data">
  <figcaption><em>Figure 2:</em> Model outputs compared to UP Government data.</figcaption>
</figure>

## Compliance monitoring

Brick kilns have to be compliant in the following two ways
1. follow minimum distance guidelines from residential areas, hospitals, schools and important structures like railway lines and highways.
2. shift from old polluting layouts to zigzag layout over time

Tracking these are very simple with the methodology proposed by the authors where we identify the type of brick kiln alongside the brick kiln.

Following patterns were observed:
- most kilns operate in clusters (probably to leverage economies of scale, and manage logistics)
- Proximity to population is a recurrent issue across all studied areas, with less violations for the states of Punjab and Haryana which can be due to simpler / relaxed local regulations.

>
monitoring compliance over time ===> helps set realistic deadlines for future initiatives concerning the same group of industries.

### Obtain year of establishment for a kiln
For each identified kiln, it could either have been a re-furbished one (switching types) or a brand new one.<br>
Authors use a simple binary search to get the year of establishment over the time period of 2010-22, checking first for the year 2016 and then updating the search pivot accordingly. <br>
Used to monitor conversions of brick kilns.

<figure>
  <img src="https://hackmd.io/_uploads/B1s0feJHxx.png" alt="Model outputs compared to UP Govt data">
  <figcaption><em>Figure 3:</em> Zigzag technology gains popularity in Delhi airshed </figcaption>
</figure>

### Quantify pollution
The paper quantifies air pollution from these brick kilns using a CTM, Chemical Transport Model, which from my naive understanding tracks how pollutants move through the air.

## Challenges with using satellite imagery
1. Low resolution from satellite imagery
2. Low emission intensity of brick kilns, as they operate on slow and long cooking of bricks.

Brick kilns produce on average <strong>0.8tonnnes</strong> of CO every day, as compared to a Coal fired Thermal plant emitting <strong>48tonnes</strong> of CO. The paper also mentions that how even with such a <strong>weak emission rate, brick kilns are more potent than Thermal plants</strong>, due to the sheer number of brick kilns operating in the country.

<hr>

## Funny correlation with a childhood story

Comparing the model identified brick kiln counts with the government data, we can have the following two cases
1. model_count > govt_survey_count
2. model_count < govt_survey_count

As the govt survey was done quite a while ago as compared to the data used to train the model, the authors point to two logical causes

1. For model_count being higher
	1.1 brick kilns opened between sruvery date and satellite image capture date
	1.2 high inclusion errors, where we falsely identify brick kilns

2. For model_count being lower
	2.1 brick kilns closed down between survey date and satellite imagery date
	2.2 high exclusion errors, where the model fails to identify existing brick kilns


This reminded me of the [Akbar birbal story of counting crows](https://sfipodcast.com/akbar-birbal-counting-crows-and-growing-wit-ep-152/)