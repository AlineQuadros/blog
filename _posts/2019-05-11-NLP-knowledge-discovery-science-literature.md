---
layout: post
title:  "Using NLP and data-mining for knowledge discovery"
author: Aline
categories: [ NLP, data-science ]
image: assets/images/nlp.jpg
featured: true
hidden: false
toc: true
beforetoc: "NLP gives scientists the possibility to connect precious information that is scattered over thousands of publications"

---

I'm absolutely **in love with NLP**, and I'm excited to see how this field is progressing fast.
NLP has many diverse applications, but I'm particularly interested in the data-mining of scientific literature.  

This small project was my first attempt to explore NLP and see how much knowledge I could extract just by analyzing the information contained in titles and abstracts of publications.
Because I was working with _mangrove ecology_ at the time, I naturally applied it to understand more of mangroves. Nonetheless, the procedures I'll describe below can be used to investigate any subjects.


The question was simple:

> How mangrove ecology has been studied and how has it changed over the years?

Here's how I did it:

* First stage was getting the data. My sources where three major databases of scientific literature: **PubMED**, **Web of Knowledge**, and **Scopus**. Was it necessary to use three databases? Well, yes, because each database indexes a diferent set of journals and publishers, so even if you use the exact same keywords you'll get a diferent set of publications. There's a lot of overlap, yes, but by using a combination of keywords I obtained the titles and abstracts of nearly* all published papers on the ecology of mangroves.

* Second stage was, you know, data wrangling: identification of duplicates, normalization of charsets, etc. The final dataset had **10834 records**.

* Third stage: build a corpora (including text normalization, stemming)

* Fourth stage is _where the fun starts_: To find out the most common used word in the titles.
Because I had the publication year of each publication (I grouped them per decade), I was able to detect trends in time. I assumed that this analysis would provide a good overview of how the interest on mangroves has been changing along time. The results look like this:


> Fun fact: The oldest publication about **mangrove ecology** that I could find was ...

That's it! I'm looking forward to getting more data and extracting more knowledge from published stuff

> *Note: I only analyzed publications in English, which is a pity. Most mangroves of the world occur in non-English speaking  countries (like Brazil ), so there's a lot published in other languages.


 <span class="spoiler">I just love NLP</span>
