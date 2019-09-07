---
layout: post
title:  "Quick script to analyze GCMS data and metabolomics"
author: aline
categories: [ data-science, ecology ]
image: assets/images/gcms.jpg

---


> Metabolomics

#### Processing and transforming raw GC-MS data in R

After the deconvolution of the original signal, the GC-MS exports a file containing

<img src="/blog/assets/images/file_header.png">

The goal is to identify common compounds across samples, based on retention time and mass, and taking into account that the retention times may vary slightly between diferent runs.

```
sample	c_1	c_2	c_3	c_4	c_5 ...
001	    0	  0	  0	  0	  1
002	    0	  0	  1	  0	  0
003	    0	  0	  0	  0	  0
004	    0	  0	  1	  0	  0
005	    0	  1	  0	  1	  0
006	    0	  0	  0	  0	  0
007	    0	  0	  1	  0	  0
021	    1   0   1   1   0
028	    0	  1	  0	  0	  0
029   	0	  1	  0	  0	  1
...

```

Import the original files (Excel format) using the package `xlsx` for R.

```
library(xlsx)

run_time <- 60  # set in

input_t <- read.xlsx("input.xlsx", 1 )
colnames(input_t) <- c("original_peak", "rt", "base_peak", "mz", "area", "sample_name")

# extract sample name from the file path in the column "File"
input_t$sample_name <- as.character(input_t$sample_name)
input_t$sample_name <- substr(input_t$sample_name, 35, 40)
input_t$sample_name <- as.factor(input_t$sample_name)

# small trick to allow the matching between retention times
input_t$rt <- as.character(round(input_t$rt, digits = 2))
```

This is the data structure:
```
str(input_t)

## 'data.frame':    5225 obs. of  6 variables:
##  $ original_peak: num  1 2 3 4 5 6 7 8 9 10 ...
##  $ rt           : chr  "6.16" "6.25" "6.41" "6.53" ...
##  $ base_peak    : num  105 110 117 96.1 103 115 108 97.1 107 91 ...
##  $ mz           : num  105 110 117 96.1 103 115 108 97.1 107 91 ...
##  $ area         : num  174324 2568353 256358 980743 425833 ...
##  $ sample_name  : Factor w/ 27 levels "leaf_1","leaf_2","leaf_3","leaf_4",..:
```
Create the dataframe that will keep the rows sorted per retention time

```
dataset_rt <- data.frame(sequential_rt=1:6000, stringsAsFactors = FALSE)
seqs <- seq(0.01, run_time, by=0.01)
dataset_rt$sequential_rt <- as.character(seqs, digits = 2)
```
Organize the dataset

```
options(digits = 3)
for (l in 1:nlevels(input_t$sample_name)) {
  sub <- input_t[input_t$sample_name == levels(input_t$sample_name)[l], c(1,2, 4)]
  #create two columns to receive ID and MASS of this sample
  colu <- ncol(dataset_rt)
  dataset_rt[, colu+1] <- 0
  dataset_rt[, colu+2] <- 0
  colnames(dataset_rt)[colu+1] <- paste(as.character(levels(input_t$sample_name)[l]), "__ID")
  colnames(dataset_rt)[colu+2] <- paste(as.character(levels(input_t$sample_name)[l]), "__mz")
  for (lin in 2 : nrow(sub)) {  
    lin2 <- match(sub[lin, "rt"], dataset_rt$sequential_rt)
    dataset_rt[lin2, colu+1] <- sub[lin, "original_peak"]                # ID = peak number
    dataset_rt[lin2, colu+2] <-  as.double(sub[lin, "mz"], digits = 1)   # mass
  }
}
dataset_rt_red<-data.frame(stringsAsFactors = FALSE)
co<-0
nc <- ncol(dataset_rt)

#trim  lines with only zeros
for (m in 1:nrow(dataset_rt)){
  if (sum(dataset_rt[m, 2:nc]) != 0)  {
   co <- co + 1
   dataset_rt_red <- rbind(dataset_rt_red, dataset_rt[m,])
  }
}

dataset_rt_red <- dataset_rt_red[, seq(from = 3, to = nc, by=2) ]
```
Set the variation in retention time to be used as a buffer

```
buf <- 5
nc2 <- ncol(dataset_rt_red)
```

Create dataset of all compounds taking the buffer above into account

