---
layout: post
title:  "Modelling the production of mangrove forests with PLS-R and PCA"
author: aline
categories: [ data-science, ecology ]
image: assets/images/mangrovesfranciscoblancosstock.jpg
tags: [featured]

---

Combining PLS-R and PCA to predict how biomass production might change depending on the vegetation features of mangroves.

Mangroves are fascinating coastal ecosystems. Unlike tropical or temperate forests, mangroves are not well studied so there's a lot we don't know about them. In 2017 I undertook a one-year long project to fill some gaps, and this post has a brief presentation of one of its outcomes. The full paper containing the results is this:

<hr>
Aline Ferreira Quadros, Inga Nordhaus, Hauke Reuter, Martin Zimmer. *Modelling of mangrove annual leaf litterfall with emphasis on the role of vegetation structure.* Published in <a href="https://doi.org/10.1016/j.ecss.2018.12.012">Estuarine, Coastal and Shelf Science 218, 292–299 (2018)</a>
<hr>

#### Background  


Initially, my goal was to review published and unpublished papers (basically, a manual text-mining, you can read more about it <a href="https://alinequadros.github.io/AlineQuadros/playing-data-detective/">in this post</a>) and extract some specific information about mangrove plants species, to build a structured dataset of traits (which you can access here: <a href="https://doi.org/10.3897/BDJ.5.e22089">Dataset of "true mangroves" plant species traits</a>).


>__Traits__ are quantitative and qualitative features of living beings that help researchers to model the ecology and behavior of species, and are extremely useful in **quantitative ecology** these days.


While I was doing the reviews, I found two interesting sets of studies about the mangroves of Ajuruteua (north of Brazil). One set contained studies about the **vegetation structure** of mangrove stands (tree height, diameter, density, etc.), and another set had estimates of the **litterfall production** of these and other mangrove sites. Some studies even contained both information (the list of studies is at the end of this post). While most studies explained the distribution of litterfall along the year, based on climatic features, I noticed that none attempted to predict **how much** litterfall could be produced. That's how I came up with this research question:

>Can we predict the annual litterfall of a given mangrove site, just by knowing the features of the vegetation?   


But why is it important to predict annual litterfall the first place? Litterfall is a big component (and a proxy) of the annual aboveground production of a forest, and this information is used to track how fast (and efficiently) the forest is growing, the amount of carbon and important nutrients that is made available to all trophic levels.

Mangroves store huge amounts of carbon in the sediments. This is so special in terms of global ecology that this carbon received a special name: <a href="https://en.wikipedia.org/wiki/Blue_carbon">Blue Carbon</a>. Much of this carbon comes from the freshwater inflow from rivers, but a lot comes from the decomposition of the leaves shed by the trees everyday, __the leaf litterfall__.


<img src="/blog/assets/images/mangrove_npp.png">

Every year, mangroves shed about 57% of the total Net Primary Production (NPP). Globally, this represents an input of organic matter of about  6.7 Mg per Ha. In the mangroves of Ajuruteua, which are highly productive, this value is even bigger: 9.5 Mg per Ha.


Could we predict how much biomass a mangrove will produce if we have basic information on the structure of its trees?
And _the answer is YES!_ You can read the full publication <a href="https://doi.org/10.1016/j.ecss.2018.12.012">here</a>, or <a href="mailto:aline.fquadros@outlook.com"> send me an email</a> if you don't have access to it.


Why did I choose to use PLS-R? Well PLS-R is __a fantastic tool for anyone dealing with biological data__. That's because biological features are usually orthogonal (cross-correlated). That hampers, for instance, the use of more common techniques, such as (multivariate) linear regressions. If PLS is completely new to you (as it was new to me before this project) let me tell you I learned a lot about it by reading Gaston Sanchez's <a href="https://sagaofpls.github.io/"> The Saga of PLS </a>.


Basically, the steps needed to apply a PLS-R to your data are:

