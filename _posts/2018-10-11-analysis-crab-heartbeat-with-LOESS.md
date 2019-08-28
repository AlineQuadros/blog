---
layout: post
title:  "Analysis of crab heartbeats using LOESS for curve smoothing and peak detection"
author: Aline
categories: [ data-science ]
image: assets/images/crab-tree.jpg
featured: false
hidden: false
beforetoc: "beforetoc"
toc: true
---


Scientists worldwide have been recording and analyzing the heartbeats of all animals you can think of. From the largest to the tiniest. In the same way your doctor can learn a lot from your health by monitoring your heart frequency, scientists can learn a lot about the physiology of animal too.

A couple years ago I collaborated on a project where scientists wanted to see how *mangrove crabs* were being affected by global warming. They used a system that records the heartbeats by an infra-red sensor attached to the crabs’ carapace connected to an oscilloscope. ) to get the crab's heartbeats by attaching small sensors to the crabs' back (_fun fact_ the crab's heart is not located on its "chest" but rather on its "back"). The crabs were then confined to a space where one or more environmental features were being manipulated. In this case, it was the water temperature. The experiment can be repeated with diferent individuals, different species, under diferent conditions, depending on the *research question*.

<img src='/AlineQuadros/assets/images/signal1.png'>  

> Heart frequency is one of the most important physiological parameters of animals. Through the analysis of heartbeats, scientists can understand the conditions that stress animals, and how much of each stress they support until reaching a critical condition. This information is latter used to predict how different species will respond to environmental changes, such as global warming or ocean acidification.  



Commercial devices such as Picoscope convert the analogical signal coming from the animal and convert it to a digital form that can be saved in text files and analyzed. Well, after an experiment like this is done, a researcher may end up with dozens hours of recordings per individual, and dozens of individuals.  

*Example of the data structure derived from an oscilloscope with two channels (simultaneous recordings of two sensors):*

```
Time	Channel A	Channel B
(s)	(V)	(V)

0.000000	-0.4205451	0.1358074
0.001334	-0.4652547	0.1329081
0.002668	-0.5101169	0.1301614
0.004002	-0.5548265	0.1274148
0.005336	-0.5548265	0.1245155
0.006675	-0.5548265	0.1217689
0.008004	-0.5548265	0.1190222
0.009338	-0.5996887	0.1161229

```
After exporting the data from the oscilloscope and importing it into R, applying the LOESS function is pretty simple and R has a built-in function `loess()`   ready to use. The most important parameter that is needed to tune the model is *span*. The parameter *span* ranges between 0 and 1, and controls the degree of smoothing.

```

# apply the LOESS function trying diferent values of span
y.loess <- loess(y ~ x, span=0.01, data.frame(x=time, y=beats))

#
y.predict <- predict(y.loess, data.frame(x=time))

# save the predicted y values (smoothed) in a dataframe
mat2 <- data.frame(time, y.predict)

# find valleys
b<- findValleys(y.predict)
# select only the rows corresponding to peaks from the original dataframe
piks <- mat2[b, ]
piks <- piks[piks$y.predict <=0,]

```

And this is like the signal looks like after processing:  


<img src='/AlineQuadros/assets/images/signal2.png'>


>The top panel shows the raw signal (gray lines) and the curves (red) and peaks (green dots) detected using LOESS (local non-parametric regression). From there, we calculate the number of peaks per unit of time (beats per minute, per example), as shown in the second panel. This series of data is then plotted against the variation of another factor (water temperature, for instance) to monitor the animal's response along time.

The tricky thing is that, the heartbeat alone tells little about an animal's condition. The interesting question is to see how the heart frequency changes upon the changes in the variables being manipulated (temperature, gas concentration, light, etc.). These variables, in turn, are being recorded by their own specific sensors, and the researcher will have to integrate these data at some point.

In situations like this it is really handy to know how to code in languages like R and Python. While the modelling required for this project (a few simple linear models), there's a lot of data import and export, merging, and transformations.


This project has lots of interesting findings, and the partial results were presented in November 2018 by the author of the study, **Pedro Jimenez**, and can be read <a href="/AlineQuadros/assets/images/study_pedro.pdf"> here</a>. There's more coming up!
