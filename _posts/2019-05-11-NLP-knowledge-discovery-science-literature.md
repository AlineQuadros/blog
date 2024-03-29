---
layout: post
title:  "Data-mining of scientific literature using NLP"
author: aline
categories: [ machine learning, data-science ]
image: assets/images/nlp.jpg
tags: [sticky]

---

NLP gives scientists the tools to connect precious information that is scattered over thousands of publications. I'm absolutely **in love with NLP**, and I'm excited to see how this field is progressing fast. NLP has many diverse applications, but I'm particularly interested in the data-mining of scientific literature.  

This small project was my first attempt to explore NLP and see how much knowledge I could extract just by analyzing the information contained in titles and abstracts of publications. Because I was working with _mangrove ecology_ at the time, I naturally applied it to understand more of mangroves. Mangroves are threatened ecosystems but largely understudied, thus any information we can extract about them can be helpful to future. Nonetheless, the procedures I'll describe below can be used to investigate any subject.


S question was simple:

#### How has mangrove ecology been studied and how has it changed over the years?

This is an example of **unsupervised learning**. More specifically, I . So I used two techniques of **topic modelling**: **Singular value decomposition** and **LDA**.

Here's the steps of this analysis:

* First stage was getting the data. My sources where three major databases of scientific literature: **PubMED**, **Web of Knowledge**, and **Scopus**. Was it necessary to use three databases? Well, yes, because each database indexes a diferent set of journals and publishers, so even if you use the exact same keywords you'll get a diferent set of publications. There's a lot of overlap, yes, but by using a combination of keywords I obtained the titles and abstracts of nearly* all published papers on the ecology of mangroves.

* Second stage was, you know, data wrangling: removal of duplicates, removal of punctuation, normalization of charsets, etc. The final dataset had **21686 records** with Titles and **999 records** with Titles and Abstracts.

* Third stage was build a corpora (including text normalization, stemming)

* Fourth stage is _where the fun starts_: To find out the most common words in the titles.
Because I had the publication year of each publication (I grouped them per decade), I was able to detect trends in time. I assumed that this analysis would provide a good overview of how the interest on mangroves has been changing along time.




### visualization

The tool `PyLDAVis` . I learned about it here: <ahref="https://nlpforhackers.io/topic-modeling/">https://nlpforhackers.io/topic-modeling/</a>



That's it! I'm looking forward to getting more data and extracting more knowledge from published stuff

*Note: I only analyzed publications in English, which is a pity. Most mangroves of the world occur in non-English speaking  countries (like Brazil ), so there's a lot published in other languages.*
