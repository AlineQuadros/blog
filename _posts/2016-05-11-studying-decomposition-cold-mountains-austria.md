---
layout: post
title:  "Predicting the effects of global warming on the mountains of Austria"
author: Aline
categories: [ ecology, data-science ]
image: assets/images/mountains-austria.jpg
featured: false
hidden: false
beforetoc: "Temperate mountains hold precious stocks of organic carbon"
toc: true
---


Our experimental design was like this

5 mountains to replicate the study
2 sites per mountain (diferent altitudes)
5 blocks of mesocosms per site
4 mesocosms per block

<img src="faber_experimentaldesign.png">

> Illustration of our experimental design

This is a classic example of __nested data__ (mesocosms belong to blocks, which belong to sites, which belong to mountains...). Experiments like this are necessary to isolate (or to account for) the effects of other variables that might influence the results. So, in our case,

To analyse this data, I used a __mixed effect model__. Mixed effect models (that's why the "mixed") quantify the effects of the desired treatments (*fixed effects*) while taking into account the variation that is due to the influence of other, unknown factors (*random effects*).

Here's how to run a __mixed effect model__ in R:

```
library(doBy)
library(lme4)
library(piecewiseSEM)
library(lattice)
library(car)
library(ggplot2)
require(MASS)
require(readr)
require(lubridate)
require(lmerTest)

# define the random part of all mixed models
## mixed models with random intercept model and block nested WITHIN sites
## Use restricted maximum likelihood-fitted models (REML=true) to compare models that differ in random effects
rand = "(1|site:bl)"
fixe = "weight_loss_per_day_mg"

  # define all possible models for model selection
  fnull <<- as.formula(paste(fixe, "~",rand))                # null model       
  f1<<-as.formula(paste(fixe, "~ altitude +",rand))          # 1-way main effects      
  f2<<-as.formula(paste(fixe, "~ fauna +",rand))             # 1-way main effects
  f3<<-as.formula(paste(fixe, "~ altitude + fauna + ",rand)) # 2-way main effects
  f4<<-as.formula(paste(fixe, "~ altitude * fauna + ",rand)) # 2-way interaction effects

  m_null=lmer(fnull, data=urtica, REML=FALSE)
  summary(m_null)
  m_1=lmer(f1, data=urtica, REML=FALSE)
  summary(m_1)
  m_2=lmer(f2, data=urtica, REML=FALSE)
  summary(m_2)
  m_3=lmer(f3, data=urtica, REML=FALSE)
  summary(m_3)
  m_4=lmer(f4, data=urtica, REML=FALSE)
  summary(m_4)

  #
  anova(m_3, m_1)

  # don'- [ ]
  qqnorm(resid(m_3),  main = "Normal Q-Q Plot - URTICA")
  qqline(resid(m_3))
  shapiro.test(residuals(m_3))

  ## EFFECT SIZE marginal= variance explained by fixed factors and conditional=explained by fixed and random
varfix <- sem.model.fits(m_3)[5]
vartot <- sem.model.fits(m_3)[6]
varrand <- as.numeric(sem.model.fits(m_3)[6]) - as.numeric(sem.model.fits(m_3)[5])

```