* Organize the features dataset
* Organize the response dataset (PLS-R in <a href="https://github.com/gastonstat/plsdepot">package plsdepot</a> can handle univariate and multivariate responses)
* Apply normalization to your data, since biological traits come in a variety of scales (cm, ind/m2, mm, counts, etc.)
* Use a PCA with the features dataset to check how they are related (I mean, the PLS-R is useless if there's no meaningful relationships between your predictors)
* Run the PLS-R with cross-validation
* Predict responses based on new data using the best models  


The PCA step of this analysis really surprised me. Of course, I was expecting to find some structure in the data since the cross-correlation between the tree features is well known, but I never thought the PCA was going to show me the development (or **ecological sucession**) of the mangrove sites so clearly. Check it out:

<img src="/blog/assets/images/development.png">


This is the PCA depicting the development (or succession) of the mangroves of Ajuruteua. Ten features were used to ordinate the sites (black dots), corresponding to five features of each mangrove plant, *Rhizophora mangle* (Rm) and *Avicennia germinans* (Ag). In the top-right set we see the sites composed of a huge density of very small thin individuals (actually, species of *Avicennia* often form monospecific stands of dwarf trees like these). From the lower-right to the upper-left, we see the transition from young sites to mature sites, and the forest changes are indicated by the arrows. "Young" sites are dominated by *Avicennia germinans* (high relative density of this species). As the forest transitions to "intermediate" sites, the relative density of *Avicennia germinans* decreases (the sites become mixed), and the tree size is bigger (diameter and height). In the "mature" sites, *Rhizophora mangle* dominates and its basal area is larger, indicating a higher density of large trees.  


Here's some useful functions to run the analysis with <a href="https://github.com/gastonstat/plsdepot">plsdepot</a> in R:  

```
library(plsdepot)

X = dataset[predictor_vars]   # set predictors/features
y = labels                    # set the response variable

# fit the model
pls1_model <- plsreg1(X, Y)

# inspect the results
pls1_model$R2          # Vector of PLS R-squared
pls1_model$cor.xyt     # Correlations between the variables and the PLS components
pls1_model$reg.coefs   # Vector of regression coefficients (original data scale)
pls1_model$Q2          # Table with the cross validation results.

# calculate the Variable Importance - useful for interpretability of the model
library(PLSbiplot1)
mod.VIP(X=X, Y=y, algorithm=mod.SIMPLS, A=2, cutoff=1)

# plot
plot(pls1_model)
```

Here's how my best models look like in numbers (i. e., the coefficients). The model of *Rhizophora mangle* is not nearly as good as the one for *Avicennia germinans*. I discuss the possible reasons for that in the paper (*spoiler alert:* I needed more data).

*Avicennia germinans* (R-squared = 0.85):
```
Model 4 (diameter + height + density + basal area + rel. density)  
LLAg=0.06247 – 0.01566 X1 + 0.00314 X2 - 0.00154 X3 + 0.14053 X4 + 0.17194 X5
R-squared = 0.85
```

*Rhizophora mangle* (R-squared = 0.66):
```
Model 4 diameter + height + density + basal area + rel. density
LLRm = −1.9959 + 0.0721 X1 + 0.9180 X2 + 0.0301 X3 - 0.4701 X4 + 0.2119 X5
```

Cool, huh?  
Once equations like these are obtained, we can **predict** the litterfall production of new sites that contain these two species, as long as we have the same features and species. Because I didn't have additional data to use with my models, I created a set of **artificial data** to experiment with my models.  

There's a cool trick here. In order to create a list of sites that made sense (i. e., had feature values that could actually occur in nature). As I said above, the diameter, height, density, and basal area of trees are highly correlated, and they co-vary within limits of each other. For instance, a stand with 40 m tall trees can't have a mean diameter of 2 cm. So, to generate **artificial but realistic data**, I used the function `mvrnorm()` from `Package MASS`.

`mvrnorm()` is a cool and useful function that generates **multivariate data** from a specified multivariate normal distribution. You can find a detailed explanation and examples <a href="https://blog.revolutionanalytics.com/2016/02/multivariate_data_with_r.html">in this very nice post </a>. Here's a simple example:


```
# calculate the mean value of each feature across sites
sim_sites.mean <- apply(X, 2, mean,  na.rm = TRUE)

# calculate the covariance between sites
sim_sites.cov <- cov(X, method="pearson")

set.seed(1)

#set the number of simulated sites
n <- 5000

# generate multivariate values based on a given mean and covariance matrix
require(MASS)
new.sites_sim <- mvrnorm(n, sim_sites.mean, sim_sites.cov, empirical=TRUE)
```

Here's a visualization of the results, now combining the PCA and the simulation of mangrove sites and prediction of annual litterfall using the PLS-r models:  


<img src="/blog/assets/images/predicted.png">  


Predicted annual leaf litterfall of *Avicennia germinans* and *Rhizophora mangle* (in megagrams of biomass per hectare per year). Each colored dot corresponds to a simulated site with a given set of mangrove tree features. The color is obtained by predicting the annual litterfall for that given mangrove structure, and then plotting it according to the scale. The black dots show the original sites used to build the PCA. The position of each dot (site) in the PCA space indicates the features of that simulated mangrove site.


How can this model be used? Ideally, we could visit a few mangrove stands, collect data from a few trees (species ID, height, diameter), and collect data about how the trees are distributed within each stand (density and basal area). Feeding this data into the model, __we could predict how much biomass this stand will produce in a year__.


But is this helpful? **YESSS it is!!**  
Consider that measuring, identifying, and counting trees takes a few hours or a few days (depending on forest size, accessibility, etc.), but estimating the annual biomass production __takes at least an year__. And researchers need to set traps to collect litterfall, visit the mangrove weekly or monthly for an year to empty them, and process the leaves in the laboratory (wash, dry, weight). **That's how annual litterfall is traditionally obtained**. As you can see, this is a lot of work. And understanding biomass production is crucial if we want to understand nutrient cycling in mangroves and the stability of carbon stocks.

However, this was only the first attempt to develop such a predictive model, and it is limited in many aspects. I hope it will serve as a "guide" for future studies based on more robust and comprehensive data. Science is always auto-correcting itself, and we learn a little bit more with every new model, table, dataset, or chart that comes around.


**References containing the vegetation and litterfall data:**

Abreu M.M.O. et al. 2006. Analysis of floristic composition and structure in a fragment of terra firme forest and an adjacent mangrove stand on Ajuruteua peninsula, Bragança, Pará. Boletim do Museu Paraense Emílio Goeldi 2: 27–34.

Fernandes, M.E.B., Nascimento, A.A.M., Carvalho, M.L., 2007. Estimativa da produçao anual de serapilheira dos bosques de mangue no Furo Grande, Bragança-Pará. Rev. Árvore 31, 949–958.

Mehlig U et al. 2010. Mangrove Vegetation of the Caeté Estuary. – In: Saint-Paul, U. and Schneider, H. (ed.), Mangrove Dynamics and Management in North Brazil. Ecological Studies, Springer, pp. 71–107.

Mehlig U. 2001. Aspects of tree primary production in an equatorial mangrove forest in Brazil. ZMT Contributions vol 14. PhD thesis, University of Bremen, Bremen. 155 p.

Menezes MPM et al. 2003. Annual growth rings and long-term growth patterns of mangrove trees from the Braganca peninsula, North Brazil. Wetlands Ecology and Management 11: 233–242.

Menezes MPM. 2006. Investigations of mangrove forest dynamics in Amazonia, North Brazil. PhD thesis, University of Bremen, Bremen.

Pereira MVS. 2005. Análise da estrutura florística de “bosques de Avicennia” na península de Ajuruteua, Bragança, Pará. Thesis, University of Pará, Bragança.

Reise A. 1999. Untersuchungen zum Streufall und Streuumsatz als Basis zur Charakterisierung des Stoffflusses in verschieden strukturierten Mangroven waldern Braganças/Nordostbrasiliens. Diploma thesis, University of Lüneburg, Lüneburg.

Reise A. 2003. Estimates of biomass and productivity in fringe mangroves on North-Brazil. PhD thesis, University of Bremen, ZMT Contribution 16, Bremen.

Seixas JAS et al. 2006. Análise estrutural da vegetação arbórea dos mangues no Furo Grande, Bragança-Pará. Boletim do Museu Paraense Emílio Goeldi Ciências Naturais 1: 61–69.
