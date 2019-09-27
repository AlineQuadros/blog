---
layout: post
title:  "Predicting the effects of global warming on the mountains of Austria"
author: aline
categories: [ ecology, data-science ]
image: assets/images/mountains-austria.jpg
tags: [featured]
---

This was one of my all-time-favorite experiments. We used mixed-models and the ___space-for-time___ approach to explore the connections between temperature, soil animals, and litter decomposition.


The idea of ___space-for-time___ is to identify natural gradients (**the
space**), with two or more contrasting conditions at the endpoints that represent
current and future scenarios (**the time**).

Our SFT approach to study the effects of temperature increase used the
natural environmental gradients of mountain slopes. Altitudinal gradients in mountains
provide an excellent opportunity to address the abiotic effects on
ecological processes under field conditions: some abiotic factors change
in a predictable way with increasing altitude, while biotic factors, such
as fauna and plant communities, remain similar, at least within altitudinal
ecotones (e.g. montane, alpine, nival).

**Here's the link for the full publication:**
<hr>
Jenny Faber, Aline Ferreira Quadros,  Martin Zimmer. *A Space-For-Time approach to study the effects of increasing temperature on leaf litter decomposition under natural conditions* Published in <a href="https://doi.org/10.1016/j.soilbio.2018.05.010">Soil Biology and Biochemistry 123, 250–256 (2018)</a>
<hr>

 **Illustration of our experimental design**

<img src="/blog/assets/images/faber_experimentaldesign.png">

Our experimentt was replicated in 5 mountains. There were 2 sites per mountain (diferent altitudes), 5 blocks of mesocosms per site, and 4 mesocosms per block (one for each level of 2 treatments).

This is a classic example of __nested data__ (mesocosms belong to blocks, which belong to sites, which belong to mountains...). Experiments like this are necessary to isolate (or to account for) the effects of other variables that might influence the results.

To analyze this data, I used a __mixed effect model__. Mixed effect models quantify the effects of the desired treatments (*fixed effects*) while taking into account the variation that is due to the influence of other, unknown factors (*random effects*). That's why the name "mixed".

To run a __mixed effects model__ in R you can use the packages `lme4` and `lmertest`. Here I used `lme4` and the function `lmer()` to compare 5 models. In `lmer()` you can use restricted maximum likelihood-fitted models (REML=true) to compare models that have different random effects (which was not my case).

```
library(lme4)

# define the random part of all mixed models
rand = "(1|site:bl)"

y_var = "weight_loss"

# define all possible models for model selection

fnull <<- as.formula(paste(y_var, "~", rand))                  # null model       
f1 <<- as.formula(paste(y_var, "~ altitude +", rand))          # 1-way main effects      
f2 <<- as.formula(paste(y_var, "~ fauna +", rand))             # 1-way main effects
f3 <<- as.formula(paste(y_var, "~ altitude + fauna + ", rand)) # 2-way main effects
f4 <<- as.formula(paste(y_var, "~ altitude * fauna + ", rand)) # 2-way interaction effects

m_null = lmer(fnull, data=urtica, REML=FALSE)
summary(m_null)
m_1 = lmer(f1, data=urtica, REML=FALSE)
summary(m_1)
m_2 = lmer(f2, data=urtica, REML=FALSE)
summary(m_2)
m_3 = lmer(f3, data=urtica, REML=FALSE)
summary(m_3)
m_4 = lmer(f4, data=urtica, REML=FALSE)
summary(m_4)

# Use ANOVA to pick the best minimum model
anova(m_3, m_1)
```

An important step in every linear modelling  is to check the distribution of residuals:

```
qqnorm(resid(m_3),  main = "Normal Q-Q Plot")
qqline(resid(m_3))
shapiro.test(residuals(m_3))
```

Once the residuals looks good, you can check the **model significance**, or **effect size**.

To calculate the EFFECT SIZEs of a mixed-effects model in R, the function `sem.model.fits()` from package `piecewiseSEM` can be used. The **marginal effect** indicate the % of variance explained by the fixed factors, while then **conditional effect** indicate the % of variance explained by the fixed and random factors.

```
library(piecewiseSEM)

var_total <- sem.model.fits(m_3)[6]              # conditional effect
var_fixed <- sem.model.fits(m_3)[5]              # marginal effect
var_random <- as.numeric(var_total - var_fixed)

```

Here's a plot of the data that shows our main findings: 

<img src="/blog/assets/images/faber_fig3.png"  style="width:70%">
