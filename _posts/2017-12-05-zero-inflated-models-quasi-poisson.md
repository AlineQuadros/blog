---
layout: post
title:  "Dealing with zero-inflated models using quasi-Poisson regressions"
author: aline
categories: [ data-science, ecology ]
image: assets/images/zero-inflated.jpg

---

Zero-inflated data is really common in ecology but rather difficult to model.

PQL is a flexible technique that can deal with non-normal data, unbalanced design, and crossed random effects. However, it produces biased estimates if your response variable fits a discrete count distribution, like Poisson or binomial, and the mean is less than 5 - or if your response variable is binary

```
site   temperature period    sampling_date    count
site1  high        day       1                8      
site1  high        day       1                0   
site1  high        night     1               14    
site1  high        night     1                1  
site1  low         day       2                0    
site1  low         day       2                0  
site1  low         night     2                1    
site1  low         night     2                0   
site2  high        day       3                0     
site2  high        day       3                0      
site2  high        night     3                0  
site2  high        night     3                0  
site2  low         day       4                0  
site2  low         day       4                0   
site2  low         night     4                0   
site2  low         night     4                0
```

#### First test: **LMER**


```
lmer.1 <- lmer(sqrt(density.median) ~ site * temp * period + (1|sampling_date), data = dt)

qqnorm(resid(lmer.1), main = "Normal Q-Q Plot")
qqline(resid(lmer.1))
```
Look at the residuals (it's a disaster!!):

#### Second try. Use **Generalized linear models** with Poisson distribution:

```
glmer.1 <-  glmer(counts  ~ site * temp * period + (1|sampling_date), data= dt, family="poisson")

overdisp_fun(glmer.1)
```

The function `overdisp_fun()` checks whether the model's residuals are **overdispersed**
And indeed:
#        chisq        ratio          rdf            p
# 5.806295e+02 3.651758e+00 1.590000e+02 2.438708e-49

#### Third try: Quasi-poisson. Using function `glmmPQL()` from package
