---
layout: post
title:  "Analysis of crab heartbeats using LOESS"
author: aline
categories: [ data-science ]
image: assets/images/crab-tree.jpg
tags: [featured]
---

LOESS (aka local regression) is an ideal tool for the analysis of physiological signals, and can be used on the raw data for curve smoothing and peak detection.

Scientists worldwide have been recording and analyzing the heartbeats of all animals you can think of. From the largest to the tiniest ones. In this way, scientists learn a lot about the physiology of animals, in the same way doctors learn a lot about your health by monitoring your heart frequency.

A couple years ago I collaborated on a project where scientists were interested in how *mangrove crabs* were affected by global warming. They recorded the heartbeats using infra-red sensors attached to the crabs’ carapace and an oscilloscope (_fun fact_ did you know that the crab's heart is not located on its "chest" but rather on its "back"?!). The crabs were then confined to a space where one or more environmental features were being manipulated. In this case, it was the water temperature. Experiments like this can be repeated with diferent individuals, species, under diferent conditions, depending on the *research question*.

<img src='/blog/assets/images/signal1.png'>  

> Heart frequency is one of the most important physiological parameters of animals. Scientists use it to understand the conditions that stress animals, and how much of each stress they can tolerate before reaching a critical condition. This information is latter used to predict how different species will respond to environmental changes, such as global warming or ocean acidification.  


Commercial devices such as Picoscope convert the analogical signal coming from the animal and convert it to a digital form that can be saved in text files and analyzed. After an experiment like this is done, a researcher may end up with thousands of hours of recordings per individual, and dozens of individuals.  

**Example of the raw data obtained from an oscilloscope with two channels (simultaneous recordings of two sensors):**

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

After exporting the data from the oscilloscope and importing it into R, applying the LOESS function is pretty simple and R has a built-in function `loess()` ready to use.

The most important parameter that is needed to tune the model is *span*. The parameter *span* ranges between 0 and 1, and controls the degree of smoothing.


```
# apply the LOESS function
y.loess <- loess(y ~ x, span=0.01, data.frame(x=time, y=beats))

y.predict <- predict(y.loess, data.frame(x=time))

# save the predicted y values (smoothed) in a dataframe
pred_dataset <- data.frame(time, y.predict)

# find peaks
peaks <- findPeaks(y.predict)

# alternatively, you can also find valleys
valleys <- findValleys(y.predict)

```

And this is how the signal looks like after processing:  


<img src="/blog/assets/images/signal2.png" style="width:80%">


The top panel shows the raw signal (gray lines), and the curves (red) and peaks (green dots) detected using LOESS (local non-parametric regression). From that we can calculate the number of peaks per unit of time (beats per minute, for example), as shown in the second panel. This series of data is then plotted against the variation of another factor (water temperature, oxygen, etc.) to monitor the animal's response along time.

The tricky thing is that the heartbeat alone tells little about an animal's condition. The interesting question is how the heart frequency changes upon changes in the variables being manipulated (temperature, gas concentration, light, etc.). These variables, in turn, are being recorded by their own specific sensors, and the researcher will have to integrate these data at some point. In situations like this it is really handy to know how to code in languages like R and Python. While the statistics and models required for this project were relatively simple, there's a lot of data import and export, merging, and transformation.

**Example of a data file containing the simultaneous readings of water temperature and air saturation**

```
Date & Time	       Timestamp code	  temp [°C]	O2 [% air saturation]
09-Dez-14 1:12:13  PM	3587778733	  27.07	    91.2
09-Dez-14 1:12:24  PM	3587778734 	  27.08	    91.1
09-Dez-14 1:12:35  PM	3587778735 	  27.09	    91.0
09-Dez-14 1:12:46  PM	3587778736 	  27.08	    91.1

```
As you can see, here the data was obtained regularly in 10 second intervals, whereas the heartbeat data (which is also in another format) is calculated in the scale of minutes. That's another example of the usefulness of LOESS. After the data (signals) were smoothed, **interpolation** can be used to predict the values at a given time.


<table>
<tr>
<td>
This project has lots of interesting findings, and the partial results were presented in November 2018 by the author of the study, **Pedro Jimenez**, and can be read <a href="/blog/assets/images/study_pedro.pdf"> here</a>. There's more coming up!
</td>
<td>
<embed width="291" height="307" name="plugin" src="/blog/assets/images/study_pedro.pdf" type="application/pdf">
</td>
</tr>
</table>