```
dataset_list <- data.frame(stringsAsFactors = FALSE)
dataset_list_ord <- data.frame(stringsAsFactors = FALSE)

for (c in 1:nc2){
  for (l in 1:nrow(dataset_rt_red)) {
      if (dataset_rt_red[l,c]!= 0) {
        newrec <- c(rownames(dataset_rt_red)[l], dataset_rt_red[l,c])
        dataset_list <- rbind(dataset_list, as.numeric(newrec))
      }
  }
}
colnames(dataset_list) <- c("ret_time", "mass")
dataset_list_ord <- dataset_list[order(dataset_list$mass, dataset_list$ret_time),]

# eliminate exact duplicates
dataset_list_ord <- dataset_list_ord[!duplicated(dataset_list_ord), ]

# merge according to buffer size
dataset_compounds <- data.frame(stringsAsFactors = FALSE)

cind <- 0
p<-1

while (p <= nrow(dataset_list_ord)){
  # add coumpound
  cind <- cind + 1

  dataset_compounds[cind, "name"] <- paste("compound_", cind, sep="")
  dataset_compounds[cind, "min_ret_time"] <-  dataset_list_ord[p, "ret_time"]
  dataset_compounds[cind, "max_ret_time"]  <- dataset_list_ord[p, "ret_time"]
  dataset_compounds[cind, "mass"] <- dataset_list_ord[p, "mass"]

  # check next rows according to buffer to see if there's more
  n_added <- 0
  for (b in 1:buf-1) {
    if (dataset_list_ord[p, "mass"] == dataset_list_ord[p+b, "mass"]){
      #next line is same compund - update ret time
      dif <- dataset_list_ord[p+b, "ret_time"] - dataset_compounds[cind, "min_ret_time"]
      if (dif <=5 & dif >=0){
          dataset_compounds[cind, "max_ret_time"] <- dataset_list_ord[p+b, "ret_time"]
          n_added <- n_added+1   
      }
    } else {
      break
    }
  }
  p <- p+1+n_added
}
```

Finally, create a matrix with the presence/absence of each compound in each sample

```
dataset_final <- matrix(nrow=ncol(dataset_rt_red), ncol=nrow(dataset_compounds)+1)
dataset_final <- as.data.frame(dataset_final)
colnames(dataset_final)[1] <- "sample"
colnames(dataset_final)[2:nrow(dataset_compounds)+1] <- dataset_compounds$name
i <-1
for (cc in 1:ncol(dataset_rt_red)){
  # read each sample
  dataset_final[i, "sample"] <- colnames(dataset_rt_red)[cc]
  for (nr in 1:nrow(dataset_rt_red))
    if (dataset_rt_red[nr,cc] != 0){
      ma <- dataset_rt_red[nr,cc]
      rt <- as.numeric(rownames(dataset_rt_red)[nr])
      com_id <- dataset_compounds[dataset_compounds$mass == ma
                                  & dataset_compounds$min_ret_time <= rt
                                  & dataset_compounds$max_ret_time >= rt , "name"]
      dataset_final[i, com_id] <- 1
    }
  i<-i+1
}
dataset_final[is.na(dataset_final[,]) ] <- 0
```
Here's how the `dataset_final` looks like:

```
sample	V2	compound_1	compound_2	compound_3	compound_4	compound_5
1	#NAME?	0	0	0	0	0	0
2	_125.D __mz	0	0	0	0	1	0
3	_158.D __mz	0	0	1	0	0	0
4	_175.D __mz	0	0	0	0	0	0
5	_177.D __mz	0	0	1	0	0	0
6	_287.D __mz	0	1	0	0	0	0
7	_293.D __mz	0	0	0	0	0	0
8	120.D __mz	0	0	0	0	0	0
9	121.D __mz	0	0	0	0	0	0
10	122.D __mz	0	0	0	0	1	0


```

#### Multivariate analyses

Now that you transformed the raw GC-MS data into a presence/absence matrix of **compounds x samples**,
you can use several multivariate analyses to compare your sample's composition and to know more about them.
Here's an example of a simple **cluster analysis**, which clusters samples based on a measure of dissimilarity (distance). The function `hclust()` does that in R:

```
datt <- dataset_final[, 3:ncol(dataset_final)]
rownames(datt) <- dataset_final$sample
simi <- hclust(dist(datt))
plot(simi)
```
<img src="/blog/assets/images/cluster.jpg">
